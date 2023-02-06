# Timer(Objc) 관련

## scheduledTimerWithTimeInterval 메소드 파라미터
1. second : 반복 실행될 타이머 지정 함수 사이의 간격. (단위: 초)
2. target : 다음 Selector 함수가 위치할 객체. (self 면 현재 클래스 의미)
3. selector : 타이머에서 반복 처리할 함수 이름
4. userInfo : 타이머에서 참조할 사용자 정보. 일반적으로 nil 지정
5. repeats : 타이머 반복 처리 결정.