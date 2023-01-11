# # StoryBoard 로 화면 전환

## 1. Storyboard 자체를 값을 객체화 시킨다.
```objectivec
UIStoryboard *mainSB = [UIStoryboard storyboardWithName:storyBoardName bundle:nil];
```

## 2. 사용할 VC 를 객체화 시킨다.
```objectivec
// .h 
@property (strong, nonatomic) TestRelateDateVC *nextVC;

// .m
self.nextVC = [mainSB instantiateViewControllerWithIdentifier:dateVCName];
```

### 주의: 1,2 는 viewDidLoad 급의 앱의 선언 단게에서나 해야 다른 곳에서 사용이 가능함

## 3. Present 를 할 생각이라면 다음처럼 present 시킴
```objectivec
[self presentViewController:self.dateVC animated:true completion:nil];
```

## 3-1. dismiss 를 할 생각이라면 다음 처럼 dismiss 시킴
```objectivec
[self.presentingViewController dismissViewControllerAnimated:true completion:nil];
```