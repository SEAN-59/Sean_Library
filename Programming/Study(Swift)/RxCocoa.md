# RxCocoa 정리
### RxCocoa 는 또 뭔데?
- 애플의 운영체제의 앱을 만들기 위해서는 필수적으로 사용이 되는 framework인 cocoaFramwork를 사용해야 하는데 이를 Rx 에서 사용하기 용이하게 잘 포장해놓은것
- RxSwift 를 코드내에서 자연스럽게 사용하려면 필수!

### 등장 개념
1. Binder
2. Driver 나 Signal : Rx에서 나온 maybe, single 같은 거 -> Traits 라 함
3. Rx extension

### Binder
- 방송국과 시청자 같은 느낌 : 단방향 데이터 스트림 = 앱의 데이터 흐름을 단순화하는 하나의 방법
    - 값을 생성하는 생성자 --> 값을 수신하는 수신자 ( o )
        값을 생성하는 생성자 <-- 값을 수신하는 수신자 ( x )
- 함수로는 .bind(to:) 가 존재함
- Observable을 바인딩하기 위해서는 수신자도 Observable 타입이어야 함
- ##### Binder는 데이터를 소비하는 역할만 함
    - Rx에 Subject 가 Observable이면서 Observer였다면 Binder는 Observer의 역할 만 함
- ##### Binder는 error이벤트를 받지 않음 -> 정확히 말하면 무시함
    - 만약 error 이벤트 발생하게 되면 Observable 시퀀스 자체가 종료되게 되는데 Binder는  UIComponent, UIBinding에 사용을 하게 되는데 error를 받게 되어 종료되게 되면 이벤트 전달이 멈춰서 더이상 업데이트 되지 않아 동작하지 않는것처럼 보이게 됨
- UIBinding에 사용된다는 특징이 반영되어 Binding method가 Main Thread에서 실행됨을 보장함
    - Binding 성공시 UI 업데이트가 되는데 이 업데이트는 mainThread 에서만 이루어져야 하기에 이를 보장하는것