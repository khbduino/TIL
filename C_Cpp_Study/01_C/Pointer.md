# Pointer

> **학습 목표**
>
> C 포인터(Pointer)를 **주소를 저장하는 변수**라는 출발점에서 이해하고,
> `&`, `*`, 포인터 타입, 역참조, 배열과 포인터, 함수 매개변수, 포인터
> 연산, `NULL`, 이중 포인터까지 단계적으로 학습한다.
>
> 최종적으로 **변수 → 메모리 → 주소 → 포인터 → 배열 → 함수 → 버퍼 →
> 레지스터 → 임베디드 하드웨어**의 연결을 설명할 수 있어야 한다.
>
> 이 노트는 `C_Cpp_Study/01_C/Pointer.md`에 위치하며 `Array_String.md`
> 다음 단계로 학습한다.

------------------------------------------------------------------------

# 1. 포인터를 배우기 전에

``` c
int number = 10;
```

변수에는 이름, 타입, 값뿐 아니라 실제 데이터가 저장된 **메모리 주소**가
존재한다.

``` mermaid
flowchart LR
    A["number"] --> B["Type: int"]
    A --> C["Value: 10"]
    A --> D["Memory"]
    D --> E["Address"]
```

포인터 학습에서는 다음 관점을 추가한다.

``` text
변수
├── 이름
├── 타입
├── 값
└── 주소
```

------------------------------------------------------------------------

# 2. 메모리 주소

메모리의 위치를 구분하기 위한 값이 주소(Address)다.

``` text
Address      Value
────────     ─────
0x1000       ...
0x1001       ...
0x1002       ...
```

`number`가 `0x1000`이라는 가상의 위치에 있다고 생각하면:

``` text
number

Address : 0x1000
Value   : 10
Type    : int
```

> \[!NOTE\] 위 주소는 설명용이다. 실제 주소는 실행 환경에 따라 달라진다.

------------------------------------------------------------------------

# 3. 주소 연산자 `&`

변수의 주소를 얻을 때 `&`를 사용한다.

``` c
int number = 10;

&number;
```

`&number`은 `number`의 주소를 의미한다.

주소 출력 예:

``` c
#include <stdio.h>

int main(void)
{
    int number = 10;

    printf("%p\n", (void *)&number);

    return 0;
}
```

``` mermaid
flowchart LR
    A["number"] -->|"&number"| B["number의 주소"]
```

> \[!IMPORTANT\] `a & b`의 `&`는 Bitwise AND이고 `&number`의 `&`는 주소
> 연산자다. 문맥으로 구분한다.

------------------------------------------------------------------------

# 4. 포인터란?

### 한 문장 정의

**포인터(Pointer)는 메모리 주소를 값으로 저장하는 객체다.**

``` c
int number = 10;
int *ptr = &number;
```

  표현        의미
  ----------- -----------------------------------
  `number`    일반 `int` 변수
  `&number`   `number`의 주소
  `ptr`       포인터 변수
  `int *`     `int` 객체를 가리키는 포인터 타입

``` text
ptr
┌──────────────┐
│ &number      │ ─────────┐
└──────────────┘          │
                          ▼
                       number
                    ┌──────────┐
                    │ 10       │
                    └──────────┘
```

------------------------------------------------------------------------

# 5. 포인터 선언과 초기화

기본 형식:

``` c
자료형 *포인터이름;
```

예:

``` c
int *int_ptr;
char *char_ptr;
double *double_ptr;
```

초기화:

``` c
int number = 10;
int *ptr = &number;
```

``` mermaid
flowchart LR
    A["number"] --> B["&number"]
    B --> C["ptr에 주소 저장"]
```

> **포인터도 타입을 가진다.**

------------------------------------------------------------------------

# 6. 역참조 연산자 `*`

포인터가 가리키는 객체에 접근할 때 `*`를 사용한다.

``` c
int number = 10;
int *ptr = &number;

printf("%d\n", *ptr);
```

결과:

``` text
10
```

``` mermaid
flowchart LR
    A["ptr"] --> B["number의 주소"]
    B -->|"*ptr"| C["number 객체"]
    C --> D["10"]
```

이를 **역참조(Dereference)**라고 한다.

------------------------------------------------------------------------

# 7. `*`의 문맥

``` c
a * b;      /* 곱셈 */
int *ptr;   /* 포인터 선언 */
*ptr;       /* 역참조 */
```

``` mermaid
flowchart TD
    A["*"] --> B["a * b → 곱셈"]
    A --> C["int *ptr → 포인터 선언"]
    A --> D["*ptr → 역참조"]
```

기호만 보지 않고 전체 표현식을 본다.

------------------------------------------------------------------------

# 8. 주소와 값 구분

``` c
int number = 10;
int *ptr = &number;
```

다음 네 표현을 반드시 구분한다.

  표현        의미
  ----------- ------------------------
  `number`    `number`의 값
  `&number`   `number`의 주소
  `ptr`       포인터에 저장된 주소
  `*ptr`      포인터가 가리키는 객체

개념적으로:

``` text
number   → 10
&number  → 0x1000
ptr      → 0x1000
*ptr     → 10
```

포인터 문제의 첫 번째 질문은 항상 다음과 같다.

> **지금 보고 있는 표현은 값인가, 주소인가?**

------------------------------------------------------------------------

# 9. 포인터를 통해 원본 변경

``` c
int number = 10;
int *ptr = &number;

*ptr = 100;
```

결과:

``` text
number = 100
```

``` mermaid
flowchart LR
    A["ptr"] --> B["number"]
    C["*ptr = 100"] --> B
    B --> D["number = 100"]
```

`*ptr`은 별개의 정수 변수가 아니라 `ptr`이 가리키는 객체에 접근하는
표현이다.

------------------------------------------------------------------------

# 10. 포인터 자체도 변수다

``` c
int number = 10;
int *ptr = &number;
```

`ptr` 자체도 메모리에 존재하는 변수다.

따라서:

``` c
&ptr
```

도 존재한다.

``` mermaid
flowchart LR
    A["&ptr"] --> B["ptr"]
    B --> C["number"]
    C --> D["10"]
```

  표현     의미
  -------- --------------------------
  `&ptr`   포인터 변수 `ptr`의 주소
  `ptr`    `number`의 주소
  `*ptr`   `number`

이 관계가 이후 이중 포인터로 이어진다.

------------------------------------------------------------------------

# 11. `NULL` 포인터

아직 유효한 객체를 가리키지 않는 상태를 명시할 때 널 포인터를 사용할 수
있다.

``` c
int *ptr = NULL;
```

``` text
ptr
┌──────────┐
│ NULL     │
└──────────┘
```

`NULL` 포인터를 역참조하면 안 된다.

``` c
int *ptr = NULL;

*ptr = 10;  /* 잘못된 접근 */
```

> \[!WARNING\] 널 포인터를 역참조하면 Undefined Behavior가 발생한다.

------------------------------------------------------------------------

# 12. 포인터 검사와 Short-Circuit

``` c
if (ptr != NULL)
{
    printf("%d\n", *ptr);
}
```

또는:

``` c
if (ptr != NULL && *ptr == 10)
{
    /* ... */
}
```

``` mermaid
flowchart TD
    A["ptr != NULL?"] -->|"No"| B["*ptr 평가하지 않음"]
    A -->|"Yes"| C["*ptr 평가"]
```

`Operator.md`에서 배운 short-circuit evaluation이 실제 포인터 코드와
연결된다.

> \[!NOTE\] `ptr != NULL`이라고 해서 모든 포인터가 안전한 것은 아니다.
> 수명이 끝난 객체를 가리키는 dangling pointer도 존재한다.

------------------------------------------------------------------------

# 13. 초기화되지 않은 포인터

위험한 코드:

``` c
int *ptr;

*ptr = 10;
```

`ptr`이 유효한 객체의 주소로 초기화되지 않았다.

올바른 기본 예:

``` c
int number = 0;
int *ptr = &number;
```

아직 대상이 없다면:

``` c
int *ptr = NULL;
```

> \[!WARNING\] 포인터를 역참조하기 전에 무엇을 가리키는지 설명할 수
> 있어야 한다.

------------------------------------------------------------------------

# 14. 함수와 포인터

`Function.md`에서 다음을 배웠다.

``` c
void change_value(int value)
{
    value = 100;
}
```

``` c
int number = 10;

change_value(number);
```

이 경우 `number`는 변경되지 않는다.

주소를 전달하면 원본 객체에 접근할 수 있다.

``` c
void change_value(int *value)
{
    *value = 100;
}
```

호출:

``` c
int number = 10;

change_value(&number);
```

결과:

``` text
number = 100
```

``` mermaid
sequenceDiagram
    participant M as main()
    participant N as number
    participant F as change_value()

    M->>N: number = 10
    M->>F: change_value(&number)
    F->>N: *value = 100
    N-->>M: number = 100
```

------------------------------------------------------------------------

# 15. `scanf()`에서 `&`를 사용한 이유

``` c
int number;

scanf("%d", &number);
```

이제 이유를 설명할 수 있다.

``` text
입력값을 number에 저장해야 함
        ↓
number가 위치한 메모리를 알아야 함
        ↓
&number로 주소 전달
        ↓
scanf가 해당 위치에 값을 기록
```

``` mermaid
flowchart LR
    A["Input"] --> B["scanf"]
    C["&number"] --> B
    B --> D["number의 메모리"]
    D --> E["값 저장"]
```

------------------------------------------------------------------------

# 16. 두 변수 교환

포인터의 대표적인 함수 예제다.

``` c
void swap(int *a, int *b)
{
    int temp = *a;

    *a = *b;
    *b = temp;
}
```

호출:

``` c
int x = 10;
int y = 20;

swap(&x, &y);
```

결과:

``` text
x = 20
y = 10
```

``` mermaid
flowchart LR
    A["x"] --> C["swap"]
    B["y"] --> C
    C --> D["x와 y의 원본 값 교환"]
```

------------------------------------------------------------------------

# 17. 배열과 포인터

``` c
int numbers[4] = {10, 20, 30, 40};
```

배열의 요소는 연속적으로 배치된다.

``` text
numbers

┌────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │
└────┴────┴────┴────┘
 [0]  [1]  [2]  [3]
```

배열 이름은 많은 표현식에서 첫 번째 요소를 가리키는 포인터로 변환된다.

``` c
int *ptr = numbers;
```

이때 `ptr`은 첫 번째 요소를 가리킨다.

``` text
ptr
 │
 ▼
┌────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │
└────┴────┴────┴────┘
```

> \[!IMPORTANT\] **배열과 포인터가 완전히 같은 것은 아니다.**
>
> 배열 표현식이 특정 문맥에서 첫 요소를 가리키는 포인터로 변환되는
> 규칙이 있는 것이다.

------------------------------------------------------------------------

# 18. `numbers`와 `&numbers[0]`

다음 배열이 있을 때:

``` c
int numbers[4] = {10, 20, 30, 40};
```

많은 표현식에서:

``` c
numbers
```

와:

``` c
&numbers[0]
```

는 첫 번째 요소를 가리키는 주소값으로 사용된다.

``` mermaid
flowchart LR
    A["numbers"] --> C["첫 요소의 주소"]
    B["&numbers[0]"] --> C
    C --> D["numbers[0]"]
```

단, 배열 타입 자체와 포인터 타입 자체가 동일하다는 뜻은 아니다.

------------------------------------------------------------------------

# 19. 포인터 연산

``` c
int numbers[4] = {10, 20, 30, 40};

int *ptr = numbers;
```

다음 관계를 가진다.

``` text
ptr       → numbers[0]
ptr + 1   → numbers[1]
ptr + 2   → numbers[2]
ptr + 3   → numbers[3]
```

``` mermaid
flowchart LR
    A["ptr"] --> B["numbers[0]"]
    C["ptr + 1"] --> D["numbers[1]"]
    E["ptr + 2"] --> F["numbers[2]"]
    G["ptr + 3"] --> H["numbers[3]"]
```

포인터 연산은 가리키는 타입의 **요소 단위**로 이해한다.

------------------------------------------------------------------------

# 20. 배열 인덱스와 포인터 표현

핵심 관계:

``` c
numbers[i]
```

와:

``` c
*(numbers + i)
```

는 같은 배열 요소에 접근하는 대응 표현이다.

예:

``` c
int numbers[3] = {10, 20, 30};

printf("%d\n", numbers[1]);
printf("%d\n", *(numbers + 1));
```

둘 다:

``` text
20
```

을 출력한다.

``` mermaid
flowchart LR
    A["numbers[1]"] --> C["두 번째 요소"]
    B["*(numbers + 1)"] --> C
```

------------------------------------------------------------------------

# 21. 포인터 연산과 범위

포인터 연산을 사용할 수 있다고 해서 임의의 메모리를 자유롭게 이동해도
되는 것은 아니다.

배열을 순회할 때는 해당 배열 범위와 관련된 유효한 포인터 연산 규칙을
지켜야 한다.

``` c
int numbers[5];
int *ptr = numbers;
```

요소 접근 시:

``` text
numbers[0] ~ numbers[4]
```

범위를 벗어난 객체를 역참조하지 않는다.

> \[!WARNING\] 포인터를 이용한 Out-of-Bounds 접근 역시 Undefined
> Behavior를 일으킬 수 있다.

------------------------------------------------------------------------

# 22. 배열을 함수에 전달하기

``` c
void print_array(int numbers[], int size)
{
    for (int i = 0; i < size; i++)
    {
        printf("%d\n", numbers[i]);
    }
}
```

함수 매개변수 문맥에서는 다음 표기도 사용한다.

``` c
void print_array(int *numbers, int size)
```

호출:

``` c
int numbers[5] = {1, 2, 3, 4, 5};

print_array(numbers, 5);
```

``` mermaid
flowchart LR
    A["numbers"] --> B["함수 매개변수"]
    C["size = 5"] --> B
    B --> D["numbers[i] 처리"]
```

------------------------------------------------------------------------

# 23. 함수 내부의 `sizeof(array)` 주의

다음 코드에서:

``` c
void print_array(int numbers[])
{
    size_t count = sizeof(numbers) / sizeof(numbers[0]);
}
```

`numbers`는 함수 매개변수 문맥에서 원래 배열 전체 객체 자체가 아니다.

따라서 호출한 쪽의 배열 요소 수를 위 식으로 복원할 수 있다고 생각하면 안
된다.

일반적으로 크기를 별도로 전달한다.

``` c
void print_array(int numbers[], int size);
```

> **배열을 함수에 전달할 때 `pointer + size` 패턴이 자주 등장하는
> 이유다.**

------------------------------------------------------------------------

# 24. 문자열과 포인터

``` c
char text[] = "ABC";
char *ptr = text;
```

문자열:

``` text
'A' 'B' 'C' '\0'
 ^
 |
ptr
```

포인터로 순회:

``` c
while (*ptr != '\0')
{
    printf("%c\n", *ptr);
    ptr++;
}
```

``` mermaid
flowchart TD
    A["ptr가 현재 문자 가리킴"] --> B{"*ptr == '\\0'?"}
    B -->|"No"| C["문자 처리"]
    C --> D["ptr++"]
    D --> A
    B -->|"Yes"| E["종료"]
```

------------------------------------------------------------------------

# 25. `const T *`

함수가 데이터를 읽기만 해야 한다면 `const`를 사용할 수 있다.

``` c
int sum_array(const int *numbers, int size)
{
    int sum = 0;

    for (int i = 0; i < size; i++)
    {
        sum += numbers[i];
    }

    return sum;
}
```

`const int *numbers`는 이 포인터를 통해 가리키는 `int` 객체를 수정하지
않겠다는 인터페이스를 표현한다.

``` mermaid
flowchart LR
    A["int *ptr"] --> B["읽기 / 쓰기"]
    C["const int *ptr"] --> D["이 포인터를 통한 읽기"]
```

`const`의 여러 포인터 조합은 이후 별도로 확장한다.

------------------------------------------------------------------------

# 26. `void *`

C에서는 일반적인 객체 주소를 전달하기 위한 포인터 타입으로 `void *`를
사용할 수 있다.

``` c
int number = 10;
void *ptr = &number;
```

`void *` 자체는 가리키는 객체의 구체적인 타입 정보를 표현하지 않는다.

실제 객체에 타입에 맞게 접근하려면 적절한 포인터 타입이 필요하다.

``` c
int value = *(int *)ptr;
```

`void *`는 이후 `malloc()`에서 다시 등장한다.

------------------------------------------------------------------------

# 27. 이중 포인터

포인터의 주소를 저장하는 포인터다.

``` c
int number = 10;

int *ptr = &number;
int **pptr = &ptr;
```

``` text
pptr
  │
  ▼
 ptr
  │
  ▼
number
  │
  ▼
 10
```

  표현       의미
  ---------- -----------------
  `ptr`      `number`의 주소
  `*ptr`     `number`
  `pptr`     `ptr`의 주소
  `*pptr`    `ptr`
  `**pptr`   `number`

``` mermaid
flowchart LR
    A["pptr"] --> B["ptr"]
    B --> C["number"]
    C --> D["10"]
```

------------------------------------------------------------------------

# 28. 이중 포인터가 필요한 상황

이후 다음 주제에서 등장한다.

``` text
함수에서 포인터 자체 변경
동적 메모리
문자열 배열
자료구조
일부 C API
```

``` mermaid
flowchart TD
    A["Object"] --> B["Pointer"]
    B --> C["Pointer의 주소"]
    C --> D["Double Pointer"]
```

현재는 `**pptr`을 그림으로 추적할 수 있는 수준을 목표로 한다.

------------------------------------------------------------------------

# 29. 구조체 포인터 예고

구조체를 배우면:

``` c
struct Sensor
{
    int value;
};
```

다음 형태를 사용하게 된다.

``` c
struct Sensor sensor;
struct Sensor *ptr = &sensor;
```

구조체 포인터에서는:

``` c
ptr->value
```

라는 표현이 등장한다.

``` mermaid
flowchart LR
    A["Pointer"] --> B["Struct Pointer"]
    B --> C["->"]
    C --> D["Driver / Data Structure"]
```

`Struct.md`에서 자세히 다룬다.

------------------------------------------------------------------------

# 30. 동적 메모리 예고

이후:

``` c
int *ptr = malloc(sizeof(int));
```

와 같은 코드를 배우게 된다.

사용이 끝나면:

``` c
free(ptr);
```

``` mermaid
flowchart LR
    A["Pointer"] --> B["malloc"]
    B --> C["Heap"]
    C --> D["Use"]
    D --> E["free"]
```

이 단계에서 Pointer, Lifetime, Ownership의 관계가 중요해진다.

------------------------------------------------------------------------

# 31. Dangling Pointer 예고

포인터가 가리키던 객체의 수명이 끝났지만 포인터가 이전 주소를 계속
보유하고 있는 상황이 발생할 수 있다.

``` mermaid
flowchart LR
    A["Pointer"] --> B["Valid Object"]
    B --> C["Object Lifetime 종료"]
    C --> D["Pointer는 이전 주소 보유"]
    D --> E["Dangling Pointer"]
```

> \[!WARNING\] 포인터가 `NULL`이 아니라는 사실만으로 유효성을 보장할 수
> 없다.

객체의 수명은 `Memory_Model.md`에서 자세히 다룬다.

------------------------------------------------------------------------

# 32. 포인터와 메모리 모델

``` mermaid
flowchart TD
    A["Pointer"] --> B["Stack Object"]
    A --> C["Static / Global Object"]
    A --> D["Heap Object"]
    A --> E["Memory-Mapped Register"]
```

다음 단계에서는 포인터가 가리키는 객체의 **저장 위치와 수명**을
학습한다.

``` text
Pointer
 ↓
Object
 ↓
Storage
 ↓
Lifetime
```

------------------------------------------------------------------------

# 33. 임베디드와 포인터

임베디드에서는 주소가 하드웨어 레지스터와 연결될 수 있다.

개념:

``` mermaid
flowchart LR
    A["Address"] --> B["Pointer"]
    B --> C["Register"]
    C --> D["Peripheral"]
    D --> E["Hardware"]
```

이후 다음 형태를 접하게 된다.

``` c
volatile unsigned int *reg;
```

학습 흐름:

``` text
Pointer
+
volatile
+
Bit Operation
=
Register Access
```

> \[!IMPORTANT\] 실제 MMIO 주소 접근은 MCU 데이터시트, 메모리 맵, 타입
> 폭, 정렬, `volatile` 등의 조건을 이해한 뒤 다룬다.

------------------------------------------------------------------------

# 34. Memory-Mapped I/O

Memory-Mapped I/O에서는 주변장치 레지스터가 CPU 주소 공간에 배치된다.

``` mermaid
flowchart LR
    A["Pointer"] --> B["Address"]
    B --> C["MMIO"]
    C --> D["Register"]
    D --> E["Bit Operation"]
    E --> F["GPIO / UART / Timer"]
```

포인터를 배우는 이유가 임베디드에서 특히 명확해지는 부분이다.

------------------------------------------------------------------------

# 35. 포인터와 버퍼

``` c
unsigned char rx_buffer[64];
```

버퍼 처리 함수:

``` c
void process_buffer(const unsigned char *buffer, int size)
{
    for (int i = 0; i < size; i++)
    {
        /* buffer[i] 처리 */
    }
}
```

``` mermaid
flowchart LR
    A["UART"] --> B["Array Buffer"]
    B --> C["Pointer"]
    C --> D["Function"]
    D --> E["Parser"]
```

이후 다음으로 확장된다.

``` text
UART Buffer
→ Ring Buffer
→ Packet Parser
→ Protocol
```

------------------------------------------------------------------------

# 36. C++ Reference와의 연결

C:

``` c
void change(int *value)
{
    *value = 100;
}
```

C++에서는 Reference도 사용할 수 있다.

``` cpp
void change(int& value)
{
    value = 100;
}
```

``` mermaid
flowchart TD
    A["원본 객체 접근"] --> B["C Pointer"]
    A --> C["C++ Pointer"]
    A --> D["C++ Reference"]
```

C 포인터를 먼저 이해하면 C++ Reference와 Smart Pointer의 목적도 더
명확해진다.

------------------------------------------------------------------------

# 37. 대표적인 포인터 버그

``` mermaid
mindmap
  root((Pointer Bugs))
    NULL Dereference
    Uninitialized Pointer
    Dangling Pointer
    Out of Bounds
    Use After Free
    Double Free
    Wrong Pointer Arithmetic
```

현재 단계에서 우선 집중할 것:

``` text
NULL Dereference
Uninitialized Pointer
Out-of-Bounds
```

`Use-After-Free`, `Double Free`, Memory Leak은 동적 메모리에서 자세히
다룬다.

------------------------------------------------------------------------

# 38. 포인터 디버깅 체크리스트

-   [ ] 포인터 타입은 무엇인가?
-   [ ] 포인터가 초기화되었는가?
-   [ ] 어떤 객체를 가리키는가?
-   [ ] `NULL`인가?
-   [ ] 객체의 수명은 유효한가?
-   [ ] 배열 범위 안인가?
-   [ ] 역참조가 필요한 표현인가?
-   [ ] 포인터 자체를 바꾸는가, 가리키는 값을 바꾸는가?

``` mermaid
flowchart TD
    A["Pointer Type"] --> B["Stored Address"]
    B --> C["Target Object"]
    C --> D["Lifetime"]
    D --> E["Range"]
    E --> F["Dereference"]
```

------------------------------------------------------------------------

# 39. 실습 1 --- 값과 주소

``` c
int number = 10;
int *ptr = &number;
```

다음을 출력하고 각각 값인지 주소인지 설명한다.

``` text
number
&number
ptr
*ptr
&ptr
```

------------------------------------------------------------------------

# 40. 실습 2 --- 포인터로 값 변경

``` c
int number = 10;
int *ptr = &number;
```

`ptr`을 이용해 `number`를 `100`으로 변경한다.

``` c
*ptr = 100;
```

------------------------------------------------------------------------

# 41. 실습 3 --- 함수에서 원본 변경

다음 함수를 완성한다.

``` c
void set_zero(int *value)
{
    /* ... */
}
```

호출:

``` c
int number = 100;

set_zero(&number);
```

목표:

``` text
number = 0
```

------------------------------------------------------------------------

# 42. 실습 4 --- Swap

``` c
void swap(int *a, int *b);
```

호출 전:

``` text
a = 10
b = 20
```

호출 후:

``` text
a = 20
b = 10
```

------------------------------------------------------------------------

# 43. 실습 5 --- 배열과 포인터

``` c
int numbers[5] = {10, 20, 30, 40, 50};
```

다음 결과를 예상한다.

``` c
*numbers
*(numbers + 1)
*(numbers + 2)
```

그리고 다음과 비교한다.

``` c
numbers[0]
numbers[1]
numbers[2]
```

------------------------------------------------------------------------

# 44. 실습 6 --- 배열 합계 함수

``` c
int sum_array(const int *numbers, int size);
```

예:

``` c
int numbers[] = {1, 2, 3, 4, 5};

int result = sum_array(numbers, 5);
```

목표:

``` text
15
```

------------------------------------------------------------------------

# 45. 실습 7 --- 문자열 포인터 순회

``` c
char text[] = "Hello";
char *ptr = text;
```

`'\0'`을 만날 때까지 포인터를 이동하며 출력한다.

``` c
while (*ptr != '\0')
{
    printf("%c\n", *ptr);
    ptr++;
}
```

------------------------------------------------------------------------

# 46. 실습 8 --- 이중 포인터

``` c
int number = 10;
int *ptr = &number;
int **pptr = &ptr;
```

다음을 설명한다.

``` text
number
ptr
*ptr
pptr
*pptr
**pptr
```

------------------------------------------------------------------------

# 47. 코드 추적 연습

``` c
int a = 10;
int b = 20;

int *ptr = &a;

*ptr = 30;
ptr = &b;
*ptr = 40;
```

  실행 단계       `a`   `b` `ptr`이 가리키는 객체     `*ptr`
  ------------- ----- ----- ----------------------- --------
  초기             10    20 `a`                           10
  `*ptr = 30`                                       
  `ptr = &b`                                        
  `*ptr = 40`                                       

------------------------------------------------------------------------

# 48. 반드시 설명할 수 있어야 하는 것

-   [ ] 메모리 주소를 설명할 수 있다.
-   [ ] `&variable`의 의미를 설명할 수 있다.
-   [ ] 포인터를 정의할 수 있다.
-   [ ] `int *ptr`의 의미를 설명할 수 있다.
-   [ ] `ptr`과 `*ptr`을 구분할 수 있다.
-   [ ] `number`, `&number`, `ptr`, `*ptr`을 구분할 수 있다.
-   [ ] 역참조를 설명할 수 있다.
-   [ ] 포인터로 원본 객체를 변경할 수 있다.
-   [ ] `NULL` 역참조가 위험한 이유를 설명할 수 있다.
-   [ ] 초기화되지 않은 포인터가 위험한 이유를 설명할 수 있다.
-   [ ] 함수에 주소를 전달하는 이유를 설명할 수 있다.
-   [ ] `scanf()`에서 `&`를 사용하는 이유를 설명할 수 있다.
-   [ ] 배열과 포인터의 관계를 설명할 수 있다.
-   [ ] `array[i]`와 `*(array + i)`의 관계를 설명할 수 있다.
-   [ ] 포인터 연산이 요소 단위임을 설명할 수 있다.
-   [ ] 배열과 함께 크기를 전달하는 이유를 설명할 수 있다.
-   [ ] 문자열을 포인터로 순회할 수 있다.
-   [ ] `const T *`의 기본 의미를 설명할 수 있다.
-   [ ] 이중 포인터를 그림으로 추적할 수 있다.
-   [ ] 포인터와 Stack/Heap/MMIO의 연결을 설명할 수 있다.
-   [ ] 임베디드에서 포인터가 중요한 이유를 설명할 수 있다.

------------------------------------------------------------------------

# 49. 복습 문제

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q1. 포인터란?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 메모리 주소를 값으로 저장하는 객체다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q2. 다음 코드에서 ptr에 저장되는
것은?`</strong>`{=html}
```{=html}
</summary>
```
``` c
int number = 10;
int *ptr = &number;
```

**정답:** `number`의 주소.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q3. 위 코드에서 \*ptr은?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** `10`

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q4. 다음 코드 실행 후 number는?`</strong>`{=html}
```{=html}
</summary>
```
``` c
int number = 10;
int *ptr = &number;

*ptr = 50;
```

**정답:** `50`

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q5. ptr과 \*ptr의 차이는?`</strong>`{=html}
```{=html}
</summary>
```
**정답:**

``` text
ptr  → 주소
*ptr → 그 주소가 가리키는 객체
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
`<strong>`{=html}Q6. NULL 포인터를 역참조해도 되는가?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 안 된다. Undefined Behavior가 발생한다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q7. scanf("%d", &number)에서 &number가 필요한
이유는?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** `scanf()`가 입력값을 기록할 `number` 객체의 메모리 주소를
전달하기 위해서다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q8. array\[i\]와 \*(array + i)의
관계는?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 같은 배열 요소에 접근하는 대응 표현이다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q9. 다음 코드에서 \*\*pptr은?`</strong>`{=html}
```{=html}
</summary>
```
``` c
int number = 10;
int *ptr = &number;
int **pptr = &ptr;
```

**정답:** `10`

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q10. 포인터가 임베디드에서 중요한
이유는?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 버퍼, 함수 인터페이스, 메모리 주소 및 MMIO 기반 하드웨어
레지스터 접근처럼 주소를 직접 다루는 작업에 사용되기 때문이다.

```{=html}
</details>
```

------------------------------------------------------------------------

# 50. Bug Log

`99_Bug_Log/Pointer_Bugs.md`에 다음 유형을 기록한다.

```` markdown
## 초기화되지 않은 포인터 역참조

### 잘못 작성한 코드

```c
int *ptr;

*ptr = 10;
```

### 원인

`ptr`이 유효한 `int` 객체의 주소로 초기화되지 않았다.

### 수정

```c
int number = 0;
int *ptr = &number;

*ptr = 10;
```

### 재발 방지 규칙

포인터 역참조 전 확인한다.

1. 초기화되었는가?
2. 유효한 객체를 가리키는가?
3. 객체의 수명이 유효한가?
4. 배열이라면 범위 안인가?
````

추천 항목:

``` text
Uninitialized Pointer
NULL Dereference
Out-of-Bounds
Dangling Pointer
Wrong Pointer Arithmetic
함수 주소 전달 누락
Use-After-Free
Double Free
```

------------------------------------------------------------------------

# 51. 포인터가 헷갈리면 그림을 그린다

``` c
int number = 10;
int *ptr = &number;
```

``` text
ptr
┌──────────────┐
│ &number      │ ─────────┐
└──────────────┘          │
                          ▼
                       number
                    ┌──────────┐
                    │ 10       │
                    └──────────┘
```

이중 포인터:

``` text
pptr
  │
  ▼
 ptr
  │
  ▼
number
  │
  ▼
 10
```

> \[!TIP\] 포인터 학습 초기에는 머릿속으로만 추적하지 말고 **변수 박스와
> 화살표를 직접 그린다.**

------------------------------------------------------------------------

# 52. 지금까지의 개념 연결

``` mermaid
flowchart LR
    A["Data Type"] --> B["Operator"]
    B --> C["Control Flow"]
    C --> D["Function"]
    D --> E["Array / String"]
    E --> F["Pointer"]
```

  기존 개념      포인터에서의 역할
  -------------- -----------------------
  Data Type      포인터 타입
  Operator       `&`, `*`, 포인터 연산
  Control Flow   포인터 검사
  Function       주소 전달
  Array          연속 메모리
  String         `char *` 순회

------------------------------------------------------------------------

# 53. 코딩 테스트로의 연결

C++ 코딩 테스트에서 raw pointer를 자주 직접 작성하지 않더라도 다음
개념의 기반이 된다.

``` mermaid
flowchart LR
    A["C Pointer"] --> B["Memory"]
    B --> C["C++ Reference"]
    B --> D["Iterator"]
    B --> E["Linked Structure"]
    B --> F["Smart Pointer"]
```

포인터를 이해하면 C++이 제공하는 추상화의 목적도 이해하기 쉬워진다.

------------------------------------------------------------------------

# 54. 임베디드로의 연결

``` mermaid
flowchart TD
    A["Pointer"] --> B["Buffer"]
    A --> C["Register"]
    A --> D["Driver"]
    A --> E["DMA Buffer"]

    B --> F["UART / SPI / I2C"]
    C --> G["MMIO"]
    G --> H["GPIO / Timer / ADC"]
```

핵심 학습 경로:

``` text
Pointer
 ↓
Memory Model
 ↓
Bit Operation
 ↓
volatile
 ↓
MMIO
 ↓
Register
 ↓
Peripheral Driver
```

------------------------------------------------------------------------

# 55. 핵심 요약

``` mermaid
mindmap
  root((Pointer))
    기본
      Address
      &
      *
      Dereference
      Pointer Type
    Safety
      NULL
      Uninitialized
      Dangling
      Out of Bounds
    Function
      Address Argument
      Original Object
      scanf
    Array
      First Element
      Pointer Arithmetic
      array[i]
      *(array+i)
    String
      char pointer
      null terminator
    확장
      const pointer
      void pointer
      double pointer
    Memory
      Stack
      Heap
      MMIO
    Embedded
      Buffer
      Register
      UART
      DMA
```

## 최종적으로 기억할 일곱 문장

> **1. 포인터는 메모리 주소를 값으로 저장하는 객체다.**

> **2. `&variable`은 객체의 주소를 얻고 `*ptr`은 포인터가 가리키는
> 객체에 접근한다.**

> **3. 포인터 문제에서는 항상 값, 주소, 포인터, 가리키는 객체를
> 구분한다.**

> **4. 함수에 객체의 주소를 전달하면 포인터를 통해 호출자의 원본 객체를
> 변경할 수 있다.**

> **5. 배열과 포인터는 밀접하지만 배열 자체와 포인터 자체가 완전히 같은
> 것은 아니다.**

> **6. 잘못된 주소, 수명 또는 범위를 역참조하면 Undefined Behavior가
> 발생할 수 있다.**

> **7. 임베디드에서는 `포인터 → 주소 → MMIO → 레지스터 → 하드웨어`로
> 직접 연결된다.**

------------------------------------------------------------------------

# 56. 현재 학습 진행 상태

``` mermaid
flowchart LR
    A["Data_Type.md<br/>완료"] --> B["Operator.md<br/>완료"]
    B --> C["Control_Flow.md<br/>완료"]
    C --> D["Function.md<br/>완료"]
    D --> E["Array_String.md<br/>완료"]
    E --> F["Pointer.md<br/>현재"]
    F --> G["Memory_Model.md"]
```

``` text
C_Cpp_Study/
├── 00_Common/
│   ├── Data_Type.md       ✓
│   ├── Operator.md        ✓
│   ├── Control_Flow.md    ✓
│   └── Function.md        ✓
│
└── 01_C/
    ├── Array_String.md    ✓
    └── Pointer.md         ← 현재
```

------------------------------------------------------------------------

# 57. 다음 학습

포인터 다음에는 포인터가 가리키는 객체가 **어디에 저장되고 얼마나 오래
존재하는지**를 학습한다.

``` mermaid
flowchart LR
    A["Pointer"] --> B["Address"]
    B --> C["Object"]
    C --> D["Storage / Lifetime"]
    D --> E["Stack"]
    D --> F["Static Storage"]
    D --> G["Heap"]
```

다음 질문을 해결하게 된다.

``` text
지역 변수는 어디에 존재하는가?
함수가 끝나면 지역 변수는 어떻게 되는가?
전역/정적 객체는 얼마나 오래 존재하는가?
malloc으로 얻은 메모리는 어떻게 관리하는가?
Dangling Pointer는 왜 생기는가?
Memory Leak은 왜 생기는가?
```

**다음 문서:** `00_Common/Memory_Model.md`
