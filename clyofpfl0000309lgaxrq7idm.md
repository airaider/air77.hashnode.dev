---
title: "코틀린(Kotlin)이 뭔데?"
datePublished: Tue Jul 16 2024 13:13:34 GMT+0000 (Coordinated Universal Time)
cuid: clyofpfl0000309lgaxrq7idm
slug: kotlin
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/lqcvMiBABHw/upload/791092b05e231091e1a3933d6a5f45b5.jpeg
tags: java, kotlin

---

## 급부상 📈

최근 IT 업계에서 코틀린의 인기가 급상승하고 있다.

구글, 넷플릭스, 우버 등 유명 기업들이 앞다투어 코틀린을 도입하고 있고 채용공고에서도 심심찮게 코틀린 전환을 하고 있다는 안내문이 많다.

2017년 구글이 안드로이드의 공식 개발 언어로 코틀린을 채택한 이후, 많은 기업들이 자바 대신 코틀린을 선택하고 있다.

코틀린이 어떤 매력이 있길래 이러한 트랜드가 발생했는지 알아보자!

---

## 코틀린의 장점

1. ### 간결한 문법 📖
    

코틀린은 자바에 비해 훨씬 간결한 문법을 제공한다.

이는 개발 시간 단축과 코드에 있어서 매우 중요한 가독성 향상으로 이어진다.

예를 들어, 자바에서 데이터 클래스를 만들려면 다음과 같이 작성해야 한다.

```java
public class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

물론 @Getter, @Setter 어노테이션으로 줄일 수 있다.

코틀린 버전을 알아보자.

```kotlin
data class Person(var name: String, var age: Int)
```

끝이다.

2. ### Null 안정성 🧯
    

개발자의 영원한 적이자 숙제, NPE(Null Pointer Exception)

멀쩡하다고 생각한 코드도 왜 운영에만 나가면 터지는 것일까, 단골손님은 아마 Null check를 빼먹어서 일 것이다.

코틀린은 이를 효과적으로 해결한다.

코틀린에서는 기본적으로 모든 타입이 non-null이며, null을 허용하려면 타입 뒤에 ?를 붙여야 한다.

```kotlin
var name: String = "John" // non-null String
var nullableName: String? = null // nullable String

// 컴파일 에러: name = null
// 정상 동작: nullableName = null

// Safe call
println(nullableName?.length) // null이면 null 반환, 아니면 길이 반환

// Elvis 연산자
val len = nullableName?.length ?: 0 // nullableName이 null이면 0 반환
```

3. ### 자바와의 완벽한 호환성 🔄
    

많은 회사들은 구시대의 유물인 레거시 코드를 걷어내고 싶어할 것이다.

기존 자바 프로젝트에 코틀린을 점진적으로 도입할 수 있어 전환 비용이 낮다.

코틀린은 자바 바이트코드로 컴파일되므로 기존 자바 라이브러리를 그대로 사용할 수 있다.

```kotlin
// 코틀린에서 자바 라이브러리 사용 예시
import java.util.ArrayList

fun main() {
    val list = ArrayList<String>()
    list.add("Kotlin")
    list.add("Java")
    println(list) // 출력: [Kotlin, Java]
}
```

4. ### 함수형 프로그래밍 지원 🧑‍💻
    

코틀린은 객체지향과 함수형 프로그래밍을 모두 지원하여 더 유연한 코드 작성이 가능합니다.

* **람다 표현식**
    
    간결한 문법으로 함수를 정의할 수 있다
    

```kotlin
val sum = { x: Int, y: Int -> x + y } println(sum(3, 5)) // 출력: 8
```

* **고차 함수**
    
    함수를 인자로 받거나 함수를 반환할 수 있다
    

```kotlin
fun operation(x: Int, y: Int, op: (Int, Int) -> Int): Int {
    return op(x, y)
}

val result = operation(4, 5) { a, b -> a * b }
println(result) // 출력: 20
```

* **불변성과 순수 함수**
    
    불변 데이터 구조를 쉽게 만들 수 있고, 부작용 없는 순수 함수를 작성할 수 있다
    

```kotlin
data class Person(val name: String, val age: Int)

fun incrementAge(person: Person): Person {
    return person.copy(age = person.age + 1)
}

val john = Person("John", 30)
val olderJohn = incrementAge(john)
println(john) // 출력: Person(name=John, age=30)
println(olderJohn) // 출력: Person(name=John, age=31)
```

* **컬렉션 처리:**
    
    함수형 스타일로 컬렉션을 쉽게 처리할 수 있다
    

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val evenSquares = numbers.filter { it % 2 == 0 }.map { it * it }
println(evenSquares) // 출력: [4, 16]

val sum = numbers.fold(0) { acc, n -> acc + n }
println(sum) // 출력: 15
```

* **지연 평가(Lazy evaluation):**
    
    시퀀스를 사용하여 대량의 데이터를 효율적으로 처리할 수 있다
    

```kotlin
val largeList = (1..1000000).toList()
val result = largeList.asSequence()
    .filter { it % 2 == 0 }
    .map { it * it }
    .take(5)
    .toList()
println(result) // 출력: [4, 16, 36, 64, 100]
```

* **패턴 매칭:**
    
    when 표현식을 사용하여 강력한 패턴 매칭을 구현할 수 있습니다
    

```kotlin
fun describe(obj: Any): String =
    when (obj) {
        1 -> "One"
        "Hello" -> "Greeting"
        is Long -> "Long number"
        !is String -> "Not a string"
        else -> "Unknown"
    }

println(describe(1)) // 출력: One
println(describe("Hello")) // 출력: Greeting
println(describe(1000L)) // 출력: Long number
println(describe(2)) // 출력: Not a string
println(describe("Other")) // 출력: Unknown
```

이러한 함수형 프로그래밍 기능들을 통해 코틀린에서는 더 선언적이고, 간결하며, 유지보수가 쉬운 코드를 작성할 수 있다.

또한 이러한 기능들은 동시성 프로그래밍이나 대규모 데이터 처리와 같은 복잡한 문제를 해결하는 데에도 매우 유용하다.

---

## 정리 🧹

종합하여 요약해보자

* **생산성 향상:** 간결한 문법과 강력한 기능으로 인해 같은 기능을 구현하는 데 필요한 코드량이 줄어들어 개발 속도가 빨라집니다.
    
* **안정성 증가:** Null 안정성, 불변성(immutability) 기본 지원 등으로 인해 런타임 에러가 줄어들고 더 안정적인 애플리케이션을 만들 수 있습니다.
    
* **현대적인 언어 기능:** 확장 함수, 데이터 클래스, 코루틴 등 현대적인 프로그래밍 패러다임을 쉽게 적용할 수 있는 기능들을 제공합니다.
    
* **안드로이드 개발의 공식 언어:** 구글이 안드로이드 개발의 공식 언어로 코틀린을 채택함에 따라, 안드로이드 개발자들 사이에서 코틀린의 사용이 급증하고 있습니다.
    

---

## 결론 🚀

코틀린은 자바의 장점을 계승하면서도 현대 프로그래밍 언어의 장점을 잘 흡수다.

IT 기업들이 코틀린을 선택하는 이유는 명확하다.

개발할 때 효율도 훨씬 좋아지고, 더 안정적이면서 나중에 고치기도 쉬운 코드를 만들 수 있기 때문이다.

또한, 자바와의 완벽한 호환성으로 인해 기존 프로젝트에서 점진적인 전환이 가능하다는 점도 큰 장점이다.

앞으로도 코틀린 인기는 쭉쭉 올라갈 것 같으며 자바 개발자들한테는 새로운 도전이 될 수도 있겠지만, 동시에 새로운 기회가 될 것이다.

이제 코틀린을 배워두면 나중에 큰 힘이 될 것이다.

자바만 고집하다간 뒤처질 수 있으니, 이참에 기존 프로젝트를 코틀린으로 전환해보는 연습도 해보는것이 좋겠다!