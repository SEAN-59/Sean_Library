# Objc Foundation 설명

## 1. NSNumber
### [Numbers, Data, Basic Values정리](Foundation_(Number,Data,BasicValues).md)
- 기본 데이터 형들은 객체가 아니었고 메시지를 보내는 일은 불가능 했다.   
하지만 그럼에도 이러한 형 값을 객체로 다루어야 할 때가 있다.
- 예를 들어 NSArray 는 값을 저장할 수 있는 배열을 생성해주는데 이 값은 객체여야 한다.   
그래서 기본 데이터 형을 바로 저장 할 수 없다.
- 그래서 그 대신에 NSNumber 클래스를 사용해서 이 데이터 형들에게 객체를 생성 할 수 있다.
    ```objectivec
    [NSNumber numberwith~~~: xxx];
    ```
    - 관련 메서드는 공식 문서 참고

## 2. NSString = 문자 스트링 객체를 다룰 수 있다.
### [NSString Method 정리](Foundation_(Strings and Text).md)
- 기본 C 의 문자는 char 형이지만 NSString 의 경우에는 unichar 형으로 이루어져 있음
- %@ 포맷문자를 이용해서 NSLog 에 띄울 수 있는데 이는 이 객체 뿐 아니라 다른 객체 표시도 가능하다.
    ```objectivec
    @"stringstring"
    ```
    - 위의 형식으로 저장하면 된다.

- 그런데 위의 방식으로 저장을 하게 되면 중간에 문자열을 바꾼다던가 대치를 한다던가 하는 행위를 못하게 되는 '수정 불가능한 객체'가 되어 버린다.
- 이렇게 수정을 해야하는 스트링을 다룰 때는 NSMutableString 에서 처리한다.
- 관련 메서드는 공식 문서 참고

## 3. NSArray = 값을 저장할 수 있는 배열 생성
- 배열의 원소는 주로 특정한 하나의 데이터 형으로 되어 있지만 여러 형일 떄도 있다.
- 배열도 수정 가능, 불가능 2가지 배열이 존재 한다.
- arrayWithObjects: 메서드를 사용해 객체 목록을 원소로 갖는 배열을 만들 수 있다.
- 관련 메서드는 공식 문서 참고
- https://developer.apple.com/documentation/foundation/nsarray?language=objc


## 4. NSDictionary
- https://developer.apple.com/documentation/foundation/nsdictionary?language=objc

## 5. NSSet
- https://developer.apple.com/documentation/foundation/nsset?language=objc

## 6. NSFileManager
- https://developer.apple.com/documentation/foundation/nsfilemanager?language=objc

## 7. NSFileHandle
- https://developer.apple.com/documentation/foundation/nsfilehandle?language=objc

## 8. NSData
- https://developer.apple.com/documentation/foundation/nsdata?language=objc



