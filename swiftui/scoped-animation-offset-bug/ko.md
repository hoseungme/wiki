# Scoped Animation .offset 버그

SwiftUI에서 아래처럼 scoped animation 안에서 .offset을 사용하면 애니메이션이 걸리지 않는다.

```swift
SomeView()
    .animation(.spring) {
        $0.offset(y: isHover ? -2 : 0)
    }
```

이는 SwiftUI 내부 구현상 버그라고 오피셜 답변이 있다. 그래서 어떻게 해결해야 할지 회사에서 동료랑 삽질을 정말 엄청나게 했는데.. 내장 offset을 쓰지 말고, 그냥 직접 `GeometryEffect`로 offset을 구현하면 애니메이션이 잘 먹힌다.

```swift
public extension View {
    func projectionOffset(x: CGFloat = 0, y: CGFloat = 0) -> some View {
        self.projectionOffset(.init(x: x, y: y))
    }

    func projectionOffset(_ translation: CGPoint) -> some View {
        modifier(ProjectionOffsetEffect(translation: translation))
    }
}

private struct ProjectionOffsetEffect: GeometryEffect {
    var translation: CGPoint
    var animatableData: CGPoint.AnimatableData {
        get { translation.animatableData }
        set { translation = .init(x: newValue.first, y: newValue.second) }
    }

    public func effectValue(size: CGSize) -> ProjectionTransform {
        .init(CGAffineTransform(translationX: translation.x, y: translation.y))
    }
}
```

```swift
SomeView()
    .animation(.default) {
        $0
            .opacity(animate ? 1 : 0.2)
            // .offset(y: animate ? 0 : 100) // 애니메이션 안됨
            .projectionOffset(y: animate ? 0 : 100) // 애니메이션 됨
    }
```

그냥 내장 offset이랑 똑같은 일을 해주는 건데.. 거기에는 다른 구현이 더 있어서 버그가 나나보다.
