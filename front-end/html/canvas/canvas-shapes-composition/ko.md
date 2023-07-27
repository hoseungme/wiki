# Canvas 요소 합성하기

Canvas의 2D 컨텍스트에는 `globalCompositeOperation` 이라는 프로퍼티가 있다. 이를 통해 기존에 그린 요소와 겹치는 위치에 새로운 요소를 그리는 경우, 그 두 요소가 어떻게 합쳐져서 그려질지를 결정할 수 있다.

`globalCompositeOperation`에 할당 가능한 값의 종류는 [MDN](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/globalCompositeOperation#operations)에 잘 나와있다. 나는 그중 `destination-out`을 활용하여 [scratchable](https://github.com/HoseungJang/scratchable) 이라는 오픈소스를 만들었다.
