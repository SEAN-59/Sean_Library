# Objective-C 정리
#### 교재: 프로그래밍 오브젝티브-C 2.0 Programming in Objective-C 2.0 5/E
---
### 1. 값 표시 하기
```objectivec
int number;
```
- 세미콜론 (;) 표기를 해야 함
---
### 2. 클래스, 객체, 메서드
1. @interface
- 클래스 선언부의 역할을 담당함
    ```objectivec
    @interface ClassName: ParentsClassName
    @end
    ```
- 내부에 클래스의 메서드를 선언'만' 해줌
    ```objectivec
    - (void) setNum: (int) n;
    ```
    - 메서드타입 (반환타입) 메서드이름: (인수타입) 인수이름;

    - 메서드 타입은 - (인스턴스 메서드), + (클래스 메서드) 으로 구성   
        -의 인스턴스 메서드는 클래스의 특정 인스턴스 작업을 수행   
        +의 클래스 메서드는 새로운 인스턴스 만드는것과 같이 클래스 자체 작업 수행
    

2. @implementation
- 클래스 구현부의 역할을 담당함
    ```objectivec
    @implementation ClassName
    @end
    ```

3. Program Part
- 실질적인 코드가 흘러갈 부분 : main 코드
---

### 3. 데이터 형과 표현식
1. 데이터 타입
- int(%i), double(%f), float(%f), char($c), id(%p)
2. 형 변환 연산자
    ```objectivec
    (데이터 타입) 변수
    ```
---
### 4. 프로그램 반복문
1. for 문
    ```objectivec
    for(int i = 0; i > 100; i++){
        ...;
    }
    ```
2. while 문
    ```objectivec
    while(i > 100){
        ...;
    }
    ```
3. do 문 (= do-while 문)
    ```objectivec
    do {
        ...;
    } while(i > 100);
    ```
4. break : 해당 명령어 만나면 해당 명령어 감싸고 있던 반복문 종료 후 빠져 나옴
5. continue : 해당 명열어는 반복문 종료 않고 만남과 동시에 그 이후의 모든 명령문 건너띄고 조건 검사 수식으로 돌아감
---
### 5. 의사결정하기
1. if 문
    ```objectivec
    if ( i > 100) {
        ...;
    }
    ```
2. if-else 문
    ```objectivec
    if ( i > 100 ) {
        ...;
    } else {
        ...;
    }
    ```
3. else-if 문
    ```objectivec
    if ( i > 100 ) {
        ...;
    } else {
        if ( j > 100 ) {
            ...;
        } else {
            ...;
        }
    }
    ```
    ```objectivec
    if ( i > 100 ) {
        ...;
    } else if ( j > 100 ) {
        ...;
    } else {
        ...;
    }
    ```

4. switch 문
    ```objectivec
    switch (조건) {
        case value1:
            ...;
            break;
        case value2:
            ...;
            break;
        default:
            ...;
            break;
    }
    ```
    - switch 문에서 break 를 사용해주지 않으면 그 아래의 case 도 실행이 되어 버림
---
### 6. 클래스에 대해서
- @interface (.h) 와 @implementation (.m) 파일로 나누어서 작성
- 프로퍼티에 좀 더 쉽게 접근하기 위해서 .(점) 연산자 사용 가능
- 메서드에 여러 인수 넘기기도 가능
    ```objectivec
    - (void) setX: (int)x andSetY: (int)y;
    ```

- 인수 이름 없는 메서드도 가능함
    ```objectivec
    - (void) set: (int)x: (int)y;
    ```

- 메서드의 입출력 타입으로 클래스도 가능함 : 객체를 생성하고 반환 할 수 있음
    ```objectivec
    - (class *) method: (class *) c;
    ```

- 메서드 내부에 선언한 지역변수의 경우에는 메서드 사용시마다 초기화 됨   
정적 변수(static)로 사용하면 그 값은 유지됨
    - static 변수를 메서드 선언 밖에 만들면 다른 메서드에서도 접근 가능
    
- self 키워드 : 메서드 내에서는 인스턴스 변수를 이름으로 직접 부를 수 있다.
    ```objectivec
    [self method];
    ```

- 인스턴스 변수의 getter, setter 를 만들어서 사용을 하는데 (캡슐화) 그걸 쉽게 선언하는 방법
    ```objectivec
    @property 변수이름
    ```
- 인스턴스 변수의 getter, setter 부분 쉽게 구현 방법
    ```objectivec
    @synthesize 변수이름
    ```

---
### 7. 상속
- 부모 클래스를 갖지 않는 클래스가 존재 = 루트 클래스
    - 만들 수 는 있는데 만들지 않고 기존의 것을 활용함
- 보통 상속을 사용해 클래스를 확장함
    - 서브는 부모가 지닌 퍼블릭 인스턴스 변수를 모두 상속받기에 사용이 가능함
    - 서브는 상속받고 자신만의 특징을 위해 다른 메서드들을 추가할 수 있음
- @class 지시어
    - .h 파일 전체를 처리할 필요가 없어지기 때문에 좀 더 효율적일 수 있으나 해당 클래스의 메서드를 사용해야 한다면 import 꼭 해야 한다.
- 메서드 재정의하기
    - 상속으로는 메서드를 제거/삭제가 불가능하다.
    - 하지만 재정의는 가능하다

    ```objectivec
    - (void) initX{
        x = 100;
    }
    ```
    - 위와 같은 메서드를 부모에서 갖고 있을 때 서브는 해당 메서드를 다음과 같이 재정의 할 수 도 있다.
    ```objectivec
    - (void) initX{
        x = 999;
    }
    ```
    - 재정의를 하기 위해서 새 메서드는 <반환형, 인수개수, 데이터형>이 재정의하는 메서드와 같아야 한다.

- 추상클래스
    - 다른 사람이 좀 더 쉽게 서브를 만들 수 있도록 돕는 용도로만 사용되는 클래스
    - 인스턴스 변수가 선언되어 있기는 하지만 그 클래스에서 인스턴스를 바로 만들어서 사용해서는 안된다.

---
### 8. 다형성, 동적 타이핑, 동적 바인딩
- 다형성 : 다른 클래스의 객체들이 동일한 메서드 이름을 사용 가능
- 동적 타이핑 : 객체가 속한 클래스를 알아내는 단계를 프로그램 실행될 때로 미룰 수 있음
- 동적 바인딩 : 객체에 호출되는 실제 메서드를 알아내는 시기를 프로그램 실행 중으로 미룰 수 있음
- @try 사용해 예외 처리
    ```objectivec
    @try {
        ...;
    }
    @catch(NSException *exception) {
        ...;
    }
    ```

    - 예외를 날리면 실행이 종료되지 않고 @catch 블록으로 넘어가 블록 내부에서 에외를 처리한다.
    - @throw 지시어를 사용해서 직접 만든 예외를 던질 수도 있음
    - finally 블록을 사용하면 예외를 던지든 던지지 않든 실행할 코드를 추가 할 수 있다.
    - @catch 블록을 여러 개 연속 사용하여 각각 다른 종류의 예외처리도 가능함

---
### 9. 변수와 데이터 타입
- 객체 초기화
    - 초기화 메서드는 메모리 상 객체를 바꾸거나 이동시킬 권한이 있기에 반환 값을 self에 넣어줌
    - 보통 인스턴스 변수를 생성하고 초기화 작업을 수행
    - 클래스가 초기화 메서드를 하나 이상 갖는 다면 그 중 하나는 '지정된 초기화 메서드'여야 하고 해당 클래스의 모든 메서드는 해당 초기화 메서드를 사용해야 한다.
- 변수의 범위 조정
    - @protected    : 인터페이스 부분에서 정의된 변수의 기본 값, 해당 클래스와 서브에서 바로 접근 가능
    - @private      : 구현붸서 정의된 인스턴스 변수의 기본 값, 해당 클래스에서만 접근 가능, 서브 클래스는 접근 불가
    - @public       : 인스턴스 변수가 정의된 클래스, 그 밖의 클래스, 모듈에 정의된 메서드는 다 접근 가능
    - @package
- 전역 변수
    - 보통 변수의 첫글자를 g 로 시작함
    ```objectivec
    int gNumber;
    ```
- 열거형 (enum)
    - 컴파일러는 열거형의 목록 맨 처음에 0을 할당하고 그 다음에는 1을 할당하는 방식으로 값을 넣는다.
    - 중간에 다른 값이 들어가게 될 경우 해당 부분부터 1씩 증가한다.
    ```objectivec
    enum flag { false, true };
    flag = endOfData;
    endOfdata = false;
    ```
    ```objectivec
    enum flag { false, true } endOfData = fasle;
    ```
    - 둘 다 사용 가능한 방법이다.

- typedef 명령문
    ```objectivec
    typedef int Counter;
    ```
    - int 형과 동일한 Counter 이라는 이름의 변수 이름을 사용 할 수 있음

---
### 10. 카테고리와 프로토콜
- 카테고리
    - 클래스 정의를 다루다 새 메서드 추가하고 싶을 때 사용
    - 서브 클래스를 만들어서 사용을 해도 되지만 조금 더 쉬운 방법
    ```objectivec
    @interface Class (categoryName)
        ...;
    @end

    @implementation Class (categoryName)
        ...;
    @end
    ```
    - 선언과 구현의 방법은 클래스와 동일하며 카테고리로 만든 소스는 Clss+categoryName.h 로 따로 저장해서 import 하는 방식으로 해도 되고 직접 구현 부에 카테고리 선언과 구현을 작성해도 된다.
    - 클래스에서 만든 인스턴스 변수는 기본 클래스에서는 그냥 이름만 불러도 사용이 가능했는데 카테고리의 경우네는 self 로 불러와야 사용이 가능하다.

    - 클래스 확장
        - 카테고리 사이를 아무 이름 없이 비워두고도 사용이 가능하다.
        - 클래스에 추가 인스턴스 변수를 정의하여 확장할 수 있다.
        - 이름 없는 카테고리로 선언된 메서드는 분리된 구현 부분이 아니라 클래스의 메인 구현 부분에 구현된다.
            - 확장시에 초기의 .h 에 선언한 부분을 .m 부분으로 옮기고 구현도 .m 에서 해준다.

    - 카테고리는 클래스의 메서드를 재정의 할 수 있는데 하게 되면 원래의 메서드에는 접근이 불가능 해지기에 좋은 방법은 아님 그럴바에는 그냥 서브 클래스 만드는것이 올바른 선택

- 프로토콜과 델리게이션
1. 프로토콜
    - 프로토콜 이름 앞에 @protocol 지시어를 붙이면 된다.
    ```objectivec
    @protocol NSCopying
        ...;
    @end
    ```
    - 해당 프로토콜을 받아들이려면 인터페이스에서 프로토콜의 이름을 <> 로 감싸주면 된다.
    ```objectivec
    @interface Class: parentsClass <ProtocolName>
    @end
    ```
    - 여러개의 프로토콜을 받을 때에는 <> 내부에 , 로 구분을 지어서 프로토콜 이름을 나열해준다.
    - 해당 프로토콜을 받아들이면 필수 구현 메서드를 구현해야 한다는 점을 알릴 수 있다.
        - @optional 지시어를 사용하면 이 지시어 다음에 위치하는 메서드들은 선택 사항이 되며 프로토콜을 구현할 때 이 메서드를 만들지 않아도 따를 수 있다는 뜻이다.
        - @required 지시어를 다시 사용하면 필수 메서드 목록을 다시 작성할 수 있다.

2. 델리게이션
    - 프로토콜 = 두 클래스간의 인터페이스
        - 프로토콜을 정의하는 클래스는 정의된 처리를 구현하는 클래스에게 위임하는 것
        - 특정 이벤트의 응답으로 취하는 특별한 동작이나 특정 속성을 정의하는 등의 일을 델리게이트 클래스가 처리시 클래스를 더 일반적으로 정의할 수 있다.
    - 클래스가 어떤 액션 취할지 모르기에 책임을 위임하는 것

3. 비공식 프로토콜
    - 비공식 프로토콜 = 메서드 나열은 하지만 구현 않는 카테고리
        - 모든 객체는 동일한 루트에서 상속 받음 = 보통은 루트 클래스에 정의 됨
        ```objectivec
        @interface Class (protocol)
            ...;
        @end
        ```
    - 비공식 프로토콜을 선언하는 클래스는 프로토콜 메서드를 직접 구현 않고 필요에 따라 구현할 메서드들을 선택해 인터페이스 부분에서 다시 선언하고 메서드를 구현한다.

4. 복합 객체
    - 클래스의 정의를 확장하는 방법에는 복합 객체 방법이 존재한다.
    - 다른 클래스에서 객체를 하나 이상 받아 와서 클래스를 정의하는 것, 이 새 클래스의 객체는 다른 객체들로 구성되어 있다.
    - 서브 클래스 만드는 대신 확장하고 싶은 클래스의 객체 하나를 인스턴스 변수로 포함하는 새 클래스를 정의하면 된다.
    ```objectivec
    @interface ClassB: parentsClass
    {
        ClassA *aClass;
    }
    -(void) result;
    @end
    ```

---
### 11. 전처리기
1. #define 명령문
    - 프로그램 상수에 심벌명을 부여
        ```objectivec
        #define TRUE 1
        ```
    - ; 을 붙이지 않는다.
    - 정의한 값 자체를 다른 difine 에서 참조 가능하다
        ```objectivec
        #define PI 3.14
        #define TWO_PI PI * 2
        ```
    - 인수를 받을 수 있게 할 수 도 있다.
        ```objectivec
        #define IS_TOGGLE(t) t > 0 ? 1 : 0  
        ```

2. #import 명령문
    - 다른 파일에 있는 정의를 해당 명령문을 사용하여 프로그램에 포함 시킨다.
    - ""  처리는 전처리기에게 지정한 파일을 하나 혹은 그 이상인 파일 디렉터리에서 찾으라는 지시
    - <> 로 감싸는 방법도 가능 = 검색할 디렉터리 지정 가능

3. 조건 컴파일
    - #ifdef, #endif, #else
        ```objectivec
        #ifdef IPAD
        # deifne kImageFile @"barnHD.png"
        #else
        # define kImageFile @"barn.png"
        #endif
        ```
        - 전처리기 명령문을 시작하는 # 다음에는 빈칸을 얼마든지 넣어도 된다.
        - ifdef 에서 지시한 심벌이 이미 정의되어 있다면 #ifdef, #endif, #else 가 나올 때까지만 코드를 처리하고, 그외의 경우는 무시한다.

    - #ifndef 
        - ifdef 와 유사하게 사용되는데 지정한 심벌이 정의되지 않았을때만 따라 나오는 코드들을 처리한다.

    - #if, #elif
        - if 문은 상수 표현식이 0이 아닌지를 테스트 한다.
        - 0 이 아니라면 else, elif, endif 가 나올때까지 코드 처리하고 아니면 무시한다.
        - define (name) 을 if 와 같이 사용한다면 ifdef 와 동일한 기능을한다.
            ```objectivec
            #if define (name)
            # ...
            #endif
            ```
    
    - #undef 명령문
        - 정의할 이름을 취소할 때 사용
            ```objectivec
            #undef IPAD
            ```

---
### 12. 하부 C 언어 기능
1. 배열
    ```objectivec
    int numbers[5] = {1,2,3,4,5};
    ```
    - 배열에서 한 번 크기가 정해지면 안바뀜

2. 함수
    ```objectivec
    반환타입 함수이름 (인수목록) {
        ...;
    }
    ``` 

3. 블록
    ```objectivec
    ^(인수목록) {
        ...;
    }
    ```
    ```objectivec
    void (^print) (char) = ^(char str[]) {
        ...;
    };

    print("Test");
    ```
    - 이런 식으로 블록을 변수에 대입해서 사용할 수 있다.

4. 구조체: struct
    ```objectivec
    struct date {
        int month;
        int day;
        int year;
    };
    ```
    ```objectivec
    struct date today, purchaseDate;
    today.month = 1;
    ```
    ```objectivec
    struct date today = { 1,9,2023 };
    struct date today = { .month = 1, .day = 9, .year = 2023};
    ```
    - 선언과 초기화 방법은 다음과 같고 당연하게도 정의와 동시에 선언 그리고 대입도 가능함
    ```objectivec
    struct date {
        int month;
        int day;
        int year;
    } todayDate = { 1, 9, 2023 };
    ```
    - 구조체의 이름 없이도 선언해서 사용 가능한데 후에 다시 같은 구조체를 사용하기 위해서는 다시 정의하는 수 밖에 없으니 이름으로 사용.

5. 포인터
- &연산자: 주소 연산자
- *연산자: 간접 참조 연산자
    ```objectivec
    int number = 10;
    int *numPt;
    numPt = &number;
    
    x = *numPt;
    ```
    - number 라는 int 형 변수를 선언했고 numPt라는 변수가 int의 포인터형 임을 알렸다.   
    즉, numPt는 하나 이상의 int 변수에 간접적으로 접근하는데 사용한다는 의미   
    그리고 numpt 와 number 사이에 &로 간접 참조를 생성.
    - numPt는 number 의 값인 10이 아니라 number 의 포인터가 들어간다.
    - numPt를 통해 간접적으로 참조한 값을 사용하려면 * 를 사용한다.

    결론
    - 일반변수 = &포인터변수 : 포인터 변수에 주소 들감
    - 일반변수2 = *포인터변수 : 포인터변수가 참조중인 변수1의 값을 사용 가능

- 구조체를 가리키는 포인터도 지정 할 수 있음
    - 위에는 int의 포인터형을 사용했지만 구조체를 정의 후 같은 방식으로 진행하면 된다.
    ```objectivec
    struct date {
        int month;
        int day;
        int year;
    } today = {1,8,2022};

    date *datePt;
    date = &today;

    (*date).day = 9;

    date->day = 2023;
    ```
    - 구조체 포인터 연산자를 사용하지 않으면 ()감싸고 내부에 *도 넣어주면서 사용해야 하지만 구조체 포인터 연산자를 사용하게 되면 조금 더 깔끔해진것을 볼 수 있다.

- 배열
    - 배열도 포인터를 정의해 배열에 든 값에 접근하는 용도로 사용할 수 있는데 배열의 포인터형으로 만들지는 않는다.   
    대신 포인터를 배열에 포함된 원소의 형태의 포인터로 지정한다.
        - int 배열 이 있으면 int 의 포인터 형으로 만든다는 소리다.
    - 만약, 어느 객체의 배열을 포인터로 참조하려면 그 배열의 원소인 객체의 포인터 형으로 만들어야 한다.
        ```objectivec
        Fracion fracts[];
        Fraction **fractsPtr;
        // 뭐..대충 이런식으로?
        ```
    - 첨자 연산자([])를 사용하지 않는 배열 이름은, 배열의 첫 원소를 가리키는 포인터로 여기기에 다음과 같은 방법으로 배열의 첫 원소를 가리킬 수 있다.
    ```objectivec
    int number[];
    int *numPt;
    numPt = number;

    numPt = &number[0];
    ```
    - 이러한 방식을 사용하기에 배열 원소를 연속적으로 훑고 싶을 때 그 수치만큼 더해주면 된다.
    ```objectivec
    *(numPt + 3) = 27;
    ```

- 함수의 포인터 
    - 포인터 변수가 함수를 가리키고 있음은 물론 반환 값의 타입, 인수 타입, 인수 개수를 알아야 한다.
    - int 를 반환하고 인수 받지 않는 함수의 포인터 형
        ```objectivec
        int (*fnPt) (void);
        ```
    - 괄호들은 반드시 씌워야 한다 이름 부분에 괄호를 씌우지 않으면 int 의 포인터 형인 줄 알고 오류가 날것이다.
    - 함수 포인터가 특정 함수를 가리키도록 하려면 함수 이름을 대입하면 된다.
        ```objectivec
        fnPt = lookup;
        fnPt = lookup(void);
        ```
    - 이름 뒤에 괄호를 붙이지 않으면 첨자 없이 사용하는 것과 유사
    - 지정된 함수의 포인터를 자동으로 생성, & 를 붙일 수 있지만 꼭 붙이지 않아도 됨

6. 기타 기능
- 복합 리터럴
    - 초기화 목록이 뛰따르는, 괄호 안에 표시된 형 이름
        ```objectivec
        (struct date) {.month = 1, .day = 9, .year = 2023}
        ```
    - 이것은 지정된 형의 이름 없는 값을 생성한다.

- goto 문

- null 문
    - 일반 명령문이 나와야 하는데 ; 가 나오는 경우 아무 작업도 하지 않는 명령문이다.

- 콤마 연산자
- sizeof 연산자
- 커맨드라인 인수
