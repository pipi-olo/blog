---
title: [Twitter Finatra] Twitter Server
description: >-
  Twitter Server 는 관리자 페이지, 로깅 등 애플리케이션의 공통적인 요소들을 제공합니다.
date: 2024-12-10 22:00:00 +0800
categories: [Backend, Twitter Finatra]
tags: [Twitter, Finatra, Scala]
---
# 개요
Twitter Finatra 는 Twitter Server 와 Finagle 을 사용합니다. Twitter Finatra 를 이해하기 위해서는 Twitter Server 에 대해 알아야합니다.
Twitter Server 는 관리자 페이지, 로깅 등 애플리케이션의 공통적인 요소들을 제공합니다.

다음 명령어를 통해 의존성을 추가합니다.
```text
LibraryDependencies += "com.twitter" %% "twitter-server" % "24.2.0"
```

Twitter Server 를 상속하고 main 메서드를 정의해야 합니다.
```scala
import com.twitter.server.TwitterServer

object MyServer extends TwitterServer {

  def main(): Unit = {
    print("Hello World!")
  }
}
```

# Life Cycle
Twitter Server 는 com.twitter.app.App 을 상속하기 때문에 c.t.a.App 의 Life Cycle 과 매우 유사합니다. \
Twitter Server 는 c.t.a.App Life Cycle 에 WarmUp 단계를 추가합니다.

WarmUp 단계에서 외부 트래픽을 수용하기 전, 서버를 워밍업하는 로직을 추가할 수 있습니다. \
warmupComplete() 를 통해 WarmUp 단계 종료 여부를 알 수 있습니다.


# Flags
서버 설정, 조건에 따른 동작 제어에 Flag 를 사용할 수 있습니다.

## Type Safety
Flag 는 Scala Type Safety 를 보장합니다.
```scala
import com.twitter.app.Flag
import com.twitter.server.TwitterServer

import java.net.InetSocketAddress

object MyServer extends TwitterServer {

  val what: Flag[String] = flag(name = "what", default = "hello", help = "String to return")
  val addr: Flag[InetSocketAddress] = flag(name = "bind", default = new InetSocketAddress(0), help = "Bind address")
  
  def main(): Unit = {
    print("Hello World!")
  }
}
```

## Help
Flag 를 추가하면, 자동으로 help 을 통해 정보를 표시합니다.
```shell
$ java -jar target/myserver-1.0.0-SNAPSHOT.jar -help

MyServer
  -what='hello': String to return
  -bing=':0': Bind address
```

## Phase parsing args
Flag 는 parseArgs 단계에서 값이 정해지기 때문에, \
생성자 혹은 onInit 단계에서 flag 를 정의해야 합니다. 동일하게, Flag 는 preMain 단계부터 읽을 수 있습니다.

<kbd>failfastOnFlagsNotParsed</kbd> 옵션을 키는 것이 좋습니다. \
이 옵션을 키면 Flag 가 파싱되기 전에, Flag 에 접근하면 <kbd>IllegalStateException</kbd> 가 발생합니다.
```scala
override def failfastOnFlagsNotParsed: Boolean = true
```

# Logging
Twitter Server 는 로깅을 위해 [SLF4J](https://www.slf4j.org) 를 사용합니다.
* [Logback](https://logback.qos.ch) 은 Twitter Server 에서 권장하는 SLF4J 구현체입니다.

```text
LibraryDependencies += "ch.qos.logback" % "logback-classic" % "1.4.7"
```

## Log Format
로그 출력 형식을 변경하려면, <kbd>defaultFormatter</kbd> 를 오버라이딩합니다.
```scala
override def defaultFormatter = new Formatter(
  timezone = Some("KST"),
  prefix = "[yyyy-MM-dd HH:mm:ss.SSS] [%.3s] %s: "
)
```

## 로그 레벨 동적 변경
관리자 인터페이스의 로깅 핸들러를 통해 로그 레벨을 동적으로 변경할 수 있습니다. \
이를 위해, SLF4J 로깅 라이브러리에 직접 의존하는 대신, Twitter Server 의 로깅 구현에 의존해야 합니다.

| Implementation          | Dependency                                                                                               |
|-------------------------|----------------------------------------------------------------------------------------------------------|
| java.util.logging (JUL) | [twitter-server/slf4j-jdk14](https://github.com/twitter/twitter-server/tree/develop/slf4j-jdk14)         |
| Log4j                   | [twitter-server/slf4j-log4j12](https://github.com/twitter/twitter-server/tree/develop/slf4j-log4j12)     |
| Logback (recommended)   | [twitter-server/logback-classic](https://github.com/twitter/twitter-server/tree/develop/logback-classic) |


# References
* [Twitter Server GitHub](https://github.com/twitter/twitter-server)
* [Twitter Server Docs](https://twitter.github.io/twitter-server/)
