# LIL-165 섹션 2 — 프로젝트 환경설정 (학습 노트)

> 인프런 "스프링 입문"(김영한, courseId 325630) 섹션 2를 강의 시청 대신 QnA로 학습한 정리.
> 복기(액티브 리콜)용: 각 질문에 스스로 답해본 뒤 아래 답과 대조한다.

---

## 1. 빌드 도구: Gradle vs Maven

### Q. 왜 요즘은 Gradle을 더 쓰나? (Maven과 차이)

**흔한 오답**: "라이브러리 호환성이 좋아서 / 안정성이 높아서" → 아니다.
둘 다 Maven Central 같은 **동일한 저장소**에서 라이브러리를 받아오므로 호환성·안정성은 사실상 같다.

**실제 차이 두 가지**:

1. **빌드 스크립트 작성 방식**
   - Maven: `pom.xml` (XML). 선언적이지만 장황하고 조건문·반복문 같은 로직을 넣기 어렵다.
   - Gradle: `build.gradle`(Groovy DSL) 또는 `build.gradle.kts`(Kotlin DSL). **진짜 프로그래밍 언어**라서 변수·if·for·함수까지 쓸 수 있어 간결하고 유연하다.
   - Groovy(JVM 동적 타입 언어), Kotlin(JVM 정적 타입 언어) 모두 실제 언어. XML은 데이터 구조일 뿐이라 로직이 안 들어간다.

2. **빌드 속도** — Gradle이 다음 세 가지 덕에 빠르다.
   - **incremental build**: 이번 빌드 안에서 바뀐 부분만 다시 빌드.
   - **빌드 캐시**: 예전에 만든 결과물 자체(class, jar)를 재사용. (incremental보다 넓은 개념)
   - **데몬 프로세스**: 아래 참고.

### 데몬 프로세스(Gradle Daemon)

Gradle 자체도 Java로 만들어진 프로그램이라 실행되려면 JVM이 필요하다.
`gradle build`를 칠 때마다 JVM을 새로 띄우면 매번 시작 비용(부팅 + 클래스 로딩)이 든다.
데몬은 이 "Gradle용 JVM"을 **백그라운드에 계속 살려두는 것**. 두 번째 실행부터는 켜진 JVM을 재사용해 빨라진다.

### 빌드 캐시 & 해시

- 각 빌드 작업(task)의 **입력**(소스코드 + 컴파일러 버전 + 설정값)을 해시로 계산해 그 결과물을 캐시(`~/.gradle/caches`)에 저장.
- 다음 빌드 때 입력이 안 바뀌었으면 다시 컴파일하지 않고 캐시에서 꺼낸다.
- **해시 = map의 key 비유**: `hash(입력) → key`, 그 key로 캐시 저장소에서 결과물(class/jar)을 찾는다. 있으면 재사용(빌드 스킵), 없으면 실제 빌드 후 그 key로 저장.
- 해시는 원본을 저장하는 게 아니라 "요약 지문"만 만든다. SHA-256이면 원본이 1KB든 100MB든 **항상 고정된 32바이트**. → "설정을 다 합치면 너무 뚱뚱하지 않냐"는 오해였음.

### Maven과의 대비 정리

- Maven 표준은 매번 새 JVM을 띄운다(비공식 확장 `mvnd` 예외).
- 다운로드 라이브러리 캐시(`~/.m2`)는 Maven에도 있다.
- 하지만 Gradle처럼 "**작업 결과물 자체를 재사용**하는 세밀한 빌드 캐시"는 Maven엔 없다. ← 이게 속도 차이의 핵심.

---

## 2. 전이 의존성 (Transitive Dependency)

### Q. `build.gradle`에 2줄만 썼는데 왜 수십 개가 딸려오나?

내가 선언한 라이브러리(`spring-boot-starter-web`)도 자기 메타데이터에 "나는 tomcat, spring-webmvc, jackson이 필요하다"고 선언해 놨다.
Gradle은 **내가 선언한 것 → 그것이 필요로 하는 것 → 또 그것이 필요로 하는 것...** 을 재귀적으로 끝까지 따라가며 전부 다운로드한다.
→ "빌드하는" 게 아니라 "**필요한 걸 찾아서 받아오는**" 것.

### 실제 확인 (IntelliJ Gradle 탭 → runtimeClasspath)

Dependencies는 4가지 scope로 분류된다:
- `compileClasspath`: 컴파일 시 필요
- `runtimeClasspath`: 실행 시 필요 (컴파일 때와 다를 수 있음)
- `testCompileClasspath` / `testRuntimeClasspath`: 테스트 코드용

`Spring Web` + `Thymeleaf` 2개만 선언 → runtimeClasspath에 실제로 딸려온 것:
```
spring-boot-starter-thymeleaf
spring-boot-starter-webmvc      ← 옛날의 starter-web (버전 올라가며 이름 구체화)
spring-boot-http-converter
spring-boot-starter-jackson     ← JSON 직렬화/역직렬화
spring-boot-starter-tomcat      ← 내장 톰캣 (직접 선언 안 했는데 딸려옴!)
spring-boot-starter (*)         ← (*) = 이미 다른 곳에서 펼쳤으니 중복 생략 표시
spring-boot-webmvc
```
> 버전 노트: 강의는 Spring Boot 2.x라 `spring-boot-starter-web`. 실습은 4.1.0이라 `spring-boot-starter-webmvc`로 이름이 바뀌었다. 개념은 동일.

### Q. 버전이 충돌하면? (같은 라이브러리를 다른 버전이 요구)

**의존성 충돌 해결(conflict resolution)**:
- Gradle 기본: **더 높은 버전이 이긴다**.
- Maven 기본: **선언 위치가 더 가까운(depth 짧은) 것이 이긴다** ← Gradle/Maven 헷갈리는 실무 트랩.

### Q. 버전을 다운그레이드하려면?

- Gradle에선 **직접 명시한 버전이 전이 의존성보다 우선**한다.
  예: `implementation("com.fasterxml.jackson.core:jackson-databind:2.10.0")` 직접 선언 → 그게 이김.
- 더 세밀하게는 `resolutionStrategy.force`, `constraints`. (지금 단계는 "직접 명시하면 이긴다"만 알면 충분)

---

## 3. 내장 톰캣 & 배포 방식의 변화

### 내장 톰캣(Embedded Tomcat)

- `spring-boot-starter-web` → `spring-boot-starter-tomcat`이 전이 의존성으로 딸려온다.
- 이건 "톰캣 서버 프로그램"이 아니라 **톰캣 실행 코드가 담긴 라이브러리(jar)**.
- 앱을 빌드하면 이 톰캣 코드가 내 jar 안에 포함되고, 앱 시작 시(`SpringApplication.run()`) 그 코드로 톰캣 서버를 프로그램적으로 띄운다.
- **딱 그 의존성이 classpath에 있어야만** 일어나는 일. 자동으로 그냥 되는 게 아니다.
- 참고: 리액티브 스타일이면 `spring-boot-starter-webflux`를 쓰고, 그땐 톰캣 대신 Netty가 딸려온다.

### 옛날(WAR) vs 지금(내장 jar)

| | 옛날 방식 | 지금(스프링 부트) |
|---|---|---|
| 서버 | Tomcat을 서버에 **미리 설치** | Java만 있으면 됨 |
| 배포물 | WAR를 정해진 폴더에 넣음 | jar 하나 복사 |
| 실행 | 설치된 Tomcat이 WAR를 읽어 구동 | `java -jar` = 곧 서버 구동 |
| 단계 | "서버 세팅"과 "앱 배포"가 분리된 2단계 | 하나로 합쳐짐 |

> 정정 포인트: "옛날엔 Gradle이 없어서"가 아니다. 빌드 도구(Ant, Maven)는 Gradle 전에도 있었다. 진짜 차이는 **빌드 도구 유무가 아니라 아키텍처(내장 서버) 차이**.

### WAR / JAR

- **JAR (Java ARchive)**: 여러 `.class` + 리소스 + 메타데이터를 압축(사실상 zip)한 파일 하나.
- **WAR (Web Application Archive)**: 형식은 jar와 같은 압축 파일이지만, 외부 서블릿 컨테이너(Tomcat)에 올려 구동하는 용도.

---

## 4. JVM / 컴파일 / jar 의 관계

### 빌드 vs 실행 (헷갈렸던 핵심)

- **빌드** = `javac` 컴파일(`.java` → `.class`) + `jar` 도구가 압축(`.class`들 → `.jar`). 이 전체를 **Gradle/Maven이 지휘**.
- **JVM** = 만들어진 `.class`/`.jar`를 **실행**하는 주체. (빌드가 끝난 다음 역할)
- Gradle 자신도 Java 프로그램이라 실행되려면 JVM이 필요하지만, 그건 "Gradle을 돌리는 JVM"이지 빌드 작업 자체를 JVM이 하는 게 아니다.

### 컴파일 파이프라인

```
.java (소스코드)
   │  javac (컴파일)
   ▼
.class (바이트코드) ← 여러 개
   │  jar 도구가 .class들 + 리소스를 압축 (Gradle의 jar task)
   ▼
.jar (배포용 압축 파일 1개)
```
> jar는 javac가 만드는 게 아니다. javac는 `.class`까지만, 그 뒤 `jar` 도구가 묶는다.

### JVM이 하는 일

- 바이트코드(`.class`)는 특정 OS 기계어가 아니라 "가상 CPU"용 코드.
- JVM(Java Virtual Machine)이 그 가상 CPU 역할을 하는 프로그램. OS/하드웨어 위에서 프로세스로 떠서 바이트코드를 해석·실행.
- 메모리(힙) 관리 + GC(가비지 컬렉션)도 담당.
- 덕분에 같은 `.class`가 JVM만 있으면 어떤 OS에서든 동일하게 동작 = **Write Once, Run Anywhere**.
- 단, JVM 자체는 OS마다 다르게 구현됨(윈도우용/리눅스용 따로). 그 JVM "위"에서 도는 바이트코드가 공통.
- JVM 없으면 → 컴파일은 돼도 실행 주체가 없어 실행 불가.

### JDK / JRE

- **JRE** = JVM + 실행에 필요한 라이브러리. "실행만" 가능.
- **JDK** = JRE + `javac` 등 개발 도구. "개발(컴파일)까지" 가능.

### 증분 컴파일(Incremental Compilation)

- `javac`의 기능이 아니라 **Gradle(빌드 도구) 레벨의 orchestration**.
- Gradle이 "지난 빌드 이후 바뀐 소스 파일"만 골라 javac에 넘긴다. javac는 자기가 증분인지 모른다.
- 데몬 + 증분 컴파일 조합: 코드 수정 후 다시 빌드하면 → JVM은 켜진 채 유지, 바뀐 파일만 컴파일, 나머지는 이전 결과 재사용.

---

## 5. JVM 메모리 구조 & GC (일부는 Phase 5로 이월)

### 메모리 구조

- **Heap**: `new`로 만든 객체 인스턴스 저장. **모든 스레드가 공유**. GC 대상.
- **Stack**: 스레드마다 **하나씩(독립)**. 메서드 호출 1건의 실행 상태(지역변수·매개변수·복귀 주소)를 "스택 프레임"으로 쌓고, 메서드 끝나면 pop.
- **Method Area(메타스페이스)**: 클래스 정보·static 변수·상수 등 **정적 메타데이터**. (인스턴스는 Heap, 그 인스턴스의 "설계도" 정보는 여기)
- **PC Register / Native Method Stack**: → Phase 5 이월.

```
스레드 A                    스레드 B
┌─────────────────┐        ┌─────────────────┐
│ method3() 프레임  │  ←실행중                    
├─────────────────┤        ├─────────────────┤
│ method2() 프레임  │        │ method1() 프레임  │ ←실행중
├─────────────────┤        └─────────────────┘
│ method1() 프레임  │
└─────────────────┘
   (각자 독립된 스택 → 지역변수가 thread-safe한 이유)
```

- 지역변수가 thread-safe = 각 스레드가 자기 스택만 봐서.
- Heap은 프로세스에 하나뿐이라 여러 스레드가 같은 객체를 동시에 건드림 → **race condition** 발생 → 락 필요. (LIL-185에서 직접 재현 예정)

### GC 동작

- **대상**: Heap의 객체 인스턴스만. (Stack 지역변수는 메서드 종료 시 자동 소멸 → GC 대상 아님)
- **기준**: **도달 가능성(reachability)**. "과거에 참조된 적 있음"이 아니라 "**지금 이 순간 루트에서 따라가면 닿는가**".
  - 루트 = Stack 지역변수, static 변수 등.
  - 참조 카운팅이 아니라 도달 가능성 → 순환 참조라도 루트에서 안 닿으면 같이 제거.

```java
Object a = new Car();  // a가 Car 객체를 가리킴 (도달 O)
a = null;               // Car를 가리키는 변수가 세상에 하나도 없음
                        // → 다음 GC 스캔 때 루트에서 안 닿음 → 쓰레기 판정
```
- "매핑"은 고정 상태가 아니라 계속 바뀌는 상태(재할당·스코프 종료로 끊김). GC는 매번 새로 참조 그래프를 훑는다.

### Phase 5(CS 기초)로 이월한 주차장(parking lot)

- Young/Old Generation, Minor/Major(Full) GC, Mark & Sweep, G1/ZGC
- PC Register, Native Method Stack
- Docker, 네임스페이스/cgroup (컨테이너 격리 — OS 레벨 추상화. JVM은 언어/런타임 레벨 추상화라 다른 층)
- 해시 충돌 & 알고리즘 (MD5/SHA-1은 실제 충돌 발견돼 보안용 폐기, SHA-256은 아직 안전. 아발란치 효과. 비둘기집 원리)
- 스택/힙, 프로세스/네트워크/파일시스템 전반

---

## 6. 핸즈온 결과

- `start.spring.io`에서 프로젝트 생성: Gradle-Groovy / Java / Spring Boot 4.1.0 / Jar / Java 17 / deps: Spring Web + Thymeleaf.
- Group `com.kdelay`, Artifact `hello-spring`.
- IntelliJ로 열어 `build.gradle` 확인 → Gradle 탭에서 runtimeClasspath 전이 의존성 트리 실제 확인.
- 메인 클래스 실행 → Tomcat 8080 구동 → `localhost:8080` 접속 → **Whitelabel Error Page (404)**.
  - 이게 **성공 신호**: 스프링/톰캣은 정상 구동, 단지 `/` 경로 핸들러(컨트롤러·index.html)가 아직 없어서 뜬 기본 에러 페이지.

---

## 다음(섹션 3 예고)

스프링 웹 개발 기초 — 정적 컨텐츠 / MVC·템플릿 엔진 / API. 이 Whitelabel 에러를 컨트롤러·웰컴 페이지로 없애보는 단계.
