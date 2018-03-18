---
layout: post
title: "내비게이션(Navigation in Swift)"
date: 2018-03-20 10:00:00 +0900
author: Jay
categories: swift
---

*원문: [Navigation in Swift](https://www.swiftbysundell.com/posts/navigation-in-swift)*

---

# 내비게이션(Navigation in Swift)

## 푸싱 문제

내비게이션을 하는 일반적으로 방법 중 하나는 `UINavigationController`를 사용해서 각 뷰 컨트롤러에서 다른 뷰 컨트롤러로 푸시나 팝을 하는 것이다.

{% highlight swift %}
class ImageListViewController: UITableViewController {
    override func tableView(_ tableView: UITableView,
                            didSelectRowAt indexPath: IndexPath) {
        let image = images[indexPath.row]
        let detailVC = ImageDetailViewController(image: image)
        navigationController?.pushViewController(detailVC, animated: true)
    }
}
{% endhighlight %}

위 방법은 단순한 앱에서는 잘 동작한다. 하지만 앱이 커지거나 하면 문제가 발생할 수 있다. 여러 곳에서 같은 뷰 컨트롤러를 사용하거나 외부에서 딥 링킹하거나 하면 처리하기기 어려워진다.

## 코디네이터

네이게이션을 좀 더 유연하게 만드는 방법 중 하나는 _코디네이터 패턴(coordinator pattern)_ 을 사용하는 것이다. 여러 뷰 컨트롤러를 관리하는 중간 또는 부모 객체를 사용하는 방법이다.
예를 들어 온보딩 플로우를 만든다고 하자. 각 뷰 컨트롤러에서 `navigationController`을 통해 다음 뷰 컨트롤러로 푸시하는 대신 코디네이터를 사용해서 관리할 수 있다.

{% highlight swift %}
protocol OnboardingViewControllerDelegate: AnyObject {
    func onboardingViewControllerNextButtonTapped(
        _ viewController: OnboardingViewController
    )
}

class OnboardingViewController: UIViewController {
    weak var delegate: OnboardingViewControllerDelegate?

    private func handleNextButtonTap() {
        delegate?.onboardingViewControllerNextButtonTapped(self)
    }
}
{% endhighlight %}

{% highlight swift %}
class OnboardingCoordinator: OnboardingViewControllerDelegate {
    weak var delegate: OnboardingCoordinatorDelegate?

    private let navigationController: UINavigationController
    private var nextPageIndex = 0

    // MARK: - Initializer

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    // MARK: - API

    func activate() {
        goToNextPageOrFinish()
    }

    // MARK: - OnboardingViewControllerDelegate

    func onboardingViewControllerNextButtonTapped(
        _ viewController: OnboardingViewController) {
        goToNextPageOrFinish()
    }

    // MARK: - Private

    private func goToNextPageOrFinish() {
        // We use an enum to store all content for a given onboarding page
        guard let page = OnboardingPage(rawValue: nextPageIndex) else {
            delegate?.onboardingCoordinatorDidFinish(self)
            return
        }

        let nextVC = OnboardingViewController(page: page)
        nextVC.delegate = self
        navigationController.pushViewController(nextVC, animated: true)

        nextPageIndex += 1
    }
}
{% endhighlight %}

코디네이터를 사용했을 때의 큰 장법은 내비게이션 로직을 뷰 컨트롤러에서 완전히 걷어낼 수 있다는 것이다.  그로 인해 유연성이 생기고, 뷰 컨트롤러는 뷰를 관리하는 데 집중할 수 있다.
위에서 주의할 것은 `OnboardingCoordinator`가 델리게이트를 가지고 있다는 것이다. 이것을 이용해서 `AppDelegate`가 코디네이터를 관리하게 하거나 다양한 단계의 코디네이터를 조합해서 사용한다.

## 내비게이터?

매우 많은 화면을 가진 앱에서 사용하기 좋은 방법으로 내비게이터 타입이 있다.

{% highlight swift %}
protocol Navigator {
    associatedtype Destination

    func navigate(to destination: Destination)
}
{% endhighlight %}

위 프로토콜을 사용해서 주어진 범위의 내비게이션을 하는 여러 내비게이터를 만들 수 있다.

{% highlight swift %}
class LoginNavigator: Navigator {
    // Here we define a set of supported destinations using an
    // enum, and we can also use associated values to add support
    // for passing arguments from one screen to another.
    enum Destination {
        case loginCompleted(user: User)
        case forgotPassword
        case signup
    }

    // In most cases it's totally safe to make this a strong
    // reference, but in some situations it could end up
    // causing a retain cycle, so better be safe than sorry :)
    private weak var navigationController: UINavigationController?

    // MARK: - Initializer

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    // MARK: - Navigator

    func navigate(to destination: Destination) {
        let viewController = makeViewController(for: destination)
        navigationController?.pushViewController(viewController, animated: true)
    }

    // MARK: - Private

    private func makeViewController(for destination: Destination) -> UIViewController {
        switch destination {
        case .loginCompleted(let user):
            return WelcomeViewController(user: user)
        case .forgotPassword:
            return PasswordResetViewController()
        case .signup:
            return SignUpViewController()
        }
    }
}
{% endhighlight %}

내비게이터를 사용하면 많은 레벨의 델리게이트를 만들 필요가 없다.

{% highlight swift %}
class LoginViewController: UIViewController {
    private let navigator: LoginNavigator

    init(navigator: LoginNavigator) {
        self.navigator = navigator
        super.init(nibName: nil, bundle: nil)
    }

    private func handleLoginButtonTap() {
        performLogin { [weak self] result in
            switch result {
            case .success(let user):
                self?.navigator.navigate(to: .loginCompleted(user: user))
            case .failure(let error):
                self?.show(error)
            }
        }
    }

    private func handleForgotPasswordButtonTap() {
        navigator.navigate(to: .forgotPassword)
    }

    private func handleSignUpButtonTap() {
        navigator.navigate(to: .signup)
    }
}
{% endhighlight %}

여기서 다 나아가 팩토리 패턴([여기서 볼 수 있다](https://www.swiftbysundell.com/posts/dependency-injection-using-factories-in-swift))과 내비게이터를 조합하면, 내비게이터에서 뷰 컨트롤러 생성을 제거하고 결합도를 낮출 수 있다.

{% highlight swift %}
class LoginNavigator: Navigator {
    private weak var navigationController: UINavigationController?
    private let viewControllerFactory: LoginViewControllerFactory

    init(navigationController: UINavigationController,
         viewControllerFactory: LoginViewControllerFactory) {
        self.navigationController = navigationController
        self.viewControllerFactory = viewControllerFactory
    }

    func navigate(to destination: Destination) {
        let viewController = makeViewController(for: destination)
        navigationController?.pushViewController(viewController, animated: true)
    }

    private func makeViewController(for destination: Destination) -> UIViewController {
        switch destination {
        case .loginCompleted(let user):
            return viewControllerFactory.makeWelcomeViewController(forUser: user)
        case .forgotPassword:
            return viewControllerFactory.makePasswordResetViewController()
        case .signup:
            return viewControllerFactory.makeSignUpViewController()
        }
    }
}
{% endhighlight %}

위처럼 하면 다른 내비게이터를 뷰 컨트롤러에 삽입할 때 각각에 대해 몰라도 된다. 예를 들어 `WelcomeViewController`에 `WelcomeNavigator`를 삽입하고 싶으면 팩토리만 수정하면 되고 `LoginNavigator`는 몰라도 된다.

## URL과 딥 링킹

코디네이터나 내비게이터를 사용하면 URL과 딥 링킹이 쉬워진다. 내비게이션을 위한 URL 처리 로직을 삽입할 수 있다. URL 기반 내비게이션은 나중에 살펴볼 것이다.

## 결론

내비게이션 로직을 뷰 컨트롤러에서 코디네이터나 내비게이터 같은 곳으로 빼내면 여러 뷰 컨트롤러 사이를 이동하는 게 매우 간단해진다. 코디네이터와 내비게이터에 대해 내가 좋아하는 점은 조합하기 좋고 다양한 범위와 객체에 내비게이션 로직을 분리하는 게 가능하다는 것이다.
다른 좋은 점은 이 방법들이 [non-optional optionals](https://www.swiftbysundell.com/posts/handling-non-optional-optionals-in-swift)를 제거할 수 있다는 점이다. 그로 인해 더 예측 가능하고 확신할 수 있는 코드가 된다.
