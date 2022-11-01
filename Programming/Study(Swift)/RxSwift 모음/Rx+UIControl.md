# RxCocoa 코드 사용 방법 (UIControl).md
- 독학 버전이라서 잘못된 정보도 많고 아직 다듬어야 할 부분도 엄청 많겠지만 수정해야 될 부분은 하나하나 사용해보면서 고치면 될듯하다.
- 일단 Rx 생각보다 더 어렵고 정보가 있어도 어떻게 내꺼로 만들어야 할 지 잘 몰라서 직접 하나하나 코드를 열어보고 사용해보면서 익숙해지는 수 밖에 없을듯 하다.

### RxCocoa + UIControl
- 두번째로 확인하는 코드인데 rx 로 여러 요소들 동작을 하기 위해서 호출하다보니 여기에 있는 .controlEvent 를 사용하는것을 보게  되었고 그래서 한 번 확인 한다.

<details>
<summary> UIControl+Rx 코드 </summary>
<div markdown="1">

```swift
#if os(iOS) || os(tvOS)

import RxSwift
import UIKit

extension Reactive where Base: UIControl {
    /// Reactive wrapper for target action pattern.
    ///
    /// - parameter controlEvents: Filter for observed event types.
    public func controlEvent(_ controlEvents: UIControl.Event) -> ControlEvent<()> {
        let source: Observable<Void> = Observable.create { [weak control = self.base] observer in
                MainScheduler.ensureRunningOnMainThread()

                guard let control = control else {
                    observer.on(.completed)
                    return Disposables.create()
                }

                let controlTarget = ControlTarget(control: control, controlEvents: controlEvents) { _ in
                    observer.on(.next(()))
                }

                return Disposables.create(with: controlTarget.dispose)
            }
            .take(until: deallocated)

        return ControlEvent(events: source)
    }

    /// Creates a `ControlProperty` that is triggered by target/action pattern value updates.
    ///
    /// - parameter controlEvents: Events that trigger value update sequence elements.
    /// - parameter getter: Property value getter.
    /// - parameter setter: Property value setter.
    public func controlProperty<T>(
        editingEvents: UIControl.Event,
        getter: @escaping (Base) -> T,
        setter: @escaping (Base, T) -> Void
    ) -> ControlProperty<T> {
        let source: Observable<T> = Observable.create { [weak weakControl = base] observer in
                guard let control = weakControl else {
                    observer.on(.completed)
                    return Disposables.create()
                }

                observer.on(.next(getter(control)))

                let controlTarget = ControlTarget(control: control, controlEvents: editingEvents) { _ in
                    if let control = weakControl {
                        observer.on(.next(getter(control)))
                    }
                }
                
                return Disposables.create(with: controlTarget.dispose)
            }
            .take(until: deallocated)

        let bindingObserver = Binder(base, binding: setter)

        return ControlProperty<T>(values: source, valueSink: bindingObserver)
    }

    /// This is a separate method to better communicate to public consumers that
    /// an `editingEvent` needs to fire for control property to be updated.
    internal func controlPropertyWithDefaultEvents<T>(
        editingEvents: UIControl.Event = [.allEditingEvents, .valueChanged],
        getter: @escaping (Base) -> T,
        setter: @escaping (Base, T) -> Void
        ) -> ControlProperty<T> {
        return controlProperty(
            editingEvents: editingEvents,
            getter: getter,
            setter: setter
        )
    }
}

#endif
```

</div>
</details>

----
- 생각보다 별건 없었으나 해당 코드가 UIControl 상속받아 사용한다는거에 집중을 해야 했음
- UIControl은 평상시도 자주 사용하던거라 큰 어려움은 없기에 링크 달아 두겠음

[UIControl 링크](https://developer.apple.com/documentation/uikit/uicontrol)<br>
    - 해당 링크만크으로도 충분히 볼건 많았지만 제대로 봐야 할건 아래다.<br> 
[struct UIControl.Event 링크](https://developer.apple.com/documentation/uikit/uicontrol/event)<br>
[struct UIControl.State 링크](https://developer.apple.com/documentation/uikit/uicontrol/state)<br>

- 그런데 UIControl+Rx 에서 있는 함수는 3가지로 controlEvent, controlProperty, controlPropertyWithDefaultEvents<T> 인데 여기서 사용하는 것들은 다 .Event 가 되겠다.

----
1. controlEvent

    ```swift
    public func controlEvent(_ controlEvents: UIControl.Event) -> ControlEvent<()> {
            let source: Observable<Void> = Observable.create { [weak control = self.base] observer in
                    MainScheduler.ensureRunningOnMainThread()

                    guard let control = control else {
                        observer.on(.completed)
                        return Disposables.create()
                    }

                    let controlTarget = ControlTarget(control: control, controlEvents: controlEvents) { _ in
                        observer.on(.next(()))
                    }

                    return Disposables.create(with: controlTarget.dispose)
                }
                .take(until: deallocated)

            return ControlEvent(events: source)
        }
    ```
    - 딱 보면 알겠지만 .Event 를 입력으로 받아 controlEvent를 반환하게 됨

2. controlProperty

    ```swift
     public func controlProperty<T>(
        editingEvents: UIControl.Event,
        getter: @escaping (Base) -> T,
        setter: @escaping (Base, T) -> Void
    ) -> ControlProperty<T> {
        let source: Observable<T> = Observable.create { [weak weakControl = base] observer in
                guard let control = weakControl else {
                    observer.on(.completed)
                    return Disposables.create()
                }

                observer.on(.next(getter(control)))

                let controlTarget = ControlTarget(control: control, controlEvents: editingEvents) { _ in
                    if let control = weakControl {
                        observer.on(.next(getter(control)))
                    }
                }
                
                return Disposables.create(with: controlTarget.dispose)
            }
            .take(until: deallocated)

        let bindingObserver = Binder(base, binding: setter)

        return ControlProperty<T>(values: source, valueSink: bindingObserver)
    }
    ```

3. controlPropertyWithDefaultEvents

    ```swift
    internal func controlPropertyWithDefaultEvents<T>(
        editingEvents: UIControl.Event = [.allEditingEvents, .valueChanged],
        getter: @escaping (Base) -> T,
        setter: @escaping (Base, T) -> Void
        ) -> ControlProperty<T> {
        return controlProperty(
            editingEvents: editingEvents,
            getter: getter,
            setter: setter
        )
    }
    ```

- 근데 이게 코드를 그닥 볼 필요를 못 느끼겠는게 그냥 .Event 할 때 이거 사용하는 정도로만 보면 될듯 함
- Rx+OOO 이거에서 개인적으로 따로 빠진거 없이 동작이 들어가야 할 때는 이거 쓰는 듯 함
