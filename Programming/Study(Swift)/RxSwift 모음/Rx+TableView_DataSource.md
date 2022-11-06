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

#if os(iOS) || os(tvOS)

import UIKit
import RxSwift

// objc monkey business
class _RxTableViewReactiveArrayDataSource
    : NSObject
    , UITableViewDataSource {
    
    func numberOfSections(in tableView: UITableView) -> Int {
        1
    }
   
    func _tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        0
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        _tableView(tableView, numberOfRowsInSection: section)
    }

    fileprivate func _tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        rxAbstractMethod()
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        _tableView(tableView, cellForRowAt: indexPath)
    }
}


class RxTableViewReactiveArrayDataSourceSequenceWrapper<Sequence: Swift.Sequence>
    : RxTableViewReactiveArrayDataSource<Sequence.Element>
    , RxTableViewDataSourceType {
    typealias Element = Sequence

    override init(cellFactory: @escaping CellFactory) {
        super.init(cellFactory: cellFactory)
    }

    func tableView(_ tableView: UITableView, observedEvent: Event<Sequence>) {
        Binder(self) { tableViewDataSource, sectionModels in
            let sections = Array(sectionModels)
            tableViewDataSource.tableView(tableView, observedElements: sections)
        }.on(observedEvent)
    }
}

// Please take a look at `DelegateProxyType.swift`
class RxTableViewReactiveArrayDataSource<Element>
    : _RxTableViewReactiveArrayDataSource
    , SectionedViewDataSourceType {
    typealias CellFactory = (UITableView, Int, Element) -> UITableViewCell
    
    var itemModels: [Element]?
    
    func modelAtIndex(_ index: Int) -> Element? {
        itemModels?[index]
    }

    func model(at indexPath: IndexPath) throws -> Any {
        precondition(indexPath.section == 0)
        guard let item = itemModels?[indexPath.item] else {
            throw RxCocoaError.itemsNotYetBound(object: self)
        }
        return item
    }

    let cellFactory: CellFactory
    
    init(cellFactory: @escaping CellFactory) {
        self.cellFactory = cellFactory
    }
    
    override func _tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        itemModels?.count ?? 0
    }
    
    override func _tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        cellFactory(tableView, indexPath.item, itemModels![indexPath.row])
    }
    
    // reactive
    
    func tableView(_ tableView: UITableView, observedElements: [Element]) {
        self.itemModels = observedElements
        
        tableView.reloadData()
    }
}

#endif

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

### 사용법 
1. 사용을 할 dataSource를 미리 정의
2. 사용할 section 도 정의
3. dataSource 에 대한 기타 메소드도 구현 가능(선택)