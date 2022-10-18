#  TableView 관련

### TableView Cell 관련

- tableView 같은 경우 각 셀을 재활용해서 사용하기 때문에 만약에 새롭게 데이터를 저장하기 위해 tableview를 불러오고 싶은데 나는 새롭게 구성을 한거 같은데 얘는 재활용을 하기 때문에 내부의 내용이 전혀 초기화가 안되는 경우가 발생함
그럴 경우 다른 방법보다는 애초에 셀을 만들때 즉, cellForRowAt 에서 조지면 됨.
- 근데 하단에 새로운 cell 추가와 삭제하는 코드는

```swift
tableView.insertRows(at:[IndexPath(row: 0, section:0)],
										 with: .automatic)

tableView.deleteRows(at:[IndexPath(row: 0, section:0)],
										 with: .automatic)
```