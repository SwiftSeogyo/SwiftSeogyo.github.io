---
layout: post
title: "공유 상태를 막기 위한 팩토리 패턴 사용(Using the factory pattern to avoid shared state in Swift)"
date: 2018-04-10 10:00:00 +0900
author: Jay
categories: swift
---

*원문: [Using the factory pattern to avoid shared state in Swift](https://www.swiftbysundell.com/posts/using-the-factory-pattern-to-avoid-shared-state-in-swift)*

---

# 공유 상태를 막기 위한 팩토리 패턴 사용(Using the factory pattern to avoid shared state in Swift)

공유 상태는 대부분의 앱에서 버그 발생의 원인이다. 시스템의 여러 곳에서 동일한 가변 상태에 의존할 때 발생한다.

팩토리 패턴을 사용하여 자신의 상태를 관리하는 명확히 분리된 인스턴스를 생성하는 방법을 살펴본다.

## 문제

{% highlight swift %}
class Request {
    enum State {
        case pending
        case ongoing
        case completed(Result)
    }

    let url: URL
    let parameters: [String : String]
    fileprivate(set) var state = State.pending

    init(url: URL, parameters: [String : String] = [:]) {
        self.url = url
        self.parameters = parameters
    }
}
{% endhighlight %}

{% highlight swift %}
dataLoader.perform(request) { result in
    // Handle result
}
{% endhighlight %}

위의 문제는 뭘까? `Request`는 요청을 수행하는 위치 및 방법 뿐 아니라 상태도 가지고 있다.

{% highlight swift %}
class TodoListViewController: UIViewController {
    private let request = Request(url: .todoList)
    private let dataLoader = DataLoader()

    func loadItems() {
        dataLoader.perform(request) { [weak self] result in
            self?.render(result)
        }
    }
}
{% endhighlight %}

위에서 `pending` 상태의 `Request`가 `completed`되기 전에 `loadItems()`가 여러번 호출될 때 의도치 않는 상황이 발생할 수 있다. 모든 요청은 동일한 인스턴스를 사용하여 수행되므로 상태를 계속 재설정한다.

이 문제를 해결하는 한 가지 방법은 새 요청이 실행될 때 보류중인 요청을 자동으로 취소하는 것이다. 이것은 문제를 해결하기도 하지만 다른 문제를 일으킬 수도 있으며 API를 예측할 수 없게 만들고 사용하기 더 어렵게 한다.

## 팩토리 메소드

위 문제를 해결하기 위해 팩토리 메소드를 사용한다. 이러한 디커플링은 일반적으로 공유 상태를 피할 때 필요하며 예측 가능한 코드를 만든다.

그럼 어떻게 할까? `Request`의 서브클래스인 `StatefulRequest`를 만들고 상태 정보를 옮긴다.

{% highlight swift %}
// Our Request class remains the same, minus the statefulness
class Request {
    let url: URL
    let parameters: [String : String]

    init(url: URL, parameters: [String : String] = [:]) {
        self.url = url
        self.parameters = parameters
    }
}

// We introduce a stateful type, which is private to our networking code
private class StatefulRequest: Request {
    enum State {
        case pending
        case ongoing
        case completed(Result)
    }

    var state = State.pending
}
{% endhighlight %}

그리고 `Request`에 팩토리 메소드를 추가한다.

{% highlight swift %}
private extension Request {
    func makeStateful() -> StatefulRequest {
        return StatefulRequest(url: url, parameters: parameters)
    }
}
{% endhighlight %}

마지막으로 `DataLoader`가 요청을 시작할 때 매번 `StatefulRequest`를 생성한다.

{% highlight swift %}
class DataLoader {
    func perform(_ request: Request) {
        perform(request.makeStateful())
    }

    private func perform(_ request: StatefulRequest) {
        // Actually perform the request
        ...
    }
}
{% endhighlight %}

요청이 수행될 때마다 항상 새 인스턴스를 생성함으로써 상태가 공유되는 가능성을 제거했다.

## 일반적인 패턴

이것은 시퀀스를 통한 반복과 같은 패턴이다. 반복 상태를 공유하는 대신 이터레이터는 각 이터레이션의 상태를 가지고 생성된다.

{% highlight swift %}
for book in books {
    ...
}
{% endhighlight %}

`books.makeIterator()`를 호출하면 어떻게 될까? 콜렉션에 대해서는 다음에 알아보자.

## 팩토리

다른 상황을 보자.

사용자가 카테고리나 추천을 통해 영화를 나열한다고 하자. 다음과 같이 싱글톤 `MovieLoader`를 사용하여 백엔드에 요청한다.

{% highlight swift %}
class CategoryViewController: UIViewController {
    // We paginate our view using section indexes, so that we
    // don't have to load all data at once
    func loadMovies(atSectionIndex sectionIndex: Int) {
        MovieLoader.shared.loadMovies(in: category, sectionIndex: sectionIndex) {
            [weak self] result in
            self?.render(result)
        }
    }
}
{% endhighlight %}

싱글톤을 사용하는 게 문제가 되지 않을 수 있지만 사용자가 요청 완료보다 빠르게 앱을 사용하면 까다로운 상황이 발생할 수 있다. 끝나지 않은 요청의 긴 대기열로 인해 앱이 느려질 수 있다.

여기서 문제가 되는 것은 상태(대기열)가 공유된다는 것입니다.

이 문제를 해결하기 위해 각 뷰 컨트롤러에서 `MovieLoader`의 새 인스턴스를 사용한다. 이렇게 하면 각 로더가 제거될 때 모든 대기중 요청을 취소하여 관심 없는 요청을 없앤다.

{% highlight swift %}
class MovieLoader {
    deinit {
        cancelAllRequests()
    }
}
{% endhighlight %}

하지만 뷰 컨트롤러를 만들 때마다 `MovieLoader`의 새 인스턴스를 직접 만들어야 하는 것은 아니다. 공장을 사용하자.

{% highlight swift %}
class MovieLoaderFactory {
    private let cache: Cache
    private let session: URLSession

    // We can have the factory contain references to underlying dependencies,
    // so that we don't have to expose those details to each view controller
    init(cache: Cache, session: URLSession) {
        self.cache = cache
        self.session = session
    }

    func makeLoader() -> MovieLoader {
        return MovieLoader(cache: cache, session: session)
    }
}
{% endhighlight %}

각 뷰 컨트롤러를 `MovieLoaderFactory`와 함께 초기화하고, 로더가 필요하면 팩토리로 늦게 생성합니다.

{% highlight swift %}
class CategoryViewController: UIViewController {
    private let loaderFactory: MovieLoaderFactory
    private lazy var loader: MovieLoader = self.loaderFactory.makeLoader()

    init(loaderFactory: MovieLoaderFactory) {
        self.loaderFactory = loaderFactory
        super.init(nibName: nil, bundle: nil)
    }

    private func openRecommendations(forMovie movie: Movie) {
        let viewController = RecommendationsViewController(
            movie: movie,
            loaderFactory: loaderFactory
        )

        navigationController?.pushViewController(viewController, animated: true)
    }
}
{% endhighlight %}

위에서 볼 수 있듯이 팩토리 패턴을 사용하는 것의 큰 이점은 팩토리를 하위 뷰 컨트롤러로 넘길 수 있다는 것이다.

## 결론

팩토리는 코드를 디커플링하기 매유 유용하고, 상태 관리나 생성에 있어 관심사를 더 분리할 수 있게 한다. 항상 새로운 인스턴스를 생성함으로써 공유 상태를 피할 수 있으며, 팩토리는 이러한 인스턴스의 생성을 캡슐화하는 좋은 방법이다.
