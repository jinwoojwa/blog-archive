---
series: Java 개발자를 위한 Kotlin 기초 문법
title: 변수, 함수, 클래스
---

# 변수, 함수, 클래스

> [코틀린 공식 문서](https://kotlinlang.org/docs/basic-syntax.html#variables)를 정리한 내용이다.
> Java를 이미 알고 있다는 전제 하에, Java와 Kotlin의 차이점을 위주로 정리한다.

## val / var

```kotlin
val name = "kim"
var age = 25
```

- `val`은 재할당이 불가능한 읽기 전용 변수를 선언한다.
- `var`은 재할당이 가능한 가변 변수를 선언한다.
- Kotlin은 타입 추론을 지원하므로, 초기값이 있다면 타입을 생략할 수 있다.
- Java와 달리 변수 선언 시 `val`/`var`를 먼저 쓰고, 굳이 타입을 명시하지 않아도 된다.
- 특별한 이유가 없다면 `var`보다 `val`을 우선적으로 사용하는 것이 권장된다. 불변 변수를 기본값으로 두면 예상치 못한 값 변경으로 인한 버그를 줄일 수 있다.

`val`은 변수 자체의 재할당만 막을 뿐, 객체 내부 상태까지 불변으로 만들지는 않는다.

```kotlin
val list = mutableListOf(1, 2, 3)

list.add(4)                // 가능: 객체 내부 상태는 변경 가능
// list = mutableListOf()  // 불가능: 재할당은 컴파일 에러
```

## 타입 선언

```kotlin
val name: String
val age: Int

fun hello(name: String): String {
    return "hello $name"
}
```

- 변수는 `이름: 타입` 순서로 선언한다. Java(`String name`)와 반대 순서라는 점에 유의한다.
- 함수는 `fun 함수명(파라미터: 타입): 반환타입` 형태로 선언한다.
- 타입을 먼저 쓰지 않고 이름을 먼저 쓰는 방식은 타입 추론과도 잘 어울린다. 초기값만 봐도 타입을 유추할 수 있는 경우가 많기 때문이다.

## 함수 선언

```java
public int sum(int a, int b) {
    return a + b;
}
```

```kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}

// 본문이 표현식 하나뿐이라면 중괄호와 return을 생략하고 이렇게 줄여 쓸 수 있다
fun sum(a: Int, b: Int) = a + b
```

- Java의 `void`에 대응하는 타입은 `Unit`이다.
- 반환 타입이 `Unit`이면 타입 선언 자체를 생략할 수 있다.

```kotlin
fun printName(name: String): Unit {
    println(name)
}

// Unit은 생략 가능
fun printName(name: String) {
    println(name)
}
```

## new 키워드 없음

```java
User user = new User("Kim", 30);
```

```kotlin
val user = User("Kim", 30)

// 생성자 호출도 일반 함수 호출과 동일한 문법을 사용한다
```

- Kotlin에는 `new` 키워드가 없다. 객체 생성도 함수를 호출하는 것과 똑같은 문법으로 처리한다.

## 클래스 생성

```java
public class User {

    private final String name;
    private final int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```

```kotlin
class User(
    val name: String,
    val age: Int
)
```

- 생성자, 필드, getter가 primary constructor 한 줄로 한 번에 정의된다.
- 필드 접근도 getter 메서드 호출이 아니라 `user.name`처럼 프로퍼티 접근 문법으로 한다.
- `val`로 선언하면 읽기 전용 프로퍼티(getter만 생성), `var`로 선언하면 가변 프로퍼티(getter/setter 모두 생성)가 된다.

## data class

```java
public class UserDto {

    private String name;
    private int age;

    // constructor
    // getter
    // setter
    // equals
    // hashCode
    // toString
}
```

```kotlin
data class UserDto(
    val name: String,
    val age: Int
)
```

- `data` 키워드를 붙이면 `equals()`, `hashCode()`, `toString()`, `copy()` 등을 컴파일러가 자동으로 생성해준다.
- DTO, VO처럼 데이터를 담는 역할만 하는 클래스에 적합하다.
- `copy()`를 이용하면 일부 필드만 바꾼 새 객체를 손쉽게 만들 수 있다. 원본 객체는 그대로 유지된다.

```kotlin
val user1 = UserDto("Kim", 30)

val user2 = user1.copy(age = 31)
```

## 정리

- `val`/`var`로 가변성을 타입 선언 시점에 구분하고, 기본적으로 `val`을 우선 사용한다.
- 변수는 `이름: 타입`, 함수는 `fun 함수명(파라미터: 타입): 반환타입` 순서로 선언한다.
- 객체 생성에 `new` 키워드가 필요 없다.
- 생성자, 클래스, `data class`를 이용해 Java에서 반복 작성하던 보일러플레이트 코드를 크게 줄일 수 있다.

다음 편에서는 Kotlin의 핵심 특징 중 하나인 Null Safety를 다룬다.
