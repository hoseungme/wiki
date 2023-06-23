# ESM에서 @emotion/styled를 import하면 생기는 현상

`@emotion/styled`를 `import`해서 사용하던 타입스크립트 코드가 따르는 모듈 시스템을 CJS에서 ESM으로 바꾸었더니 타입 에러가 발생했다.

```typescript
import styled from "@emotion/styled";

styled.div``; // Property 'div' does not exist on ...
```

이때 위 코드를 아래와 같이 바꾸면 타입 에러가 사라지는데,

```typescript
import styled from "@emotion/styled";

styled.default.div``;
```

그 이유는 [가짜 import/export 문서](../fake-import-export/ko.md)와 연관되어 있었다. `@emotion/styled` 에서는 `styled` 객체를 `default export` 하고 있었는데, 문제는 CJS에서 `default export`는 없는 스펙이라는 것이다.

따라서 `default export`는 아래와 같이 `default`로 named export되는 것으로 트랜스파일된다.

```typescript
export default styled;
```

```javascript
module.exports = { default: styled };
```

따라서 위와 같은 가짜 `export` 문에 맞춰 추론하지 않아도 되는 ESM에서 위와 같은 문제가 발생했던 것.

## 해결 방법

이럴 때 해결 방법은 두 가지인데, 첫 번째는 `export default`가 아니라 `export =`를 사용하는 것이다.

```typescript
export = styled;
```

두 번째는 아래와 같이 re-export 하는 것이다. 첫 번째 방법대로 하려면 emotion에 직접 PR 올려야 하므로, 두 번째 방법으로 해결하는게 훨씬 간단했다.

```typescript
import _styled from "@emotion/styled";

const styled = _styled.default;

export default styled;
```
