# RxCocoa 문제 발생 후 해결 방법 (UITableView).md
- 독학 버전이라서 잘못된 정보도 많고 아직 다듬어야 할 부분도 엄청 많겠지만 수정해야 될 부분은 하나하나 사용해보면서 고치면 될듯하다.
- 일단 Rx 생각보다 더 어렵고 정보가 있어도 어떻게 내꺼로 만들어야 할 지 잘 몰라서 직접 하나하나 코드를 열어보고 사용해보면서 익숙해지는 수 밖에 없을듯 하다.
----

Q1: TableView 를 잘 만들었고 내부에 cell 도 잘 넣었는데 왜 cell 내부에 버튼이나 여러 Components 가 작동을 하지 않을까?

A1: SnapKit을 사용하여 cell에 .addSubView 를 사용하고 있는데 그렇게 할 경우에 객체가 contentView 에 올라가는게 아니라 그 밑에 즉 cell과 contentView 사이에 들어가기에 외부에서 상호작용이 불가능하게 되어버리는 불상사가 발생한다.   
<참고 링크: https://minios.tistory.com/35>

----
