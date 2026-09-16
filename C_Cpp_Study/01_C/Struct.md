# Struct

> **학습 목표**
>
> C의 구조체(`struct`)를 사용해 서로 관련된 여러 데이터를 하나의 사용자
> 정의 타입으로 묶는 방법을 학습한다.
>
> 단순 문법을 넘어 **구조체 객체 → 멤버 → 메모리 배치 → 포인터 → `.` /
> `->` → 함수 전달 → 배열 → 임베디드 데이터 모델**의 연결을 이해한다.
>
> 이 노트는 `C_Cpp_Study/01_C/Struct.md`에 위치하며 `Memory_Model.md`
> 다음 단계로 학습한다.

------------------------------------------------------------------------

# 1. 왜 구조체가 필요한가?

센서 정보를 저장한다고 가정한다.

``` c
int sensor_id = 1;
float temperature = 25.5f;
int status = 0;
```

변수가 늘어나면 서로 관련된 데이터라는 관계가 코드에 명확하게 드러나지
않는다.

구조체를 사용하면 하나로 묶을 수 있다.

``` c
struct Sensor
{
    int id;
    float temperature;
    int status;
};
```

``` mermaid
flowchart LR
    A["Sensor"] --> B["id"]
    A --> C["temperature"]
    A --> D["status"]
```

> **구조체는 서로 관련된 여러 데이터를 하나의 타입으로 표현할 수 있게
> 한다.**

------------------------------------------------------------------------

# 2. 한 문장 정의

**구조체(`struct`)는 여러 객체를 멤버(Member)로 묶어 새로운 구조를
정의하는 기능이다.**

예:

``` c
struct Point
{
    int x;
    int y;
};
```

`Point`라는 구조는 다음 데이터를 가진다.

``` text
Point
├── x : int
└── y : int
```

------------------------------------------------------------------------

# 3. 구조체 정의

기본 문법:

``` c
struct 구조체태그
{
    자료형 멤버1;
    자료형 멤버2;
};
```

예:

``` c
struct Student
{
    int id;
    char grade;
    float score;
};
```

> \[!IMPORTANT\] 구조체 정의 끝의 `;`를 빠뜨리지 않는다.

------------------------------------------------------------------------

# 4. 구조체 객체 선언

구조체를 정의한 뒤 객체를 선언할 수 있다.

``` c
struct Student student;
```

여기서:

``` text
struct Student → 타입
student        → 객체 이름
```

``` mermaid
flowchart LR
    A["struct Student"] --> B["student"]
    B --> C["id"]
    B --> D["grade"]
    B --> E["score"]
```

------------------------------------------------------------------------

# 5. 멤버 접근 연산자 `.`

구조체 객체의 멤버에 접근할 때 `.`을 사용한다.

``` c
student.id = 1;
student.grade = 'A';
student.score = 95.5f;
```

출력:

``` c
printf("%d\n", student.id);
printf("%c\n", student.grade);
printf("%.1f\n", student.score);
```

형식:

``` text
구조체객체.멤버
```

------------------------------------------------------------------------

# 6. 구조체 초기화

``` c
struct Point
{
    int x;
    int y;
};

struct Point p = {10, 20};
```

결과:

``` text
p.x = 10
p.y = 20
```

``` mermaid
flowchart LR
    A["{10, 20}"] --> B["p"]
    B --> C["x = 10"]
    B --> D["y = 20"]
```

------------------------------------------------------------------------

# 7. 지정 초기화

C에서는 지정 초기화(Designated Initializer)를 사용할 수 있다.

``` c
struct Point p =
{
    .x = 10,
    .y = 20
};
```

멤버 이름이 코드에 직접 나타나므로 큰 구조체에서 의미를 파악하기 쉽다.

``` c
struct Sensor sensor =
{
    .id = 1,
    .temperature = 25.5f,
    .status = 0
};
```

------------------------------------------------------------------------

# 8. 구조체의 일부 초기화

``` c
struct Point p = {10};
```

첫 멤버는 `10`으로 초기화되고 나머지 멤버는 해당 초기화 규칙에 따라
0으로 초기화된다.

더 명확하게 작성하려면:

``` c
struct Point p =
{
    .x = 10
};
```

처럼 지정 초기화를 사용할 수 있다.

------------------------------------------------------------------------

# 9. 구조체와 메모리

``` c
struct Point
{
    int x;
    int y;
};

struct Point p;
```

구조체 객체는 멤버들을 포함하는 하나의 객체다.

개념적으로:

``` text
p
┌─────────────┐
│ x           │
├─────────────┤
│ y           │
└─────────────┘
```

하지만 실제 메모리에는 정렬(Alignment)을 위해 멤버 사이 또는 끝에
Padding이 들어갈 수 있다.

------------------------------------------------------------------------

# 10. `sizeof(struct)`

``` c
printf("%zu\n", sizeof(struct Point));
```

구조체의 크기가 단순히 모든 멤버의 `sizeof` 합과 항상 같다고 가정하면 안
된다.

예:

``` c
struct Example
{
    char a;
    int b;
};
```

개념적으로:

``` text
a
padding 가능
b
```

``` mermaid
flowchart LR
    A["Struct Members"] --> B["Alignment Requirement"]
    B --> C["Padding 가능"]
    C --> D["sizeof(struct)"]
```

> \[!IMPORTANT\] Padding의 정확한 크기와 정렬 규칙은 ABI, 컴파일러,
> 타깃에 따라 달라질 수 있다. 직접 `sizeof`와 `offsetof` 등을 사용해
> 확인할 수 있다.

------------------------------------------------------------------------

# 11. Alignment와 Padding

CPU는 특정 타입을 특정 주소 경계에 배치할 때 효율적으로 접근할 수 있는
경우가 많다.

이를 위해 컴파일러가 Padding을 추가할 수 있다.

예시 개념도:

``` text
struct Example
{
    char c;
    int value;
};
```

가능한 개념:

``` text
┌──────────┐
│ char c   │
├──────────┤
│ padding  │
├──────────┤
│ int      │
│ value    │
└──────────┘
```

> \[!NOTE\] 그림의 실제 byte 수를 고정해서 외우지 않는다. 플랫폼별로
> 확인한다.

------------------------------------------------------------------------

# 12. 멤버 순서와 구조체 크기

멤버 순서에 따라 Padding 양이 달라질 수 있다.

``` c
struct A
{
    char c;
    int value;
    char flag;
};
```

``` c
struct B
{
    int value;
    char c;
    char flag;
};
```

두 구조체의 `sizeof`가 환경에 따라 다를 수 있다.

실습:

``` c
printf("A = %zu\n", sizeof(struct A));
printf("B = %zu\n", sizeof(struct B));
```

> 임베디드에서는 RAM 사용량과 통신 데이터 구조를 설계할 때 구조체 배치를
> 의식해야 한다.

------------------------------------------------------------------------

# 13. 구조체 복사

같은 구조체 타입의 객체끼리는 대입할 수 있다.

``` c
struct Point a = {10, 20};
struct Point b;

b = a;
```

이후:

``` text
b.x = 10
b.y = 20
```

구조체의 멤버 값들이 복사된다.

``` mermaid
flowchart LR
    A["struct Point a"] -->|"b = a"| B["struct Point b"]
```

------------------------------------------------------------------------

# 14. 구조체를 함수에 값으로 전달

``` c
struct Point
{
    int x;
    int y;
};

void print_point(struct Point p)
{
    printf("%d, %d\n", p.x, p.y);
}
```

호출:

``` c
struct Point p = {10, 20};

print_point(p);
```

함수 매개변수 `p`는 값으로 전달된다.

큰 구조체는 복사 비용을 고려해야 할 수 있다.

------------------------------------------------------------------------

# 15. 구조체 포인터

구조체 역시 객체이므로 주소를 얻을 수 있다.

``` c
struct Point p = {10, 20};

struct Point *ptr = &p;
```

``` mermaid
flowchart LR
    A["ptr"] --> B["struct Point p"]
    B --> C["x"]
    B --> D["y"]
```

이전 `Pointer.md`의 개념이 그대로 적용된다.

``` text
&p   → p의 주소
ptr  → p의 주소
*ptr → p 객체
```

------------------------------------------------------------------------

# 16. 구조체 포인터와 `.`

포인터를 역참조한 뒤 `.`을 사용할 수 있다.

``` c
(*ptr).x = 100;
```

괄호가 필요한 이유는 연산자 우선순위 때문이다.

``` text
(*ptr).x
```

의 의미:

``` text
ptr 역참조
 ↓
구조체 객체
 ↓
x 멤버 접근
```

------------------------------------------------------------------------

# 17. `->` 연산자

구조체 포인터에서는 `->`를 사용하면 더 간결하게 멤버에 접근할 수 있다.

``` c
ptr->x = 100;
```

이는 다음과 대응한다.

``` c
(*ptr).x = 100;
```

핵심 관계:

``` text
object.member
pointer->member
```

``` mermaid
flowchart TD
    A["Struct Object"] --> B["."]
    C["Struct Pointer"] --> D["->"]
    B --> E["Member"]
    D --> E
```

------------------------------------------------------------------------

# 18. `.`과 `->` 구분

  가지고 있는 것   접근 방식
  ---------------- -------------------
  구조체 객체      `object.member`
  구조체 포인터    `pointer->member`

예:

``` c
struct Point p;
struct Point *ptr = &p;

p.x = 10;
ptr->y = 20;
```

> \[!TIP\] `.`과 `->`가 헷갈리면 현재 변수의 타입을 먼저 확인한다.

------------------------------------------------------------------------

# 19. 함수에서 구조체 수정

포인터를 전달하면 원본 구조체를 수정할 수 있다.

``` c
void reset_point(struct Point *p)
{
    p->x = 0;
    p->y = 0;
}
```

호출:

``` c
struct Point point = {10, 20};

reset_point(&point);
```

``` mermaid
sequenceDiagram
    participant M as main
    participant P as point
    participant F as reset_point

    M->>F: &point 전달
    F->>P: p->x = 0
    F->>P: p->y = 0
```

------------------------------------------------------------------------

# 20. 읽기 전용 구조체 포인터

함수가 구조체를 수정할 필요가 없다면:

``` c
void print_point(const struct Point *p)
{
    printf("%d, %d\n", p->x, p->y);
}
```

`const struct Point *`를 사용할 수 있다.

``` mermaid
flowchart LR
    A["const struct Point *"] --> B["원본 구조체 참조"]
    B --> C["이 포인터를 통한 수정 제한"]
```

큰 구조체를 불필요하게 복사하지 않으면서 읽기 전용 의도를 표현할 수
있다.

------------------------------------------------------------------------

# 21. 구조체 배열

``` c
struct Point points[3] =
{
    {1, 2},
    {3, 4},
    {5, 6}
};
```

접근:

``` c
printf("%d\n", points[0].x);
printf("%d\n", points[1].y);
```

``` mermaid
flowchart TD
    A["points[]"] --> B["points[0]"]
    A --> C["points[1]"]
    A --> D["points[2]"]

    B --> E["x / y"]
    C --> F["x / y"]
    D --> G["x / y"]
```

------------------------------------------------------------------------

# 22. 구조체 배열 순회

``` c
for (int i = 0; i < 3; i++)
{
    printf("%d %d\n",
           points[i].x,
           points[i].y);
}
```

코딩 테스트, 센서 목록, 장치 테이블, 패킷 목록 등에서 자주 사용하는
형태다.

------------------------------------------------------------------------

# 23. 구조체 안에 배열

``` c
struct Student
{
    int id;
    char name[32];
    int scores[3];
};
```

구조:

``` mermaid
flowchart TD
    A["Student"] --> B["id"]
    A --> C["name[32]"]
    A --> D["scores[3]"]
```

접근:

``` c
student.name[0]
student.scores[2]
```

------------------------------------------------------------------------

# 24. 구조체 안에 구조체

구조체는 다른 구조체를 멤버로 포함할 수 있다.

``` c
struct Date
{
    int year;
    int month;
    int day;
};

struct Student
{
    int id;
    struct Date birthday;
};
```

접근:

``` c
student.birthday.year
```

``` mermaid
flowchart LR
    A["Student"] --> B["id"]
    A --> C["birthday"]
    C --> D["year"]
    C --> E["month"]
    C --> F["day"]
```

------------------------------------------------------------------------

# 25. `typedef`와 구조체

C에서는 다음처럼 작성한다.

``` c
struct Point
{
    int x;
    int y;
};

struct Point p;
```

`typedef`를 이용하면 별칭을 만들 수 있다.

``` c
typedef struct Point Point;
```

이후:

``` c
Point p;
```

처럼 사용할 수 있다.

------------------------------------------------------------------------

# 26. 흔한 `typedef struct` 형태

``` c
typedef struct
{
    int x;
    int y;
} Point;
```

이후:

``` c
Point p = {10, 20};
```

임베디드 C 코드에서 자주 볼 수 있는 스타일이다.

또 다른 형태:

``` c
typedef struct Sensor
{
    int id;
    int status;
} Sensor;
```

이 경우 구조체 태그 `Sensor`와 typedef 이름 `Sensor`를 함께 정의하는
패턴이다.

------------------------------------------------------------------------

# 27. `typedef`는 새로운 객체를 만드는 것이 아니다

``` c
typedef unsigned int uint_alias;
```

`typedef`는 타입 이름의 별칭을 제공한다.

구조체에서도 마찬가지다.

``` c
typedef struct Point Point;
```

``` mermaid
flowchart LR
    A["struct Point"] --> B["typedef"]
    B --> C["Point라는 타입 별칭"]
```

------------------------------------------------------------------------

# 28. 자기 자신을 가리키는 구조체

연결 리스트 같은 자료구조에서는 구조체가 같은 타입의 다른 객체를
가리키는 포인터를 포함할 수 있다.

``` c
struct Node
{
    int value;
    struct Node *next;
};
```

``` mermaid
flowchart LR
    A["Node A<br/>value=10"] --> B["Node B<br/>value=20"]
    B --> C["Node C<br/>value=30"]
    C --> D["NULL"]
```

구조체 안에 자기 자신을 **값으로 직접 포함**할 수는 없지만 자기 타입을
가리키는 포인터는 가질 수 있다.

------------------------------------------------------------------------

# 29. 연결 리스트로의 연결

``` c
struct Node
{
    int value;
    struct Node *next;
};
```

각 Node는:

``` text
value
+
next pointer
```

를 가진다.

``` mermaid
flowchart LR
    A["value | next"] --> B["value | next"]
    B --> C["value | NULL"]
```

이 구조는 Pointer + Struct + Dynamic Memory가 결합되는 대표 사례다.

------------------------------------------------------------------------

# 30. 구조체와 동적 메모리

``` c
struct Point *ptr = malloc(sizeof(struct Point));
```

할당 성공 여부를 확인한 뒤:

``` c
if (ptr != NULL)
{
    ptr->x = 10;
    ptr->y = 20;

    free(ptr);
}
```

``` mermaid
flowchart LR
    A["malloc"] --> B["struct Point 저장 공간"]
    B --> C["Point *ptr"]
    C --> D["ptr->member"]
    D --> E["free"]
```

`malloc_free.md`에서 더 자세히 다룬다.

------------------------------------------------------------------------

# 31. `sizeof *ptr` 패턴

다음 코드:

``` c
struct Point *ptr = malloc(sizeof(struct Point));
```

는 다음처럼 작성할 수도 있다.

``` c
struct Point *ptr = malloc(sizeof *ptr);
```

장점은 포인터가 가리키는 타입을 `sizeof` 안에 반복해서 적지 않는다는
것이다.

``` c
struct Point *ptr = malloc(sizeof *ptr);
```

현재는 두 표현 모두 이해할 수 있으면 된다.

------------------------------------------------------------------------

# 32. 구조체와 함수 인터페이스

드라이버나 라이브러리에서는 관련 상태를 구조체로 묶고 함수에 포인터를
전달하는 패턴이 흔하다.

``` c
typedef struct
{
    int id;
    int state;
} Device;

void device_init(Device *device);
void device_update(Device *device);
void device_print(const Device *device);
```

``` mermaid
flowchart TD
    A["Device Struct"] --> B["device_init"]
    A --> C["device_update"]
    A --> D["device_print"]
```

이 구조는 C에서 객체 지향적인 설계를 일부 표현할 때도 사용된다.

------------------------------------------------------------------------

# 33. 임베디드 드라이버 Context

예:

``` c
typedef struct
{
    unsigned char *rx_buffer;
    unsigned int rx_size;
    unsigned int state;
} UartContext;
```

``` mermaid
flowchart TD
    A["UartContext"] --> B["rx_buffer"]
    A --> C["rx_size"]
    A --> D["state"]

    A --> E["UART Driver Functions"]
```

관련 상태를 하나로 묶으면 함수 인자와 전역 변수 사용을 정리하기
쉬워진다.

------------------------------------------------------------------------

# 34. 구조체와 상태 머신

FSM에서도 구조체를 사용할 수 있다.

``` c
typedef struct
{
    int state;
    int retry_count;
    int timeout;
} Controller;
```

``` mermaid
flowchart LR
    A["Controller Struct"] --> B["Current State"]
    A --> C["Retry Count"]
    A --> D["Timeout"]
    B --> E["FSM Logic"]
```

이후 `FSM.md`에서 확장한다.

------------------------------------------------------------------------

# 35. 구조체와 통신 패킷

패킷 데이터를 구조체로 표현하고 싶을 수 있다.

``` c
typedef struct
{
    unsigned char type;
    unsigned short length;
    unsigned int value;
} Packet;
```

하지만 실제 통신 바이트 배열을 구조체에 그대로 덮어씌우는 방식은 주의가
필요하다.

이유:

``` text
Padding
Alignment
Endianness
Type Width
Protocol Packing Rules
```

``` mermaid
flowchart TD
    A["Wire Bytes"] --> B["Padding?"]
    A --> C["Endianness?"]
    A --> D["Alignment?"]
    A --> E["Exact Width?"]
    B --> F["직접 구조체 매핑 주의"]
    C --> F
    D --> F
    E --> F
```

> \[!WARNING\] **구조체의 메모리 배치와 통신 프로토콜의 바이트 배치가
> 자동으로 같다고 가정하지 않는다.**

------------------------------------------------------------------------

# 36. 고정 폭 정수형과 구조체

임베디드 및 프로토콜에서는 다음 타입을 자주 사용한다.

``` c
#include <stdint.h>

typedef struct
{
    uint8_t type;
    uint16_t length;
    uint32_t value;
} Packet;
```

이 타입들은 크기 의도를 명확하게 표현하는 데 유용하다.

단, 구조체 Padding과 Endianness 문제는 별도로 고려해야 한다.

------------------------------------------------------------------------

# 37. 구조체를 레지스터에 매핑하는 코드 예고

일부 MCU 헤더에서는 주변장치 레지스터 블록을 구조체로 표현한다.

개념적인 예:

``` c
typedef struct
{
    volatile uint32_t CTRL;
    volatile uint32_t STATUS;
    volatile uint32_t DATA;
} PeripheralRegisters;
```

그리고 특정 base address와 연결하는 형태가 등장할 수 있다.

``` mermaid
flowchart LR
    A["Base Address"] --> B["Struct Pointer"]
    B --> C["CTRL"]
    B --> D["STATUS"]
    B --> E["DATA"]
    C --> F["Hardware"]
    D --> F
    E --> F
```

> \[!IMPORTANT\] 실제 레지스터 구조체는 MCU 제조사가 제공하는 CMSIS/SDK
> 헤더와 데이터시트의 정확한 레이아웃을 따라야 한다. 임의로 추측해
> 작성하지 않는다.

------------------------------------------------------------------------

# 38. 구조체와 `volatile`

레지스터 구조체에서는 다음과 같은 표현을 볼 수 있다.

``` c
volatile uint32_t STATUS;
```

또는 구조체 포인터 자체에 `volatile`이 관련될 수 있다.

구조체, 포인터, `volatile`, MMIO가 결합되는 것이다.

``` text
Struct
+
Pointer
+
volatile
+
Memory Map
=
Peripheral Register Interface
```

세부 규칙은 `volatile.md`에서 학습한다.

------------------------------------------------------------------------

# 39. 구조체와 C++

C++에서도 `struct`를 사용할 수 있다.

``` cpp
struct Point
{
    int x;
    int y;
};
```

C++에서는:

``` cpp
Point p;
```

처럼 `struct Point` 대신 타입 이름 `Point`를 바로 사용할 수 있다.

또한 C++ 구조체는 멤버 함수, 생성자 등도 가질 수 있다.

``` mermaid
flowchart LR
    A["C struct"] --> B["Data Grouping"]
    C["C++ struct"] --> B
    C --> D["Member Functions 등 확장"]
```

클래스와의 자세한 비교는 `02_CPP/Class.md`에서 다룬다.

------------------------------------------------------------------------

# 40. C와 C++ 구조체 기본 비교

  항목             C                            C++
  ---------------- ---------------------------- ------------
  구조체 정의      `struct Point { ... };`      동일 가능
  객체 선언        `struct Point p;`            `Point p;`
  멤버 함수        일반적인 C struct에는 없음   가능
  접근 제어        없음                         가능
  기본 접근 수준   해당 개념 없음               `public`

현재는 C 구조체의 메모리와 데이터 묶음 역할에 집중한다.

------------------------------------------------------------------------

# 41. 구조체 사용 시 흔한 실수

### 1. 정의 뒤 세미콜론 누락

``` c
struct Point
{
    int x;
    int y;
}   /* ; 누락 */
```

### 2. `.`과 `->` 혼동

``` c
struct Point p;
struct Point *ptr = &p;

p.x = 10;
ptr->x = 20;
```

### 3. 초기화되지 않은 멤버 사용

``` c
struct Point p;

printf("%d\n", p.x);
```

자동 저장 기간의 `p`가 초기화되지 않았다면 멤버 값을 읽으면 안 된다.

### 4. Padding 무시

네트워크/파일/레지스터 형식과 구조체 메모리 레이아웃이 항상 같다고
가정하지 않는다.

------------------------------------------------------------------------

# 42. 구조체 디버깅 사고 방식

``` mermaid
flowchart TD
    A["구조체 타입은?"] --> B["객체인가 포인터인가?"]
    B --> C[". 또는 -> 선택"]
    C --> D["멤버 타입은?"]
    D --> E["초기화되었는가?"]
    E --> F["Padding / Alignment 영향이 있는가?"]
    F --> G["포인터라면 Lifetime은 유효한가?"]
```

------------------------------------------------------------------------

# 43. 실습 1 --- Point

다음 구조체를 만든다.

``` c
struct Point
{
    int x;
    int y;
};
```

객체를 생성하고:

``` text
x = 10
y = 20
```

으로 초기화한 뒤 출력한다.

------------------------------------------------------------------------

# 44. 실습 2 --- Student

다음을 표현한다.

``` text
학번
이름
점수
```

예:

``` c
struct Student
{
    int id;
    char name[32];
    float score;
};
```

직접 초기화하고 출력한다.

------------------------------------------------------------------------

# 45. 실습 3 --- 구조체 수정 함수

``` c
void reset_point(struct Point *p);
```

호출 전:

``` text
x = 10
y = 20
```

호출 후:

``` text
x = 0
y = 0
```

이 되도록 구현한다.

------------------------------------------------------------------------

# 46. 실습 4 --- 구조체 배열

학생 3명을 저장한다.

``` c
struct Student students[3];
```

반복문으로 각 학생 정보를 출력한다.

------------------------------------------------------------------------

# 47. 실습 5 --- 구조체 크기 확인

다음을 직접 실행한다.

``` c
struct A
{
    char c;
    int value;
    char flag;
};

struct B
{
    int value;
    char c;
    char flag;
};

printf("%zu\n", sizeof(struct A));
printf("%zu\n", sizeof(struct B));
```

결과를 기록하고 Padding 관점에서 이유를 추론한다.

------------------------------------------------------------------------

# 48. 실습 6 --- 구조체 포인터

``` c
struct Point p = {10, 20};
struct Point *ptr = &p;
```

다음을 각각 사용해 같은 값을 읽는다.

``` c
p.x
(*ptr).x
ptr->x
```

------------------------------------------------------------------------

# 49. 실습 7 --- 중첩 구조체

``` c
struct Date
{
    int year;
    int month;
    int day;
};

struct Event
{
    int id;
    struct Date date;
};
```

다음을 설정한다.

``` text
id = 1
date = 2026-09-16
```

멤버 접근식을 직접 작성한다.

------------------------------------------------------------------------

# 50. 실습 8 --- Node

``` c
struct Node
{
    int value;
    struct Node *next;
};
```

Stack에 Node 3개를 만들고:

``` text
10 → 20 → 30 → NULL
```

형태로 연결한다.

동적 메모리는 아직 사용하지 않아도 된다.

------------------------------------------------------------------------

# 51. 코드 추적 연습

``` c
struct Point
{
    int x;
    int y;
};

struct Point p = {10, 20};
struct Point *ptr = &p;

ptr->x += 5;
p.y *= 2;
```

표를 채운다.

  단계              `p.x`   `p.y`   `ptr->x`   `ptr->y`
  --------------- ------- ------- ---------- ----------
  초기                 10      20         10         20
  `ptr->x += 5`                              
  `p.y *= 2`                                 

------------------------------------------------------------------------

# 52. 반드시 설명할 수 있어야 하는 것

-   [ ] 구조체가 필요한 이유를 설명할 수 있다.
-   [ ] `struct`를 정의할 수 있다.
-   [ ] 구조체 객체를 선언할 수 있다.
-   [ ] `.` 연산자로 멤버에 접근할 수 있다.
-   [ ] 구조체를 초기화할 수 있다.
-   [ ] 지정 초기화를 이해한다.
-   [ ] `sizeof(struct)`가 단순 멤버 크기의 합과 다를 수 있는 이유를
    설명할 수 있다.
-   [ ] Alignment와 Padding의 기본 개념을 설명할 수 있다.
-   [ ] 같은 구조체 타입끼리 대입할 수 있음을 이해한다.
-   [ ] 구조체를 함수에 값으로 전달할 수 있다.
-   [ ] 구조체 포인터를 선언할 수 있다.
-   [ ] `(*ptr).member`와 `ptr->member`의 관계를 설명할 수 있다.
-   [ ] `.`과 `->`를 구분할 수 있다.
-   [ ] 구조체 포인터를 함수에 전달해 원본을 수정할 수 있다.
-   [ ] `const struct T *`의 목적을 설명할 수 있다.
-   [ ] 구조체 배열을 사용할 수 있다.
-   [ ] 구조체 안에 배열/구조체를 포함할 수 있다.
-   [ ] `typedef struct`의 기본 형태를 사용할 수 있다.
-   [ ] 자기 참조 구조체의 의미를 설명할 수 있다.
-   [ ] 구조체와 Linked List의 연결을 설명할 수 있다.
-   [ ] 구조체와 동적 메모리의 연결을 설명할 수 있다.
-   [ ] 통신 패킷을 구조체에 직접 매핑할 때 주의할 점을 설명할 수 있다.
-   [ ] 임베디드 드라이버 Context에 구조체가 유용한 이유를 설명할 수
    있다.
-   [ ] 구조체가 MMIO 레지스터 표현과 연결될 수 있음을 설명할 수 있다.

------------------------------------------------------------------------

# 53. 복습 문제

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q1. 구조체란?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 여러 멤버를 하나의 구조로 묶어 사용자 정의 데이터 구조를
표현하는 기능이다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q2. 구조체 객체의 멤버에 접근하는
연산자는?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** `.`

``` c
point.x
```

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q3. 구조체 포인터의 멤버에 접근하는
연산자는?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** `->`

``` c
ptr->x
```

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q4. ptr-\>x와 대응하는 표현은?`</strong>`{=html}
```{=html}
</summary>
```
**정답:**

``` c
(*ptr).x
```

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q5. sizeof(struct)가 멤버 sizeof 합보다 커질 수 있는
이유는?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 정렬 요구사항을 맞추기 위해 Padding이 삽입될 수 있기 때문이다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q6. 지역 구조체 객체의 초기화되지 않은 멤버를 읽어도
되는가?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 자동 저장 기간 객체가 적절히 초기화되지 않았다면 해당 불확정
값을 읽는 코드를 작성하면 안 된다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q7. 구조체가 자기 자신을 멤버로 직접 포함할 수
있는가?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 완전한 자기 타입 객체를 값으로 직접 포함할 수는 없지만 자기
타입을 가리키는 포인터는 포함할 수 있다.

``` c
struct Node *next;
```

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q8. 구조체 메모리 레이아웃을 통신 패킷과 항상
동일하다고 볼 수 있는가?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 아니다. Padding, Alignment, Endianness, 타입 폭 등을 고려해야
한다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q9. const struct Point \*p의 기본
목적은?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 포인터를 통해 원본 구조체를 읽되 해당 경로로 수정하지 않겠다는
의도를 표현하는 것이다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q10. 임베디드에서 구조체는 어디에 활용될 수
있는가?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 드라이버 상태, 센서 데이터, 버퍼 Context, FSM 상태, 통신
데이터 모델, 제조사 헤더의 레지스터 블록 표현 등에 활용될 수 있다.

```{=html}
</details>
```

------------------------------------------------------------------------

# 54. Bug Log

`99_Bug_Log/Memory_Bugs.md` 또는 구조체 관련 별도 항목에 기록할 수 있다.

```` markdown
## Struct Pointer에서 . 사용

### 잘못 작성한 코드

```c
struct Point *ptr = &point;

ptr.x = 10;
```

### 문제

`ptr`은 구조체 객체가 아니라 구조체 포인터다.

### 수정

```c
ptr->x = 10;
```

또는:

```c
(*ptr).x = 10;
```

### 재발 방지

멤버 접근 전에 변수 타입을 확인한다.

- 객체 → `.`
- 포인터 → `->`
````

추가 기록 항목:

``` text
Struct 초기화 누락
. / -> 혼동
Padding 가정 오류
잘못된 Packet Struct 매핑
Dangling Struct Pointer
구조체 배열 Out-of-Bounds
```

------------------------------------------------------------------------

# 55. 지금까지의 개념 연결

``` mermaid
flowchart LR
    A["Data Type"] --> B["Array / String"]
    B --> C["Pointer"]
    C --> D["Memory Model"]
    D --> E["Struct"]
```

구조체는 앞에서 배운 개념을 결합한다.

  기존 개념      Struct에서의 역할
  -------------- -----------------------
  Data Type      멤버 타입
  Array          구조체 배열/배열 멤버
  Function       구조체 전달
  Pointer        구조체 포인터
  Memory Model   Layout/Lifetime
  Operator       `.` / `->`

------------------------------------------------------------------------

# 56. C 자료구조로의 연결

``` mermaid
flowchart TD
    A["Struct"] --> B["Struct Pointer"]
    B --> C["Self-reference"]
    C --> D["Linked List"]
    C --> E["Tree"]
    C --> F["Graph Node"]
```

자료구조에서는 다음 세 개념이 반복된다.

``` text
Struct
+
Pointer
+
Dynamic Memory
```

------------------------------------------------------------------------

# 57. 임베디드로의 연결

``` mermaid
flowchart TD
    A["Struct"] --> B["Sensor Data"]
    A --> C["Driver Context"]
    A --> D["FSM State"]
    A --> E["Register Block"]
    A --> F["Packet Model"]

    C --> G["UART / SPI / I2C"]
    E --> H["MMIO"]
```

구조체는 펌웨어에서 단순 데이터 묶음 이상의 역할을 한다.

``` text
Hardware State
Driver State
Protocol State
Application State
```

를 명확하게 모델링하는 기본 도구가 된다.

------------------------------------------------------------------------

# 58. 핵심 요약

``` mermaid
mindmap
  root((Struct))
    Basic
      Definition
      Member
      Object
      Initialization
    Access
      dot
      arrow
    Memory
      sizeof
      Alignment
      Padding
    Pointer
      struct pointer
      const pointer
    Composition
      Array
      Nested Struct
      Self Reference
    Type
      typedef
    Data Structure
      Linked List
      Tree
    Embedded
      Driver Context
      Packet
      Register Block
      FSM
```

## 최종적으로 기억할 여덟 문장

> **1. 구조체는 서로 관련된 여러 데이터를 하나의 구조로 묶는다.**

> **2. 구조체 객체의 멤버는 `.`으로 접근한다.**

> **3. 구조체 포인터의 멤버는 `->`로 접근하며 `ptr->x`는 `(*ptr).x`와
> 대응한다.**

> **4. 구조체에는 Alignment 때문에 Padding이 들어갈 수 있으므로 크기를
> 단순 합으로 가정하지 않는다.**

> **5. 구조체 포인터를 함수에 전달하면 큰 객체의 불필요한 복사를 피하고
> 원본을 다룰 수 있다.**

> **6. Struct + Pointer는 Linked List와 같은 C 자료구조의 핵심
> 기반이다.**

> **7. 통신 바이트 배열과 구조체 메모리 레이아웃이 자동으로 동일하다고
> 가정하지 않는다.**

> **8. 임베디드에서는 구조체가 Driver Context, FSM, Buffer, Register
> Block 등의 모델링에 활용된다.**

------------------------------------------------------------------------

# 59. 현재 학습 진행 상태

``` mermaid
flowchart LR
    A["Pointer.md<br/>완료"] --> B["Memory_Model.md<br/>완료"]
    B --> C["Struct.md<br/>현재"]
    C --> D["malloc_free.md"]
```

``` text
C_Cpp_Study/
├── 00_Common/
│   └── Memory_Model.md    ✓
│
└── 01_C/
    ├── Array_String.md    ✓
    ├── Pointer.md         ✓
    ├── Struct.md          ← 현재
    ├── malloc_free.md
    ├── Bit_Operation.md
    ├── volatile.md
    └── Embedded_C.md
```

------------------------------------------------------------------------

# 60. 다음 학습

다음 단계에서는 Pointer와 Memory Model을 실제 동적 메모리 관리에
적용한다.

``` mermaid
flowchart LR
    A["Pointer"] --> D["Dynamic Memory"]
    B["Memory Model"] --> D
    C["Struct"] --> D
    D --> E["malloc"]
    D --> F["calloc"]
    D --> G["realloc"]
    D --> H["free"]
    D --> I["Memory Bugs"]
```

다음 질문을 해결한다.

``` text
malloc은 정확히 무엇을 반환하는가?
sizeof와 malloc을 어떻게 함께 사용하는가?
배열을 동적으로 만드는 방법은?
calloc과 malloc의 차이는?
realloc은 언제 필요한가?
누가 free해야 하는가?
Memory Leak은 어떻게 예방하는가?
Use-After-Free와 Double Free는 어떻게 발생하는가?
```

**다음 문서:** `01_C/malloc_free.md`
