# Data Type

> **학습 목표**
>
> C와 C++에서 사용하는 기본 자료형을 이해하고, 단순히 자료형 이름을
> 암기하는 것이 아니라 **값 → 자료형 → 메모리 → 비트 → 포인터 → 임베디드
> 하드웨어**로 이어지는 관계를 이해한다.
>
> 이 노트는 `C_Cpp_Study/00_Common/Data_Type.md`에 위치하며, 이후
> `Memory_Model`, `Pointer`, `Bit_Operation`, `Embedded_C` 학습의
> 기반으로 사용한다.

------------------------------------------------------------------------

## 1. 자료형이란?

### 한 문장 정의

**자료형(Data Type)은 데이터를 어떤 종류의 값으로 해석하고 다룰 것인지를
나타내는 타입이다.**

``` c
int age = 20;
```

위 코드를 구성 요소로 나누면 다음과 같다.

  코드    의미
  ------- ----------------------
  `int`   자료형
  `age`   변수 이름
  `20`    초기값
  `=`     초기화에 사용된 기호

``` mermaid
flowchart LR
    A["int"] --> B["자료형"]
    C["age"] --> D["변수 이름"]
    E["20"] --> F["초기값"]
```

즉, `int age = 20;`은 **`age`라는 이름의 `int` 타입 변수를 만들고
`20`으로 초기화한다**는 의미다.

------------------------------------------------------------------------

## 2. 왜 자료형이 필요한가?

프로그램에서는 서로 다른 종류의 데이터를 처리한다.

``` text
나이      → 20
온도      → 23.5
등급      → 'A'
거리      → 125.75
```

C/C++에서는 데이터의 성격에 맞게 자료형을 지정한다.

``` c
int age = 20;
float temperature = 23.5f;
char grade = 'A';
double distance = 125.75;
```

자료형을 공부할 때는 단순히 `int = 정수`라고 외우는 것보다 다음 세 가지
질문을 같이 생각한다.

``` mermaid
mindmap
  root((Data Type))
    어떤 값인가?
      정수
      실수
      문자
    메모리
      저장 공간
      sizeof
    어떻게 해석하는가?
      signed
      unsigned
      integer
      floating point
```

> **핵심 관점:** 값만 보지 말고 항상 타입을 같이 본다.

예를 들어 `10`은 정수 리터럴이고 `10.0`은 실수 리터럴이다. 이 차이는
실제 연산 결과에도 영향을 준다.

------------------------------------------------------------------------

## 3. C/C++ 기본 자료형 전체 그림

``` mermaid
flowchart TD
    A["기본 자료형"] --> B["정수형"]
    A --> C["실수형"]
    A --> D["void"]

    B --> B1["char"]
    B --> B2["short"]
    B --> B3["int"]
    B --> B4["long"]
    B --> B5["long long"]

    B1 --> S["signed / unsigned"]
    B2 --> S
    B3 --> S
    B4 --> S
    B5 --> S

    C --> C1["float"]
    C --> C2["double"]
    C --> C3["long double"]
```

첫 학습에서는 다음 네 가지를 우선 사용한다.

  자료형     기본 용도                    예
  ---------- ---------------------------- -------------
  `char`     문자 또는 작은 정수 데이터   `'A'`
  `int`      정수                         `10`, `-20`
  `float`    실수                         `3.14f`
  `double`   실수                         `3.14`

``` c
char grade = 'A';
int age = 20;
float temperature = 23.5f;
double pi = 3.141592;
```

------------------------------------------------------------------------

## 4. 정수형

정수형(Integer Type)은 소수점이 없는 정수 값을 표현하는 데 사용한다.

``` c
int age = 20;
int temperature = -5;
int count = 100;
```

대표적인 정수형은 다음과 같다.

``` text
char
short
int
long
long long
```

### 4.1 `int`

가장 기본적으로 많이 사용하는 정수형이다.

``` c
int number = 10;
int temperature = -20;
int count = 1000;
```

현재 단계에서는 일반적인 정수 데이터가 필요하면 우선 `int`를 사용한다고
생각하면 된다.

### 4.2 `short`, `long`, `long long`

``` c
short a = 10;
int b = 20;
long c = 30;
long long d = 40;
```

> `short`, `int`, `long`, `long long`의 정확한 바이트 크기를 이름만 보고
> 무조건 단정해서는 안 된다. 필요한 경우 `sizeof`로 현재 환경에서
> 확인한다.

``` c
sizeof(short);
sizeof(int);
sizeof(long);
sizeof(long long);
```

------------------------------------------------------------------------

## 5. `signed`와 `unsigned`

``` c
signed int temperature = -10;
unsigned int count = 100;
```

  종류         표현
  ------------ ---------------
  `signed`     음수, 0, 양수
  `unsigned`   0 이상의 정수

일반적인 `int`는 보통 다음처럼 작성한다.

``` c
int number = -10;
```

### 5.1 `unsigned` 사용 시 주의

``` c
int a = -1;
unsigned int b = 1;
```

signed와 unsigned가 섞인 연산에서는 숫자 관계만 보고 결과를 판단하면 안
된다. 연산 과정에서 타입 변환이 발생할 수 있기 때문이다.

> \[!WARNING\] **signed와 unsigned가 섞인 연산에서는 값뿐 아니라
> 피연산자의 자료형을 반드시 확인한다.**

정확한 정수 승격과 변환 규칙은 이후 심화 과정에서 다룬다.

------------------------------------------------------------------------

## 6. 실수형

대표적인 실수형은 다음과 같다.

``` text
float
double
long double
```

``` c
float temperature = 23.5f;
double pi = 3.141592;
```

### 6.1 실수 리터럴

``` c
3.14   // double 리터럴
3.14f  // float 리터럴
```

따라서 다음처럼 작성할 수 있다.

``` c
float a = 3.14f;
double b = 3.14;
```

------------------------------------------------------------------------

## 7. `char`

`char`는 문자 데이터를 다룰 때 처음 접하게 되는 자료형이다.

``` c
char grade = 'A';
char command = 'R';
```

### 7.1 문자와 문자열은 다르다

``` c
'A'   // 문자
"A"   // 문자열
```

  표현    의미
  ------- --------
  `'A'`   문자
  `"A"`   문자열

> \[!IMPORTANT\] `'A'`와 `"A"`를 같은 것으로 생각하지 않는다.

문자열의 실제 메모리 구조는 이후 `Array_String.md`에서 배열과 함께
자세히 다룬다.

------------------------------------------------------------------------

## 8. 선언, 초기화, 대입

### 8.1 선언

``` c
int number;
```

`number`라는 이름의 `int` 변수를 선언한다.

### 8.2 초기화

``` c
int number = 10;
```

변수를 만들면서 처음 값을 지정한다.

``` mermaid
flowchart LR
    A["int number"] --> B["변수 생성"]
    B --> C["= 10"]
    C --> D["초기값 설정"]
```

### 8.3 대입

이미 존재하는 변수에 새로운 값을 넣는다.

``` c
int number = 10;
number = 20;
```

  코드            의미
  --------------- ---------------
  `int a;`        선언
  `int b = 10;`   선언 + 초기화
  `b = 20;`       대입

------------------------------------------------------------------------

## 9. 변수의 값 변경

``` c
int count = 10;

count = 20;
count = 30;
```

최종적으로 `count`의 값은 `30`이다.

다음 코드도 가능하다.

``` c
int count = 10;
count = count + 1;
```

``` mermaid
flowchart LR
    A["count = 10"] --> B["count 값 읽기"]
    B --> C["10 + 1"]
    C --> D["11"]
    D --> E["count에 11 대입"]
```

> \[!NOTE\] 프로그래밍의 `=`는 기본적으로 **대입**을 나타낸다. 같은지
> 비교할 때는 `==`를 사용한다.

  연산자   의미
  -------- -------------
  `=`      대입
  `==`     같은지 비교

------------------------------------------------------------------------

## 10. `sizeof`

`sizeof`를 사용하면 타입 또는 객체가 사용하는 크기를 **byte 단위**로
확인할 수 있다.

``` c
sizeof(char);
sizeof(int);
sizeof(float);
sizeof(double);
```

직접 확인하는 예제:

``` c
#include <stdio.h>

int main(void)
{
    printf("char        : %zu byte\n", sizeof(char));
    printf("short       : %zu byte\n", sizeof(short));
    printf("int         : %zu byte\n", sizeof(int));
    printf("long        : %zu byte\n", sizeof(long));
    printf("long long   : %zu byte\n", sizeof(long long));
    printf("float       : %zu byte\n", sizeof(float));
    printf("double      : %zu byte\n", sizeof(double));

    return 0;
}
```

### 10.1 변수에도 사용할 수 있다

``` c
int number = 10;
double pi = 3.14;

sizeof(number);
sizeof(pi);
```

### 10.2 `sizeof`와 메모리

``` mermaid
flowchart LR
    A["자료형"] --> B["sizeof"]
    B --> C["저장 공간의 크기"]
    C --> D["메모리"]
```

> \[!TIP\] 자료형의 크기를 무조건 숫자로 외우기보다 `sizeof`로 직접
> 확인하고, 왜 크기가 중요한지를 이해한다.

------------------------------------------------------------------------

## 11. 정수 나눗셈

자료형은 연산 결과에도 영향을 준다.

``` c
int a = 10;
int b = 3;

int result = a / b;
```

결과는 `3`이다. `a`와 `b`가 정수형이기 때문에 정수 나눗셈이 수행된다.

### 11.1 실수 나눗셈

``` c
double a = 10.0;
double b = 3.0;

double result = a / b;
```

이번에는 실수 연산이 이루어진다.

``` mermaid
flowchart TD
    A["나눗셈"] --> B{"피연산자의 타입"}
    B -->|"int / int"| C["정수 나눗셈"]
    B -->|"실수형 포함"| D["실수 연산"]
    C --> E["10 / 3 → 3"]
    D --> F["10.0 / 3.0 → 약 3.333..."]
```

> **연산 결과를 예상할 때 값만 보지 말고 자료형을 확인한다.**

------------------------------------------------------------------------

## 12. 형변환

서로 다른 자료형 사이에서 값을 변환하는 것을 **형변환(Type
Conversion)**이라고 한다.

``` c
int a = 10;
double b = a;
```

필요한 경우 프로그래머가 명시적으로 타입을 변환할 수도 있다.

``` c
int a = 10;
int b = 3;

double result = (double)a / b;
```

비교:

``` c
double result1 = a / b;
double result2 = (double)a / b;
```

``` mermaid
flowchart TD
    A["a = 10, b = 3"] --> B["a / b"]
    A --> C["(double)a / b"]
    B --> D["int / int"]
    D --> E["정수 나눗셈"]
    E --> F["3"]
    C --> G["double / int"]
    G --> H["실수 연산"]
    H --> I["약 3.333..."]
```

> \[!NOTE\] C++에서는 이후 `static_cast<double>(a)`와 같은 C++ 스타일
> 형변환도 배운다. 현재 `00_Common` 단계에서는 형변환이 왜 필요한지를
> 이해하는 것이 우선이다.

------------------------------------------------------------------------

## 13. C와 C++의 공통점

기본적인 자료형은 C와 C++에서 상당 부분 비슷하게 사용할 수 있다.

### C

``` c
int age = 20;
char grade = 'A';
float temperature = 23.5f;
double pi = 3.14;
```

### C++

``` cpp
int age = 20;
char grade = 'A';
float temperature = 23.5f;
double pi = 3.14;
```

``` mermaid
flowchart TD
    A["공통 자료형 개념"] --> B["C"]
    A --> C["C++"]
    B --> D["C에서 활용"]
    C --> E["C++에서 활용"]
    D --> F["Pointer / Memory"]
    E --> G["Reference / Class / RAII"]
```

따라서 기본 자료형은 `01_C`, `02_CPP`에 중복해서 작성하기보다
`00_Common/Data_Type.md`에 정리한다.

------------------------------------------------------------------------

## 14. C와 C++ 입출력 비교

### C

``` c
#include <stdio.h>

int main(void)
{
    int age = 20;
    printf("age = %d\n", age);
    return 0;
}
```

### C++

``` cpp
#include <iostream>

int main()
{
    int age = 20;
    std::cout << "age = " << age << '\n';
    return 0;
}
```

  기능        C            C++
  ----------- ------------ -------------
  기본 출력   `printf()`   `std::cout`
  기본 입력   `scanf()`    `std::cin`

세부적인 차이는 이후 `03_Comparison/C_vs_CPP_IO.md`에서 다룬다.

------------------------------------------------------------------------

## 15. 자료형과 메모리

``` c
int value = 10;
```

처음에는 단순히 `value = 10`으로 생각하기 쉽지만, C/C++를 제대로
이해하려면 관점을 확장해야 한다.

``` mermaid
flowchart LR
    A["value"] --> B["자료형: int"]
    A --> C["값: 10"]
    A --> D["메모리 공간"]
    D --> E["메모리 주소"]
```

개념적인 메모리 모습:

``` text
Memory

Address       Data
─────────     ────────
0x1000        ...
0x1001        ...
0x1002        ...
0x1003        ...
0x1004        ...
...
```

이후 메모리의 위치, 즉 **주소를 직접 다루는 개념**으로 넘어가면 포인터가
등장한다.

``` mermaid
flowchart LR
    A["Data Type"] --> B["Variable"]
    B --> C["Memory"]
    C --> D["Address"]
    D --> E["Pointer"]
```

------------------------------------------------------------------------

## 16. 임베디드에서 자료형이 중요한 이유

임베디드 시스템에서는 다음과 같은 데이터를 다룬다.

``` text
MCU Register
Sensor Data
ADC Value
UART Data
CAN Data
Protocol Packet
Memory Buffer
GPIO State
```

이런 데이터는 **몇 비트의 데이터인지**가 중요한 경우가 많다.

따라서 이후 다음 타입들을 자주 접한다.

``` c
uint8_t
uint16_t
uint32_t

int8_t
int16_t
int32_t
```

이들은 `<stdint.h>`에서 제공되는 fixed-width integer type 계열이다.

``` c
#include <stdint.h>

uint8_t uart_data;
uint16_t adc_value;
uint32_t register_value;
```

### 16.1 임베디드로 이어지는 관계

``` mermaid
flowchart TD
    A["Data Type"] --> B["데이터 크기"]
    B --> C["Byte"]
    C --> D["Bit"]
    D --> E["Fixed-width Integer"]
    E --> F["uint8_t"]
    E --> G["uint16_t"]
    E --> H["uint32_t"]
    F --> I["통신 데이터"]
    G --> J["ADC / Sensor Data"]
    H --> K["Hardware Register"]
    I --> L["Embedded"]
    J --> L
    K --> L
```

> \[!IMPORTANT\] 현재 단계에서 `uint8_t`, `uint16_t`, `uint32_t`를
> 완벽하게 공부할 필요는 없다. 지금은 **자료형 → 크기 → 비트 →
> 하드웨어**가 연결된다는 점을 이해한다.

------------------------------------------------------------------------

## 17. 앞으로 확장될 개념

``` mermaid
flowchart TD
    A["Data_Type.md"] --> B["Operator.md"]
    B --> C["Control_Flow.md"]
    C --> D["Function.md"]
    D --> E["Array_String.md"]
    E --> F["Pointer.md"]
    F --> G["Memory_Model.md"]
    G --> H["Struct"]
    G --> I["Dynamic Memory"]
    G --> J["C++ Reference"]
    H --> K["Embedded Register"]
    J --> L["Class / RAII"]
    K --> M["Driver"]
    M --> N["UART / I2C / SPI"]
```

특히 임베디드 학습에서는 다음 흐름을 기억한다.

``` mermaid
flowchart LR
    A["자료형"] --> B["sizeof"]
    B --> C["메모리"]
    C --> D["주소"]
    D --> E["포인터"]
    E --> F["비트"]
    F --> G["레지스터"]
    G --> H["Peripheral"]
    H --> I["Driver"]
```

------------------------------------------------------------------------

## 18. 자주 하는 실수

### 18.1 초기화하지 않은 지역 변수 사용

``` c
int number;
printf("%d\n", number);
```

초기화되지 않은 지역 자동 변수의 값을 읽지 않는다.

``` c
int number = 0;
```

> \[!WARNING\] **"변수를 선언하면 자동으로 0이 된다"라고 생각하지
> 않는다.** 변수의 저장 기간과 초기화 규칙에 따라 다르며, 해당 내용은
> 이후 `Memory_Model.md`에서 자세히 다룬다.

### 18.2 정수 나눗셈 실수

``` c
int a = 10;
int b = 3;

double result = a / b;
```

`result`가 `double`이라고 해서 `a / b` 자체가 실수 나눗셈으로 바뀌는
것은 아니다.

``` c
double result = (double)a / b;
```

### 18.3 문자와 문자열 혼동

``` c
'A'  // 문자
"A"  // 문자열
```

### 18.4 `=`와 `==` 혼동

  연산자   의미
  -------- ------
  `=`      대입
  `==`     비교

### 18.5 값만 보고 타입을 보지 않음

``` c
10 / 3
10.0 / 3.0
```

``` mermaid
flowchart LR
    A["Expression"] --> B["Value"]
    A --> C["Type"]
```

> **값 + 자료형을 함께 본다.**

------------------------------------------------------------------------

## 19. 실습

### 실습 1 --- 기본 자료형 선언

  데이터   사용할 자료형
  -------- ---------------
  나이     `int`
  등급     `char`
  온도     `float`
  원주율   `double`

``` c
int age = 20;
char grade = 'A';
float temperature = 25.5f;
double pi = 3.141592;
```

### 실습 2 --- `sizeof`

다음 자료형의 크기를 직접 출력한다.

``` text
char
short
int
long
long long
float
double
```

  자료형          내 환경의 `sizeof` 결과
  ------------- -------------------------
  `char`        
  `short`       
  `int`         
  `long`        
  `long long`   
  `float`       
  `double`      

> \[!TIP\] 검색해서 채우지 말고 **직접 프로그램을 실행해서 결과를
> 기록한다.**

### 실습 3 --- 정수 연산

실행하기 전에 결과를 먼저 예상한다.

``` c
int a = 10;
int b = 3;

printf("%d\n", a + b);
printf("%d\n", a - b);
printf("%d\n", a * b);
printf("%d\n", a / b);
printf("%d\n", a % b);
```

### 실습 4 --- 정수 나눗셈과 실수 나눗셈

#### Case A

``` c
int a = 10;
int b = 3;

double result = a / b;
```

#### Case B

``` c
int a = 10;
int b = 3;

double result = (double)a / b;
```

**질문:** 두 코드에서 `result`가 달라지는 이유는 무엇인가?

### 실습 5 --- C/C++ 비교

#### C

``` c
#include <stdio.h>

int main(void)
{
    int age = 20;
    char grade = 'A';
    double temperature = 23.5;

    printf("age = %d\n", age);
    printf("grade = %c\n", grade);
    printf("temperature = %f\n", temperature);

    return 0;
}
```

#### C++

``` cpp
#include <iostream>

int main()
{
    int age = 20;
    char grade = 'A';
    double temperature = 23.5;

    std::cout << "age = " << age << '\n';
    std::cout << "grade = " << grade << '\n';
    std::cout << "temperature = " << temperature << '\n';

    return 0;
}
```

------------------------------------------------------------------------

## 20. 코드 추적 연습

다음 코드를 실행하지 말고 변수의 값을 직접 추적한다.

``` c
int a = 10;
int b = 20;

a = a + 5;
b = a + b;
a = b - a;
```

  실행 시점       `a`   `b`
  ------------- ----- -----
  초기화 직후      10    20
  `a = a + 5`         
  `b = a + b`         
  `a = b - a`         

이런 식으로 변수의 변화를 직접 추적하는 습관은 이후 포인터와 디버깅을
배울 때 중요하다.

------------------------------------------------------------------------

## 21. 반드시 설명할 수 있어야 하는 것

-   [ ] 자료형(Data Type)이 무엇인지 설명할 수 있다.
-   [ ] `int a = 10;`을 구성 요소별로 설명할 수 있다.
-   [ ] 선언, 초기화, 대입을 구분할 수 있다.
-   [ ] `char`, `int`, `float`, `double`의 기본 용도를 설명할 수 있다.
-   [ ] `'A'`와 `"A"`의 차이를 설명할 수 있다.
-   [ ] `signed`와 `unsigned`의 기본적인 차이를 설명할 수 있다.
-   [ ] `sizeof`가 무엇인지 설명할 수 있다.
-   [ ] `10 / 3`의 결과가 왜 `3`인지 설명할 수 있다.
-   [ ] 형변환이 필요한 이유를 설명할 수 있다.
-   [ ] 자료형과 메모리의 관계를 설명할 수 있다.
-   [ ] 임베디드에서 데이터의 크기가 중요한 이유를 설명할 수 있다.
-   [ ] `uint8_t`, `uint16_t`, `uint32_t`가 어떤 목적으로 등장하는지
    대략 설명할 수 있다.

------------------------------------------------------------------------

## 22. 복습 문제

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q1. 최종 a의 값은?`</strong>`{=html}
```{=html}
</summary>
```
``` c
int a = 10;

a = 20;
a = a + 5;
```

**정답:** `25`

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q2. result의 값은?`</strong>`{=html}
```{=html}
</summary>
```
``` c
int a = 7;
int b = 2;

int result = a / b;
```

**정답:** `3`

`int / int`이므로 정수 나눗셈이 수행된다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q3. 다음 결과가 Q2와 다른 이유는?`</strong>`{=html}
```{=html}
</summary>
```
``` c
int a = 7;
int b = 2;

double result = (double)a / b;
```

**정답:** `a`를 `double`로 명시적으로 변환했기 때문에 실수 연산으로
진행된다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q4. 문자와 문자열을 구분하라.`</strong>`{=html}
```{=html}
</summary>
```
``` c
'A'
"A"
```

**정답:**

``` text
'A' → 문자
"A" → 문자열
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
`<strong>`{=html}Q5. 선언 / 초기화 / 대입을 구분하라.`</strong>`{=html}
```{=html}
</summary>
```
``` c
int a;
int b = 10;
b = 20;
```

**정답:**

``` text
int a;       → 선언
int b = 10;  → 선언 + 초기화
b = 20;      → 대입
```

```{=html}
</details>
```

------------------------------------------------------------------------

## 23. Bug Log에 기록할 항목

학습 중 실수를 했다면 `99_Bug_Log`에 다음 형식으로 기록한다.

```` markdown
## 정수 나눗셈 실수

### 잘못 생각한 코드

```c
int a = 10;
int b = 3;

double result = a / b;
```

### 내가 예상한 결과

3.333...

### 실제 원인

`a / b`가 먼저 `int / int` 연산으로 수행된다.

### 수정

```c
double result = (double)a / b;
```

### 재발 방지 규칙

연산 결과를 볼 때 결과 변수의 타입만 보지 말고 피연산자의 타입을 먼저 확인한다.
````

**실제 실수 → 원인 → 수정 → 재발 방지 규칙** 순으로 남긴다.

------------------------------------------------------------------------

## 24. 현재 단계에서 깊게 공부하지 않을 내용

다음 내용은 중요하지만 지금 모두 학습하면 자료형의 핵심 흐름을 놓칠 수
있으므로 이후 단계에서 확장한다.

-   integer promotion
-   usual arithmetic conversions
-   signed/unsigned 세부 변환 규칙
-   integer overflow 세부 규칙
-   2의 보수 표현
-   IEEE 754
-   floating-point 오차
-   alignment
-   padding
-   endianness
-   object representation
-   C와 C++의 타입 시스템 차이
-   `bool`
-   C++ `enum class`
-   `auto`
-   `decltype`
-   `std::byte`
-   `std::size_t`

> \[!NOTE\] **모른다는 의미가 아니라 학습 순서를 뒤로 미룬 것이다.**
>
> 특히 정수 표현, byte/bit, endianness, fixed-width integer는 임베디드
> 학습에서 다시 중요하게 다룬다.

------------------------------------------------------------------------

## 25. 노트 연결 관계

``` mermaid
flowchart TD
    DATA["00_Common/Data_Type.md"]

    DATA --> OP["00_Common/Operator.md"]
    OP --> CF["00_Common/Control_Flow.md"]
    CF --> FN["00_Common/Function.md"]

    FN --> ARR["01_C/Array_String.md"]
    ARR --> PTR["01_C/Pointer.md"]
    PTR --> MEM["00_Common/Memory_Model.md"]

    MEM --> STRUCT["01_C/Struct.md"]
    MEM --> MALLOC["01_C/malloc_free.md"]

    PTR --> BIT["01_C/Bit_Operation.md"]
    BIT --> VOL["01_C/volatile.md"]
    VOL --> EMB["01_C/Embedded_C.md"]

    MEM --> REF["02_CPP/Reference.md"]
    REF --> CLASS["02_CPP/Class.md"]
    CLASS --> RAII["02_CPP/RAII.md"]
```

------------------------------------------------------------------------

## 26. 핵심 요약

``` mermaid
mindmap
  root((Data Type))
    정수형
      char
      short
      int
      long
      long long
      signed
      unsigned
    실수형
      float
      double
      long double
    기본 개념
      선언
      초기화
      대입
      sizeof
      형변환
    Memory
      저장 공간
      Address
      Pointer
    Embedded
      Bit
      uint8_t
      uint16_t
      uint32_t
      Register
```

### 최종적으로 기억할 세 문장

> **1. 자료형은 데이터를 어떤 종류의 값으로 해석하고 다룰 것인지를
> 나타낸다.**

> **2. C/C++에서는 값만 보지 말고 항상 `값 + 자료형 + 메모리`를 함께
> 생각한다.**

> **3. 임베디드에서는 이 개념이
> `자료형 → 크기 → Byte → Bit → Register → Hardware`로 이어진다.**

------------------------------------------------------------------------

## 다음 학습

``` mermaid
flowchart LR
    A["Data_Type.md<br/>현재"] --> B["Operator.md"]
    B --> C["Control_Flow.md"]
    C --> D["Function.md"]
    D --> E["Array / String"]
    E --> F["Pointer"]
    F --> G["Memory Model"]
```

**다음 문서:** `00_Common/Operator.md`
