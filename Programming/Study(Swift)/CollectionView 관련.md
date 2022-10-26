# CollectionView 관련

### CollectionView 에서 Cell을 클릭할 경우는 ?

- func 에서 didselectItemAt 을 검색해서 나오는 함수를 사용하면 된다.
    - 해당 함수의 사용법은 해당 함수를 검색하게 되었을 때 나오는 입출력 관련 읽어보면 됨 (쉬움)
- 근데 나는 Cell 내부에 있는 값을 가진 상태로 클릭 되는걸 원한다하면 다음의 방법을 적용하면 됡듯함
    - cell 을 건드는 class 부분에 isSelected: 함수를 사용하면 됨
    
    ```swift
    override var isSelected: Bool {
            didSet {
    //            if 조건 {
    //                
    //            } else {
    //                
    //            }
            }
        }
    ```