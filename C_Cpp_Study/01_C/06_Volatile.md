# volatile

> **학습 목표**
>
> C의 `volatile`이 **컴파일러에게 어떤 의미를 전달하는지** 이해하고,
> 임베디드의 **Memory-Mapped I/O(MMIO)** 및 Interrupt와 연결할 수 있어야
> 한다.
>
> 핵심 흐름:
>
> **Memory Access → Compiler Optimization → External Change → volatile →
> MMIO / ISR → Hardware**
>
> 위치: `C_Cpp_Study/01_C/volatile.md`

------------------------------------------------------------------------

# 1. 한 문장 정의

**`volatile`은 객체의 값이 프로그램의 일반적인 코드 흐름만으로는 예측할
수 없는 방식으로 변경될 수 있음을 컴파일러에 알리는 타입 한정자(type
qualifier)다.**

대표적인 임베디드 사례:

``` text
Hardware Register
Interrupt Service Routine에서 변경되는 상태
일부 저수준 시스템 인터페이스
```

``` mermaid
flowchart LR
    A["Program"] --> B["Compiler"]
    C["External Change"] --> D["Object"]
    B --> D
    D --> E["volatile access"]
```

------------------------------------------------------------------------

# 2. 왜 필요한가?

컴파일러는 프로그램의 의미를 유지하는 범위에서 코드를 최적화할 수 있다.

예를 들어 일반 변수의 값이 코드상 변경되지 않는다고 판단하면 반복적인
메모리 읽기를 줄이는 최적화를 수행할 수 있다.

하지만 하드웨어 Register처럼 **CPU 코드 외부의 원인으로 값이 바뀔 수
있는 대상**은 일반 변수와 다르다.

``` mermaid
flowchart TD
    A["Memory Location"] --> B{"누가 값을 바꾸는가?"}
    B -->|"일반 프로그램 코드"| C["Normal Object"]
    B -->|"Hardware 등 외부 요인"| D["volatile 고려"]
```

------------------------------------------------------------------------

# 3. 기본 문법

``` c
volatile int value;
```

고정 폭 정수형:

``` c
#include <stdint.h>

volatile uint32_t status;
```

`volatile`은 새로운 자료형 자체가 아니라 기존 타입에 성질을 추가하는
**type qualifier**다.

``` text
uint32_t
   +
volatile
   ↓
volatile uint32_t
```

------------------------------------------------------------------------

# 4. 일반 변수와 비교

일반 변수:

``` c
uint32_t status;
```

volatile 객체:

``` c
volatile uint32_t status;
```

개념적으로:

  -----------------------------------------------------------------------
  구분                    일반 객체               `volatile` 객체
  ----------------------- ----------------------- -----------------------
  외부 변화 고려          일반적으로 코드 의미에  외부 변화 가능성을 표현
                          따라 판단               

  접근 최적화             비교적 자유롭게 가능    volatile 접근의 관찰
                                                  가능한 동작을 보존해야
                                                  함

  대표 사례               일반 계산 데이터        MMIO 등
  -----------------------------------------------------------------------

> \[!IMPORTANT\] `volatile`을 단순히 **"최적화를 끄는 키워드"**라고
> 외우지 않는다. 프로그램 전체의 최적화를 비활성화하는 기능이 아니다.

------------------------------------------------------------------------

# 5. Polling 예제

하드웨어 상태를 기다리는 개념적인 코드:

``` c
while ((STATUS_REG & READY_MASK) == 0u)
{
}
```

`STATUS_REG`가 하드웨어에 의해 변경되는 Register라면 매 반복마다 상태를
관찰해야 한다.

``` mermaid
flowchart TD
    A["Read STATUS"] --> B{"READY?"}
    B -->|"No"| A
    B -->|"Yes"| C["Continue"]
```

이런 대상은 보통 volatile-qualified access가 필요하다.

------------------------------------------------------------------------

# 6. MMIO란?

Memory-Mapped I/O에서는 하드웨어 Peripheral Register가 CPU의 주소 공간에
배치된다.

개념:

``` text
Memory Address Space

0x....
+------------------+
| RAM              |
+------------------+
| ...              |
+------------------+
| Peripheral Regs  |
+------------------+
```

CPU는 특정 주소를 읽고 쓰는 방식으로 하드웨어를 제어한다.

``` mermaid
flowchart LR
    A["CPU"] --> B["Address"]
    B --> C["MMIO Register"]
    C --> D["Peripheral"]
```

------------------------------------------------------------------------

# 7. MMIO Register Pointer

개념적인 예:

``` c
#include <stdint.h>

volatile uint32_t *reg =
    (volatile uint32_t *)0x40000000u;
```

읽기:

``` c
uint32_t value = *reg;
```

쓰기:

``` c
*reg = value;
```

> 실제 MCU에서는 임의 주소를 사용하지 말고 제조사에서 제공하는 CMSIS/SDK
> 헤더와 Reference Manual을 따른다.

------------------------------------------------------------------------

# 8. 왜 Register에 `volatile`이 필요한가?

Register 값은 프로그램 코드 외부의 하드웨어에 의해 변경될 수 있다.

예:

``` text
UART receives byte
       ↓
Hardware changes STATUS register
       ↓
CPU reads STATUS
```

``` mermaid
sequenceDiagram
    participant H as Hardware
    participant R as Register
    participant C as CPU

    H->>R: Change Status Bit
    C->>R: volatile Read
    R-->>C: Current Value
```

따라서 컴파일러가 일반 RAM 변수처럼 값이 변하지 않는다고 가정하면 안
되는 대상이다.

------------------------------------------------------------------------

# 9. Bit Operation과 연결

이전 `Bit_Operation.md`에서 학습한 Mask를 Register에 적용한다.

``` c
#define ENABLE_MASK (UINT32_C(1) << 0)
```

개념적인 Register:

``` c
volatile uint32_t *control_reg;
```

Set:

``` c
*control_reg |= ENABLE_MASK;
```

Clear:

``` c
*control_reg &= ~ENABLE_MASK;
```

Test:

``` c
if ((*control_reg & ENABLE_MASK) != 0u)
{
}
```

``` mermaid
flowchart LR
    A["volatile Register"] --> B["Bit Mask"]
    B --> C["Set / Clear / Test"]
    C --> D["Hardware"]
```

------------------------------------------------------------------------

# 10. Read-Modify-Write

다음 코드:

``` c
*control_reg |= ENABLE_MASK;
```

개념적으로는:

``` text
1. Register Read
2. OR 연산
3. Register Write
```

``` mermaid
sequenceDiagram
    participant C as CPU
    participant R as Register

    C->>R: Read
    R-->>C: Current Value
    Note over C: OR mask
    C->>R: Write
```

`volatile`은 접근이 필요하다는 의미를 전달하지만, 이 전체 과정이 하나의
원자적(atomic) 연산이 된다는 뜻은 아니다.

------------------------------------------------------------------------

# 11. `volatile`은 Atomic이 아니다

다음 변수:

``` c
volatile uint32_t counter;
```

그리고:

``` c
counter++;
```

가 있다고 하자.

개념적으로 여러 단계가 필요할 수 있다.

``` text
Read counter
↓
Add 1
↓
Write counter
```

``` mermaid
flowchart LR
    A["Read"] --> B["+1"]
    B --> C["Write"]
```

`volatile`이라고 해서 이 과정이 다른 실행 주체에 의해 중간에 방해받지
않는다는 보장은 없다.

> **`volatile != atomic`**

------------------------------------------------------------------------

# 12. `volatile`은 Thread Synchronization이 아니다

`volatile`만으로 다음을 보장할 수 있다고 생각하면 안 된다.

``` text
Mutual Exclusion
Atomicity
Thread Synchronization
Memory Ordering between threads
Race Condition 해결
```

멀티스레드 동기화에는 언어/플랫폼에서 제공하는 적절한 atomic, mutex,
synchronization primitive 등을 사용한다.

``` mermaid
flowchart TD
    A["volatile"] --> B["volatile access semantics"]
    C["Synchronization"] --> D["Atomic / Mutex / RTOS Primitive"]
```

------------------------------------------------------------------------

# 13. ISR과 공유 변수

개념:

``` c
volatile int event_flag = 0;
```

ISR:

``` c
void interrupt_handler(void)
{
    event_flag = 1;
}
```

main:

``` c
while (event_flag == 0)
{
}
```

``` mermaid
sequenceDiagram
    participant M as Main
    participant I as ISR
    participant F as event_flag

    M->>F: Read
    I->>F: Write 1
    M->>F: Read again
```

`volatile`은 main 코드가 해당 객체에 대한 volatile access를 수행하도록
하는 데 중요할 수 있다.

------------------------------------------------------------------------

# 14. ISR 공유에서 `volatile`만으로 충분한가?

항상 그렇지는 않다.

예:

``` c
volatile uint32_t counter;
```

main과 ISR이 모두:

``` c
counter++;
```

를 수행한다면 Read-Modify-Write가 충돌할 수 있다.

``` mermaid
sequenceDiagram
    participant M as Main
    participant I as ISR
    participant X as counter

    M->>X: Read 10
    I->>X: Read 10
    I->>X: Write 11
    M->>X: Write 11
```

두 번 증가했지만 결과가 `11`이 될 수 있는 형태의 경쟁 문제가 생긴다.

필요한 해결책은 MCU, 데이터 폭, ISR 구조, RTOS 사용 여부 등에 따라
달라진다.

------------------------------------------------------------------------

# 15. `volatile sig_atomic_t`

표준 C에서 signal handler와 일반 실행 흐름 사이의 단순 flag 같은
경우에는 `<signal.h>`의 `volatile sig_atomic_t`가 관련된다.

``` c
#include <signal.h>

volatile sig_atomic_t flag;
```

다만 MCU의 하드웨어 Interrupt와 hosted C의 signal은 같은 개념이 아니다.

임베디드 ISR 동기화는 대상 MCU와 컴파일러, RTOS 규칙을 함께 확인해야
한다.

------------------------------------------------------------------------

# 16. Pointer와 `volatile`

다음 선언:

``` c
volatile uint32_t *ptr;
```

의 핵심 의미:

> `ptr`이 가리키는 `uint32_t` 객체가 volatile이다.

즉:

``` text
ptr
 ↓
volatile uint32_t object
```

``` mermaid
flowchart LR
    A["ptr"] --> B["volatile uint32_t"]
```

------------------------------------------------------------------------

# 17. Pointer 자체가 volatile인 경우

다음은 다르다.

``` c
uint32_t *volatile ptr;
```

여기서는 **포인터 변수 `ptr` 자체**가 volatile이다.

``` text
volatile pointer
      ↓
uint32_t object
```

------------------------------------------------------------------------

# 18. 둘 다 volatile

``` c
volatile uint32_t *volatile ptr;
```

의미:

``` text
pointer 자체도 volatile
+
가리키는 객체도 volatile
```

선언을 오른쪽과 왼쪽 관계로 천천히 읽는다.

------------------------------------------------------------------------

# 19. `const volatile`

하드웨어 상태 Register처럼 프로그램에서는 읽기만 하지만 하드웨어가 값을
변경할 수 있는 경우를 생각할 수 있다.

``` c
const volatile uint32_t status;
```

의미:

``` text
const
→ 이 C 코드에서 해당 lvalue를 통해 수정하지 않음

volatile
→ 외부 요인으로 값이 변할 수 있음
```

``` mermaid
flowchart LR
    A["const"] --> C["const volatile"]
    B["volatile"] --> C
    C --> D["Read-only view of externally changing object"]
```

------------------------------------------------------------------------

# 20. Read-only Register 개념

개념적인 선언:

``` c
const volatile uint32_t *status_reg;
```

읽기:

``` c
uint32_t status = *status_reg;
```

해당 타입을 통해 쓰기:

``` c
*status_reg = 1u;
```

는 허용되지 않는다.

하드웨어가 값을 변경하는 것과 프로그램이 해당 lvalue를 통해 수정하는
것은 다른 문제다.

------------------------------------------------------------------------

# 21. `const`와 `volatile` 비교

  -----------------------------------------------------------------------
  Qualifier                           핵심 의미
  ----------------------------------- -----------------------------------
  `const`                             해당 lvalue를 통한 수정을 제한

  `volatile`                          접근이 외부에서 관찰되거나 값이
                                      외부 요인으로 변할 수 있음을 표현

  `const volatile`                    코드에서는 수정하지 않지만 외부에서
                                      변할 수 있는 대상에 적합할 수 있음
  -----------------------------------------------------------------------

대표적 연결:

``` text
const          → Read-only interface
volatile       → Hardware / external change
const volatile → Status Register
```

------------------------------------------------------------------------

# 22. Register 구조체

MCU 헤더에서는 Peripheral Register 집합을 구조체로 표현하는 방식을 볼 수
있다.

개념 예:

``` c
typedef struct
{
    volatile uint32_t CONTROL;
    const volatile uint32_t STATUS;
    volatile uint32_t DATA;
} UART_Registers;
```

``` mermaid
flowchart TD
    A["UART_Registers"] --> B["CONTROL"]
    A --> C["STATUS"]
    A --> D["DATA"]
```

실제 제조사 헤더에서는 전용 매크로나 typedef가 사용될 수 있다.

------------------------------------------------------------------------

# 23. CMSIS 스타일 개념

ARM CMSIS 계열 코드에서는 구현에 따라 다음과 유사한 의미의 매크로를 볼
수 있다.

``` text
Read Only
Write Only
Read / Write
```

이들은 `volatile`, `const volatile` 등의 qualifier를 이용해 Register
접근 의도를 표현할 수 있다.

실제 정의는 사용하는 CMSIS 버전과 제조사 헤더를 확인한다.

------------------------------------------------------------------------

# 24. `volatile`과 캐싱 오해

다음 식으로 단순하게 외우지 않는다.

> "`volatile`이면 CPU Cache를 사용하지 않는다."

`volatile`은 C 언어 수준에서 volatile access의 의미를 규정하는 것이며
CPU Cache 정책 자체를 제어하는 범용 기능이 아니다.

``` mermaid
flowchart TD
    A["volatile"] --> B["Compiler / Language Access Semantics"]
    C["CPU Cache"] --> D["Architecture / Memory Attributes"]
```

MMU, MPU, Cache, Device Memory 속성은 CPU 아키텍처와 플랫폼 설정의 별도
주제다.

------------------------------------------------------------------------

# 25. `volatile`과 Memory Barrier 오해

`volatile`을 Memory Barrier와 동일하게 생각하면 안 된다.

``` text
volatile
≠
memory barrier
≠
atomic operation
≠
mutex
```

``` mermaid
flowchart LR
    A["volatile"] --> B["Access Semantics"]
    C["Barrier"] --> D["Ordering"]
    E["Atomic"] --> F["Atomic Operation"]
    G["Mutex"] --> H["Mutual Exclusion"]
```

이 차이는 RTOS와 멀티코어/동시성 학습에서 중요해진다.

------------------------------------------------------------------------

# 26. 컴파일러 최적화와 확인

학습용으로 다음과 같은 코드를 생각할 수 있다.

``` c
int flag = 0;

while (flag == 0)
{
}
```

그리고:

``` c
volatile int flag = 0;

while (flag == 0)
{
}
```

두 코드의 컴파일 결과는 최적화 옵션과 컴파일러에 따라 달라질 수 있다.

Assembly를 직접 확인하면 `volatile` access가 코드 생성에 어떤 영향을
주는지 관찰할 수 있다.

------------------------------------------------------------------------

# 27. Assembly 확인 예

예:

``` bash
gcc -std=c17 -O2 -S volatile_test.c
```

생성된 Assembly를 비교한다.

학습 목표:

``` text
C source
↓
Compiler Optimization
↓
Assembly
↓
Memory Access
```

> 특정 Assembly 형태가 반드시 생성된다고 암기하지 않는다. 컴파일러,
> 버전, 아키텍처, 옵션에 따라 달라질 수 있다.

------------------------------------------------------------------------

# 28. `volatile`을 남용하지 않는다

일반 계산 변수에 무조건 `volatile`을 붙이는 것은 좋은 해결책이 아니다.

잘못된 접근:

``` text
버그가 있다
↓
최적화 문제 같음
↓
모든 변수에 volatile
```

올바른 접근:

``` mermaid
flowchart TD
    A["Object"] --> B{"외부 요인으로 변경되거나 volatile access가 필요한가?"}
    B -->|"Yes"| C["volatile 고려"]
    B -->|"No"| D["일반 타입"]
```

`volatile`에는 명확한 이유가 있어야 한다.

------------------------------------------------------------------------

# 29. 자주 발생하는 오해

### 오해 1

``` text
volatile이면 thread-safe다.
```

아니다.

### 오해 2

``` text
volatile이면 atomic이다.
```

아니다.

### 오해 3

``` text
volatile이면 Cache를 끈다.
```

일반적으로 그런 의미가 아니다.

### 오해 4

``` text
volatile이면 모든 최적화가 꺼진다.
```

아니다.

### 오해 5

``` text
ISR과 공유하면 무조건 volatile 하나면 충분하다.
```

아니다. 공유 연산의 atomicity와 경쟁 상태도 별도로 분석해야 한다.

------------------------------------------------------------------------

# 30. Polling과 Timeout

다음과 같은 무한 polling은:

``` c
while ((*status_reg & READY_MASK) == 0u)
{
}
```

하드웨어가 READY 상태가 되지 않으면 영원히 빠져나오지 못할 수 있다.

실제 펌웨어에서는 요구사항에 따라 timeout을 고려한다.

``` mermaid
flowchart TD
    A["Read Status"] --> B{"Ready?"}
    B -->|"Yes"| C["Success"]
    B -->|"No"| D{"Timeout?"}
    D -->|"No"| A
    D -->|"Yes"| E["Error Handling"]
```

------------------------------------------------------------------------

# 31. Polling과 Interrupt

Polling:

``` text
CPU가 계속 상태 확인
```

Interrupt:

``` text
이벤트 발생
↓
Hardware가 CPU에 알림
↓
ISR 실행
```

``` mermaid
flowchart LR
    A["Event"] --> B{"Method"}
    B --> C["Polling"]
    B --> D["Interrupt"]
```

`volatile`은 두 방식 모두에서 하드웨어/비동기 상태와 연결될 수 있지만,
Interrupt 자체를 대신하는 기능은 아니다.

------------------------------------------------------------------------

# 32. `volatile`과 DMA

DMA는 CPU가 직접 데이터를 복사하지 않고 주변장치와 메모리 사이의 데이터
이동을 수행할 수 있다.

``` mermaid
flowchart LR
    A["Peripheral"] --> B["DMA"]
    B --> C["Memory"]
    D["CPU"] --> C
```

DMA와 CPU가 공유하는 메모리에서는 단순히 `volatile`만 추가한다고 모든
Cache coherency와 memory ordering 문제가 해결되는 것은 아니다.

실제 처리는 MCU의 Cache 구조, DMA 규칙, Memory Barrier, Cache
maintenance 요구사항을 확인해야 한다.

------------------------------------------------------------------------

# 33. `volatile`과 Bit Field

Register를 C bit-field 구조체로 표현하는 코드도 볼 수 있다.

하지만 C bit-field의 배치에는 구현 의존적인 요소가 있으므로 하드웨어
Register mapping에 사용할 때는 toolchain/ABI/제조사 정의를 따라야 한다.

직접 만든 범용 코드에서는 Mask 방식이 더 명시적인 경우가 많다.

``` c
reg |= ENABLE_MASK;
```

``` mermaid
flowchart LR
    A["Register Access"] --> B["Mask"]
    A --> C["C Bit-field"]
    C --> D["Implementation Details 주의"]
```

------------------------------------------------------------------------

# 34. 실습 1 --- 기본 선언

다음 선언의 의미를 각각 설명한다.

``` c
uint32_t value;
volatile uint32_t value;
const uint32_t value;
const volatile uint32_t value;
```

------------------------------------------------------------------------

# 35. 실습 2 --- Pointer 선언 읽기

다음 세 선언의 차이를 설명한다.

``` c
volatile uint32_t *a;
uint32_t *volatile b;
volatile uint32_t *volatile c;
```

질문:

``` text
가리키는 객체가 volatile인가?
포인터 자체가 volatile인가?
둘 다인가?
```

------------------------------------------------------------------------

# 36. 실습 3 --- Register Polling

다음 상수를 사용한다.

``` c
#define READY_MASK (UINT32_C(1) << 0)
```

개념적인 `status_reg`에서 READY bit가 설정될 때까지 기다리는 코드를
작성한다.

------------------------------------------------------------------------

# 37. 실습 4 --- Register Control

``` c
#define ENABLE_MASK (UINT32_C(1) << 3)
```

다음을 구현한다.

``` text
ENABLE Set
ENABLE Test
ENABLE Clear
```

`Bit_Operation.md`의 공식을 사용한다.

------------------------------------------------------------------------

# 38. 실습 5 --- ISR Flag

다음 구조를 완성한다.

``` c
volatile int event_flag = 0;

void interrupt_handler(void)
{
    /* event 발생 표시 */
}

int main(void)
{
    while (1)
    {
        if (/* event 발생 */)
        {
            /* event 처리 */
            /* flag clear */
        }
    }
}
```

질문:

> ISR에서는 어떤 작업을 최소화하고 main loop로 넘길 수 있는가?

------------------------------------------------------------------------

# 39. 실습 6 --- Atomicity 분석

다음 코드:

``` c
volatile uint32_t counter = 0;

void interrupt_handler(void)
{
    counter++;
}
```

main에서도:

``` c
counter++;
```

를 실행한다고 가정한다.

다음을 그림으로 설명한다.

``` text
Read
Modify
Write
```

그리고 왜 `volatile`만으로 경쟁 상태가 해결되지 않는지 설명한다.

------------------------------------------------------------------------

# 40. 실습 7 --- Read-only Status Register

프로그램에서는 읽기만 가능하고 하드웨어가 변경할 수 있는 32-bit status
register를 표현하기 위한 타입을 작성한다.

힌트:

``` text
const
+
volatile
```

------------------------------------------------------------------------

# 41. 코드 추적

``` c
volatile uint32_t status = 0u;

while ((status & 0x1u) == 0u)
{
}
```

질문:

  질문                                            답
  ----------------------------------------------- ----
  반복 조건은 무엇인가?                           
  어떤 bit를 검사하는가?                          
  `status`가 왜 volatile일 수 있는가?             
  `volatile`이 atomic을 보장하는가?               
  실제 하드웨어라면 timeout이 필요할 수 있는가?   

------------------------------------------------------------------------

# 42. 반드시 설명할 수 있어야 하는 것

-   [ ] `volatile`의 한 문장 정의
-   [ ] `volatile`이 type qualifier임을 설명
-   [ ] 일반 변수와 volatile 객체의 차이
-   [ ] 컴파일러 최적화와의 관계
-   [ ] MMIO의 기본 개념
-   [ ] 하드웨어 Register에서 volatile이 필요한 이유
-   [ ] `volatile uint32_t *ptr`의 의미
-   [ ] `uint32_t *volatile ptr`의 의미
-   [ ] `const volatile`의 의미
-   [ ] Polling과 volatile의 연결
-   [ ] ISR 공유 flag와 volatile의 연결
-   [ ] `volatile != atomic`
-   [ ] `volatile != thread synchronization`
-   [ ] `volatile != memory barrier`
-   [ ] `volatile != cache control`
-   [ ] Read-Modify-Write의 의미
-   [ ] Register 특수 동작을 데이터시트에서 확인해야 하는 이유
-   [ ] DMA에서는 volatile만으로 충분하지 않을 수 있음을 설명
-   [ ] volatile 남용을 피해야 하는 이유

------------------------------------------------------------------------

# 43. 복습 문제

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q1. volatile의 핵심 목적은?`</strong>`{=html}
```{=html}
</summary>
```
객체의 값이 일반적인 프로그램 흐름 외부의 요인으로 변경될 수 있거나 접근
자체가 의미를 갖는 경우, 해당 volatile access를 컴파일러가 적절히
보존하도록 표현하는 것이다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q2. volatile은 프로그램 전체 최적화를
끄는가?`</strong>`{=html}
```{=html}
</summary>
```
아니다. volatile-qualified 객체에 대한 접근의 의미와 관련된 것이며
프로그램 전체 최적화를 비활성화하는 기능이 아니다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q3. 하드웨어 Register에 volatile이 사용되는 대표적인
이유는?`</strong>`{=html}
```{=html}
</summary>
```
Register 값이 하드웨어에 의해 변경될 수 있고, Register에 대한 읽기/쓰기
자체가 하드웨어 동작과 연결될 수 있기 때문이다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q4. volatile이면 atomic한가?`</strong>`{=html}
```{=html}
</summary>
```
아니다. `counter++` 같은 연산은 여러 단계의 Read-Modify-Write가 될 수
있다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q5. volatile이면 thread-safe한가?`</strong>`{=html}
```{=html}
</summary>
```
아니다. 경쟁 상태, atomicity, mutual exclusion, thread synchronization은
별도로 해결해야 한다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q6. volatile이 CPU Cache를 끄는가?`</strong>`{=html}
```{=html}
</summary>
```
일반적으로 아니다. Cache 정책과 memory attribute는 CPU 아키텍처와 플랫폼
설정의 별도 문제다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q7. `volatile uint32_t *ptr`의
의미는?`</strong>`{=html}
```{=html}
</summary>
```
`ptr`이 가리키는 `uint32_t` 객체가 volatile이라는 의미다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q8. `uint32_t *volatile ptr`의
의미는?`</strong>`{=html}
```{=html}
</summary>
```
포인터 변수 `ptr` 자체가 volatile이라는 의미다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q9. `const volatile`은 언제 유용할 수
있는가?`</strong>`{=html}
```{=html}
</summary>
```
프로그램에서는 해당 lvalue를 통해 수정하지 않지만 하드웨어 등 외부
요인에 의해 값이 변경될 수 있는 Status Register 같은 대상에 적합할 수
있다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q10. ISR 공유 변수에 volatile만 붙이면 모든 문제가
해결되는가?`</strong>`{=html}
```{=html}
</summary>
```
아니다. volatile access와 별개로 연산의 atomicity, 경쟁 상태, Interrupt
제어 또는 플랫폼의 동기화 방법을 검토해야 한다.

```{=html}
</details>
```

------------------------------------------------------------------------

# 44. Bug Log

`99_Bug_Log/Memory_Bugs.md` 또는 별도의 `Volatile_Bugs.md`에 기록한다.

```` markdown
## volatile을 atomic으로 오해

### 코드

```c
volatile uint32_t counter;

counter++;
```

### 잘못된 생각

volatile이므로 counter++ 전체가 atomic하다.

### 실제 문제

counter++는 Read → Modify → Write 과정이 될 수 있다.
다른 실행 주체가 중간에 접근하면 경쟁 상태가 발생할 수 있다.

### 재발 방지

volatile, atomicity, synchronization을 서로 다른 개념으로 구분한다.
````

추천 항목:

``` text
Missing volatile on MMIO
volatile Everywhere
volatile == atomic 오해
volatile == mutex 오해
volatile == cache control 오해
ISR Shared Counter Race
Register RMW Race
DMA Coherency 오해
```

------------------------------------------------------------------------

# 45. `volatile` 판단 흐름

``` mermaid
flowchart TD
    A["Object 확인"] --> B{"값/접근이 외부 요인과 연결되는가?"}
    B -->|"No"| C["일반 객체"]
    B -->|"Yes"| D["volatile 필요성 검토"]
    D --> E{"공유 동시성도 있는가?"}
    E -->|"No"| F["volatile semantics 적용"]
    E -->|"Yes"| G["Atomicity / Synchronization 추가 검토"]
    G --> H["MCU / RTOS / Compiler 규칙 확인"]
```

------------------------------------------------------------------------

# 46. 이전 학습과 연결

``` mermaid
flowchart LR
    A["Pointer"] --> D["MMIO Address"]
    B["Bit Operation"] --> E["Register Mask"]
    C["Memory Model"] --> D
    D --> F["volatile"]
    E --> F
    F --> G["Hardware Access"]
```

  기존 개념       `volatile`에서의 역할
  --------------- ---------------------------
  Data Type       Register 데이터 폭
  Pointer         MMIO 주소 접근
  Memory Model    주소 공간과 객체 수명
  Bit Operation   Register bit 조작
  Struct          Peripheral Register Block
  Function        Driver API

------------------------------------------------------------------------

# 47. 임베디드 연결

``` mermaid
flowchart TD
    A["volatile"] --> B["MMIO"]
    A --> C["Polling"]
    A --> D["ISR Shared State"]

    B --> E["GPIO"]
    B --> F["UART"]
    B --> G["Timer"]

    D --> H["Interrupt"]
    H --> I["RTOS / Concurrency"]
```

`volatile`은 임베디드 C에서 매우 중요하지만 혼자 모든 동시성 문제를
해결하는 기능은 아니다.

------------------------------------------------------------------------

# 48. 핵심 요약

``` mermaid
mindmap
  root((volatile))
    Purpose
      External Change
      Observable Access
    Embedded
      MMIO
      Register
      Polling
      ISR
    Qualifiers
      const
      volatile
      const volatile
    Not Guaranteed
      Atomicity
      Thread Safety
      Mutex
      Memory Barrier
      Cache Control
    Advanced
      DMA
      RMW Race
      RTOS
```

## 최종적으로 기억할 아홉 문장

> **1. `volatile`은 값이 외부 요인으로 변경될 수 있거나 접근 자체가
> 의미를 갖는 객체에 사용되는 type qualifier다.**

> **2. `volatile`은 프로그램 전체 최적화를 끄는 키워드가 아니다.**

> **3. Memory-Mapped Hardware Register는 `volatile`의 대표적인
> 사용처다.**

> **4. `volatile uint32_t *ptr`은 가리키는 객체가 volatile이라는
> 뜻이다.**

> **5. `const volatile`은 코드에서는 수정하지 않지만 외부에서 변경될 수
> 있는 상태 Register 등에 사용할 수 있다.**

> **6. `volatile`은 atomicity를 보장하지 않는다.**

> **7. `volatile`은 thread synchronization이나 mutex를 대신하지
> 않는다.**

> **8. `volatile`은 CPU Cache 제어나 Memory Barrier와 동일한 개념이
> 아니다.**

> **9. 임베디드에서는
> `Pointer → Bit Mask → volatile Register → Hardware`의 연결을 이해해야
> 한다.**

------------------------------------------------------------------------

# 49. 현재 학습 진행 상태

``` mermaid
flowchart LR
    A["malloc_free.md<br/>완료"] --> B["Bit_Operation.md<br/>완료"]
    B --> C["volatile.md<br/>현재"]
    C --> D["Embedded_C.md"]
```

``` text
C_Cpp_Study/
└── 01_C/
    ├── Array_String.md    ✓
    ├── Pointer.md         ✓
    ├── Struct.md          ✓
    ├── malloc_free.md     ✓
    ├── Bit_Operation.md   ✓
    ├── volatile.md        ← 현재
    └── Embedded_C.md
```

------------------------------------------------------------------------

# 50. 다음 학습

다음 단계는 **Embedded C**다.

``` mermaid
flowchart LR
    A["Pointer"] --> E["Embedded C"]
    B["Memory Model"] --> E
    C["Bit Operation"] --> E
    D["volatile"] --> E
    E --> F["Register"]
    F --> G["GPIO / UART / Timer / Interrupt"]
```

다음 문서에서는 지금까지 학습한 C 개념을 실제 펌웨어 구조로 통합한다.

``` text
Fixed-width Integer
Register
MMIO
Bit Mask
volatile
Polling
Interrupt 기초
Driver Layer
Header / Source 분리
Error Handling
Static Buffer
Firmware Main Loop
```

**다음 문서:** `01_C/Embedded_C.md`
