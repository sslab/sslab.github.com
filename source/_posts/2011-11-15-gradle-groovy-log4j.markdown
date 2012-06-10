---
layout: post
title: "Gradle + Groovy 1.8 + Log4j"
date: 2011-11-15 11:44
comments: true
categories: Groovy 
---

Groovy 從 1.8 版本開始支援 @Log 標記（Annotation），共有四種：

* @Log for java.util.logging
* @Commons for Commons-Logging
* @Log4j for Log4J
* @Slf4j for SLF4J

這篇教學以 Apache Log4j 為例，搭配 Gradle 專案建置 Script。

先在 build.gradle 設定 dependencies，加入 groovy 1.8 及 log4j 1.2 的設定。

``` groovy
apply plugin: 'java'
apply plugin: 'groovy'

dependencies {
	groovy 'org.codehaus.groovy:groovy-all:1.8.3'
	runtime 'log4j:log4j:1.2.16'
}
```

在 class 上方加入 ＠Log4j，
這個 Annotation 由 groovy.util.logging package 提供。
在程式碼需要除錯訊息的地方加入 Log 輸出。

> 路徑： src/main/groovy/SimpleClass.groovy

``` groovy
import groovy.util.logging.*

@Log4j
class SimpleClass {
  def log2test {
    log.debug 'message send to logger'
  }
}
```

依照訊息的層級，可以呼叫其他不同方法：

* log.info
* log.debug
* log.warning
* log.error

接下來要定義一個 log4j.properties 設定檔。

> 路徑： src/main/resources/log4j.properties

``` properties
log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.File=stacktrace.log
log4j.appender.file.encoding=utf-8
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n

log4j.rootLogger=debug, file
```

這一組設定會將 Log 寫入 stacktrace.log 記錄檔，
並且採用 UTF-8 編碼（支援中文訊息）。
