# TableView(Objc) 관련

## Delegate & DataSource 설정
1. VC 에 프로토콜 잡아주는게 먼저 해줘야 할 일
(스토리보드에서 직접 하나 하나 해주는 방법도 있겠지만 그보다는 직접 코드로 잡아주는게 좋다 생각함)
    ```objectivec
    @interface ViewController: UIViewController <UITableViewDelegate, UiTableViewDataSource>
    @end
    ```

2. VC 의 viewDidLoad 또는 원하는 위치에서 set을 해준다.
    ```objectivec
    [self.tableView setDelegate: self];
    [self.tableView setDataSource: self];
    ```

3. 그리고 사용하고자 하는 메서드들을 잘 사용해주면 된다.

---
## UITableViewDelegte Method 종류
### [UITableViewDelegate Document Link](https://developer.apple.com/documentation/uikit/uitableviewdelegate?language=objc)
### 1. heightForRowAtIndexPath: 테이블 항목 높이 설정
### 2. willSelectRowAtIndexPath: 선택될 테이블 항목 처리
### 3. didSelectRowAtIndexPath: 선택한 테이블 항목 처리
### 4. editStyleForRowAtIndextPath: 행 편집 스타일 지정

---
## UITableViewDataSource Method 종류
### [UITableViewDataSource Document Link](https://developer.apple.com/documentation/uikit/uitableviewdatasource?language=objc)
### 1. numberOfRowsinSection: 각 섹션당 항목의 개수를 방출
### 2. cellForRowAtIndexPath: 인덱스에 해당하는 항목의 셀 정보 방츨
### 3. numberOfSectionsInTableView: 현재 테이블 뷰의 섹션 수 방출
### 4. titleForHeaderInSection: 현재 테이블 뷰의 제목 방출


---
## xib 파일을 활용한 tableView 만드는 방법

### 1. 화면 배치 및 tableView와 cell 에 Identity 값 제대로 넣어두기
### 2. 사용할 tableView를 outlet 변수 지정과 프로토콜(datasource, delegate) 등록하기
```objectivec
    [self.tableView setDelegate:self];
    [self.tableView setDataSource:self];
```
### 3. nib파일 객체화 시켜서 만들어 두고 tableView에 register 시키기
```objectivec
    UINib *nib = [UINib nibWithNibName:cellIdentity
                        bundle:nil];
    [_tableView registerNib:nib                            
                forCellReuseIdentifier:cellIdentity];
```

### 4. 프로토콜 받아들이면서 작성해야 할 required 메소드 작성
- datasource 부분에 2개 존재 
    - numberofRowInSection:
    - cellForRowAtIndexPath:

### 5. cellforRowAtIndexPath: 부분에 cell을 return 할 수 있게 함
1. 일단 xib 파일의 .h 파일을 해당 tableField 가 있는곳에서 import를 시킴
- import 를 시켜 놔야 cell의 객체를 만들 클래스로 사용을 할 수 있다.
2. 그 후에는 재사용가능한 현재 사용하는 tableView 의 cell 로서 만들어서 방출함
    ```objectivec
    MyTbCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentity];
    ```

### 6. 만약 cell 에 정의된 메서드를 사용하고 싶다면 5-2의 cell 부분에 클래스에서 내가 만들어둔 xib 파일의 클래스를 꼭 지정을 해둬야 함 -> 이거 실수해서 UI~~ 의 대중적인거 지정하면 백날해도 메서드 동작 안함