# Rx / extension Reactive where
- 독학 버전이라서 잘못된 정보도 많고 아직 다듬어야 할 부분도 엄청 많겠지만 수정해야 될 부분은 하나하나 사용해보면서 고치면 될듯하다.
- 일단 Rx 생각보다 더 어렵고 정보가 있어도 어떻게 내꺼로 만들어야 할 지 잘 몰라서 직접 하나하나 코드를 열어보고 사용해보면서 익숙해지는 수 밖에 없을듯 하다.

<details>
<summary> Reactive.swift </summary>
<div markdown="1">

```swift

@dynamicMemberLookup
public struct Reactive<Base> {
    /// Base object to extend.
    public let base: Base

    /// Creates extensions with base object.
    ///
    /// - parameter base: Base object.
    public init(_ base: Base) {
        self.base = base
    }

    /// Automatically synthesized binder for a key path between the reactive
    /// base and one of its properties
    public subscript<Property>(dynamicMember keyPath: ReferenceWritableKeyPath<Base, Property>) -> Binder<Property> where Base: AnyObject {
        Binder(self.base) { base, value in
            base[keyPath: keyPath] = value
        }
    }
}

/// A type that has reactive extensions.
public protocol ReactiveCompatible {
    /// Extended type
    associatedtype ReactiveBase

    /// Reactive extensions.
    static var rx: Reactive<ReactiveBase>.Type { get set }

    /// Reactive extensions.
    var rx: Reactive<ReactiveBase> { get set }
}

extension ReactiveCompatible {
    /// Reactive extensions.
    public static var rx: Reactive<Self>.Type {
        get { Reactive<Self>.self }
        // this enables using Reactive to "mutate" base type
        // swiftlint:disable:next unused_setter_value
        set { }
    }

    /// Reactive extensions.
    public var rx: Reactive<Self> {
        get { Reactive(self) }
        // this enables using Reactive to "mutate" base object
        // swiftlint:disable:next unused_setter_value
        set { }
    }
}

import Foundation

/// Extend NSObject with `rx` proxy.
extension NSObject: ReactiveCompatible { }

```
</div>
</details>
----

## 학습 내용
- ReaciveCompatible 을 채택하면 해당 클래스에서 rx 프로퍼티 접근 가능하고 rx 프로퍼티에서 base 인스턴스 갖고 있으니, 즉 rx.base 접근 가능