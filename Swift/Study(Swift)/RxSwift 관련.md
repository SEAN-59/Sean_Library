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
<summary>Observable의 명령어 </summary>
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
    <details>
    <summary>코드</summary>
    <div markdown="1">
    ```swift
    Observable<Int>.of(1,2,3)
        .subscribe(onNext: {
            print($0)
        })
    // 1
    // 2
    // 3
    ```
    ```swift
    Observable<Int>.of([1,2,3])
        .subscribe(onNext: {
            print($0)
        })  
    // [1,2,3]
    ```
    <div>
    </details>

3. From : array 형태의 요소만 받는 명령어
    <details>
    <summary>코드</summary>
    <div markdown="1">
    ```swift
    Observable.from([1,2,3]) 
    .subscribe(onNext:{
        print($0)
    })
    // 1
    // 2
    // 3
    ```
    <div>
    </details>
4. Subscribe : 어떤 명령어를 사용을 하던 구독을 하지 않으면 그 값을 보여주지 않음
    - onNext 와 같은 내부 파라미터를 선언하지 않으면 과정을 보여주게 됨
    <details>
    <summary>코드</summary>
    <div markdown="1">
    ```swift
    Observable.of(1,2,3).subscribe{ print($0) }
    // next(1)
    // next(2)
    // next(3)
    // completed
    ```
    ```swift
    Observable.of(1,2,3).subscribe {
        if let element = $0.element {
            print(element)
        }
    }
    // 1
    // 2
    // 3
    ```
    <div>
    </details>

5. Empty : 아무런 요소를 가지지 않음
    - 아무런 요소를 가지지 않기에 Observable 에서 타입 추론 할 수 없음
    - Type을 명시적으로 써주면 추론이 가능해진다. void 와 매우 잘 맞음
    - 즉시 종료하고자 하는 Observable 을 갖고자 하거나, 의도적으로 0개의 값을 갖는 Observable 리턴시 사용
    <details>
    <summary>코드</summary>
    <div markdown="1">
    ```swift
    Observable.empty().subscribe { print($0) }
    //
    ```
    ```swift
    Observable<Void>.empty().subscribe { print($0) }
    // completed
    ```
    ```swift
    Observable<Void>.empty()
    .subscribe(onNext: {},
               onCompleted: { print("Completed") } )
    // completed
    ```
    ```swift
    Observable<Int>.empty()
    .subscribe(onNext: {_ in
    	print("Next"
    },
               onCompleted: { print("Completed") } )
    // completed
    ```
    <div>
    </details>

6. Never : 작동은 하지만 아무것도 내보내지 않음 -> debug 를 사용해 동작 되는지 확인 가능
    <details>
    <summary>코드</summary>
    <div markdown="1">
    ```swift
    Observable.never()
    .subscribe(onNext: {
        print($0)
    },
               onCompleted: {
        print("Completed")
    })
    //
    ```
    ```swift
    Observable.never()
    .debug()
    .subscribe(onNext: {
        print($0)
    },
               onCompleted: {
        print("Completed")
    })
    // 2022-10-26 00:44:36.770: Observable.playground:61 (__lldb_expr_123) -> subscribed
    ```
    <div>
    </details>

7. Range : start 값을 count 만큼 증가하면서 요소에 추가함 -> 반복문 느낌이랄까
    <details>
    <summary>코드</summary>
    <div markdown="1">
    ```swift
    Observable.range(start: 1, count: 10)
    .subscribe(onNext: {
        print("2*\($0) = \(2*$0)")
    },
               onCompleted: {
        print("Completed")
    })
    // 2*1 = 2
    // 2*2 = 4
    // 2*3 = 6
    // 2*4 = 8
    // 2*5 = 10
    // 2*6 = 12
    // 2*7 = 14
    // 2*8 = 16
    // 2*9 = 18
    // 2*10 = 20
    // Completed
    ```
    <div>
    </details>

8. Dispose : Subscribe 를 끊고 싶을 때 사용함 -> 메모리 누수 방지를 위해 사용
    <details>
    <summary>코드</summary>
    <div markdown="1">
    ```swift
    Observable.of(1,2,3)
    .subscribe(onNext: {
        print($0)
    }).dispose()
    // 1
    // 2
    // 3
    ```
    <div>
    </details>

9. DisposeBag : 8번과 같지만 사용법이 조금 다름
    <details>
    <summary>코드</summary>
    <div markdown="1">
    ```swift
    let disposeBag = DisposeBag()
    Observable.of(1,2,3)
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)
    // 1
    // 2
    // 3
    ```
    <div>
    </details>

10. Create : escaping Closure 로 Any Observable 을 취하고 diposable 을 리턴하는 방식
    - Error 는 error 단에서 Observable을 종료시킴
    <details>
    <summary>코드</summary>
    <div markdown="1">
    ```swift
    Observable.create { observer -> Disposable in
    observer.onNext(1)
    observer.onCompleted()
    observer.onNext(2)
    return Disposables.create()
    } .subscribe{print($0)}
        .disposed(by: disposeBag)
    // 1
    ```
    
    ```swift
    enum MyError: Error {
        case anError
    }
    
    Observable.create { observer -> Disposable in
        observer.onNext(1)
        observer.onError(MyError.anError)
        observer.onCompleted()
        return Disposables.create()
    }.subscribe(
        onNext: {
            print($0)
        },
        onError: {
            print($0.localizedDescription)
        },
        onCompleted: {
            print("Completed")
        },
        onDisposed: {
            print("Disposed")
        }
    ).disposed(by: disposeBag)
    // 1
    // The operation couldn’t be completed. (__lldb_expr_123.MyError error 0.)
    // Disposed
    ```
    <div>
    </details>

11. Deffered : subscribe 를 기다리는 Observable을 만드는 대신, 각 subscribe에 Observable 항목을 제공하는 Observable Factory를 만드는 방식
    - 그냥 observable을 모아서 만들고 한 번에 subscribe 하는 느낌
    <details>
    <summary>코드</summary>
    <div markdown="1">
    ```swift
    Observable.deferred {
        Observable.of(1,2,3)
    }.subscribe{ print($0) }
        .disposed(by: disposeBag)
    // next(1)
    // next(2)
    // next(3)
    // completed
    ```
    ```swift
    var shake: Bool = false
    let factory: Observable<String> = Observable.deferred {
        shake = !shake
        if shake {
            return Observable.of("🤝")
        } else {
            return Observable.of("👏")
        }
    }
    for _ in 0...3 {
        factory.subscribe(onNext: { print($0) } )
            .disposed(by: disposeBag)
    }
    // 🤝
    // 👏
    // 🤝
    // 👏
    ```
    <div>
    </details>

</div>
</details>

