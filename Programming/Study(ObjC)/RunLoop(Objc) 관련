# RunLoop 관련

```objectivec
NSMutableString *lblto = [[NSMutableString alloc]init];
    NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    
    for (int i = 0; i<10; i++) {
        [lblto appendString:@"a."];
        [runLoop runMode:NSDefaultRunLoopMode beforeDate: [NSDate dateWithTimeIntervalSinceNow:0.3]];
        [self.typingLbl setText:lblto];
    }
```

- 이런식으로 사용하는 방법이 존재 함 타이머 대신 쓰기 좋아보임
- 정확한 사용 방법이나 존재 의의에 대해서는 공부를 조금 더 진행을 해봐야겠지만 일단은 타이머 대신으로 사용하면 좋을 듯 하다