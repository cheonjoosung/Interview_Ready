### DSL (Domain-Specific Language)
- 정의
  + **도메인 특화 언어** (Domain-Specific Language)
  + Kotlin 문법을 활용해서 **내 도메인에 특화된 코드를 자연어처럼 작성**
  + 일반적인 코드보다 **선언적(declarative)**이고 **가독성 높음** 
  + 대표적인 예로 `Gradle Kotlin DSL`, `Jetpack Compose`, `Anko`, `HTML DSL` 

- 구성 핵심 요소
  + | 요소                  | 설명                                            |
    |---------------------|-----------------------------------------------|
    | `Lambda with receiver` | 람다 블록 내에서 `this`로 대상 객체의 함수/프로퍼티 사용 가능 |
    | `Extension function`   | 기존 타입에 함수를 추가하여 DSL 문법처럼 사용 가능         |
    | `Named arguments`      | 인자명을 명확하게 지정하여 의도를 코드에 드러낼 수 있음     |
    | `Function literals`    | 함수를 값처럼 전달 가능 → 고차함수 기반 DSL 구현 가능       |

- 대표 예제
  + HTML DSL
    * 각 블록은 수신 객체(this)를 가진 람다로 동작
    * ```kotlin
      fun html(block: HtmlBuilder.() -> Unit): String {
        val builder = HtmlBuilder()
        builder.block()
        return builder.build()
      }
      
      class HtmlBuilder {
        private val content = StringBuilder()

        fun body(block: BodyBuilder.() -> Unit) {
            content.append("<body>")
            val body = BodyBuilder()
            body.block()
            content.append(body.build())
            content.append("</body>")
        }
      
        fun build(): String = content.toString()
      }

      class BodyBuilder {
        private val content = StringBuilder()

        fun p(text: String) {
            content.append("<p>$text</p>")
        }
    
        fun build(): String = content.toString()
      }

        // 사용
      val result = html {
          body {
              p("Hello, Kotlin DSL!")
              p("This is a paragraph.")
          }
      }
      println(result) // <body><p>Hello, Kotlin DSL!</p><p>This is a paragraph.</p></body>
      ```
  + Gradle Kotlin DSL (build.gradle.kts)
    * plugins {}, repositories {}, dependencies {} 전부 DSL
    * ```kotlin
      plugins {
        kotlin("jvm") version "1.9.0"
        application
      }

      repositories {
        mavenCentral()
      }

      dependencies {
        implementation("com.squareup.retrofit2:retrofit:2.9.0")
        testImplementation("org.jetbrains.kotlin:kotlin-test")
      }

      application {
        mainClass.set("com.example.MainKt")
      }
      ```
  + SQL
    * JetBrains 의 Exposed 는 코틀린 DSL 로 SQL 작성할 수 있게 해주는 QRM 쿼리 빌더
    * ```kotlin
      object Users : Table() { 
        val id = integer("id").autoIncrement()
        val name = varchar("name", 50)
        override val primaryKey = PrimaryKey(id)
      }
      
      fun insertUser(name: String) {
            transaction {
                Users.insert {
                    it[Users.name] = name
                }
            }
        }

        fun getAllUsers(): List<String> {
            return transaction {
                Users.selectAll().map { it[Users.name] }
            }
        }
      ```
    * SQL 문법을 함수로 래핑한 DSL
    * insert {}, selectAll().map {} 등이 고차함수 + 람다 수신 기반
    * 타입 안정성 있는 쿼리 작성 가능
  + Jetpack Compose
    * ```kotlin
      fun GreetingScreen(name: String) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Text(text = "Hello, $name!")
            Button(onClick = { /* handle click */ }) {
            Text("Click Me")
            }
        }
      }
      ```
    * Column {}, Button {}, Text() 전부 DSL 스타일 컴포저블 함수
    * XML 없이 UI를 함수로 구성
    * 중첩 가능, 선언적, 가독성 높음

- 실전 DSL 활용 사례
  + | 사례                | 설명                    |
    | ----------------- | --------------------- |
    | Gradle Kotlin DSL | build.gradle.kts 파일   |
    | Jetpack Compose   | UI 선언을 함수로 구현         |
    | kotlinx.html      | HTML 빌더 DSL           |
    | SQL DSL           | Exposed 같은 라이브러리      |
    | Android UI DSL    | Anko (JetBrains, 구버전) |

- 면접 관련 질문 
  + Kotlin DSL은 어떤 상황에서 사용하는 게 적절하다고 생각하나요?
    * 특정 도메인 작업을 명확하고 선언적으로 표현하고 싶을 때
    * 반복적 구조, 트리 형태, 구성형 API 등에서 효과적 (예: Gradle, UI, SQL, HTML, JSON 등)
  + Jetpack Compose는 어떻게 DSL로 작동하나요?
    * Compose 함수들은 대부분 수신 객체를 가진 고차 함수
    * 예: Column { Text(...) }는 ColumnScope.() -> Unit 형태
    * 이로써 UI 구조를 중첩함수 + 선언형 스타일로 자연스럽게 표현 가능
  + build.gradle.kts vs build.gradle
    * | 항목     | Groovy DSL (`.gradle`) | Kotlin DSL (`.gradle.kts`) |
      | ------ | ---------------------- | -------------------------- |
      | 언어     | Groovy (동적)            | Kotlin (정적)                |
      | 타입 안정성 | ❌ 없음 (런타임 오류 가능)       | ✅ 컴파일 타임 검사 가능             |
      | IDE 지원 | 제한적 자동완성               | 강력한 자동완성 및 타입 추론           |
      | 유연성    | 문법 유연함 (다소 혼란스러울 수 있음) | 명확한 문법 필요 (컴파일 엄격)         |
      | 학습 난이도 | 진입 장벽 낮음               | Kotlin 익숙해야 쉽게 사용 가능       |


---
