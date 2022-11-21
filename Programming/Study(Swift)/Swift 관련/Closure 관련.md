# Closure 관련

### Closure 설명

```swift
let testClosure (name: String, handler: { check: Bool in 
		return true
})
```

- 위의 코드는 내가 그냥 한 번 짜본 코드이지만 대략적으로 해석을 해보자면 testClosure 는 name 과 handler 를 입력인자로 받고 있는데 name 은 string 값을 받을 것이며 handler 는  클로저 의 반환 값인 check 를 값으로 할것이다.
- 즉, 클로저는 함수랑 다를게 없음 in 앞에 써지는거로 클로저의 동작방식이 달라지겠지만 그래도 결국에는 함수가 작게 바뀐 느낌임

```swift
let testClosure (name: String, handler: { (first: Int, sedond: Int) -> check: Bool in 
		return first < second
})
```

- 함수와 다를게 없다는건 결국 위에처럼 해도 동작 한다는 소리고 해석 방식도 다를게 없음을 의미함
- 만약 따로 반환해줄게 없다거나 하는 경우 그냥 _ :  로 퉁쳐도 됨

---

### Escaping Closure

```swift
class SomeClass {
	func fetchData(completion: @escaping () -> Void) {
		...
	}
}
```

- 실행 순서
    1. fetchData() 의 completion 인자로 클로저가 전달
    2. fetchData 내부 명령 실행
    3. fetchData() 함수가 값을 반환하고 종료
    4. 클로저 completion은 아직 실행 안됨
    - completion은 함수의 실행이 종료되기 전에 실행되지 않음 : escaping closure
        - 즉, 함수 밖에서 실행이 되는 클로저 임
- escaping 이 붙은 클로저는 반드시 escaping 클로저만 인자로 사용을 해야 하지는 않음: non-escaping 클로저 인자 사용 가능
- 하지만 반대로 escaping 클로저를 @escaping 없이 사용 안됨
- 그럼 그냥 타입에 붙여서 사용하면 둘다 사용 가능한데 왜 안쓰냐?
    - 간단함 : 컴파일러의 퍼포먼스와 최적화 때문
    - non-escaping closure 는 컴파일러가 클로저 실행이 언제 종료되는지 알기에 라이프사이클을 효율적 관리가 가능하나 escaping closure는 함수 밖에서 실행되기 때문에 클로저가 함수 밖에서도 적절히 실행되는 것을 보장해야 하기에 추가적인 참조사이클 관리 등을 해줘야 함