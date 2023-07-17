# Element의 크기 변경을 감지하기: ResizeObserver

[flickable-scroll](https://github.com/HoseungJang/flickable-scroll) 이라는 오픈소스를 만들면서, 최대 스크롤 길이 등을 구해야 하기 때문에 엘리먼트 사이즈가 바뀌면 업데이트를 해줘야 하는 요구사항이 있었다.

처음에는 아래와 같이 resize 이벤트를 등록해 봤는데, resize 이벤트는 윈도우의 크기 변경에 트리거되는 것이었고, 엘리먼트 자체의 크기 변화에는 트리거되지 않았다.

```typescript
element.addEventListener("resize", handler);
```

알아보니 이럴 때 사용할 수 있는 것으로 `ResizeObserver`라는 API가 있었다. 단순 크기 변화만 알려주는 것이 아니고, content box, border box의 크기 등 꽤 자세한 정보를 준다.

```typescript
const observer = new ResizeObserver((entries) => /* ... */);

observer.observe(element);
```

또한 여러 개의 엘리먼트를 observe할 수 있다.

```typescript
const observer = new ResizeObserver((entries) => /* ... */);

Array.prototype.slice.call(element.children).forEach((child) => observer.observe(child));
```
