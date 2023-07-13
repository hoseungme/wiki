# iOS에서 클립보드 복사가 안되는 이슈

iOS 브라우저에서 클립보드 복사가 안되는 이슈가 있었는데, 자세한 상황은 아래와 같았다.

1. 클릭 이벤트 발생
2. 클릭 이벤트 핸들러에서 유저가 복사해야할 링크를 내려주는 API 요청
3. 해당 링크를 클립보드에 복사

이를 대략 코드로 표현하면 아래와 같다.

```tsx
<button
  onClick={async () => {
    const link = await fetchLink();
    await navigator.clipboard.writeText(link);
  }}
/>
```

브라우저들은 항상 클립보드 액션은 유저의 제스쳐를 통해 이루어지는 것을 강조하는데, 그 중간에 Ajax 요청이 있는 경우, 유저 제스쳐가 아닌 Ajax 응답에 의해서 클립보드 액션이 실행된 것으로 판단하고 보안상 차단시키는 것 같다.

다행히 클릭 시점에 불러올 필요는 없는 링크여서, `react-query`를 사용해 미리 불러온 뒤 클릭 이벤트가 일어나면 클립보드에 복사하는 방식으로 바꾸어 해결했다.

```tsx
const link = useQuery(key, fetchLink, { suspense: true }).data;

<button
  onClick={async () => {
    await navigator.clipboard.writeText(link);
  }}
/>;
```
