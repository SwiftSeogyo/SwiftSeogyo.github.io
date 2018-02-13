---
layout: post
title:  "클로저의 객체 캡처링(Capturing objects in Swift closures)"
date:   2018-02-13 10:00:00 +0900
categories: swift
---

*원문: [Capturing objects in Swift closures](https://www.swiftbysundell.com/posts/capturing-objects-in-swift-closures)*

---

## 대탈주

클로저에는 이스케이핑과 논-이스케이핑의 두 가지 변형이 있다. 클로저가 이스케이핑 된다는 것은 어딘가(프로퍼티 혹은 다른 클로저에 캡처됨) 저장된다는 것이다. 반면에 논-이스케이핑 클로저는 저장되지 않고 사용할 때 실행된다.

논-이스케이핑 클로저의 예로 `forEach`가 있다.

{% highlight swift %}
[1, 2, 3].forEach { number in
    ...
}
{% endhighlight %}

클로저가 콜렉션의 각 멤버들에 의해 직접 실행된다.

이스케이핑 클로저는 `DispatchQueue` 같은 비동기 API에서 쉽게 볼 수 있다.

{% highlight swift %}
DispatchQueue.main.async {
    ...
}
{% endhighlight %}

뭐가 다를까? 이스케이핑 클로저는 저장되므로 그것이 정의된 컨텍스트도 저장해야 한다. 컨텍스트에 다른 값이나 객체가 연관되면 클로저가 실행될 때까지 캡처해야 한다. 가장 일반적인 구현은 `self`에서 API를 사용할 때 명시적으로 `self` 캡처해야 한다.

## 캡처링과 리테인 사이클

{% highlight swift %}
class ListViewController: UITableViewController {
    private let viewModel: ListViewModel

    init(viewModel: ListViewModel) {
        self.viewModel = viewModel

        super.init(nibName: nil, bundle: nil)

        viewModel.observeNumberOfItemsChanged {
            self.tableView.reloadData()
        }
    }
}
{% endhighlight %}

뷰 컨트롤러에서 뷰 모델을 리테인 할 때 이스케이핑 클로저에 의해 뷰 컨트롤러가 캡처링 된다.

일반적인 해법은 `self`를 `weak`하게 캡처해서 리테인 사이클을 깨는 것이다.

{% highlight swift %}
viewModel.observeNumberOfItemsChanged { [weak self] in
    self?.tableView.reloadData()
}
{% endhighlight %}

## self 대신 컨텍스트 캡처링

위의 `[weak self]` 해법은 대부분의 상황에 적합하지만 단점도 있다. 우선, 컴파일러가 잠재적인 리테인 사이클에 대한 경고를 하지 않는다. 그리고 약한 참조를 강한 참조로 변환할 때 코드가 복잡해진다.

{% highlight swift %}
dataLoader.loadData(from: url) { [weak self] data in
    guard let strongSelf = self else {
        return
    }

    let model = try strongSelf.parser.parse(data, using: strongSelf.schema)
    strongSelf.titleLabel.text = model.title
    strongSelf.textLabel.text = model.text
}
{% endhighlight %}

다른 해법은 `self` 대신 개별 객체를 캡처하는 것이다. 이것은 "weak/strong self 댄스" 없이 리테인 사이클을 막는다. context 튜플을 사용하는 방법이다.

{% highlight swift %}
let context = (
    parser: parser,
    schema: schema,
    titleLabel: titleLabel,
    textLabel: textLabel
)

dataLoader.loadData(from: url) { data in
    let model = try context.parser.parse(data, using: context.schema)
    context.titleLabel.text = model.title
    context.textLabel.text = model.text
}
{% endhighlight %}

## 캡처링 대신 인자

캡처링의 다른 대안은 객체를 인자로 넘기는 것이다.

## 결론

모든 경우에 캡처하지 않는 것을 권장하지는 않지만 이 글이 매번 `self`를 캡처링하는 것의 대안이 되기를 바란다. 어떤 경우에는 고전적인 `weak self` 캡처가 적합하지만 경우에 따라 다른 테크닉을 사용하면 클로저 기반 코드를 쉽게 사용하고 유지 관리하는 데 도움이 될 것이다.
