# java-goof (fork of snyk-labs/java-goof) — expected upgrade-impact verdicts

Scan branch `main` @ d26fdbf. Maven multi-module: `todolist-goof/{todolist-core,todolist-web-common,todolist-web-struts}` and
`log4shell-goof/{log4shell-client,log4shell-server}`. Versions live in `<properties>` of `todolist-goof/pom.xml`
(`spring.version` 3.2.6.RELEASE, `hibernate.version` 4.3.7.Final, `struts2.version` 2.3.20) — the apply stage must edit
the property, not the dependency line. Maven has no resolver check yet, so verdicts rest on japicmp + grep + notes.

| package | installed | fix | expected | what to look for |
|---|---|---|---|---|
| log4j-core | 2.7 (web-struts) | 2.25.4 | **SAFE**, medium | code imports `org.apache.logging.log4j.{LogManager,Logger}` (that is log4j-api); same shape as the fixture |
| log4j-core | 2.15.0 (log4shell-server) | 2.25.4 | **SAFE** | the famous Log4Shell module; usage is plain logging |
| jackson-databind | 2.6.5 (web-common) | 2.9.x+ (137 advisories) | **SAFE** or NEEDS CHANGES, medium | `ObjectMapper` use in web-common; 2.6 -> 2.9+ removed some `JsonParser.Feature`s; japicmp list will be long — the agent must find what is actually used |
| struts2-core | 2.3.20 | 6.x / 7.1.1 (major) | **NEEDS CHANGES** | `com.opensymphony.xwork2.ActionSupport` imports (16); Struts 6 moved packages (`org.apache.struts2.ActionSupport`); confirmed import sites are correct |
| spring-web / spring-context | 3.2.6.RELEASE (property) | 6.x (major x3) | **NEEDS CHANGES** | Spring 6 needs Java 17 + jakarta; `javax.persistence` imports break; `spring-test` 4 usages; also TARGET REQUIREMENTS should mention Java 17 |
| hibernate-core | 4.3.7.Final | 5.4.24.Final+ | **NEEDS CHANGES** likely | `hibernate-entitymanager` was merged into core in 5.2; `javax.persistence` + `TypedQuery` use in `todolist-core` |
| hibernate-validator | 4.3.1.Final | 6.2.0.Final / 7.x | **NEEDS CHANGES** likely | 5 usages of `org.hibernate.validator`; 6.x moved to `javax.validation` 2.0; 7.x to `jakarta.validation` |
| commons-collections | 3.1 (web-common) | 3.2.2 | **SAFE** | 1 import of `org.apache.commons`; patch-level fix |
| zt-zip | 1.12 | 1.13 | **SAFE** | `org.zeroturnaround.zip` 1 import; the fix is a zip-slip check |
| undertow-core | 2.2.13.Final (log4shell-server) | 2.3.21.Final | **SAFE**, medium | server bootstrap API stable |
| commons-lang3 | 3.11 | 3.18.0 | **SAFE** | |
| unboundid-ldapsdk | 3.1.1 | 4.0.5 | **SAFE**, medium | used by the log4shell exploit server |
| hsqldb | 2.3.2 | 2.7.1 | **SAFE** | JDBC driver only |

Traps: `log4shell-goof` is an exploit demo — the agent should not treat exploit code as "the app uses X".
