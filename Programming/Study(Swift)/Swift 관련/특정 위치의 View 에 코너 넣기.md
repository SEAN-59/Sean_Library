# 특정 위의 View 에 코너 넣기
```swift
    view.layer.maskedCorners = CACornerMask(arrayLiteral: <#구성 요소#>)
```

## 요소 설명
1. .layerMinXMaxYCorner
    - 좌/하단
2. .layerMaxXMaxYCorner
    - 우/하단
3. .layerMinXMinYCorner
    - 좌/상단
4. .layerMaxXMinYCorner
    - 우/상단

## 좌표 설명
- x, y 는 view의 좌측이 최소 X값이고 상단이 최소 Y 값이다.