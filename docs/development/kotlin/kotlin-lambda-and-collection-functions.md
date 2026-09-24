---
title: 람다와 컬렉션 함수로 표현하는 데이터 변환
---

# 람다와 컬렉션 함수로 표현하는 데이터 변환

## 람다

람다는 함수를 값처럼 표현하는 방법이다.

```kotlin
val add = { a: Int, b: Int -> a + b }

println(add(1, 2))
```

타입을 명시하면 다음과 같다.

```kotlin
val add: (Int, Int) -> Int = { a, b -> a + b }
```

`(Int, Int) -> Int`는 두 개의 `Int`를 받아 `Int`를 반환하는 함수 타입이다. 람다 본문에서 마지막 줄에 오는 표현식이 곧 반환값이 되며, 별도의 `return` 키워드는 필요하지 않다. 또한 람다는 자신이 정의된 위치의 지역 변수를 캡처할 수 있어서, 바깥 변수를 참조하는 작은 단위의 동작을 그 자리에서 바로 만들어 전달할 수 있다.

```kotlin
var count = 0
val increment = { count++ }

increment()
increment()
println(count) // 2
```

## it

매개변수가 하나라면 기본 이름 `it`을 사용할 수 있다.

```kotlin
val double: (Int) -> Int = { it * 2 }
```

`it`은 람다가 파라미터를 하나만 가질 때만 자동으로 제공되는 암묵적인 이름이다. 람다 안에 또 다른 람다가 중첩되면 바깥쪽 `it`과 안쪽 `it`이 헷갈릴 수 있으므로, 이런 경우에는 `{ outer -> ... { inner -> ... } }`처럼 파라미터 이름을 명시적으로 지어주는 것이 가독성 측면에서 더 낫다.

## 고차 함수

함수를 인자로 받거나 함수를 반환하는 함수를 고차 함수라고 한다.

```kotlin
fun calculate(
    a: Int,
    b: Int,
    operation: (Int, Int) -> Int
): Int {
    return operation(a, b)
}

val result = calculate(10, 20) { a, b ->
    a + b
}
```

여기서 `operation: (Int, Int) -> Int`처럼 함수 타입을 파라미터 타입 자리에 그대로 선언할 수 있다는 점이 핵심이다. 덕분에 "무엇을 계산할지"를 함수 바깥에서 주입할 수 있고, `calculate` 자체는 계산 로직을 몰라도 된다. 함수 타입은 `((Int, Int) -> Int)?`처럼 nullable로도 선언할 수 있으며, 이 경우 호출 전에 `null` 체크가 필요하다.

## Trailing Lambda

마지막 인자가 함수라면 람다를 괄호 밖으로 뺄 수 있다.

```kotlin
calculate(10, 20) { a, b -> a + b }
```

Kotlin 컬렉션 API에서 매우 자주 등장하는 문법이다. 심지어 함수의 인자가 람다 하나뿐이라면 괄호 자체를 완전히 생략할 수도 있다(`repeat(3) { println(it) }`). 이 문법 덕분에 람다를 인자로 받는 함수 호출이 마치 언어에 내장된 제어 구문처럼 자연스럽게 읽힌다.

## 함수 참조

이미 존재하는 함수를 값으로 전달할 수도 있다.

```kotlin
fun double(value: Int) = value * 2

val function = ::double
println(function(10))
```

`::` 뒤에 함수 이름을 적으면 그 함수를 가리키는 참조를 만들 수 있다. 최상위 함수뿐 아니라 `ClassName::methodName` 형태로 특정 클래스의 멤버 함수를, `::ClassName` 형태로 생성자를 참조할 수도 있다. 매번 `{ value -> double(value) }`처럼 감싸는 람다를 작성하는 대신, 이미 있는 함수를 그대로 재사용하고 싶을 때 유용하다.

람다와 함수 타입을 이해했다면, Kotlin 컬렉션 API가 왜 이런 형태로 설계됐는지도 자연스럽게 이해할 수 있다. Kotlin에서는 컬렉션을 직접 반복하면서 결과를 수정하기보다, 아래처럼 고차 함수를 이용해 변환하는 패턴을 자주 사용한다. 이런 함수들은 대부분 원본 컬렉션을 바꾸지 않고 새로운 컬렉션을 반환하므로, 여러 함수를 이어 붙여도(`filter().map()`) 원본은 그대로 유지된다.

## filter

조건에 맞는 원소만 선택한다.

```kotlin
val result = listOf(1, 2, 3, 4)
    .filter { it % 2 == 0 }
```

결과는 `[2, 4]`다. 원본 리스트는 변경되지 않고, 조건을 만족하는 원소만 담긴 새로운 리스트가 반환된다.

## map

각 원소를 다른 값으로 변환한다.

```kotlin
val result = listOf(1, 2, 3)
    .map { it * 10 }
```

`filter`가 원소의 개수를 줄이는 연산이라면, `map`은 원소의 개수는 그대로 두고 각 값을 다른 값(다른 타입이어도 무방하다)으로 바꾸는 연산이다.

## flatMap

각 원소에서 컬렉션을 만든 뒤 하나의 컬렉션으로 평탄화한다.

```kotlin
val result = listOf("AB", "CD")
    .flatMap { it.toList() }
```

만약 단순히 `map { it.toList() }`만 사용했다면 결과는 `List<List<Char>>`, 즉 리스트의 리스트가 된다. `flatMap`은 여기서 한 단계 더 나아가 안쪽 리스트들을 하나로 합쳐 `List<Char>`로 평탄화해 준다는 점이 `map`과의 차이다.

## firstOrNull

```kotlin
val value = numbers.firstOrNull { it > 10 }
```

조건을 만족하는 값이 없다면 `null`을 반환한다. Kotlin의 Null Safety와 잘 결합되는 API다. 조건을 만족하는 값이 반드시 있다고 가정할 수 있다면 `first { ... }`를 사용할 수도 있지만, 이 경우 조건을 만족하는 값이 없으면 `NoSuchElementException`이 발생한다는 차이가 있다.

## any / all / none

```kotlin
numbers.any { it > 10 }
numbers.all { it > 0 }
numbers.none { it < 0 }
```

각각 하나라도 만족하는지, 모두 만족하는지, 아무것도 만족하지 않는지를 검사한다. 세 함수 모두 `Boolean`을 반환하며, 조건을 만족하는 원소를 찾는 즉시(또는 만족하지 않는 원소를 찾는 즉시) 나머지 원소는 검사하지 않고 결과를 반환한다.

## groupBy

```kotlin
val words = listOf("apple", "ant", "banana")

val grouped = words.groupBy { it.first() }
```

결과 타입은 `Map<Char, List<String>>` 형태가 된다. 즉 람다가 반환하는 값을 Key로 삼아, 같은 Key를 가진 원소들을 하나의 리스트로 묶어준다. 위 예시라면 `{ 'a': [apple, ant], 'b': [banana] }`와 같은 형태가 된다.

## associateBy

```kotlin
data class User(val id: Long, val name: String)

val byId = users.associateBy { it.id }
```

`List<User>`를 `Map<Long, User>` 형태로 변환할 때 유용하다. `groupBy`와 헷갈리기 쉬운데, `groupBy`는 같은 Key를 가진 원소를 리스트로 모으는 반면, `associateBy`는 각 Key에 원소를 하나씩만 대응시킨다. 만약 같은 Key를 가진 원소가 여러 개 있다면, 나중에 처리된 원소가 앞의 값을 덮어써서 최종적으로 마지막 원소만 남는다.

## fold

누적값을 이용해 하나의 결과를 만든다.

```kotlin
val sum = listOf(1, 2, 3)
    .fold(0) { acc, value -> acc + value }
```

`fold`는 초기값(위 예시에서는 `0`)을 명시적으로 지정하고, 그 값을 시작으로 각 원소를 순서대로 누적해 나간다. 비슷한 함수로 `reduce`가 있는데, `reduce`는 초기값 없이 컬렉션의 첫 번째 원소를 누적의 시작값으로 사용한다는 차이가 있다. 컬렉션이 비어 있을 수 있는 상황에서는 `reduce`가 예외를 던질 수 있으므로, 안전한 기본값이 있다면 `fold`를 사용하는 편이 낫다.

## 정리

- 람다와 함수 타입을 이해해야 `map`, `filter`, `fold` 같은 Kotlin API를 자연스럽게 이해할 수 있다.
- Trailing lambda와 함수 참조(`::`)는 고차 함수 호출을 더 간결하고 자연스럽게 만들어준다.
- 컬렉션 함수는 단순 암기보다 **입력 컬렉션이 어떤 출력 타입으로 변환되는지**를 생각하면서 학습하는 것이 중요하다.
- `groupBy`와 `associateBy`, `fold`와 `reduce`처럼 비슷해 보이지만 동작이 다른 함수들은 반환 타입과 예외 발생 조건을 기준으로 구분해서 기억해두는 것이 좋다.
