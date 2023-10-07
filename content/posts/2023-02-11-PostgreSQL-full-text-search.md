---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2023-02-11T11:38:27+08:00"
description: PostgreSQL 内置全文搜索记录
lastmod: "2023-02-11T12:06:40+08:00"
showToc: true
tags: [PostgreSQL]
title: PostgreSQL 全文搜索
---

# PostgreSQL 全文搜索

在数据量较小的项目中可以使用 PostgreSQL 自带的全文搜索（Full Text Search）支持，代替非常重的 ElasticSearch，减少开发和维护成本，简单又好用，记录下最近的学习

实现全文搜索主要分为几步，**分词**、**向量化**、**创建索引（倒排索引）**、**匹配查询**

## 分词

默认自带的分词配置是不支持中文的，但可以安装第三方扩展来支持，检查支持的配置库在 psql 中使用 `\dF` 命令，`\dFp` 列出解析器 。

```sql
postgres=> \dF
               List of text search configurations
   Schema   |    Name    |              Description
------------+------------+---------------------------------------
 pg_catalog | danish     | configuration for danish language
 pg_catalog | dutch      | configuration for dutch language
 pg_catalog | english    | configuration for english language
 pg_catalog | portuguese | configuration for portuguese language
 pg_catalog | russian    | configuration for russian language
 pg_catalog | simple     | simple configuration
 pg_catalog | spanish    | configuration for spanish language

 postgres-# \dFp
        List of text search parsers
   Schema   |  Name   |     Description
------------+---------+---------------------
 pg_catalog | default | default word parser
```

### 安装支持中文的扩展

支持中文的扩展有以下这两个

- [pg_jieba](https://github.com/jaiminpan/pg_jieba)
- [zhparser](https://github.com/amutu/zhparser)

根据上面文档安装完后查看，zhparser 和结巴都能用了

```sql
postgres=# \dF
                  List of text search configurations
   Schema   |    Name    |                Description
------------+------------+--------------------------------------------
 ...
 public     | jiebacfg   | Mix segmentation configuration for jieba
 public     | jiebahmm   | Hmm segmentation configuration for jieba
 public     | jiebamp    | MP segmentation configuration for jieba
 public     | jiebaqry   | Query segmentation configuration for jieba
 public     | testzhcfg  |

postgres=# \dFp
         List of text search parsers
   Schema   |   Name   |     Description
------------+----------+---------------------
 ...
 public     | jieba    |
 public     | jiebahmm |
 public     | jiebamp  |
 public     | jiebaqry |
 public     | zhparser |

```

## 向量化

### 文本向量化

我们要全文搜索的一般是我们表中的某个字符串字段，必须先向量化此字段，可以直接计算向量也可以存储在新的一列中，该向量的数据类型是 `tsvector` 它是词和位置组成的列表，用来保存的是分词后的结果 (文本向量)，这两种方式都可以建索引加快查询时间。

```sql
postgres=> select 'hello world hello world'::tsvector;
    tsvector
-----------------
 'hello' 'world'
```

字符串转化成 `tsvector` 会以空格分隔去掉重复的词条，按照一定的顺序装入，也可以设置位置

```sql
postgres=> select 'hello:2 world:1'::tsvector;
      tsvector
---------------------
 'hello':2 'world':1
```

一般我们使用 `to_tsvector(regconfig, text)` 函数分词并构建 `tsvecotr` ， `regconfig` 参数必须在上文 `\dF` 返回的配置语言

```sql
postgres=> select to_tsvector('english', 'hello a world, hello worlds !');
       to_tsvector
-------------------------
 'hello':1,4 'world':3,5
```

如果分词在程序中实现那 `regconfig` 使用 `simple` 就可以，`text` 中传分好的词然后构建向量。上面返回的向量 _hello_ 出现在 1，4 的位置，_world_ 出现在 3，5 的位置，分词忽略了量词、标点、复数

如果需要搜索两个字段可以使用 `||` 符号连接字符串后传给 `to_tsvector`

```sql
to_tsvector('english', coalesce(col1, '') || ' ' || coalesce(col2, ''))
```

还有一种方法是使用 `setweight` 函数，可以单独指定每列的权重，从 A - D 权重依次递减

```sql
setweight(to_tsvector('english', coalesce(col1,'')), 'A') || setweight(to_tsvector('english', coalesce(col2,'')), 'B')
```

在工程上我们可以创建单独的一列存储 `tsvector` 然后创建索引，详见下文。

### 搜索关键字向量化

查询的关键词也需要向量化类型是 `tsquery` ，`tsquery` 相当于是查询 `tsvector` 的查询条件然后通这条件去搜索向量。`tsquery` 可以组合多种操作符  `&` (AND), `|` (OR), `!` (NOT), and `<->` (FOLLOWED BY)， `@@` 是全文搜索匹配操作符参考下文搜索

```sql
postgres=> select 'hello world'::tsvector @@ 'hello | happy '::tsquery;
 ?column?
----------
 t
```

我们一般不直接转成 `tsquery` 来查询，而使用 PostgreSQL 提供的 `to_tsquery(regconfig, querytext)` 函数来将词组织成 `tsquery` 向量，`querytext` 为搜索语句必须为单个关键词或者多个 词和 `&` 、`|` 、`!` 等组合，如上是匹配 `hello` 或 `happy` 词。

```sql
postgres=> select to_tsquery('simple', 'hello | happy');
    to_tsquery
-------------------
 'hello' | 'happy'

 postgres=> select 'hello world'::tsvector @@ to_tsquery('simple', 'hello | happy');
 ?column?
----------
 t

postgres=> select 'hello world'::tsvector @@ to_tsquery('simple', 'hello & happy');
 ?column?
----------
 f
```

> `to_tsquery` , `to_tsvecotr` 等函数参数中的第一个参数配置文件（`regconfig`）可以忽略，这样就使用 PostgreSQL 默认的 `default_text_search_config` 配置参数

```sql
postgres=# show default_text_search_config;
 default_text_search_config
----------------------------
 pg_catalog.english
```

## 搜索

PostgreSQL 中的全文搜索基于匹配操作符 `@@` ，如果`tsvector`(文档)与`tsquery`(查询)匹配，则返回`true`。两个位置无所谓可以调换。

```sql
postgres=> select to_tsvector('Cats are very cute') @@ to_tsquery('cats');
 ?column?
----------
 t

postgres=> select 'Cats are very cute'::tsvector @@ to_tsquery('cats');
 ?column?
----------
 f
```

注意上文的两个查询，第一个能匹配上是使用了 `to_tsvector` 进行了分词转 `tsvector`，结果中会有 `cat` 词素，第二没有使用则只有 `Cats`

`@@` 操作符还有以下几种写法

```sql
tsvector @@ tsquery
tsquery  @@ tsvector
text @@ tsquery
text @@ text
```

- 第一种和第二种相等
- 第三种等于 `to_tsvector(text) @@ tsquery`
- 第四种等于 `to_tsvector(text) @@ plainto_tsquery(text)`.

## 索引

`tsvector` 支持的索引分为 `gin` 和 `gist` 索引，GIST 索引是有损的，构建速度快。GIN 索引构建速度慢点但查询更快，推荐使用 GIN

- 创建 GIN 索引。列必须为 tsvector 类型。

`CREATE INDEX name ON table USING GIN (column);`

- 创建基于 GiST 的索引。该列可以是 `tsvector` 或 `tsquery` 类型。

`CREATE INDEX name ON table USING GIST (column);`

## 排序

对于搜索结果的排序 PostgreSQL 提供了下面两个函数

```sql
ts_rank(tsvector, tsquery) returns float4
ts_rank_cd(tsvector, tsquery) returns float4
```

`ts_rank` 函数依据 _tsquery_ 和 _tsvector_ 考虑查询在搜索字段中出现的频率、位置、相关性等信息计算出一个值，`ts_rank_cd` 和 `ts_rank` 基本一致只是考虑了匹配词之间的覆盖密度（_cover density_）。

## 示例

下面看个例子，先创建一张表

```sql
create table post (
	id serial primary key,
	title varchar(128) not null,
	subtitle varchar(256),
	body text
);
```

创建 `search_vector` 列存储 `tsvector` 然后创建索引

```sql
alter table post
    add column search_vector tsvector
               generated always as (to_tsvector('english', coalesce(title, '') || ' ' || coalesce(subtitle, ''))) stored;
create index search_vector_idx on post using gin (search_vector);
```

上面使用了 `generated always` 在插入和更新的时候都会更新 `search_vector` 列，拼接了 `title` 和 `subtitle` 同时搜索这两列，也可以使用 `setweight` 替换成

```sql
... as (setweight(to_tsvector('english', coalesce(title,'')), 'A') || setweight(to_tsvector('english', coalesce(subtitle,'')), 'B'))) stored;
```

> 上面使用 `generated always` 自动更新 tsvector ，适用 PostgreSQL 版本大于 11 的，如果版在 11 及以下的可以配合 trigger 使用官方也提供了两个 `tsvector_update_trigger` 和 `tsvector_update_trigger_column` 也可以使用自定义的参考[官方文档](https://www.postgresql.org/docs/current/textsearch-features.html)

我们为 `search_vector` 列创建了 gin 索引，最后使用下面的 sql 就能搜索了

```sql
select * from post where search_vector @@ to_tsquery('english', 'querytext');
```

如果我们不使用单独的列存储 `tsvector` 列，也可以使用索引用下面的 sql 完成

```sql
create index search_vector_bare_idx on post using gin (to_tsvector('english', coalesce(title, '') || ' ' || coalesce(subtitle, '')));
select * from post where to_tsvector('english', coalesce(title, '') || ' ' || coalesce(subtitle, '')) @@ to_tsquery('english', 'querytext');
```

查询的时候必须也指定 `to_tsvector(regconfig, text)` 不能忽略 regconfig 。上面虽然没有占用一列存 `tsvector` ，但不建议使用这种方式，因为新加一列可以不需要显示指定 regconfig 也就是使用 `to_tsquery(text)` 的方式，已经预先计算完了 `tsvector` 所以查询效率更高。

常用的会加上排序，就是下面这样的 SQL

```sql
select * from
	post,
	to_tsquery('english', 'querytext1 | querytext2') query
where
	query @@ search_vector
order by
	ts_rank_cd(search_vector, query) desc;
```

## Reference

- [https://www.postgresql.org/docs/current/textsearch.html](https://www.postgresql.org/docs/current/textsearch.html)
- [https://www.jianshu.com/p/0dc2a8bf9631](https://www.jianshu.com/p/0dc2a8bf9631)
