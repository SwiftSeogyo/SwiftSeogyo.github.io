---
layout: post
title:  "Core Animation gems: Using replicator layers in Swift"
date:   2018-02-13 10:00:00 +0900
categories: swift
---

*원문: [Core Animation gems: Using replicator layers in Swift](https://swiftbysundell.com/posts/ca-gems-using-replicator-layers-in-swift)*

---

# Core Animation gems: Using replicator layers in Swift

Core Animation은 CALayer단에서 더 낮은 수준의 랜더링을 수행할 수 있습니다.

## Playground liveView 사용하기

예제에 나오는 Playground liveView는 Assistant editor에서 Live View를 선택합니다.

liveView는 `PlaygroundPage.current.liveView`에 view를 셋팅해서 사용합니다.

liveView 랜더링을 종료하려면
PlaygroundPage.finishExecution를 호출합니다.

## CAReplicatorLayer

객체를 사용하여 위치, 회전 색상 및 시간에 영향을 줄 수있는 변형 규칙으로 복제 된 단일 소스 레이어를 기반으로 복잡한 레이아웃을 작성할 수 있습니다.

{% highlight swift %}
let replicatorLayer = CAReplicatorLayer()
replicatorLayer.frame.size = view.frame.size
replicatorLayer.masksToBounds = true

// 복제대상 layer
let imageLayer = CALayer()
imageLayer.contents = image.cgImage
imageLayer.frame.size = image.size
replicatorLayer.addSublayer(imageLayer)
{% endhighlight %}

## Replicator Instance

원본 layer에서 복제 layer로, 다시 복제 layer에서 또다른 복제 layer로 새로운 복제가 일어납니다.
그 때마다 복제의 기준의 되는 Instance를 기준으로 값 변화가 일어나게 됩니다.
InstanceCount를 제외한 나머지 속성이 위의 설명에 따라 작동합니다.

- instanceCount: 복제할 인스턴스 갯수
- instanceTransform
- instanceRedOffset
- instanceGreenOffset
- instanceBlueOffset
- instanceDelay

{% highlight swift %}
let instanceCount = view.frame.width / image.size.width
replicatorLayer.instanceCount = Int(instanceCount)

replicatorLayer.instanceTransform = CATransform3DMakeTranslation(
    image.size.width, 0, 0
)

let colorOffset = -1 / Float(replicatorLayer.instanceCount)
replicatorLayer.instanceRedOffset = colorOffset
replicatorLayer.instanceGreenOffset = colorOffset

let delay = TimeInterval(1)
replicatorLayer.instanceDelay = delay
{% endhighlight %}

## Animation 적용

Animation은 원본 Layer인 imageLayer에만 적용하면 됩니다.

{% highlight swift %}
let animation = CABasicAnimation(keyPath: "transform.scale")
animation.duration = 2
animation.fromValue = 1
animation.toValue = 0.1
animation.autoreverses = true
animation.repeatCount = .infinity
imageLayer.add(animation, forKey: "hypnoscale")
{% endhighlight %}

## 결론

CAReplicatorLayer는 일정한 반복패턴 animation을 rendering 하는데 사용 할 수 있습니다. 여러개의 UIView나 CALayer를 사용하지 않아서 더 나은 성능을 기대 할 수 있습니다.

완성 된 샘플코드는 [원문](https://swiftbysundell.com/posts/ca-gems-using-replicator-layers-in-swift)에서 확인 가능합니다.
