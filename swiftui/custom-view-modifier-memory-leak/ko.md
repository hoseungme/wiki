# Custom view modifier memory leak

Custom view modifier를 사용하는 경우, 예상하기 힘들고 찾기도 힘든 memory leak이 굉장히 많이 발생한다.

예를 들어, onTapGesture에서 closure의 메모리 해제가 안되는 버그가 있다. 아래 예시 코드를 보자.

```swift
import Combine
import SwiftUI

struct ContentView: View {
    @State private var toggle: Bool = false

    var body: some View {
        VStack {
            Button {
                self.toggle.toggle()
            } label: {
                Text("Toggle")
            }
            ModelView(model: .init(id: toggle ? "1" : "2"))
        }
    }
}

struct ModelView: View {
    let model: LeakModel

    var body: some View {
        Text("Hello, world!")
            .padding()
            .modifier(Modifier(action: {
                print(self.model.id)
            }))
    }
}

class LeakModel: ObservableObject {
    let id: String

    init(id: String) {
        self.id = id
    }

    deinit {
        print("deinit")
    }
}

struct Modifier: ViewModifier {
    let action: () -> Void

    func body(content: Content) -> some View {
        content.onTapGesture {
            self.action()
        }
    }
}
```

여기서 내 의도는 당연히 ContentView의 Toggle Button을 클릭해서 State가 변경되면, LeakModel이 새롭게 생성되면서 이전 model은 deinit 되는 것이다.

하지만 위 예제에서는 State가 바뀐 순간에 deinit이 되지 않는다. 왜냐하면, 위처럼 onTapGesture를 custom view modifier 안에서 단독으로 사용하는 경우, 실제로 tap gesture가 발생하기 전까진 onTapGesture에 넘긴 closure가 메모리에서 사라지지 않는 버그가 있다.

따라서 로직 정리한답시고 custom view modifier 만들지 말고, 웬만해선 view 레벨에서 해결하는게 좋다.

이렇게 view에서 직접 사용하던가,

```swift
struct ModelView: View {
    let model: LeakModel

    var body: some View {
        Text("Hello, world!")
            .padding()
            .onTapGesture {
                print(self.model.id)
            }
    }
}
```

아니면 아래처럼 새로운 view를 만들던가.

```swift
struct ModelView: View {
    let model: LeakModel

    var body: some View {
        WithTapGesture {
            print(self.model.id)
        } label: {
            Text("Hello, world!")
        }
    }
}
```
