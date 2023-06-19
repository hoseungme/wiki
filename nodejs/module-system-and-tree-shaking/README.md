# 모듈 시스템과 트리 쉐이킹

[이 문서](../module-system/README.md)에 CJS, ESM에 대해 정리했는데, CJS와 ESM은 프론트엔드의 트리 쉐이킹과도 밀접한 관련이 있다.

결론부터 말하면 CJS는 트리 쉐이킹이 어렵고, ESM은 트리 쉐이킹이 쉽다.

왜냐하면 아래와 같이 CJS는 `require` / `module.exports`를 동적으로 하는 것에 제약이 없다.

```javascript
// require
const utilName = /* 동적인 값 */
const util = require(`./utils/${utilName}`);

// module.exports
function foo() {
  if (/* 동적인 조건 */) {
    module.exports = /* ... */;
  }
}
foo();
```

따라서 CJS는 빌드 타임에 정적 분석을 하기가 어렵고, 런타임에서만 모듈 관계를 파악할 수 있다.

반면 ESM은 모듈 간 정적인 구조로 의존하도록 강제하고 있다. `import`에 동적인 값을 사용할 수 없고, `export`는 항상 최상위에서만 사용할 수 있다.

```javascript
import util from `./utils/${utilName}.js`; // 불가능

import { add } from "./utils/math.js"; // 가능

function foo() {
  export const value = "foo"; // 불가능
}

export const value = "foo"; // 가능
```

따라서 빌드 타임에 정적 분석을 통해 모듈 관계를 파악할 수 있어 트리 쉐이킹이 쉬운 것이다.
