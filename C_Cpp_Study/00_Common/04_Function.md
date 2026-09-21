# Function

> **학습 목표**
>
> C와 C++에서 함수(Function)가 왜 필요한지 이해하고, **함수 선언 →
> 매개변수 → 인자 → 반환값 → 지역 변수 → 스코프 → 호출 스택 →
> 배열/포인터 → 모듈화 → 임베디드 드라이버**로 이어지는 관계를 이해한다.
>
> 이 노트는 `C_Cpp_Study/00_Common/Function.md`에 위치하며
> `Data_Type.md`, `Operator.md`, `Control_Flow.md` 다음 단계로 학습한다.

------------------------------------------------------------------------

## 1. 함수란?

### 한 문장 정의

**함수(Function)는 특정 작업을 수행하도록 여러 문장을 하나의 이름으로
묶은 코드 단위다.**

예:

``` c
int add(int a, int b)
{
    return a + b;
}
```

사용:

``` c
int result = add(10, 20);
```

동작을 개념적으로 보면 다음과 같다.

``` mermaid
flowchart LR
    A["10, 20"] --> B["add() 호출"]
    B --> C["a + b"]
    C --> D["30 반환"]
    D --> E["result"]
```

함수는 다음 구조를 가진다.

``` text
입력
 ↓
함수
 ↓
처리
 ↓
출력
```

> **핵심 관점:** 함수는 단순한 문법이 아니라 프로그램을 작은 작업 단위로
> 나누는 방법이다.

------------------------------------------------------------------------

# 2. 함수가 필요한 이유

다음 작업을 여러 곳에서 반복한다고 가정한다.

``` c
int result1 = a + b;
int result2 = c + d;
int result3 = e + f;
```

공통 작업을 함수로 만들 수 있다.

``` c
int add(int x, int y)
{
    return x + y;
}
```

이후:

``` c
int result1 = add(a, b);
int result2 = add(c, d);
int result3 = add(e, f);
```

함수를 사용하면 다음 장점이 있다.

``` mermaid
mindmap
  root((Function))
    재사용
      같은 코드 반복 감소
    가독성
      작업에 이름 부여
    모듈화
      기능 분리
    유지보수
      수정 범위 축소
    테스트
      작은 단위 검증
```

------------------------------------------------------------------------

# 3. 함수의 기본 구조

``` c
int add(int a, int b)
{
    int result = a + b;

    return result;
}
```

구성 요소:

  부분              의미
  ----------------- -----------
  `int`             반환 타입
  `add`             함수 이름
  `int a, int b`    매개변수
  `{ ... }`         함수 본문
  `return result`   결과 반환

``` mermaid
flowchart TD
    A["int"] --> B["반환 타입"]
    C["add"] --> D["함수 이름"]
    E["int a, int b"] --> F["매개변수"]
    G["함수 본문"] --> H["처리"]
    I["return"] --> J["호출한 곳으로 결과 반환"]
```

------------------------------------------------------------------------

# 4. 함수 호출

함수를 실행하도록 요청하는 것을 **함수 호출(Function Call)**이라고 한다.

``` c
int result = add(10, 20);
```

여기서:

``` text
add(10, 20)
```

이 함수 호출이다.

실행 흐름:

``` mermaid
sequenceDiagram
    participant M as main()
    participant A as add()

    M->>A: add(10, 20)
    A->>A: 10 + 20
    A-->>M: return 30
```

------------------------------------------------------------------------

# 5. 매개변수와 인자

두 용어를 구분한다.

함수 정의:

``` c
int add(int a, int b)
{
    return a + b;
}
```

함수 호출:

``` c
int result = add(10, 20);
```

  용어                  예           의미
  --------------------- ------------ --------------------------------
  매개변수(Parameter)   `a`, `b`     함수 정의에서 입력을 받는 변수
  인자(Argument)        `10`, `20`   함수를 호출할 때 전달하는 값

``` mermaid
flowchart LR
    A["인자 10"] --> B["매개변수 a"]
    C["인자 20"] --> D["매개변수 b"]
    B --> E["add()"]
    D --> E
```

------------------------------------------------------------------------

# 6. 반환값

함수가 계산한 결과를 호출한 곳으로 전달할 때 `return`을 사용한다.

``` c
int square(int number)
{
    return number * number;
}
```

호출:

``` c
int result = square(5);
```

결과:

``` text
result = 25
```

``` mermaid
flowchart LR
    A["square(5)"] --> B["5 * 5"]
    B --> C["return 25"]
    C --> D["result = 25"]
```

------------------------------------------------------------------------

## 6.1 반환 타입

함수 앞의 자료형은 반환되는 값의 타입을 나타낸다.

``` c
int get_count(void);
double get_temperature(void);
char get_grade(void);
```

  함수                  반환 타입
  --------------------- -----------
  `get_count()`         `int`
  `get_temperature()`   `double`
  `get_grade()`         `char`

`Data_Type.md`에서 학습한 자료형이 함수와 연결된다.

``` mermaid
flowchart LR
    A["Data Type"] --> B["Parameter Type"]
    A --> C["Return Type"]
    B --> D["Function"]
    C --> D
```

------------------------------------------------------------------------

# 7. `void`

함수가 값을 반환하지 않을 때 `void`를 사용한다.

``` c
void print_hello(void)
{
    printf("Hello\n");
}
```

호출:

``` c
print_hello();
```

이 함수는 특정 동작을 수행하지만 결과값을 반환하지 않는다.

``` mermaid
flowchart LR
    A["print_hello()"] --> B["Hello 출력"]
    B --> C["반환값 없음"]
```

------------------------------------------------------------------------

## 7.1 입력도 없는 함수

C에서는 매개변수가 없음을 명확하게 표현할 때 다음처럼 작성한다.

``` c
void print_hello(void)
{
    printf("Hello\n");
}
```

C++에서는 다음 형태를 일반적으로 사용한다.

``` cpp
void print_hello()
{
    std::cout << "Hello\n";
}
```

> \[!NOTE\] C와 C++은 빈 매개변수 목록의 의미에 역사적인 차이가 있다. C
> 학습에서는 매개변수가 없음을 명확히 하기 위해 `(void)` 형태를
> 익혀둔다.

------------------------------------------------------------------------

# 8. `return`과 함수 종료

`return`을 만나면 현재 함수의 실행이 종료된다.

``` c
int get_value(void)
{
    return 10;

    /* 여기는 실행되지 않는다. */
}
```

조건과 함께 사용할 수도 있다.

``` c
int absolute(int value)
{
    if (value < 0)
    {
        return -value;
    }

    return value;
}
```

``` mermaid
flowchart TD
    A["value < 0?"] -->|"Yes"| B["return -value"]
    A -->|"No"| C["return value"]
    B --> D["함수 종료"]
    C --> D
```

------------------------------------------------------------------------

# 9. 지역 변수

함수 내부에서 선언된 변수는 일반적으로 해당 블록 안에서 사용한다.

``` c
int add(int a, int b)
{
    int result = a + b;

    return result;
}
```

`result`는 `add()` 함수 내부의 지역 변수다.

``` mermaid
flowchart TD
    A["add()"] --> B["a"]
    A --> C["b"]
    A --> D["result"]
```

함수 밖에서 다음처럼 직접 사용할 수 없다.

``` c
int add(int a, int b)
{
    int result = a + b;
    return result;
}

int main(void)
{
    /* result는 여기서 직접 사용할 수 없음 */
}
```

------------------------------------------------------------------------

# 10. 스코프

**스코프(Scope)는 이름을 사용할 수 있는 코드 영역이다.**

예:

``` c
int main(void)
{
    int a = 10;

    if (a > 0)
    {
        int b = 20;

        printf("%d\n", b);
    }

    return 0;
}
```

`b`는 `if` 블록 내부에서 선언되었다.

개념:

``` mermaid
flowchart TD
    A["main scope"] --> B["a"]
    A --> C["if block"]
    C --> D["b"]
```

`b`를 블록 밖에서 사용하려고 하면 문제가 된다.

``` c
if (a > 0)
{
    int b = 20;
}

printf("%d\n", b);  /* b는 이 영역에서 사용할 수 없음 */
```

------------------------------------------------------------------------

# 11. 같은 이름의 지역 변수

서로 다른 스코프에서는 같은 이름이 등장할 수 있다.

``` c
int value = 10;

if (value > 0)
{
    int value = 20;

    printf("%d\n", value);
}
```

내부 블록의 `value`가 바깥쪽 이름을 가리는 현상을 **shadowing**이라고
한다.

학습 초기에는 불필요하게 같은 이름을 중첩해서 사용하는 것을 피하는 편이
좋다.

> \[!TIP\] 스코프를 이해하기 전에는 변수 이름을 의도적으로 명확하게
> 구분한다.

------------------------------------------------------------------------

# 12. 값 전달

기본적인 C 함수 호출에서는 인자의 **값이 매개변수로 전달**된다고
이해한다.

``` c
void change_value(int value)
{
    value = 100;
}
```

호출:

``` c
int number = 10;

change_value(number);
```

`change_value()` 내부의 `value`를 변경해도 호출한 쪽의 `number`가 직접
변경되는 것은 아니다.

``` mermaid
flowchart LR
    A["number = 10"] --> B["값 10 전달"]
    B --> C["value = 10"]
    C --> D["value = 100"]
    D --> E["함수 종료"]
    A --> F["number는 별도 객체"]
```

> **현재 단계의 핵심:** 함수에 정수 변수를 전달했다고 해서 원래 변수가
> 자동으로 변경되는 것은 아니다.

------------------------------------------------------------------------

# 13. C의 Pass-by-Value와 이후 포인터

C에서는 함수 인자가 값으로 전달된다.

원본 데이터를 함수에서 변경해야 하는 상황에서는 이후 포인터를 사용하게
된다.

현재:

``` c
void change_value(int value)
{
    value = 100;
}
```

이후 포인터 학습:

``` c
void change_value(int *value)
{
    *value = 100;
}
```

``` mermaid
flowchart LR
    A["Function"] --> B["Pass by Value"]
    B --> C["원본과 별도 값"]
    C --> D["원본 변경 필요"]
    D --> E["Pointer"]
```

현재는 포인터 문법을 외우지 않는다.

**함수의 값 전달 개념이 포인터 학습의 출발점이라는 것만 기억한다.**

------------------------------------------------------------------------

# 14. C++의 Reference로 확장

C++에서는 이후 Reference를 배우게 된다.

개념적인 형태:

``` cpp
void change_value(int& value)
{
    value = 100;
}
```

학습 연결:

``` mermaid
flowchart TD
    A["Function Parameter"] --> B["C"]
    A --> C["C++"]

    B --> D["Value"]
    D --> E["Pointer"]

    C --> F["Value"]
    F --> G["Reference"]
    F --> H["Pointer"]
```

현재 `00_Common`에서는 함수의 공통 기본 구조를 먼저 익힌다.

------------------------------------------------------------------------

# 15. 함수 선언과 정의

함수는 **선언(Declaration)**과 **정의(Definition)**를 구분해서 이해해야
한다.

함수 선언:

``` c
int add(int a, int b);
```

함수 정의:

``` c
int add(int a, int b)
{
    return a + b;
}
```

  구분   역할
  ------ -----------------------------------
  선언   함수의 이름과 인터페이스를 알려줌
  정의   함수가 실제로 무엇을 하는지 구현

``` mermaid
flowchart LR
    A["Declaration"] --> B["함수의 존재와 형태 알림"]
    C["Definition"] --> D["실제 코드 구현"]
```

------------------------------------------------------------------------

# 16. 함수 원형

다음과 같은 선언을 **함수 원형(Function Prototype)**이라고 부른다.

``` c
int add(int a, int b);
```

이를 이용하면 함수 정의가 아래쪽에 있어도 먼저 함수를 사용할 수 있다.

``` c
#include <stdio.h>

int add(int a, int b);

int main(void)
{
    int result = add(10, 20);

    printf("%d\n", result);

    return 0;
}

int add(int a, int b)
{
    return a + b;
}
```

실행 흐름과 소스 코드의 배치 순서는 같은 개념이 아니다.

------------------------------------------------------------------------

# 17. 함수 호출 흐름

다음 코드를 보자.

``` c
#include <stdio.h>

int add(int a, int b)
{
    return a + b;
}

int main(void)
{
    int result = add(10, 20);

    printf("%d\n", result);

    return 0;
}
```

실행 흐름:

``` mermaid
sequenceDiagram
    participant OS as Program Start
    participant M as main()
    participant A as add()

    OS->>M: main 시작
    M->>A: add(10, 20)
    A-->>M: 30 반환
    M->>M: printf()
    M-->>OS: return 0
```

> \[!IMPORTANT\] 함수를 코드에 작성한 순서대로 모두 실행하는 것이
> 아니다. **호출된 함수가 실행된다.**

------------------------------------------------------------------------

# 18. 함수와 호출 스택

함수를 호출하면 실행 중인 함수의 정보와 지역 변수 등을 관리하기 위한
메모리 구조가 필요하다.

이후 `Memory_Model.md`에서 **Stack**을 자세히 학습한다.

현재는 다음 관계만 이해한다.

``` mermaid
flowchart TD
    A["main()"] --> B["functionA() 호출"]
    B --> C["functionA 실행"]
    C --> D["functionB() 호출"]
    D --> E["functionB 실행"]
    E --> F["functionB 종료"]
    F --> G["functionA로 복귀"]
    G --> H["functionA 종료"]
    H --> I["main으로 복귀"]
```

개념적인 호출 스택:

``` text
┌─────────────────┐
│ functionB()     │
├─────────────────┤
│ functionA()     │
├─────────────────┤
│ main()          │
└─────────────────┘
```

> \[!NOTE\] 정확한 스택 프레임, ABI, 레지스터 저장 규칙 등은 현재
> 단계에서 다루지 않는다.

------------------------------------------------------------------------

# 19. 함수 안에서 다른 함수 호출

함수는 다른 함수를 호출할 수 있다.

``` c
int square(int value)
{
    return value * value;
}

int sum_of_squares(int a, int b)
{
    return square(a) + square(b);
}
```

호출:

``` c
int result = sum_of_squares(3, 4);
```

``` mermaid
flowchart TD
    A["sum_of_squares(3, 4)"] --> B["square(3)"]
    A --> C["square(4)"]
    B --> D["9"]
    C --> E["16"]
    D --> F["9 + 16"]
    E --> F
    F --> G["25"]
```

------------------------------------------------------------------------

# 20. 함수 이름

함수 이름은 함수가 하는 일을 표현하도록 작성한다.

좋은 예:

``` c
calculate_sum()
read_sensor()
check_temperature()
send_packet()
initialize_uart()
```

의미가 불분명한 예:

``` c
func1()
do_it()
abc()
test2()
```

> **함수 이름은 코드의 설명 역할도 한다.**

임베디드에서는 특히 다음과 같은 이름을 자주 접하게 된다.

``` text
gpio_init()
uart_init()
uart_send()
adc_read()
timer_start()
sensor_update()
```

------------------------------------------------------------------------

# 21. 한 함수는 하나의 책임

처음부터 엄격한 설계 규칙으로 외울 필요는 없지만, 가능하면 하나의 함수가
하나의 명확한 작업을 담당하도록 한다.

좋은 구조의 개념:

``` mermaid
flowchart TD
    A["main()"] --> B["read_input()"]
    A --> C["process_data()"]
    A --> D["write_output()"]
```

모든 작업을 `main()`에 넣는 구조:

``` text
main()
 ├─ 입력
 ├─ 계산
 ├─ 검증
 ├─ 출력
 ├─ 통신
 ├─ 오류 처리
 └─ 상태 처리
```

프로그램이 커질수록 기능을 함수로 나누는 것이 중요해진다.

------------------------------------------------------------------------

# 22. `main()`도 함수다

지금까지 계속 사용한 `main()`도 함수다.

C:

``` c
int main(void)
{
    return 0;
}
```

C++:

``` cpp
int main()
{
    return 0;
}
```

`main()`은 프로그램 실행의 시작점 역할을 하는 특별한 함수다.

``` mermaid
flowchart TD
    A["Program Start"] --> B["main()"]
    B --> C["다른 함수 호출"]
    C --> D["main()으로 복귀"]
    D --> E["Program End"]
```

------------------------------------------------------------------------

# 23. 함수와 조건문

함수 내부에서 조건문을 사용할 수 있다.

``` c
int max_value(int a, int b)
{
    if (a > b)
    {
        return a;
    }

    return b;
}
```

``` mermaid
flowchart TD
    A["max_value(a, b)"] --> B{"a > b?"}
    B -->|"Yes"| C["return a"]
    B -->|"No"| D["return b"]
```

`Control_Flow.md`에서 학습한 조건문이 함수 내부 로직으로 사용된다.

------------------------------------------------------------------------

# 24. 함수와 반복문

함수 내부에서 반복문도 사용할 수 있다.

``` c
int sum_to_n(int n)
{
    int sum = 0;

    for (int i = 1; i <= n; i++)
    {
        sum += i;
    }

    return sum;
}
```

호출:

``` c
int result = sum_to_n(5);
```

결과:

``` text
15
```

``` mermaid
flowchart TD
    A["sum_to_n(5)"] --> B["sum = 0"]
    B --> C["for 반복"]
    C --> D["sum += i"]
    D --> C
    C -->|"종료"| E["return sum"]
```

------------------------------------------------------------------------

# 25. 함수와 배열로의 연결

이후 배열을 배우면 배열을 처리하는 함수를 작성하게 된다.

개념적인 예:

``` c
int sum_array(/* array */, int size)
{
    /* 배열 요소 반복 */
}
```

학습 흐름:

``` mermaid
flowchart LR
    A["Function"] --> B["Parameter"]
    B --> C["Array"]
    C --> D["Pointer"]
    D --> E["Buffer"]
```

임베디드에서는 이 구조가 다음으로 이어진다.

``` text
Array
→ Buffer
→ UART Buffer
→ Ring Buffer
→ Packet
```

------------------------------------------------------------------------

# 26. 재귀 함수

함수가 자기 자신을 호출하는 것을 **재귀(Recursion)**라고 한다.

개념적인 예:

``` c
int factorial(int n)
{
    if (n <= 1)
    {
        return 1;
    }

    return n * factorial(n - 1);
}
```

``` mermaid
flowchart TD
    A["factorial(4)"] --> B["4 * factorial(3)"]
    B --> C["3 * factorial(2)"]
    C --> D["2 * factorial(1)"]
    D --> E["1"]
```

현재 단계에서는 다음만 이해한다.

> **재귀 함수는 자기 자신을 호출하며 반드시 종료 조건이 필요하다.**

재귀는 이후 DFS, 트리 탐색 등의 알고리즘에서 다시 학습한다.

------------------------------------------------------------------------

## 26.1 재귀와 임베디드

재귀는 호출이 반복될수록 호출 스택을 사용한다.

메모리가 제한된 임베디드 환경에서는 재귀의 최대 깊이와 스택 사용량을
의식해야 한다.

``` mermaid
flowchart LR
    A["Recursion"] --> B["Function Call 증가"]
    B --> C["Stack 사용 증가"]
    C --> D["Embedded에서는 비용 고려"]
```

> \[!IMPORTANT\] 재귀가 무조건 금지된다는 의미는 아니다. 사용 환경과
> 최대 호출 깊이, 스택 제한을 고려해야 한다.

------------------------------------------------------------------------

# 27. 전역 변수와 함수

함수 밖에 변수를 선언할 수도 있다.

``` c
int system_state = 0;

void update_state(void)
{
    system_state = 1;
}
```

이런 변수는 여러 함수에서 접근할 수 있는 구조를 만들 수 있지만
프로그램의 의존 관계가 복잡해질 수 있다.

현재 단계에서는:

> **필요하지 않다면 함수 내부의 지역 변수를 우선 사용한다.**

전역 변수, `static`, 저장 기간, linkage 등은 이후 `Memory_Model.md`와
컴파일/링크 학습에서 자세히 다룬다.

------------------------------------------------------------------------

# 28. C와 C++ 함수의 공통점

기본적인 함수 구조는 C와 C++에서 매우 유사하다.

### C

``` c
int add(int a, int b)
{
    return a + b;
}
```

### C++

``` cpp
int add(int a, int b)
{
    return a + b;
}
```

따라서 다음 개념은 `00_Common`에서 함께 학습한다.

``` text
함수 호출
매개변수
인자
반환값
지역 변수
스코프
선언
정의
```

------------------------------------------------------------------------

# 29. C++에서 이후 확장되는 함수 개념

C++에서는 이후 다음 개념으로 확장된다.

``` text
Function Overloading
Default Argument
Reference Parameter
Member Function
Constructor
Lambda
Template Function
constexpr Function
```

``` mermaid
flowchart TD
    A["Common Function"] --> B["C"]
    A --> C["C++"]

    B --> D["Pointer Parameter"]
    B --> E["C API"]

    C --> F["Overloading"]
    C --> G["Reference"]
    C --> H["Member Function"]
    C --> I["Template"]
```

현재는 기본 함수 개념을 확실하게 익힌다.

------------------------------------------------------------------------

# 30. 임베디드에서 함수

펌웨어는 기능별로 함수를 분리하는 경우가 많다.

예:

``` c
void gpio_init(void);
void uart_init(void);
void timer_init(void);

int read_sensor(void);
void process_sensor(int value);
void send_data(int value);
```

전체 구조를 개념적으로 보면:

``` mermaid
flowchart TD
    A["main()"] --> B["gpio_init()"]
    A --> C["uart_init()"]
    A --> D["timer_init()"]

    A --> E["Main Loop"]

    E --> F["read_sensor()"]
    F --> G["process_sensor()"]
    G --> H["send_data()"]
    H --> E
```

함수를 잘 분리하면 하드웨어 초기화, 데이터 처리, 통신 로직 등을 구분하기
쉬워진다.

------------------------------------------------------------------------

# 31. 드라이버 함수로의 연결

임베디드 학습이 진행되면 다음과 같은 형태를 접하게 된다.

``` c
void uart_init(void);
void uart_send_byte(unsigned char data);
int uart_receive_byte(void);
```

또는:

``` c
void gpio_set(unsigned int pin);
void gpio_clear(unsigned int pin);
```

``` mermaid
flowchart LR
    A["Function"] --> B["Driver API"]
    B --> C["GPIO"]
    B --> D["UART"]
    B --> E["SPI"]
    B --> F["I2C"]
    B --> G["Timer"]
```

즉 함수는 단순히 계산을 재사용하는 문법을 넘어 **하드웨어 기능을
추상화하는 인터페이스**로 발전한다.

------------------------------------------------------------------------

# 32. 헤더 파일과 함수 선언

프로그램이 여러 파일로 나뉘면 함수 선언을 헤더 파일에 작성하는 구조를
사용하게 된다.

예:

``` text
project/
├── main.c
├── calculator.c
└── calculator.h
```

`calculator.h`:

``` c
int add(int a, int b);
int subtract(int a, int b);
```

`calculator.c`:

``` c
int add(int a, int b)
{
    return a + b;
}

int subtract(int a, int b)
{
    return a - b;
}
```

`main.c`:

``` c
#include "calculator.h"

int main(void)
{
    int result = add(10, 20);

    return 0;
}
```

``` mermaid
flowchart LR
    A["calculator.h"] --> B["함수 선언"]
    C["calculator.c"] --> D["함수 정의"]
    E["main.c"] --> A
    E --> F["함수 호출"]
    F --> C
```

이 내용은 이후 **Header / Source 분리와 Compile / Link** 단계에서 자세히
다룬다.

------------------------------------------------------------------------

# 33. 자주 하는 실수

## 33.1 반환 타입과 실제 반환값 불일치

``` c
int get_value(void)
{
    return 3.14;
}
```

타입 변환이 발생할 수 있으므로 함수의 의도와 반환 타입을 확인한다.

------------------------------------------------------------------------

## 33.2 값을 반환해야 하는 함수에서 `return` 누락

``` c
int add(int a, int b)
{
    int result = a + b;

    /* return 누락 */
}
```

수정:

``` c
int add(int a, int b)
{
    return a + b;
}
```

------------------------------------------------------------------------

## 33.3 지역 변수를 함수 밖에서 사용

``` c
void test(void)
{
    int value = 10;
}

int main(void)
{
    /* value 사용 불가 */
    return 0;
}
```

스코프를 확인한다.

------------------------------------------------------------------------

## 33.4 매개변수 변경이 원본을 변경한다고 생각함

``` c
void change(int value)
{
    value = 100;
}
```

기본적인 값 전달에서는 호출자의 원본 변수가 직접 변경되지 않는다.

------------------------------------------------------------------------

## 33.5 함수 선언 없이 사용

함수의 선언과 정의 위치를 확인한다.

``` c
int add(int a, int b);
```

필요하다면 함수 원형을 먼저 제공한다.

------------------------------------------------------------------------

## 33.6 재귀 종료 조건 누락

``` c
void recursive(void)
{
    recursive();
}
```

종료 조건이 없으면 계속 함수 호출이 발생한다.

------------------------------------------------------------------------

## 33.7 너무 많은 작업을 하나의 함수에 넣음

``` text
main()
└─ 모든 로직
```

프로그램이 커질수록 기능별로 분리한다.

------------------------------------------------------------------------

# 34. 실습

## 실습 1 --- 두 수의 합

다음 함수를 작성한다.

``` c
int add(int a, int b);
```

예:

``` text
add(10, 20) → 30
```

------------------------------------------------------------------------

## 실습 2 --- 큰 값 반환

``` c
int max_value(int a, int b);
```

예:

``` text
max_value(10, 20) → 20
```

`if`를 사용한다.

------------------------------------------------------------------------

## 실습 3 --- 절댓값

``` c
int absolute(int value);
```

예:

``` text
absolute(-10) → 10
absolute(10)  → 10
```

------------------------------------------------------------------------

## 실습 4 --- 짝수 판별

다음 형태의 함수를 작성한다.

``` c
int is_even(int number);
```

짝수이면 `1`, 아니면 `0`을 반환하도록 구현해본다.

------------------------------------------------------------------------

## 실습 5 --- 1부터 N까지 합

``` c
int sum_to_n(int n);
```

예:

``` text
sum_to_n(5) → 15
```

`for`를 사용한다.

------------------------------------------------------------------------

## 실습 6 --- 구구단 출력

``` c
void print_gugudan(int dan);
```

예:

``` text
print_gugudan(2)
```

출력:

``` text
2 x 1 = 2
2 x 2 = 4
...
2 x 9 = 18
```

------------------------------------------------------------------------

## 실습 7 --- 함수 조합

다음 두 함수를 작성한다.

``` c
int square(int value);
int sum_of_squares(int a, int b);
```

`sum_of_squares()` 내부에서 `square()`를 호출한다.

예:

``` text
sum_of_squares(3, 4) → 25
```

------------------------------------------------------------------------

## 실습 8 --- C와 C++에서 동일 함수 작성

### C

``` c
#include <stdio.h>

int add(int a, int b)
{
    return a + b;
}

int main(void)
{
    int result = add(10, 20);

    printf("%d\n", result);

    return 0;
}
```

### C++

``` cpp
#include <iostream>

int add(int a, int b)
{
    return a + b;
}

int main()
{
    int result = add(10, 20);

    std::cout << result << '\n';

    return 0;
}
```

함수 자체보다 입출력 방식이 어떻게 다른지 비교한다.

------------------------------------------------------------------------

# 35. 코드 추적 연습

다음 코드를 실행하지 않고 직접 추적한다.

``` c
int multiply(int a, int b)
{
    int result = a * b;

    return result;
}

int main(void)
{
    int x = 3;
    int y = 4;

    int value = multiply(x, y);

    return 0;
}
```

다음 표를 채운다.

  실행 단계                 `x`   `y`   `a`   `b`   `result`   `value`
  ----------------------- ----- ----- ----- ----- ---------- ---------
  `main` 변수 초기화          3     4    \-    \-         \-        \-
  `multiply(x, y)` 호출                                      
  `a * b`                                                    
  `return`                                                   
  `main` 복귀                            \-    \-         \- 

------------------------------------------------------------------------

# 36. 값 전달 추적

``` c
void change(int value)
{
    value = 100;
}

int main(void)
{
    int number = 10;

    change(number);

    return 0;
}
```

다음 질문에 답한다.

1.  `change()`가 호출될 때 `value`의 값은?
2.  함수 내부에서 `value = 100` 이후 `value`는?
3.  함수 종료 후 `number`는?
4.  왜 그렇게 되는가?

------------------------------------------------------------------------

# 37. 반드시 설명할 수 있어야 하는 것

-   [ ] 함수가 무엇인지 설명할 수 있다.
-   [ ] 함수를 사용하는 이유를 설명할 수 있다.
-   [ ] 함수 이름, 반환 타입, 매개변수, 본문을 구분할 수 있다.
-   [ ] 함수 호출이 무엇인지 설명할 수 있다.
-   [ ] 매개변수와 인자의 차이를 설명할 수 있다.
-   [ ] 반환값과 반환 타입을 설명할 수 있다.
-   [ ] `void` 함수의 의미를 설명할 수 있다.
-   [ ] `return`이 함수 실행을 종료한다는 것을 설명할 수 있다.
-   [ ] 지역 변수가 무엇인지 설명할 수 있다.
-   [ ] 스코프가 무엇인지 설명할 수 있다.
-   [ ] 기본적인 값 전달을 설명할 수 있다.
-   [ ] 매개변수를 변경해도 호출자의 변수가 자동으로 변경되지 않는
    이유를 설명할 수 있다.
-   [ ] 함수 선언과 정의를 구분할 수 있다.
-   [ ] 함수 원형이 무엇인지 설명할 수 있다.
-   [ ] `main()`도 함수라는 것을 설명할 수 있다.
-   [ ] 함수가 다른 함수를 호출할 수 있음을 이해한다.
-   [ ] 재귀 함수의 기본 개념과 종료 조건의 필요성을 설명할 수 있다.
-   [ ] 함수 호출과 Stack의 관계를 대략 설명할 수 있다.
-   [ ] 임베디드에서 함수가 드라이버 API로 발전하는 이유를 설명할 수
    있다.

------------------------------------------------------------------------

# 38. 복습 문제

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q1. 다음 함수의 반환 타입은?`</strong>`{=html}
```{=html}
</summary>
```
``` c
int add(int a, int b)
{
    return a + b;
}
```

**정답:** `int`

```{=html}
</details>
```

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q2. 다음 함수의 매개변수는?`</strong>`{=html}
```{=html}
</summary>
```
``` c
double divide(double a, double b)
{
    return a / b;
}
```

**정답:** `a`, `b`

둘 다 `double` 타입의 매개변수다.

```{=html}
</details>
```

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q3. 다음 호출에서 인자는?`</strong>`{=html}
```{=html}
</summary>
```
``` c
add(10, 20);
```

**정답:** `10`, `20`

```{=html}
</details>
```

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q4. void의 의미는?`</strong>`{=html}
```{=html}
</summary>
```
``` c
void print_message(void)
{
    printf("Hello\n");
}
```

**정답:** 이 함수는 값을 반환하지 않는다.

```{=html}
</details>
```

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q5. 다음 코드 실행 후 number는?`</strong>`{=html}
```{=html}
</summary>
```
``` c
void change(int value)
{
    value = 100;
}

int main(void)
{
    int number = 10;

    change(number);

    return 0;
}
```

**정답:** `10`

`number`의 값이 매개변수 `value`로 전달되며, 함수 내부의 `value`를
변경해도 `number` 자체를 직접 변경하는 것은 아니다.

```{=html}
</details>
```

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q6. 선언과 정의의 차이는?`</strong>`{=html}
```{=html}
</summary>
```
**정답:**

``` c
int add(int a, int b);
```

는 함수의 선언이다.

``` c
int add(int a, int b)
{
    return a + b;
}
```

는 함수의 정의다.

```{=html}
</details>
```

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q7. 다음 함수의 결과는?`</strong>`{=html}
```{=html}
</summary>
```
``` c
int square(int value)
{
    return value * value;
}

int result = square(5);
```

**정답:** `25`

```{=html}
</details>
```

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q8. 다음 함수의 문제는?`</strong>`{=html}
```{=html}
</summary>
```
``` c
int add(int a, int b)
{
    int result = a + b;
}
```

**정답:** 값을 반환해야 하는 `int` 반환 함수인데 적절한 `return`이 없다.

```{=html}
</details>
```

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q9. 재귀 함수에서 중요한 것은?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 재귀 호출이 끝날 수 있도록 종료 조건이 필요하다.

```{=html}
</details>
```

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q10. 함수와 임베디드 드라이버의
관계는?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** GPIO, UART, SPI 등의 하드웨어 제어 동작을 함수 인터페이스로
분리하면 호출하는 코드가 하드웨어 제어의 세부 구현을 직접 반복하지 않고
기능 단위로 사용할 수 있다.

```{=html}
</details>
```

------------------------------------------------------------------------

# 39. Bug Log에 기록할 항목

함수 관련 실수는 `99_Bug_Log`에 다음 형식으로 기록한다.

```` markdown
## 함수의 값 전달 오해

### 코드

```c
void change(int value)
{
    value = 100;
}

int main(void)
{
    int number = 10;

    change(number);

    return 0;
}
```

### 잘못 예상한 결과

`number == 100`

### 실제 결과

`number == 10`

### 원인

`number`의 값이 함수의 매개변수 `value`로 전달되었다.
함수 내부에서 `value`를 변경해도 호출자의 `number` 자체를 직접 변경하지 않는다.

### 이후 연결

원본 객체를 함수에서 변경하는 방법은 Pointer 학습에서 다시 다룬다.

### 재발 방지 규칙

함수 호출을 볼 때 다음을 확인한다.

1. 무엇이 인자로 전달되는가?
2. 매개변수 타입은 무엇인가?
3. 함수 내부에서 변경되는 객체는 무엇인가?
4. 반환값은 무엇인가?
````

추천 Bug Log 항목:

``` text
return 누락
반환 타입 불일치
Parameter / Argument 혼동
지역 변수 Scope 오류
값 전달 오해
함수 선언 누락
함수 이름 오타
재귀 종료 조건 누락
전역 변수 과도한 사용
```

------------------------------------------------------------------------

# 40. 함수를 볼 때 사용하는 사고 방식

함수를 만나면 다음 순서로 분석한다.

``` mermaid
flowchart TD
    A["함수 이름은?"] --> B["입력은 무엇인가?"]
    B --> C["매개변수 타입은?"]
    C --> D["함수 내부에서 무엇을 하는가?"]
    D --> E["지역 변수는 무엇인가?"]
    E --> F["무엇을 반환하는가?"]
    F --> G["호출한 곳에서 반환값을 어떻게 사용하는가?"]
```

체크리스트:

-   [ ] 함수 이름 확인
-   [ ] 반환 타입 확인
-   [ ] 매개변수 확인
-   [ ] 인자 확인
-   [ ] 지역 변수 확인
-   [ ] 조건/반복 흐름 확인
-   [ ] `return` 확인
-   [ ] 호출 후 결과 확인

------------------------------------------------------------------------

# 41. 지금까지 학습한 개념 통합

현재까지의 학습 내용은 다음처럼 연결된다.

``` mermaid
flowchart LR
    A["Data Type"] --> B["Operator"]
    B --> C["Control Flow"]
    C --> D["Function"]
```

각 단계의 역할:

  개념           핵심 질문
  -------------- -------------------------------------
  Data Type      어떤 데이터를 다루는가?
  Operator       데이터를 어떻게 계산/비교하는가?
  Control Flow   어떤 조건과 순서로 실행하는가?
  Function       작업을 어떻게 기능 단위로 나누는가?

이 네 가지를 결합하면 기본적인 프로그램을 작성할 수 있다.

------------------------------------------------------------------------

# 42. 통합 예제

두 정수를 입력받아 큰 값을 함수로 구한다.

### C

``` c
#include <stdio.h>

int max_value(int a, int b)
{
    if (a > b)
    {
        return a;
    }

    return b;
}

int main(void)
{
    int a;
    int b;

    scanf("%d %d", &a, &b);

    int result = max_value(a, b);

    printf("max = %d\n", result);

    return 0;
}
```

사용된 개념:

``` mermaid
flowchart TD
    A["int"] --> B["Data Type"]
    C[">"] --> D["Operator"]
    E["if"] --> F["Control Flow"]
    G["max_value()"] --> H["Function"]
```

이 예제를 설명할 수 있다면 지금까지의 공통 기초 개념이 서로 어떻게
연결되는지 이해하기 시작한 것이다.

------------------------------------------------------------------------

# 43. 알고리즘으로의 연결

코딩 테스트에서 함수는 알고리즘을 기능 단위로 나누는 데 사용한다.

예:

``` text
solve()
dfs()
bfs()
binary_search()
is_prime()
calculate_sum()
```

``` mermaid
flowchart TD
    A["Algorithm"] --> B["Function"]
    B --> C["Input"]
    B --> D["Processing"]
    B --> E["Return"]

    D --> F["Loop"]
    D --> G["Condition"]
```

이후 배열과 STL을 배우면 함수의 입력으로 다양한 데이터 구조를 전달하게
된다.

------------------------------------------------------------------------

# 44. 임베디드로의 연결

``` mermaid
flowchart TD
    A["Function"] --> B["Initialization"]
    A --> C["Input"]
    A --> D["Processing"]
    A --> E["Output"]

    B --> F["gpio_init()"]
    B --> G["uart_init()"]

    C --> H["adc_read()"]
    C --> I["uart_receive()"]

    D --> J["process_data()"]

    E --> K["gpio_write()"]
    E --> L["uart_send()"]
```

펌웨어 구조의 개념적인 모습:

``` c
int main(void)
{
    system_init();

    while (1)
    {
        int sensor = read_sensor();

        int result = process_sensor(sensor);

        update_output(result);
    }
}
```

현재까지 배운 개념만으로도 다음 구조를 읽을 수 있다.

``` text
Function
+
Control Flow
+
Data Type
+
Operator
```

------------------------------------------------------------------------

# 45. 핵심 요약

``` mermaid
mindmap
  root((Function))
    구조
      Return Type
      Function Name
      Parameter
      Body
      Return
    호출
      Argument
      Return Value
    변수
      Local Variable
      Scope
      Shadowing
    설계
      Reuse
      Modularity
      Single Responsibility
    Memory
      Call Stack
      Stack Frame
    C
      Pass by Value
      Pointer로 확장
    C++
      Reference로 확장
      Overloading
      Member Function
    Embedded
      Driver API
      Initialization
      Sensor Read
      UART
      GPIO
```

## 최종적으로 기억할 다섯 문장

> **1. 함수는 특정 작업을 하나의 이름으로 묶은 코드 단위다.**

> **2. 함수는 `입력(인자) → 매개변수 → 처리 → 반환값`의 흐름으로
> 분석한다.**

> **3. 지역 변수는 스코프를 가지며 기본적인 값 전달에서는 매개변수를
> 변경해도 호출자의 원본 변수가 자동으로 변경되지 않는다.**

> **4. 함수 호출은 이후 Stack, Pointer, Reference, Header/Source 분리와
> 연결된다.**

> **5. 임베디드에서는 함수가 `gpio_init()`, `uart_send()`,
> `adc_read()`와 같은 드라이버 인터페이스로 발전한다.**

------------------------------------------------------------------------

# 46. 현재 공통 기초 진행 상태

``` mermaid
flowchart LR
    A["Data_Type.md<br/>완료"] --> B["Operator.md<br/>완료"]
    B --> C["Control_Flow.md<br/>완료"]
    C --> D["Function.md<br/>현재"]
    D --> E["Array / String"]
    E --> F["Pointer"]
    F --> G["Memory Model"]
```

현재까지:

``` text
00_Common/
├── Data_Type.md       ✓
├── Operator.md        ✓
├── Control_Flow.md    ✓
└── Function.md        ← 현재
```

------------------------------------------------------------------------

# 47. 다음 학습

함수까지 완료하면 공통 기본 문법을 실제 데이터 집합에 적용하기 위해
**배열과 문자열**로 넘어간다.

``` mermaid
flowchart LR
    A["Function"] --> B["Array"]
    B --> C["String"]
    C --> D["Pointer"]
    D --> E["Memory Model"]
```

배열부터는 C의 메모리 개념과 포인터 학습이 본격적으로 연결되기 시작한다.

**다음 문서:** `01_C/Array_String.md`
