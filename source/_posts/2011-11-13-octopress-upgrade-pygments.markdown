---
layout: post
title: "讓 Octopress 支援更多語言的程式碼區塊（更新 Pygments）"
date: 2011-11-13 16:00
comments: true
categories: Octopress
---

如果在 Octopress 的文章中，程式碼區塊用了不支援的程式語言，例如 Groovy：

```
 ``` groovy
 println 'hello'
 ```
```

執行 rake generate 就會爆炸！

> /Users/user1/.rvm/gems/ruby-1.9.2-p290/gems/rubypython-0.5.1/lib/rubypython/rubypyproxy.rb:198:in `method_missing': ClassNotFound: no lexer for alias 'groovy' found (RubyPython::PythonError)

因為 Octopress 的程式碼區塊 Syntax Highlight，
是使用 [pygments.rb](https://github.com/tmm1/pygments.rb) 這個套件；
而 pygments.rb 會再去呼叫 Python 的 [Pygments](http://pygments.org/)，
因此災難的源頭就是 Pygments 不支援區塊指定的程式語言。

但如果從 Pygments 的 [Supported languages](http://pygments.org/languages/) 清單，
卻又可能發現清單中明明有列出該語言名稱。
這時候必須到 [Available lexers](http://pygments.org/docs/lexers/) 查詢，
如果在程式語言的說明有一行 **New in Pygments 1.5.** ，
就表示只要將 Pygments 更新到 1.5 版，
即可支援該程式語言（目前 Pygments 的穩定版為 1.4）。

更新的步驟如下：

先切換到 pygments.rb 資料夾

``` bash
cd .rvm/gems/ruby-1.9.2-p290/gems/pygments.rb-0.1.3/vendor
```

用 hg 下載一份最新版 [Pygments](https://bitbucket.org/birkenfeld/pygments-main)，

``` bash
hg clone https://bitbucket.org/birkenfeld/pygments-main
```

再將 Pygments-1.4 的資料夾掉包

``` bash
mv Pygments-1.4 Pygments-1.4-old
mv pygments-main Pygments-1.4
```

回到 Octopress Blog 重新執行 rake generate，應該就不會爆炸了。
