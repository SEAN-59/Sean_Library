# Data 저장 방법

## 종류
1. UserDefaults
2. CoreData
3. OpenSource 이용 : Realm 등
4. Network DB :  FireBase 등

----

## UserDefaults

[UserDefaults의 공식 문서](https://developer.apple.com/documentation/foundation/userdefaults)

1. UserDefaults 를 통해 plist 에 저장!
2. 사용자의 정보를 저장하는 싱글톤 
3. 간단한 사용자 정보 저장 및 불러오기 가능하지만 내부적 plist 파일에 저장되기에 보안이 강력하지는 않음

- UserDefaults 의 주요 항목!
    ```swift
    class var standard: UserDefaults
        
    func object(forKey: String) -> Any?
    func url(forKey: String) -> URL?
    func array(forKey: String) -> [Any]?
    func dictionary(forKey: String) -> [String : Any]?
    func string(forKey: String) -> String?
    func stringArray(forKey: String) -> [String]?
        
    func set(Any?, forKey: String)
    func set(Float, forKey: String)
    func set(Double, forKey: String)

    func removeObject(forKey: String)
    ```
- 보면 알겠지만 String 이나 Int 같은 기본 타입은 내부적으로 다 적용이 되서 바로 사용이 가능은 한데 struct를 따로 구현을 해야 되는 경우에는 별도의 작업이 필요로 하다.
(그외의 다른 동작을 알아보고 싶으면 위에 문서 들어가서 하나하나 함수 확인할 것)
- 해당 struct는 Codable 프로토콜 준수를 해야 함

- 사용
    ```swift
    let userDefaults = UserDefaults.standard

    self.userDefaults.set("No.1", forKey: "key")

    self.userDefaults.object(forKey: "key")
    ```
- 싱글턴 객체이기에 변수로 잡아주고 set과 object 등 여러 함수를 이용하여 저장하거나 불러오기가 가능함

### 중요!
- 하지만 저장하려는 데이터가 언제나 저렇게 기본 타입일리 없음
- 아카이빙과 언아카이빙 작업이 별도로 필요로 하게 됨 : 안하면 에러요!
    - 아카이빙 : 객체를 Data 형과 같이 바이트형태로 변경하는 작업 -> 메모리 저장 가능하게 해줌
    - 언아카이빙 : 아카이빙의 반대
- 이 작업을 위해서 struct 는 Codable 프로토콜을 준수해야 함
- 이렇게 처리를 바로 해주고 싶어도 이렇게 바로 하게 되면 에러가 딱하고 뜨게 됨
    ```swift
    userDefaults.setValue(<value: Any>, forKey: <String>)
    ```

 - 그래서 JSONEncoder를 이용해 객체를 아카이빙 시켜야 가능함
    ```swift
    let encoder = JSONEncoder()
    if let encoded = try? encoder.encode(firstData) {
        self.userDefaults.setValue(encoded, forKey: firstData.titleLabel)
    }
    ```
- 이런식으로 아카이빙을 하고 저장을 하고 나면 이제 불러올때는 언아카이빙을 해야함
- 아카이빙은 JSONEncoder 였으니 언아카이빙은 JSONDeCoder를 이용
    ```swift
    let decoder = JSONDecoder()
    if let savedData = self.userDefaults.object(forKey: firstData.titleLabel) as? Data {
        if let decoded = try? decoder.decode(MainData.self, from: savedData){
            print(decoded)
        }
    }
    ```

- 최종은 이런식으로 사용하면 됨
    ```swift
     if let savedData = self.userDefaults.object(forKey: firstData.titleLabel) as? Data {
        if let decoded = try? decoder.decode(MainData.self, from: savedData){
            print(decoded)
        }
    } else {
        if let encoded = try? encoder.encode(firstData) {
            self.userDefaults.setValue(encoded, forKey: firstData.titleLabel)
        }
    }
    ```
- 데이터가 저장되어있으면 불러오고 저장되어 있지 않으면 저장하는 방식이 되는것


## Update
- 따로 Update 라는 개념이 없는듯 함 그냥 Set 으로 키 값에 덮어 씌우면 끝남