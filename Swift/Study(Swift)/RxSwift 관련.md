# RxSwift 정리
### 1. RxSwift 란?
- Rx 는 비동기적으로 수시로 상태가 변하는 환경에서 직관적이고 효율적인 코드 작성을 도와줌
- 애플에서 제공하는 Notification, Delegate, GCD, Closure 보다 훨씬 직관적이고 효율적 표현 가능

### 2. 구성요소
- Observable
- Operator
- Scheduler : 직접 만들어서 사용하는 경우가 많지는 않음
#### Observable
- 비동기적이며 일정시간 동안 계속해서 이벤트를 생성함
- Every Observable instance is just a sequence : subscribe 되기 전에는 아무 이벤트도 내보내지 않음
    ##### LifeCycle
    1. 어떠한 구성요소를 가지는 next 이벤트 계속 방출 가능
    2. complete 이벤트 방출해 완전 종료 가능
    3. error 이벤트 방출해 완전 종료 가능

<details>
##### <summary> Observable의 명령어</summary>
<div markdown="1">

1. Just : 하나의 요소만 포함하는 Observable 시퀀스를 생성하는 명령어
    ```swift
    Observable<Int>.just(1)
        .subscribe(onNext: {
            print($0)
        })
        // 1
    ```
2. Of : 하나 이상의 이벤트를 넣을 수 있는 명령어
    ```swift
    Observable<Int>.of(1,2,3)
        .subscribe(onNext: {
            print($0)
        })
    // 1
    // 2
    // 3
    
    Observable<Int>.of([1,2,3])
        .subscribe(onNext: {
            print($0)
        })
    // [1,2,3]
    ```
3. From : array 형태의 요소만 받는 명령어
4. Subscribe : 어떤 명령어를 사용을 하던 구독을 하지 않으면 그 값을 보여주지 않음
    - onNext 와 같은 내부 파라미터를 선언하지 않으면 과정을 보여주게 됨
5. Empty : 아무런 요소를 가지지 않음
    - 아무런 요소를 가지지 않기에 Observable 에서 타입 추론 할 수 없음
    - Type을 명시적으로 써주면 추론이 가능해진다. void 와 매우 잘 맞음
    - 즉시 종료하고자 하는 Observable 을 갖고자 하거나, 의도적으로 0개의 값을 갖는 Observable 리턴시 사용
6. Never : 작동은 하지만 아무것도 내보내지 않음 -> debug 를 사용해 동작 되는지 확인 가능
7. Range : start 값을 count 만큼 증가하면서 요소에 추가함 -> 반복문 느낌이랄까
8. Dispose : Subscribe 를 끊고 싶을 때 사용함 -> 메모리 누수 방지를 위해 사용
9. DisposeBag : 8번과 같지만 사용법이 조금 다름

</div>
</details>

