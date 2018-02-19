---
layout: post
title: "함수를 통한 의존성 주입(Simple Swift dependency injection with functions)"
date: 2018-02-20 10:00:00 +0900
author: Jay
categories: swift
---

*원문: [Simple Swift dependency injection with functions](https://www.swiftbysundell.com/posts/capturing-objects-in-swift-closures)*

---

# 함수를 통한 의존성 주입(Simple Swift dependency injection with functions)

의존성 주입은 코드를 분리하고 쉽게 테스트 할 수있는 훌륭한 기술이다. 외부에서 개체를 주입하여 다양한 상황(예: 프로덕션 대 테스트)에서 다르게 설정할 수 있다.
대체로 스위프트에서는 프로토콜을 써서 의존성을 주입한다.

{% highlight swift %}
class CardGame {
    private let deck: Deck
    private let randomizer: Randomizer

    init(deck: Deck, randomizer: Randomizer = DefaultRandomizer()) {
        self.deck = deck
        self.randomizer = randomizer
    }

    func drawRandomCard() -> Card {
        let index = randomizer.randomNumber(upperBound: deck.count)
        let card = deck[index]
        return card
    }
}
{% endhighlight %}

{% highlight swift %}
protocol Randomizer {
    func randomNumber(upperBound: UInt32) -> UInt32
}

class DefaultRandomizer: Randomizer {
    func randomNumber(upperBound: UInt32) -> UInt32 {
        return arc4random_uniform(upperBound)
    }
}
{% endhighlight %}

프로토콜 기반 의존성 주입은 API가 복잡하면 유용하지만 하나의 목적(하나의 메소드)만 있을 때는 함수를 사용해 복잡성을 줄일 수 있다.

{% highlight swift %}
class CardGame {
    typealias Randomizer = (UInt32) -> UInt32

    private let deck: Deck
    private let randomizer: Randomizer

    init(deck: Deck, randomizer: @escaping Randomizer = arc4random_uniform) {
        self.deck = deck
        self.randomizer = randomizer
    }

    func drawRandomCard() -> Card {
        let index = randomizer(deck.count)
        let card = deck[index]
        return card
    }
}
{% endhighlight %}

랜더마이저 프로토콜을 간단한 타입알리아스로 변경하고 기본 인수로 함수를 넘겼다. 더이상 기본 구현 클래스가 필요 없고, 테스트에서 쉽게 랜더마이저를 대신할 수 있다.

{% highlight swift %}
class CardGameTests: XCTestCase {
    func testDrawingRandomCard() {
        var randomizationUpperBound: UInt32?

        let deck = Deck(cards: [Card(value: .ace, suite: .spades)])

        let game = CardGame(deck: deck, randomizer: { upperBound in            
            // Capture the upper bound to be able to assert it later
            randomizationUpperBound = upperBound

            // Return a constant value to remove randomness from
            // our test, making it run consistently.
            return 0
        })

        XCTAssertEqual(randomizationUpperBound, 1)
        XCTAssertEqual(game.drawRandomCard(), Card(value: .ace, suite: .spades))
    }
}
{% endhighlight %}
