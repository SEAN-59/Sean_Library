# FireBase

## 사용법 
Reference: [Apple 플랫폼에서 Firebase 인증 시작하기](https://firebase.google.com/docs/auth/ios/start)   

1. 패키지 추가 링크 : https://github.com/firebase/firebase-ios-sdk
2. 사용하는 라이브러리 골라서 설치 하면 됨
3. AppDelegate 에 코드 추가
    ```swift
    import FirebaseCore // <- 추가
    
    ...

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        FirebaseApp.configure() // <- 추가
        // Override point for customization after application launch.
        return true
    }
    ```
