---
title: Introduce to Quarkus with Kotlin
description: 
author: pipiolo
date: 2026-01-01 20:00:00 +0800
categories: [Backend, Quarkus]
tags: [Quarkus, Kotlin]
---

## Overview
[Quarkus](https://quarkus.io) 는 Java 및 Kotlin 애플리케이션을 위한 Supersonic Subatomic 프레임워크로, 클라우드 네이티브 환경에 최적화되어 있습니다.
빠른 시작 시간(Supersonic) 과 낮은 메모리 사용량(Subatomic) 을 제공하여 서버리스 및 컨테이너화된 환경에서 뛰어난 성능을 발휘합니다. Quarkus는 개발자 생산성을 높이기 위해 라이브 코딩, 핫 리로드, 통합 개발 도구 등을 지원합니다.

### Spring 비교
Java / Kotlin 기반의 대표적인 프레임워크인 Spring Boot 와 Quarkus 를 비교하면 다음과 같습니다.

| 항목        | Quarkus                  | Spring Boot       |
|-----------|--------------------------|-------------------|
| 시작 시간     | 매우 빠름 (몇 밀리초)            | 상대적으로 느림 (수 초 이상) |
| 메모리 사용량   | 낮음 (네이티브 이미지 사용 시 더욱 낮음) | 상대적으로 높음          |
| 개발 생산성    | 라이브 코딩, 핫 리로드 지원         | 다양한 개발 도구 지원      |
| 클라우드 네이티브 | 컨테이너화 및 서버리스에 최적화        | 클라우드 네이티브 지원      | 
| 확장성       | 수평 확장에 최적화               | 수평 및 수직 확장 지원     |
| 커뮤니티 지원   | 성장 중                     | 매우 활발함            |

## CRUD Application Example
Quarkus 와 Kotlin 을 사용하여 간단한 Post Service 를 만드는 예제를 살펴봅니다. 이 예제에서는 RESTful API 를 통해 게시물을 생성, 조회, 수정, 삭제하는 기능을 구현합니다.

### Set Up
Quarkus 프로젝트를 생성하기 위해 [Quarkus Initializer](https://code.quarkus.io/) 혹은 IntelliJ IDEA 를 사용하여 프로젝트를 생성할 수 있습니다.
Kotlin 언어와 필요한 확장 기능(예: RESTEasy, Hibernate ORM 등)을 선택하여 프로젝트를 생성합니다. 다음은 IntelliJ IDEA 를 사용하여 Quarkus 프로젝트를 생성하는 방법입니다.

![img.png](/assets/img/posts/backend/qurkus/introduce-to-quarkus-kotlin/intellij-quarkus-new-project.png)
* Java 는 최소 17 버전 이상이 필요합니다.

![img.png](/assets/img/posts/backend/qurkus/introduce-to-quarkus-kotlin/intellij-quarkus-new-project-library.png)
* **REST**, **REST Jackson** 은 REST API 서버 구현을 위해 필요합니다.
  * **REST** 는 JAX-RS(Java API for RESTful Web Services) 표준을 구현한 Quarkus 확장 기능입니다. RESTful 웹 서비스를 쉽게 개발할 수 있도록 도와줍니다.
  * **REST Jackson** 은 객체를 JSON 데이터로 변환(직렬화) 혹은 JSON 을 객체로 변환(역직렬화) 해주는 Quarkus 확장 기능입니다. REST API 에서 JSON 형식의 요청과 응답을 처리할 수 있게 해줍니다.
* **Hibernate ORM with Panache and Kotlin** 는 JPA(Jakarta Persistence API) 기반의 데이터베이스 연동을 위해 필요합니다.
  * **[Hibernate ORM](https://hibernate.org/orm/)** 은 자바 기반의 ORM(Object-Relational Mapping) 프레임워크입니다. 자바의 객체 지향 모델과 관계형 데이터베이스(RDB)의 데이터 모델 사이의 불일치를 해결하여, 개발자가 SQL 쿼리를 직접 작성하는 대신 자바 객체를 통해 데이터를 다룰 수 있게 해줍니다. Hibernate 는 자바 표준 명세인 JPA 의 가장 대표적인 구현체로 널리 사용되고 있습니다.
  * **Hibernate ORM with Panache** 는 Hibernate ORM 을 더욱 쉽게 사용할 수 있도록 도와주는 Quarkus 확장 기능입니다. Panache 는 '품격' 또는 '당당한 태도' 라는 뜻으로, 복잡한 설정 없이도 데이터베이스 작업을 수행할 수 있게 해줍니다.
  * **Hibernate ORM with Panache and Kotlin** 은 널 안정성(null-safety) 과 확장 함수(extension function) 등 Kotlin 언어의 특징을 활용할 수 있도록 지원합니다.
* **JDBC Driver** 는 사용할 데이터베이스에 따라 선택합니다. (예: H2, PostgreSQL 등)
* **SmallRye OpenAPI** 는 OpenAPI 스펙을 기반으로 API 문서를 자동 생성해주는 확장 기능입니다.

### Main Function
Quarkus Application 에서 `main` 함수는 선택 사항입니다. 개발자가 ``main`` 함수를 작성하지 않아도 애플리케이션을 실행할 수 있습니다.
Quarkus 는 내부적으로 `Quarkus.run()` 메서드를 호출하여 애플리케이션을 시작합니다. 다음과 같이 `main` 함수를 작성하여 애플리케이션의 진입점을 명시적으로 정의할 수도 있습니다.

```kotlin
import io.quarkus.runtime.Quarkus
import io.quarkus.runtime.annotations.QuarkusMain

@QuarkusMain
class MyMain {
    companion object {
        @JvmStatic
        fun main(args: Array<String>) {
            println("--- Quarkus가 시작됩니다! ---")
            Quarkus.run(*args) // Quarkus 엔진 실행
        }
    }
}
```

### Define Entity
게시물(Post) 엔티티를 정의합니다.
```kotlin
import io.quarkus.hibernate.orm.panache.kotlin.PanacheCompanion
import io.quarkus.hibernate.orm.panache.kotlin.PanacheEntity
import jakarta.persistence.Entity

@Entity
class Post: PanacheEntity() {
    companion object : PanacheCompanion<Post>

    lateinit var title: String
    lateinit var content: String
}
```
* `@Entity` 어노테이션을 사용하여 JPA 엔티티임을 표시합니다.
  * Hibernate ORM 은 이 어노테이션이 붙은 클래스를 데이터베이스 테이블과 매핑합니다.
  * Quarkus 개발 모드에서 자동으로 데이블을 생성합니다.
  * `jakarta.persistence` 는 자바 진영의 표준 데이터베이스 연동 규격(JPA) 입니다.
* `PanacheEntity` 상속 및 `PanacheCompanion` 동반 객체를 통해 기본적인 CRUD 기능을 사용할 수 있습니다.
  * 기본 키 필드(`id: Long`) 가 자동으로 생성됩니다.
    * 별도로 기본 키 필드를 관리하고 싶다면, `PanacheEntityBase` 를 상속받아 직접 필드를 정의할 수 있습니다.
  * 별도 Repository 클래스를 작성하지 않아도 `Post.findById(id)`, `Post.deleteById(id)` 등 데이터 저장, 조회, 수정, 삭제 기능을 사용할 수 있습니다.
* `lateinit` 키워드를 사용하여 나중에 초기화할 필드를 정의합니다.

### Define Resource
RESTful API 를 처리하는 Resource 클래스를 정의합니다. Spring 의 Controller 와 유사한 역할을 합니다.

```kotlin
import jakarta.transaction.Transactional
import jakarta.ws.rs.Consumes
import jakarta.ws.rs.DELETE
import jakarta.ws.rs.DefaultValue
import jakarta.ws.rs.GET
import jakarta.ws.rs.POST
import jakarta.ws.rs.Path
import jakarta.ws.rs.Produces
import jakarta.ws.rs.QueryParam
import jakarta.ws.rs.core.MediaType
import jakarta.ws.rs.core.Response

@Path("/posts")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
class PostResource {

    @GET
    @Path("/{id}")
    fun get(id: Long): Response {
        val post = Post.findById(id) ?: return Response.status(Response.Status.NOT_FOUND).build()
        return Response.ok(post).build()
    }

    @POST
    @Transactional
    fun create(post: Post): Response {
        post.persist()
        return Response.status(Response.Status.CREATED).entity(post).build()
    }

    @DELETE
    @Path("/{id}")
    @Transactional
    fun delete(id: Long): Response {
        val post = Post.findById(id) ?: return Response.status(Response.Status.NOT_FOUND).build()
        post.delete()
        return Response.status(Response.Status.NO_CONTENT).build()
    }
}
```
* `@Path` 어노테이션을 사용하여 리소스의 기본 경로를 지정합니다.
* `@Produces` 와 `@Consumes` 어노테이션을 사용하여 JSON 형식의 요청과 응답을 처리하도록 설정합니다.
* 각 메서드에 HTTP 메서드 어노테이션(`@GET`, `@POST`, `@DELETE`) 을 사용하여 해당 메서드가 처리할 HTTP 요청 유형을 지정합니다.
* `@Transactional` 어노테이션을 사용하여 데이터베이스 트랜잭션을 관리합니다. 이 어노테이션이 붙은 메서드는 트랜잭션 내에서 실행됩니다.

### Paging
게시물 목록을 페이징 처리하여 조회하는 기능을 추가합니다.

```kotlin
import io.quarkus.panache.common.Page

...

@Path("/posts")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
class PostResource {

  @GET
  fun listAll(
    @QueryParam("index") @DefaultValue("0") index: Int,
    @QueryParam("size") @DefaultValue("20") size: Int
  ): List<Post> {
    return Post.findAll().page(Page.of(index, size)).list()
  }

  ...
}
```
* `@QueryParam` 어노테이션을 사용하여 쿼리 파라미터를 메서드 매개변수로 매핑합니다.
* `@DefaultValue` 어노테이션을 사용하여 쿼리 파라미터의 기본값을 지정합니다.
* `Post.findAll().page(Page.of(index, size)).list()` 를 사용하여 페이징된 게시물 목록을 조회합니다.

## Run Dev Mode
Quarkus 애플리케이션을 개발 모드로 실행하려면, 프로젝트 루트 디렉토리에서 다음 명령어를 실행합니다.

```bash
./gradlew quarkusDev
```
* JAVA_HOME 환경 변수가 올바르게 설정되어 있어야 합니다. (최소 Java 17 이상)
* `8080` 포트에서 애플리케이션이 실행됩니다.
* `quarkus-jdbc-mysql` 등 데이터베이스 드라이버 라이브러리를 포함하고, `application.properties` 파일에 데이터베이스 연결 설정이 없다면, 자동으로 데이터베이스 도커 컨테이너를 실행합니다. 이를 **Dev Services** 라고 합니다.
  * 단, 도커가 설치되어 있고 도커 데몬이 실행 중이어야 합니다.

다양한 개발자 도구를 활용하여 개발 생산성을 높일 수 있습니다.

| URL                                       | Document                 |
|-------------------------------------------|--------------------------|
| http://localhost:8080/q/arc               | CDI Overview             |
| http://localhost:8080/q/arc/beans         | Active CDI Beans         |
| http://localhost:8080/q/arc/observers     | Active CDI Observers     |
| http://localhost:8080/q/arc/removed-beans | Removed CDI Beans        |
| http://localhost:8080/q/dev-ui            | Dev UI                   |
| http://localhost:8080/q/openapi           | Open API Schema document |
| http://localhost:8080/q/swagger-ui        | Open API UI (Swagger UI) |

### Dev UI
Quarkus 는 개발자 생산성을 높이기 위해 Dev UI 를 제공합니다. Dev UI 는 애플리케이션의 상태, 설정, 확장 기능 등을 시각적으로 확인하고 관리할 수 있는 웹 인터페이스입니다.

![img.png](/assets/img/posts/backend/qurkus/introduce-to-quarkus-kotlin/quarkus-dev-ui.png)
![img.png](/assets/img/posts/backend/qurkus/introduce-to-quarkus-kotlin/quarkus-dev-ui-dev-services.png)
* Dev Service 에서 실행 중인 MySQL 도커 컨테이너 정보를 확인할 수 있습니다.

### Swagger UI
`quarkus-smallrye-openapi` 라이브러리를 포함하면, OpenAPI 스펙을 기반으로 API 문서를 자동 생성할 수 있습니다. Swagger UI 를 통해 API 문서를 시각적으로 확인하고 테스트할 수 있습니다.

![img.png](/assets/img/posts/backend/qurkus/introduce-to-quarkus-kotlin/quarkus-swagger-ui.png)
* Swagger UI 를 통해 API 엔드포인트를 테스트할 수 있습니다.

## References
* [Quarkus GitHub](https://github.com/quarkusio/quarkus)
* [Blog Code](https://github.com/pipi-olo/blog-code/tree/main/backend/quarkus/introduce-to-quarkus-kotlin)
