# Rx로 UIButton Refactoring.md
- 독학 버전이라서 잘못된 정보도 많고 아직 다듬어야 할 부분도 엄청 많겠지만 수정해야 될 부분은 하나하나 사용해보면서 고치면 될듯하다.
- 일단 Rx 생각보다 더 어렵고 정보가 있어도 어떻게 내꺼로 만들어야 할 지 잘 몰라서 직접 하나하나 코드를 열어보고 사용해보면서 익숙해지는 수 밖에 없을듯 하다.

---

## 조건
1. 버튼을 누르게 되면 크기가 변동이 있는 애니메이션 기능의 추가

---
## 구현
```swift

import UIKit

class SButton: UIButton {
    /// 버튼의 확대와 축소를 담당하는 변수
    ///
    /// 0 보다 크면 확대 하는 애니메이션
    ///
    /// 0 보다 작으면 축소하는 애니메이션
    public var sizeChangeBtn: (Bool,Int) = (false, 0)
    
    override var isHighlighted: Bool {
        didSet {
            if sizeChangeBtn.0 {
                self.sizeChangeAnimation()
            }
        }
    }
    private func sizeChangeAnimation() {
        if self.sizeChangeBtn.1 > 0 {
            
        }
        let animation: SizeChange = {
            if self.sizeChangeBtn.1 > 0 {
                return SizeChange.sizeUpTouchDown
            } else if self.sizeChangeBtn.1 < 0 {
                return SizeChange.sizeDownTouchDown
            } else {
                fatalError()
            }
        }()
        
        let animationElement = self.isHighlighted ? animation.element : SizeChange.touchUp.element
        UIView.animate(withDuration: animationElement.duration,
                       delay: animationElement.delay,
                       options: animationElement.options,
                       animations: {
            self.transform = animationElement.scale
            self.alpha = animationElement.alpha
        })
    }
}

private enum SizeChange {
    typealias Element = (
        duration: TimeInterval,
        delay: TimeInterval,
        options: UIView.AnimationOptions,
        scale: CGAffineTransform,
        alpha: CGFloat
    )
    
    case sizeUpTouchDown
    case sizeDownTouchDown
    case touchUp
    
    var element: Element {
        switch self {
        case .sizeUpTouchDown:
            return Element(duration: 0,
                           delay: 0,
                           options: .curveLinear,
                           scale: .init(scaleX: 1.1, y: 1.1),
                           alpha: 0.8)
            
        case .sizeDownTouchDown:
            return Element(duration: 0,
                           delay: 0,
                           options: .curveLinear,
                           scale: .init(scaleX: 0.7, y: 0.7),
                           alpha: 0.8)
            
        case .touchUp:
            return Element(duration: 0,
                           delay: 0,
                           options: .curveLinear,
                           scale: .identity,
                           alpha: 1)
        }
    }
}
```