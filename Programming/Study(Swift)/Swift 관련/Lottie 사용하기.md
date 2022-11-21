# Lottie 사용하기

## 사용법 
1. 사용할 코드에 import Lottie 하기
2. JSON 파일 미리 구비해두기
3. AnimationView 를 정의하기
    - ex) let 이름 : LottieAnimationView = .init(name: "JSON 이름")
4. 사용할 AnimationView 를 적당한 위치에서 .play 하면 애니메이션 나옴

## 주의사항
- 단 애니메이션은 1회만 재생 될것

## 기타 명령 (AnimationView 를 AV 로 지칭)
1. loop 설정 
    1. AV.loopMode = .playOnce : 1회만 실행 -> 초기랑 다를게 뭐지?
    2. AV.loopMode = .repeat(5) : 5회 실행후 종료
    3. AV.loopMode = .loop : 무한 반복
    4. AV.loopMode = .repeatBackwards(2) : 정상 재생 후 바로 역재생 을 2회 반복
    5. AV.loopMode = .autoReverse : 4번 동작 무한 반복

2. 속도 설정
    - AV.animationSpeed = 0.5 : 0.5배속

3. Frame 만큼만 재생하기!
    - AV.play(fromFrame: 5, toFram: 30) : 5 프레임부터 30 프레임까지만 동작함
    - AV.animation?.endFrame : 프레임 갯수가 몇개인지 모를때

4. Progress 로 재생하기
    - AV.play(fromProgress: 0.4, toProgress: 0.6) : 40% ~ 60% 사이를 동작함

5. 딱 원하는 위치에 멈추게 하기
    1. AV.currentFrame = 20
    2. AV.currentProgress = 0.2