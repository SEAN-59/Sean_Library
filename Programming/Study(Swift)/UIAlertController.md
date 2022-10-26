# UIAlertController

- 두가지 형태로 메시지 창을 표현할 수 있음 : 알림창 and 액션시트
1. 알림창 : 그 화면 중앙에 박스 형태로 뜨는거를 말함
    - modal 방식으로 나타남 → 이 창이 닫히기 전까지 이 창을 제외한 화면의 다른 부분은 반응 못함
    - UIAlertController 를 통해 메시지 창 컨트롤러 인스턴스 생성
    - UIAlertAction 를 통해 메시지 창 컨트롤러에 들어갈 버튼 액션 객체 생성
    - 컨트롤러에 .addAction 을 사용해 생성한 액션객체를 추가함
    - .present(컨트롤러, animated) 를 사용해 컨트롤러를 표시
2. 액션시트 : 화면 밑에서 올라오는 형태로 뜨는거를 말함

## Alert

```swift
let alert = UIAlertController(title: "초기화", message: "저장되었습니다.", preferredStyle: UIAlertController.Style.alert)
let okAction = UIAlertAction(title: "확인", style: .default, hadler: ~~)
alert.addAction(okAction)
present(alert, animated: false)
```