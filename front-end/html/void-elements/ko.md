# Void Elements

HTML Element 중에서 content, 즉 children을 갖지 않는 엘리먼트를 void element 라고 한다.

예를 들면,

- `<br>`
- `<img>`
- `<input>`
- ...

등등이 있다.

대부분의 프론트엔드 개발자들이 React의 JSX 문법에 익숙하기 때문에 void element를 아래와 같이 작성할 것이다.

```jsx
<img src="" alt="" />
```

`/>` 와 같이 작성하는 것을 자기 스스로 닫는다고 하여 self closing tag 라고 한다. 이커머스 팀에서 판매자용 어드민에 HTML 입력 기능을 추가하면서 배운 내용인데, 사실 이는 HTML에는 없는 문법이라고 한다. HTML 파서는 그것을 무시하고 DOM을 생성한다.

다만 prettier 같은 일부 코드 포맷팅 도구는 일부러 self closing tag를 붙여주기도 한다. 이유는 모르겠지만 편의상인듯.
