# RxSwift 정리
### 1. RxSwift 란?
- Rx 는 비동기적으로 수시로 상태가 변하는 환경에서 직관적이고 효율적인 코드 작성을 도와줌
- 애플에서 제공하는 Notification, Delegate, GCD, Closure 보다 훨씬 직관적이고 효율적 표현 가능

### 2. 구성요소
- Observable
- Operator
- Scheduler : 직접 만들어서 사용하는 경우가 많지는 않음

### Observable
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
    - 결과에서 보면 알 수 있듯이 처음 .onNext 는 subscribe 가 되지 않아 출력이 되지 않으며 마지막 .onNext 는 dispose 되어 출력 되지 않음

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

### Operator
- RxSwift 에는 많은 Operator 가 존재한다.

<details>
<summary>1. Filtering Operator</summary>
<div markdown="1">

1. ignoreElements : onNex를 무시한다고 보면 됨
```swift
let sleepMode = PublishSubject<String>()

sleepMode
    .ignoreElements()
    .subscribe { _ in
        print("WAkE UP!!!")
    }.disposed(by: disposeBag)

sleepMode.onNext("WAKEEEEE")
sleepMode.onNext("WAKEEEEE")
sleepMode.onNext("WAKEEEEE")
sleepMode.onCompleted()

// WAKE UP!!
```

2. element(at:) : 특정 인덱스 번호(값이 아님!!!)에 대한 onNext 만 방출
```swift
let secondWake = PublishSubject<String>()

secondWake
    .element(at: 2)
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

secondWake.onNext("0")
secondWake.onNext("1")
secondWake.onNext("2")
secondWake.onNext("3")
secondWake.onNext("4")

// 2
```

3. filter : 조건에 부합하는 요소들만 방출
```swift
Observable.of(1,2,3,4,5,6,7,8)
    .filter { $0 % 2 == 0 }
    .subscribe(onNext : {
        print($0)
    }).disposed(by: disposeBag)
    
// 2
// 4
// 6
// 8
```

4. skip : 첫번째 요소부터 n번째 요소까지 스킵
```swift
Observable.of(1,2,3,4,5,6,7)
    .skip(5)
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)
    
// 6
// 7
```

5. skip(while:) : 어떤 요소를 내보내지 않다가 조건에 맞을 경우 내보냄
```swift
Observable.of(1,2,3,4,5,6,7,8,9)
    .skip(while: {
        $0 < 8
    })
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

// 8
// 9
```

6. skip(until:) : until에 있는 Observable이 방출되기 전까지 그 전의 동작에 대해서는 무시
```swift
let person = PublishSubject<String>()
let openTime = PublishSubject<String>()

person
    .skip(until: openTime)
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

person.onNext("First")
person.onNext("Second")
openTime.onNext("Open")
person.onNext("Third")

// Third
```

7. take : 첫번째부터 n번째까지만 내보내고 그 이후는 스킵 > skip의 반대 방식
```swift
Observable.of(1,2,3,4,5)
    .take(3)
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)
    
// 1
// 2
// 3
```

8. take(while:) : 요소들을 내보내다가 조건에 맞는 경우 요소를 내보내지 않음 > skip(while:)의 반대 방식
```swift
Observable.of(1,2,3,4,5)
    .take(while: {
        $0 != 3
    })
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)
    
// 1
// 2
```

9. enumerated : 방출될때 해당 요소의 인덱스까지 같이 방출함 > 방출된 요소의 인덱스 필요할 때 사용함
```swift
Observable.of(1,2,3,4,5)
    .enumerated()
    .take(while: {
        $0.index < 3
    }).subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)
    
// (index: 0, element: 1)
// (index: 1, element: 2)
// (index: 2, element: 3)
```

10. take(until:) : until에 들어있는 Observable이 방출된 후의 동작은 무시 > skip(until:)과 반대로 작용
```swift
let applicate = PublishSubject<String>()
let end = PublishSubject<String>()

applicate
    .take(until: end)
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

applicate.onNext("First")
applicate.onNext("Second")  
end.onNext("End")
applicate.onNext("Third")

// First
// Second
```

11. distinctUntilChanged : 연달아 같은 값이 입력될 때 중복된 값을 무시하는 필터
```swift
Observable.of(1,1,2,2,3,3,4,4,5,5,1,2,2,2,2,3)
    .distinctUntilChanged()
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)
    
// 1
// 2
// 3
// 4
// 5
// 1
// 2
// 3
```

</div>
</details>

<details>
<summary>2. Transforming Operator</summary>
<div markdown="1">

1. toArray : 독립적으로 많은 양의 요소를 넣어도 하나의 array로 나오게 만듬 > 요소를 배열로 append 같은거지 뭐.
```swift
Observable.of(1,2,3,4,5)
    .toArray()
    .subscribe(onSuccess: {
        print($0)
    } ).disposed(by: disposeBag)

// [1, 2, 3, 4, 5]
```

2. map : map = map
```swift
Observable.of(Date())
    .map { date -> String in
        let dateFormatter = DateFormatter()
        dateFormatter.dateFormat = "yyyy-MM-dd"
        dateFormatter.locale = Locale(identifier: "ko_KR")
        return dateFormatter.string(from: date)
    }.subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)
    
// 2022-08-10
```

3. flatMap : Observable 안에 Observable 같이 중첩되어 있는 경우에 이 명령어로 꺼내 볼 수 있음
- koreaMember 는 BowMan 이면서 Player 이므로 score를 보기 위해서 사용함
- 결과는 초기값이 있는 상황에서 koreaMember가 onNext 가 되었으므로 그 값이 나오고 후에 onNext 로 점수를 넣게되어 값이 방출됨
```swift
protocol Player {
    var score: BehaviorSubject<Int> { get }
}

struct BowMan: Player {
    var score: BehaviorSubject<Int>
}

let koreaMember = BowMan(score: BehaviorSubject<Int>(value: 10))
let amricaMember = BowMan(score: BehaviorSubject<Int>(value: 8))

let stadium = PublishSubject<Player>()

stadium
    .flatMap { Player in
        Player.score
    }.subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

stadium.onNext(koreaMember)
koreaMember.score.onNext(10)

stadium.onNext(amricaMember)
amricaMember.score.onNext(10)

// 10
// 10
// 8
// 10
```

4. flatMapLatest : 가장 최근의 Observable에서 값을 생성하고 그 이전에 발생한 Observable의 subscribe를 해지 > flat + switchLatest 의 형태라 보면 됨
- 결과에서 처음 seoul이 onNext 될 때 초기값이 7이 나오게 되고 다시 onNext 되었을 때 9가 나오게 된다.
- 하지만 두번째 contest에 jeju가 onNext 되면서 flatMapLatest 의 기능인 가장 최근의 Observable 에서 값을 생성하는 기능으로 seoul의 subscribe는 해지된다.
- 즉, 3번째 결과인 6은 jeju 의 초기값인 6이지만 4번째 결과인 8은 seoul.score.onNext(10)을 했음에도 
contest.onNext(jeju) 이 명령어로 해지가 되었기에 그 값은 방출되지 않고 바로 다음 명령어인 
jeju.score.onNext(8) 가 동작해 8이 방출되었다.

```swift
struct Jump: Player {
    var score: BehaviorSubject<Int>
}

let seoul = Jump(score: BehaviorSubject<Int>(value: 7))
let jeju = Jump(score: BehaviorSubject<Int>(value: 6))

let contest = PublishSubject<Player>()

contest
    .flatMapLatest { Player in
        Player.score
    }.subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

contest.onNext(seoul)
seoul.score.onNext(9)

contest.onNext(jeju)
seoul.score.onNext(10)
jeju.score.onNext(8)


// 7
// 9
// 6
// 8
```

5. materialize and dematerialize : Observable을 Observable의 이벤트로 변환해야 할 때 사용.
- 보통의 경우 Observable 속성을 가진 Observable을 제어할 수 없는데 외부적으로 Observable이 종료되는것을 방지하기 위해서 error 이벤트를 처리하고 싶을 수 있는 데 그때 사용함
```swift
enum Foul: Error {
    case FastStart
}

struct Runner: Player {
    var score: BehaviorSubject<Int>
}

let kim = Runner(score: BehaviorSubject<Int>(value: 0))
let lee = Runner(score: BehaviorSubject<Int>(value: 1))

let game = BehaviorSubject<Player>(value: kim)

game
    .flatMapLatest { Player in
        Player.score
            .materialize()
    }.subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

kim.score.onNext(1)
kim.score.onError(Foul.FastStart)
kim.score.onNext(2)

game.onNext(lee)

// next(0)
// next(1)
// error(FastStart)
// next(1)
```
----
- materialize를 사용하지 않을 경우
- 사용하지 않을 경우 Error가 발생하게 되면 그 순간 완전종료되어 그 다음 진행이 되지 않게 됨
```swift
enum Foul: Error {
    case FastStart
}

struct Runner: Player {
    var score: BehaviorSubject<Int>
}

let kim = Runner(score: BehaviorSubject<Int>(value: 0))
let lee = Runner(score: BehaviorSubject<Int>(value: 1))

let game = BehaviorSubject<Player>(value: kim)

game
    .flatMapLatest { Player in
        Player.score
    }.subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

kim.score.onNext(1)
kim.score.onError(Foul.FastStart)
kim.score.onNext(2)

game.onNext(lee)

// 0
// 1
// Unhandled error happened: FastStart
```
- dematerialize : materialize 에서 동작 과정까지 나오던 걸 없애고 보여준다.

```swift
enum Foul: Error {
    case FastStart
}

struct Runner: Player {
    var score: BehaviorSubject<Int>
}

let kim = Runner(score: BehaviorSubject<Int>(value: 0))
let lee = Runner(score: BehaviorSubject<Int>(value: 1))

let game = BehaviorSubject<Player>(value: kim)

game
    .flatMapLatest { Player in
        Player.score
            .materialize()
    }
    .filter {
        guard let error = $0.error else { return true}
        print(error)
        return false
    }.dematerialize()
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

kim.score.onNext(1)
kim.score.onError(Foul.FastStart)
kim.score.onNext(2)

game.onNext(lee)

// 0
// 1
// FastStart
// 1
```

</div>
</details>




<details>
<summary>3. Combining</summary>
<div markdown="1">

- 다양한 방법으로 시퀀스를 모으고 각각의 시퀀스 내 데이터들을 병합하는 방법

1. startWith : startWith에 들어간게 가장 먼저 방출 됨
```swift
let yellowClass = Observable.of("😁","🥲","🤭")

yellowClass
    .enumerated() // Observable 의 index 와 elements 를 분리 해주는 명령
    .map ({ index, element in
        return element + "어린이" + "\(index)"
    })
    .startWith("😆선생님") // 당연히 해당 Observable 과 동일한 타입으로 들어가야 함
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)
// 😆선생님
// 😁어린이0
// 🥲어린이1
// 🤭어린이2
```

2. concat : 지정된 순서에 맞게 방출 됨
- startWith는 concat의 변형이라 할 수 있음
```swift
let yellowClassChildren = Observable.of("😁","🥲","🤭")
let teacher = Observable.of("😆선생님")

let walkInLine = Observable
    .concat([teacher,yellowClassChildren])
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

// 😆선생님
// 😁
// 🥲
// 🤭
```
---
```swift
teacher
    .concat(yellowClassChildren)
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

// 😆선생님
// 😁
// 🥲
// 🤭
```

3. concatMap : flatMap과 밀접한 관계를 갖고 있다고 보면 됨
- 각각의 시퀀스가 다음 시퀀스가 구독이 되기전에 합쳐짐
- 각각의 시퀀스를 어떻게 append 할 수 있는가를 정하는게 이 역할
```swift
let careCenter: [String: Observable<String>] = [
    "Yellow" : Observable.of("😁","🥲","🤭"),
    "Blue" : Observable.of("🥶","🤖")
]
Observable.of("Yellow", "Blue")
    .concatMap { classes in
        careCenter[classes] ?? .empty()
    }.subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

// 😁
// 🥲
// 🤭
// 🥶
// 🤖
```

4. Merge : 시퀀스를 합치는 방식들 중에서 가장 쉬운 방법
- 합쳐지는 시퀀스의 요소는 순서를 보장하지는 않음
- merge를 사용하면 요소가 되는 Observable과 감싸고 있는 Observable이 끝이 나야 종료가 됨
- 내부에 존재하는 시퀀스 간의 관계는 없음
- 만약 하나라도 error가 발생을 하게 될 경우 그 즉시 error을 방출 후 멈추게 됨
```swift
let north = Observable.from(["강북구", "성북구", "동대문구", "종로구"])
let south = Observable.from(["강남구", "강동구", "영등포구", "양천구"])

Observable.of(north, south)
    .merge()
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

// 강북구
// 성북구
// 강남구
// 동대문구
// 강동구
// 종로구
// 영등포구
// 양천구
```
----
- 아래 코드에서의 결과값이 순서를 가진것처럼 보이겠지만 maxConcurrent에 적힌 수 만큼의 시퀀스만 한 번에 받기에 그렇게 보이는것 뿐이고 몇개가 되던 merge 안에 들어가게 되면 순서는 보장 할 수 없음
- 네트워크의 양이 많아질 경우, 그 연결 수를 제한하거나 할 때 사용할것 같음 
```swift
Observable.of(north, south)
    .merge(maxConcurrent: 1)
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

// 강북구
// 성북구
// 동대문구
// 종로구
// 강남구
// 강동구
// 영등포구
// 양천구
```

5. CombineLatest : 값을 방출할 때 마다 정해진 클로저를 호출하게 되고 받는 값은 combineLatest 의 최종값을 받게 됨
```swift
let lastName = PublishSubject<String>()
let firstName = PublishSubject<String>()

let name = Observable
    .combineLatest(lastName, firstName) { lastName, firstName in
        lastName + firstName
    }

name.subscribe(onNext: {
    print($0)
}).disposed(by: disposeBag)

lastName.onNext("Kim")
firstName.onNext("Sean")
firstName.onNext("Jo")
firstName.onNext("James")
lastName.onNext("Lee")
lastName.onNext("Park")
lastName.onNext("Choi")

// KimSean
// KimJo
// KimJames
// LeeJames
// ParkJames
// ChoiJames
```
----
```swift
let dateFormat = Observable<DateFormatter.Style>.of(.short, .long)
let currentDate = Observable<Date>.of(Date())

let nowDate = Observable
    .combineLatest(dateFormat,
                   currentDate,
                   resultSelector: { type, date -> String in
        let dateFormatter = DateFormatter()
        dateFormatter.dateStyle = type
        return dateFormatter.string(from: date)
    })

nowDate
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

// 10/26/22
// October 26, 2022
```
----
```swift
let fullName = Observable
    .combineLatest([firstName, lastName]) { name in
        name.joined(separator: " ")
    }

fullName
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

lastName.onNext("Kim")
firstName.onNext("Paul")
firstName.onNext("Stella")
firstName.onNext("Lily")

// KimJames
// KimPaul
// Paul Kim
// KimStella
// Stella Kim
// KimLily
// Lily Kim
```

6. zip : zip으로 합치는 시퀀스 들 중 어느 하나의 요소가 적어 먼저 완료되게 되면 zip 전체가 끝남
- 두 Observable 합치는데 5개, 10개의 시퀀스를 각각 갖고 있다면 5개만 합쳐지고 끝나게 됨
```swift
num VictoryOrDefeat {
    case victory
    case defeat
}

let fight = Observable<VictoryOrDefeat>.of(.victory,.defeat,.victory,.victory,.defeat)
let player = Observable<String>.of("🇰🇷","🇬🇩","🇳🇿","🇬🇬","🇦🇶","🇳🇱")

let result = Observable
    .zip(fight, player) { result, player in
        return player + "선수" + ": \(result)"
    }

result
    .subscribe(onNext: {
        print($0)
    }).disposed(by: disposeBag)

// 🇰🇷선수: victory
// 🇬🇩선수: defeat
// 🇳🇿선수: victory
// 🇬🇬선수: victory
// 🇦🇶선수: defeat
```


</div>
</details>


<details>
<summary>4. TimeBased</summary>
<div markdown="1">


</div>
</details>






<details>
<summary></summary>
<div markdown="1">


</div>
</details>














