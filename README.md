# Spring REST Docs API 명세 통합

[![oss lifecycle](https://img.shields.io/badge/oss_lifecycle-maintenance-yellow.svg)](https://github.com/ePages-de/restdocs-api-spec/issues/204)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=ePages-de_restdocs-api-spec&metric=coverage)](https://sonarcloud.io/summary/new_code?id=ePages-de_restdocs-api-spec)
![](https://img.shields.io/github/license/ePages-de/restdocs-openapi.svg)
[![Maven Central](https://img.shields.io/maven-central/v/com.epages/restdocs-api-spec)](https://search.maven.org/artifact/com.epages/restdocs-api-spec)
[![Gitter](https://img.shields.io/gitter/room/nwjs/nw.js.svg)](https://gitter.im/restdocs-api-spec/Lobby)

이 프로젝트는 [Spring REST Docs](https://projects.spring.io/spring-restdocs/)의 출력 포맷에 API 명세를 추가하는 확장입니다.  
현재 지원하는 포맷은 다음과 같습니다:
- [OpenAPI 2.0](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md) (json, yaml)
- [OpenAPI 3.0.1](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md) (json, yaml)
- [Postman Collections 2.1.0](https://schema.getpostman.com/json/collection/v2.1.0/docs/index.html)

> ⚠️ 이 확장은 JSON 기반 API용으로 개발되었습니다.  
> JSON이 아닌 요청/응답 본문에 대해서는 사용 가능한 API 명세가 생성되지 않을 수 있습니다.

## 동기

[Spring REST Docs](https://projects.spring.io/spring-restdocs/)는 RESTful 서비스의 정확하고 읽기 쉬운 문서를 만들어주는 훌륭한 도구입니다.

우리는 특히 테스트 주도 방식이 마음에 들어서 이 도구를 선택했습니다.

AsciiDoc과 Markdown을 지원하여 HTML 기반 문서를 쉽게 생성할 수 있습니다.  
하지만 이들은 마크업 언어이기 때문에 정적으로 생성된 HTML 이상의 것을 얻기 어렵습니다.

OpenAPI 같은 API 명세는 훨씬 더 유연합니다.  
예를 들어 OpenAPI를 사용하면 API를 기계가 읽을 수 있는 형태로 기술할 수 있으며,  
이를 바탕으로 한 다양한 도구(예: [ReDoc](https://github.com/Rebilly/ReDoc), [stoplight.io](https://stoplight.io), [readme.io](https://readme.io))가 존재합니다.

또한 OpenAPI와 같은 명세는 [Postman](https://www.getpostman.com), [Paw](https://paw.cloud)와 같은 많은 REST 클라이언트에서도 지원됩니다.  
따라서 REST API에 대한 명세를 갖추는 것은 큰 이점입니다.

대부분의 경우 OpenAPI 명세는 코드 인트로스펙션과 주석(annotation) 기반으로 생성됩니다.  
하지만 우리는 프로덕션 코드에 문서 관련 주석을 추가하여 코드를 더 복잡하게 만드는 방식을 선호하지 않습니다.  
테스트 기반 문서화가 더 정확하다고 생각하며, 그것이 바로 이 프로젝트를 만든 이유입니다.

## 목차

- [동기](#동기)
- [시작하기](#시작하기)
    - [버전 호환성](#버전-호환성)
    - [프로젝트 구조](#프로젝트-구조)
    - [빌드 설정](#빌드-설정)
        - [Gradle](#gradle)
        - [Maven](#maven)
    - [Spring REST Docs와의 사용법](#spring-rest-docs와의-사용법)
    - [Bean Validation 제약 문서화](#bean-validation-제약-문서화)
    - [기존 Spring REST Docs 테스트 마이그레이션](#기존-spring-rest-docs-테스트-마이그레이션)
    - [OpenAPI의 보안 정의](#openapi의-보안-정의)
    - [Gradle 플러그인 실행](#gradle-플러그인-실행)
    - [Gradle 플러그인 설정](#gradle-플러그인-설정)
- [OpenAPI로부터 HTML API 레퍼런스 생성](#openapi로부터-html-api-레퍼런스-생성)

## 시작하기

### 버전 호환성

Spring Boot 및 Spring REST Docs 3.0.0에서는 [RequestParameterSnippet이 QueryParameterSnippet과 FormParameterSnippet으로 분리](https://spring.io/blog/2022/11/24/spring-rest-docs-3-0-0-released)되는 등 변경점이 있습니다.

|Spring Boot 버전 | restdocs-api-spec 버전|
|---|---|
|3.x|0.17.1 이상|
|2.x|0.16.4|

### 프로젝트 구조

- **restdocs-api-spec** - 실제 Spring REST Docs 확장 기능을 포함합니다.  
  테스트에서 사용하는 [ResourceDocumentation](restdocs-api-spec/src/main/kotlin/com/epages/restdocs/apispec/ResourceDocumentation.kt)이 진입점입니다.
- **restdocs-api-spec-mockmvc** - MockMvc 기반 테스트에서 쉽게 마이그레이션할 수 있게 도와주는 래퍼입니다.
- **restdocs-api-spec-restassured** - Rest Assured 기반 테스트에서 쉽게 마이그레이션할 수 있게 도와주는 래퍼입니다.
- **restdocs-api-spec-gradle-plugin** - `resource.json` 파일을 API 명세 파일로 집계해주는 Gradle 플러그인입니다.

### 빌드 설정

#### Gradle

1. 플러그인 추가
    - plugins DSL 사용:
        ```groovy
        plugins {
            id 'com.epages.restdocs-api-spec' version '0.18.2'
        }
        ```
    - 레거시 플러그인 적용:
        ```groovy
        buildscript {
          repositories {
            maven {
              url "https://plugins.gradle.org/m2/"
            }
          }
          dependencies {
            classpath "com.epages:restdocs-api-spec-gradle-plugin:0.18.2"
          }
        }
        apply plugin: 'com.epages.restdocs-api-spec'
        ```
2. 테스트에 필요한 의존성 추가
    ```groovy
    repositories {
        mavenCentral()
    }

    dependencies {
        testImplementation('com.epages:restdocs-api-spec-mockmvc:0.18.2')
    }

    openapi {
        host = 'localhost:8080'
        basePath = '/api'
        title = 'My API'
        description = 'My API description'
        tagDescriptionsPropertiesFile = 'src/docs/tag-descriptions.yaml'
        version = '1.0.0'
        format = 'json'
    }

    openapi3 {
        server = 'https://localhost:8080'
        title = 'My API'
        description = 'My API description'
        tagDescriptionsPropertiesFile = 'src/docs/tag-descriptions.yaml'
        version = '0.1.0'
        format = 'yaml'
    }

    postman {
        title = 'My API'
        version = '0.1.0'
        baseUrl = 'https://localhost:8080'
    }
    ```
> 샘플 프로젝트의 [build.gradle](samples/restdocs-api-spec-sample/build.gradle) 참고

#### Maven

루트 프로젝트에서는 별도의 Maven 플러그인을 제공하지 않습니다.  
[Maven 플러그인 예시](https://github.com/BerkleyTechnologyServices/restdocs-spec)를 참고하세요.

### Spring REST Docs와의 사용법

테스트 코드에서 [ResourceDocumentation](restdocs-api-spec/src/main/kotlin/com/epages/restdocs/apispec/ResourceDocumentation.kt)의 `resource` 메서드를 사용합니다.

가장 단순한 형태는 다음과 같습니다:

```java
mockMvc
  .perform(post("/carts"))
  .andDo(document("carts-create", resource("Create a cart")));
```

이 테스트는 스니펫 디렉토리에 `resource.json` 파일을 생성합니다.  
이 파일에는 리소스에 대한 모든 정보가 담깁니다.

```json
{
  "operationId" : "carts-create",
  "summary" : "Create a cart",
  "description" : "Create a cart",
  ...
}
```

Spring REST Docs와 마찬가지로 요청 필드, 응답 필드, 경로 변수, 파라미터, 헤더, 링크 등도 문서화할 수 있습니다.  
또한 리소스에 대한 설명과 요약도 추가할 수 있습니다.  
확장 기능은 `Authorization` 헤더의 JWT 토큰을 분석해 필요한 스코프도 문서화할 수 있습니다. 기본 인증 헤더도 자동으로 인식하여 문서화합니다.

자세한 예시는 [CartIntegrationTest](samples/restdocs-api-spec-sample/src/test/java/com/epages/restdocs/apispec/sample/CartIntegrationTest.java)를 참고하세요.

> **주의: 경로 변수는 템플릿 URI로 작성해야 합니다.**
> 
> 예시:
> ```java
> mockMvc.perform(get("/carts/{id}", cartId)
> ```

### Bean Validation 제약 문서화

Spring REST Docs처럼 [bean validation constraints](https://docs.spring.io/spring-restdocs/docs/current/reference/html5/#documenting-your-api-constraints)를 활용하여 필드의 제약 조건을 문서화할 수 있습니다.  
`restdocs-api-spec`의 [ConstrainedFields](restdocs-api-spec/src/main/kotlin/com/epages/restdocs/apispec/ConstrainedFields.kt)를 사용하세요.

지원되는 제약 조건 예시:
- `NotNull`, `NotEmpty`, `NotBlank`: 필수 필드로 처리
- `Length`: minLength, maxLength 값 반영
- `Pattern`: 정규식 패턴 반영
- `Min`, `Max`, `Size`: 숫자 필드의 최소/최대값 반영

직접 구현한 ConstrainedFields가 있다면, `FieldDescriptor`의 attributes map에 `validationConstraints` 키로 제약 정보를 추가하면 됩니다.

### 기존 Spring REST Docs 테스트 마이그레이션

#### MockMvc 기반 테스트

`MockMvcRestDocumentationWrapper.document`를 사용하면 기존 Spring REST Docs의 `MockMvcRestDocumentation.document`를 대체할 수 있습니다.

예시:
```java
resultActions
  .andDo(
    MockMvcRestDocumentationWrapper.document(operationName,
      requestFields(...),
      responseFields(...),
      links(...)
  )
);
```

#### REST Assured 기반 테스트

`RestAssuredRestDocumentationWrapper.document`를 사용하세요.  
예시:
```java
RestAssured.given(this.spec)
        .filter(RestAssuredRestDocumentationWrapper.document("{method-name}",
                "API 설명",
                requestParameters(...),
                responseFields(...)
        ))
        .when()
        .queryParam("param", "foo")
        .get("/restAssuredExample")
        .then()
        .statusCode(200);
```

#### WebTestClient 기반 테스트

`WebTestClientRestDocumentationWrapper.document`를 사용하세요.  
예시:
```
webTestClient.get().uri("/sample/{id}?queryParam=something", "1024").exchange()
    .expectStatus().isOk().expectBody()
    .consumeWith(
        WebTestClientRestDocumentationWrapper
            .document("sample",
                ...
            )
    );
```

### OpenAPI의 보안 정의

이 프로젝트는 OAuth2(JWT)와 HTTP Basic Auth에 대한 기본적인 보안 명세를 지원합니다.  
`Authorization` 헤더의 JWT 토큰 또는 기본 인증 헤더를 분석해 필요한 스코프 정보를 추출하여 문서화합니다.

`restdocs-api-spec-gradle-plugin`의 `oauth2SecuritySchemeDefinition` 옵션을 사용하면 OpenAPI의 보안 정의가 생성됩니다.

### Gradle 플러그인 실행

API 명세 파일을 생성하려면 다음과 같이 Gradle 태스크를 실행하세요.

- **OpenAPI 2.0**
    ```
    ./gradlew openapi
    ```
- **OpenAPI 3.0.1**
    ```
    ./gradlew openapi3
    ```
    결과물: `build/api-spec/openapi3.yaml`

- **Postman**
    ```
    ./gradlew postman
    ```
    결과물: `build/api-spec/postman-collection.json`

### Gradle 플러그인 설정

#### 공통 설정

|이름|설명|기본값|
|----|---|---|
|separatePublicApi|비공개 리소스를 제외한 별도의 명세 파일도 생성할지 여부|false|
|outputDirectory|API 명세 파일 출력 디렉토리|build/api-spec|
|snippetsDirectory|Spring REST Docs 스니펫 디렉토리|build/generated-snippets|

#### OpenAPI 공통 설정

|이름|설명|기본값|
|----|---|---|
|title|애플리케이션 제목|API documentation|
|description|애플리케이션 설명|빈값|
|version|API 버전|프로젝트 버전|
|format|json 또는 yaml|json|
|tagDescriptionsPropertiesFile|태그 설명 매핑 yaml 파일|없음|
|oauth2SecuritySchemeDefinition|보안 정의 설정 클로저|없음|

#### OpenAPI 2.0 전용 설정

|이름|설명|기본값|
|----|---|---|
|host|API 서버 호스트|없음|
|basePath|API 기본 경로|없음|
|schemes|지원 프로토콜|없음|
|outputFileNamePrefix|출력 파일 접두사|openapi|

#### OpenAPI 3.0.1 전용 설정

|이름|설명|기본값|
|----|---|---|
|outputFileNamePrefix|출력 파일 접두사|openapi3|
|servers|여러 서버 정의|http://localhost|
|server|단일 서버 정의|http://localhost|

#### Postman 전용 설정

|이름|설명|기본값|
|----|---|---|
|title|컬렉션 이름|API documentation|
|version|컬렉션 버전|프로젝트 버전 또는 1.0.0|
|baseUrl|기본 URL|http://localhost|

## OpenAPI로부터 HTML API 레퍼런스 생성

[redoc](https://github.com/Rebilly/ReDoc)으로 OpenAPI 명세를 HTML로 변환할 수 있습니다.

```
# redoc-cli 설치
npm install -g redoc-cli

# HTML 파일로 번들링
redoc-cli bundle build/api-spec/openapi.json

# 서버에서 문서 확인
redoc-cli serve build/api-spec/openapi.json
```

## 유지보수

### 프로젝트 배포

이 프로젝트는 [GitHub Actions](./.github/workflows)로 배포됩니다.  
버전 관리는 Git 태그로 합니다([allegro/axion-release-plugin](https://axion-release-plugin.readthedocs.io) 참고).  
Java 라이브러리는 Sonatype에, Gradle 플러그인은 Gradle Plugin Portal에 배포됩니다.

배포 방법 요약:
1. GitHub UI에서 릴리즈 생성(태그 예: 0.18.2)
2. Sonatype 로그인, Staging Repository 이동
3. 생성된 Staging Repository를 닫고 오류 확인
4. Staging Repository를 Release 처리
5. README의 버전 정보 업데이트 후 커밋
