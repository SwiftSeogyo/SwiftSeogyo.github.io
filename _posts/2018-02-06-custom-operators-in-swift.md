# Custom Operation
> 원문 : [https://www.swiftbysundell.com/posts/custom-operators-in-swift](https://www.swiftbysundell.com/posts/custom-operators-in-swift)<br>
> kevin Kwon

커스텀 오퍼레이션의 사용만큰 뜨겁게 논쟁되는 스위프트의 기능은 거의없다.
몇몇 사람들이 이것들이 코드 상세를 줄이거나 가벼운 구문확장에 정말 유용하다고 생각하는 동안에 
다른 이들은 그들이 완전히 사용하지 않아야 한다고 생각한다.

좋든 싫든 우리가 존재하던 오퍼레이션을 오버로딩 하든 자신의 것으로 정의하든, 커스텀 오퍼레이션을 가지고 할 수 있는 재밌는 것들이 있다.

## Numeric containers
때때로 우리는 다른 것들이나 보다 원시적인값 컨테이너로써 밸류타입을 정의한다.
예를 들어 전략게임에서 작업중이다. 플레이어는 2가즤 종류의 리소스를 모을 수 있다. - 나무와 골드. 
이 자원을 코드로 모델링 하기위해서, 난 나무와 골드 밸류 쌍을 위한 컴테이너로 행동할 수 있는 리소스 구조체를 사용한다.

```swift
struct Resources {
    var gold: Int
    var wood: Int
}
```

내가 자원셋을 조회할 때마다, 난 이 구조체를 사용한다.(유져가 현재 사용가능한 자원을 추적하기위한 인스턴스로)

```swift
struct Player {
    var resources: Resources
}
```

게임에서 자원을 소비하는 방법으로 유닛을 훈련할 수 있다. 간단히 골드와 나무 비용을 제거할 수 있다.

```swift
func trainUnit(ofKind kind: Unit.Kind) {
    let unit = Unit(kind: kind)
    board.add(unit)

    currentPlayer.resources.gold -= kind.cost.gold
    currentPlayer.resources.wood -= kind.cost.wood
}
```

위 코드는 영향을 주는 일이 많을 수 있습니다.

* 자원을 한줄 빼먹거나
* 코드를 중복 작성해야합니다.
* 새로운 자원 (예: 실버)를 도입하는 것이 훨씬 어렵다.

모든 코드를 업데이트 해줘야 하기 때문이다.

# Operator overloading
현재 대부분의 언어는 operation overloading을 지원하고 있다. 두가지 방법이 있는데 기존의 operation을 재정의 하거나, 새로운 operation을 추가하는 것이다.

재정의 하는 경우

```swift
extension Resources {
    static func -=(lhs: inout Resources, rhs: Resources) {
        lhs.gold -= rhs.gold
        lhs.wood -= rhs.wood
    }
}
```

아래와 같이 처리할 수 있다.

```swift
currentPlayer.resources -= kind.cost
```

외부로직에서 항상 리소스 인스턴스 전체를 변경하기 원하기 때문에 모든 속성을 읽기전용으로 바꿀 수 있다.

```swift
struct Resources {
    private(set) var gold: Int
    private(set) var wood: Int

    init(gold: Int, wood: Int) {
        self.gold = gold
        self.wood = wood
    }
}
```

mutating function으로 구현할 수 도 있다. 

```swift
extension Resources {
    mutating func reduce(by resources: Resources) {
        gold -= resources.gold
        wood -= resources.wood
    }
}
```

# Layout calculations
연산자 오버로딩을 사용하는 것이 꽤 좋은 또 다른 시나리오를 살펴 보겠습니다.
자동 레이아웃과 강력한 레이아웃 앵커 API가 있지만 수동 레이아웃 계산이 필요할 때가 있습니다.

이와 같은 상황에서는 CGPoint, CGSize 및 CGVector와 같은 2 차원 값에 대해 수학을 수행해야하는 것이 일반적입니다. 예를 들어 이미지 뷰의 크기와 몇 가지 추가 여백을 사용하여 레이블의 출처를 계산해야 할 수도 있습니다.

```swift
label.frame.origin = CGPoint(
    x: imageView.bounds.width + 10,
    y: imageView.bounds.height + 20
)
```

기본 구성 요소를 사용하기 위해 항상 포인트와 크기를 확장해야하는 대신, 간단히 추가 할 수 있다면 좋지 않을까요? (우리가 Resources 구조체에서했던 것처럼)? 🤔

이를 수행하기 위해 + 연산자를 오버로드하여 두 개의 CGSize 인스턴스를 입력으로 받아들이고 CGPoint 값을 출력 할 수 있습니다.

```swift
extension CGSize {
    static func +(lhs: CGSize, rhs: CGSize) -> CGPoint {
        return CGPoint(
            x: lhs.width + rhs.width,
            y: lhs.height + rhs.height
        )
    }
}
```
그러면 이렇게 작성할 수 있다.

```swift
label.frame.origin = imageView.bounds.size + CGSize(width: 10, height: 20)
```

꽤 멋지지만 마진을 위해 CGSize를 만들어야하는 것이 다소 이상하다고 생각합니다. 조금 더 좋게 만드는 한 가지 방법은 다음과 같이 두 개의 CGFloat 값을 포함하는 크기와 튜플을 허용하는 또 다른 + 오버로드를 정의하는 것입니다.

```swift
extension CGSize {
    static func +(lhs: CGSize, rhs: (x: CGFloat, y: CGFloat)) -> CGPoint {
        return CGPoint(
            x: lhs.width + rhs.x,
            y: lhs.height + rhs.y
        )
    }
}
```
이제 2가지 방법을 쓸 수 있습니다.

```swift
// Using a tuple with labels:
label.frame.origin = imageView.bounds.size + (x: 10, y: 20)

// Or without:
label.frame.origin = imageView.bounds.size + (10, 20)
```

#A custom operator for error handling
지금까지 간단하게 기존 오퍼레이션에 추가하였습니다. 그러나 기존 오퍼레이션에 매핑 할 수 없는 기능을 위해 연산자를 사용하려면 먼저 자체 함수를 정의해야합니다.

다른 예를 살펴 보겠습니다. 스위프트는 오류 처리 메커니즘을 잡아내는 것이 유용합니다. 동기식 작업을 처리 할 때 좋습니다. 디스크에 저장된 모델을 로딩 할 때와 같이 오류가 발생하자마자 우리는 쉽게 안전 기능을 종료 할 수 있습니다.

```swift
class NoteManager {
    func loadNote(fromFileNamed fileName: String) throws -> Note {
        let file = try fileLoader.loadFile(named: fileName)
        let data = try file.read()
        let note = try Note(data: data)
        return note
    }
}
```

위와 같은 일을 할 때의 주된 단점은 우리 함수의 호출자에게 직접적으로 근본적인 오류를 던지고 있다는 것입니다. 첫 번째 블로그 게시물 인 "[Providing a unified Swift error API](https://www.swiftbysundell.com/posts/providing-a-unified-swift-error-api)"에서 API가 던질 수 있는 오류의 양을 줄이는 것이 좋습니다. 그렇지 않으면 의미있는 오류 처리 및 테스트가 실제로 어려워집니다.

이상적으로 우리가 원하는 것은 주어진 API가 던질 수 있는 유한한 오류 집합이고 이것은 각 사례를 개별적으로 쉽게 처리 할 수 있습니다. 우리는 또한 모든 근본적인 오류를 포착해서 우리에게 두 세계의 장점을 제공하고 싶다고 가정 해 봅시다. 그래서 우리는 명시적 케이스로 error enum을 정의합니다. 각각은 다음과 같이 기본 에러에 대한 연관된 값을 사용합니다

```swift
extension NoteManager {
    enum LoadingError: Error {
        case invalidFile(Error)
        case invalidData(Error)
        case decodingFailed(Error)
    }
}
```

그러나 기본 오류를 포착하고이를 자체 유형으로 변환하는 것은 더 까다 롭습니다. 표준 오류 처리 메커니즘 만 사용하면 다음과 같이 작성해야합니다.

```swift
class NoteManager {
    func loadNote(fromFileNamed fileName: String) throws -> Note {
        do {
            let file = try fileLoader.loadFile(named: fileName)

            do {
                let data = try file.read()

                do {
                    return try Note(data: data)
                } catch {
                    throw LoadingError.decodingFailed(error)
                }
            } catch {
                throw LoadingError.invalidData(error)
            }
        } catch {
            throw LoadingError.invalidFile(error)
        }
    }
}
```

위의 코드와 같이 코드를 읽는 사람은 없을 것이라고 생각합니다. 하나의 옵션은 수행 함수를 소개하는 것입니다 (위에서 언급 한 것처럼). 이것은 한 오류를 다른 오류로 변환하는 데 사용할 수 있습니다.

```swift
class NoteManager {
    func loadNote(fromFileNamed fileName: String) throws -> Note {
        let file = try perform(fileLoader.loadFile(named: fileName),
                               orThrow: LoadingError.invalidFile)

        let data = try perform(file.read(),
                               orThrow: LoadingError.invalidData)

        let note = try perform(Note(data: data),
                               orThrow: LoadingError.decodingFailed)

        return note
    }
}
```

더 낫네요 하지만 우리는 여전히 많은 오류 변환 코드가 우리의 실제 논리를 어지럽히는군요. 새로운 연산자를 도입하면이 코드를 약간 정리하는 데 도움이되는지 보겠습니다.

# Adding a new operator
새로운 연산자를 정의하는 것으로 시작하겠습니다. 이 경우 ~> 을 선택합니다 (이것은 대체 리턴 타입이라는 동기 부여와 함께, 그래서 우리는 -> 와 비슷한 것을 찾고있다). 이 연산자는 양면에서 작동하는 연산자이므로이를 다음과 같이 중위어로 정의합니다.

```swift
infix operator ~>
```

오퍼레이션를 매우 강력하게 만드는 이유는 오퍼레이션이 자동으로 양측의 컨텍스트를 캡처 할 수 있기 때문입니다. Swift의 @autoclosure 기능과 결합하면 우리는 아주 멋진 것을 만들 수 있습니다.

~>를 구현하자. throwing expression과 error transform을 취하고 원래 표현식과 동일한 유형을 던지거나 반환하는 연산자입니다.

```swift
func ~><T>(expression: @autoclosure () throws -> T,
           errorTransform: (Error) -> Error) throws -> T {
    do {
        return try expression()
    } catch {
        throw errorTransform(error)
    }
}
```

그렇다면 우리는 무엇을할까요? 연관된 값을 가진 enum 사례가 Swift의 정적 함수이기 때문에 간단하게 ~>; throwing expression과 기본 오류를 다음과 같이 변환하려는 오류 케이스 사이의 연산자를 추가할 수 있다.

```swift
class NoteManager {
    func loadNote(fromFileNamed fileName: String) throws -> Note {
        let file = try fileLoader.loadFile(named: fileName) ~> LoadingError.invalidFile
        let data = try file.read() ~> LoadingError.invalidData
        let note = try Note(data: data) ~> LoadingError.decodingFailed
        return note
    }
}
```

# Conclusion
사용자 정의 연산자와 연산자 오버로딩은 매우 흥미로운 솔루션을 구축 할 수있는 매우 강력한 기능입니다. 중첩된 함수 호출을 사용하지 않아도 자세한 정보를 줄일 수 있으므로 코드를보다 명확하게 처리 할 수 ​​있습니다. 그러나 미끄러운 경사면이기도하기 때문에 다른 개발자에게 매우 위협적이며 혼란스러워하는 코드를 읽기 어렵게 만들 수 있습니다.

보다 고급 방법으로 퍼스트 클래스 함수를 사용할 때와 마찬가지로 새로운 연산자를 도입하거나 추가 오버로드를 생성하기 전에 두 번 생각하는 것이 중요하다고 생각합니다. 다른 개발자로부터 피드백을 얻는 것은 또한 당신에게 완전한 감각을 줄 수있는 새로운 연산자가 다른 사람에게 완전히 외계인이라고 느낄 수도 있으므로 매우 가치가있을 수 있습니다. 많은 것들이 그렇듯이, 각 조건에 가장 적합한 도구를 선택하기 위해 절충점을 이해하고 해결하려고 노력합니다.

어떻게 생각해? 트위터 @johnsundell에 대한 질문이나 의견 또는 피드백이 있으시면 알려 주시기 바랍니다. 더 많은 콘텐츠를 보려면 카테고리 페이지로 가거나 Sundell Podcast의 Swift를 확인하십시오.

읽어 주셔서 감사합니다! 🚀