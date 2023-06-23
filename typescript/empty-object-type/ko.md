# 빈 객체를 타입으로 표현하는 법: {}에 대해

타입스크립트에서 빈 객체를 표현하기 위해 아래와 같이 해본 적이 있었다.

```typescript
type EmptyObject = {};
const a: EmptyObject = {};
```

근데 저렇게 하면 아래와 같이 빈 객체가 아닌 값을 넣어도 타입 에러가 안난다. ([Playground](https://www.typescriptlang.org/play?ts=4.8.4#code/C4TwDgpgBAogtmUB5ARgKwgY2FAvFAbwF8BuAKDMwHsA7AZxwEMAuWBZdLHfY86+nClbxEIVBmx4oAIkZ0AJgDNpfWgyiZh7MZ0n4AjACYAzABZVAqPK2jxXKQSiKqVVtJSMATtKikyQA))

```typescript
const a: EmptyObject = "asdf";
const b: EmptyObject = 1234;
const c: EmptyObject = { foo: "bar" };
```

그 이유는 타입스크립트에서 `{}`라 함은 `any non-nullish value`를 의미하기 때문이다. (대체 왜 이런 것인지는 모름) 실제로 `undefined`, `null`을 넣어보면 에러가 발생한다. ([Playground](https://www.typescriptlang.org/play?ts=4.8.4#code/C4TwDgpgBAogtmUB5ARgKwgY2FAvFAbwF8BuAKDMwHsA7AZxwEMAuWBZdLHfAVxoBMIAMwCWNCP3LV6OFK3iIQqDNjxQaPADabyQA))

```typescript
type EmptyObject = {};

const a: EmptyObject = undefined; // Type 'undefined' is not assignable to type 'EmptyObject'.
const b: EmptyObject = null; // Type 'null' is not assignable to type 'EmptyObject'.
```

`{}`가 의미하는 것이 `any non-nullish value`가 아닐 때도 있는데, 그것은 바로 intersect되었을 때이다. 단, `{}`끼리 intersect하는 경우는 똑같이 `any non-nullish value`이다. ([Playground](https://www.typescriptlang.org/play?ts=4.8.4#code/MYewdgzgLgBAhgLhgbwL4wGQygJwK4CmMAvCqgNwwD0VMgAwuCh4zIAujgN+0BQoksARkmpmz4ipXIXKdw0GMH7osyGADMQIJNBwBLMAHMY6Umko16TNpO4wAJnMGKVamBu16DKZaqQAiHnBxf9CS5pAlsFNwBGACYAZgAWCSA))

```typescript
const a: {} & true = {}; // Type '{}' is not assignable to type 'true'.
const b: {} & true = true;
const c: {} & { foo: string } = {}; // Property 'foo' is missing in type '{}' but required in type '{ foo: string; }'
const d: {} & { foo: string } = { foo: "bar" };
const e: {} & {} = 1234;
```

결론은 빈 객체를 표현하고 싶을 땐 never 타입을 사용하면 된다.

```typescript
type EmptyObject = Record<string, never>;
// 또는 { [k: string]: never }
```
