---
layout: post
title: "String API(Exploring the new String API in Swift4)"
date: 2018-03-20 10:00:00 +0900
author: William
categories: swift
---

*원문: [Exploring the new String API in Swift 4](https://www.swiftbysundell.com/posts/exploring-the-new-string-api-in-swift-4)*

---

# String API(Exploring the new String API in Swift 4)

## 멀티라인 리터럴

여러줄의 정적인 스크립트가 필요하거나 print() 출력에 개행을 추가하는 경우 여러번 호출해야 한다.

{% highlight swift %}
func printHelp() {
    print("🚘  Test Drive")
    print("--------------")
    print("Quickly try out any Swift pod or framework in a playground.")
    print("\nUsage:")
    print("- Simply pass a list of pod names or URLs that you want to test drive.")
    print("- You can also specify a platform (iOS, macOS or tvOS) using the '-p' option")
    print("- To use a specific version or branch, use the '-v' argument (or '-m' for master)")
    print("\nExamples:")
    print("- testdrive Unbox Wrap Files")
    print("- testdrive https://github.com/johnsundell/unbox.git Wrap Files")
    print("- testdrive Unbox -p tvOS")
    print("- testdrive Unbox -v 2.3.0")
    print("- testdrive Unbox -v swift3")
}
{% endhighlight %}

Swift3에서는 위와같이 print()를 여러번 호출한다.

다음은 Swift4

{% highlight swift %}
func printHelp() {
    print(
        """
        🚘  Test Drive
        --------------
        Quickly try out any Swift pod or framework in a playground.

        Usage:
        - Simply pass a list of pod names or URLs that you want to test drive.
        - You can also specify a platform (iOS, macOS or tvOS) using the '-p' option
        - To use a specific version or branch, use the '-v' argument (or '-m' for master)

        Examples:
        - testdrive Unbox Wrap Files
        - testdrive https://github.com/johnsundell/unbox.git Wrap Files
        - testdrive Unbox -p tvOS
        - testdrive Unbox -v 2.3.0
        - testdrive Unbox -v swift3
        """
    )
}
{% endhighlight %}

들여쓰기를 위해 맨 아래 """를 사용하며 문자열 내에 별도의 들여쓰기는 없다.

## 문자열은 컬렉션

Swift4에서 문자열은 다시 컬렉션이 되었습니다.
예를들어 문자열에서 특정 문자를 필터링 할 때 아래와 같이 간단하게 작성할 수 있다.

{% highlight swift %}
let filtered = string.filter { $0 != "!" }
{% endhighlight %}

## Substring

Swift4는 Substring타입을 사용하여 부분 문자열을 처리하는 방법을 제공한다.

{% highlight swift %}
// index에서 끝까지 자름
let substring = string[index...]
{% endhighlight %}

Swift3에서는 텍스트를 특정길이로 제한하여 문자열을 반환하려면 아래와 같이 작성한다.

{% highlight swift %}
extension String {
    func truncated() -> String {
        return String(characters.prefix(truncationLimit))
    }
}
{% endhighlight %}

Swift4에서도 위의 코드는 작동하지만 String은 문자열 컬렉션이기 때문에 직접 문자열에서 작업할 수 있다.

{% highlight swift %}
extension String {
    func truncated() -> Substring {
        return prefix(10)
    }
}
{% endhighlight %}

위에서 prefix()하위 시퀀스를 반환하는 API를 사용하며 n범위 검사도 수행한다.
따라서 인자보다 적은 수의 컬렉션이라도 오류가 발생하지 않는다.

위에서 truncated() 메서드의 리턴 타입은 Substring이다. 처음에는 문자열이 다른 유형을 반환하는 것이 번거로운 것처럼 보일 수 있지만 메모리 예측 가능성 측면에서 큰 이점을 제공한다.
Swift 문자열은 많은 중복 복사본을 만들 필요가 없도록 "copy on write"메서드를 사용하여 필요할 때만 복사본을 만들면 된다.
즉, 하위 문자열은 종종 부모 문자열과 함께 메모리에서 동일한 기본 버퍼를 공유한다.

Substring은 전체 string 대신 타입을 줌으로써 필요할 때 복사를 명시적으로 수행하여 부모 문자열의 메모리가 해제되도록 해준다.

다음 String과 같이 잘린 부분 문자열에서 새 문자열을 간단히 만들면 된다.

{% highlight swift %}
label.text = String(userInput.truncated())
{% endhighlight %}

## 결론

새로운 API가 정확성과 사용성이 용이해서 좋은 거래라고 생각한다.
또한 Substring 처럼 복사와 같은 의도적인 선택을 하도록 요구한다.
더 간단한 문자 관리를 위해 유니코드9 지원과 문자를 구성하는 기본 유니코드 포인트에 쉽게 액세스 할 수있는 기능과 같이
포스트에서 다루지 않는 문자열 API에 대한 발전이 있다.
