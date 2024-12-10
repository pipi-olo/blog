---
title: [Twitter Finatra] Twitter Server
description: >-
  Twitter Server 는 관리자 페이지, 로깅 등 애플리케이션의 공통적인 요소들을 제공합니다.
date: 2024-12-10 22:00:00 KST
categories: [Backend, Twitter Finatra]
tags: [Twitter, Finatra, Scala]
---

# 개요
Twitter Finatra 는 Twitter Server 와 Finagle 을 사용합니다. Twitter Finatra 를 이해하기 위해서는 Twitter Server 에 대해 알아야합니다.
Twitter Server 는 관리자 페이지, 로깅 등 애플리케이션의 공통적인 요소들을 제공합니다.

다음 명령어를 통해 의존성을 추가합니다.
```sbt
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

## LifeCycle
Twitter Server 는 com.twitter.app.App 을 상속하기 때문에 c.t.a.App 의 Life Cycle 과 매우 유사합니다.
Twitter Server 는 c.t.a.App Life Cycle 에 WarmUp 단계를 추가합니다.

WarmUp 단계에서 외부 트래픽을 수용하기 전, 서버를 워밍업하는 로직을 추가할 수 있습니다.
warmupComplete() 를 통해 WarmUp 단계 종료 여부를 알 수 있습니다.

## References
* [Twitter Server GitHub](https://github.com/twitter/twitter-server)
* [Twitter Server Docs](https://twitter.github.io/twitter-server/)
