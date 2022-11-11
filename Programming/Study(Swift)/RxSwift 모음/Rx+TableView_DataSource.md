# RxCocoa 코드 사용 방법 (RxTableViewReactiveArrayDataSource).md
- 독학 버전이라서 잘못된 정보도 많고 아직 다듬어야 할 부분도 엄청 많겠지만 수정해야 될 부분은 하나하나 사용해보면서 고치면 될듯하다.
- 일단 Rx 생각보다 더 어렵고 정보가 있어도 어떻게 내꺼로 만들어야 할 지 잘 몰라서 직접 하나하나 코드를 열어보고 사용해보면서 익숙해지는 수 밖에 없을듯 하다.

### RxTableViewReactiveArrayDataSource
- DataSource 이게 왜 있는지 전혀 모르고 있었는데 어떻게 쓰는건지 알아보기 위해서 공부를 하다보니 RxSwift 에서 Table 이나 Collection 을 더 Rx 스럽게 사용하기 위한 라이브러리라고 한다.
- 저 view 들에 데이터를 바인딩 할 때 하나의 섹션만 사용하는 거라면 기본적으로 RxCocoa 에서 제공하는 것으로 충분하다.
- 하지만 2개 이상의 섹션을 원하거나 row를 추가, 삭제, 수정 등의 기능에 적절한 Anime효과 까지 구현 할 생각이면 이게 적합할것이다.
    - 왜냐? 기본으로 주는거에서 저거 하기 어렵..


<details>
<summary> RxTableViewReactiveArrayDataSource 코드 </summary>
<div markdown="1">

```swift
    나는 이게 RxCocoa 안에 들어있는 소스 인줄 알았는데 RxDataSources 를 따로 Import 해줘야 함
```

</div>
</details>

----

## 학습 내용
- 여러 Section이 존재하거나, 데이터 변경의 경우 단순 Reload 로 데이터 바인딩 하기에 애니메이션 존재하지 않아 UX 떨어진다는 단점이 존재한다.

- tableView 구현 방식에는 크게 2가지 정도로 나타낼 수 있다.
    1. rx.items에 bind 하는 방법
    2. RxDataSource 를 사용하는 방법

- rx.items 는 이미 해보았으니 RxDataSource 를 이용하는 방법을 수행한다.
    
- RxDataSource 라이브러리에서 제공하는 것들은 다음과 같다.
    1. operator : rx.items(dataSource:)
    2. protocol : SectionModelType = 여러 Section 에 대해 데이터 설정할 수 있음
    3. class : DataSource = Section 에 대한 설정, animation 적용할 수 있음

### Model 정의
- DataSource 에서 사용되는 데이터 형은 AnimatableSectionModelType 를 준수해야 함
    ```swift
    public protocol AnimatableSectionModelType
        : SectionModelType
        , IdentifiableType where Item: IdentifiableType, Item: Equatable {
    }
    ```

- AnimatableSectionModelType 은 SectionModelType 를 준수하는 Protocol 이다
    ```swift
    public protocol SectionModelType {
        associatedtype Item

        var items: [Item] { get }

        init(original: Self, items: [Item])
    }
    ```
    - original 에 해당하는 Model 은 Section 에 해당
    - items 에 해당되는 인수는 rows 값

- AnimatableSectionModelType 에 준수하는 모델을 정의하기 전에 [Item] 타입에 들어갈 row 모델부터 정의
    - Item 은 IdentifiableType 과 Equatable 을 준수해야 함
    ```swift
    struct MyModel {
        var message: String
        var isDone: Bool = false
    }

    extension MyModel: IdentifiableType, Equatable {
        var identity: String { return UUID().uuidString }
            // 실제로는 데이터를 구분하는 id 사용 할것
    }

    struct MySection {
        var headerTitle: String
        var items: [Item]
    }

    extension MySection: AnimatableSectionModelType {
        typealias Item = MyModel
        
        var identity: String { return headerTitle }
        
        init(original: MySection, items: [MyModel]) {
            self = original
            self.items = items
        }
    }
    ```

### RxTableViewSectionReloadDataSource 구현
1. VC 준비
    - UI Components 배치

2. 데이터 준비
    - Model 코드 작성

3. RxTableViewSectionReloadDataSource 를 구현하기
    - 어느 변수에 해당 클래스를 구현하기
    - 해당 변수에 들어가는 코드는 row 데이터가 적용이 됨
    ```swift
    var dataSource = RxTableViewSectionReloadDataSource<MySection> { dataSource, tableView, indexPath, item in
        let cell = tableView.dequeueReusableCell(withIdentifier: tableViewCell.reuseIdentifier, for: indexPath) as! tableViewCell
        // cell 의 초기화 선언 부
        return cell
    }
    ```
    - 위 코드로 틀을 만들어고 주석 처리 부분에 내용을 집어 넣는것임, 구성 원본은 밑에 따로 작성하겠음

4. section 데이터는 dataSource.titleForHeaderInSection 으로 따로 설정을 해줘야 함
    ```swift
    public typealias TitleForHeaderInSection = (TableViewSectionedDataSource<Section>, Int) -> String?
    ```
    - 이렇게 코드를 구성을 해야 되는데 TableViewSectionedDataSource<Section> 이부분은 3번에서 만든 부분이기에 해당 부분을 넣어주면 되고 Int 의 값은 index 정도로 생각하면 될 듯 함

5. 이제 TableView 에 Cell 을 등록
    ```swift
    tableView.rx.register(tableViewCell.self, forCellReuseIdentifier: TableViewCell.reuseIdentifier)
    ```

6.  TableView 에 delegate 할당
    ```swift
    tableView.rx.setDelegate(self)
        .disposed(by: disposeBag)
    ```

7. BehaviorRelay 를 만들기
    ```swift
    var sectionRelay = BehaviorRelay(value: [MySection]())
    ```
    - 왜, BehaviorRelay 를 썼나?
        1. Relay 는 RxCocoa 의 클래스 (Subject 는 RxSwift 의 클래스)
        2. Relay 는 .completed, .error 를 발생시키지 않고 Dispose 되기 전까지 계속 동작하기에 UI 작업 용이함 (Subject 는 해당 이벤트가 발생하면 그 즉시 종료 됨)
        3. Behavior 는 초기값을 가지게 할 수 있고 값을 발행받기 전이라면 초기 값을 사용하기에 초기값이 없는 Publish 는 구독을 한 뒤 발생하는 값들에 반응을 하기에 데이터가 이미 있는 UI를 처음에 만들때는 문제가 있다고 생각했고 데이터가 없더라도 Behavior 의 초기값을 비워두면 되는것이다.

8. 7 에서 만든 Relay 에 초기값 입력
    ```swift
    sectionRelay.accept(initData)
    ```

9. 8 까지 마친 Relay 를 이용해서 TableView 에 데이터 소스 맵핑
    ```swift
    sectionRelay
        .bind(to: tableView.rx.items(dataSource: dataSource))
        .disposed(by: disposeBag)
    ```


### code 로 따져보자

1. RxTableViewSectionReloadDataSource.swift
    ```swift
    open class RxTableViewSectionedReloadDataSource<Section: SectionModelType>
        : TableViewSectionedDataSource<Section>
        , RxTableViewDataSourceType {
        public typealias Element = [Section]

        open func tableView(_ tableView: UITableView, observedEvent: Event<Element>) {
            Binder(self) { dataSource, element in
                #if DEBUG
                    dataSource._dataSourceBound = true
                #endif
                dataSource.setSections(element)
                tableView.reloadData()
            }.on(observedEvent)
        }
    }
    ```

| 이름 | 라이브러리 | 위치 |
|:-:|:-:|:-:|
| SectionModelType |  | |
| TableViewSectionDataSource | RxDataSources | Sources > RxDataSources |
| RxTableViewDataSourceType | RxSwift | RxCocoa > iOS > Protocols |
| 

2. 