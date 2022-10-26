# Notification 관련

- 연결 관계가 없는 각 뷰 간에 데이터 이동과 관련된 기능
- 등록
    - NotificationCenter.default.addObserver(_: ,selector: ,name: ,object:) 으로 등록을 하게 될 경우 name 의 이름으로 selector 의 동작을 행하는 observer가 등록이 됨
    - 다른 곳에서 전송 즉, post 를 하게 될경우 selector 의 함수가 호출되게 됨
    
    ```swift
    NotificationCenter.default.addObserver(self,
                                           selector: #selector(test(_:)),
                                           name: NSNotification.Name("SendTest"),
                                           object: nil)
    
    @objc func test(_ notification: Notification) {
            let get = notification.object
            print("get:\(get!)")
        }
    
    NotificationCenter.default.removeObserver(self)
    ```
    
    - 참고로 Notification 은 등록을 하게 되면 계속 등록이 됨
        - 즉, for 문이나 버튼같은거로 똑같은 addObserver 100번 돌리면 같은 이름의 notification 이 등록이 되어있어서 해당 이름으로 불러오게 되면 등록이 되어있는 100개가 한번에 불러와지는 참사가 발생하니 안쓰게 되는경우(cell을 deinit 한다거나) remove 하는게 좋음 → 그냥 버튼이나 반복문 같이 자주 돌려지는 곳에서는 안쓰는게 좋을듯 함
        - 뷰의 처음부터 불러와야 하는 경우에는 viewDidLoad() 있으니 거기가 좋겠네
        - 그리고 .object 의 값은 optional 값이니까 guard 를 쓰던 if 를 쓰던 바인딩해서 사용할것
- 전송
    - 예제의 코드같이 사용을 하면 됨
    - 만약 사용을 해야하는 곳에서 .post 를 해주게 되면 등록시 selector 에 지정해둔 함수가 실행될것임
    
    ```swift
    NotificationCenter.default.post(name: NSNotification.Name("SendTest"),
                                            object: ["Test" : "test2"],
                                            userInfo: nil)
    ```
    
- 결론
    - 이거 이야기 할때 방송국 이야기를 많이 하던데 내가 느낀게 맞다면
    addObserver 의 역할은 기지국의 역할을 한다 보면 되는거 같음 .post 에서 방송에 내보낼 파일을 이름 붙여서 보내면 해당 데이터를 바탕으로 정해진 프로그램 편성대로 맞춰서 내보내는거지
    - 근데 기지국은 기지국인데 1:1 통신으로 하는 기지국 같은 느낌이랄까? 딱 정해진 이름을 가진 .post에만 동작을 하는거지
    - 그런데 이름이 같은거지 데이터가 같지는 않음 예제에서 한거처럼 같은 name 을 사용하지만 다른 데이터 값이면 다르게 동작함