# Cache Policy

CloudFront는 Cache Policy를 사용해 캐싱을 어떻게 할지 결정한다.

Cache Policy는 응답이 캐싱될 시간을 정하는 TTL과, 사용자의 요청에서 캐시 키에 포함될 정보가 무엇인지를 가지고 있다. 기본으로 제공되는 Managed Cache Policy를 써도 되고, 우리가 직접 생성할 수도 있다.

이때 캐시 키에 포함될 정보란, HTTP Headers, Query Parameters, Cookie를 말한다.

CloudFront는 요청에 대해 캐시 키를 생성하고, 동일한 캐시 키에 대한 요청은 오리진으로 보내지 않고 바로 캐싱된 결과를 응답하는데, 이때 기본적으로 캐시 키에 포함되는 정보는 요청 도메인과 URL 밖에 없다.

따라서 캐시 적중률을 높히기 위해 필요에 따라 캐시 키에 더 많은 정보를 포함시킬 수 있는 것이다.

나는 CSR 프론트엔드에서 SEO와 Open Graph를 처리할 때, 크롤링 봇과 실제 유저의 요청을 구분하여 캐싱하기 위해 HTTP Headers 기반의 캐싱을 활용했다.. ([AWS Lambda@Edge로 CSR 환경에서 SEO 처리하기](https://blog.hoseung.me/2021-11-28-lambda-edge-seo/))

또한 이미지 리사이징 서버를 구축할 때 요청을 `?url={URL}&w={가로}&h={세로}&q={퀄리티}` 형태로 받도록 구현했었는데, 동일한 요청에 대해 다시 리사이징이 진행되지 않도록 Query Parameters 기반의 캐싱을 활용했다. ([디바이스 해상도에 딱 맞는 이미지 제공하기](https://blog.hoseung.me/2023-04-02-provide-fit-image/))

## Minimum TTL, Maximum TTL, Default TTL

Cache Policy를 만들 때는 Minimum/Maximum/Default 세 가지의 TTL을 설정하는데, 처음 보면 각각이 무슨 역할인지 굉장히 헷갈린다.

쉽게 말하면, Origin에서 응답한 `Cache-Control`, `Expires` 등의 캐시 헤더 값을 제어하는 것이다.

예를 들어, Origin에서 `Cache-Control: max-age=31536000`을 CloudFront로 응답하면, CloudFront는 31536000초 동안 Origin에서 응답한 리소스를 캐싱해야 한다. 이때 Maximum TTL이 3600초로 설정되어 있다면, Origin의 `Cache-Control`을 무시하고 3600초 동안 캐시한다.

반대로 Origin에서 `Cache-Control: max-age=0`을 CloudFront로 응답하면, CloudFront는 Origin에서 응답한 리소스를 캐싱하면 안된다. 이때 Minimum TTL이 86400초로 설정되어 있다면, Origin의 `Cache-Control`이 무시되고 86400초 동안 캐시한다.

Origin에서 아예 캐시 관련 헤더를 설정하지 않는다면, CloudFront는 Origin이 응답한 리소스를 Default TTL 만큼 캐시한다.
