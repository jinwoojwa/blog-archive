---
title: 제네릭과 Variance - out과 in
---

# 제네릭과 Variance: out과 in

## 기본 제네릭

```kotlin
class Box<T>(val value: T)

val box = Box("Hello")
```

함수에도 사용할 수 있다.

```kotlin
fun <T> first(list: List<T>): T {
    return list[0]
}
```

Kotlin의 제네릭도 JVM 위에서 동작하기 때문에 Java와 마찬가지로 컴파일 시점 이후에는 타입 정보가 지워지는 타입 소거(type erasure)의 영향을 받는다. 다만 함수를 `inline`으로 선언하고 타입 파라미터에 `reified`를 붙이면, 런타임에도 `T`의 실제 타입에 접근할 수 있는 예외적인 방법이 있다. 이 부분은 그 자체로 별도의 주제이므로, 여기서는 "일반적인 제네릭 타입 파라미터는 런타임에 소거된다"는 점만 기억해두면 충분하다.

## 타입 제한

```kotlin
fun <T : Number> double(value: T) {
    println(value.toDouble() * 2)
}
```

`T : Number`는 `T`가 반드시 `Number`(또는 그 하위 타입)여야 한다는 제약이다. 이렇게 상한을 지정하면 함수 내부에서 `T` 타입의 값에 대해 `Number`가 제공하는 메서드(`toDouble()` 등)를 안전하게 호출할 수 있다. 제약이 두 개 이상 필요하다면 `where` 절을 사용한다.

```kotlin
fun <T> process(value: T) where T : Comparable<T>, T : CharSequence {
    // value는 Comparable이면서 동시에 CharSequence인 타입이어야 한다
}
```

## out

`out T`는 주로 값을 생산하는 타입에 사용한다.

```kotlin
interface Producer<out T> {
    fun produce(): T
}
```

Java의 `? extends T`와 연결해서 이해할 수 있다. `out`을 붙이면 그 타입은 공변(covariant)이 된다. 즉 `Dog`가 `Animal`의 하위 타입이라면 `Producer<Dog>`도 `Producer<Animal>`의 하위 타입으로 취급된다. 이렇게 안전하게 동작하려면 `T`가 오직 반환 타입(out 위치)에만 등장해야 하며, 만약 `T`를 파라미터 타입으로 사용하려고 하면 컴파일 에러가 발생한다. `Producer<out T>`가 값을 밖으로 "생산"하기만 하고 안으로 받아들이지는 않기 때문에 안전한 것이다.

## in

`in T`는 주로 값을 소비하는 타입에 사용한다.

```kotlin
interface Consumer<in T> {
    fun consume(value: T)
}
```

Java의 `? super T`와 연결해서 이해할 수 있다. `in`을 붙이면 반공변(contravariant)이 되어, 관계가 반대로 뒤집힌다. 즉 `Animal`이 `Dog`의 상위 타입이라면 `Consumer<Animal>`이 오히려 `Consumer<Dog>`의 하위 타입으로 취급된다("Animal을 소비할 수 있다면, 당연히 Dog도 소비할 수 있다"는 논리다). 이 경우 `T`는 오직 파라미터 타입(in 위치)에만 등장할 수 있고, 반환 타입으로 사용하면 컴파일 에러가 발생한다.

## PECS와 비교

Java의 `Producer Extends, Consumer Super`라는 PECS 원칙과 Kotlin의 `out`, `in`은 밀접한 관계가 있다. Java에서는 `List<? extends Animal>`, `List<? super Dog>`처럼 와일드카드를 호출하는 쪽(use-site)에서 그때그때 붙여야 하지만, Kotlin은 declaration-site variance를 지원하기 때문에 타입을 선언하는 시점에 아예 `out`, `in`을 못박아 둘 수 있다는 특징이 있다. 덕분에 `Producer<out T>`처럼 한 번 선언해두면, 이 타입을 사용하는 모든 곳에서 매번 와일드카드를 다시 적어줄 필요가 없다.

물론 Kotlin도 필요하다면 사용하는 시점에 variance를 지정하는 use-site variance(`Box<out Any>`처럼)를 지원하지만, 애초에 타입 자체의 성격이 "생산 전용"이거나 "소비 전용"이라면 declaration-site에서 `out`/`in`을 붙여 의도를 명확히 드러내는 편이 더 Kotlin다운 방식이다.

## 정리

- 제네릭에서는 단순히 `<T>` 문법보다 **타입이 값을 생산하는가, 소비하는가**를 생각하는 것이 variance 이해의 핵심이다.
- `out T`는 공변(반환 위치에만 사용 가능), `in T`는 반공변(파라미터 위치에만 사용 가능)이라는 위치 제약을 기억해두면 컴파일 에러의 원인을 빠르게 파악할 수 있다.
- Kotlin은 declaration-site variance를 지원해, Java처럼 사용하는 쪽에서 매번 와일드카드를 붙이지 않고도 타입 선언 자체에 variance 의도를 담을 수 있다.
