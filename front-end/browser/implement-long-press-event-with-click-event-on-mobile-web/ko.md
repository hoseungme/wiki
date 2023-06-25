# 모바일 웹에서 클릭 이벤트를 유지하면서 롱 프레스 이벤트 구현하기

아래와 같은 요구사항이 있었다.

- 1초 동안 꾹 누르고 있으면 작업 A를 실행한다.
- 그냥 클릭한 경우 작업 B를 실행한다.

즉, 롱 프레스 이벤트를 기존의 클릭 이벤트를 유지하면서 구현하라는 뜻이다.

이 경우, touchstart, touchmove, touchend, click 네 가지 이벤트를 활용해야 했다.

각각의 이벤트가 하는 일은 아래와 같다.

- `touchstart`: 1초 후 작업 A를 실행하는 타이머를 설정한다. 타이머가 실행된 경우, 이후 발생하는 `click` 이벤트를 무시하라고 알린다.
- `touchmove`: `touchstart` 이후 엘리먼트를 가만히 누르고 있는지를 감지하여, 롱 프레스를 의도한 것인지, 스크롤 등 다른 액션을 의도한 것인지를 판단한다. 만약 롱 프레스를 의도한게 아니라면, `touchstart`에서 등록한 타이머를 해제한다.
- `touchend`: 롱 프레스 구현을 위한 변수 값들을 처음 상태로 초기화 한다.
- `click`: 작업 B를 실행한다. 단, 롱 프레스로 판단되어 타이머가 실행된 경우, 작업 B를 실행하지 않는다.

로직은 아래와 같다.

```typescript
const button = buttonRef.current;
if (buttonbutton == null) {
  return;
}

let longTouchListener: NodeJS.Timeout | null = null;
let startTouchClientY: number | null = null;
let disableClickEvent = false;

const clearLongTouchListener = () => {
  if (longTouchListener != null) {
    clearTimeout(longTouchListener);
    longTouchListener = null;
  }
};

const handleTouchStart = (e: TouchEvent) => {
  startTouchClientY = e.changedTouches[0].clientY;
  longTouchListener = setTimeout(() => {
    doJobA(); // 작업 A 실행
    disableClickEvent = true; // click 이벤트 무시
    clearLongTouchListener();
  }, 1000);
};

const handleTouchMove = (e: TouchEvent) => {
  if (startTouchClientY == null) {
    return;
  }

  const movedClientY = Math.abs(startTouchClientY - e.changedTouches[0].clientY);
  // 20 픽셀 넘게 움직였을 경우, 롱 프레스가 아닌 다른 의도로 간주
  if (movedClientY > 20) {
    clearLongTouchListener(); // 롱 프레스 이벤트 무시
  }
};

const handleTouchEnd = () => {
  // 모두 초기화
  clearLongTouchListener();
  startTouchClientY = null;
  disableClickEvent = false;
};

const handleClick = () => {
  if (disableClickEvent) {
    return; // click 이벤트 무시된 경우 종료
  }

  doJobB(); // 작업 B 실행
};

button.addEventListener("touchstart", handleTouchStart);
button.addEventListener("touchmove", handleTouchMove);
button.addEventListener("touchend", handleTouchEnd);
button.addEventListener("click", handleClick);
```
