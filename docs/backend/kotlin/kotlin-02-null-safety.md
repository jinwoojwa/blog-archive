---
series: Java 개발자를 위한 Kotlin 기초 문법
title: Null Safety
---

# Null Safety

> [코틀린 공식 문서](https://kotlinlang.org/docs/basic-syntax.html#variables)를 정리한 내용이다.
> Java를 이미 알고 있다는 전제 하에, Java와 Kotlin의 차이점을 위주로 정리한다.

## Null Safety

```java
String name = null;

System.out.println(name.length());
```

- 위 Java 코드는 컴파일은 되지만, 런타임에 `NullPointerException`이 발생한다. 즉 컴파일러가 null 가능성을 잡아주지 못한다.

```kotlin
var name: String = "Kim"

// 컴파일 에러
name = null
```

- Kotlin에서는 타입 시스템 차원에서 null 허용 여부를 구분한다. 기본 `String` 타입에는 null을 대입할 수 없다.
- null을 허용하고 싶다면 타입 뒤에 `?`를 붙여 nullable 타입으로 선언해야 한다.

```kotlin
var name: String? = null
```

- 이렇게 하면 "이 변수는 null일 수 있다"는 사실이 타입에 드러나고, 컴파일러가 이후 사용 지점마다 null 처리를 강제한다.

## ?. (Safe Call)

nullable 객체에 접근할 때는 `?.`를 사용한다.

```kotlin
val name: String? = null

val length = name?.length
```

위 코드는 아래 Java 코드와 동일하게 동작한다.

```java
Integer length;

if (name != null) {
    length = name.length();
} else {
    length = null;
}
```

- `name`이 null이 아니면 `length`를 정상적으로 반환하고, null이면 뒤의 호출을 건너뛰고 전체 결과가 null이 된다.
- 매번 if문으로 null 체크를 하지 않아도 되므로 코드가 훨씬 간결해진다.

## `?:` Elvis Operator

```java
String result;

if (name != null) {
    result = name;
} else {
    result = "Unknown";
}
```

```kotlin
val result = name ?: "Unknown"
```

- `?:`는 좌변이 null이면 우변 값을 사용한다는 의미다. null인 경우의 기본값을 지정할 때 사용한다.
- 주로 `?.`와 함께 사용해서 "null이면 기본값" 패턴을 한 줄로 표현한다.

```kotlin
val length = name?.length ?: 0
```

## `!!` (Non-null Assertion)

```kotlin
val name: String? = getName()

println(name!!.length)
```

- `!!`는 "이 값은 null이 아님을 내가 보장한다"고 컴파일러에게 알리는 연산자다.
- 실제로 값이 null이면 그 시점에 `NullPointerException`이 발생한다. 즉 Kotlin의 null 안전성을 우회하는 탈출구이자, NPE를 다시 만들어내는 주범이다.
- 정말 null이 될 수 없다는 확신이 있는 경우가 아니라면 사용을 피하고, `?.`나 `?:`, 스마트 캐스트 등으로 대체하는 것이 좋다.

## 정리

- Kotlin은 nullable 여부(`String` vs `String?`)를 타입 시스템에 포함시켜, 컴파일 타임에 NPE 가능성을 잡아낼 수 있다.
- `?.`는 null이면 호출을 건너뛰고, `?:`는 null일 때 기본값을 지정한다.
- `!!`는 null 안전성을 우회하는 연산자이므로 꼭 필요한 경우가 아니라면 피하는 것이 좋다.

다음 편에서는 제어문(`if`, `when`, `for`)과 상속, static, 예외 처리 등 나머지 문법을 다룬다.
