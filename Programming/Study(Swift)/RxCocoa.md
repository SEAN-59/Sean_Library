# RxCocoa 정리
### RxCocoa 는 또 뭔데?
- 애플의 운영체제의 앱을 만들기 위해서는 필수적으로 사용이 되는 framework인 cocoaFramwork를 사용해야 하는데 이를 Rx 에서 사용하기 용이하게 잘 포장해놓은것
- RxSwift 를 코드내에서 자연스럽게 사용하려면 필수!

### 등장 개념
1. Binder
2. Driver 나 Signal : Rx에서 나온 maybe, single 같은 거 -> Traits 라 함
3. Rx extension

## Binder
- 방송국과 시청자 같은 느낌 : 단방향 데이터 스트림 = 앱의 데이터 흐름을 단순화하는 하나의 방법
    - 값을 생성하는 생성자 --> 값을 수신하는 수신자 ( o )
        값을 생성하는 생성자 <-- 값을 수신하는 수신자 ( x )
- 함수로는 .bind(to:) 가 존재함
- Observable을 바인딩하기 위해서는 수신자도 Observable 타입이어야 함
- Binder는 데이터를 소비하는 역할만 함
    - Rx에 Subject 가 Observable이면서 Observer였다면 Binder는 Observer의 역할 만 함
- Binder는 error이벤트를 받지 않음 -> 정확히 말하면 무시함
    - 만약 error 이벤트 발생하게 되면 Observable 시퀀스 자체가 종료되게 되는데 Binder는  UIComponent, UIBinding에 사용을 하게 되는데 error를 받게 되어 종료되게 되면 이벤트 전달이 멈춰서 더이상 업데이트 되지 않아 동작하지 않는것처럼 보이게 됨
- UIBinding에 사용된다는 특징이 반영되어 Binding method가 Main Thread에서 실행됨을 보장함
    - Binding 성공시 UI 업데이트가 되는데 이 업데이트는 mainThread 에서만 이루어져야 하기에 이를 보장하는것

## Traits
- 여러가지가 존재하지만 대표적으로 Driver 와 Signal의 형태가 존재함
- Signal 과 Driver
    - 이 둘은 모두 error를 방출하지 않는 특별한 형태의 Observable
    - UI변경이 background Thread 에서 일어나는것을 방지하기 위해서 MainThread에서 이루어짐
    - Binder와 비슷하지만 Stream 공유가 가능하다는 차이점이 존재
    - as Driver(Signal)로 Observable을 Driver와 Signal로 전환이 가능함
- Signal 과 Driver 의 차이점
    - Public Subject 와 Behavior Subject의 차이와 비슷함
    - Signal 은 구독한 이후 발생한 값 만을 전달함
    - Driver 는 새로운 구독자에게 구독한 순간 초기값 또는 최신값을 줌
- Traits 은 굳이 사용을 안해도 다 구현이 가능은 하지만 컴파일링 중 또는 UI와 관련된 예정된 법칙의 체크를 원할때 유용함

## Rx Extension
- RxCocoa 는 자주 사용하는 유요한 CocoaFramework에 다양한 객체 속성들에 대해 RxExtension을 제공하고 있다.
- Ui의 특정 속성을 Rx스럽게 사용을 하고 싶은데 원하는 속성이 없거나 기존의 Cocoa의 속성을 커스텀해서 Rx에서 쓰고자 할 때 사용
- extension Reactive where Base: T { }

## Error 관리
- 에러 관리는 Rx 말고도 어느곳이던 중요하게 하기에 항상 Error 관리 매커니즘이 필요로 한다.
- Rx에서 Error 관리는 framework 중 하나로 분리가 되어 있다.
    1. Catch : doTtyCatch 랑 비슷함
        - 기본값인 default value 로 error를 복구하는 방법
    2. Retry
        - 제한적 또는 무제한적으로 재시도하는 방법

## Rx 로 코드를 짤때
- 이건 개인적으로 이렇게 사용한다
```swift
import UIkit
import RxSwift
import RxCocoa
import SnapKit
...
class SomeClass: ... {
    let disposeBag = DisposeBag()
    public func bind(){...} // Rx코드를 작성할 곳
    public func attribute(){...} // View를 꾸밀 곳
    public func layout(){...}
    init(){
        ... 
        bind()
        ...
    }
    ...
}
```