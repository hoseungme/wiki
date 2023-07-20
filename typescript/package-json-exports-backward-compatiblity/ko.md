# package.json의 exports 사용시 하위 호환성 지원

토스에서 일하며 프론트엔드 라이브러리 오픈소스인 Slash의 메인테이너로 활동하면서 [이런 PR](https://github.com/toss/slash/pull/247)이 올라왔었다.

내용을 요약하면, `@toss/storage/typed`를 import하면 `cannot find modules` 에러가 난다는 것이었다.

처음에는 순간 이해가 가지 않았다. 분명히 package.json의 `exports`를 아래와 같이 잘 명시해 두었고, 내 로컬에서는 문제가 없었기 때문이다.

```json
{
  "exports": {
    ".": {
      "require": {
        "types": "./dist/index.d.ts",
        "default": "./dist/index.js"
      },
      "import": {
        "types": "./esm/index.d.mts",
        "default": "./esm/index.mjs"
      }
    },
    "./typed": {
      "require": {
        "types": "./dist/typed/index.d.ts",
        "default": "./dist/typed/index.js"
      },
      "import": {
        "types": "./esm/typed/index.d.mts",
        "default": "./esm/typed/index.mjs"
      }
    }
  }
}
```

그런데 잘 생각해보니, 타입스크립트가 `exports`를 기준으로 모듈 declaration을 찾는 것은 tsconfig의 `moduleResolution`이 `nodenext` 또는 `node16`일 때만 동작한다. 즉, 다른 값으로 설정한 유저는 기존 동작 대로 file system에 기반하여 모듈을 찾는다는 뜻이므로, file system 상으로는 `./typed.ts`도 `./typed.d.ts`도 없으니 resolution에 실패한 것. ([코멘트](https://github.com/toss/slash/pull/247#issuecomment-1575288572))

따라서 패키지 루트에 아래와 같이 `./typed.d.ts`를 추가하고, package.json의 `files` 목록에 추가하여 빌드 결과물과 함께 npm에 publish하면 문제가 해결된다. ([코멘트](https://github.com/toss/slash/pull/247#issuecomment-1575288949))

```typescript
// typed.d.ts
export * from "./dist/typed";
```

```json
{
  "files": ["dist", "esm", "typed.d.ts"]
}
```

정리하면, package.json exports를 사용해 subpath export를 사용할 땐, file system 기반으로 모듈을 찾는 타입스크립트 하위 버전에 대한 호환성 지원을 해주자.
