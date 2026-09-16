# Bit Operation

> **학습 목표**
>
> C의 정수 값을 **bit 단위로 해석하고 조작**할 수 있어야 한다.
>
> 핵심 흐름:
>
> **정수 → 이진 표현 → Bitwise Operator → Shift → Mask → Set / Clear /
> Toggle / Test → Register → Hardware**
>
> 위치: `C_Cpp_Study/01_C/Bit_Operation.md`

------------------------------------------------------------------------

# 1. 왜 Bit Operation을 배우는가?

일반적인 프로그램에서는 정수를 하나의 숫자로 보는 경우가 많다.

``` c
unsigned int value = 10;
```

하지만 임베디드에서는 하나의 정수 안에서 각각의 bit가 서로 다른 하드웨어
상태를 의미할 수 있다.

``` text
bit 0 → LED Enable
bit 1 → UART Enable
bit 2 → Interrupt Enable
bit 3 → Error Flag
```

``` mermaid
flowchart LR
    A["Integer"] --> B["Bits"]
    B --> C["Flags"]
    C --> D["Register"]
    D --> E["Hardware"]
```

Bit Operation은 코딩 테스트에서도 비트마스크, 부분집합, 상태 압축 등에
사용된다.

------------------------------------------------------------------------

# 2. Bit와 Byte

bit는 0 또는 1을 표현한다.

``` text
0
1
```

여러 bit를 묶어 값을 표현한다.

예:

``` text
0000 1010
```

이는 이진수 `1010₂`, 즉 십진수 `10`이다.

> \[!NOTE\] C/C++ 표준에서 `sizeof(char) == 1`은 항상 성립하지만, 한
> byte가 반드시 8 bit라고 언어 표준 자체가 보장하는 것은 아니다.
> 일반적인 현대 MCU와 PC 환경에서는 8-bit byte가 널리 사용된다.

------------------------------------------------------------------------

# 3. 2진수와 16진수

bit를 직접 다룰 때 16진수를 자주 사용한다.

``` text
Binary       Hex
0000         0
0001         1
0010         2
0011         3
0100         4
...
1111         F
```

4 bit는 16진수 한 자리와 대응한다.

``` text
Binary : 1010 1100
Hex    :   A    C
         = 0xAC
```

``` mermaid
flowchart LR
    A["4 bits"] --> B["1 Hex Digit"]
    C["8 bits"] --> D["2 Hex Digits"]
```

------------------------------------------------------------------------

# 4. C의 정수 리터럴

십진수:

``` c
unsigned int a = 10u;
```

16진수:

``` c
unsigned int b = 0x0Au;
```

둘은 같은 값을 나타낼 수 있다.

``` text
10 decimal
=
0x0A hexadecimal
=
0000 1010 binary representation
```

비트 연산에서는 16진수 표기가 읽기 편한 경우가 많다.

------------------------------------------------------------------------

# 5. Bitwise Operator

주요 비트 연산자:

  연산자   이름          역할
  -------- ------------- -----------------------
  `&`      Bitwise AND   둘 다 1인 bit만 1
  `|`      Bitwise OR    하나라도 1이면 1
  `^`      Bitwise XOR   서로 다르면 1
  `~`      Bitwise NOT   bit 반전
  `<<`     Left Shift    bit를 왼쪽으로 이동
  `>>`     Right Shift   bit를 오른쪽으로 이동

``` mermaid
mindmap
  root((Bit Operation))
    AND
      &
    OR
      |
    XOR
      ^
    NOT
      ~
    Shift
      <<
      >>
```

------------------------------------------------------------------------

# 6. Bitwise AND `&`

``` text
A : 1010
B : 1100
-----------
& : 1000
```

각 bit 위치에서 둘 다 `1`일 때만 결과가 `1`이다.

``` c
unsigned int result = 0xAu & 0xCu;
```

``` text
0xA = 1010
0xC = 1100
------------
      1000 = 0x8
```

AND는 특정 bit를 검사하거나 특정 bit를 0으로 만들 때 매우 중요하다.

------------------------------------------------------------------------

# 7. Bitwise OR `|`

``` text
A : 1010
B : 1100
-----------
| : 1110
```

둘 중 하나라도 `1`이면 결과가 `1`이다.

``` c
unsigned int result = 0xAu | 0xCu;
```

결과:

``` text
1110 = 0xE
```

OR는 특정 bit를 `1`로 만드는 **Set** 연산에 사용한다.

------------------------------------------------------------------------

# 8. Bitwise XOR `^`

``` text
A : 1010
B : 1100
-----------
^ : 0110
```

두 bit가 서로 다르면 `1`이다.

XOR는 특정 bit를 반전시키는 **Toggle**에 사용한다.

------------------------------------------------------------------------

# 9. Bitwise NOT `~`

``` text
A  : 0000 1111
~A : 1111 0000
```

모든 bit를 반전한다.

``` c
unsigned int result = ~value;
```

> \[!IMPORTANT\] `~` 결과는 피연산자의 실제 정수 타입과 정수
> 승격(Integer Promotion)의 영향을 받는다. 화면에 그린 8 bit만
> 뒤집힌다고 단순하게 가정하지 않는다.

------------------------------------------------------------------------

# 10. Logical Operator와 구분

논리 연산:

``` c
&&
||
!
```

비트 연산:

``` c
&
|
^
~
```

예:

``` c
if (a && b)
{
}
```

는 논리 조건을 평가한다.

``` c
unsigned int c = a & b;
```

는 각 bit를 AND한다.

  목적        연산자
  ----------- --------------------
  조건 결합   `&&`, `||`, `!`
  bit 조작    `&`, `|`, `^`, `~`

------------------------------------------------------------------------

# 11. Left Shift `<<`

``` c
1u << 0
```

``` text
0000 0001
```

``` c
1u << 1
```

``` text
0000 0010
```

``` c
1u << 3
```

``` text
0000 1000
```

``` mermaid
flowchart LR
    A["1u"] --> B["1u << 1"]
    B --> C["1u << 2"]
    C --> D["1u << 3"]
```

특정 bit 위치에 `1`을 만드는 Mask를 생성할 때 핵심적으로 사용한다.

------------------------------------------------------------------------

# 12. 왜 `1u`를 사용하는가?

다음과 같이 unsigned 값을 사용한다.

``` c
1u << bit
```

비트 연산에서는 signed 정수의 부호와 overflow 관련 문제를 피하기 위해
unsigned 타입을 사용하는 습관이 중요하다.

예:

``` c
uint32_t mask = UINT32_C(1) << bit;
```

고정 폭 정수형을 다룰 때는 `<stdint.h>`의 타입과 관련 매크로를 사용할
수도 있다.

------------------------------------------------------------------------

# 13. Shift 주의사항

Shift는 아무 값이나 사용할 수 있는 것이 아니다.

다음과 같은 코드는 피해야 한다.

``` c
value << 32
```

`value` 타입의 bit 폭이 32라고 가정하면 shift count가 유효 범위를
벗어난다.

일반적인 규칙:

``` text
shift count >= 0
shift count < promoted left operand의 bit width
```

유효 범위를 벗어난 shift는 정의되지 않은 동작을 일으킬 수 있다.

> 비트 조작에서는 **타입의 폭과 signed/unsigned 여부**를 항상 확인한다.

------------------------------------------------------------------------

# 14. Right Shift `>>`

unsigned 예:

``` text
value      : 1000 0000
value >> 1 : 0100 0000
```

``` c
unsigned int result = value >> 1;
```

unsigned 정수에서는 왼쪽에 0이 채워지는 형태로 이해할 수 있다.

signed 음수의 right shift 결과는 구현에 따라 달라질 수 있으므로 bit
manipulation에서는 unsigned 타입을 우선 사용한다.

------------------------------------------------------------------------

# 15. Bit Mask

Mask는 특정 bit 또는 bit 집합을 선택하기 위한 값이다.

예:

``` c
unsigned int mask = 1u << 3;
```

개념:

``` text
bit index : 7 6 5 4 3 2 1 0
mask      : 0 0 0 0 1 0 0 0
```

``` mermaid
flowchart LR
    A["Bit Index"] --> B["1u << index"]
    B --> C["Mask"]
    C --> D["Set / Clear / Toggle / Test"]
```

------------------------------------------------------------------------

# 16. Bit 번호

일반적으로 가장 오른쪽 bit를 bit 0으로 표현한다.

``` text
bit index
7 6 5 4 3 2 1 0
│ │ │ │ │ │ │ │
0 0 0 0 1 0 1 0
```

예:

``` c
1u << 0
1u << 1
1u << 2
1u << 3
```

------------------------------------------------------------------------

# 17. 특정 Bit Set

특정 bit를 `1`로 만든다.

공식:

``` c
value |= (1u << bit);
```

예: bit 3 Set

``` c
value |= (1u << 3);
```

``` text
Before : 0000 0010
Mask   : 0000 1000
OR     : 0000 1010
```

``` mermaid
flowchart LR
    A["value"] --> C["OR"]
    B["1u << bit"] --> C
    C --> D["bit = 1"]
```

------------------------------------------------------------------------

# 18. 특정 Bit Clear

특정 bit를 `0`으로 만든다.

공식:

``` c
value &= ~(1u << bit);
```

예: bit 3 Clear

``` text
Mask       : 0000 1000
~Mask      : 1111 0111
Value      : 0000 1010
AND        : 0000 0010
```

``` mermaid
flowchart LR
    A["1u << bit"] --> B["~"]
    B --> C["Clear Mask"]
    D["value"] --> E["AND"]
    C --> E
    E --> F["bit = 0"]
```

------------------------------------------------------------------------

# 19. 특정 Bit Toggle

특정 bit를 반전한다.

공식:

``` c
value ^= (1u << bit);
```

현재 값이:

``` text
0 → 1
1 → 0
```

으로 바뀐다.

``` mermaid
flowchart LR
    A["value"] --> C["XOR"]
    B["1u << bit"] --> C
    C --> D["Toggle"]
```

------------------------------------------------------------------------

# 20. 특정 Bit Test

특정 bit가 `1`인지 확인한다.

``` c
if ((value & (1u << bit)) != 0u)
{
    /* bit is set */
}
```

흐름:

``` mermaid
flowchart TD
    A["value"] --> B["AND mask"]
    C["1u << bit"] --> B
    B --> D{"result != 0?"}
    D -->|"Yes"| E["Set"]
    D -->|"No"| F["Clear"]
```

------------------------------------------------------------------------

# 21. 네 가지 핵심 공식

  목적     공식
  -------- ------------------------
  Set      `value |= mask;`
  Clear    `value &= ~mask;`
  Toggle   `value ^= mask;`
  Test     `(value & mask) != 0u`

bit 번호를 사용하면:

``` c
value |=  (1u << bit);   /* Set */
value &= ~(1u << bit);   /* Clear */
value ^=  (1u << bit);   /* Toggle */

if ((value & (1u << bit)) != 0u)
{
    /* Test */
}
```

> 이 네 가지는 직접 손으로 여러 번 작성해 익힌다.

------------------------------------------------------------------------

# 22. 여러 Bit를 한 번에 Mask

예:

``` c
unsigned int mask =
    (1u << 1) |
    (1u << 3) |
    (1u << 5);
```

개념:

``` text
bit : 7 6 5 4 3 2 1 0
mask: 0 0 1 0 1 0 1 0
```

Set:

``` c
value |= mask;
```

Clear:

``` c
value &= ~mask;
```

------------------------------------------------------------------------

# 23. 이름 있는 Mask

숫자를 코드에 반복하기보다 의미 있는 이름을 붙인다.

``` c
#define UART_ENABLE_MASK      (1u << 0)
#define UART_TX_ENABLE_MASK   (1u << 1)
#define UART_RX_ENABLE_MASK   (1u << 2)
```

사용:

``` c
control |= UART_ENABLE_MASK;
control |= UART_RX_ENABLE_MASK;
```

``` mermaid
flowchart LR
    A["Magic Bit Number"] --> B["Named Mask"]
    B --> C["Readable Register Code"]
```

------------------------------------------------------------------------

# 24. 여러 Flag 조합

``` c
#define FLAG_A (1u << 0)
#define FLAG_B (1u << 1)
#define FLAG_C (1u << 2)
```

조합:

``` c
unsigned int mask = FLAG_A | FLAG_C;
```

``` text
FLAG_A : 001
FLAG_C : 100
OR     : 101
```

------------------------------------------------------------------------

# 25. Flag Test

하나의 Flag:

``` c
if ((flags & FLAG_A) != 0u)
{
}
```

두 Flag가 모두 설정되었는지:

``` c
unsigned int required = FLAG_A | FLAG_C;

if ((flags & required) == required)
{
}
```

"하나라도 설정"과 "모두 설정"을 구분한다.

------------------------------------------------------------------------

# 26. Register와 Bit

임베디드 레지스터는 여러 제어 bit를 하나의 정수 값으로 표현하는 경우가
많다.

개념:

``` text
CONTROL REGISTER

bit 7 6 5 4 3 2 1 0
    │ │ │ │ │ │ │ │
    │ │ │ │ │ │ │ └─ ENABLE
    │ │ │ │ │ │ └─── MODE
    │ │ │ │ │ └───── IRQ_ENABLE
    ...
```

``` mermaid
flowchart LR
    A["Register"] --> B["Bit Fields"]
    B --> C["Peripheral Configuration"]
```

------------------------------------------------------------------------

# 27. Register Bit Set

개념적인 레지스터:

``` c
volatile uint32_t *control_reg;
```

bit 3을 Set:

``` c
*control_reg |= (UINT32_C(1) << 3);
```

``` mermaid
flowchart LR
    A["Register Value"] --> B["OR Mask"]
    B --> C["Register Write"]
    C --> D["Hardware State Change"]
```

실제 주소와 bit 정의는 반드시 MCU 데이터시트/제조사 헤더를 따른다.

------------------------------------------------------------------------

# 28. Register Bit Clear

``` c
*control_reg &= ~(UINT32_C(1) << 3);
```

개념:

``` text
Read Register
 ↓
Clear Mask AND
 ↓
Write Register
```

단, 모든 하드웨어 레지스터에서 Read-Modify-Write가 안전한 것은 아니다.

> \[!WARNING\] 일부 레지스터는 Write-1-to-Clear, Write-only,
> Read-to-Clear 등의 특수 동작을 가진다. 실제 하드웨어에서는
> 데이터시트의 레지스터 동작 규칙을 우선한다.

------------------------------------------------------------------------

# 29. Read-Modify-Write

다음 표현:

``` c
reg |= mask;
```

개념적으로는:

``` text
1. reg 읽기
2. mask와 OR
3. 결과를 reg에 쓰기
```

``` mermaid
sequenceDiagram
    participant C as CPU
    participant R as Register

    C->>R: Read
    R-->>C: Current Value
    Note over C: OR with mask
    C->>R: Write New Value
```

임베디드에서는 Interrupt나 다른 실행 주체가 같은 값을 변경할 수 있다면
원자성 문제도 고려해야 한다.

이 내용은 이후 `volatile`, Interrupt, RTOS에서 확장한다.

------------------------------------------------------------------------

# 30. Bit Field 추출

여러 bit가 하나의 값을 표현할 수도 있다.

예:

``` text
bits 4~2 = MODE
```

Mask:

``` text
0001 1100
```

추출:

``` c
unsigned int mode =
    (value >> 2) & 0x7u;
```

흐름:

``` mermaid
flowchart LR
    A["value"] --> B["Right Shift"]
    B --> C["AND Mask"]
    C --> D["Field Value"]
```

------------------------------------------------------------------------

# 31. Field 추출 일반 형태

Field가 `position`에서 시작하고 `width` bit라고 가정한다.

개념:

``` c
field = (value >> position) & mask;
```

예: 3 bit field

``` c
unsigned int field = (value >> 2) & 0x7u;
```

`0x7`:

``` text
0000 0111
```

------------------------------------------------------------------------

# 32. Field 값 설정

특정 bit field를 새 값으로 변경하려면 일반적으로:

``` text
1. 기존 field Clear
2. 새 값 위치 이동
3. OR
```

예:

``` c
#define MODE_POS   2u
#define MODE_MASK  (0x7u << MODE_POS)

value &= ~MODE_MASK;
value |= ((new_mode << MODE_POS) & MODE_MASK);
```

``` mermaid
flowchart TD
    A["Original Value"] --> B["Clear Field"]
    C["New Field Value"] --> D["Shift"]
    D --> E["Mask"]
    B --> F["OR"]
    E --> F
    F --> G["Updated Value"]
```

------------------------------------------------------------------------

# 33. 왜 마지막에 Mask하는가?

``` c
(new_mode << MODE_POS) & MODE_MASK
```

`new_mode`에 field 폭보다 큰 값이 들어왔을 때 주변 bit까지 침범하는 것을
제한하는 데 도움이 된다.

하지만 API 수준에서 입력 범위도 별도로 검사하는 것이 좋다.

------------------------------------------------------------------------

# 34. Bit Macro 예

``` c
#define BIT(n) (1u << (n))
```

사용:

``` c
flags |= BIT(3);
flags &= ~BIT(3);
flags ^= BIT(3);
```

> \[!WARNING\] `n`이 타입의 유효 shift 범위를 벗어나면 문제가 된다.
> Macro를 사용한다고 범위 문제가 사라지는 것은 아니다.

------------------------------------------------------------------------

# 35. 괄호가 중요한 이유

Macro:

``` c
#define BIT(n) (1u << (n))
```

처럼 인자와 전체 표현에 괄호를 사용한다.

비트 연산과 비교 연산을 함께 사용할 때도 괄호를 적극 사용한다.

권장:

``` c
if ((flags & MASK) != 0u)
{
}
```

의도가 명확하고 연산자 우선순위 실수를 줄인다.

------------------------------------------------------------------------

# 36. Signed보다 Unsigned

비트 패턴을 조작할 때는 보통 unsigned 정수형이 더 적합하다.

예:

``` c
unsigned int flags;
uint32_t reg;
```

이유:

``` text
부호 bit 해석을 피함
unsigned shift 의미가 더 명확함
모듈러 정수 연산 규칙 활용
레지스터 타입과 자연스럽게 연결
```

------------------------------------------------------------------------

# 37. `<stdint.h>`

임베디드에서는 다음 타입을 자주 사용한다.

``` c
#include <stdint.h>

uint8_t
uint16_t
uint32_t
uint64_t
```

예:

``` c
uint32_t flags = 0u;
```

정확한 폭의 타입은 해당 구현이 그 폭을 지원할 때 제공된다.

------------------------------------------------------------------------

# 38. `UINT32_C`

32-bit unsigned 상수를 명시적으로 표현해야 할 때 `<stdint.h>`의 매크로를
사용할 수 있다.

``` c
uint32_t mask = UINT32_C(1) << 31;
```

단순 학습 코드에서는:

``` c
1u << bit
```

도 자주 사용하지만, 타입 폭과 상수 타입을 의식하는 습관을 만든다.

------------------------------------------------------------------------

# 39. Endianness와 Bit Number는 다른 개념

Endianness는 multi-byte 값의 **byte 순서**와 관련된다.

Bit mask는 정수 값의 bit 위치를 다룬다.

``` mermaid
flowchart TD
    A["Integer Representation"] --> B["Bit Position"]
    A --> C["Byte Order"]
    B --> D["Bit Mask"]
    C --> E["Endianness"]
```

둘을 같은 개념으로 혼동하지 않는다.

Endianness는 이후 통신/메모리 학습에서 확장한다.

------------------------------------------------------------------------

# 40. 코딩 테스트와 Bitmask

예를 들어 `N`개의 선택 여부를 하나의 정수 상태로 표현할 수 있다.

``` text
bit 0 → item 0 선택
bit 1 → item 1 선택
bit 2 → item 2 선택
...
```

``` mermaid
flowchart LR
    A["Set / State"] --> B["Integer Bits"]
    B --> C["Bitmask"]
    C --> D["Subset Enumeration"]
```

------------------------------------------------------------------------

# 41. 부분집합 순회 예고

`N`이 작다면:

``` c
for (unsigned int mask = 0;
     mask < (1u << n);
     mask++)
{
    /* mask가 하나의 부분집합 상태 */
}
```

각 원소 포함 여부:

``` c
if ((mask & (1u << i)) != 0u)
{
    /* i번째 원소 포함 */
}
```

> `n`은 사용한 정수 타입의 유효 shift 범위 안에 있어야 한다.

------------------------------------------------------------------------

# 42. 상태 압축

Boolean 상태 여러 개를 각각 변수로 둘 수도 있다.

``` c
int flag_a;
int flag_b;
int flag_c;
```

또는 bit flag로 표현할 수 있다.

``` c
unsigned int flags;
```

``` mermaid
flowchart LR
    A["Many Boolean States"] --> B["Bit Flags"]
    B --> C["Compact State"]
```

다만 읽기 쉬운 이름 있는 Mask를 함께 사용하는 것이 중요하다.

------------------------------------------------------------------------

# 43. 실습 1 --- AND / OR / XOR

다음 결과를 손으로 계산한다.

``` text
A = 1010
B = 1100
```

계산:

``` text
A & B
A | B
A ^ B
```

그 후 C 코드로 확인한다.

------------------------------------------------------------------------

# 44. 실습 2 --- Shift

다음 결과를 예상한다.

``` c
1u << 0
1u << 1
1u << 2
1u << 3
```

이진수와 십진수로 각각 기록한다.

------------------------------------------------------------------------

# 45. 실습 3 --- Set

초기값:

``` text
0000 0000
```

bit 3을 Set하여:

``` text
0000 1000
```

을 만든다.

코드:

``` c
value |= (1u << 3);
```

------------------------------------------------------------------------

# 46. 실습 4 --- Clear

초기값:

``` text
0000 1111
```

bit 2를 Clear한다.

목표:

``` text
0000 1011
```

------------------------------------------------------------------------

# 47. 실습 5 --- Toggle

bit 1을 반복해서 Toggle한다.

``` text
0 → 1 → 0 → 1
```

코드:

``` c
value ^= (1u << 1);
```

------------------------------------------------------------------------

# 48. 실습 6 --- Test

``` c
unsigned int flags = 0x0Au;
```

다음을 확인한다.

``` text
bit 1은 Set인가?
bit 2는 Set인가?
bit 3은 Set인가?
```

------------------------------------------------------------------------

# 49. 실습 7 --- Named Flags

다음을 정의한다.

``` c
#define FLAG_POWER   (1u << 0)
#define FLAG_READY   (1u << 1)
#define FLAG_ERROR   (1u << 2)
```

연습:

``` text
POWER Set
READY Set
ERROR Test
READY Clear
POWER Toggle
```

------------------------------------------------------------------------

# 50. 실습 8 --- Field 추출

다음 값이 있다고 가정한다.

``` text
value = 1011 0100
```

bits `4~2`를 추출한다.

절차:

``` text
Right Shift
↓
Mask
↓
Field Value
```

코드 형태:

``` c
field = (value >> 2) & 0x7u;
```

------------------------------------------------------------------------

# 51. 실습 9 --- Register Simulation

실제 하드웨어 대신 일반 변수로 레지스터를 흉내 낸다.

``` c
#include <stdint.h>

uint32_t control = 0u;

#define CTRL_ENABLE (UINT32_C(1) << 0)
#define CTRL_IRQ    (UINT32_C(1) << 3)
```

다음을 구현한다.

``` text
ENABLE Set
IRQ Set
ENABLE Test
IRQ Clear
```

------------------------------------------------------------------------

# 52. 코드 추적

``` c
unsigned int value = 0u;

value |= (1u << 1);
value |= (1u << 3);
value ^= (1u << 1);
value &= ~(1u << 3);
```

표를 채운다.

  단계           Binary     Decimal
  -------------- -------- ---------
  초기                    
  bit 1 Set               
  bit 3 Set               
  bit 1 Toggle            
  bit 3 Clear             

------------------------------------------------------------------------

# 53. 자주 발생하는 실수

### 논리 연산과 비트 연산 혼동

``` c
flags && mask
```

와:

``` c
flags & mask
```

는 다르다.

### Clear에서 `~` 누락

잘못된 의도:

``` c
value &= mask;
```

특정 bit 하나만 Clear하려는 것이라면 보통:

``` c
value &= ~mask;
```

가 필요하다.

### 괄호 부족

권장:

``` c
if ((value & mask) != 0u)
```

### signed shift 남용

bit 조작에는 unsigned 타입을 우선 고려한다.

### 유효 범위를 벗어난 Shift

``` c
1u << bit
```

에서 `bit`가 타입 폭 이상이면 안 된다.

------------------------------------------------------------------------

# 54. Register 코드에서 추가 주의

일반 RAM 변수와 하드웨어 Register는 동작 특성이 다를 수 있다.

다음 코드를 기계적으로 모든 Register에 적용하면 안 된다.

``` c
REG |= MASK;
```

Register가 다음 특성을 가질 수 있기 때문이다.

``` text
Read-only
Write-only
Write-1-to-Clear
Read-to-Clear
Reserved Bits
Self-clearing Bits
```

> 실제 Register 조작은 반드시 데이터시트 또는 Reference Manual의 설명을
> 확인한다.

------------------------------------------------------------------------

# 55. Reserved Bit

하드웨어 문서에서 Reserved bit를 볼 수 있다.

``` text
31 ........ 8 | 7 ... 4 | 3 | 2 | 1 | 0
 Reserved       Mode      IRQ EN ...
```

Reserved bit는 제조사가 정의한 규칙을 따라야 한다.

임의로 값을 변경하지 않는다.

이 때문에 Register 전체에 임의의 상수를 쓰기보다 Mask 기반 조작이 중요한
경우가 많다.

------------------------------------------------------------------------

# 56. 반드시 설명할 수 있어야 하는 것

-   [ ] bit와 byte의 기본 개념
-   [ ] 2진수와 16진수의 관계
-   [ ] `&`, `|`, `^`, `~`
-   [ ] `<<`, `>>`
-   [ ] 논리 연산과 비트 연산의 차이
-   [ ] unsigned 타입을 사용하는 이유
-   [ ] Shift 범위에 주의해야 하는 이유
-   [ ] Bit Mask의 의미
-   [ ] 특정 bit Set
-   [ ] 특정 bit Clear
-   [ ] 특정 bit Toggle
-   [ ] 특정 bit Test
-   [ ] 여러 Mask를 `|`로 결합하는 방법
-   [ ] 이름 있는 Mask의 장점
-   [ ] Bit Field 추출
-   [ ] Bit Field 설정의 기본 절차
-   [ ] Read-Modify-Write의 의미
-   [ ] Register에서 단순 RMW가 항상 안전하지 않은 이유
-   [ ] `<stdint.h>` 타입의 목적
-   [ ] Endianness와 Bit Position의 차이
-   [ ] 코딩 테스트 Bitmask의 기본 아이디어
-   [ ] 임베디드 Register와 Bit Operation의 연결

------------------------------------------------------------------------

# 57. 복습 문제

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q1. 특정 bit를 Set하는 공식은?`</strong>`{=html}
```{=html}
</summary>
```
``` c
value |= (1u << bit);
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
`<strong>`{=html}Q2. 특정 bit를 Clear하는 공식은?`</strong>`{=html}
```{=html}
</summary>
```
``` c
value &= ~(1u << bit);
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
`<strong>`{=html}Q3. 특정 bit를 Toggle하는 공식은?`</strong>`{=html}
```{=html}
</summary>
```
``` c
value ^= (1u << bit);
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
`<strong>`{=html}Q4. 특정 bit가 Set인지 검사하는
공식은?`</strong>`{=html}
```{=html}
</summary>
```
``` c
(value & (1u << bit)) != 0u
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
`<strong>`{=html}Q5. `&&`와 `&`의 차이는?`</strong>`{=html}
```{=html}
</summary>
```
`&&`는 논리 AND이고 `&`는 각 bit를 대상으로 하는 Bitwise AND다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q6. 비트 연산에서 unsigned 타입을 선호하는
이유는?`</strong>`{=html}
```{=html}
</summary>
```
부호와 관련된 해석 및 shift 문제를 줄이고 bit pattern을 더 명확하게 다룰
수 있기 때문이다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q7. `1u << 3`은 어떤 Mask인가?`</strong>`{=html}
```{=html}
</summary>
```
bit 3만 1인 Mask다.

``` text
0000 1000
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
`<strong>`{=html}Q8. bits 4\~2의 3-bit field를 추출하는 기본
형태는?`</strong>`{=html}
```{=html}
</summary>
```
``` c
(value >> 2) & 0x7u
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
`<strong>`{=html}Q9. `REG |= MASK`가 모든 하드웨어 Register에서
안전한가?`</strong>`{=html}
```{=html}
</summary>
```
아니다. Write-1-to-Clear, Read-to-Clear, Write-only 등 특수 동작을 가진
Register가 있으므로 데이터시트를 확인해야 한다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q10. Endianness와 Bit Mask는 같은
개념인가?`</strong>`{=html}
```{=html}
</summary>
```
아니다. Endianness는 multi-byte 값의 byte 순서와 관련되고 Bit Mask는
정수 값의 특정 bit 위치를 선택하거나 조작하는 데 사용한다.

```{=html}
</details>
```

------------------------------------------------------------------------

# 58. Bug Log

`99_Bug_Log/Memory_Bugs.md`와 별도로 `Bit_Bugs.md`를 만들어 관리해도
좋다.

```` markdown
## Bit Clear에서 NOT 누락

### 잘못 작성한 코드

```c
value &= (1u << bit);
```

### 의도

특정 bit만 0으로 만들기.

### 문제

이 코드는 선택한 bit 이외의 bit를 대부분 0으로 만드는 동작이 된다.

### 수정

```c
value &= ~(1u << bit);
```

### 재발 방지

Set / Clear / Toggle / Test 공식을 이진수로 직접 검증한다.
````

추천 항목:

``` text
&& / & 혼동
|| / | 혼동
Clear Mask 오류
Shift 범위 오류
Signed Shift
Field Mask 오류
Reserved Bit 변경
Register RMW 오류
```

------------------------------------------------------------------------

# 59. Bit 문제 사고 방식

``` mermaid
flowchart TD
    A["어떤 bit를 다루는가?"] --> B["bit 번호 확인"]
    B --> C["Mask 생성"]
    C --> D{"목적?"}
    D -->|"Set"| E["OR"]
    D -->|"Clear"| F["AND NOT"]
    D -->|"Toggle"| G["XOR"]
    D -->|"Test"| H["AND"]
    E --> I["Binary로 검증"]
    F --> I
    G --> I
    H --> I
```

비트 코드가 헷갈리면 십진수가 아니라 작은 이진수 예제로 직접 계산한다.

------------------------------------------------------------------------

# 60. 이전 학습과 연결

``` mermaid
flowchart LR
    A["Data Type"] --> B["Unsigned Integer"]
    B --> C["Bit Operation"]
    D["Operator"] --> C
    C --> E["Mask"]
    E --> F["Register"]
```

  기존 개념      Bit Operation에서의 역할
  -------------- ---------------------------
  Data Type      정수 폭과 signed/unsigned
  Operator       비트 연산자
  Control Flow   Flag Test
  Pointer        MMIO Register 접근
  Memory Model   Address Space
  Struct         Register Block 표현

------------------------------------------------------------------------

# 61. 코딩 테스트 연결

``` mermaid
flowchart TD
    A["Bit Operation"] --> B["Bitmask"]
    B --> C["Subset"]
    B --> D["State Compression"]
    B --> E["Flag"]
```

C++ 코딩 테스트에서도:

``` cpp
mask & (1 << i)
mask | (1 << i)
```

와 같은 아이디어를 자주 사용한다.

타입 폭이 커질 때는 적절한 unsigned 타입을 선택한다.

------------------------------------------------------------------------

# 62. 임베디드 연결

``` mermaid
flowchart TD
    A["Bit Operation"] --> B["Mask"]
    B --> C["MMIO Register"]
    C --> D["GPIO"]
    C --> E["UART"]
    C --> F["Timer"]
    C --> G["Interrupt Controller"]
```

펌웨어에서는 다음 형태가 반복된다.

``` text
Read Register
↓
Mask
↓
Set / Clear / Test
↓
Write or Branch
↓
Hardware Behavior
```

------------------------------------------------------------------------

# 63. 핵심 요약

``` mermaid
mindmap
  root((Bit Operation))
    Representation
      Binary
      Hex
    Operators
      AND
      OR
      XOR
      NOT
      Shift
    Mask
      Set
      Clear
      Toggle
      Test
    Field
      Extract
      Update
    Safety
      Unsigned
      Shift Range
      Parentheses
    Embedded
      Register
      RMW
      Reserved Bits
      MMIO
    Algorithm
      Bitmask
      Subset
```

## 최종적으로 기억할 아홉 문장

> **1. 비트 연산은 정수 값을 bit 단위로 조작한다.**

> **2. `&`, `|`, `^`, `~`는 각각 AND, OR, XOR, NOT이다.**

> **3. `1u << bit`는 특정 bit 위치의 Mask를 만드는 대표적인 방법이다.**

> **4. Set은 OR, Clear는 AND NOT, Toggle은 XOR, Test는 AND를 사용한다.**

> **5. 비트 조작에서는 signed보다 unsigned 정수형을 우선 고려한다.**

> **6. Shift count는 피연산자 타입의 유효 bit 범위를 벗어나면 안 된다.**

> **7. 여러 bit로 구성된 Field는 Shift와 Mask를 조합해 추출하고 변경할
> 수 있다.**

> **8. 하드웨어 Register는 특수한 읽기/쓰기 동작을 가질 수 있으므로
> 단순한 Read-Modify-Write가 항상 안전한 것은 아니다.**

> **9. Bit Operation은 `Integer → Mask → Register → Hardware`로 이어지는
> 임베디드 C의 핵심 기반이다.**

------------------------------------------------------------------------

# 64. 현재 학습 진행 상태

``` mermaid
flowchart LR
    A["Struct.md<br/>완료"] --> B["malloc_free.md<br/>완료"]
    B --> C["Bit_Operation.md<br/>현재"]
    C --> D["volatile.md"]
```

``` text
C_Cpp_Study/
└── 01_C/
    ├── Array_String.md    ✓
    ├── Pointer.md         ✓
    ├── Struct.md          ✓
    ├── malloc_free.md     ✓
    ├── Bit_Operation.md   ← 현재
    ├── volatile.md
    └── Embedded_C.md
```

------------------------------------------------------------------------

# 65. 다음 학습

다음 단계는 **`volatile`**이다.

``` mermaid
flowchart LR
    A["Memory"] --> B["Compiler Optimization"]
    B --> C["volatile"]
    C --> D["MMIO Register"]
    C --> E["ISR Shared State"]
```

다음 질문을 해결한다.

``` text
volatile은 왜 필요한가?
컴파일러 최적화와 어떤 관계가 있는가?
하드웨어 Register에 왜 volatile이 사용되는가?
ISR과 공유되는 값에서 어떤 의미가 있는가?
volatile이 atomic을 보장하는가?
volatile이 thread synchronization을 대신할 수 있는가?
const volatile은 무엇인가?
```

**다음 문서:** `01_C/volatile.md`
