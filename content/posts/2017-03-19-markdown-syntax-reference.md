---
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowReadingTime: true
ShowRssButtonInSectionTermList: true
ShowWordCount: true
TocOpen: false
UseHugoToc: true
date: "2017-03-19T09:59:44+08:00"
description: Markdown quick reference
lastmod: "2020-04-11T08:50:22+08:00"
showToc: true
tags: [Markdown]
title: Markdown 语法说明
---

# Markdown quick reference

> See the Markdown page for instructions on enabling Markdown for posts, pages and comments on your blog, and for more detailed information about using Markdown.

## Syntax

### Emphasis

`*This text will be italic*`

_This text will be italic_

`_This will also be italic_`

_This will also be italic_

`**This text will be bold**`

**This text will be bold**

`__This will also be bold__`

**This will also be bold**

`_You **can** combine them_`

_You **can** combine them_

### Inline Links

`A [link](https://www.google.com "Google").`A [link](https://www.google.com "Google").

### Referenced Links

    Some text with [a link][1] and another [link][2].

    [1]: http://example.com/ "Title"
    [2]: http://example.org/ "Title"

Some text with [a link][1] and another [link][2].

[1]: http://example.com/ "Title"
[2]: http://example.org/ "Title"

### Inline Images

`Logo: ![favicon](../images/favicon.ico "Title")`

Logo: ![favicon](../images/favicon.ico "Title")

### Linked Images

`Linked logo: [![Fython](../images/favicon.ico "Fython")](https://www.google.com/ "Fython")`

Linked logo: [![Fython](../images/favicon.ico "Fython")](https://www.google.com/ "Fython")

### Unordered Lists

    * Item
    * Item
    - Item
    - Item

- Item
- Item

* Item
* Item

### Ordered Lists

    1. Item
    2. Item
    3. Item

1. Item
2. Item
3. Item

### Blockquotes

    > Quoted text.
    > > Quoted quote.

    > * Quoted
    > * List

> Quoted text.
>
> > Quoted quote.

> - Quoted
> - List

### Inline Code

```
`This is code` This is text
```

`This is code` This is text

### Code block

Code block with backticks

<pre>
```
    This is a
    piece of code 
    in a block
```
</pre>

```
    This is a
    piece of code
    in a block
```

indented with four spaces

<pre>
    This is a
    piece of code
    in a block
</pre>

    This is a
    piece of code
    in a block

### Heads

    # Header 1
    ## Header 2
    ### Header 3
    #### Header 4
    ##### Header 5
    ###### Header 6

# Header 1

## Header 2

### Header 3

#### Header 4

##### Header 5

###### Header 6

---

## Extend Markdown

### Tables

You can create tables by assembling a list of words and dividing them with hyphens - (for the first row), and then separating each column with a pipe |:

```
First Header | Second Header
------------ | -------------
Content from cell 1 | Content from cell 2
Content in the first column | Content in the second column
```

| First Header                | Second Header                |
| --------------------------- | ---------------------------- |
| Content from cell 1         | Content from cell 2          |
| Content in the first column | Content in the second column |

### Strikethrough

```
~~this~~
```

Any word wrapped with two tildes (like ~~this~~) will appear crossed out.

### Syntax highlighting in code fences

    ```python {linenos=true,hl_lines=[1,"3-4"],linenostart=1}
    def foo():
        if not bar:
            print("hello world")
            return True
    ```

Here’s an example

```python {linenos=true,hl_lines=[1,"3-4"],linenostart=1}
def foo():
    if not bar:
        print("hello world")
        return True
```

### Code block with Hugo's internal highlight shortcode

    {{</* highlight html */>}}

    <!doctype html>
    <html lang="en">
    <head>
      <meta charset="utf-8">
      <title>Example HTML5 Document</title>
    </head>
    <body>
      <p>Test</p>
    </body>
    </html>
    {{</* /highlight */>}}

{{< highlight html >}}

<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Example HTML5 Document</title>
</head>
<body>
  <p>Test</p>
</body>
</html>
{{< /highlight >}}

highlight line

    {{</* highlight go "linenos=table,hl_lines=8 15-17,linenostart=199" */>}}

    // GetTitleFunc returns a func that can be used to transform a string to
    // title case.
    //
    // The supported styles are
    //
    // - "Go" (strings.Title)
    // - "AP" (see https://www.apstylebook.com/)
    // - "Chicago" (see https://www.chicagomanualofstyle.org/home.html)
    //
    // If an unknown or empty style is provided, AP style is what you get.
    func GetTitleFunc(style string) func(s string) string {
    switch strings.ToLower(style) {
    case "go":
    return strings.Title
    case "chicago":
    return transform.NewTitleConverter(transform.ChicagoStyle)
    default:
    return transform.NewTitleConverter(transform.APStyle)
    }
    }
    {{</* / highlight */>}}

{{< highlight go "linenos=table,hl_lines=8 15-17,linenostart=199" >}}

// GetTitleFunc returns a func that can be used to transform a string to
// title case.
//
// The supported styles are
//
// - "Go" (strings.Title)
// - "AP" (see https://www.apstylebook.com/)
// - "Chicago" (see https://www.chicagomanualofstyle.org/home.html)
//
// If an unknown or empty style is provided, AP style is what you get.
func GetTitleFunc(style string) func(s string) string {
switch strings.ToLower(style) {
case "go":
return strings.Title
case "chicago":
return transform.NewTitleConverter(transform.ChicagoStyle)
default:
return transform.NewTitleConverter(transform.APStyle)
}
}
{{< / highlight >}}

### Blockquote with attribution

    > Don't communicate by sharing memory, share memory by communicating.
    >
    > — <cite>Rob Pike[^1]</cite>

    [^1]: The above quote is excerpted from Rob Pike's [talk](https://www.youtube.com/watch?v=PAAkCSZUG1c) during Gopherfest, November 18, 2015.

> Don't communicate by sharing memory, share memory by communicating.
>
> — <cite>Rob Pike[^1]</cite>

[^1]: The above quote is excerpted from Rob Pike's [talk](https://www.youtube.com/watch?v=PAAkCSZUG1c) during Gopherfest, November 18, 2015.

### Gist

{{< gist spf13 7896402 >}}

### Task List

```
- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> supported
- [x] list syntax required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item
```

- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> supported
- [x] list syntax required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item

### Other Elements — abbr, sub, sup, kbd, mark

<abbr title="Graphics Interchange Format">GIF</abbr> is a bitmap image format.

H<sub>2</sub>O

X<sup>n</sup> + Y<sup>n</sup> = Z<sup>n</sup>

Press <kbd><kbd>CTRL</kbd>+<kbd>ALT</kbd>+<kbd>Delete</kbd></kbd> to end the session.

Most <mark>salamanders</mark> are nocturnal, and hunt for insects, worms, and other small creatures.
