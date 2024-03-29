# CloudFront OAC

CloudFront의 OAC(Origin Access Control)를 통해서 Origin에 대한 외부의 직접적인 접근을 전부 차단하고, CloudFront를 거쳐서만 접근할 수 있도록 만들 수 있다.

<img src="./origin-edit.png" alt="" width="500px" />

보통 위처럼 S3을 Origin으로 둘텐데, 편집 페이지에서 바로 OAC를 생성 후, `정책 복사` 버튼을 눌러 S3 버킷 정책에 붙여넣으면 끝이다.
