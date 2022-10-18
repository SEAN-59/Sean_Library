# Realm

### Realm 사용시

- 렐름 사용시 여러 VC에서 Realm() 땡겨오면 오류나기 쉬움
- 그러니까 그냥 클래스 하나 정의해서 거기서 다 가능하게 만드는 방식도 있음
- 예를 들면 이렇게
    
    ```swift
    
    import Foundation
    import RealmSwift
    
    class RealmManager {
        private var realm: Realm
        static let sharedInstance = RealmManager()
        
        private init() {
            realm = try! Realm()
        }
        
        func defaultRealm() {
            guard let realm = try? Realm() else { return }
            print(realm.configuration.fileURL!)
        } // Realm 저장 위치 확인
        
        func readData(object: Object, key: String) -> Object {
            let objectType = type(of: object)
            guard let readData = realm.object(ofType: objectType.self, forPrimaryKey: key) else {
                let errorObject = ErrorObject()
                return errorObject
            }
            return readData
        }
        
        func add(object: Object, shouldUpdate: Bool = true){
            try! realm.write {
                if shouldUpdate {
                    realm.add(object, update: .all)
                } else {
                    realm.add(object)
                }
            }
        }
        func reset(object: Object, key: String) {
            let objectType = type(of: object)
            let objectToDelete = realm.objects(objectType.self).filter(NSPredicate(format: "type = %@", key))
            print(objectToDelete)
            
            try! realm.write({
                realm.delete(objectToDelete)
            })
        }
        func update(action: () -> Void) {
            try! realm.write {
                action()
            }
        }
        
        func deleteAll() {
            try! realm.write({
                realm.deleteAll()
            })
        }
        
        func deleteThing(object: Object) {
            try! realm.write({
                realm.delete(object)
            })
        }
        
        func cleanDelete() {
            let realmURL = Realm.Configuration.defaultConfiguration.fileURL!
            let realmURLs = [
                realmURL,
                realmURL.appendingPathExtension("lock"),
                realmURL.appendingPathExtension("note"),
                realmURL.appendingPathExtension("management")]
            for URL in realmURLs {
                do {
                    try FileManager.default.removeItem(at: URL)
                } catch {
    //                handler ERROR
                }
            }
        }
        
    }
    ```
    

## 생성

```swift
let realm = try! Realm()
let test = Test()
test.key = value

try! realm.write {
	realm.add(test)
}
```

## 업데이트

```swift
try! realm.write ({
	realm?.key = value
})
```

- 업데이트 하거나 할때 무조건 try! realm.write 를 꼭 해야함

## 삭제

- 데이터만 삭제하는 경우에는 아래의 코드를 사용하면 됨

```swift
try! realm.write ({
	realm.delete(realm.objects(Test.self).filter("key == 'value'"))
})
// or
try! realm.write ({
	realm.deleteAll()
})
//or 특정한거만 삭제하고자 할때는 이런식으로 하는게 좋음
let objectToDelete = realm.objects(objectType.self).filter(NSPredicate(format: "type = %@", key))
try! realm.write({
	realm.delete(objectToDelete)
}
```

- 데이터 말고 그냥 디비 자체를 싹 날려버리는 cleanDelete 를 하고 싶은 경우 아래 코드를 사용하면 됨
    - 단, 아래 코드 사용시 realm 파일 자체를 날려버리니까 다시 파일루트 찾아서 새로 확인해야 함

```swift
func cleanDelete() {
        guard let realm = try? Realm() else { return }
        print(realm.configuration.fileURL!)
        let realmURL = Realm.Configuration.defaultConfiguration.fileURL!
        let realmURLs = [
            realmURL,
            realmURL.appendingPathExtension("lock"),
            realmURL.appendingPathExtension("note"),
            realmURL.appendingPathExtension("management")]
        for URL in realmURLs {
            do {
                try FileManager.default.removeItem(at: URL)
            } catch {
//                handler ERROR
            }
        }
    }
```

## 읽기

```swift
let readThis = realm.objects(Test.self)
readThis.someKey
```

## Realm default 찍어보기

```swift
guard let realm = try? Realm() else { return }
print(realm.configuration.fileURL!)
```

## 내부에 배열 입력

```swift
// 구성을 이렇게 
class Dog: Object {
    @Persisted(primaryKey: true) var name: String = ""
    @Persisted var age: Int = 0   
}
class Human: Object {
    @Persisted(primaryKey: true) var name: String = ""
    @Persisted var dogs: List<Dog>
}

// 후에 데이터를 넣을떄는

let human = Human()
let dog = Dog()
human.dogs.append(dog)
```