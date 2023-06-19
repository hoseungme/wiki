# 모듈 시스템

Node.js에 존재하는 모듈 시스템은 크게 CommonJS, ECMAScript Modules(이하 CJS, ESM) 두 가지이다.

## CJS

```javascript
// add.js
module.exports.add = (x, y) => x + y;
// main.js
const { add } = require("./add");
add(1, 2);
```

## ESM

```javascript
// add.js
export function add(x, y) {
  return x + y;
}
// main.js
import { add } from "./add.js";
add(1, 2);
```

- CJS는 `require` / `module.exports` 를 사용하고, ESM은 `import` / `export` 문을 사용한다.
- CJS module loader는 동기적으로 작동하고, ESM module loader는 비동기적으로 작동한다.
  - ESM은 [Top-level Await](https://nodejs.org/api/esm.html#top-level-await)을 지원하기 때문에 비동기적으로 동작한다.
- 따라서 ESM에서 CJS를 import 할 수는 있지만, CJS에서 ESM을 require 할 수는 없다. 왜냐하면 CJS는 Top-level Await을 지원하지 않기 때문.
- 이 외에도 두 Module System은 기본적으로 동작이 다르다.
- 따라서 두 Module System은 서로 호환되기 어렵다.
