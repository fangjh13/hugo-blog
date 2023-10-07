---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2018-11-08T03:11:51+08:00"
lastmod: "2020-07-20T03:17:00+08:00"
showToc: true
tags: [Mariadb, Shell, Database, Mysql]
title: Mysql数据库备份脚本(使用mysqldump)
---

## Mysql(mysqldump)备份脚本

记录下服务器上一个备份 mysql 数据库的脚本，使用 mysql 自带的`mysqldump`命令

```shell
!/usr/bin/env bash

USER=username
PASSWORD=password
MAXIMUM_BACKUP_FILES=10
BACKUP_FOLDER=/path/to/save/folder
DATABASES=(
db_name_0
db_name_1
)


# check mysqldump instlled
_=$(command -v mysqldump)

if [[ $? != 0 ]]
then
    printf "You don't seem to mysqldump installed, exit..\n"
    exit 1
fi

# create backup folder
if [ ! -d $BACKUP_FOLDER ]
then
    mkdir $BACKUP_FOLDER
fi

# backup
for DB in ${DATABASES[@]}
do
    echo backing up ${DB} database ...
    if $(mysqldump --host=localhost --user=${USER} --password=${PASSWORD} ${DB} | gzip -9 > ${BACKUP_FOLDER}/db_${DB}_$(date +"%Y%m%d").sql.gz)
    then
        echo dump db_${DB}_$(date +"%Y%m%d").sql.gz done.
    else
        echo dump db_${DB}_$(date +"%Y%m%d").sql.gz failed.
    fi
done

# remove older files
find ${BACKUP_FOLDER} -type f -name *.sql.gz -mtime +${MAXIMUM_BACKUP_FILES} -delete
```

脚本就是使用 mysqldump 备份指定的数据库（在 DATABASES 使用空格分隔）然后 gzip 压缩保存到指定目录，使用系统自带的 find 命令删除旧文件

最后加入 crontab，设置每天凌晨 3 点备份

```shell
* 3 * * * bash /path/to/mysql_backup.sh
```

数据恢复可以使用如下命令

```shell
gzip -dc db_dbName_20181012.sql.gz | mysql -u userName -p dbName
```

如果跑在 docker 环境下可用如下脚本

```shell
#!/usr/bin/env bash

CONTAINER_NAME=docker-db-container-name
USER=username
PASSWORD=password
MAXIMUM_BACKUP_FILES=10
BACKUP_FOLDER=/path/to/save/folder
DATABASES=(
db_name_0
db_name_1
)

# create backup folder
if [ ! -d $BACKUP_FOLDER ]; then
        mkdir $BACKUP_FOLDER
fi

# check container running
n=$(docker ps -f name=${CONTAINER_NAME} -q | wc -l)
if [ "$n" -eq 0 ]; then
        echo "Contaniner ${CONTAINER_NAME} is not running, please check it"
        exit 1
fi

# backup
for DB in ${DATABASES[@]}
do
    echo backing up ${DB} database ...
    EXEC_SH="docker exec -i ${CONTAINER_NAME} sh -c 'mysqldump --user=${USER} --password=${PASSWORD} ${DB}' | gzip -9 > ${BACKUP_FOLDER}/db_${DB}_$(date +"%Y%m%d").sql.gz"
    eval $EXEC_SH
    if [ $? -eq 0 ]
    then
        echo dump db_${DB}_$(date +"%Y%m%d").sql.gz done.
    else
        echo dump db_${DB}_$(date +"%Y%m%d").sql.gz failed.
    fi
done

# remove older files
find ${BACKUP_FOLDER} -type f -name *.sql.gz -mtime +${MAXIMUM_BACKUP_FILES} -delete
```
