# 프로젝트 환경설정 — Gradle·전이 의존성·내장 톰캣·jar
> Linear: LIL-193 | Phase 1 | 2026-07-21 | 스프링 입문 섹션 2
> → velog: back-to-basics-02 (톰캣을 몰라도 스프링은 켜졌다)
> ※ 이 파일은 대화 원본 유실 후 진행로그·velog 초안에서 역구성한 것. 이후 세션부터는 실시간 축적.

## Q1. 빌드 도구로 Gradle을 쓰던데, Maven이랑 뭐가 달라?
**A.** 둘 다 의존성 관리 + 빌드 도구. Maven은 XML(`pom.xml`), Gradle은 Groovy/Kotlin DSL(`build.gradle`)로 선언한다. Gradle이 더 간결하고 빌드 캐시·증분 빌드로 빠르다. 요즘 스프링 부트 신규 프로젝트는 대부분 Gradle.

**💡 왜/트레이드오프:** XML은 정형화돼 도구 지원이 강하지만 장황하다. Gradle DSL은 유연하고 짧지만 문법 학습 비용이 있다. 신규는 Gradle, 레거시 유지보수는 Maven을 그대로 쓰는 경우가 많다.

## Q2. build.gradle에 라이브러리 몇 개만 적었는데 왜 수십 개가 딸려와?
**A.** 전이 의존성(transitive dependency). 내가 A를 선언하면 A가 필요로 하는 B·C도 Gradle이 알아서 가져온다. `spring-boot-starter-web` 하나가 톰캣·잭슨·스프링MVC 등을 줄줄이 끌고 온다.

**💡 왜/트레이드오프:** 편하지만 "내가 직접 안 쓴 라이브러리가 왜 있지?"의 원인. 버전 충돌 시 의존성 트리(`gradle dependencies`)를 봐야 한다.

## Q3. 스프링 부트는 톰캣을 따로 안 깔았는데 어떻게 웹서버가 돌아?
**A.** 내장 톰캣(embedded Tomcat). `spring-boot-starter-web`에 톰캣이 라이브러리로 들어있어서 `main()` 실행 시 톰캣을 코드로 띄운다. 그래서 war를 외부 톰캣에 배포하던 옛 방식 없이 jar 하나로 실행된다.

**💡 왜/트레이드오프:** "한 번 빌드하면 어디서든 `java -jar`로 실행"의 이식성. 대신 서버 설정이 애플리케이션에 묶인다.

## Q4. JVM·컴파일·jar가 정확히 무슨 관계야?
**A.** 자바 소스(`.java`) → 컴파일 → 바이트코드(`.class`) → 여러 `.class`와 리소스를 묶은 게 jar. JVM이 이 바이트코드를 읽어 각 OS에 맞게 실행한다. "한 번 작성, 어디서나 실행"이 JVM 덕분.

**💡 왜/트레이드오프:** OS마다 다시 컴파일할 필요 없음(이식성). 대신 JVM이라는 런타임 계층을 항상 거친다(약간의 오버헤드, 대신 JIT로 런타임 최적화).

## 핸즈온
`hello-spring` 프로젝트 생성 → `back-to-basics/phase1-web-spring/`에 커밋 → 실행해서 Whitelabel Error Page 확인.
