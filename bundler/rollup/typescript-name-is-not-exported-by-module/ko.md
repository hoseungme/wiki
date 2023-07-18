# 타압스크립트 번들링시 발생하는 Error: "[name] is not exported by [module]" 에러

오픈소스 코드를 Babel로 타입스크립트 코드를 트랜스파일링 하고, Rollup으로 번들링하는 도중 아래와 같은 에러가 발생했었다.

```
[!] RollupError: "ScrollerOptions" is not exported by "src/scroller.ts", imported by "src/index.ts".
```

[트러블슈팅 문서](https://rollupjs.org/troubleshooting/#error-name-is-not-exported-by-module)를 보니 export 되지 않은 모듈을 import 하려고 시도할 때 발생하는 문제이다.

당시 `src/index.ts`는 아래와 같이 작성되어 있었는데,

```typescript
// src/index.ts

import { Scroller, ScrollerOptions, ScrollEvent, TouchScroller } from "./scroller";

class FlickableScroller {
  private readonly scroller: Scroller;

  constructor(container: HTMLElement, options?: ScrollerOptions) {
    this.scroller = new TouchScroller(container, options);
  }

  public destroy() {
    this.scroller.destroy();
  }
}

export { FlickableScroller, ScrollerOptions, ScrollEvent };
```

당연히 `src/scroller.ts` 에서는 저 모든 것들을 export 하고 있었기에 무엇이 문제인가 싶었다.

알고보니 문제는 `ScrollerOptions`, `ScrollEvent`가 타입이라는 것에 있었다.

```typescript
// src/scroller.ts

export interface ScrollerOptions {
  direction?: "x" | "y";
  reverse?: boolean;
  onScrollStart?: (e: ScrollEvent) => void;
  onScrollMove?: (e: ScrollEvent) => void;
  onScrollEnd?: (e: ScrollEvent) => void;
}

export interface ScrollEvent {
  position: number;
  minPosition: number;
  maxPosition: number;
  minOverflowPosition: number;
  maxOverflowPosition: number;
  isScrollTop: boolean;
  isScrollBottom: boolean;
}
```

Babel은 Single File Transpiler로, 한 번에 하나의 파일만 처리하는 도구이다. 이는 다시 말해 특정 파일에서 import하고 있는 모듈이 타입인지 알 수 없다는 뜻이다.

따라서 위의 `src/index.ts` 파일의 import/export 부분을 다시 보면,

```typescript
import { Scroller, ScrollerOptions, ScrollEvent, TouchScroller } from "./scroller";

/* ... */

export { FlickableScroller, ScrollerOptions, ScrollEvent };
```

Babel 입장에서는 `src/index.ts`를 transpile 할 때, `ScrollerOptions`, `ScrollEvent`이 타입인지 아닌지 모르는 상황에서 re-export 되고 있으므로 지울 수가 없다.

하지만 `src/scroller.ts`를 transpile 할 때는 타입이라는 것을 알고있기 때문에 지워버리게 된다. 결국 `src/scroller.ts`에서 export하지 않은 모듈을 `src/index.ts`가 import하게 되므로 에러가 발생하게 된다.

따라서 이러한 문제를 해결하기 위해서는, `import type` 또는 `export type` 문을 사용하면 된다.

```typescript
import { Scroller, TouchScroller } from "./scroller";
import type { ScrollerOptions, ScrollEvent } from "./scroller";

/* ... */

export { FlickableScroller, ScrollerOptions, ScrollEvent };
```

```typescript
import { Scroller, ScrollerOptions, ScrollEvent, TouchScroller } from "./scroller";

/* ... */

export { FlickableScroller };

export type { ScrollerOptions, ScrollEvent };
```

이러면 import/export된 모듈이 타입이라는 것을 Babel이 인지하고 삭제할 수 있기 때문에 에러가 발생하지 않는다.
