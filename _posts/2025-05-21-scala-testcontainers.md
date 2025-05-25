---
title: How to use Testcontainers in scala
description: 
author: pipiolo
date: 2025-05-21 20:00:00 +0800
categories: [Scala]
tags: [Scala, Testcontainers]
---

## Overview
[Testcontainers](https://testcontainers.com) 는 단위 테스트에서 실제 서비스 환경에 가까운 테스트를 할 수 있도록, Docker 컨테이너를 자동으로 생성 및 제거해주는 라이브러리입니다.
사용자는 로컬에서 실행된 Docker 컨테이너와의 통신을 통해 실제 서비스 환경과 매우 유사한 테스트를 진행할 수 있습니다.
Java, Go, Python 등 다양한 언어와 Kafka, Cassandra, Postgres 등 다양한 컨테이너를 제공합니다.

| 항목         | Testcontainers           | Embedded Library   |
|------------|--------------------------|--------------------|
| 실제 서비스 유사성 | 동일한 도커 이미지 사용            | Mock 기반 구현         |
| 설정 유연성     | 환경 변수 등 도커 컨테이너 수준 설정 가능 | 제한된 API            |
| 버전 테스트     | 다양한 버전 가능                | 라이브러리가 지원하는 버전만 동작 |
| 리소스        | 도커 컨테이너 생성 및 제거 비용       | 가볍고 빠르다.           |

### testcontainers-scala
[Testcontainers](https://testcontainers.com) 공식 문서에서는 Scala 지원에 대한 내용을 확인할 수 없습니다.
[testcontainers-scala](https://github.com/testcontainers/testcontainers-scala) 는 [개인 오픈소스](https://github.com/dimafeng/testcontainers-scala)로 시작하여 testcontainers 에 포함되었습니다.
testcontainers-scala 는 testcontainers-java 에서 사용하는 각 도커 컨테이너를 사용하기 위한 Wrapper 라이브러리입니다. testcontainers-scala 에서 사용할 수 있는 컨테이너 모듈은 [testcontainers-scala-modules](https://github.com/testcontainers/testcontainers-scala/tree/master/modules) 에서 확인할 수 있습니다.

## How to use
### Set Up
Scala 에서 Cassandra TestContainers 를 사용하기 위해 다음과 같이 build.sbt 파일을 작성합니다.
```sbt
lazy val versions = new {
  val cassandra     = "4.19.0"
  val logback       = "1.5.18"
  val scalaTest     = "3.2.19"
  val testContainer = "0.43.0"
}

lazy val root = (project in file("."))
  .configs(IntegrationTest)
  .settings(Defaults.itSettings)
  .settings(
    libraryDependencies ++= Seq(
      "ch.qos.logback"       % "logback-classic"                % versions.logback,
      "org.apache.cassandra" % "java-driver-core"               % versions.cassandra,
      "org.scalatest"       %% "scalatest"                      % versions.scalaTest     % IntegrationTest,
      "com.dimafeng"        %% "testcontainers-scala-scalatest" % versions.testContainer % IntegrationTest,
      "com.dimafeng"        %% "testcontainers-scala-cassandra" % versions.testContainer % IntegrationTest
    ),
    IntegrationTest / fork := true
  )
```
* slf4j 로깅 구현체 라이브러리를 추가하지 않으면, testcontainers 에서 발생하는 로그를 확인할 수 없습니다.
  ```
  SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
  SLF4J: Defaulting to no-operation (NOP) logger implementation
  SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
  SLF4J: Failed to load class "org.slf4j.impl.StaticMDCBinder".
  SLF4J: Defaulting to no-operation MDCAdapter implementation.
  SLF4J: See http://www.slf4j.org/codes.html#no_static_mdc_binder for further details.
  ```
* testcontainers 는 단위 테스트 이지만, 별도의 환경 및 구성이 필요하기 때문에 통합 테스트로 하는 것이 좋습니다.
  * `src/it/scala` 에 통합 테스트 코드를 작성합니다.
  * `sbt IntegrationTest/test` 명령어를 통해 테스트를 실행합니다.
* `IntegrationTest / fork := true` 옵션을 통해 SBT 와 별도의 JVM 에서 테스트를 진행합니다. 이를 통해 컨테이너의 종료를 보장합니다.

### Test
```scala
class CassandraRepositoryTest extends AnyFlatSpec with Matchers with TestContainerForAll {
  override val containerDef: CassandraContainer.Def = CassandraContainer.Def(dockerImageName = DockerImageName.parse("cassandra:5.0.3"))

  override def afterContainersStart(containers: CassandraContainer): Unit = {
    super.afterContainersStart(containers)

    ...
  }

  override def beforeContainersStop(containers: CassandraContainer): Unit = {
    super.beforeContainersStop(containers)

    ...
  }
}
```
* `TestContainerForAll` 을 상속하고, `containerDef` 을 오버리이딩 합니다.
  * `containerDef` 는 컨테이너 빌드 방법을 정의합니다. `container` 는 실행된 컨테이너를 의미합니다.
  * `ForAllTestContainer` 은 Deprecated 되었습니다. (v0.34.0)
  * `TestContainerForAll`, `TestContainerForEach`, `TestContainersForAll`, `TestContainersForEach` 총 4가지 옵션이 존재합니다.
* `afterContainersStart`, `beforeContainersStop` 을 통해서 각 컨테이너 시작 후, 컨테아너 종료 전 작업을 정의할 수 있습니다. 
* `com.dimafeng.testcontainers` 라이브러리를 사용해야 합니다.

```scala
import com.datastax.oss.driver.api.core.CqlSession
import com.dimafeng.testcontainers.CassandraContainer
import com.dimafeng.testcontainers.scalatest.TestContainerForAll
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.should.Matchers
import org.testcontainers.utility.DockerImageName

class CassandraRepositoryTest extends AnyFlatSpec with Matchers with TestContainerForAll {
  override val containerDef: CassandraContainer.Def = CassandraContainer.Def(dockerImageName = DockerImageName.parse("cassandra:5.0.3"))

  ...

  it should "데이터를 저장 및 조회할 수 있다." in withContainers { cassandraContainer: CassandraContainer =>
    val session = CqlSession
      .builder()
      .addContactPoint(cassandraContainer.cassandraContainer.getContactPoint)
      .withLocalDatacenter(cassandraContainer.cassandraContainer.getLocalDatacenter)
      .withAuthCredentials(cassandraContainer.cassandraContainer.getUsername, cassandraContainer.cassandraContainer.getPassword)
      .build()
    
    ...
  }
}
```
* 각 테스트에 `withContainers` 를 통해서 실행된 컨테이너를 얻을 수 있습니다.

## One More Thing
### Multi Containers Test

```sbt
lazy val versions = new {
  val cassandra     = "4.19.0"
  val postgresql    = "42.5.1"
  val logback       = "1.5.18"
  val scalaTest     = "3.2.19"
  val testContainer = "0.43.0"
}

lazy val root = (project in file("."))
  .configs(IntegrationTest)
  .settings(Defaults.itSettings)
  .settings(
    libraryDependencies ++= Seq(
      "ch.qos.logback"       % "logback-classic"                 % versions.logback,
      "org.apache.cassandra" % "java-driver-core"                % versions.cassandra,
      "org.postgresql"       % "postgresql"                      % versions.postgresql,
      "org.scalatest"       %% "scalatest"                       % versions.scalaTest     % IntegrationTest,
      "com.dimafeng"        %% "testcontainers-scala-scalatest"  % versions.testContainer % IntegrationTest,
      "com.dimafeng"        %% "testcontainers-scala-cassandra"  % versions.testContainer % IntegrationTest,
      "com.dimafeng"        %% "testcontainers-scala-postgresql" % versions.testContainer % IntegrationTest
    ),
    IntegrationTest / fork := true
  )

```
* build.sbt 에 `postgresql` 관련 라이브러리를 추가합니다.

```scala
import com.dimafeng.testcontainers.lifecycle.and
import com.dimafeng.testcontainers.{CassandraContainer, PostgreSQLContainer}
import com.dimafeng.testcontainers.scalatest.TestContainersForAll
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.should.Matchers
import org.testcontainers.utility.DockerImageName

class ServiceTest extends AnyFlatSpec with Matchers with TestContainersForAll {

  override type Containers = CassandraContainer and PostgreSQLContainer

  override def startContainers(): Containers = {
    val cassandraContainer = CassandraContainer.Def(dockerImageName = DockerImageName.parse("cassandra:5.0.3")).start()
    val postgresContainer  = PostgreSQLContainer.Def(dockerImageName = DockerImageName.parse("postgres:9.6.12")).start()

    cassandraContainer and postgresContainer
  }

  it should "데이터를 저장 및 조회할 수 있다." in withContainers { case cassandraContainer and postgresContainer =>

    ...
  }
}
```
* `Containers` 타입에 사용할 도커 컨테이너 목록을 정의합니다.
* `startContainers` 는 도커 컨테이너 목록의 실행 방법을 정의합니다. 각 컨테이너의 시작 후 로직을 정의할 수 있습니다.
  * `ContainerDef.start()` 는 실행된 컨테이너를 획득합니다.
* `withContainers` 를 통해서 실행된 컨테이너를 통해 테스트를 진행합니다.

## References
* [Testcontainers-scala Github](https://github.com/testcontainers/testcontainers-scala)
