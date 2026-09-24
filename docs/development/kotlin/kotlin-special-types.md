---
title: data, enum, sealed, object로 표현하는 Kotlin의 타입들
---

# data, enum, sealed, object로 표현하는 Kotlin의 타입들

## data class

데이터 표현이 주목적인 클래스에 사용한다.

```kotlin
data class User(
    val id: Long,
    val name: String
)
```

`equals`, `hashCode`, `toString`, `componentN`, `copy` 등이 자동으로 제공된다. Java였다면 IDE로 생성하거나 Lombok 같은 라이브러리에 의존해야 했던 보일러플레이트를, 클래스 선언 앞에 `data` 키워드 하나만 붙여서 대체하는 셈이다. 단, `data class`는 primary constructor에 파라미터가 최소 하나 이상 있어야 하고, `abstract`, `open`, `sealed`, `inner` 클래스로는 선언할 수 없다는 제약이 있다.

```kotlin
val user2 = user.copy(name = "Lee")
```

`copy()`는 지정한 프로퍼티만 바꾼 새로운 객체를 만들어 반환하고, 원본 객체는 그대로 유지한다. 이때 `equals`/`hashCode`는 primary constructor에 선언된 프로퍼티만을 기준으로 계산되므로, 본문에 별도로 선언한 프로퍼티는 동등성 비교에 포함되지 않는다는 점도 함께 알아두면 좋다.

## enum class

정해진 상수 집합을 표현한다.

```kotlin
enum class Status {
    READY,
    RUNNING,
    DONE
}
```

Kotlin의 enum은 단순한 상수 나열을 넘어, 각 상수마다 프로퍼티나 메서드를 가질 수도 있고 인터페이스를 구현할 수도 있다.

```kotlin
enum class Status(val label: String) {
    READY("대기중"),
    RUNNING("진행중"),
    DONE("완료");

    fun isFinished() = this == DONE
}
```

전체 상수 목록이 필요하다면 `Status.entries`(Kotlin 1.9 이상)를 사용할 수 있고, 그 이전 버전에서는 `Status.values()`를 사용한다.

## sealed class / interface

제한된 타입 계층을 표현한다.

```kotlin
sealed interface Result {
    data class Success(val value: String) : Result
    data class Failure(val error: Throwable) : Result
}
```

`when`과 함께 사용하면 가능한 하위 타입을 컴파일러가 파악할 수 있다는 장점이 있다. 그래서 `when (result) { ... }`으로 모든 하위 타입을 분기 처리하면 `else` 분기 없이도 컴파일러가 "모든 경우를 다뤘다"고 판단해준다. `enum`과 비슷하게 하위 타입의 종류가 제한되어 있다는 점은 같지만, `enum`은 상수 하나당 상태가 고정되어 있는 반면 `sealed`의 각 하위 타입은 `Success(val value: String)`처럼 서로 다른 형태의 데이터를 가질 수 있다는 차이가 있다. sealed 타입의 하위 클래스는 원칙적으로 같은 패키지, 같은 모듈 안에 선언되어야 한다.

## object

싱글턴 객체를 선언한다.

```kotlin
object Logger {
    fun log(message: String) {
        println(message)
    }
}
```

`object`로 선언한 타입은 클래스이자 동시에 그 클래스의 유일한 인스턴스이며, 별도의 동기화 코드 없이도 초기화 시점의 스레드 안전성이 보장된다(JVM의 클래스 로딩 메커니즘을 그대로 활용하기 때문이다). `class`와 마찬가지로 인터페이스를 구현하거나 다른 클래스를 상속할 수도 있다.

## companion object

클래스에 연결된 객체를 정의한다.

```kotlin
class User private constructor(val name: String) {
    companion object {
        fun create(name: String): User {
            return User(name)
        }
    }
}
```

Java의 `static`과 비슷하게 사용되는 경우가 많지만 실제 개념은 객체라는 차이가 있다. `companion object`는 클래스당 하나만 가질 수 있고, 이름을 붙이지 않으면 `Companion`이라는 기본 이름을 갖는다. 실제로는 하나의 객체이기 때문에 인터페이스를 구현하거나 다른 클래스를 상속할 수도 있어서, 단순한 정적 유틸리티 이상의 역할(예: 팩토리 패턴, 특정 인터페이스의 기본 구현 제공 등)을 맡길 수 있다.

`data`, `enum`, `sealed`, `object`가 상태와 타입 계층을 표현하는 방법이라면, `interface`와 `abstract class`는 역할과 공통 구현을 표현하는 방법이다.

## Interface

```kotlin
interface Flyable {
    fun fly()

    fun land() {
        println("land")
    }
}
```

인터페이스에도 구현이 있는 메서드를 정의할 수 있다. 다만 인터페이스는 상태(값을 실제로 저장하는 backing field)를 가질 수 없다는 제약이 있다. 프로퍼티를 선언할 수는 있지만, 그 프로퍼티는 구현하는 쪽에서 값을 채워 넣거나 커스텀 getter로 계산해서 제공해야 한다.

## Abstract Class

```kotlin
abstract class Animal {
    abstract fun sound()

    fun sleep() {
        println("sleep")
    }
}
```

추상 클래스는 인터페이스와 달리 생성자를 가질 수 있고, 상태를 저장하는 일반 프로퍼티도 가질 수 있다. 즉 "구현이 없는 메서드(`abstract fun sound()`)"와 "이미 구현된 메서드(`sleep()`)", 그리고 "저장된 상태"를 함께 하위 클래스에 물려주고 싶을 때 인터페이스보다 추상 클래스가 더 적합하다.

## 구현과 상속

```kotlin
class Bird : Animal(), Flyable {
    override fun sound() {
        println("tweet")
    }

    override fun fly() {
        println("fly")
    }
}
```

Kotlin은 클래스의 다중 상속은 지원하지 않지만 여러 인터페이스를 구현할 수 있다. 상위 타입을 나열할 때는 클래스는 하나만(있다면 항상 생성자 호출과 함께, `Animal()`처럼) 앞에 오고, 그 뒤로 인터페이스를 원하는 만큼 콤마로 나열한다. 만약 클래스와 여러 인터페이스에 동일한 시그니처의 메서드가 있어서 어떤 구현을 사용할지 컴파일러가 판단할 수 없다면, `override`한 뒤 `super<Animal>.sound()`처럼 어떤 타입의 구현을 호출할지 명시해야 한다.

## 언제 무엇을 사용할까?

인터페이스는 주로 역할과 계약을 표현하는 데 적합하다. 추상 클래스는 공통 상태나 구현을 강하게 공유해야 하는 타입 계층에 적합하다. 예를 들어 "날 수 있다"처럼 서로 관련 없는 여러 타입이 공통으로 가질 수 있는 행동은 인터페이스로 표현하는 것이 자연스럽고, "동물은 모두 나이와 이름이라는 상태를 갖는다"처럼 하나의 개념 계층 안에서 상태와 기본 구현을 공유해야 한다면 추상 클래스가 더 적합하다.

## 정리

- `data class`는 `equals`/`hashCode`/`toString`/`copy`/`componentN`을 자동으로 만들어주지만, 이 값들은 모두 primary constructor의 프로퍼티만을 기준으로 계산된다.
- `enum class`는 상수마다 상태와 동작을 가질 수 있고, `sealed`는 서로 다른 형태의 데이터를 가진 하위 타입 계층을 `when`과 함께 안전하게 다룰 수 있게 해준다.
- `object`는 스레드 세이프한 싱글턴이고, `companion object`는 클래스에 연결된 하나의 객체로서 Java의 `static`과 유사하게 쓰이지만 실제로는 인터페이스 구현 등 더 많은 일을 할 수 있다.
- 인터페이스는 상태를 가질 수 없고 역할/계약을 표현하는 데, 추상 클래스는 상태와 공통 구현을 함께 공유하는 데 적합하다. 상속 자체보다 **역할을 인터페이스로 표현하고 필요한 구현만 공유하는 것**에 초점을 두는 것이 좋다.
