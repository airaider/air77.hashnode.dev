---
title: "템플릿 메소드 패턴이 뭔데?"
datePublished: Tue Aug 13 2024 07:46:37 GMT+0000 (Coordinated Universal Time)
cuid: clzs4cths00130alfex8n0iev
slug: 7ywc7zsm66aiouploygjoutncdtjkjthltsnbqg662u642wpw
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/U3-jqWs2o2g/upload/d97a2fbf97e24a91f38b38b1fcbb90cf.jpeg
tags: design-patterns, template-method-pattern

---

## 1\. 템플릿 메소드 패턴이란? 🤔

템플릿 메소드 패턴은 상위 클래스에서 알고리즘의 구조를 정의하고, 하위 클래스에서 그 구조의 세부 단계를 구현하는 디자인 패턴이다. 이 패턴을 사용하면 코드 재사용성이 높아지고, 알고리즘의 공통적인 부분과 각 단계별 세부 사항을 분리할 수 있다.

---

## 2\. 예시 ⚽️

농구와 축구의 예시로 알아보자

두 종목은 공통적으로

1. 심판에 의해 경기가 시작하고
    
2. 어떠한 규칙에 의해서 게임이 진행되고
    
3. 끝나면서 점수에 의해 승패가 갈린다
    

이 공통 부분을 템플릿 메소드 코드로 구성하면

```java
abstract class Game {
    // 템플릿 메소드
    public final void play() {
        start();
        playTurn();
        end();
    }

    protected abstract void start();
    protected abstract void playTurn();
    protected abstract void end();
}
```

축구와 농구는 이 게임이라는 틀을 가져가서 각자의 입맛에 맞게 확대시키면 된다

```java
class Football extends Game {
    @Override
    protected void start() {
        System.out.println("Football Game Started");
    }
    @Override
    protected void playTurn() {
        System.out.println("Playing Football");
    }
    @Override
    protected void end() {
        System.out.println("Football Game Ended");
    }
}

class Basketball extends Game {
    @Override
    protected void start() {
        System.out.println("Basketball Game Started");
    }
    @Override
    protected void playTurn() {
        System.out.println("Playing Basketball");
    }
    @Override
    protected void end() {
        System.out.println("Basketball Game Ended");
    }
}
```

위 코드에서 Game 클래스는 템플릿 메소드인 play()를 정의하고 있으며, Football과 Basketball 클래스는 각자의 세부 동작을 구현하고 있다

---

## **3\. 후크(Hook)에 대한 설명 🪝**

후크는 서브클래스에서 선택적으로 오버라이드할 수 있는 메소드를 의미한다.

템플릿 메소드 패턴에서는 보통 기본 구현을 제공하는 후크 메소드를 만들어, 서브클래스가 필요한 경우에만 이를 오버라이드하게 한다.

```java
abstract class Game {
    public final void play() {
        start();
        if (extraTurnNeeded()) {
            playTurn();
        }
        end();
    }

    protected abstract void start();
    protected abstract void playTurn();
    protected abstract void end();

    // 후크 메소드
    protected boolean extraTurnNeeded() {
        return true; // 기본값
    }
}
```

이 예시에서 extraTurnNeeded()는 후크 메소드로, 서브클래스에서 이를 오버라이드하여 추가적인 로직을 구현할 수 있다

---

## **4\. 할리우드 원칙(Hollywood Principle) 🤠**

> “Don’t call us, we’ll call you”

이 원칙은 객체 지향 프로그래밍에서 사용되는 설계 원칙으로, 상위 계층(또는 프레임워크)이 하위 계층(또는 컴포넌트)을 호출하도록 구조화하는 것을 의미한다.

이를 통해 하위 계층이 상위 계층에 의존하지 않고, 반대로 상위 계층이 하위 계층의 동작을 제어하는 방식으로 설계된다.

템플릿 메소드 패턴에서 할리우드 원칙이 적용되는 방식은 다음과 같다:

• 상위 클래스(예: Game)가 전체 알고리즘의 구조를 정의하고, 하위 클래스(예: Football, Chess)가 특정 단계에서 필요한 동작만을 구현한다

• 상위 클래스는 알고리즘의 흐름을 제어하며, 필요한 시점에 하위 클래스의 메소드를 호출한다. 이는 상위 클래스가 “우리가 너를 호출할게, 너는 스스로 호출하지 마”라고 하는 것과 같다.

```java
abstract class Game {
    // 템플릿 메소드
    public final void play() {
        start();
        playTurn();
        end();
    }

    protected abstract void start();
    protected abstract void playTurn();
    protected abstract void end();
}

class Football extends Game {
    @Override
    protected void start() {
        System.out.println("Football Game Started");
    }
    @Override
    protected void playTurn() {
        System.out.println("Playing Football");
    }
    @Override
    protected void end() {
        System.out.println("Football Game Ended");
    }
}
```

여기서 Game 클래스는 상위 클래스이며, play() 메소드를 통해 전체 흐름을 제어한다.

Football 클래스는 하위 클래스이며, start(), playTurn(), end() 메소드만 구현하며 play() 메소드가 전체 흐름을 제어하고, 하위 클래스의 메소드들을 적절한 시점에 호출함으로써 할리우드 원칙이 적용된다.

---

## **5\. 템플릿 메소드의 장단점**

**장점**

• **코드 재사용성 증가**: 알고리즘의 공통적인 부분을 상위 클래스에 정의하여, 여러 하위 클래스에서 재사용 가능.

• **유연성 향상**: 하위 클래스에서 세부적인 동작을 오버라이드하여 다양하게 활용 가능.

• **캡슐화 강화**: 알고리즘의 흐름을 상위 클래스에 숨기고, 세부적인 구현만을 하위 클래스에서 다룸.

**단점**

• **구조 복잡성 증가**: 너무 많은 후크 메소드와 서브클래스를 사용하면 코드 구조가 복잡해질 수 있음.

• **상속의 남용 가능성**: 상속을 과도하게 사용하면 유지보수가 어려워질 수 있음.