# DispatchQueue

- 초기화 방법

```swift
DispatchQueue init (label: String, 
										qos: DispatchQoS = default, 
										attributes: DispatchQueue.Attributes = default,
										autoreleaseFrequency: DispatchQueue.AutoreleaseFrequency = default, 
										target: DispatchQueue? = default)
```

- 설정

```swift
let queue = DispatchQueue(label: "Name")
let queue = DispatchQueue(label: "Name", attributes: .concurrent)
```

```swift
DispatchQueue.global().sync {
//task
}
queue.sync {
//task
}
```

- 큐안이 끝날때까지 밖의 다른 작업을 하지 않음

```swift
DispatchQueue.global().async {
//task
}
queue.async {
//task
}
```

- 큐 안과 밖에 있는 건 결과가 매번 실행때마다 다름