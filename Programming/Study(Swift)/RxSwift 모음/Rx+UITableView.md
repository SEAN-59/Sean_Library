# RxCocoa 코드 사용 방법 (UITableView).md
- 독학 버전이라서 잘못된 정보도 많고 아직 다듬어야 할 부분도 엄청 많겠지만 수정해야 될 부분은 하나하나 사용해보면서 고치면 될듯하다.
- 일단 Rx 생각보다 더 어렵고 정보가 있어도 어떻게 내꺼로 만들어야 할 지 잘 몰라서 직접 하나하나 코드를 열어보고 사용해보면서 익숙해지는 수 밖에 없을듯 하다.

### RxCocoa + UITableView 
- 처음으로 파볼 요소는 UITableView 요소이다.

<details>
<summary> UITableView+Rx 코드 </summary>
<div markdown="1">

```swift
#if os(iOS) || os(tvOS)

import RxSwift
import UIKit

// Items

extension Reactive where Base: UITableView {

    /**
    Binds sequences of elements to table view rows.
    
    - parameter source: Observable sequence of items.
    - parameter cellFactory: Transform between sequence elements and view cells.
    - returns: Disposable object that can be used to unbind.
     
     Example:
    
         let items = Observable.just([
             "First Item",
             "Second Item",
             "Third Item"
         ])

         items
         .bind(to: tableView.rx.items) { (tableView, row, element) in
             let cell = tableView.dequeueReusableCell(withIdentifier: "Cell")!
             cell.textLabel?.text = "\(element) @ row \(row)"
             return cell
         }
         .disposed(by: disposeBag)

     */
    public func items<Sequence: Swift.Sequence, Source: ObservableType>
        (_ source: Source)
        -> (_ cellFactory: @escaping (UITableView, Int, Sequence.Element) -> UITableViewCell)
        -> Disposable
        where Source.Element == Sequence {
            return { cellFactory in
                let dataSource = RxTableViewReactiveArrayDataSourceSequenceWrapper<Sequence>(cellFactory: cellFactory)
                return self.items(dataSource: dataSource)(source)
            }
    }

    /**
    Binds sequences of elements to table view rows.
    
    - parameter cellIdentifier: Identifier used to dequeue cells.
    - parameter source: Observable sequence of items.
    - parameter configureCell: Transform between sequence elements and view cells.
    - parameter cellType: Type of table view cell.
    - returns: Disposable object that can be used to unbind.
     
     Example:

         let items = Observable.just([
             "First Item",
             "Second Item",
             "Third Item"
         ])

         items
             .bind(to: tableView.rx.items(cellIdentifier: "Cell", cellType: UITableViewCell.self)) { (row, element, cell) in
                cell.textLabel?.text = "\(element) @ row \(row)"
             }
             .disposed(by: disposeBag)
    */
    public func items<Sequence: Swift.Sequence, Cell: UITableViewCell, Source: ObservableType>
        (cellIdentifier: String, cellType: Cell.Type = Cell.self)
        -> (_ source: Source)
        -> (_ configureCell: @escaping (Int, Sequence.Element, Cell) -> Void)
        -> Disposable
        where Source.Element == Sequence {
        return { source in
            return { configureCell in
                let dataSource = RxTableViewReactiveArrayDataSourceSequenceWrapper<Sequence> { tv, i, item in
                    let indexPath = IndexPath(item: i, section: 0)
                    let cell = tv.dequeueReusableCell(withIdentifier: cellIdentifier, for: indexPath) as! Cell
                    configureCell(i, item, cell)
                    return cell
                }
                return self.items(dataSource: dataSource)(source)
            }
        }
    }


    /**
    Binds sequences of elements to table view rows using a custom reactive data used to perform the transformation.
    This method will retain the data source for as long as the subscription isn't disposed (result `Disposable` 
    being disposed).
    In case `source` observable sequence terminates successfully, the data source will present latest element
    until the subscription isn't disposed.
    
    - parameter dataSource: Data source used to transform elements to view cells.
    - parameter source: Observable sequence of items.
    - returns: Disposable object that can be used to unbind.
    */
    public func items<
            DataSource: RxTableViewDataSourceType & UITableViewDataSource,
            Source: ObservableType>
        (dataSource: DataSource)
        -> (_ source: Source)
        -> Disposable
        where DataSource.Element == Source.Element {
        return { source in
            // This is called for side effects only, and to make sure delegate proxy is in place when
            // data source is being bound.
            // This is needed because theoretically the data source subscription itself might
            // call `self.rx.delegate`. If that happens, it might cause weird side effects since
            // setting data source will set delegate, and UITableView might get into a weird state.
            // Therefore it's better to set delegate proxy first, just to be sure.
            _ = self.delegate
            // Strong reference is needed because data source is in use until result subscription is disposed
            return source.subscribeProxyDataSource(ofObject: self.base, dataSource: dataSource as UITableViewDataSource, retainDataSource: true) { [weak tableView = self.base] (_: RxTableViewDataSourceProxy, event) -> Void in
                guard let tableView = tableView else {
                    return
                }
                dataSource.tableView(tableView, observedEvent: event)
            }
        }
    }

}

extension Reactive where Base: UITableView {
    /**
    Reactive wrapper for `dataSource`.
    
    For more information take a look at `DelegateProxyType` protocol documentation.
    */
    public var dataSource: DelegateProxy<UITableView, UITableViewDataSource> {
        RxTableViewDataSourceProxy.proxy(for: base)
    }
   
    /**
    Installs data source as forwarding delegate on `rx.dataSource`.
    Data source won't be retained.
    
    It enables using normal delegate mechanism with reactive delegate mechanism.
     
    - parameter dataSource: Data source object.
    - returns: Disposable object that can be used to unbind the data source.
    */
    public func setDataSource(_ dataSource: UITableViewDataSource)
        -> Disposable {
        RxTableViewDataSourceProxy.installForwardDelegate(dataSource, retainDelegate: false, onProxyForObject: self.base)
    }
    
    // events
    
    /**
    Reactive wrapper for `delegate` message `tableView:didSelectRowAtIndexPath:`.
    */
    public var itemSelected: ControlEvent<IndexPath> {
        let source = self.delegate.methodInvoked(#selector(UITableViewDelegate.tableView(_:didSelectRowAt:)))
            .map { a in
                return try castOrThrow(IndexPath.self, a[1])
            }

        return ControlEvent(events: source)
    }

    /**
     Reactive wrapper for `delegate` message `tableView:didDeselectRowAtIndexPath:`.
     */
    public var itemDeselected: ControlEvent<IndexPath> {
        let source = self.delegate.methodInvoked(#selector(UITableViewDelegate.tableView(_:didDeselectRowAt:)))
            .map { a in
                return try castOrThrow(IndexPath.self, a[1])
            }

        return ControlEvent(events: source)
    }
    
    /**
     Reactive wrapper for `delegate` message `tableView:didHighlightRowAt:`.
     */
    public var itemHighlighted: ControlEvent<IndexPath> {
        let source = self.delegate.methodInvoked(#selector(UITableViewDelegate.tableView(_:didHighlightRowAt:)))
            .map { a in
                return try castOrThrow(IndexPath.self, a[1])
            }

        return ControlEvent(events: source)
    }

    /**
     Reactive wrapper for `delegate` message `tableView:didUnhighlightRowAt:`.
     */
    public var itemUnhighlighted: ControlEvent<IndexPath> {
        let source = self.delegate.methodInvoked(#selector(UITableViewDelegate.tableView(_:didUnhighlightRowAt:)))
            .map { a in
                return try castOrThrow(IndexPath.self, a[1])
            }

        return ControlEvent(events: source)
    }

    /**
     Reactive wrapper for `delegate` message `tableView:accessoryButtonTappedForRowWithIndexPath:`.
     */
    public var itemAccessoryButtonTapped: ControlEvent<IndexPath> {
        let source: Observable<IndexPath> = self.delegate.methodInvoked(#selector(UITableViewDelegate.tableView(_:accessoryButtonTappedForRowWith:)))
            .map { a in
                return try castOrThrow(IndexPath.self, a[1])
            }
        
        return ControlEvent(events: source)
    }
    
    /**
    Reactive wrapper for `delegate` message `tableView:commitEditingStyle:forRowAtIndexPath:`.
    */
    public var itemInserted: ControlEvent<IndexPath> {
        let source = self.dataSource.methodInvoked(#selector(UITableViewDataSource.tableView(_:commit:forRowAt:)))
            .filter { a in
                return UITableViewCell.EditingStyle(rawValue: (try castOrThrow(NSNumber.self, a[1])).intValue) == .insert
            }
            .map { a in
                return (try castOrThrow(IndexPath.self, a[2]))
        }
        
        return ControlEvent(events: source)
    }
    
    /**
    Reactive wrapper for `delegate` message `tableView:commitEditingStyle:forRowAtIndexPath:`.
    */
    public var itemDeleted: ControlEvent<IndexPath> {
        let source = self.dataSource.methodInvoked(#selector(UITableViewDataSource.tableView(_:commit:forRowAt:)))
            .filter { a in
                return UITableViewCell.EditingStyle(rawValue: (try castOrThrow(NSNumber.self, a[1])).intValue) == .delete
            }
            .map { a in
                return try castOrThrow(IndexPath.self, a[2])
            }
        
        return ControlEvent(events: source)
    }
    
    /**
    Reactive wrapper for `delegate` message `tableView:moveRowAtIndexPath:toIndexPath:`.
    */
    public var itemMoved: ControlEvent<ItemMovedEvent> {
        let source: Observable<ItemMovedEvent> = self.dataSource.methodInvoked햣 (#selector(UITableViewDataSource.tableView(_:moveRowAt:to:)))
            .map { a in
                return (try castOrThrow(IndexPath.self, a[1]), try castOrThrow(IndexPath.self, a[2]))
            }
        
        return ControlEvent(events: source)
    }

    /**
    Reactive wrapper for `delegate` message `tableView:willDisplayCell:forRowAtIndexPath:`.
    */
    public var willDisplayCell: ControlEvent<WillDisplayCellEvent> {
        let source: Observable<WillDisplayCellEvent> = self.delegate.methodInvoked(#selector(UITableViewDelegate.tableView(_:willDisplay:forRowAt:)))
            .map { a in
                return (try castOrThrow(UITableViewCell.self, a[1]), try castOrThrow(IndexPath.self, a[2]))
            }

        return ControlEvent(events: source)
    }

    /**
    Reactive wrapper for `delegate` message `tableView:didEndDisplayingCell:forRowAtIndexPath:`.
    */
    public var didEndDisplayingCell: ControlEvent<DidEndDisplayingCellEvent> {
        let source: Observable<DidEndDisplayingCellEvent> = self.delegate.methodInvoked(#selector(UITableViewDelegate.tableView(_:didEndDisplaying:forRowAt:)))
            .map { a in
                return (try castOrThrow(UITableViewCell.self, a[1]), try castOrThrow(IndexPath.self, a[2]))
            }

        return ControlEvent(events: source)
    }

    /**
    Reactive wrapper for `delegate` message `tableView:didSelectRowAtIndexPath:`.
    
    It can be only used when one of the `rx.itemsWith*` methods is used to bind observable sequence,
    or any other data source conforming to `SectionedViewDataSourceType` protocol.
    
     ```
        tableView.rx.modelSelected(MyModel.self)
            .map { ...
     ```
    */
    public func modelSelected<T>(_ modelType: T.Type) -> ControlEvent<T> {
        let source: Observable<T> = self.itemSelected.flatMap { [weak view = self.base as UITableView] indexPath -> Observable<T> in
            guard let view = view else {
                return Observable.empty()
            }

            return Observable.just(try view.rx.model(at: indexPath))
        }
        
        return ControlEvent(events: source)
    }

    /**
     Reactive wrapper for `delegate` message `tableView:didDeselectRowAtIndexPath:`.

     It can be only used when one of the `rx.itemsWith*` methods is used to bind observable sequence,
     or any other data source conforming to `SectionedViewDataSourceType` protocol.

     ```
        tableView.rx.modelDeselected(MyModel.self)
            .map { ...
     ```
     */
    public func modelDeselected<T>(_ modelType: T.Type) -> ControlEvent<T> {
         let source: Observable<T> = self.itemDeselected.flatMap { [weak view = self.base as UITableView] indexPath -> Observable<T> in
             guard let view = view else {
                 return Observable.empty()
             }

            return Observable.just(try view.rx.model(at: indexPath))
        }

        return ControlEvent(events: source)
    }
    
    /**
     Reactive wrapper for `delegate` message `tableView:commitEditingStyle:forRowAtIndexPath:`.
     
     It can be only used when one of the `rx.itemsWith*` methods is used to bind observable sequence,
     or any other data source conforming to `SectionedViewDataSourceType` protocol.
     
     ```
        tableView.rx.modelDeleted(MyModel.self)
            .map { ...
     ```
     */
    public func modelDeleted<T>(_ modelType: T.Type) -> ControlEvent<T> {
        let source: Observable<T> = self.itemDeleted.flatMap { [weak view = self.base as UITableView] indexPath -> Observable<T> in
            guard let view = view else {
                return Observable.empty()
            }
            
            return Observable.just(try view.rx.model(at: indexPath))
        }
        
        return ControlEvent(events: source)
    }

    /**
     Synchronous helper method for retrieving a model at indexPath through a reactive data source.
     */
    public func model<T>(at indexPath: IndexPath) throws -> T {
        let dataSource: SectionedViewDataSourceType = castOrFatalError(self.dataSource.forwardToDelegate(), message: "This method only works in case one of the `rx.items*` methods was used.")
        
        let element = try dataSource.model(at: indexPath)

        return castOrFatalError(element)
    }
}

@available(iOS 10.0, tvOS 10.0, *)
extension Reactive where Base: UITableView {

    /// Reactive wrapper for `prefetchDataSource`.
    ///
    /// For more information take a look at `DelegateProxyType` protocol documentation.
    public var prefetchDataSource: DelegateProxy<UITableView, UITableViewDataSourcePrefetching> {
        RxTableViewDataSourcePrefetchingProxy.proxy(for: base)
    }

    /**
     Installs prefetch data source as forwarding delegate on `rx.prefetchDataSource`.
     Prefetch data source won't be retained.

     It enables using normal delegate mechanism with reactive delegate mechanism.

     - parameter prefetchDataSource: Prefetch data source object.
     - returns: Disposable object that can be used to unbind the data source.
     */
    public func setPrefetchDataSource(_ prefetchDataSource: UITableViewDataSourcePrefetching)
        -> Disposable {
            return RxTableViewDataSourcePrefetchingProxy.installForwardDelegate(prefetchDataSource, retainDelegate: false, onProxyForObject: self.base)
    }

    /// Reactive wrapper for `prefetchDataSource` message `tableView(_:prefetchRowsAt:)`.
    public var prefetchRows: ControlEvent<[IndexPath]> {
        let source = RxTableViewDataSourcePrefetchingProxy.proxy(for: base).prefetchRowsPublishSubject
        return ControlEvent(events: source)
    }

    /// Reactive wrapper for `prefetchDataSource` message `tableView(_:cancelPrefetchingForRowsAt:)`.
    public var cancelPrefetchingForRows: ControlEvent<[IndexPath]> {
        let source = prefetchDataSource.methodInvoked(#selector(UITableViewDataSourcePrefetching.tableView(_:cancelPrefetchingForRowsAt:)))
            .map { a in
                return try castOrThrow(Array<IndexPath>.self, a[1])
        }

        return ControlEvent(events: source)
    }

}
#endif

#if os(tvOS)
    
    extension Reactive where Base: UITableView {
        
        /**
         Reactive wrapper for `delegate` message `tableView:didUpdateFocusInContext:withAnimationCoordinator:`.
         */
        public var didUpdateFocusInContextWithAnimationCoordinator: ControlEvent<(context: UITableViewFocusUpdateContext, animationCoordinator: UIFocusAnimationCoordinator)> {
            
            let source = delegate.methodInvoked(#selector(UITableViewDelegate.tableView(_:didUpdateFocusIn:with:)))
                .map { a -> (context: UITableViewFocusUpdateContext, animationCoordinator: UIFocusAnimationCoordinator) in
                    let context = try castOrThrow(UITableViewFocusUpdateContext.self, a[1])
                    let animationCoordinator = try castOrThrow(UIFocusAnimationCoordinator.self, a[2])
                    return (context: context, animationCoordinator: animationCoordinator)
            }
            
            return ControlEvent(events: source)
        }
    }
#endif

```

</div>
</details>

----

### 구현부

- 해당 코드를 하나하나 주석 처리 된것들을 읽어보며 사용을 설명을 달아보자
1. 요소의 시퀀스를 테이블 뷰에 바인딩 하는 방법

```swift
let items = Observable.just([
             "First Item",
             "Second Item",
             "Third Item"
         ])

items
    .bind(to: tableView.rx.items) { (tableView, row, element) in
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell")!
        cell.textLabel?.text = "\(element) @ row \(row)"
        return cell
    }.disposed(by: disposeBag)
```

- 이 코드가 사용법인거 같은데 items에 bind 걸어주면 tableView, row, element 로 구성된 클로저를 구현할 수 있게된다.
- 이때 items는 Observable<[String]> 타입이 된다.
- 해당 소스를 기반해서 직접 코드를 구현해보면 아래와 같이 쓸 수 있다.

```swift
bind(viewModel(items: ["first", "second", "third"]))

bind() {
    viewModel.getCellData().bind(to: tableView.rx.items) { (tableView: UITableView, row: Int, element: String) in
            guard let cell = tableView.dequeueReusableCell(withIdentifier: viewCell.reuseIdentifier) else { fatalError() }
            cell.textLabel?.text = "\(element) @ row \(row)"
            return cell
        }.disposed(by: disposeBag)
}
```

2. 기본 cell을 사용하는 것이 아닌 customCell을 사용
```swift
let items = Observable.just([
             "First Item",
             "Second Item",
             "Third Item"
         ])

items
    .bind(to: tableView.rx.items(cellIdentifier: "Cell", cellType: UITableViewCell.self)) { (row, element, cell) in
        cell.textLabel?.text = "\(element) @ row \(row)"
    }.disposed(by: disposeBag)
```

- 1번과 비슷한 방식 단 기본 cell을 사용하지 않고 Custom Cell 사용시 이 방법을 사용해야 함

```swift
class viewCell: UITableViewCell { ... }

bind() {
    viewModel.getCellData().bind(to: tableView.rx.items(cellIdentifier: viewCell.reuseIdentifier, cellType: viewCell.self)) { (row : Int, element: String, cell: viewCell) in
        cell.textLabel?.text = "\(element) @ row \(row)"
    }.disposed(by: disposeBag)
}
```
- 당연 결과는 1과 같은 값이 출력이 되지만 사용법이 다르기에 내가 만약에 직접 Cell을 하나하나 다 구성해서 하나하나 다 맞춰야 한다면 이 방법을 쓰는게 훨씬 수월 할 듯 함

----

### 기능 부

- TableView 를 구성하는 방법은 이 쯤 하면 될것 같고 다음은 delegate 로서 사용했던 여러 기능들에 대해서 알아보자

1. itemSelected : cell 선택시


```swift
public var itemSelected: ControlEvent<IndexPath> {
    let source = self.delegate.methodInvoked(#selector(UITableViewDelegate.tableView(_:didSelectRowAt:)))
        .map { a in
            return try castOrThrow(IndexPath.self, a[1])
        }

    return ControlEvent(events: source)
}
```

- 설명은 위와 같이 되어있으나 해당 코드의 사용은 쉽다.

```swift
tableView.rx.itemSelected
    .subscribe(onNext: { indexPath in
        print("select \(indexPath.row)")
    }).disposed(by: disposeBag)
``` 

- 해당 코드의 경우 해당 셀을 누르게 되면 그 즉시 해당 셀의 셀렉을 푸는 동작을 수행한다 
- itemSelected 같은 경우에는 Delegate의 didSelectRowAt 함수와 같은 동작을 한다.

2. itemDeselected : Cell의 select 해제시 

```swift
public var itemDeselected: ControlEvent<IndexPath> {
    let source = self.delegate.methodInvoked(#selector(UITableViewDelegate.tableView(_:didDeselectRowAt:)))
        .map { a in
            return try castOrThrow(IndexPath.self, a[1])
        }
        return ControlEvent(events: source)
}
```

```swift
tableView.rx.itemDeselected
    .subscribe(onNext:  { indexPath in
        print("deselect \(indexPath.row)")
    }).disposed(by: disposeBag)
```

- Delegate 의 didDeselectRowAtIndexPath 함수와 같은 동작을 수행하며 해당 셀의 선택이 풀릴 경우에 동작한다.

3. itemUnhighlighted : Delegate의 didUnhighlightRowAt 함수와 같은 역할을 한다.
4. itemAccessoryButtonTapped : Delegate의 accessoryButtonTapped 와 같은 동작
5. itemInserted : Delegate의 commitEditingStyle 
6. itemDeleted : Delegate의 commitEditingStyle
7. itemMoved : Delegate의 moveRowAtIndexPath
8. willDisplayCell : Delegate의 willDisplayCell
9. didEndDisplayingCell : Delegate의 didEndDisplayingCell

- 9번까지는 1,2 번과 방식이 거의 다 똑같고 문서에서도 딱히 특별한 내용이 적힌게 없어서 어떠한 Delegate의 함수와 같은 동작을 하는지만 적어두겠다.

10. modelSelected : itemSelected 랑 같은 기능을 수행하는데 방출되는게 model? 이라 해야하나 타입이라 해야하나 조금 다른걸 내보낼 수 있음
```swift
public func modelSelected<T>(_ modelType: T.Type) -> ControlEvent<T> {
        let source: Observable<T> = self.itemSelected.flatMap { [weak view = self.base as UITableView] indexPath -> Observable<T> in
            guard let view = view else {
                return Observable.empty()
            }
            return Observable.just(try view.rx.model(at: indexPath))
        }   
        return ControlEvent(events: source)
    }
```
```swift
tableView.rx.modelSelected(MyModel.self)
    .map { ... 
```
- 공식에는 이런식으로 되어있어서 참고해서 쓰면 될것 같은데 아직 이 함수를 어떻게 써야 하고 저 모델 타입에는 어떠한 타입을 만들어 넣어야 되는지 파악을 못함 (이해 했음)

```swift
 tableView.rx.modelSelected(Member.self)
    .subscribe(onNext: { member in
        print("\(member) good")
    }).disposed(by:disposeBag)
```
- 이 녀석은 위의 코드만 보면 딱 바로 이게 어떻게 뭘 참조하는건지 안 들어 올 수 있는데 (나는 그랬음) 일단 tableView의 model 에 대해서 알 필요가 있음

```swift
struct Member {
    var item1: String
    var item2: String
}

final class D_DayViewModel {
    
    private var itemArray: [Member]
    
    init(itemArray: [Member]) {
        self.itemArray = itemArray
    }
    
    func getCellData() -> Observable<[Member]> {
        return Observable.of(itemArray)
    }
}
```

- 이런식으로 구성이 되어 있고 tableView는 getCellData() 로 리턴 되는 Observable로 만들어지기에 저기에 들어가는 타입이 modelSelected 에서 Type.self 에 들어가는 타입이 될거임
- 이게 맞지 않으면 아무리 값이 멀쩡하게 있어도 나중에 Fatal error: Failure converting from Optional() to () 라면서 Fatal error가 뜨게 되니까 해당 함수를 사용하게 될때는 타입 잘 맞출것
- 얘는 itemSelected 와 다른 점으로는 해당 셀에 있는 값들을 불러 올 수 있다는게 정말 큰 장점이라 생각한다.

11. modelDeselected : itemDeselected 
12. modelDeleted : itemDeleted

13. model : 이건 뭐에 어떻게 쓰는건지 모르겠음
- 주석 처리 된거 해석해보면 indexPath에서 모델을 검색하는 방법이라는거 같으니 indexPath 를 이용해서 해당 셀의 data를 시스템상으로 알 수 있게 해주는게 아닐까 함
- 조금 더 검색과 사용 방법에 대해서 공부를 해봐야 할 것 같음

- 여기 부턴 조금 심화과정 같은 느낌
- prefetchDataSource 이거 이상 쓸일이 생기면 여기를 보면 될듯 함
- view 에 보여지기 전 셀에서 처리해야 할 연산이 길어 미리 진행 할 수 있게 해주는 DataSource = 해당 경우가 아니면 잘 사용되지 않는다고 하니 필요로 하게 되거나 해당 내용이 쓰인 다른 코드를 보게 되면 확인을 하는게 좋을 듯 함

14. prefetchDataSource
15. setPrefetchDataSource
16. prefetchRows
17. cancelPrefetchingForRows

18. didUpdateFocusInContextWithAnimationCoordinator

----

### 구현시 주의 사항

- 이렇게 열심히 써봤자 reuseIdentifier 쪽 에러 날텐데 register 꼭해줘야 하는거임
    ```swift
        ~~~.bind(to: tableView.rx.items(...)){ row, element, cell in
            ....
        }.disposed(by: disposeBag)
    ```

- 아래 코드를 등록 무조건 해줘야 함, 높이도 정해줘야 함
    ```swift
        self.setTableView.register(TableViewCell.self, forCellReuseIdentifier: TableViewCell.reuseIdentifier)
        self.setTableView.rowHeight = 각 셀 높이
    ```