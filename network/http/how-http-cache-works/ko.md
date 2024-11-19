# HTTP 캐시의 동작

웹 개발시 HTTP 캐시에 대해 잘 알고 있어야, 브라우저 캐시로 인해 배포가 안되거나, 불필요한 네트워크 비용이 계속 발생하는 등의 문제를 막을 수 있다.

## HTTP 캐시의 종류

캐시는 크게 두 가지 종류로 나뉜다.

- Private Cache
  - HTTP 응답의 엔드 유저가 사용하는 캐시이다.
  - 예를 들면 브라우저 캐시가 있다.
- Shared Cache
  - HTTP 응답의 중간 서버가 사용하는 캐시이다.
  - 예를 들면 CDN이 있다. 이들은 특정 클라이언트에 의해 생성된 캐시를 다른 클라이언트에게도 사용하기 때문에 Shared Cache라고 불리는 것이다.

## HTTP 캐시 헤더

HTTP 캐시는 크게 아래 헤더들의 조합을 통해 이루어진다.

- `Cache-Control`
  - 말 그대로 캐시를 컨트롤하는 헤더이다. HTTP 요청, 응답에 모두 사용할 수 있다.
  - 캐시 사용 여부, 캐시의 위치, 캐시 만료 시간, 검증 여부 등을 설정할 수 있다.
  - 예시: `Cache-Control: private, max-age=60`
- `Expires`
  - 응답이 만료되었다고 판단할 날짜와 시간을 나타내는 헤더이다. HTTP 응답에 사용한다.
  - 명시된 날짜와 시간이 지난 경우, 캐시를 만료시키고 서버에 다시 요청을 보낸다.
  - `Cache-Control` 헤더가 이미 명시된 경우 무시된다.
  - 예시: `Expires: Thu, 27 Mar 2024 23:00:00 GMT`
- `ETag`
  - 응답에 대한 식별자를 나타내는 헤더이다. HTTP 응답에 사용한다.
  - 나중에 다시 요청을 보낼 때 `ETag` 값을 함께 보내서, 서버의 응답에 따라 캐시를 만료시킬지 결정한다.
  - 예시: `ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"`
- `If-None-Match`
  - 캐시된 응답의 ETag 값과 현재 서버 응답의 `ETag` 값이 같은지 확인하라는 의미의 헤더이다. HTTP 요청에 사용한다.
  - 클라이언트는 `If-None-Match`에 캐시된 응답의 `ETag` 값을 넣어서 서버로 요청을 보낸다.
  - 서버는 현재 응답의 ETag와 `If-None-Match`의 `ETag`를 비교해서, 똑같은 경우 `304 Not Modified`를 응답하고, 달라진 경우 `200 OK`를 응답한다.
  - `200 OK`의 경우 클라이언트는 기존 캐시를 만료시키고, 현재 응답에 대해 새롭게 캐시한다.
  - `304 Not Modified`의 경우 캐시를 계속 쓰라는 의미여서 응답 본문이 없다. 따라서 네트워크 비용도 저렴하다.
  - 예시: `If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"`
- `If-Modified-Since`
  - 서버의 응답이 특정 날짜와 시간 이후에 수정되었는지 확인하라는 의미의 헤더이다. HTTP 요청에 사용한다.
  - 서버는 클라이언트가 보낸 날짜와 시간 이후에 수정된 적이 있다면 `200 OK`를 응답하고, 수정된 적이 없다면 `304 Not Modified`를 응답한다.
  - `200 OK`와 `304 Not Modified`에 대한 클라이언트의 동작은 `If-None-Match`와 똑같다.
  - 예시: `If-Modified-Since: Thu, 27 Mar 2024 23:00:00 GMT`

## Cache-Control 헤더 자세히보기

다른 헤더는 넣어줄 값이 명확하지만, `Cache-Control` 헤더는 값이 매우 다양하기 때문에 살펴볼 필요가 있다. 위의 예시에서 봤다시피, `Cache-Control`은 아래 값들의 조합으로 이루어진다.

- `private`
  - 응답을 Private Cache에만 캐시하라는 의미이다.
- `public`
  - 응답을 Private/Shared Cache 어디에 캐시하든 상관 없다는 의미이다.
- `max-age=N`
  - 응답에 대한 캐시가 `N`초 동안 유효하다는 의미이다.
  - 예를 들어 `max-age=3600`으로 설정하면 3600초 동안은 서버에 새로운 요청을 보내지 않고 캐시만 사용한다.
- `s-maxage=N`
  - 응답을 Shared Cache에서 캐시할 때 캐시가 `N`초 동안 유효하다는 의미이다.
  - 예를 들어 `max-age=3600, s-maxage=31536000`으로 설정하면 클라이언트에는 3600초 동안 캐시되지만, CDN 같은 중간 서버에는 31536000초(1년) 동안 캐시된다.
- `must-revalidate`
  - 캐시가 만료되고 나면 서버로 다시 요청을 보내라는 의미이다. 재검증을 하라는 의미가 아니다.
- `no-cache`
  - 응답을 캐시하되, 캐시를 사용하기 전 매번 재검증 요청을 보내라는 의미이다. 캐시를 하지 말라는 의미가 아니다.
  - 말이 재검증 요청이지 특별한 절차가 있는 것은 아니고, 그냥 서버에 똑같이 요청 보내는 것이다. 대신 위에서 설명한 `If-None-Match`, `If-Modified-Since`와 같은 헤더가 함께 보내지고, 응답이 `200 OK`인지 `304 Not Modified`인지에 따라서 캐시를 갱신하는 것이다.
- `no-store`
  - 절대로 아무런 캐시도 사용하지 말라는 의미이다.
  - 캐시를 사용하지 않으므로 매번 새로 요청을 보내고 새로 응답을 받게 된다.