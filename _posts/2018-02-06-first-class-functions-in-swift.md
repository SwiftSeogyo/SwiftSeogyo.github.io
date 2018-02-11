---
layout: post
title:  "일급 함수(First class functions in Swift)"
date:   2018-02-06 11:00:00 +0900
categories: swift
---

*원문: [First class functions in Swift](https://www.swiftbysundell.com/posts/first-class-functions-in-swift)*

---

언어에서 일급 함수를 지원하면 함수와 메소드를 객체나 값처럼 사용할 수 있다. 함수를 인자로 넘기고, 프로퍼티에 저장하고, 함수에서 반환할 수 있다.

## 함수를 인자로 넘기기

{% highlight swift %}
let subviews = [button, label, imageView]

subviews.forEach { subview in
    view.addSubview(subview)
}
{% endhighlight %}

위 코드는 잘 동작하고 이상이 없다. 하지만 일급 함수의 이점을 쓰면 복잡해보이는 것을 줄일 수 있다.

`addSubview` 메소드를 `(UIView) -> Void` 타입의 클로저로 취급할 수 있다.  
`forEach`가 받는 인자가 `(Element) -> Void` 타입의 클로저고(이 경우 `Element` 타입은 `UIView`이기 때문에) 딱 맞아 떨어진다.

{% highlight swift %}
subviews.forEach(view.addSubview)
{% endhighlight %}

유의할 점이 있다. 인스턴스 메소드를 클로저로 사용할 때는 클로저를 사용하는 동안 인스턴스를 자동으로 리테인한다.  
위 예제처럼 논-이스케이핑 클로저일 때는 문제가 없다. 하지만 이스케이핑 클로저일 경우 리테인 사이클을 피하려면 알아두어야 한다.

## 이니셜라이저를 인자로 넘기기

{% highlight swift %}
images.map(UIImageView.init)
      .forEach(stackView.addArrangedSubview)
{% endhighlight %}

## 인스턴스 메소드 참조 생성하기

엑스코드에서 타입 메소드를 호출하려 할 때 인스턴스 메소드가 자동완성으로 제안된다. 타입 메소드에 인스턴스를 인자로 전달해 인스턴스 메소드를 클로저로 받을 수 있다.

{% highlight swift %}
let closure = UIView.removeFromSuperview(view)
{% endhighlight %}

## 결론

일급 함수는 강력한 기능이다. 하지만 큰 힘에는 큰 책임이 따른다.  
우리의 목표는 사용하기 쉬운 API를 만들고, 유지보수가 쉬운 코드를 작성하는 것이다. 일급 함수는 목표를 달성하는 데 도움이 되지만 지나치면 정반대의 결과를 얻을 수 있다.
