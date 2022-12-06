# Rx로 UITextField Refactoring.md
- 독학 버전이라서 잘못된 정보도 많고 아직 다듬어야 할 부분도 엄청 많겠지만 수정해야 될 부분은 하나하나 사용해보면서 고치면 될듯하다.
- 일단 Rx 생각보다 더 어렵고 정보가 있어도 어떻게 내꺼로 만들어야 할 지 잘 몰라서 직접 하나하나 코드를 열어보고 사용해보면서 익숙해지는 수 밖에 없을듯 하다.
----
## 자료
[RxSwift 를 이용한 공통 UITextField 리팩토링](https://dealicious-inc.github.io/2021/12/06/rxswift-textfield.html)
---
## 조건
1. textField 에 글이 작성됨과 동시에 몇가지 작업들이 처리되었으면 한다.
    1. 입력 제한
    2. 작성 포맷 제한
    3. 최대 길이 출력
    4. 현재 입력되고 있는 값이 무엇인지 알려주는 출력

2. delegate, addTarget, rx 로 파편화 되어있는 방식 rx로 통합

----
## 내용
- rx 로 통합을 예정이기에 rx 를 이용한 작업이 주가 되어야 함
    - UITextField 에서 .rx.text 는 ControlProperty 이벤트 발생시 value 를 방출 해줌

        * ControlProperty 는 Subject 처럼 프로퍼티에 값을 주입할 수 있고 동시에 값의 변화도 관찰할 수 있는 타입이다.<br>사용시 해당 프로퍼티의 변경사항을 데이터 시퀀스로 받아올 수 있다.
        * Cocoa 에서는 수많은 Event 가 발생하는데 이러한 이벤트들에 대한 시퀀스를 받아올 수 있는 기능이 바로 ControlEvent 이다.

- 조건 1.4 의 경우 값이 바뀌는 경우와 언포커싱되는 경우에 값을 불러오기를 원하기에
    .edtingChanged, .valueChanged, .editingDidEnd 의 이벤트 발생시만 방출되게 한다.

---
## 결과
- 일단 어떻게 해야 rx 에 명령어? 키워드? 를 만들어서 나한테 맞춰서 쓸 수 있을지 알게 되었음.


## 구현
```swift
    extension Reactive where Base: UITextField {
        public var changedText: ControlProperty<String?>{
            return base.rx.controlProperty(editingEvents: [.editingChanged, .valueChanged, .editingDidEnd],
                                        getter: { textField in
                textField.text
            },
                                        setter: { textField, value in
                if textField.text != value {
                    textField.text = value
                }
            })
        }
    }
```

```swift
    let test: Driver<String?> = testTxf.rx.changedText
            .orEmpty
            .scan(testTxf.text) { [weak self] (prev, new) -> String? in
                guard let self = self else { return prev }
                if prev!.count >= 5 {
                    return prev
                }
                
                return new
            }
            .map { [weak self] str -> String? in
                guard let self = self else { return ""}
                
                return str
            }
            .asDriver(onErrorJustReturn: "")
        
    test.drive(testTxf.rx.text)
        .disposed(by: disposeBag)
```