---
title: 클래스 내부 들여다보기 - 생성자, 상속, 프로퍼티
---

# 클래스 내부 들여다보기: 생성자, 상속, 프로퍼티

## Primary Constructor(주 생성자)

Kotlin에서는 primary constructor의 `constructor` 키워드를 생략할 수 있다.

```kotlin
class User constructor(
    val name: String,
    var age: Int
)
```

위 코드는 일반적으로 다음처럼 축약해서 작성한다.

```kotlin
class User(
    val name: String,
    var age: Int
)
```

생성자 매개변수 앞에 `val` 또는 `var`를 붙이면 생성자 파라미터이면서 동시에 클래스의 프로퍼티가 된다.

```kotlin
class User(
    val name: String,
    var age: Int
)
```

반대로 `val` 또는 `var`를 붙이지 않으면 프로퍼티가 아니라 단순한 생성자 파라미터다.

```kotlin
class User(name: String, age: Int) {

    val greeting = "Hello, $name"

    init {
        println("age = $age")
    }
}
```

이 경우 `name`과 `age`는 프로퍼티 초기화식과 `init` 블록에서는 사용할 수 있지만, 일반 멤버 함수에서 접근하거나 객체 생성 이후 외부에서 `user.name`처럼 접근할 수는 없다.

```kotlin
class User(name: String) {

    val greeting = "Hello, $name"

    init {
        println(name)
    }

    fun printName() {
        // println(name) // 컴파일 에러
    }
}
```

외부에서도 마찬가지다.

```kotlin
val user = User("Kim")

// println(user.name) // 컴파일 에러
```

반면 생성자 파라미터 앞에 `val` 또는 `var`를 붙이면 프로퍼티가 되기 때문에 객체 생성 이후에도 접근할 수 있다.

```kotlin
class User(
    val name: String,
    var age: Int
)

val user = User("Kim", 20)

println(user.name)
println(user.age)

user.age = 21
```

`val` 프로퍼티는 읽기만 가능하고, `var` 프로퍼티는 값을 변경할 수 있다.

---

## init 블록

객체 생성 시 실행할 초기화 로직은 `init` 블록에 작성한다.

```kotlin
class User(val age: Int) {

    init {
        require(age >= 0)
    }
}
```

`init` 블록은 primary constructor의 실행 과정에 포함된다.

Kotlin의 primary constructor에는 Java처럼 별도의 생성자 본문이 존재하지 않기 때문에, 생성 과정에서 실행해야 하는 로직을 `init` 블록에 작성한다.

`init` 블록은 여러 개 선언할 수 있다.

```kotlin
class User(val name: String) {

    init {
        println("first init")
    }

    init {
        println("second init")
    }
}
```

프로퍼티 초기화식과 `init` 블록은 클래스 본문에 작성된 순서대로 위에서부터 실행된다.

```kotlin
class User(name: String) {

    val first = run {
        println("1. property")
        name
    }

    init {
        println("2. init")
    }

    val second = run {
        println("3. property")
        name.length
    }

    init {
        println("4. init")
    }
}
```

객체를 생성하면 다음 순서대로 실행된다.

```text
1. property
2. init
3. property
4. init
```

`init` 블록은 주로 생성자 인자를 검증하거나 여러 프로퍼티에 걸친 초기화 로직을 수행할 때 사용한다.

```kotlin
class User(
    val name: String,
    val age: Int
) {

    init {
        require(name.isNotBlank())
        require(age >= 0)
    }
}
```

---

## Secondary Constructor(보조 생성자)

Kotlin에서는 primary constructor 외에도 secondary constructor를 정의할 수 있다.

```kotlin
class User(val name: String) {

    constructor() : this("Unknown")
}
```

Secondary constructor는 `constructor` 키워드를 반드시 사용한다.

클래스에 primary constructor가 존재한다면 모든 secondary constructor는 직접 또는 다른 secondary constructor를 거쳐 간접적으로 primary constructor에 위임해야 한다.

```kotlin
class User(
    val name: String,
    val age: Int
) {

    constructor(name: String) : this(name, 0)

    constructor() : this("Unknown")
}
```

이렇게 primary constructor 실행 경로를 반드시 거치도록 하는 이유는 프로퍼티 초기화와 `init` 블록을 포함한 클래스 초기화 과정을 일관되게 보장하기 위해서다.

초기화 순서는 대략 다음과 같다.

```text
Primary Constructor
→ 프로퍼티 초기화식 / init 블록
→ Secondary Constructor 본문
```

예를 들어:

```kotlin
class User(val name: String) {

    init {
        println("init")
    }

    constructor() : this("Unknown") {
        println("secondary")
    }
}
```

다음과 같이 실행된다.

```text
init
secondary
```

다만 primary constructor가 아예 없는 클래스라면 secondary constructor만 정의할 수도 있다.

```kotlin
class User {

    constructor(name: String) {
        println(name)
    }
}
```

Kotlin에서는 default argument를 활용하면 여러 생성자를 만들지 않고도 같은 목적을 달성할 수 있는 경우가 많기 때문에 secondary constructor의 사용 빈도는 Java보다 낮은 편이다.

```kotlin
class User(
    val name: String = "Unknown",
    val age: Int = 0
)
```

---

## 상속

Kotlin의 클래스와 메서드는 기본적으로 `final`이다.

즉 별도의 설정이 없다면 다른 클래스에서 상속하거나 오버라이드할 수 없다.

```kotlin
class Animal
```

다음과 같이 상속하려고 하면 컴파일 에러가 발생한다.

```kotlin
// class Dog : Animal()
```

상속을 허용하려면 클래스에 `open`을 명시해야 한다.

```kotlin
open class Animal
```

메서드도 마찬가지다.

```kotlin
open class Animal {

    open fun sound() {
        println("sound")
    }
}

class Dog : Animal() {

    override fun sound() {
        println("bark")
    }
}
```

클래스에 `open`을 붙였다고 해서 내부의 모든 메서드가 자동으로 오버라이드 가능한 것은 아니다.

오버라이드를 허용할 메서드에도 각각 `open`을 붙여야 한다.

```kotlin
open class Animal {

    fun eat() {
        println("eat")
    }

    open fun sound() {
        println("sound")
    }
}
```

위 코드에서 `sound()`는 오버라이드할 수 있지만 `eat()`은 오버라이드할 수 없다.

하위 클래스에서는 오버라이드하는 메서드에 반드시 `override`를 명시해야 한다.

```kotlin
class Dog : Animal() {

    override fun sound() {
        println("bark")
    }
}
```

이는 실수로 상위 클래스의 메서드를 가리는 문제를 방지하기 위한 Kotlin의 안전장치다.

`Dog : Animal()`처럼 상위 클래스 이름 뒤에 괄호가 붙는 이유는 하위 클래스 생성 과정에서 상위 클래스의 생성자도 함께 호출되어야 하기 때문이다.

```kotlin
open class Animal(val name: String)

class Dog(name: String) : Animal(name)
```

인터페이스 구현과 비교하면 차이가 더 명확하다.

```kotlin
open class Animal

interface Walkable

class Dog : Animal(), Walkable
```

클래스는 생성자를 호출해야 하기 때문에 `Animal()`처럼 작성하지만, 인터페이스는 생성자가 없기 때문에 `Walkable`처럼 괄호 없이 작성한다.

---

## 접근 제한자

Kotlin에서 주요 접근 제한자는 다음과 같다.

- `public`
- `private`
- `protected`
- `internal`

`public`은 기본값이므로 생략할 수 있다.

```kotlin
class User
```

위 코드는 다음과 동일하다.

```kotlin
public class User
```

Java와 비교하면 두 가지 차이가 눈에 띈다.

첫째, Kotlin은 접근 제한자를 생략하면 `public`이 기본값이다.

반면 Java에서는 접근 제한자를 생략하면 package-private이 된다.

둘째, Kotlin에는 Java의 package-private에 정확하게 대응하는 접근 제한자가 없다.

대신 `internal`이라는 접근 제한자를 제공한다.

```kotlin
internal class User
```

`internal`은 같은 Kotlin 모듈 안에서 접근할 수 있도록 제한한다.

여기서 모듈은 대략 함께 컴파일되는 코드 단위라고 이해하면 된다.

JVM 기반의 일반적인 Gradle/Maven 프로젝트에서는 하나의 Gradle 또는 Maven 모듈이 이에 대응한다고 이해할 수 있다.

생성자, 상속, 접근 제한자로 클래스의 전체적인 구조를 잡았다면, 그다음은 클래스 내부의 값을 어떻게 표현하고 노출하는지 살펴볼 차례다.

---

## 프로퍼티

Kotlin에서 객체가 가지는 값을 표현할 때 프로퍼티를 사용한다.

```kotlin
class User {

    var name: String = ""
}
```

Kotlin의 프로퍼티는 단순한 public field와 동일한 개념이 아니다.

프로퍼티는 값에 대한 접근 방식을 추상화한 언어 기능이며 getter와 setter 개념을 포함한다.

예를 들어:

```kotlin
class User {

    var name: String = ""
}
```

Kotlin에서는 다음처럼 접근한다.

```kotlin
val user = User()

user.name = "Kim"
println(user.name)
```

Java에서는 일반적으로 다음과 같은 형태로 사용한다.

```java
user.setName("Kim");
System.out.println(user.getName());
```

즉 Kotlin에서는 getter와 setter를 직접 호출하기보다 `user.name`과 같은 프로퍼티 문법으로 접근한다.

일반적인 Kotlin 프로퍼티는 JVM에서 backing field와 getter/setter 메서드 형태로 표현될 수 있다.

다만 모든 프로퍼티가 반드시 backing field를 가지는 것은 아니다.

값을 직접 저장하지 않고 접근할 때마다 계산하는 프로퍼티도 존재한다.

---

## Custom Getter

Getter의 동작을 직접 정의할 수 있다.

```kotlin
class Rectangle(
    val width: Int,
    val height: Int
) {

    val area: Int
        get() = width * height
}
```

`area`는 별도의 값을 저장하지 않는다.

```kotlin
val rectangle = Rectangle(10, 20)

println(rectangle.area)
```

`area`에 접근할 때마다 다음 계산이 실행된다.

```kotlin
width * height
```

따라서 `area`는 별도의 backing field가 필요하지 않은 계산 프로퍼티다.

이처럼 값을 미리 저장해 두는 대신 접근할 때마다 계산하도록 만들 수 있다.

다만 계산 비용이 크거나 매우 자주 조회되는 값이라면 매번 계산된다는 점을 고려해야 한다.

---

## Custom Setter와 Backing Field

Setter의 동작도 직접 정의할 수 있다.

```kotlin
class User {

    var name: String = ""
        set(value) {
            field = value.trim()
        }
}
```

이제 값을 대입하면 자동으로 앞뒤 공백이 제거된다.

```kotlin
val user = User()

user.name = "  Kim  "

println(user.name)
// Kim
```

여기서 `field`는 해당 프로퍼티의 backing field를 가리키는 특별한 식별자다.

Setter 내부에서 다음과 같이 작성하면 안 된다.

```kotlin
class User {

    var name: String = ""
        set(value) {
            name = value.trim()
        }
}
```

`name = ...`은 다시 `name`의 setter를 호출한다.

즉 setter 내부에서 자기 자신을 다시 호출하게 되어 무한 재귀에 빠진다.

이를 방지하기 위해 실제 저장 공간에 접근할 때는 `field`를 사용한다.

```kotlin
field = value.trim()
```

`field`는 getter 또는 setter와 같은 프로퍼티 accessor 내부에서 사용하는 특별한 식별자다.

---

## lateinit

Kotlin의 non-null 프로퍼티는 일반적으로 선언할 때 초기값을 제공해야 한다.

```kotlin
var name: String = ""
```

하지만 객체 생성 시점에는 값을 알 수 없고, 이후에 반드시 값을 넣어야 하는 경우도 있다.

이럴 때 `lateinit`을 사용할 수 있다.

```kotlin
lateinit var name: String
```

`lateinit`을 사용하면 초기값을 바로 지정하지 않아도 non-null 타입을 선언할 수 있다.

```kotlin
class User {

    lateinit var name: String

    fun initialize() {
        name = "Kim"
    }
}
```

다만 초기화 전에 접근하면 예외가 발생한다.

```kotlin
val user = User()

println(user.name)
```

이 경우 다음 예외가 발생한다.

```text
UninitializedPropertyAccessException
```

`lateinit`에는 몇 가지 제약이 있다.

- `var`에만 사용할 수 있다.
- nullable 타입에는 사용할 수 없다.
- `Int`, `Boolean` 등 JVM primitive 타입에 대응하는 프로퍼티에는 사용할 수 없다.

초기화 여부를 확인해야 한다면 프로퍼티를 선언한 클래스 내부에서 다음과 같이 확인할 수 있다.

```kotlin
class User {

    lateinit var name: String

    fun printName() {
        if (this::name.isInitialized) {
            println(name)
        }
    }
}
```

`lateinit`은 편리하지만 초기화 시점을 개발자가 직접 관리해야 하기 때문에 무분별하게 사용하면 런타임 예외가 발생할 가능성이 있다.

---

## lazy

`lazy`는 프로퍼티 값을 처음 사용할 때 계산하도록 만들 수 있는 기능이다.

```kotlin
val value: String by lazy {

    println("initialize")

    "Hello"
}
```

프로퍼티에 처음 접근하면 `lazy`에 전달한 람다가 실행된다.

```kotlin
println(value)
```

처음 접근할 때:

```text
initialize
Hello
```

가 출력된다.

하지만 두 번째부터는 초기화 로직이 다시 실행되지 않는다.

```kotlin
println(value)
println(value)
```

`lazy`는 최초 접근 시 계산된 값을 내부에 저장하고 이후에는 같은 값을 재사용한다.

즉 다음과 같은 흐름이다.

```text
첫 접근
→ 초기화 로직 실행
→ 결과 저장

이후 접근
→ 저장된 값 반환
```

`lazy`는 일반적으로 `val`과 함께 사용한다.

```kotlin
val value by lazy {
    createValue()
}
```

기본 `lazy` 구현은 스레드 세이프하게 동작한다.

여러 스레드가 동시에 처음 접근하더라도 기본 설정에서는 초기화 로직이 한 번만 수행되도록 보장한다.

필요하다면 동작 방식을 변경할 수도 있다.

```kotlin
val value by lazy(LazyThreadSafetyMode.NONE) {
    createValue()
}
```

단일 스레드 환경처럼 동기화가 필요 없는 경우에는 이런 옵션을 사용할 수도 있다.

`lateinit`과 `lazy`는 모두 초기화를 늦춘다는 공통점이 있지만 목적은 다르다.

`lateinit`은 다음과 같은 의미에 가깝다.

```text
지금은 값이 없지만 나중에 내가 직접 대입하겠다.
```

반면 `lazy`는 다음과 같다.

```text
지금은 계산하지 말고, 처음 필요한 순간에 이 로직을 실행해서 값을 만들어라.
```

예를 들어:

```kotlin
lateinit var name: String

val config by lazy {
    loadConfig()
}
```

`name`은 외부에서 직접 값을 넣어야 하고, `config`는 처음 접근하는 순간 `loadConfig()`가 실행되어 자동으로 초기화된다.

---

## 정리

- Primary constructor의 `constructor` 키워드는 일반적으로 생략할 수 있다.
- 생성자 파라미터에 `val` 또는 `var`를 붙이면 프로퍼티가 된다.
- `val` 또는 `var`가 없는 생성자 파라미터는 프로퍼티가 아니며 프로퍼티 초기화식과 `init` 블록에서만 사용할 수 있다.
- `init` 블록과 프로퍼티 초기화식은 클래스에 작성된 순서대로 실행된다.
- Primary constructor가 존재하는 경우 secondary constructor는 직접 또는 간접적으로 primary constructor에 위임해야 한다.
- Kotlin의 클래스와 메서드는 기본적으로 `final`이며, 상속과 오버라이드를 허용하려면 `open`을 명시해야 한다.
- Kotlin의 기본 접근 제한자는 `public`이며, `internal`을 통해 같은 모듈 내부로 접근을 제한할 수 있다.
- Kotlin의 프로퍼티는 단순한 필드가 아니라 getter와 setter를 포함하는 언어 수준의 개념이다.
- 모든 프로퍼티가 backing field를 가지는 것은 아니며, custom getter만 사용하는 계산 프로퍼티는 값을 별도로 저장하지 않을 수도 있다.
- Custom setter에서 실제 저장 공간에 접근할 때는 `field`를 사용한다.
- `lateinit`은 값을 나중에 직접 대입해야 하는 non-null `var` 프로퍼티에 사용한다.
- `lazy`는 값이 처음 필요한 순간 초기화 로직을 실행하고 결과를 저장해 이후 재사용한다.
- `lateinit`과 `lazy`는 모두 초기화를 늦추지만 목적과 동작 방식은 서로 다르다.
