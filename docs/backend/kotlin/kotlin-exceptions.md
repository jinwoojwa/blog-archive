---
title: Kotlin에서 예외 다루기
---

## try / catch

```kotlin
try {
    riskyOperation()
} catch (e: IllegalArgumentException) {
    println(e.message)
} finally {
    println("done")
}
```

기본적인 형태는 Java와 크게 다르지 않다. `catch` 블록은 여러 개를 이어서 선언할 수 있으며, 이때는 더 구체적인(상속 계층에서 하위에 있는) 예외 타입을 먼저 선언해야 한다. 그렇지 않으면 상위 타입의 `catch` 블록이 먼저 매칭되어, 그 아래에 있는 더 구체적인 `catch` 블록은 절대 실행되지 않는다. `finally` 블록은 예외 발생 여부와 관계없이 항상 실행된다.

## try는 Expression

Kotlin의 `try`는 값을 반환할 수 있다.

```kotlin
val number = try {
    text.toInt()
} catch (e: NumberFormatException) {
    0
}
```

`try`가 값으로 평가되려면, `try` 블록과 각 `catch` 블록의 마지막 표현식이 공통된 타입(또는 그 타입들의 공통 상위 타입)으로 취급될 수 있어야 한다. 위 예시에서는 `text.toInt()`와 `0`이 모두 `Int`이므로 `number`의 타입이 `Int`로 정해진다. 만약 `catch` 블록에서 값을 만들어내지 못하고 다른 예외를 던지거나 함수를 반환해버린다면, 그 분기는 값 계산에서 제외되고 정상적으로 끝난 분기의 타입만으로 전체 타입이 결정된다.

## Checked Exception

Kotlin에서는 Java처럼 checked exception을 컴파일러가 강제하지 않는다.

따라서 `throws` 선언이나 반드시 필요한 `try-catch`를 언어 차원에서 요구하지 않는다. 다만 Kotlin으로 작성한 함수를 Java 코드에서 호출해야 한다면, `@Throws(IOException::class)`처럼 애너테이션을 붙여 Java 쪽에는 checked exception 정보를 그대로 노출할 수 있다. 즉 Kotlin 자체는 checked exception 개념이 없지만, Java와의 상호운용을 위한 탈출구는 마련되어 있는 셈이다.

## require

함수 인자가 조건을 만족해야 할 때 유용하다.

```kotlin
fun setAge(age: Int) {
    require(age >= 0) {
        "age must be positive"
    }
}
```

조건이 거짓이면 `IllegalArgumentException`이 발생한다. 메시지를 문자열 대신 람다(`{ "age must be positive" }`)로 전달하는 이유는, 이 메시지를 실제로 예외가 발생할 때만 평가하기 위해서다. 조건이 참이라면 메시지 문자열을 만드는 데 드는 비용 자체가 발생하지 않는다.

## check

객체나 프로그램의 현재 상태를 검증할 때 사용할 수 있다.

```kotlin
check(isInitialized) {
    "not initialized"
}
```

`require`가 "전달받은 인자가 잘못되었다"(`IllegalArgumentException`)는 의미라면, `check`는 "현재 객체나 프로그램의 상태가 기대와 다르다"(`IllegalStateException`)는 의미다. 두 함수 모두 조건이 거짓일 때만 예외를 던지고, 메시지를 지연 평가한다는 점은 동일하다.

## error

정상적으로 도달해서는 안 되는 상황 등을 표현할 수 있다.

```kotlin
val value = when (type) {
    "A" -> 1
    "B" -> 2
    else -> error("Unknown type: $type")
}
```

`error()`는 항상 `IllegalStateException`을 던지며, 반환 타입이 `Nothing`이다. `Nothing`은 "정상적으로 끝나지 않는 함수"를 나타내는 특별한 타입이기 때문에, 위 예시처럼 `when`의 다른 분기들이 `Int`를 반환하는 상황에서도 `error(...)`가 있는 분기를 함께 둘 수 있다. 컴파일러 입장에서는 `Nothing`이 모든 타입의 하위 타입이므로, 전체 `when` 식의 타입을 `Int`로 추론하는 데 문제가 없다.

## runCatching

예외가 발생할 수 있는 연산을 `Result`로 감쌀 수 있다.

```kotlin
val result = runCatching {
    text.toInt()
}

result
    .onSuccess { println(it) }
    .onFailure { println(it.message) }
```

`runCatching`은 성공 값 또는 예외를 감싼 `Result<T>`를 반환한다. `onSuccess`/`onFailure`로 후속 처리를 이어 붙일 수도 있고, `getOrNull()`로 실패 시 `null`을 받거나, `getOrElse { ... }`로 실패 시 기본값을 계산하거나, `getOrThrow()`로 감싸져 있던 예외를 다시 던질 수도 있다. `runCatching`이 모든 예외 처리의 대체재는 아니다. 예외를 어디서 처리하고 어떤 실패를 복구할 것인지가 먼저 설계되어야 하며, 특히 코루틴 환경에서는 `CancellationException`까지 함께 잡아버릴 수 있어 취소 동작을 방해하지 않도록 주의가 필요하다.

## 정리

- Kotlin의 `try`는 expression이며, 각 분기의 마지막 표현식이 공통 타입으로 귀결되어야 값을 반환할 수 있다.
- checked exception을 강제하지 않지만, `@Throws`로 Java 상호운용을 위한 정보는 남길 수 있다.
- 입력 조건에는 `require`(`IllegalArgumentException`), 상태 조건에는 `check`(`IllegalStateException`)를 사용하며, 도달해서는 안 되는 분기에는 `Nothing`을 반환하는 `error`를 사용할 수 있다.
- `runCatching`은 실패를 `Result` 값으로 다룰 때 유용하지만, 코루틴의 취소 예외까지 함께 삼킬 수 있다는 점을 포함해 남용하지 않는 것이 좋다.
