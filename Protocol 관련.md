# Protocol 관련

### 데이터 주거니 받거니

```swift
protocol SendData {
	func sendData(data: String)
}
```

- 위의 프로토콜을 이용해 설명

```swift
class firstVC: UIViewController {
	var delegate = SendData?
	override func	viewDidLoad() {
		super.viewDidLoad()
		delegate?.dataSend(data: "TestString")
	}
}
```

```swift
class firstVC: UIViewController, SendData {
	override func	viewDidLoad() {
		super.viewDidLoad()
		let vc = firstVC()
		vc.delegate = self
		delegate?.dataSend(data: "TestString")
	}

	func sendData(data: String) {
		//data를 이용하면 됨
	}
}
```

- 이렇게 구현해서 사용하는 방식이 있음, 이방식에 대해서 조금더 자세한 사항은 더 많은 코드를 작성해보면서 연습하면 될듯 함