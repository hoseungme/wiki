# Canvas에 그려진 요소를 픽셀 데이터로 변환하기

Canvas의 2D 컨텍스트는 특정 부분을 픽셀 데이터로 변환할 수 있는 `getImageData` 메소드를 제공한다. ([MDN](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/getImageData))

리턴된 픽셀 데이터에는 색상의 RGB 값과 Alpha 값이 들어있는데, 나는 이를 [scratchable](https://github.com/HoseungJang/scratchable)이라는 오픈소스를 만들 때 [`destination-out` 합성 타입](../canvas-shapes-composition/ko.md)을 통해 캔버스에서 지워진 영역이 전체의 몇 %를 차지하는지 구하기 위해 사용했다.
