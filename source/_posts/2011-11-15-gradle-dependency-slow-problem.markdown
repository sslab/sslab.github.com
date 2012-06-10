---
layout: post
title: "解決 Gradle dependency 處理速度慢的問題"
date: 2011-11-15 09:42
comments: true
categories: Groovy
---

Gradle 的 dependencies 可以設定 Maven Repository 的套件，
讓 Gradle 自動下載相依的 Jar 檔案。
例如 [HttpBuilder](http://groovy.codehaus.org/modules/http-builder/) 的設定範例（build.gradle）：

``` groovy
dependencies {
    groovy 'org.codehaus.groovy:groovy-all:1.8.3'
    compile 'org.codehaus.groovy.modules.http-builder:http-builder:0.5.1'
}
```

但有時候加入了某些相依套件，
會讓建置程序變得超級慢，
因此在 gradle task 執行時，
會卡在「Resolve dependencies」這個訊息很久。

```
> Building > :compileGroovy > Resolve dependencies ':compile'
```

利用以下的指令，可以檢查目前專案依賴的套件共有哪些。

```
gradle dependencies
```

執行後將會看到這個輸出結果：

```
compile - Classpath for compiling the main sources.
+--- org.codehaus.groovy:groovy-all:1.8.3 [default]
\--- org.codehaus.groovy.modules.http-builder:http-builder:0.5.1 [default]
     +--- org.apache.httpcomponents:httpclient:4.0.3 [master,compile,runtime]
     |    +--- org.apache.httpcomponents:httpcore:4.0.1 [master,compile,runtime]
     |    +--- commons-logging:commons-logging:1.1.1 [master,compile,runtime]
     |    \--- commons-codec:commons-codec:1.3 [master,compile,runtime]
     +--- net.sf.json-lib:json-lib:2.3 [master,compile,runtime]
     |    +--- commons-logging:commons-logging:1.1.1 [master,compile,runtime] (*)
     |    +--- commons-beanutils:commons-beanutils:1.8.0 [master,compile,runtime]
     |    |    \--- commons-logging:commons-logging:1.1.1 [master,compile,runtime] (*)
     |    +--- commons-collections:commons-collections:3.2.1 [master,compile,runtime]
     |    +--- commons-lang:commons-lang:2.4 [master,compile,runtime]
     |    \--- net.sf.ezmorph:ezmorph:1.0.6 [master,compile,runtime]
     |         \--- commons-lang:commons-lang:2.4 [master,compile,runtime] (*)
     +--- org.codehaus.groovy:groovy:1.7.10 [master,compile,runtime]
     |    +--- antlr:antlr:2.7.7 [master,compile,runtime]
     |    +--- asm:asm:3.2 [master,compile,runtime]
     |    +--- asm:asm-commons:3.2 [master,compile,runtime]
     |    |    \--- asm:asm-tree:3.2 [master,compile,runtime]
     |    |         \--- asm:asm:3.2 [master,compile,runtime] (*)
     |    +--- asm:asm-util:3.2 [master,compile,runtime]
     |    |    \--- asm:asm-tree:3.2 [master,compile,runtime] (*)
     |    +--- asm:asm-analysis:3.2 [master,compile,runtime]
     |    |    \--- asm:asm-tree:3.2 [master,compile,runtime] (*)
     |    \--- asm:asm-tree:3.2 [master,compile,runtime] (*)
     +--- net.sourceforge.nekohtml:nekohtml:1.9.9 [master,compile,runtime]
     |    \--- xerces:xercesImpl:2.8.1 [master,compile,runtime]
     |         \--- xml-apis:xml-apis:1.3.03 [master,compile,runtime]
     \--- xml-resolver:xml-resolver:1.2 [master,compile,runtime]
```

從樹狀圖可以發現，
http-builder 依賴的套件中，
groovy:1.7.10 是多餘且冗長的部份。
我們已經在另一行指定 groovy-all:1.8.3 這個版本，
因此可以利用 exclude 設定將 groovy:1.7.10 排除。

``` groovy
dependencies {
    groovy 'org.codehaus.groovy:groovy-all:1.8.3'
    compile ('org.codehaus.groovy.modules.http-builder:http-builder:0.5.1')  {
        exclude group: 'org.codehaus.groovy', modules: 'groovy'
    }
}
```

再重新執行一次 gradle dependencies，
可以發現樹狀圖變得比較精簡，
而 gradle 執行過程也會加快許多，
不會在 Resolve dependencies 時卡住。

```
+--- org.codehaus.groovy:groovy-all:1.8.3 [default]
\--- org.codehaus.groovy.modules.http-builder:http-builder:0.5.1 [default]
     +--- org.apache.httpcomponents:httpclient:4.0.3 [master,compile,runtime]
     |    +--- org.apache.httpcomponents:httpcore:4.0.1 [master,compile,runtime]
     |    +--- commons-logging:commons-logging:1.1.1 [master,compile,runtime]
     |    \--- commons-codec:commons-codec:1.3 [master,compile,runtime]
     +--- net.sf.json-lib:json-lib:2.3 [master,compile,runtime]
     |    +--- commons-logging:commons-logging:1.1.1 [master,compile,runtime] (*)
     |    +--- commons-beanutils:commons-beanutils:1.8.0 [master,compile,runtime]
     |    |    \--- commons-logging:commons-logging:1.1.1 [master,compile,runtime] (*)
     |    +--- commons-collections:commons-collections:3.2.1 [master,compile,runtime]
     |    +--- commons-lang:commons-lang:2.4 [master,compile,runtime]
     |    \--- net.sf.ezmorph:ezmorph:1.0.6 [master,compile,runtime]
     |         \--- commons-lang:commons-lang:2.4 [master,compile,runtime] (*)
     +--- net.sourceforge.nekohtml:nekohtml:1.9.9 [master,compile,runtime]
     |    \--- xerces:xercesImpl:2.8.1 [master,compile,runtime]
     |         \--- xml-apis:xml-apis:1.3.03 [master,compile,runtime]
     \--- xml-resolver:xml-resolver:1.2 [master,compile,runtime]
```
