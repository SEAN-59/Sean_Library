# Firebase_Auth_Email (objc)

## 사전 준비
```objectivec
@import FirebaseCore;
@import FirebaseAuth;
```
- 해당 파일을 전부 import 해야 Auth 의 기능을 사용 할 수 있다.<br><br><br>
---
## 로그인 하기
```objectivec
[[FIRAuth auth] signInWithEmail: email
                           password: password
                         completion:^(FIRAuthDataResult * _Nullable authResult, NSError * _Nullable error) {
        if (error != nil) {
            // ...
        } else {
            // ...
        }
    }];
```
- error 코드는 error.userInfo 의 항목으로 Dictionary 타입으로 저장되어있다.
    ```objectivec
    [[errorDict objectForKey:@"FIRAuthErrorUserInfoNameKey"]
    ```
    - 해당 명령어로 Dictionary 안에 들어있는 오류 키 값을 받아 올 수 있다.
    - ERROR_USER_NOT_FOUND : 가입된 유저가 아닐때 발생하는 에러
<br><br><br>
---
## 계정 생성
```objectivec
[[FIRAuth auth] createUserWithEmail:email
                               password:password
                             completion:^(FIRAuthDataResult * _Nullable authResult, NSError * _Nullable error) {
                                if (error != nil) {
                                    // ...
                                } else {
                                    // ...
                                }
                             }];
```
- 계정을 생성하게 될 경우 자동으로 로그인까지 진행이 되어진다.
    ```objectivec
        FIRUser *user = [FIRAuth auth].currentUser;
    ```
    - 해당 코드로 현재 접속되어있는 유저의 정보를 받아 올 수 있다.
<br><br><br>
---
## 로그아웃 하기
```objectivec
NSError *signOutError;
BOOL staus = [[FIRAuth auth] signOut: &signOutError];

if (!staus) {
    NSLog(@"ERROR SIGNING OUT: %@", signOutError);
    return;
} else {
    ...
}
```
- signOut 시에 파라미터로 NSError 를 받으므로 넣어준다.
- 출력으로 BOOL 값이 나오기에 해당 값으로 로그아웃이 잘 되었는지 판단 할 수 있다.
<br><br><br>
---
## ERROR 관련
- [Error Type Link](https://firebase.google.com/docs/auth/ios/errors?hl=ko)