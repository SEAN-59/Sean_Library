#Objective-C 비동기 처리

## GCD (Grand Central Dispatch) ?
- 시스템이 제공하는 Bachground queue 를 가져와 추가해주고 종료가 되면 결과에 따라 분기 처리

```objectivec
// dispatch_async 함수는 내부블럭의 코드 실행에 영향을 받지 않고 바로 실행이 끝남
dispatch_asnyc(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0), ^ {
    // 작업이 오래 걸리는 API를 백그라운드 쓰레드에서 실행한다.

    dispatch_asnyc(dispatch_get_main_queue(), ^ {
        // 이 블럭은 메인쓰레드에서 실행된다.
    })
})
```obcd