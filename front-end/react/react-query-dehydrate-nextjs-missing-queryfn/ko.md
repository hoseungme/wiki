# React Query를 Next.js에서 dehydrate()와 사용할 때 생기는 Missing queryFn 이슈

React Query를 Next.js에서 SSR과 함께 사용하기 위해서는 [dehydrate()를 사용](https://tanstack.com/query/v4/docs/react/reference/hydration#dehydrate)해 server side에서 prefetch한 값들을 client side에서 hydration 이후 사용할 수 있게 해줘야 한다.

이때, client side의 코드를 잘못 작성하게 되면 "Missing queryFn" 에러를 만나게 되었는데, 매우 간헐적으로 발생되어 재현이 정말 어려웠다.

일단 내 코드는 대략 아래와 같이 작성되어 있었다.

```tsx
function Page() {
  useInterval(() => queryClient.refetchQueries(key), 3000);

  if (condition) {
    return <A />;
  } else {
    return <B />;
  }
}

function A() {
  const { data } = useQuery(key, fetcher);
  return <span>{data}</span>;
}

function B() {
  return <span>something</span>;
}
```

이때, `condition` 와 상관없이 `<Page />`를 로딩할 때 `<A />` 에서 사용하고 있는 query에 대한 prefetch는 항상 해주고 있었다.

그렇기에 `condition`이 불만족하여 `<B />`가 렌더링된 경우, `<Page />`에서 query를 3초 마다 refetch하는 로직에서 문제가 발생했다.

왜냐하면 refetch를 하려면 해당 query에 등록된 queryFn, 즉 fetcher가 있어야 하는데, client side에서 그것을 등록해주는 것은 `<A />` 밖에 없기 때문이다.

그래서 `<B />`가 렌더링된 케이스에서는 실제로 refetch가 일어나지는 않고, fetchFailureCount가 계속 증가하는 것을 볼 수 있었다.

따라서 나중에 해당 query를 다른 데서 사용하게 되면 `Missing queryFn` 에러가 간헐적으로 발생했던 것이다. 아래와 같이 고친 후로는 에러가 사라졌고, Sentry 알림도 더이상 날아오지 않았다.

```tsx
function Page() {
  if (condition) {
    return <A />;
  } else {
    return <B />;
  }
}

function A() {
  const { data } = useQuery(key, fetcher);
  useInterval(() => queryClient.refetchQueries(key), 3000);

  return <span>{data}</span>;
}

function B() {
  return <span>something</span>;
}
```

이게 어려웠던 이유는, 내가 CSR과 SSR 간의 동작 차이를 생각하지 못했기 때문이다.

일반적인 CSR 웹의 경우, `useQuery()`를 실행하지 않으면 어차피 queryCache에도 관련 key, data, fetcher 등이 저장되지 않기에 refetch를 아무리 실행해도 무시된다.

하지만 `dehydrate()`를 사용해 prefetch된 queryCache를 사용하는 경우, client side에서 `useQuery()`를 실행하지 않더라도 관련된 key, data는 들어있다. 따라서 refetch가 동작하게 되는데, 이때 fetcher는 등록이 안되어있기 때문에 문제가 발생했던 것이다.
