# FireBase_auth

## 사용법 
Reference : [Authentication 공식 문서](https://developers.google.com/identity/sign-in/ios/start)

<Google 로그인>
1. Firebase 에서 auth 관련 라이브러리 설치
2. Authentication 관련 패키지 설치 링크 : https://github.com/google/GoogleSignIn-iOS
    - swiftUI 를 사용한다면 GoogleSignInSwift 도 추가 할 것
3. OAuth 서버 클라 ID 가져오기 -> [Cloud Console](https://console.cloud.google.com/apis/credentials?project=_) 에서 사용할 프로젝트 연결
4. 1번에서 Firebase 등록시 프로젝트에 추가한 GoogleService-Info.plist 에 보면 CLIENT_ID 존재 하는데 해당 값을 가지고 스키마를 추가 할것
    1. 프로젝트 > TARGETS > INFO > URL Types > +
    2. 다른곳 그냥 다 놔두고 URL Schemes 란에 CLIENT_ID 입력하기
5. AppDelegate에 코드 추가
    ```swift
    import GoogleSignIn // <- 추가
    
    ...

    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        var handled: Bool
        handled = GIDSignIn.sharedInstance.handle(url)
        if handled {
            return true
            }
            
        // Handle other custom URL types.
        // If not handled by this app, return false.
        return false
    } // <- 해당 함수 전체 추가
    ```

### 기능
1. 사용자 로그인 상태 복원 시도
     - 앱 시작시 이미 구글을 통해 로그인한 사용자의 로그인 상태를 복원해 로그아웃을 하지 않는 한 앱을 열 때마다 매번 로그인 하지 않아도 됨
     - 해당 코드를 AppDelegate에서 사용
        ```swift
            func application(_ application: UIApplication,didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
                GIDSignIn.sharedInstance.restorePreviousSignIn { user, error in
                    if error != nil || user == nil {
                        // Show the app's signed-out state.
                    } else {
                        // Show the app's signed-in state.
                    }
                }
                return true
            }
        ```

2. 로그인 버튼 추가
    - 호출 하고자 하는 위치에 해당 코드를 추가
        ```swift
            func signIn(){
                guard let client = FirebaseApp.app()?.options.clientID else { return }
                let signInConfig = GIDConfiguration(clientID: clientID)

                GIDSignIn.sharedInstance.signIn(with: signInConfig, presenting: self) { user, error in
                    guard error == nil else { return }
                    // If sign in succeeded, display the app's main content View.
                }
            }
        ```

3. 로그아웃 버튼 추가
    - 호출 하고자 하는 위치에 해당 코드 추가
        ```swift
            func singOut() {
                GIDSignIn.sharedInstance.signOut()
            }
        
        ```

4. 프로필 정보 가져오기
    - signIn 함수에서 user.profile?. 키워드를 사용해서 정보를 가져올 수 있음
    
5. 백엔드 서버 인증
    - 로그인한 사용자의 ID를 백엔드 서버에 전달하려면 사용자의 프로필 정보 (이메일 주소 포함) 또는 GIDGoogleUser의 userId 필드를 사용하지 마세요. 대신 서버에서 안전하게 검증할 수 있는 ID 토큰을 전송합니다.
    - signIn 에서 다음과 같은 코드를 활용함
        ```swift
            user.authentication.do { authentication, error in
                guard error == nil else { return }
                guard let authentication = authentication else { return }

                let idToken = authentication.idToken
                // Send ID token to backend (example below).
            }
        ```