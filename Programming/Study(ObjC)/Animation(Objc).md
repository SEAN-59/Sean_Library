# Animation (Objc) 관련

## animateWithDuration 메소드 파라미터
1. duration : 애니메이션 실행 시간 (단위: 초)
2. delay : 애니메이션이 실행되기 전 대기 시간 (단위: 초)
3. dampingRatio: 진동 애니메이션 구현시 정지 상태로 도달하기 위한 제동 비율
4. velocity: 초기 진동 속도
5. options: 애니메이션을 실행 속도 처리에 대한 옵션
6. animations: 애니메이션을 처리하는 블록 객체. 애니메이션의 마지막 위치를 지정
7. completion: 애니메이션 처리가 끝난 뒤에 실행되는 블록 객체

### 5번 파라미터에 넣을 수 있는 값
1. UIViewAnimationOptionCurveEaseInOut
    - 처음에는 천천히 > 중간에 빨라지고 > 마지막에 느려짐
2. UIViewAnimationOptionCurveEaseIn
    - 처음에는 천천히 시작해서 점점 빨라짐
3. UIViewAnimationOptionCurveEaseOut
    - 처음에는 빠르게 시작하다 점점 느려짐
4. UIViewAnimationOptionCurveLinear
    - 처음부터 끝까지 동일한 속도

