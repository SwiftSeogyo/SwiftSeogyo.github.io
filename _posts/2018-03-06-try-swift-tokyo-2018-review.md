---
layout: post
title: "try! Swift Tokyo 2018 참관기"
date: 2018-03-06 10:00:00 +0900
author: Jay
categories: swift
---

*공식 페이지: [TOKYO - try! Swift Conference](https://www.tryswift.co/events/2018/tokyo/en/)*

---

# try! Swift Tokyo 2018 참관기

2018년 3월 1~3일 도쿄에서 있었던 try! Swift 콘퍼런스에 다녀왔습니다.
간략하게 정보를 공유하겠습니다.

## 3월 1일

- [A Secret Swift Tour](https://www.tryswift.co/events/2018/tokyo/en/#swift-tour) [[slide](https://speakerdeck.com/ezura/secret-swift-tour)] - 지각으로 못 들었다.
- ⚡️🎤 [SIL for First Time Learners](https://www.tryswift.co/events/2018/tokyo/en/#sil) [[slide](https://www.slideshare.net/kitasuke/sil-for-first-time-leaners)] - 스위프트 컴파일러는 여러 단계를 거쳐 실행되는데 SIL(Swift Intermediate Language)도 그 중 하나. 컴파일러 옵션을 통해 SIL을 확인할 수 있고 스위프트가 어떻게 최적화 되는 지 알 수 있다.
- [Exploring Clang Modules](https://www.tryswift.co/events/2018/tokyo/en/#clang) [[slide](https://speakerdeck.com/segiddins/exploring-clang-modules)] - 스위프트와 오브젝티브-씨가 모듈을 읽는 방법을 소개. 몰라도 개발하는데 문제는 없다고 생각.
- [Getting to Know the Responder Chain](https://www.tryswift.co/events/2018/tokyo/en/#responder-chain) - 리스폰더 체인의 동작을 알려주고, 확장을 통해 개발에 도움이 되는 팁 소개. 나중에 찾아보자.
- [Optimizing Swift code for separation of concerns and simplicity](https://www.tryswift.co/events/2018/tokyo/en/#simplicity) [[slide](https://speakerdeck.com/javisoto/try-swift-tokyo-2018-optimizing-swift-code-for-separation-of-concerns-and-simplicity)] - 관심사의 분리를 위한 방법 소개. 꿀팁이 많으므로 보고 또 보자.
- [Should coders design?](https://www.tryswift.co/events/2018/tokyo/en/#coders-design) [[slide](https://speakerdeck.com/zats/should-coders-design)] - 디자이너가 코딩하는 것보다 코더가 디자인하는 게 낫다. 그 이유 소개. 같은 코더로서 공감됨.
- [Event driven networking for Swift](https://www.tryswift.co/events/2018/tokyo/en/#networking) [[GitHub](https://github.com/apple/swift-nio)] - SwiftNIO 소개. 애플 개발자가 WWDC가 아닌 콘퍼런스에서 발표하는 건 이래적인 듯. SwiftNIO 는 저수준의 이벤트 드리븐 네트워크 라이브러리다. 많은 환호를 받았다. 굉장히 좋을 것 같지만 저수준이라는 것에도 주목하자.
- ⚡️🎤 [The diamond of variance](https://www.tryswift.co/events/2018/tokyo/en/#diamond) [[slide](https://speakerdeck.com/dtvd/the-diamond-of-variances)] - Variance, Covariance, Contravariance 소개. 제네릭에 대한 깊이 있는 내용. 기반 지식이 없어서 전혀 못 알아 들었다. 나중에 천천히 살펴보자.
- [SwiftyPi](https://www.tryswift.co/events/2018/tokyo/en/#swiftypi) [[slide](https://speakerdeck.com/kcastellano/swiftypi)] - 라즈베리 파이에서 스위프트로 개발하기.
- ⚡️🎤 [Swift in my home](https://www.tryswift.co/events/2018/tokyo/en/#home) [[slide](https://speakerdeck.com/yukiasai/swift-in-my-home)] - 자신이 아이들을 위해 만든 서비스 구성을 소개. 업무가 아닌 생활에서 스위프트를 실용적으로 사용한 게 좋았다.
- [UI Testing for Fun and Profit](https://www.tryswift.co/events/2018/tokyo/en/#ui-testing) [[slide](https://speakerdeck.com/saraheolson/ui-testing-for-fun-and-profit)] - 트렐로에서 한 테스트 경험을 소개.
- [Writing Blockchain Clients in Swift](https://www.tryswift.co/events/2018/tokyo/en/#blockchain) [[slide](https://speakerdeck.com/tamarnachmany/writing-blockchain-clients-in-swift)] - 이더리움 라이브러리 소개.
- ⚡️🎤 [Protocol Oriented WebAPI Abstraction](https://www.tryswift.co/events/2018/tokyo/en/#webapi) - 웹 API를 프로토콜 지향으로 구현한 방법을 소개. 자세한 기억은 나지 않지만 코드가 깔끔해서 좋았다.
- [👾](https://www.tryswift.co/events/2018/tokyo/en/#game) [[slide](https://speakerdeck.com/giginet/-11)] - 스위프트로 쉽게 게임을 만들 수 있다. 하지만... 발표자가 인기인. 직접 만든 인디 게임도 소개. 재미있었다.
- [AST Meta-programming](https://www.tryswift.co/events/2018/tokyo/en/#ast) [[slide](https://speakerdeck.com/kishikawakatsumi/ast-meta-programming-in-swift)] - AST(Abstract Syntax Tree)도 SIL과 마찬가지로 스위프트 컴파일 중간 단계이다. AST를 사용하면 오브젝티브-씨처럼 동적 프로그래밍을 할 수 있다.

## 3월 2일

- [Finally Solving the Expression Problem](https://www.tryswift.co/events/2018/tokyo/en/#proofs) [[slide](https://bkase.github.io/slides/no-problemo/#/)] - 지각으로 못 들었다.
- ⚡️🎤 [Swift Peer Lab Barcelona](https://www.tryswift.co/events/2018/tokyo/en/#peerlabs) [[slide](https://speakerdeck.com/tiagomartinho/swift-peer-lab)] - 지각으로 못 들었다.
- [Using Swift to Visualize Algorithms](https://www.tryswift.co/events/2018/tokyo/en/#visualize-algorithms) [[slide](https://speakerdeck.com/subdigital/bezier-curves)][[GitHub](https://github.com/subdigital/visualizing-bezier-curves)] - 들을 수 있었지만 브레이크 타임이 얼마남지 않아서 바리스타가 내려준 커피를 마시며 부스를 구경했다.
- [Codable Routing with Kitura](https://www.tryswift.co/events/2018/tokyo/en/#kitura) - 키투라에서 코더블을 어떻게 사용했는지 소개.
- ⚡️🎤 [Super Resolution with CoreML](https://www.tryswift.co/events/2018/tokyo/en/#coreml) [[slide](https://speakerdeck.com/kenmaz/super-resolution-with-coreml-at-try-swift-tokyo-2018)] - DeNA에서 CoreML을 사용해 만화의 용량은 줄이며 화질 개선을 시도한 사례 공유. 소스도 공개 큰 박수를 받았다.
- [Introducing Charles for iOS](https://www.tryswift.co/events/2018/tokyo/en/#charles) - 네트워크에 대한 상세한 정보를 알 수 있는 Charles for iOS 소개. 유용해 보인다.
- [Designing Experiences With Augmented Reality](https://www.tryswift.co/events/2018/tokyo/en/#ar) - AR 앱을 디자인 할 때 고려할 점을 공유. https://www.dropbox.com/s/i3smw7ciy302rb6/trySwift-Final.pdf?dl=0 http://davidhoang.com/tryswift-2018/
- [Kotlin For Swift Developers](https://www.tryswift.co/events/2018/tokyo/en/#kotlin) [[slide](https://speakerdeck.com/designatednerd/kotlin-for-swift-developers-try-swift-tokyo-march-2018)]- 개인 사정으로 듣지 못했다.
- ⚡️🎤 [Preparing for Swift 5 Ownership](https://www.tryswift.co/events/2018/tokyo/en/#swift5) [[slide](https://speakerdeck.com/kotetuco/preparing-for-swift-5-ownership)] - 스위프트 5에서 변경된 오너십 관련 내용 요약.
- [Digital Signal Processing with Swift](https://www.tryswift.co/events/2018/tokyo/en/#signal-processing) [[slide](https://speakerdeck.com/daisyramos317/digital-signal-processing-with-swift)] - 애플의 프레임워크를 통해 DSP를 구현하는 방법 소개.
- ⚡️🎤 [The Type-Safe World of Codable](https://www.tryswift.co/events/2018/tokyo/en/#codable) [[slide](https://speakerdeck.com/tattn/the-type-safe-world-of-codable)] - 코더블 안내.
- [Creating conversational interfaces in iOS/Swift](https://www.tryswift.co/events/2018/tokyo/en/#conversational-interfaces) [[slide](https://speakerdeck.com/wendylu/conversational-interfaces-in-ios)] - 개인적으로 관심이 있었는데 배탈이 나서 듣지 못했다.
- [UIImageView vs Metal](https://www.tryswift.co/events/2018/tokyo/en/#uiimageview-metal) [[slide](https://www.slideshare.net/t26v0748/uiimageview-vs-metal-89418399)] - 메탈은 얼마나 빠를까? 결론만 말하면 메탈을 제대로 쓰기 위해서는 신경 쓸 게 많고, UIKit 내부에서 최적화된 메탈을 사용하므로 그냥 UIKit을 쓰는 게 낫다. 결론은 간단하지만 이를 위한 발표자의 X고생에 찬사를 보낸다.
- ⚡️🎤 [Best Docker Container in Swift](https://www.tryswift.co/events/2018/tokyo/en/#docker) [[slide](https://speakerdeck.com/nonchalant/try-swift-tokyo-2018-best-docker-container-in-swift)] - 몇 가지 환경의 도커 이미지 벤치마크.
- ⚡️🎤 [The Type Erasure Advantage](https://www.tryswift.co/events/2018/tokyo/en/#typeerasure) [[GitHub](https://github.com/tarunon/try-erasure)] - 기반 지식이 없어서 이해가 가지 않았다.
- ⚡️🎤 Make faces big by Vision and CoreGraphics [[slide](https://speakerdeck.com/narujpn/make-faces-big-by-vision-and-coregraphics)] - 예정에 없어지만 추가된 라이트닝 토크. 데모가 재미있었다.
- [Investing time into developer tools and experience](https://www.tryswift.co/events/2018/tokyo/en/#tools) - 다양한 도구를 사용해 좋은 개발자로 성장하자. 좋은 내용이 많았지만 과연 다 할 수 있을지 모르겠다.

## 3월 3일

나는 [Building real-world server-side Swift applications with Kitura](https://www.tryswift.co/events/2018/tokyo/en/#kitura-workshop)에 참석했다. 다음 깃허브에서 튜토리얼을 따라하면 된다. [[GitHub](https://github.com/IBM/FoodTrackerBackend)]

---

## 참고

- 모든 발표 내용 캡처 및 요약 [http://niwatako.hatenablog.jp/archive](http://niwatako.hatenablog.jp/archive)
- 링크 정리 [https://qiita.com/ozwio/items/71fb765b48905d6a2193](https://qiita.com/ozwio/items/71fb765b48905d6a2193)
