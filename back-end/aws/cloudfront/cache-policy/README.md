# Cache Policy

CloudFront는 Cache Policy를 사용해 캐싱을 어떻게 할지 결정한다.

Cache Policy는 응답이 캐싱될 시간을 정하는 TTL과, 사용자의 요청에서 캐시 키에 포함될 정보가 무엇인지를 가지고 있다. 기본으로 제공되는 Managed Cache Policy를 써도 되고, 우리가 직접 생성할 수도 있다.

이때 캐시 키에 포함될 정보란, HTTP Headers, Query Parameters, Cookie를 말한다.

CloudFront는 요청에 대해 캐시 키를 생성하고, 동일한 캐시 키에 대한 요청은 오리진으로 보내지 않고 바로 캐싱된 결과를 응답하는데, 이때 기본적으로 캐시 키에 포함되는 정보는 요청 도메인과 URL 밖에 없다.

따라서 캐시 적중률을 높히기 위해 필요에 따라 캐시 키에 더 많은 정보를 포함시킬 수 있는 것이다.

나는 CSR 프론트엔드에서 SEO와 Open Graph를 처리할 때, 크롤링 봇과 실제 유저의 요청을 구분하여 캐싱하기 위해 HTTP Headers 기반의 캐싱을 활용했다.. ([AWS Lambda@Edge로 CSR 환경에서 SEO 처리하기](https://blog.hoseung.me/2021-11-28-lambda-edge-seo/))

또한 이미지 리사이징 서버를 구축할 때 요청을 `?url={URL}&w={가로}&h={세로}&q={퀄리티}` 형태로 받도록 구현했었는데, 동일한 요청에 대해 다시 리사이징이 진행되지 않도록 Query Parameters 기반의 캐싱을 활용했다. ([디바이스 해상도에 딱 맞는 이미지 제공하기](https://blog.hoseung.me/2023-04-02-provide-fit-image/))
