# 가짜 import/export

Node.js 모듈 시스템의 종류를 정리한 문서에서, CJS는 `require` / `module.exports`를 사용한다고 했다. 그런데 타입스크립트에서는 어떻게 CJS여도 `import` / `export` 문을 사용하는 것일까?

그 이유는 CJS 타입스크립트에서 사용하는 `import` / `export` 문은 ESM 스펙을 흉내내기 위한 가짜이기 때문이다.

예를 들어 아래와 같이 작성된 CJS 타입스크립트 코드는:

```typescript
import { foo } from "./foo";

export const fooBar = foo + "bar";
```

자바스크립트로 트랜스파일하면 대략 아래와 같이 기존의 CJS 문법을 쓰도록 바뀐다.

```javascript
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.fooBar = void 0;
var foo_1 = require("./foo");
exports.fooBar = foo_1.foo + "bar";
```
