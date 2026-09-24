---
title: 함수를 더 유연하게 쓰는 법 - Default/Named Argument와 vararg
---

# 함수를 더 유연하게 쓰는 법: Default/Named Argument와 vararg

## Default Argument

```kotlin
fun greet(name: String, message: String = "Hello") {
    println("$message $name")
}

greet("Kim")
greet("Kim", "Hi")
```

Java에서 흔히 사용하는 오버로딩의 일부를 기본 인자로 대체할 수 있다. Java에서는 파라미터 조합마다 메서드를 여러 개 만들어야 했지만(`greet(name)`, `greet(name, message)`), Kotlin은 함수 하나에 기본값만 지정하면 되므로 중복 코드를 줄일 수 있다.

기본값이 있는 파라미터는 보통 뒤쪽에 두는 것이 자연스럽다. 기본값이 없는 파라미터보다 앞에 오면, 뒤의 인자를 전달할 때 이름을 명시해야 하기 때문이다.

```kotlin
fun greet(message: String = "Hello", name: String) {
    println("$message $name")
}

greet(name = "Kim") // message는 생략, name은 이름을 명시해야 함
```

## Named Argument

```kotlin
fun createUser(name: String, age: Int, active: Boolean) {}

createUser(
    name = "Kim",
    age = 20,
    active = true
)
```

인자의 의미가 명확해진다는 장점이 있다. 특히 `Boolean`이나 `Int`처럼 타입만 봐서는 어떤 의미인지 알기 어려운 인자가 여러 개 나열될 때(`createUser("Kim", 20, true)`), named argument를 쓰면 호출부만 보고도 어떤 값이 무엇을 의미하는지 바로 알 수 있다. Default argument와 함께 사용하면, 앞쪽 인자를 생략하고 뒤쪽 인자만 이름으로 지정하는 것도 가능하다.

```kotlin
createUser(name = "Kim", active = true, age = 20) // 순서를 바꿔도 무방하다
```

## vararg

```kotlin
fun printAll(vararg values: String) {
    for (value in values) {
        println(value)
    }
}

printAll("A", "B", "C")
```

`vararg`로 선언된 파라미터는 함수 내부에서 배열(`Array<String>`)처럼 취급된다. Java의 가변 인자(`String... values`)와 동일한 역할을 하지만, 한 함수에 `vararg` 파라미터는 하나만 둘 수 있고 보통 마지막 위치에 선언한다.

배열을 전달하려면 spread operator `*`를 사용할 수 있다.

```kotlin
val values = arrayOf("A", "B")
printAll(*values)
```

이미 가지고 있는 배열을 개별 인자로 펼쳐서 전달한다는 의미에서 spread operator라고 부른다. 만약 `List<String>`처럼 배열이 아닌 컬렉션을 가지고 있다면, `toTypedArray()`로 배열로 변환한 뒤 `*`를 붙여야 한다.

```kotlin
val list = listOf("A", "B")
printAll(*list.toTypedArray())
```

## Local Function

함수 내부에 함수를 정의할 수도 있다.

```kotlin
fun validateUser(name: String, age: Int) {
    fun validate(value: Boolean, message: String) {
        require(value) { message }
    }

    validate(name.isNotBlank(), "name is blank")
    validate(age >= 0, "invalid age")
}
```

Local function은 자신을 감싸고 있는 바깥 함수의 파라미터와 지역 변수에 자유롭게 접근할 수 있다(클로저). 위 예시에서는 `validate`가 별도의 인자 없이도 바깥 함수의 문맥 안에서 동작한다. 이렇게 하면 특정 함수 안에서만 의미가 있는 보조 로직을, 클래스 멤버나 최상위 함수로 노출하지 않고 그 함수 내부로 감춰둘 수 있다.

## infix

멤버 함수 또는 확장 함수에 `infix`를 붙이면 점과 괄호를 생략할 수 있다.

```kotlin
infix fun Int.plusValue(other: Int) = this + other

val result = 10 plusValue 20
```

`"A" to 1`에서 사용하는 `to`도 대표적인 infix 함수다. 다만 아무 함수에나 `infix`를 붙일 수 있는 것은 아니다. 파라미터가 정확히 하나여야 하고, 그 파라미터에는 기본값을 줄 수 없으며 `vararg`로 선언할 수도 없다. 또한 멤버 함수이거나 확장 함수여야 한다는 제약도 있다. 가독성이 좋아지는 대표적인 경우(`1 to "one"`, `a shl 2`)가 아니라면, 남용했을 때 오히려 코드의 의미를 파악하기 어려워질 수 있으므로 신중하게 사용하는 것이 좋다.

## 정리

- Default/named argument를 이용하면 Java의 오버로딩 없이도 유연한 함수 시그니처를 만들 수 있다.
- `vararg`는 내부적으로 배열이며, 이미 가지고 있는 배열/컬렉션을 전달할 때는 spread operator(`*`)가 필요하다.
- Local function은 클로저를 통해 바깥 함수의 문맥을 그대로 활용할 수 있는 보조 함수다.
- `infix`는 파라미터가 하나인 멤버/확장 함수에만 붙일 수 있으며, 가독성이 분명히 좋아지는 경우에만 사용하는 것이 좋다.
