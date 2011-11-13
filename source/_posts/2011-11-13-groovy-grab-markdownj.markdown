---
layout: post
title: "MarkdownJ 使用範例（Groovy + Grab）"
date: 2011-11-13 09:40
comments: true
categories: Groovy
---

[Markdown](http://lyhdev.com/note:markdown) 是一種易讀易寫的輕量文字格式。

要將 Markdown 轉換成其他格式，除了命令列的 [Pandoc](http://johnmacfarlane.net/pandoc/) 工具外，
Java / Groovy 程式可以利用 MarkdownJ 這個函式庫。

在這個 [頁面](http://code.google.com/p/markdownj/wiki/Maven) 可以找到 MarkdownJ 的 Maven Repository 來源，
因此加入一行 Groovy 的 Grab 設定：

``` groovy
@GrabResolver(name='scala-tools', root='http://scala-tools.org/repo-releases')
@Grab('org.markdownj:markdownj:0.3.0-1.0.2b4')
```

接著就可以引用 MarkdownProcessor 類別。

``` groovy
import com.petebevin.markdown.MarkdownProcessor
```

使用 MarkdownProcessor 的 markdown 方法，可以將一段 Markdown 格式的字串轉換成 HTML 代碼。

``` groovy
def m = new MarkdownProcessor()

println m.markdown('''
Heading
=======

* item1
* item2
* item3
''')
```

範例程式碼可以從 [Gist](https://gist.github.com/1361439) 取得。

