---
layout: post
title: 통합된 에러 API 만들기(Providing a unified Swift error API)"
date: 2018-03-13 10:00:00 +0900
author: Jay
categories: swift
---

*원문: [Providing a unified Swift error API](https://www.swiftbysundell.com/posts/providing-a-unified-swift-error-api)*

---

# 통합된 에러 API 만들기(Providing a unified Swift error API)

스위프트에서 API 요청으로 받는 에러를 제한하는 것은 굉장히 유용하다.
스위프트에는 구조화된 에러가 없어서 어떠한 에러라도 던져질 수 있다. 이것은 유연함과 동시에 불편함을 준다.

{% highlight swift %}
func loadSearchData(matching query: String) throws -> Data {
    let urlString = "https://my.api.com/search?q=\(query)"

    guard let url = URL(string: urlString) else {
        throw SearchError.invalidQuery(query)
    }

    return try Data(contentsOf: url)
}
{% endhighlight %}

위 예제에서 어떤 에러가 던져질 지 알 수 없다. `Data` 타입의 오류가 발생할 수 있다는 것도 알고 있어야 한다.
`SearchError` 타입의 오류만 던져진다고 할 수 있다면 더 좋을 것이다.

{% highlight swift %}
func loadSearchData(matching query: String) throws -> Data {
    let urlString = "https://my.api.com/search?q=\(query)"

    guard let url = URL(string: urlString) else {
        throw SearchError.invalidQuery(query)
    }

    do {
        return try Data(contentsOf: url)
    } catch {
        throw SearchError.dataLoadingFailed(url)
    }
}
{% endhighlight %}

이제 문서에 항상 `SearchError` 타입의 오류만 던져진다고 할 수 있고, 사용하기 쉬워졌다.
하지만 구현이 다소 복잡해졌다. 경우에 따라 여러번 `do-try-catch` 블록을 사용해야 한다. 그것은 코드를 읽기 어렵게 만든다.
이 문제를 해결하기 위해 다음 함수를 만들었다.

{% highlight swift %}
func perform(_ expression: @autoclosure () throws -> T,
                orThrow error: Error) throws -> T {
    do {
        return try expression()
    } catch {
        throw error
    }
}
{% endhighlight %}

`perform` 함수는 던져진 표현식을 실행하고 실패하면 지정한 에러를 던진다.

{% highlight swift %}
func loadSearchData(matching query: String) throws -> Data {
    let urlString = "https://my.api.com/search?q=\(query)"

    guard let url = URL(string: urlString) else {
        throw SearchError.invalidQuery(query)
    }

    return try perform(Data(contentsOf: url),
                       orThrow: SearchError.dataLoadingFailed(url))
}
{% endhighlight %}

이제 통합된 에러 API와 간단한 구현을 갖게 됐다.
