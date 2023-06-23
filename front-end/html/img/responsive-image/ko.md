# Responsive Image: srcset, sizes

[DPR을 다룬 문서](../../../browser/device-pixel-ratio/ko.md)에 정리했듯이, 디바이스 해상도에 맞춰 올바른 크기의 이미지가 선택될 수 있도록 해야하는데, HTML 레벨에서 `img`의 `srcset`, `sizes` 속성을 사용해 쉽게 할 수 있다.

`srcset`은 이미지와 그 이미지의 실제 가로 크기의 쌍을 나열하고, `sizes`는 `img`의 가로 크기를 나열하는데, Media Query로 조건을 걸어 상황에 따라 다른 크기를 갖게 할 수 있다.

```html
<img
  srcset="300w.png 300w, 600w.png 600w, 900w.png 900w"
  sizes="(max-width: 700px) 300px, (max-width: 1000px) 600px, 900px"
/>
```

DPR이 2이고, 물리적 가로 1000 픽셀을 갖는 디바이스에서 위 HTML을 렌더링한다고 가정하면:

- 브라우저에서는 가로 500 픽셀로 표현될 것이고,
- `(max-width: 700px)` Media Query 조건에 걸렸으므로, `img`는 가로 300 픽셀로 렌더링된다.
- 이때 DPR이 2이므로, CSS상 300 픽셀을 렌더링하기 위해선 물리적 600 픽셀이 필요하다.
- 따라서 `600w`로 명시되어있는 `600w.png`가 선택된다.

`srcset`, `sizes`는 위와 같은 원리로 서로 상호작용한다.

이때, `img`는 항상 고정 크기여서 DPR로만 분기하고 싶을 수도 있다. 그땐 아래와 같이 `srcset`을 명시해주면 된다.

```html
<img srcset="300w.png 1x, 600w.png 2x, 900w.png 3x" width="300px" />
```

가로 300 픽셀로 `img`를 렌더링하기 위해, DPR 별로 알맞은 크기의 이미지를 나열해둔 모습이다.

## 브라우저 호환성 챙기기

[caniuse를 확인해보면](https://caniuse.com/?search=srcset) `srcset`의 호환성은 정말 좋지만, 만약을 대비해 `src`는 fallback으로 명시해두자.

```html
<img srcset="300w.png 1x, 600w.png 2x, 900w.png 3x" src="900w.png" width="300px" />
```
