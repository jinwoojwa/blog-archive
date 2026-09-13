---
series: Java 개발자를 위한 Kotlin 기초 문법
title: 제어문과 기타 문법
---

# 제어문과 기타 문법

> [코틀린 공식 문서](https://kotlinlang.org/docs/basic-syntax.html#variables)를 정리한 내용이다.
> Java를 이미 알고 있다는 전제 하에, Java와 Kotlin의 차이점을 위주로 정리한다.

## instanceof → is

```java
if (obj instanceof String) {
    String str = (String) obj;
    System.out.println(str.length());
}
```

```kotlin
if (obj is String) {
    println(obj.length)
}

// ---------------------

if (obj !is String) {
    return
}

println(obj.length)
```

- Java의 `instanceof`에 대응하는 연산자가 `is`이고, 부정형은 `!is`다.
- Java는 타입을 확인한 뒤에도 명시적으로 캐스팅을 해야 하지만, Kotlin은 `is`로 타입을 확인하면 해당 블록 안에서 컴파일러가 자동으로 타입을 좁혀준다(스마트 캐스트). 그래서 별도의 캐스팅 코드 없이 바로 `obj.length`처럼 사용할 수 있다.

## 캐스팅: (Type) → as

```java
String name = (String) obj;
```

```kotlin
val name = obj as String

// 캐스팅 실패 시 예외 대신 null을 반환하는 안전한 캐스팅
val name = obj as? String

val name = obj as? String ?: "Unknown"
```

- `as`는 Java의 캐스팅 `(Type)`에 대응하며, 캐스팅에 실패하면 `ClassCastException`이 발생한다.
- `as?`는 캐스팅에 실패해도 예외를 던지지 않고 null을 반환한다. Elvis 연산자와 조합하면 "캐스팅 실패 시 기본값" 패턴을 안전하게 작성할 수 있다.

## if (expression)

```java
String result;

if (age >= 20) {
    result = "adult";
} else {
    result = "child";
}
```

```kotlin
val result =
    if (age >= 20) {
        "adult"
    } else {
        "child"
    }

// 삼항연산자가 따로 없다. if 자체가 값을 반환하기 때문이다
val result = if (age >= 20) "adult" else "child"
```

- Kotlin의 `if`는 문(statement)이 아니라 표현식(expression)이다. 즉 `if` 자체가 값을 만들어낼 수 있다.
- 그래서 Java에는 있는 삼항 연산자(`? :`)가 Kotlin에는 존재하지 않는다. `if`-`else` 표현식이 그 역할을 대신한다.

## switch → when

```java
switch (status) {
    case 1:
        return "READY";
    case 2:
        return "RUNNING";
    default:
        return "UNKNOWN";
}
```

```kotlin
return when (status) {
    1 -> "READY"
    2 -> "RUNNING"
    else -> "UNKNOWN"
}

// if와 마찬가지로 표현식이므로 함수 본문에 바로 사용할 수 있다
fun getStatus(status: Int) = when (status) {
    1 -> "READY"
    2 -> "RUNNING"
    else -> "UNKNOWN"
}

// 값 하나만 비교하는 것이 아니라, 조건마다 다른 타입/패턴을 검사할 수도 있다
when (obj) {
    1 -> println("숫자 1")
    "hello" -> println("문자열")
    is String -> println("String")
    else -> println("기타")
}
```

- `when`은 Java의 `switch`에 대응하지만 훨씬 강력하다. `break`가 필요 없고, 값뿐 아니라 타입 검사(`is`), 범위 검사(`in`) 등 다양한 조건을 분기 조건으로 사용할 수 있다.
- `if`처럼 표현식으로 사용할 수 있어 `when`의 결과를 바로 반환하거나 변수에 대입할 수 있다.

## String interpolation

```java
String message = "name = " + name + ", age = " + age;
```

```kotlin
val message = "name = $name, age = $age"

// 단순 변수가 아니라 표현식을 넣고 싶다면 중괄호로 감싼다
val message = "내년 나이 = ${age + 1}"
```

- 문자열 안에 `$변수명` 또는 `${표현식}` 형태로 값을 바로 삽입할 수 있다.
- Java의 문자열 `+` 연결이나 `String.format()`보다 가독성이 좋다.

## it

```kotlin
val names = listOf("kim", "lee", "park")

val upperNames = names.map { it.uppercase() }
```

- 람다식의 파라미터가 하나뿐이고 그 이름을 굳이 정할 필요가 없을 때, Kotlin은 `it`이라는 암묵적인 이름을 제공한다.
- 위 코드는 아래와 같이 파라미터 이름을 명시한 것과 동일하다.

```kotlin
val upperNames = names.map { name -> name.uppercase() }
```

- 람다가 중첩되어 `it`이 어떤 값을 가리키는지 헷갈릴 수 있는 경우에는, `it` 대신 명시적인 파라미터 이름을 붙이는 것이 가독성 측면에서 더 낫다.

## for 문

```java
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}

for (String name : names) {
    System.out.println(name);
}
```

```kotlin
// 범위(range)를 이용한 반복
for (i in 0 until 5) {
    println(i)
}

// 0..4 는 4를 포함하고, 0 until 5 는 5를 포함하지 않는다
for (i in 0..4) {
    println(i)
}

// 컬렉션/배열 순회
for (name in names) {
    println(name)
}

// 역순, step 지정도 가능하다
for (i in 10 downTo 0 step 2) {
    println(i)
}
```

- Kotlin의 `for`는 전통적인 C 스타일 `for (초기화; 조건; 증감)` 문법을 지원하지 않고, `for (요소 in 컬렉션/범위)` 형태만 지원한다.
- 인덱스가 필요한 경우 `withIndex()`를 사용하면 인덱스와 값을 함께 순회할 수 있다.

```kotlin
for ((index, name) in names.withIndex()) {
    println("$index: $name")
}
```

## 상속과 final class

```java
public class Animal {
    public void sound() {
        System.out.println("...");
    }
}

public class Dog extends Animal {
    @Override
    public void sound() {
        System.out.println("Woof");
    }
}
```

```kotlin
// 기본적으로 상속이 불가능하다 (컴파일 에러)
class Animal {
    fun sound() {
        println("...")
    }
}

// class Dog : Animal() // 에러: Animal은 final
```

- Java는 클래스와 메서드가 기본적으로 상속/오버라이드가 가능하고, 막고 싶을 때만 `final`을 붙인다.
- Kotlin은 정반대로, 클래스와 메서드가 기본적으로 `final`이다. 상속을 허용하려면 명시적으로 `open` 키워드를 붙여야 한다.

```kotlin
open class Animal {
    open fun sound() {
        println("...")
    }
}

class Dog : Animal() {
    override fun sound() {
        println("Woof")
    }
}
```

- 상속을 의도적으로 설계하지 않은 클래스가 실수로 상속되는 것을 막기 위한 설계다. "상속을 위한 설계와 문서화가 없다면 상속을 금지하라"는 원칙이 언어 차원에서 강제되는 셈이다.

## static 없음

```java
public class MathUtils {
    public static int square(int x) {
        return x * x;
    }
}

MathUtils.square(3);
```

```kotlin
class MathUtils {
    companion object {
        fun square(x: Int) = x * x
    }
}

MathUtils.square(3)
```

- Kotlin에는 Java의 `static` 키워드가 없다. 클래스에 종속된 멤버가 필요하면 클래스 내부에 `companion object`를 선언한다.
- `companion object`는 클래스당 하나만 존재할 수 있고, 클래스 이름으로 바로 접근할 수 있다는 점에서 `static` 멤버와 비슷하게 동작한다.
- 특정 클래스에 속할 필요가 없는 순수한 유틸리티 함수라면, 굳이 클래스로 감싸지 않고 파일 최상위에 함수를 바로 선언하는 것이 더 Kotlin다운 방식이다.

```kotlin
fun square(x: Int) = x * x
```

## Checked Exception이 없음

```java
public void readFile() throws IOException {
    FileReader reader = new FileReader("file.txt");
}

// 호출하는 쪽에서 반드시 처리해야 한다
try {
    readFile();
} catch (IOException e) {
    // ...
}
```

```kotlin
fun readFile() {
    val reader = FileReader("file.txt")
}

// throws 선언도, try-catch 강제도 없다
readFile()
```

- Java는 `IOException`처럼 컴파일러가 처리를 강제하는 Checked Exception이 존재한다. 메서드 시그니처에 `throws`를 명시하고, 호출하는 쪽에서 반드시 `try-catch`나 재선언을 해야 한다.
- Kotlin에는 Checked Exception 개념이 아예 없다. 모든 예외가 Unchecked Exception처럼 동작하므로, 예외 처리 여부를 컴파일러가 강제하지 않는다.
- 대신 예외 처리 누락을 컴파일 타임에 잡아주지 못하므로, 실제로 예외가 발생할 수 있는 지점은 문서화나 코드 리뷰 등으로 별도 관리할 필요가 있다.

## 정리

- `if`, `when` 모두 표현식으로 동작해 값을 바로 반환하거나 대입할 수 있고, Kotlin에는 별도의 삼항 연산자가 없다.
- `for`는 range와 `in` 기반으로만 동작하고, C 스타일 `for`는 지원하지 않는다.
- 클래스/메서드는 기본이 `final`이며, 상속을 허용하려면 `open`을 명시해야 한다.
- `static` 대신 `companion object`나 파일 최상위 함수를 사용한다.
- Checked Exception이 없어 `throws` 선언이나 강제 `try-catch`가 필요 없다.

3편에 걸쳐 살펴본 것처럼, Kotlin의 문법은 Java에서 반복적으로 작성해야 했던 보일러플레이트를 줄이고, null 안전성처럼 런타임 에러를 컴파일 타임 에러로 앞당기는 방향으로 설계되어 있다.
