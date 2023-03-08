---
title: 코틀린 람다의 일급 함수 / 고차 함수에 대해
description: About first-class and higher-order functions in Kotlin lambdas
published: true
date: 2023-02-17T18:00:03.813Z
tags: kotlin, lambda
editor: markdown
dateCreated: 2022-12-22T10:17:44.669Z
---

- [About first-class/higher-order functions in Kotlin lambdas***English** version of this document is available*](/en/dev/Kotlin/About-first-class-and-higher-order-functions-in-Kotlin-lambdas)
{.links-list}

---

- Kotlin에서 함수는 일급 함수이므로 다른 값처럼 취급될 수 있습니다. 여기에는 변수에 저장하고, 다른 함수에 인수로 전달하고, 함수의 결과로 반환이 가능합니다. 다음은 람다 식을 인수로 사용하여 결과로 반환하는 Kotlin 함수의 예입니다.

```kotlin
fun makeAdder(x: Int): (Int) -> Int {
    return { y -> x + y }
}
```

- 이 예제에서 `makeAdder` 함수는 `Int x` 파라미터를 받으며, `Int y` 파라미터를 받는 람다 식을 반환합니다. 반환된 람다 식은 `Int` 인수를 받아 `Int` 결과를 반환하는 함수입니다. 해당 함수는 람다 식을 파라미터로 받아 결과로 반환하기 때문에 고차 함수입니다. 

> 고차 함수는 다른 함수를 파라미터로 받거나 결과로 반환하는 함수입니다.

- 고차 함수는 함수형 프로그래밍 언어의 강력한 기능으로, 연산을 통해 추상화하고 보다 유연하고 재사용 가능한 코드를 만들 수 있습니다. 예를 들어, 파라미터로 전달된 함수에 따라 요소 목록에서 다양한 작업을 수행할 수 있는 일반 함수를 생성하기 위해 고차 함수를 사용할 수 있습니다.

- Kotlin에서 람다 표현식은 종종 고차 함수와 함께 사용되어 유연하고 재사용 가능한 코드를 생성합니다. 예를 들어 `map` 함수를 사용하여 목록의 각 요소에 람다 식을 적용하고 변환된 요소가 포함된 새 목록을 반환할 수 있습니다.

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val squares = numbers.map { it * it }
```

- 이 예에서 `map` 함수는 람다 식을 파라미터로 사용하고 변환된 요소가 포함된 새 목록을 반환하는 고차 함수입니다. 람다 식은 원래 목록의 각 요소에 적용되며 요소의 제곱을 반환합니다. 반환되는 리스트에는 원래 요소의 제곱이 포함됩니다.

### 부록: Java의 람다에서 고차 함수를 구현하는 것이 불가능합니까?

- 람다식을 이용하여 자바에서 고차 함수 구현이 가능합니다. Java에서 고차 함수를 구현하려면 함수 인터페이스(단일 추상 메서드가 있는 인터페이스)를 사용하여 파라미터로 전달되거나 결과로 반환되는 함수를 나타낼 수 있습니다. 예를 들어 Java에서 다음과 같은 인터페이스 및 고차 함수를 구현할 수 있습니다.

```java
@FunctionalInterface
interface Function<T, R> {
    R apply(T t);
}

@FunctionalInterface
interface HigherOrderFunction<T, R> {
    Function<T, R> makeAdder(T x);
}
```

- 이 예에서 함수 인터페이스는 타입 T의 단일 파라미터를 사용하고 타입 R의 결과를 반환하는 함수를 나타냅니다. `HigherOrderFunction` 인터페이스는 T 유형의 단일 파라미터를 사용하고 T 타입의 인수를 사용하며, R 타입의 결과를 반환하는 `Function` 개체를 반환하는 고차 함수를 나타냅니다.

- 람다 식을 사용하여 이러한 기능 인터페이스를 구현하고 Java에서 고차 함수를 만들 수 있습니다. 예를 들어 람다 식을 사용하여 `HigherOrderFunction` 인터페이스를 구현하고 입력에 고정 값을 추가하는 함수를 반환하는 고차 함수를 만드는 방법은 다음과 같습니다.

```java
HigherOrderFunction<Integer, Integer> makeAdder = (x) -> (y) -> x + y;
```

- 이 예제에서 람다 식 `(y) -> x + y`는 `Integer y`를 받아 x를 y에 더한 결과를 반환하는 함수를 나타냅니다. 전체를 포함하는 람다 식 `(x) -> (y) -> x + y`는 `Integer x`를 받고 내부의 람다 식을 `Function` 개체로 반환하는 고차 함수를 나타냅니다.

- 전반적으로 람다 식과 기능적 인터페이스를 사용하여 Java에서 고차 함수를 구현하는 것이 가능합니다. 그러나 Java에서 고차 함수를 만들고 사용하기 위한 방법은 Kotlin과 같이 함수형 프로그래밍을 더 잘 지원하는 언어보다 더 장황하고 번거로울 수 있습니다.


> ***요우의 사담*** 😎
> 쥐꼬리만한 지금까지의 자바 개발 경험을 돌이켜 봤을 때, 일급 함수와 고차 함수를 자바 코드로 구현하는 것이 아예 불가능하지는 않아 보인다.
> 다만 지금까지의 경험으로써, 프로덕션 코드에 그렇게까지 무리하면서 일급/고차 함수를 구현할 필요는 없었고, javascript 와 유사하게 구현하려고 하면 본문과 같이 쓸데 없이 코드의 복잡도만 올라가는 경우가 대부분이었다.
> 그래서 차라리 본격적으로 람다를 일급/고차 함수로 사용하려면 코틀린을 쓰는게 정신건강에 이로워보인다.

![kotlin.jpeg](/kotlin.jpeg =500x){.align-center}