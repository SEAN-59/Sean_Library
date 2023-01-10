# Stroyboard 관련_Size Classes

- 각 Components 를 선택 후 Show the attributes inspector 창에서 보면 맨 아래 하단에 Installed 가 체크가 되어있는것을 확인 할 수 있는데 해당 부분이 Size Classes 관련 항목을 건들 수 있게 해줌.

- 기존의 체크된 사항은 가로, 세로 모드 상관 없이 해당 오토 레이아웃을 사용한다는 뜻이다.

- 하지만 세로와 가로의 오토레이아웃을 따로 잡아주고 싶으면 해당 installed 를 해제하고 새롭게 만들어준다.

- 각 기기에 대한 옵션은 Apple 의 HIG의 Layout 에서 Platform considerations 파트를 찾다보면 자세하게 잘 나타나 있다.
https://developer.apple.com/design/human-interface-guidelines/foundations/layout/#platform-considerations

- 원하는 파트로 w와 h를 잘 설정해 주었다면 그것에 맞게 오토레이아웃을 구성하면 된다.