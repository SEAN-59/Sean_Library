# How to use **'RealmManager'**
Realm 은 워낙 유명하다 보니 구글링 조금만 해본다면 더 좋은 양질의 코드와 정보를 얻을 수 있기에 구글링을 추천한다.

간단한 예시 코드
```
guard let createDB = RealmManager.sharedInstance.readData(object: Object, key: PrimaryKey) as? Object else {
    createDB.SomeThing = "anyThing"
    RealmManager.sharedInstance.create(object:createDB)
    return
}
```