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
