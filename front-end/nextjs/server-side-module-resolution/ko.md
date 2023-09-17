# Next.js가 서버 사이드에서 모듈을 읽는 방식

회사 업무 중에 이런 문제가 있었다.

- 서비스에서 라이브러리 A, B를 사용하고 있다. A, B 모두 CJS, ESM 둘 다 지원한다.
- B 내부적으로 A를 사용하고 있는데, A의 React Context를 공유해야 하기 때문에, B는 A를 peer dependency로 두었다.
- 그런데 이때 Next.js 서버에서 A는 ESM 빌드를 불러왔지만, B는 CJS 빌드를 불러왔다.
- 따라서 서비스에서는 A의 ESM 빌드를 사용하지만, B에서는 A의 CJS 빌드를 사용하게 된다.
- 결국 서비스에서 사용한 A의 React Context와 B에서 사용한 A의 React Context는 서로 달라지게 되고, 에러가 발생한다.

실제로 `next build` 커맨드의 결과물에서 서버 빌드를 살펴봤는데, A는 ESM으로 읽고, B는 CJS로 읽고 있었다.

```javascript
import("A");
require("B");
```

그래서 어떤 기준으로 모듈을 읽는 방식이 결정되는 것인지 알아보기 위해, Next.js의 코드를 직접 열어보기로 했다.

우선 Next.js는 [내부적으로 webpack을 사용](https://github.com/vercel/next.js/blob/0bfd4801e4293d7a484e6c2f40fda45cce09cbb5/packages/next/src/build/webpack-config.ts)하고 있었다. 이때 webpack은 외부 모듈(즉, 라이브러리)을 어떻게 해석할지 externals 옵션을 통해 아래와 같이 설정할 수 있다.

```javascript
module.exports = {
  // ...
  externals: [
    "commonjs A", // 외부 모듈 A를 CJS로 불러온다.
    "module B", // 외부 모듈 B를 ESM으로 불러온다.
  ],
  // ...
};
```

결론은 Next.js가 [서버 코드를 빌드할 때 externals 옵션을 어떻게 설정하는지](https://github.com/vercel/next.js/blob/0bfd4801e4293d7a484e6c2f40fda45cce09cbb5/packages/next/src/build/webpack-config.ts#L1624-L1696)를 확인하면 답을 알 수 있었다.

webpack의 externals 옵션에는 string 뿐만 아니라 function도 넘겨줄 수 있는데, Next.js에서는 아래와 같은 function을 넘겨주고 있었다.

```typescript
({
  context,
  request,
  dependencyType,
  contextInfo,
  getResolve,
}: {
  context: string;
  request: string;
  dependencyType: string;
  contextInfo: {
    issuer: string;
    issuerLayer: string | null;
    compiler: string;
  };
  getResolve: (
    options: any
  ) => (
    resolveContext: string,
    resolveRequest: string,
    callback: (err?: Error, result?: string, resolveData?: { descriptionFileData?: { type?: any } }) => void
  ) => void;
}) =>
  handleExternals(context, request, dependencyType, contextInfo.issuerLayer as WebpackLayerName, (options) => {
    const resolveFunction = getResolve(options);
    return (resolveContext: string, requestToResolve: string) =>
      new Promise((resolve, reject) => {
        resolveFunction(resolveContext, requestToResolve, (err, result, resolveData) => {
          if (err) return reject(err);
          if (!result) return resolve([null, false]);
          const isEsm = /\.js$/i.test(result)
            ? resolveData?.descriptionFileData?.type === "module"
            : /\.mjs$/i.test(result);
          resolve([result, isEsm]);
        });
      });
  });
```

여기서 externals 옵션에 넘긴 function의 파라미터들 중 `getResolve`가 중요한데, 이녀석은 모듈이 실제로 어떤 경로로 resolve되는지를 알려주는 function을 리턴한다. 예를 들면 아래와 같다.

```javascript
// 코드가 이렇게 작성되있다고 가정했을 때,
const A = require("A");

// getResolve를 사용해 "A"라는 require path가 실제로 어떤 모듈로 resolve 되는지를 알아낼 수 있다.
const resolveFunction = getResolve(options);
resolveFunction(context, "A", (err, result) => {
  console.log(result); // node_modules/A/dist/index.js
});
```

그리고 Next.js에서는 `handleExternals`라는 function을 한번 더 거쳐서 값을 리턴하게 되는데, `handleExternals`의 5번째 파라미터로 function을 넘겨주고 있었다.

```typescript
handleExternals(context, request, dependencyType, contextInfo.issuerLayer as WebpackLayerName, (options) => {
  const resolveFunction = getResolve(options);
  return (resolveContext: string, requestToResolve: string) =>
    new Promise((resolve, reject) => {
      resolveFunction(resolveContext, requestToResolve, (err, result, resolveData) => {
        if (err) return reject(err);
        if (!result) return resolve([null, false]);
        const isEsm = /\.js$/i.test(result)
          ? resolveData?.descriptionFileData?.type === "module"
          : /\.mjs$/i.test(result);
        resolve([result, isEsm]);
      });
    });
});
```

이때 위에서 설명한 `getResolve`를 사용해 모듈이 ESM인지에 대한 여부를 함께 리턴하고 있었다.

여기서 `getResolve`가 리턴한 `resolveFunction`의 3번째 파라미터에서 `resolveData`도 받는 것을 볼 수 있었는데, `isESM` 값을 구하는 부분을 살펴보면, `resolveData`에서 `descriptionFileData`를 읽는 것을 볼 수 있었다.

이녀석의 정체가 궁금해져서, 결국 webpack 코드까지 까보기로 했다.

webpack은 WebpackOptionsApply라는 곳에서 [ExternalsPlugin을 통해 externals 옵션을 읽고](https://github.com/webpack/webpack/blob/1f13ff9fe587e094df59d660b4611b1bd19aed4c/lib/WebpackOptionsApply.js#L77-L83) 있었다.

```javascript
const ExternalsPlugin = require("./ExternalsPlugin");
new ExternalsPlugin(options.externalsType, options.externals).apply(compiler);
```

그리고 externals를 넘기는 부분을 따라 올라가다 보면 [ExternalModuleFactoryPlugin에서 externals 옵션이 function으로 넘어온 경우를 처리](https://github.com/webpack/webpack/blob/1f13ff9fe587e094df59d660b4611b1bd19aed4c/lib/ExternalModuleFactoryPlugin.js#L176-L224)하고 있었다.

```javascript
const promise = externals(
  {
    context,
    request: dependency.request,
    dependencyType,
    contextInfo,
    getResolve: (options) => (context, request, callback) => {
      const resolveContext = {
        fileDependencies: data.fileDependencies,
        missingDependencies: data.missingDependencies,
        contextDependencies: data.contextDependencies,
      };
      let resolver = normalModuleFactory.getResolver(
        "normal",
        dependencyType
          ? cachedSetProperty(data.resolveOptions || EMPTY_RESOLVE_OPTIONS, "dependencyType", dependencyType)
          : data.resolveOptions
      );
      if (options) resolver = resolver.withOptions(options);
      if (callback) {
        resolver.resolve({}, context, request, resolveContext, callback);
      } else {
        return new Promise((resolve, reject) => {
          resolver.resolve({}, context, request, resolveContext, (err, result) => {
            if (err) reject(err);
            else resolve(result);
          });
        });
      }
    },
  },
  cb
);
```

여기서 위에서 설명했던 `getResolve`를 넘겨주고 있는데, `getResolve` 내부에서 [NormalModuleFactory의 getResolver](https://github.com/webpack/webpack/blob/1f13ff9fe587e094df59d660b4611b1bd19aed4c/lib/NormalModuleFactory.js#L1161-L1163)를 통해 `resolver`를 받아 `callback`을 넘겨주고 있었다.

```javascript
class NormalModuleFactory extends ModuleFactory {
  // ...

  getResolver(type, resolveOptions) {
    return this.resolverFactory.get(type, resolveOptions);
  }

  // ...
}
```

`getResolver` 내부에서는 [ResolverFactory](https://github.com/webpack/webpack/blob/1f13ff9fe587e094df59d660b4611b1bd19aed4c/lib/ResolverFactory.js)를 사용하고 있었고, 거기서 `enhanced-resolve`라는 라이브러리를 사용해 `resolver`를 생성한다는 것을 알게 되었다.

```javascript
const Factory = require("enhanced-resolve").ResolverFactory;
```

```javascript
class ResolverFactory {
  // ...

  _create(type, resolveOptionsWithDepType) {
    // ...

    const resolver = /** @type {ResolverWithOptions} */ (Factory.createResolver(resolveOptions));

    // ...
  }

  // ...
}
```

[enhanced-resolve는 웹팩에서 만든 라이브러리](https://github.com/webpack/enhanced-resolve)였고, `Resolver`` 코드를 살펴보니 [finishResolved라는 function에서 resolution 결과를 callback에 넘겨주고](https://github.com/webpack/enhanced-resolve/blob/3a28f47788de794d9da4d1702a3a583d8422cd48/lib/Resolver.js#L349-L359) 있었다.

```javascript
const finishResolved = (result) => {
  return callback(
    null,
    result.path === false
      ? false
      : `${result.path.replace(/#/g, "\0#")}${result.query ? result.query.replace(/#/g, "\0#") : ""}${
          result.fragment || ""
        }`,
    result
  );
};
```

그리고 `Resolver`를 생성하는 `ResolverFactory`에서 [DescriptionFilePlugin이라는 plugin을 Resolver에 넘겨주고](https://github.com/webpack/enhanced-resolve/blob/3a28f47788de794d9da4d1702a3a583d8422cd48/lib/ResolverFactory.js#L355-L362) 있다는 것을 알 수 있었다.

```javascript
plugins.push(new DescriptionFilePlugin("parsed-resolve", descriptionFiles, false, "described-resolve"));
```

`DescriptionFilePlugin`은 파라미터로 넘긴 `descriptionFiles`를 읽어서 `resolver`의 resolution 결과에 추가해주는 plugin이었고, [descriptionFiles에는 package.json이 명시](https://github.com/webpack/enhanced-resolve/blob/3a28f47788de794d9da4d1702a3a583d8422cd48/lib/ResolverFactory.js#L199-L201)되어 있었다.

즉, `descriptionFileData`는 `package.json`을 읽어들인 결과였고, `descriptionFileData.type`은 `package.json`의 type field를 말하는 것이었다.

`descriptionFileData`가 `package.json`이라는 것을 알게 되니 위에서 `isESM`을 구하는 조건이 무슨 의미였는지 바로 이해가 갔다.

```typescript
const isEsm = /\.js$/i.test(result) ? resolveData?.descriptionFileData?.type === "module" : /\.mjs$/i.test(result);
```

Node.js 표준에서 모듈을 ESM으로 읽는 경우는 두 가지인데:

- `package.json` type field의 값이 `module`일 때 `.js`는 ESM이다.
- `.mjs`는 무조건 ESM이다.

Next.js에서는 `package.json`을 읽어들여 위 표준의 조건을 반영한 것이었다.

`descriptionFileData`의 정체를 알았으니 다시 돌아와서, `handleExternals`의 내부에서는 `resolveExternal`을 실행해 얻은 것을 통해 값을 리턴한다.

```javascript
async function handleExternals(
  context: string,
  request: string,
  dependencyType: string,
  layer: WebpackLayerName | null,
  getResolve: (options: any) => (resolveContext: string, resolveRequest: string) => Promise<[string | null, boolean]>
) {
  // ...

  const isEsmRequested = dependencyType === "esm";

  // ...

  const resolveResult = await resolveExternal(
    dir,
    config.experimental.esmExternals,
    context,
    request,
    isEsmRequested,
    hasAppDir,
    getResolve,
    isLocal ? resolveNextExternal : undefined
  );

  // ...

  const { res, isEsm } = resolveResult;

  // ...

  const externalType = isEsm ? "module" : "commonjs";

  // ...

  if (/node_modules[/\\].*\.[mc]?js$/.test(res)) {
    if (isWebpackServerLayer(layer)) {
      // All packages should be bundled for the server layer if they're not opted out.
      // This option takes priority over the transpilePackages option.

      if (optOutBundlingPackageRegex.test(res)) {
        return `${externalType} ${request}`;
      }

      return;
    }

    if (shouldBeBundled) return;

    // Anything else that is standard JavaScript within `node_modules`
    // can be externalized.
    return `${externalType} ${request}`;
  }

  // ...
}
```

즉, `handleExternals`의 역할은 ESM 포맷을 표준에 맞춰 지원한 라이브러리인 경우 `module {라이브러리 이름}`으로 리턴하고, 아닌 경우는 `commonjs {라이브러리 이름}`으로 리턴하는 것이다.

그럼 다시 처음으로 돌아가서 정리해보면,

- Next.js 서버는 Node.js 표준을 올바르게 지켜 ESM을 지원한 라이브러리는 ESM으로 읽고, 아닌 경우는 CJS로 읽는다.
- 라이브러리 A, B 모두 CJS/ESM을 지원했지만, B만 CJS로 읽혔다.

그렇다면 라이브러리 B가 ESM을 표준에 맞지 않게 지원하고 있다는 뜻이 된다.

그래서 실제로 라이브러리 B의 `package.json`을 확인해보니, type field는 `commonjs`인데, ESM 번들의 확장자는 `.js`로 해둔 것을 확인할 수 있었다.

따라서 이는 Node.js표준에 맞지 않으므로, 위에서 설명한 Next.js 코드에서 `isESM`이 `false`가 되고, CJS 번들이 읽혔던 것이었다.

결과적으로 문제를 해결하려면 Node.js 표준에 맞춰 라이브러리 B의 ESM 번들 확장자를 `.mjs`로 바꾸면 된다.
