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
    ```swift
    Observable<Int>.of(1,2,3)
        .subscribe(onNext: {
            print($0)
        })
    // 1
    // 2
    // 3
    ```
    ----------
    ```swift
    Observable<Int>.of([1,2,3])
        .subscribe(onNext: {
            print($0)
        })  
    // [1,2,3]
    ```
    
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
    ```swift
    Observable.of(1,2,3).subscribe{ print($0) }
    // next(1)
    // next(2)
    // next(3)
    // completed
    ```
    ----------
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

5. Empty : 아무런 요소를 가지지 않음
    - 아무런 요소를 가지지 않기에 Observable 에서 타입 추론 할 수 없음
    - Type을 명시적으로 써주면 추론이 가능해진다. void 와 매우 잘 맞음
    - 즉시 종료하고자 하는 Observable 을 갖고자 하거나, 의도적으로 0개의 값을 갖는 Observable 리턴시 사용
    ```swift
    Observable.empty().subscribe { print($0) }
    //
    ```
    ----------
    ```swift
    Observable<Void>.empty().subscribe { print($0) }
    // completed
    ```
    ----------
    ```swift
    Observable<Void>.empty()
    .subscribe(onNext: {},
               onCompleted: { print("Completed") } )
    // completed
    ```
    ----------
    ```swift
    Observable<Int>.empty()
    .subscribe(onNext: {_ in
    	print("Next"
    },
               onCompleted: { print("Completed") } )
    // completed
    ```

6. Never : 작동은 하지만 아무것도 내보내지 않음 -> debug 를 사용해 동작 되는지 확인 가능
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
    ----------
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

7. Range : start 값을 count 만큼 증가하면서 요소에 추가함 -> 반복문 느낌이랄까
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

8. Dispose : Subscribe 를 끊고 싶을 때 사용함 -> 메모리 누수 방지를 위해 사용
    ```swift
    Observable.of(1,2,3)
    .subscribe(onNext: {
        print($0)
    }).dispose()
    // 1
    // 2
    // 3
    ```

9. DisposeBag : 8번과 같지만 사용법이 조금 다름
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

10. Create : escaping Closure 로 Any Observable 을 취하고 diposable 을 리턴하는 방식
    - Error 는 error 단에서 Observable을 종료시킴
    ```swift
    Observable.create { observer -> Disposable in
        observer.onNext(1)
        observer.onCompleted()
        observer.onNext(2)
        return Disposables.create()
    } .subscribe{ print($0) }
        .disposed(by: disposeBag)
    // 1
    ```
    ----------
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

11. Deffered : subscribe 를 기다리는 Observable을 만드는 대신, 각 subscribe에 Observable 항목을 제공하는 Observable Factory를 만드는 방식
    - 그냥 observable을 모아서 만들고 한 번에 subscribe 하는 느낌
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
    ----------
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
</div>
</details>

### Subject
- 보통의 앱개발에서는 실시간으로 Observable 의 새로운 값을 수동으로 추가하고 Subcribe 에게 방출한다.
- 즉, Observable 이면서 Observer 가 필요한건데 이게 바로 Subject 임
    ##### 종류
    1. Public Subject
    2. Behavior Subject
    3. Replay Subject
    <details>
    <summary>Subject 설명</summary>
    <div markdown="1">
        1. Public Subject : 비어있는 상태로 시작해서 새 값이 발생하면 새 값을 subscribe 에 방출함
            ```swift
            let publishSubject = PublishSubject<String>()
            publishSubject.onNext("Hello")
            
            let subscriber1 = publishSubject
                .subscribe(onNext: { print($0) })
            publishSubject.onNext("Hi?")
            publishSubject.onNext("Can U hear me?")

            subscriber1.dispose()

            let subscriber2 = publishSubject
                .subscribe(onNext:{ print($0) })

            publishSubject.onNext("Hello~?")
            publishSubject.onCompleted()

            publishSubject.onNext("It's done")

            subscriber2.dispose()

            publishSubject
                .subscribe{
                    print("3rd Subscribe:", $0.element ?? $0)
                }.disposed(by: disposeBag)
            
            publishSubject.onNext("R U Sure?")

            // Hi?
            // Can U hear me?
            // Hello~?
            // 3rd Subscribe: completed
            ```
            - 결과에서 보면 알 수 있듯이 처음 .onNext 는 subscribe 가 되지 않아 출력이 되지 않으며 마지막 .onNext 는 dispose 되어 출력 되지 않음을 알 수 있음
            
        2. Behavior Subject : 하나의 초기값을 가진 상태로 시작해 새 Subscribe가 생겼을 때, 초기값 또는 최근값을 방출함
        - Observable 은 subscribe 내에서 값을 가지고 활용을 하지만 Behavior Subject 는 value 를 뽑아내는게 가능함
            ```swift
            enum SubjectError: Error {
                case error1
            }

            let behaviorSubject = BehaviorSubject<String>(value: "initValue")
            behaviorSubject.onNext("First Value")

            behaviorSubject.subscribe {
                print("First: ", $0.element ?? $0)
            }.disposed(by: disposeBag)

            behaviorSubject.onError(SubjectError.error1)

            behaviorSubject.subscribe {
                print("Second: ", $0.element ?? $0)
            }.disposed(by: disposeBag)

            // First:  First Value
            // First:  error(error1)
            // Second:  error(error1)
            ```
        
        3. Replay Subject : 버퍼를 두고 초기화를 하게 되고 버퍼사이즈 만큼의 이벤트를 유지하며 새 subscribe 발생시 이벤트를 방출함
            ```swift
            let replaySubject = ReplaySubject<String>.create(bufferSize: 2)

            replaySubject.onNext("First")
            replaySubject.onNext("Second")
            replaySubject.onNext("Third")

            replaySubject.subscribe {
                print("First SubScriber:", $0.element ?? $0)
            }.disposed(by: disposeBag)

            replaySubject.subscribe {
                print("Second Subscribe: ", $0.element ?? $0)
            }.disposed(by: disposeBag)

            replaySubject.onNext("Fourth")
            replaySubject.onError(SubjectError.error1)
            replaySubject.dispose()

            replaySubject.subscribe {
                print("Third Subscribe: ", $0.element ?? $0)
            }.disposed(by: disposeBag)

            // First SubScriber: Second
            // First SubScriber: Third
            // Second Subscribe:  Second
            // Second Subscribe:  Third
            // First SubScriber: Fourth
            // Second Subscribe:  Fourth
            // First SubScriber: error(error1)
            // Second Subscribe:  error(error1)
            // Third Subscribe:  error(Object `RxSwift.(unknown context at $10573f460).ReplayMany<Swift.String>` was already disposed.)
            ```
    </div>
    </details>




























