---
layout: post
title: "팩토리를 사용한 의존성 주입(Dependency injection using factories in Swift)"
date: 2018-03-27 10:00:00 +0900
author: Jay
categories: swift
---

*원문: [Dependency injection using factories in Swift](https://www.swiftbysundell.com/posts/dependency-injection-using-factories-in-swift)*

---

# 팩토리를 사용한 의존성 주입(Dependency injection using factories in Swift)

의존성 주입은 코드를 테스트하기 쉽게 만드는데 필수적이다. 자신에 종속되게 만들거나 외부의 싱글톤에 접근하는 대신 필요한 모든 객체를 외부에서 전달 받는다는 생각이다.
하지만 주어진 객체에 대한 의존성이 증가하면 객체를 초기화하는 것이 어려워질 수 있다.

{% highlight swift %}
class UserManager {
    init(dataLoader: DataLoader, database: Database, cache: Cache,
         keychain: Keychain, tokenManager: TokenManager) {
        ...
    }
}
{% endhighlight %}

## 의존성 전달

메시징 앱을 예를 들면,

{% highlight swift %}
class MessageListViewController: UITableViewController {
    private let loader: MessageLoader

    init(loader: MessageLoader) {
        self.loader = loader
        super.init(nibName: nil, bundle: nil)
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        loader.load { [weak self] messages in
            self?.reloadTableView(with: messages)
        }
    }
}
{% endhighlight %}

`MessageListViewController`에 `MessageLoader`를 주입해서 데이터를 읽어 온다. 나쁘지 않다. 하지만 어떤 때는 다른 뷰 컨트롤러로 이동해야 한다.

목록에서 셀을 누르면 새 화면으로 이동한다. 메시지를 전부 볼 수 있고 답글을 달 수 있는 `MessageViewController`를 만든다. 답글을 다는 기능을 위해 `MessageSender`를 만들고 주입한다.

{% highlight swift %}
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let message = messages[indexPath.row]
    let viewController = MessageViewController(message: message, sender: sender)
    navigationController?.pushViewController(viewController, animated: true)
}
{% endhighlight %}

이제 문제가 생긴다. `MessageViewController`가 `MessageSender`를 필요하기 때문에 `MessageListViewController`에서 `MessageSender`를 알아야 한다. 한 가지 방법은 리스트 뷰 컨트롤러에 센더를 추가하는 것이다.

{% highlight swift %}
class MessageListViewController: UITableViewController {
    init(loader: MessageLoader, sender: MessageSender) {
        ...
    }
}
{% endhighlight %}

위 코드는 `MessageListViewController`를 사용하기 더 어렵게 만든다. 왜 리스트가 센더를 알아야 하지?

다른 방법은 `MessageSender`를 싱글톤으로 만드는 것이다.

{% highlight swift %}
let viewController = MessageViewController(
    message: message,
    sender: MessageSender.shared
)
{% endhighlight %}

하지만 [Avoiding singletons in Swift](https://www.swiftbysundell.com/posts/avoiding-singletons-in-swift)에서 본 것처럼 싱글톤은 몇 가지 단점을 가지고 있으며 명확하지 않은 의존성으로 구조를 이해하기 어렵게 한다.

## 구원의 팩토리

위의 모든 것을 건너뛰고, `MessageListViewController`가 `MessageSender`를 모르고, 이후의 모든 뷰 컨트롤러에서 필요한 모든 의존성을 사용할 수 있다면 좋지 않을까?

{% highlight swift %}
let viewController = factory.makeMessageViewController(for: message)
{% endhighlight %}

[Using the factory pattern to avoid shared state in Swift](https://www.swiftbysundell.com/posts/using-the-factory-pattern-to-avoid-shared-state-in-swift)에서 본 것처럼 팩토리의 정말 좋은 점은 객체의 생성과 사용이 완전히 분리된다는 것이다. 이것은 의존성을 낮추기 때문에 리팩토링하거나 변경할 때 도움이 된다.

**어떻게 하면 될까?**

팩토리에 대한 프로토콜을 정의해 의존성이나 이니셜라이저를 몰라도 필요한 뷰 컨트롤러를 쉽게 생성할 수 있다.

{% highlight swift %}
protocol ViewControllerFactory {
    func makeMessageListViewController() -> MessageListViewController
    func makeMessageViewController(for message: Message) -> MessageViewController
}
{% endhighlight %}

{% highlight swift %}
protocol MessageLoaderFactory {
    func makeMessageLoader() -> MessageLoader
}
{% endhighlight %}

## 하나의 의존성

`MessageListViewController`를 리팩토링해서 의존성을 가지는 대신 팩토리를 가지게 한다.

{% highlight swift %}
class MessageListViewController: UITableViewController {
    // Here we use protocol composition to create a Factory type that includes
    // all the factory protocols that this view controller needs.
    typealias Factory = MessageLoaderFactory & ViewControllerFactory

    private let factory: Factory
    // We can now lazily create our MessageLoader using the injected factory.
    private lazy var loader = factory.makeMessageLoader()

    init(factory: Factory) {
        self.factory = factory
        super.init(nibName: nil, bundle: nil)
    }
}
{% endhighlight %}

## 컨테이너

모든 핵심 유틸리티 객체가 포함된 `DependencyContainer`를 만든다.

{% highlight swift %}
class DependencyContainer {
    private lazy var messageSender = MessageSender(networkManager: networkManager)
    private lazy var networkManager = NetworkManager(urlSession: .shared)
}
{% endhighlight %}

객체를 초기화 할 때 같은 클래스의 다른 속성을 참조할 수 있도록 레이지 속성을 사용한다. 이는 순환 의존성 같은 문제를 피할 수 있게 해준다.

마지막으로 의존성 컨테이너를 팩토리 프로토콜을 따르게 한다.

{% highlight swift %}
extension DependencyContainer: ViewControllerFactory {
    func makeMessageListViewController() -> MessageListViewController {
        return MessageListViewController(factory: self)
    }

    func makeMessageViewController(for message: Message) -> MessageViewController {
        return MessageViewController(message: message, sender: messageSender)
    }
}

extension DependencyContainer: MessageLoaderFactory {
    func makeMessageLoader() -> MessageLoader {
        return MessageLoader(networkManager: networkManager)
    }
}
{% endhighlight %}

## 오너십 분산

의존성 컨테이너를 어디에 저장하고, 누가 소유하고, 어디에서 설정할 것인가? 의존성 컨테이너는 객체에 필요한 팩토리 구현체로 주입할 것이고, 그 객체는 팩토리에 대한 강한 참조를 하기 때문에 컨테이너를 다른 곳에 저장할 필요가 없다.

예를 들어 `MessageListViewController`가 앱의 초기 뷰 컨트롤러이면 그저 `DependencyContainer`를 만들어서 넘기면 된다.

{% highlight swift %}
let container = DependencyContainer()
let listViewController = container.makeMessageListViewController()

window.rootViewController = UINavigationController(
    rootViewController: listViewController
)
{% endhighlight %}

## 결론

팩토리 프로토콜과 컨테이너를 사용하여 의존성을 주입하는 것은 여러 의존성을 전달하지 않고 복잡한 초기화 프로그램을 작성하지 않아도되는 좋은 방법이다.

모든 팩토리를 프로토콜로 정의했기 때문에 테스트 버전의 목을 만들어 테스트할 수 있다.
