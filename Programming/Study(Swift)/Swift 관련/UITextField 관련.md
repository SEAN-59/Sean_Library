# UITextField 관련

### Padding
- Padding 은 textField 에서 처음 텍스트가 시작되는 위치가 TextField에 딱붙어서 동작이 되는데 이를 조금이라도 띄우는걸 말한다.
- 따로 정해진 코드가 없기에 extension으로 UITextField에 직접 함수를 만들어서 사용한다.
- 인자로서 textField 의 .alignment 와 얼마를 띄울지를 CGFloat 으로 입력을 준다.
```swift
extension UITextField {
    func addPadding(_ textAlignment: NSTextAlignment, size: CGFloat) {
        if textAlignment == .left {
            let paddingView = UIView(frame: CGRect(x: 0, y: 0, width: size, height: self.frame.size.height))
            self.leftView = paddingView
            self.leftViewMode = .always
        } else if textAlignment == .right {
            let paddingView = UIView(frame: CGRect(x: 0, y: 0, width: size, height: self.frame.size.height))
            self.rightView = paddingView
            self.rightViewMode = .always
        }
    }
}
```