# hover Media Query

단순히 아래와 같이 CSS를 작성하는 경우, 예를 들어 터치 디바이스에서는 버튼을 클릭한 후에 hover style이 풀리지 않는 등 어색함이 있다.

```scss
button:hover {
  background-color: red;
}
```

이럴 때 아래와 같은 Media Query를 사용하면, 실제로 사용자가 현재 요소 위에 포인터를 올릴 수 있는지를 판단하여 스타일을 적용시킨다. 즉, 예를 들어 마우스 등의 포인팅 장치가 연결되지 않은 터치 디바이스의 경우, `button:hover` 스타일이 적용되지 않는다.

```scss
@media (hover: hover) {
  button:hover {
    background-color: red;
  }
}
```
