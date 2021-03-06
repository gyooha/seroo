---
title: "Algebra data type"
date: 2020-08-11
last_modified_at: 2020-08-11
desc: "함수형 프로그래밍 - 대수적 타입"
keywords: "Functional Programming, Algebra data type"
permalink: "/fp/algebra_data_type"
categories: 
    - Functioanl Programming
tags: 
    - FP
    - Functioanl Programming
    - Algebra data type
---

# 대수적 타입

대수적 타입은 다른 타입들을 모아서 형성되는 합성 타입의 종류로, 곱 타입과 합 타입이 있다. **대수적 데이터 타입의 핵심은 기존의 타입들을 결합하여 새로운 타입을 정의하는 것이다.**

### 곱 타입의 예와 한계

곱 타입은 하나의 자료구조에 여러 가지 타입을 한번에 정의할 수 있는 것으로 튜플이나 레코드가 대표적인 예다. **두 개 이상의 타입을 AND로 결합한 형태**이며 대부분의 프로그래밍 언어에서 사용되고, 타입을 정의하는 가장 일반적인 방법이다. 다음은 코틀린으로 구현한 레코드의 예다.

```kotlin
class Circle(val name: String, val x: Float, val y: Float, val radius: Float)
```

Circle 클래스는 String 타입의 name과 Float 타입의 x, y, radius를 AND로 결합하여 새로운 타입을 정의하였다. 따라서 대수적 타입에서의 곱 타입 정의와 정확히 맞아 떨어진다. 만약 여기에 Square를 추가하고, Circle과 Square를 나타내는 단일 타입을 만들려면 어떻게 해야 할까? 클래스에서는 타입 간의 AND 결합만 가능하기 때문에 추상화 클래스를 만들어야 한다. 추상화 클래스를 추가해서 AND 결합하면 다음과 같다.

```kotlin
open class Shape(name: String, x: Float, y: Float)
class Circle(name: String, x: Float, y: Float, val radius: Float): Shape(name, x, y)
class Square(name: String, x: Float, y: Float, val lenghth: Float): Shape(name, x, y)
```

Circle과 Square가 Shape를 상속하는 계층 구조로 만들어서 두 클래스를 단일 타입으로 표현했다. Circle과 Square의 공통 프로퍼티와 메서드는 Shape 클래스에 구현하고, 각 하위 클래스에는 차이점만 구현하는 일반적인 상속 구조다.

이번에는 여기에 Line이 추가되야 한다고 가정해 보자. Line에는 Shape의 공통 프로퍼티인 x,y 가 없다. 따라서 완전히 리팩터링 되어야 한다. 다음은 Line을 추가하기 위해서 리팩터링한 예다.

```kotlin
open class Shape(name: String)
class Circle(name: String, val x: Float, val y: Float, val radius: Float): Shape(name)
class Square(name: String, val x: Float, val y: Float, val lenghth: Float): Shape(name)
class Line(name: String, val x1: Float, val x2: Float, val y1: Float, val y2: Float, val lenghth: Float): Shape(name)
```

Line까지 Shape라는 단일 타입으로 표현하기 위해서 x, y는 Circle과 Square로 옮겼다. 또는 Shape에 하나의 계층을 더 추가하는 방법도 있다. 그 외에도 여러가지 방법이 있겠지만, 간단한 문제를 해결하는 데 프로그램의 구조는 상당히 복잡해진다. 그리고 계층 구조가 복잡하면 프로그램의 유지보수나 유연성에 악영향을 끼치는 경우가 많다. 클래스뿐만 아니라 다른 곱 타입들도 타입을 결합하는 방법이 AND밖에 없기 때문에 동일한 한계를 가진다.

곱 타입의 특징을 살펴보기 위해서 곱 타입 Shape의 테두리 길이를 구하는 함수를 만들어 보자.

```kotlin
fun getGirthLength(shape: Shape): Double = when (shape) {
    is Circle -> 2 * Math.PI * shape.radius
    is Square -> 4 * shape.length.toDouble()
    is Line -> {
        val x2 = Math.pow(shape.x2 - shape.x1.toDouble(), 2.0)
        val y2 = Math.pow(shape.y2 - shape.y1.toDouble(), 2.0)
        Math.sqrt(x2 + y2)
    }
    else -> throw IllegalArgumentException()
}
```

여기서 when 구문을 사용할 때 else 구문을 반드시 작성해야 한다. 곱 타입은 타입을 구성하는 값들의 합이 전체를 의미하지 않는다.

컴파일러는 Shape의 하위 클래스가 얼마나 있을지 예측할 수 없기 때문에 else 구문에서 다른 입력이 들어오는 경우를 대비해야 한다. 앞으로 나올 합 타입 예제에서 어떤 차이가 있는지 더 자세히 살펴보자.

### 합 타입 이용한 OR 결합

C나 자바와 같은 언어에 비해 하스켈, 스칼라, 코틀린 등 모던 언어들은 보다 강력한 합 타입 시스템을 제공한다. **합 타입은 곱 타입과 달리 두 개 이상의 타입을 OR로 결합한다.** 코틀린은 sealed class를 사용하여 합 타입을 만들며, 위의 예제를 sealed class를 사용하면 Shape를 다음과 같이 정의할 수 있다.

```kotlin

sealed class Shape
class Circle(val name: String, val x: Float, val y: Float, val radius: Float): Shape()
class Square(val name: String, val x: Float, val y: Float, val lenghth: Float): Shape()
class Line(val name: String, val x1: Float, val x2: Float, val y1: Float, val y2: Float, val lenghth: Float): Shape()

```

여기서 Shape는 Circle 또는 Square 또는 Line이다. 이렇게 합 타입은 여러 가지 타입을 OR로 결합하여 새로운 단일 타입을 만들 수 있다. 얼핏 보면 Enum과 비슷해 보이지만, 코틀린의 Enum 타입은 모든 값의 매개변수 타입이 동일해야 한다. 따라서 여러 가지 타입을 제약 없이 OR 결합할 수는 없다.

이제 Shape의 테두리 길이를 구하는 함수는 다음과 같이 작성될 수 있다.

```kotlin

fun getGirthLength(shape: Shape): Double = when (shape) {
    is Circle -> 2 * Math.PI * shape.radius
    is Square -> 4 * shape.length.toDouble()
    is Line -> {
        val x2 = Math.pow(shape.x2 - shape.x1.toDouble(), 2.0)
        val y2 = Math.pow(shape.y2 - shape.y1.toDouble(), 2.0)
        Math.sqrt(x2 + y2)
    }
}

```

여기서는 when 구문에서 else문을 따로 작성하지 않았다. 합 타입의 경우 when 구문에 모든 케이스에 대해 대응하면 else 구문을 작성하지 않아도 된다. 즉 컴파일러는 세 가지 타입 외에 다른 타입이 들어오지 않을 것이라는 것을 예측할 수 있다.

대수적 타입의 각 값은 자신만의 생성자를 가지고, 생성자를 얻기 위해서는 패턴 매칭을 사용한다. 이러한 특징으로 **함수형 프로그래밍에서는 패턴 매칭이 쉽고, 부수효과를 처리하지 않아도 된다**는 장점이 있다. 또한 **합 타입은 복잡한 상속 구조를 피하면서도 확장이 용이한 타입을 정의**할 수 있다.

### 함수형 프로그래밍에서의 대수적 타입

함수형 프로그래밍에서 대표적인 대수적 타입을 꼽자면 리스트가 있다. 리스트 내의 값은 타입이 모두 동일하지만, 타입들을 결합하여 새로운 타입을 정의할 수 있기 때문에 대수적 타입이다.

```kotlin

sealed class FunList<out T> {
    object Nil : FunList<Nothing>()
    data class Cons<out T>(val head: T, val tail: FunList<T>): FunList<T>()
}

```

FunList의 타입이 sealed class를 사용해서 선언되어 있다. 그러므로 FunList는 합 타입이다. Nil과 Cons의 OR결합으로 작성되어 있고, Nil과 Cons의 합은 FunList이다. 그래서 다음과 같은 패턴 매칭이 가능하다.

```kotlin
fun sum(list: FunList<Int>): Int = when (list) {
    FunList.Nil -> 0
    is FunList.Cons -> list.head + sum(list.tail)
}
```

반면에 코틀린의 리스트의 타입 선언의 일부는 다음과 같이 작성되어 있다.

```kotlin
public interface List<out E>: Collection<E> {
    //... 생략
}
```

코드를 보면 코틀린의 리스트는 Collection을 상속한 계층 구조로 선언되어 있다는 것을 알 수 있다. List는 여러 가지 속성과 메소드 들로 구성되어 있을 것이다. 그리고 List의 구현체는 class로 작성될 것이다. 결론적으로 코틀린의 List는 곱 타입이다. 코틀린의 List는 패턴 매칭을 지원하지 않기 때문에, sum 함수를 재귀로 작성하려면 다음과 같이 해야 한다.

```kotlin

fun sum(list: List<Int>): Int = when {
    list.isEmpty() -> 0
    else -> list.first() + sum(list.drop(1))
}

```

언뜻 보면 패턴 매칭처럼 보이지만, if else 조건절을 when문을 사용해서 작성한 것이다. 실제 코틀린 코드에서는 리스트가 sum 함수를 제공하기 때문에 재귀를 사용해서 구현할 필요는 없다.

두 리스트는 기능적으로 동일하지만 함수형 프로그래밍에서 대수적 데이터 타입은 **FunList**와 같이 합 타입으로 정의한다. 타입에 포함되는 모든 타입에 대한 정의가 명확해서 컴파일러가 타입을 예측하기 쉽기 때문이다. 덕분에 다음과 같은 이점이 있다.

* 더 쉽게 타입을 결합하고 확장할 수 있다.
* 생성자 패턴 매칭을 활용하여 간결한 코드를 작성할 수 있다.
* 철저한 타입 체크로 더 안전한 코드를 작성할 수 있다.