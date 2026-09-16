---
title: 컬렉션 다루기 - List, Set, Map
---

# 컬렉션 다루기: List, Set, Map

## 기본 컬렉션

Kotlin의 대표적인 컬렉션은 `List`, `Set`, `Map`이다.

- `List`: 순서가 있고 중복을 허용한다.
- `Set`: 중복을 허용하지 않는다.
- `Map`: Key-Value 형태로 데이터를 저장한다.

```kotlin
val numbers = listOf(1, 2, 3)
val unique = setOf(1, 2, 3)
val scores = mapOf(
    "Kim" to 100,
    "Lee" to 90
)
```

Kotlin에서는 `listOf`, `setOf`, `mapOf`와 같은 팩토리 함수를 이용해 컬렉션을 생성하는 것이 일반적이다.

변경 가능한 컬렉션이 필요하다면 `mutableListOf`, `mutableSetOf`, `mutableMapOf`를 사용할 수 있다.

```kotlin
val numbers = mutableListOf(1, 2, 3)
val unique = mutableSetOf(1, 2, 3)
val scores = mutableMapOf(
    "Kim" to 100,
    "Lee" to 90
)

numbers.add(4)
unique.add(4)
scores["Park"] = 80
```

Kotlin/JVM의 컬렉션 타입은 JVM의 Java 컬렉션과 밀접하게 연결되어 있으며, 실제 실행 시 Java 컬렉션 구현체가 사용되는 경우가 많다.

따라서 완전히 별개의 컬렉션 체계를 새롭게 제공한다기보다는, Kotlin/JVM 관점에서는 Java 컬렉션과 자연스럽게 상호 운용하면서 읽기 전용과 변경 가능한 컬렉션을 타입 수준에서 구분할 수 있도록 API를 제공한다고 이해할 수 있다.

---

## 읽기 전용과 Mutable

Kotlin은 컬렉션 인터페이스에서 읽기 전용과 변경 가능한 타입을 구분한다.

```kotlin
val a: List<Int> = listOf(1, 2, 3)

val b: MutableList<Int> = mutableListOf(1, 2, 3)

b.add(4)
```

`Set`과 `Map`도 마찬가지다.

```kotlin
List<T>
MutableList<T>

Set<T>
MutableSet<T>

Map<K, V>
MutableMap<K, V>
```

읽기 전용 타입에는 컬렉션을 변경하는 API가 노출되지 않는다.

```kotlin
val numbers: List<Int> = listOf(1, 2, 3)

// numbers.add(4) // 컴파일 에러
```

반면 `MutableList`에서는 데이터를 추가하거나 제거할 수 있다.

```kotlin
val numbers: MutableList<Int> = mutableListOf(1, 2, 3)

numbers.add(4)
numbers.remove(1)
```

이렇게 인터페이스 단계에서 변경 가능 여부를 구분하면 함수 시그니처만으로도 해당 함수가 컬렉션을 어떻게 사용할 것인지에 대한 의도를 어느 정도 표현할 수 있다.

예를 들어:

```kotlin
fun printNumbers(numbers: List<Int>) {
    for (number in numbers) {
        println(number)
    }
}
```

파라미터를 `List<Int>`로 선언하면 함수 내부에서는 해당 참조를 통해 컬렉션을 변경할 수 없다.

반대로 컬렉션을 실제로 변경해야 하는 함수라면 `MutableList`를 받을 수 있다.

```kotlin
fun addNumber(numbers: MutableList<Int>) {
    numbers.add(100)
}
```

다만 `List`를 곧바로 완전한 immutable 객체라고 이해하면 안 된다.

Kotlin의 `List`는 수정 API를 노출하지 않는 **read-only interface**다.

따라서 `List` 타입의 참조로는 수정할 수 없더라도 실제 객체가 `MutableList`이고 다른 곳에서 해당 객체를 변경한다면 `List`에서도 변경된 결과가 보일 수 있다.

```kotlin
val mutable = mutableListOf(1, 2, 3)

val readOnlyView: List<Int> = mutable

mutable.add(4)

println(readOnlyView)
// [1, 2, 3, 4]
```

`readOnlyView`에서는 `add()`를 호출할 수 없다.

```kotlin
// readOnlyView.add(5) // 컴파일 에러
```

하지만 `mutable`과 `readOnlyView`가 같은 객체를 가리키고 있기 때문에 `mutable`을 통해 객체를 변경하면 `readOnlyView`에서도 변경된 결과가 보인다.

따라서 다음 두 개념을 구분해야 한다.

```text
read-only
→ 이 참조를 통해 수정할 수 없음

immutable
→ 객체 자체의 상태가 변경되지 않음
```

Kotlin의 기본 `List` 인터페이스가 직접 보장하는 것은 전자에 가깝다.

---

## val과 불변성

컬렉션의 변경 가능성과 변수의 `val`/`var`도 서로 다른 개념이다.

```kotlin
val numbers = mutableListOf(1, 2, 3)

numbers.add(4) // 가능
```

`val`은 참조의 재할당을 금지할 뿐, 참조하고 있는 객체 내부의 변경까지 막는 것은 아니다.

따라서 다음 코드는 가능하다.

```kotlin
val numbers = mutableListOf(1, 2, 3)

numbers.add(4)
numbers.remove(1)
numbers[0] = 100
```

하지만 새로운 객체를 다시 대입할 수는 없다.

```kotlin
val numbers = mutableListOf(1, 2, 3)

// numbers = mutableListOf(5, 6) // 컴파일 에러
```

즉 다음과 같이 기억할 수 있다.

```text
val != immutable
```

Java의 `final` 참조도 비슷한 방식으로 동작한다.

```java
final List<Integer> numbers = new ArrayList<>();

numbers.add(1);           // 가능
// numbers = new ArrayList<>(); // 불가능
```

Kotlin에서는 `val`을 자주 사용하기 때문에 참조의 재할당 가능 여부와 객체 자체의 변경 가능 여부를 구분하는 것이 중요하다.

예를 들어:

```kotlin
val numbers: MutableList<Int> = mutableListOf(1, 2, 3)
```

이 코드는 다음과 같은 의미다.

```text
numbers가 다른 객체를 가리키도록 변경 → 불가능
numbers가 가리키는 MutableList 수정 → 가능
```

반면:

```kotlin
val numbers: List<Int> = listOf(1, 2, 3)
```

에서는 `numbers`를 다른 객체로 재할당할 수 없고, `List` 인터페이스를 통해 컬렉션을 수정하는 API도 사용할 수 없다.

다만 앞서 살펴봤듯이 `List` 자체가 객체의 완전한 불변성까지 보장하는 것은 아니다.

외부에서 전달받은 mutable 컬렉션과의 연결을 끊고 별도의 읽기 전용 리스트를 만들고 싶다면 `toList()`를 이용한 복사를 고려할 수 있다.

```kotlin
val mutable = mutableListOf(1, 2, 3)

val copied: List<Int> = mutable.toList()

mutable.add(4)

println(mutable)
// [1, 2, 3, 4]

println(copied)
// [1, 2, 3]
```

이 경우 `copied`는 `mutable`과 별도의 리스트이므로 이후 `mutable`의 변경에 영향을 받지 않는다.

---

## Map

`Map`은 Key와 Value의 쌍으로 데이터를 저장한다.

```kotlin
val scores = mapOf(
    "Kim" to 100,
    "Lee" to 90
)
```

여기서 사용하는 `to`는 두 값을 `Pair`로 만들어주는 함수다.

```kotlin
val pair = "Kim" to 100

println(pair.first)
// Kim

println(pair.second)
// 100
```

따라서 다음 코드는 여러 `Pair`를 이용해 `Map`을 생성한다고 이해할 수 있다.

```kotlin
val scores = mapOf(
    "Kim" to 100,
    "Lee" to 90
)
```

Map의 값은 `[]`를 이용해 조회할 수 있다.

```kotlin
val score = scores["Kim"]
```

이때 `score`의 타입은 `Int`가 아니라 `Int?`다.

```kotlin
val score: Int? = scores["Kim"]
```

Key가 존재하지 않을 수 있기 때문이다.

```kotlin
val score = scores["Park"]

println(score)
// null
```

즉 `Map.get()` 또는 `[]` 조회 결과는 기본적으로 nullable이다.

---

## getValue

해당 Key가 반드시 존재해야 하는 상황이라면 `getValue()`를 사용할 수 있다.

```kotlin
val scores = mapOf(
    "Kim" to 100
)

val score: Int = scores.getValue("Kim")
```

`getValue()`는 값이 존재한다면 non-null 타입으로 값을 얻을 수 있다는 장점이 있다.

하지만 Key가 존재하지 않으면 `NoSuchElementException`이 발생한다.

```kotlin
val score = scores.getValue("Park")
```

따라서 `getValue()`는 해당 Key가 반드시 존재한다는 것이 프로그램의 전제인 경우 사용하는 것이 적절하다.

---

## getOrDefault와 getOrElse

Key가 존재하지 않을 때 예외를 발생시키는 대신 기본값을 사용할 수도 있다.

```kotlin
val scores = mapOf(
    "Kim" to 100
)

val score = scores.getOrDefault("Park", 0)

println(score)
// 0
```

`getOrElse()`를 사용할 수도 있다.

```kotlin
val score = scores.getOrElse("Park") {
    0
}
```

`getOrElse()`는 기본값을 람다로 전달한다.

따라서 기본값을 만드는 데 별도의 연산이 필요한 경우 실제로 기본값이 필요한 시점에 해당 로직을 실행할 수 있다.

```kotlin
val score = scores.getOrElse("Park") {
    println("create default value")
    calculateDefaultScore()
}
```

Key가 존재한다면 기본값을 만드는 람다는 실행되지 않는다.

따라서 기본값 생성 비용이 큰 경우 유용할 수 있다.

Map의 값을 가져오는 방법은 상황에 따라 다음처럼 구분할 수 있다.

```text
scores["Kim"]
→ 없으면 null

scores.getValue("Kim")
→ 반드시 존재해야 함
→ 없으면 예외

scores.getOrDefault("Kim", 0)
→ 없으면 지정한 기본값

scores.getOrElse("Kim") { 0 }
→ 없으면 람다를 실행하여 기본값 생성
```

---

## Map의 변경

변경 가능한 Map은 `mutableMapOf()`를 이용해 만들 수 있다.

```kotlin
val scores = mutableMapOf(
    "Kim" to 100,
    "Lee" to 90
)
```

값을 추가하거나 변경할 때는 `[]` 문법을 사용할 수 있다.

```kotlin
scores["Park"] = 80
scores["Kim"] = 95
```

삭제할 수도 있다.

```kotlin
scores.remove("Lee")
```

Java에서는 일반적으로 다음과 같이 작성한다.

```java
map.put("Park", 80);
```

Kotlin에서도 `put()`을 사용할 수 있지만:

```kotlin
scores.put("Park", 80)
```

일반적인 값 대입에서는 다음과 같은 형태를 자연스럽게 사용할 수 있다.

```kotlin
scores["Park"] = 80
```

---

## 순회

컬렉션은 `for`를 이용해 순회할 수 있다.

```kotlin
val numbers = listOf(10, 20, 30)

for (number in numbers) {
    println(number)
}
```

인덱스와 값을 함께 사용하고 싶다면 `withIndex()`를 사용할 수 있다.

```kotlin
for ((index, value) in numbers.withIndex()) {
    println("$index: $value")
}
```

Map도 Key와 Value를 동시에 꺼내면서 순회할 수 있다.

```kotlin
val scores = mapOf(
    "Kim" to 100,
    "Lee" to 90
)

for ((name, score) in scores) {
    println("$name: $score")
}
```

여기서:

```kotlin
(index, value)
(name, score)
```

처럼 하나의 값을 여러 변수로 나누어 받는 문법을 **구조 분해 선언(destructuring declaration)**이라고 한다.

`withIndex()`로 순회하면 각각의 원소는 `IndexedValue<T>`이며, `index`와 `value`를 가지고 있다.

개념적으로 다음과 같은 데이터를 순회한다고 생각할 수 있다.

```text
IndexedValue(index=0, value=10)
IndexedValue(index=1, value=20)
IndexedValue(index=2, value=30)
```

이를 다음과 같이 구조 분해한다.

```kotlin
for ((index, value) in numbers.withIndex()) {
    // ...
}
```

Map을 순회할 때 각각의 원소는 `Map.Entry<K, V>`다.

```text
Entry(key="Kim", value=100)
Entry(key="Lee", value=90)
```

이를 다음처럼 구조 분해할 수 있다.

```kotlin
for ((name, score) in scores) {
    println("$name: $score")
}
```

이러한 구조 분해가 가능한 이유는 해당 타입에 `component1()`, `component2()`와 같은 관례적인 함수가 존재하기 때문이다.

개념적으로:

```kotlin
val index = indexedValue.component1()
val value = indexedValue.component2()
```

와 같은 작업을:

```kotlin
val (index, value) = indexedValue
```

처럼 간결하게 표현할 수 있는 것이다.

이 방식은 이후 배우게 될 `data class`에서도 동일하게 활용된다.

```kotlin
data class User(
    val name: String,
    val age: Int
)

val user = User("Kim", 20)

val (name, age) = user
```

`data class`는 프로퍼티를 기반으로 `componentN()` 함수를 자동으로 생성하기 때문에 구조 분해 선언을 사용할 수 있다.

---

## 정리

- Kotlin의 대표적인 컬렉션에는 `List`, `Set`, `Map`이 있다.
- `listOf`, `setOf`, `mapOf`를 이용해 읽기 전용 컬렉션을 만들 수 있다.
- `mutableListOf`, `mutableSetOf`, `mutableMapOf`를 이용해 변경 가능한 컬렉션을 만들 수 있다.
- Kotlin은 `List`와 `MutableList`처럼 읽기 전용 인터페이스와 변경 가능한 인터페이스를 구분한다.
- `List`는 해당 참조를 통한 수정을 막는 read-only 인터페이스이며, 객체 자체의 완전한 불변성을 의미하지는 않는다.
- `val`은 참조의 재할당을 막는 것이며 객체 내부의 변경까지 막는 것은 아니다.
- 따라서 `val != immutable`이라는 점에 유의한다.
- 다른 mutable 컬렉션과의 연결을 끊고 별도의 리스트가 필요하다면 `toList()` 등을 이용한 복사를 고려할 수 있다.
- `Map.get()` 또는 `[]`의 결과는 Key가 존재하지 않을 수 있기 때문에 nullable이다.
- Key가 반드시 존재해야 한다면 `getValue()`, 기본값이 필요하다면 `getOrDefault()` 또는 `getOrElse()`를 사용할 수 있다.
- `getOrElse()`는 기본값을 람다로 전달하기 때문에 실제 기본값이 필요한 경우에만 해당 로직을 실행할 수 있다.
- `for ((a, b) in ...)` 형태의 문법은 구조 분해 선언이며 `componentN()` 관례를 기반으로 동작한다.
