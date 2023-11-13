# CloudFront Functions 리다이렉션 구성 후기

CloudFront에서 활용할 수 있는 엣지 컴퓨팅 서비스는 Lambda@Edge, CloudFront Functions 두 가지가 있다.

Lambda@Edge는 자주 써봤는데, CloudFront Functions는 기회가 없어서 안써봤었다. 근데 최근에 영어 이력서를 만들게 되면서 외국에서 접속한 사람들을 리다이렉션 시켜주고 싶은 니즈가 생겨서 써보게 되었다.

[AWS 한국 블로그 글](https://aws.amazon.com/ko/blogs/korea/introducing-cloudfront-functions-run-your-code-at-the-edge-with-low-latency-at-any-scale/)을 참고해봤을 때, Lambda@Edge와 다르게 CloudFront는 HTTP 요청/응답 조작 정도의 엄청나게 단순한 작업을 위해 만들어졌다고 한다.

그래서인지 viewer-request, viewer-response 트리거만 지원하고, Lambda@Edge에 비해 최대 실행 시간도 엄청 짧고, 메모리도 매우 작다. 네트워크나 파일시스템 액세스도 불가능하다.

대신 요금이 되게 저렴하다. Lambda@Edge는 실행 횟수, 그리고 실행 시간까지 요금에 포함하지만, CloudFront Functions는 실행 횟수만 요금에 포함된다.

게다가 Lambda@Edge가 실행 100만 번당 0.60 USD인 반면, CloudFront Functions는 같은 횟수에 대해 0.10 USD만 부과한다. 또한 CloudFront Functions는 프리 티어도 지원한다.

따라서 저렴한 가격에 HTTP 요청/응답 조작 수준의 간단한 작업이 필요한 경우 제격이라고 생각한다.

그래서 사용자의 위치가 한국이 아닌 경우 영어 이력서로 리다이렉션 시키는 아주 간단한 함수를 작성해봤다.

```typescript
function handler(event: AWSCloudFrontFunction.Event) {
  var request = event.request;
  var url = request.uri;
  var countryCode = request.headers["cloudfront-viewer-country"].value.toLowerCase();

  if (countryCode !== "kr" && !url.startsWith("/en")) {
    var response: AWSCloudFrontFunction.Response = {
      statusCode: 302,
      statusDescription: "Found",
      headers: {
        location: { value: `/en${url}` },
      },
    };

    return response;
  }

  return request;
}
```

뭔가 최대한 메모리를 덜 쓰기 위해서 자바스크립트 런타임도 극단적으로 최소화를 했다고 느껴졌다. 그 예시로 let/const 문법을 지원하지 않는다는 것을 들 수 있겠다. 물론 거대한 코드베이스도 아니고, 그냥 간단한 함수 하나인데 var 쓴다고 큰일 안난다. 개발진도 같은 생각으로 지원 대상에서 뺀게 아닐까?

배포하는 과정도 Lambda@Edge보다 훨씬 편했다. Lambda@Edge는 번거로운 제약사항이 많아서 처음 배포할 때 고생했던 기억이 어렴풋이 나는데, CloudFront Functions는 AWS 콘솔에서 코드 복붙 + 버튼 몇 번 누르는 것으로 배포가 끝났다.

여기서 좋았던게, 일련의 테스트/배포 파이프라인을 제공한다는 것이다. 개발 버전 / 라이브 버전 코드가 분리되어 있고, 개발 버전을 먼저 업로드 한 후, 자연스럽게 테스트를 거치고 라이브 버전으로 업로드할 수 있도록 구성되어 있다.

물론 Lambda에도 테스트 있고 좋지만, CloudFront Functions 자체가 Lambda에 비해 엄청나게 스펙이 작기 때문에 기본적으로 직관적이고 쉽게 느껴졌다. 특히 테스트가 참 좋았다.

앞으로 쓸 일이 꽤 많을 듯 하다.
