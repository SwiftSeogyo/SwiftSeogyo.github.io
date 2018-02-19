---
layout: post
title: "Composing types in Swift"
date: 2018-02-06 10:00:00 +0900
author: Kevin
categories: swift
---

*원문: [Composing types in Swift](https://www.swiftbysundell.com/posts/composing-types-in-swift)*

---

# Swift에서 Composing(조립)

Composition(조립)은 여러 유형간의 코드를 분리된 방식으로 공유할 수 있는 유용한 방식이다.
이는 상속 트리에 의존하기 보다 다수의 개별 조각을 기능적으로 합성하는 아이디어인 "[Composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance)" 문장처럼 서브클래싱을 대체할 수 있다.

서브클래싱/상속 또한 초 유용하다. (우리 모두가 의존하는 Apple's frameworks는 이 패턴에 무겁게 의존하고 있다)
컴포지션을 사용하면 더 간단하고 견고하게 구조화 된 코드를 작성할 수있는 많은 경우가 있습니다.

합성을 사용하면 당신이 간결하게 좀더 견고하게 구조적인 코드를 작성할 수 있게하는 많은 상황이 있습니다.

## Struct composition

### User와 Friend 모델이 있는 SNS앱을 작성하는 예

* `User` 모든 종류의 사용자에게 적용
* `Friend` User와 동일하지만 친구가 된 날짜를 포함한 모델

전통적으로 아래와 같이 상속으로 구현할 수 있다.

{% highlight swift %}
class User {
    var name: String
    var age: Int
}

class Friend: User {
    var friendshipDate: Date
}
{% endhighlight %}

위 단점 3가지는

1. 상속을 하려면 참조유형인 Class를 사용해야한다. 실수로 shared mutable state를 넣을 수 있다. 코드 베이스의 한 부분이 모델을 변경하면 전체가 자동으로 반영이 된다, 이러한 변경을 제대로 관찰하지 않고 정확하게 처리하지 않으면 버그를 일으킬 수 있다.
2. Friend는 User이기도 함으로 User 인스턴스를 사용하는 함수에게 전달 될 수 있습니다. 무해해 보일지 모르나. saveDataForCurrentUser함수에 전달되는 경우 잘못된 방법으로 사용될 위험성이 증가할 수 있다.
3. Friend에게 우리는 실제로 mutable한 User 속성을 원하지 않는다. (친구의 이름을 변경하는 것은 매우 어렵습니다). 그러나 상속에 의존하기 때문에 우리는 속성의 변경 가능성도 모두 상속받는다.

대신 Struct 조립을 이용하자.

{% highlight swift %}
struct User {
    var name: String
    var age: Int
}

struct Friend {
    let user: User
    var friendshipDate: Date
}
{% endhighlight %}

let을 이용하여 Friend의 user프로퍼티를 수정할 수 없다. 오오오!

## Class composition

모든 구조체를 만들란 이야기는 아니다. 클래스는 강력하고 때론 밸류 의미보다 참조의를 원할 때가 있다. 클래스를 사용하더라도 상속의 대안으로 composition을 사용할 수 있다.

### UITableView를 구성하는 예
매우 일반적인 tableView구현은 자기 자신을 dataSource로 하는 형태이다.

{% highlight swift %}
extension FriendListViewController: UITableViewDataSource {
   func tableView(_ tableView: UITableView,
                  numberOfRowsInSection section: Int) -> Int {
        return friends.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        ...
    }
}
{% endhighlight %}

이 기능을 더 분리하고 재사용 하려는 경우 대신 Composition을 사용하십시오

{% highlight swift %}
class FriendListTableViewDataSource: NSObject, UITableViewDataSource {
    var friends = [Friend]()

    func tableView(_ tableView: UITableView,
                   numberOfRowsInSection section: Int) -> Int {
        return friends.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        ...
    }
}
{% endhighlight %}

UITableViewDataSource를 구현하는 전용 dataSource 객체를 만듬으로써 친구 목록을 단순히 할당하는 것으로 테이블 보기에 필요한 정보를 제공한다.

{% highlight swift %}
class FriendListTableViewDataSource: NSObject, UITableViewDataSource {
    var friends = [Friend]()

    func tableView(_ tableView: UITableView,
                   numberOfRowsInSection section: Int) -> Int {
        return friends.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        ...
    }
}
{% endhighlight %}

이 접근법의 장점은 앱의 다른 곳에서 친구 목록을 표시하려는 경우이 기능을 재사용하는 것이 매우 쉽다는 것입니다. 일반적으로 View Controller 밖으로 물건을 옮기는 것이 "Massive View Controller" 증후군을 피할 수 있는 좋은 방법이 될 수 있습니다.
데이터 소스를 별도의 구성 가능한 유형으로 만들었 듯이 다른 기능에 대해서도 동일한 작업을 수행 할 수 있습니다.
(데이터 및 이미지로드, 캐싱 등).

특히 뷰 컨트롤러와 함께 컴포지션을 사용하는 또 다른 방법은 하위 뷰 컨트롤러를 사용하는 것입니다.
자세한 내용은 "[Swift에서 하위보기 컨트롤러를 플러그인으로 사용](https://www.swiftbysundell.com/posts/composing-types-in-swift)"을 확인하십시오.

## Enum composition

마지막으로 열거 형을 구성하는 것이 코드 중복을 줄일 수있는보다 세분화 된 설정을 제공 할 수있는 방법을 살펴 보겠습니다. 백그라운드 스레드에서 많은 작업을 수행 할 수있는 Operation 유형을 구축하고 있다고 가정 해 보겠습니다. 작업의 상태가 변경 될 때 반응 할 수 있도록 작업이로드, 실패 또는 완료되는 경우가있는 State 열거 형을 만듭니다.

{% highlight swift %}
class Operation {
    var state = State.loading
}

extension Operation {
    enum State {
        case loading
        case failed(Error)
        case finished
    }
}
{% endhighlight %}

위의 내용은 매우 직선적으로 보일 수도 있지만 이제는 이러한 작업 중 하나를 사용하여 뷰 컨트롤러에서 이미지 배열을 처리하는 방법에 대해 살펴 보겠습니다.

{% highlight swift %}
class ImageProcessingViewController: UIViewController {
    func processImages(_ images: [UIImage]) {
        // Create an operation that processes all images in
        // the background, and either throws or succeeds.
        let operation = Operation {
            let processor = ImageProcessor()
            try images.forEach(processor.process)
        }

        // We pass a closure as a state handler, and for each
        // state we update the UI accordingly.
        operation.startWithStateHandler { [weak self] state in
            switch state {
            case .loading:
                self?.showActivityIndicatorIfNeeded()
            case .failed(let error):
                self?.cleanupCache()
                self?.removeActivityIndicator()
                self?.showErrorView(for: error)
            case .finished:
                self?.cleanupCache()
                self?.removeActivityIndicator()
                self?.showFinishedView()
            }
        }
    }
}
{% endhighlight %}

언뜻보기에는 위의 코드에 문제가있는 것처럼 보이지 않을 수도 있지만 failed 사례와 finished 사례를 처리하는 방법을 면밀히 살펴보면 여기에 코드 중복이 있음을 알 수 있습니다.

코드 중복은 항상 나쁜 것은 아니지만 이렇게 다른 상태를 처리 할 때는 가능한 한 작은 코드를 복제하는 것이 좋습니다. 그렇지 않으면 가능한 모든 코드 경로를 테스트하기 위해 더 많은 테스트를 작성하고 더 많은 수동 QA를 수행해야합니다. 중복을 많이하면 상황이 바뀔 때 버그가 을 빠져 나가는 것이 더 쉬워집니다.

이것은 컴포지션이 매우 편리하다는 또 다른 상황입니다. 단일 열거 형을 사용하는 대신, 우리 State 를 유지하기 위해 두 개를 만들고 다음과 같이 Outcome 를 나타내는 또 다른 하나를 생성합시다.

{% highlight swift %}
extension Operation {
    enum State {
        case loading
        case finished(Outcome)
    }

    enum Outcome {
        case failed(Error)
        case succeeded
    }
}
{% endhighlight %}

위의 변경 사항을 적용하여, 이렇게 구성된 열거형의 장점을 얻기 위해 우리의 호출 부분을 업데이트하자

{% highlight swift %}
operation.startWithStateHandler { [weak self] state in
    switch state {
    case .loading:
        self?.showActivityIndicatorIfNeeded()
    case .finished(let outcome):
        // All common actions for both the success & failure
        // outcome can now be moved into a single place.
        self?.cleanupCache()
        self?.removeActivityIndicator()

        switch outcome {
        case .failed(let error):
            self?.showErrorView(for: error)
        case .succeeded:
            self?.showFinishedView()
        }
    }
}
{% endhighlight %}

보시다시피, 우리는 모든 코드 중복을 없애고 모든 것이 훨씬 명확 해졌습니다.

## 결론

Composition is a great tool that can lead to simpler, more focused types that are easier to maintain, reuse & test.

조립은 완변히 서브크래싱이나 types에 직접적인 인라인 코드를 완전히 대체하진 않지만, 다양한 유형사이를 관계를 설정할 때는 유의해야할 점이 있습니다.

물론 이 게시물에서 다루는 것보다 Swift에서 조립을 사용하는 다른 방법이 많이 있습니다. 특히 곧 작성 될 기능과 프로토콜을 작성하는 두 가지 사례가 있습니다.
