# UIViewController LifeCycle

- LifeCycle
1. init()
2. loadView()
3. viewDidLoad()
4. viewWillAppear()
5. viewDidAppear()
6. viewWillDisappear()
7. viewDidDisappear() → 4
8. viewDidUnload → 2

- 3. viewDidLoad : 뷰가 로드되었다.
- 4. viewWillAppear: 뷰가 나타날것이다.
- 5. viewDidAppear: 뷰가 나타났다.
- 6. viewWillDisappear: 뷰가 사라질것이다.
- 7. viewDidDisappear: 뷰가 사라졌다.