# Register

> **학습 위치:** `C_Cpp_Study/05_Embedded/Register.md`\
> **핵심 연결:**
> `Pointer → volatile → Bit Mask → MMIO → Register → Peripheral Driver`\
> **정리 범위:** 개념과 설계 관점만 포함하며 실습·퀴즈는 포함하지
> 않는다.

------------------------------------------------------------------------

## 1. 한 문장 정의

**Register(레지스터)는 CPU 또는 주변장치의 상태와 동작을 나타내거나
제어하는 하드웨어 저장 요소다.**

임베디드에서 흔히 말하는 *Peripheral Register*는 GPIO, UART, Timer 같은
주변장치를 설정하고 상태를 확인하며 데이터를 주고받는 인터페이스다.

``` mermaid
flowchart LR
    A["Firmware"] --> B["Register Read / Write"]
    B --> C["Peripheral"]
    C --> D["Physical Hardware"]
```

> CPU의 범용 레지스터(R0, R1 등)와 Peripheral Register는 관련은 있지만
> 구분되는 개념이다. 이 문서의 주제는 주로 **Peripheral Register**다.

## 2. Register가 필요한 이유

펌웨어는 하드웨어에 다음과 같은 요청을 해야 한다.

-   GPIO Pin을 출력 모드로 설정한다.
-   UART의 전송 가능 여부를 확인한다.
-   Timer를 시작하거나 정지한다.
-   Interrupt 발생 여부를 확인한다.
-   ADC 변환 결과를 읽는다.

하드웨어가 제공하는 Register는 이런 요청과 상태를 **정해진 비트 배치와
접근 규칙**으로 표현한다.

``` mermaid
flowchart TD
    A["Register"] --> B["Configuration"]
    A --> C["Status"]
    A --> D["Data"]
    A --> E["Interrupt Control"]
```

## 3. Register와 일반 RAM 변수의 차이

  -----------------------------------------------------------------------
  관점                    일반 RAM 객체           Peripheral Register
  ----------------------- ----------------------- -----------------------
  주된 목적               프로그램 데이터 저장    하드웨어 제어·상태 확인

  값 변경 주체            프로그램의 일반 실행    CPU 및 하드웨어
                          흐름                    

  읽기 부작용             일반적으로 없음         읽기 시 상태가 변할 수
                                                  있음

  쓰기 부작용             메모리 값 변경          Peripheral 동작 유발
                                                  가능

  접근 폭·순서            언어와 구현의 제약      하드웨어 문서의 추가
                                                  제약

  읽기/쓰기 허용          타입·메모리 보호에 따름 RO, WO, RW 등으로 구분
                                                  가능
  -----------------------------------------------------------------------

**Register에 `volatile`을 붙여도 RAM과 같은 방식으로 자유롭게 조작할 수
있다는 뜻은 아니다.**

------------------------------------------------------------------------

## 4. MMIO: 주소로 접근하는 Register

Memory-Mapped I/O(MMIO)는 Peripheral Register를 CPU의 주소 공간에
배치한다.

``` mermaid
flowchart TD
    A["CPU Address Space"] --> B["Program Memory"]
    A --> C["RAM"]
    A --> D["Peripheral Region"]
    D --> E["GPIO Registers"]
    D --> F["UART Registers"]
    D --> G["Timer Registers"]
```

예를 들어 Reference Manual에 다음과 같은 정보가 있을 수 있다.

``` text
Peripheral Base Address
+ Register Offset
= Register Address
```

``` text
GPIO Base       = 0x4000_0000  (가상의 예)
OUTPUT Offset   = 0x0010
OUTPUT Address  = 0x4000_0010
```

위 주소는 **구조를 설명하기 위한 가상 예시**이며 실제 보드에서
사용해서는 안 된다.

## 5. Base Address와 Offset

Register가 여러 개 모이면 Register Block을 이룬다.

``` text
Peripheral Base
│
├── +0x00  CONTROL
├── +0x04  STATUS
├── +0x08  DATA
└── +0x0C  CONFIG
```

``` mermaid
flowchart LR
    A["Base Address"] --> C["Register Address"]
    B["Offset"] --> C
    C --> D["Specific Register"]
```

Offset은 바이트 단위인지, Register 폭과 어떻게 대응되는지 등 제조사
문서의 표기를 확인한다.

------------------------------------------------------------------------

## 6. C Pointer로 표현하기

MMIO의 개념적인 표현:

``` c
#include <stdint.h>

#define DEVICE_CONTROL \
    (*(volatile uint32_t *)0x40000000u)
```

사용 개념:

``` c
uint32_t current = DEVICE_CONTROL;
DEVICE_CONTROL = current;
```

이 코드는 MMIO를 설명하기 위한 예시다. 실제 주소의 유효성, 접근 폭,
정렬, 메모리 속성은 대상 MCU와 툴체인에 달려 있다.

실제 개발에서는 **제조사 제공 Device Header/CMSIS/SDK**를 우선 사용한다.

## 7. `volatile`의 역할

``` c
volatile uint32_t *reg;
```

`volatile`은 Register 접근이 프로그램의 일반적인 값 계산과 달리 관찰
가능한 의미를 가질 수 있음을 표현한다.

``` mermaid
flowchart LR
    A["Pointer"] --> B["Address"]
    C["volatile"] --> D["Access Semantics"]
    B --> E["MMIO"]
    D --> E
```

그러나 다음은 보장하지 않는다.

``` text
volatile ≠ Atomicity
volatile ≠ Thread Synchronization
volatile ≠ CPU Cache Control
volatile ≠ Hardware Memory Barrier
```

또한 C 소스의 표현식 하나가 모든 플랫폼에서 정확히 하나의 물리적 Bus
Transaction으로 대응한다고 일반화할 수 없다. 접근 폭과 실제 Bus 동작은
구현과 하드웨어를 확인한다.

------------------------------------------------------------------------

## 8. Register Block과 `struct`

제조사 헤더는 Register들을 구조체로 묶어 표현할 수 있다.

``` c
#include <stdint.h>

typedef struct
{
    volatile uint32_t CONTROL;
    const volatile uint32_t STATUS;
    volatile uint32_t DATA;
    uint32_t RESERVED;
} DeviceRegisters;
```

``` mermaid
flowchart TD
    A["DeviceRegisters"] --> B["CONTROL"]
    A --> C["STATUS"]
    A --> D["DATA"]
    A --> E["RESERVED"]
```

-   `volatile`: 하드웨어 접근 의미를 반영한다.
-   `const volatile`: 해당 C lvalue를 통해 쓰지 않지만 하드웨어가 바꿀
    수 있는 상태에 사용할 수 있다.
-   `RESERVED`: 문서에 명시된 주소 간격을 표현하는 용도로 등장할 수
    있다.

구조체의 Offset·Padding·Alignment가 실제 Register Map과 일치해야 하므로
**임의로 만든 구조체를 하드웨어에 연결하지 않는다.**

## 9. `const volatile`은 하드웨어의 쓰기 금지와 같지 않다

``` c
const volatile uint32_t STATUS;
```

이 선언은 C 코드에서 해당 멤버를 통해 쓰는 것을 제한한다.

하드웨어가 그 Register를 변경할 수 있으며, CPU가 다른 접근 경로를
사용하거나 잘못된 캐스팅을 시도하는 것까지 하드웨어적으로 막아 주는 것은
아니다.

**C 타입의 제약과 Hardware Register의 접근 권한은 서로 다른 층위다.**

------------------------------------------------------------------------

## 10. Register의 대표 유형

  유형               의미                  대표 사례
  ------------------ --------------------- ---------------------
  Control            기능 설정·시작·중지   Peripheral Enable
  Status             현재 상태·완료·오류   TX Ready
  Data               송수신·변환 결과      UART RX Data
  Interrupt Enable   Interrupt 허용        RX Interrupt Enable
  Interrupt Status   Interrupt 발생 원인   Pending Flag
  Configuration      속도·모드 설정        Baud Rate Divider

``` mermaid
flowchart TD
    A["Register Block"] --> B["Control"]
    A --> C["Status"]
    A --> D["Data"]
    A --> E["Interrupt"]
    A --> F["Configuration"]
```

------------------------------------------------------------------------

## 11. Access Type: RO, WO, RW

Reference Manual에는 Register 또는 Field별 접근 권한이 표기된다.

  표기   의미         주의
  ------ ------------ ----------------------------------
  RO     Read-only    쓰기 시 동작은 문서 확인
  WO     Write-only   읽기값에 의존하지 않음
  RW     Read/Write   특수 부작용이 없는지는 별도 확인

**RW라고 해서 무조건 일반 RAM처럼 동작한다는 뜻은 아니다.** 특정 Bit에는
별도 Write Semantics가 적용될 수 있다.

------------------------------------------------------------------------

## 12. Bit와 Bit Field

32-bit Register는 각 Bit 또는 여러 Bit를 묶은 Field로 정의될 수 있다.

``` text
31                        8 7      4 3   2 1 0
+--------------------------+--------+-----+-+-+
|        RESERVED          |  MODE  | ... |E|R|
+--------------------------+--------+-----+-+-+
```

-   Bit: 단일 기능을 나타내는 0/1 값.
-   Field: 여러 Bit가 하나의 값이나 모드를 표현.
-   Reserved: 소프트웨어가 임의로 해석하거나 수정하지 않아야 할 수 있는
    영역.

Bit 번호, Field 폭, 유효값은 Reference Manual을 따른다.

------------------------------------------------------------------------

## 13. Bit Mask

``` c
#include <stdint.h>

#define ENABLE_MASK (UINT32_C(1) << 0)
#define READY_MASK  (UINT32_C(1) << 1)
```

기본 연산:

  목적     형태
  -------- ------------------------
  Set      `value \| mask`
  Clear    `value & ~mask`
  Toggle   `value ^ mask`
  Test     `(value & mask) != 0u`

``` mermaid
flowchart LR
    A["Register Value"] --> C["Bit Operation"]
    B["Mask"] --> C
    C --> D["Selected Bit(s)"]
```

이 표는 **일반 비트 연산 공식**이다. 실제 Register에 쓰기 전에 해당
Register의 읽기/쓰기 의미를 확인해야 한다.

## 14. 여러 Bit로 이루어진 Field

Field를 다룰 때는 위치와 폭을 구분한다.

``` c
#define MODE_POS  4u
#define MODE_MASK (UINT32_C(0x3) << MODE_POS)
```

추출 개념:

``` c
uint32_t mode = (value & MODE_MASK) >> MODE_POS;
```

일반 RW Register의 Field 설정 개념:

``` c
value = (value & ~MODE_MASK)
      | ((new_mode << MODE_POS) & MODE_MASK);
```

실제 Register에는 범위 확인과 특수 Write Semantics 검토가 추가로
필요하다.

------------------------------------------------------------------------

## 15. Read-Modify-Write(RMW)

``` c
REG |= ENABLE_MASK;
```

개념적으로 다음 단계를 거친다.

``` mermaid
sequenceDiagram
    participant CPU
    participant REG as Register
    CPU->>REG: Read
    REG-->>CPU: Old Value
    Note over CPU: OR Mask
    CPU->>REG: Write New Value
```

RMW는 일반 RW Register에서는 유용하지만, 다음 상황에서는 문제가 될 수
있다.

-   Hardware가 중간에 값을 바꿈
-   ISR/다른 Core가 같은 Register를 수정함
-   읽기 자체에 부작용이 있음
-   쓰기 시 특정 Bit가 특수하게 해석됨

------------------------------------------------------------------------

## 16. Write-1-to-Clear(W1C)

W1C Field는 **1을 쓰면 해당 상태가 지워지는** 형태다.

예:

``` text
Status = 0b0011
```

두 Flag가 발생한 상태에서 Bit 0만 지우고 싶다면 해당 W1C Field의 규칙에
따라 Bit 0에 `1`을 쓰는 식이다.

``` text
Write 0b0001
→ Bit 0 Clear
```

**W1C Register에 `REG |= MASK`를 무심코 사용하면 읽어 온 다른 1 Bit까지
함께 Clear할 위험이 있다.**

``` mermaid
flowchart TD
    A["W1C Register"] --> B["Write 1"]
    B --> C["Selected Flag Clear"]
    A --> D["RMW"]
    D --> E["Other Flags May Clear"]
```

W1C와 일반 RW Field가 섞인 Register는 문서에서 요구하는 쓰기 값을 정확히
구성해야 한다.

------------------------------------------------------------------------

## 17. Read-to-Clear

일부 Register는 읽는 순간 특정 Flag가 지워질 수 있다.

``` text
Read Status
↓
Hardware Clears Flag
```

이 경우 Debugger의 Peripheral View나 불필요한 상태 확인 코드도 동작에
영향을 줄 수 있다.

> **Register Read는 항상 무해한 관찰이 아니다.**

------------------------------------------------------------------------

## 18. Write-only와 Self-clearing

**Write-only Register:** 읽은 값이 의미 없거나 읽기가 허용되지 않을 수
있다. 기존 값을 읽는 RMW에 적합하지 않다.

**Self-clearing Bit:** CPU가 시작 명령으로 `1`을 쓰면 Hardware가 동작 후
자동으로 `0`으로 되돌릴 수 있다.

``` mermaid
sequenceDiagram
    participant CPU
    participant REG as Control Register
    participant HW as Hardware
    CPU->>REG: Write START=1
    REG->>HW: Start Operation
    HW->>REG: Clear START
```

두 경우 모두 일반 변수와 동일한 접근을 가정하지 않는다.

------------------------------------------------------------------------

## 19. Reserved Bit

Reserved Bit는 현재 사용하지 않거나 공개되지 않은 Hardware 기능을 위한
영역일 수 있다.

일반적으로 제조사 문서에는 다음과 같은 지침이 등장한다.

``` text
Write 0
Preserve Reset Value
Preserve Read Value
Do Not Modify
```

**Reserved Bit 처리 규칙은 Register마다 다를 수 있으므로 "항상 0" 또는
"항상 보존"으로 통일하지 않는다.**

------------------------------------------------------------------------

## 20. Set/Clear 전용 Register

일부 MCU는 RMW 위험을 줄이기 위해 별도 Register를 제공한다.

``` text
OUTPUT_SET
OUTPUT_CLEAR
OUTPUT_TOGGLE
```

``` mermaid
flowchart TD
    A["Change GPIO Output"] --> B{"Register Interface"}
    B --> C["Normal RMW"]
    B --> D["Dedicated SET / CLEAR"]
    D --> E["Target Bits Only"]
```

별도 Set/Clear Register가 있다면 해당 기능과 제조사 HAL을 우선 검토한다.
다만 다른 실행 주체와의 전체 동기화까지 자동으로 해결되는 것은 아니다.

------------------------------------------------------------------------

## 21. Register Reset Value

Reference Manual에는 Reset 후 Register의 초기값이 명시될 수 있다.

``` text
CONTROL reset value: 0x00000000
STATUS  reset value: device-specific
```

초기값을 사용할 때 고려할 점:

-   Power-on Reset과 Software Reset의 범위가 다를 수 있다.
-   일부 Register는 특정 Reset Domain에 속한다.
-   Bootloader나 이전 Firmware가 설정을 남길 수 있다.
-   일부 상태 Bit는 Hardware 환경에 따라 달라질 수 있다.

따라서 Driver 초기화에서는 **필요한 상태를 명시적으로 설정하는 방식**을
검토한다.

------------------------------------------------------------------------

## 22. Register 접근 폭과 Alignment

Hardware가 32-bit 접근을 요구하는 Register에 8-bit 접근을 하면 동작이
다르거나 Fault가 발생할 수 있다.

``` text
Register Width
Access Width
Address Alignment
Bus Rules
```

``` mermaid
flowchart LR
    A["C Type"] --> B["Load / Store Width"]
    B --> C["Bus Access"]
    C --> D["Register Semantics"]
```

타입 선언만 보고 안전성을 확정하지 않고 MCU 문서와 Compiler/ABI 규칙을
확인한다.

------------------------------------------------------------------------

## 23. Memory Ordering과 Barrier

Peripheral 제어에는 쓰기 순서나 완료 시점이 중요할 수 있다.

예:

``` text
Configure DMA Buffer
↓
Configure DMA Register
↓
Enable DMA
```

이 순서가 실제로 Hardware에 필요한 순서로 관찰되도록 하려면 아키텍처와
SDK가 제공하는 Memory Barrier, Device Memory Attribute, Cache
Maintenance 등이 필요할 수 있다.

``` mermaid
flowchart TD
    A["Compiler Semantics"] --> D["Correct Hardware Interaction"]
    B["CPU / Bus Ordering"] --> D
    C["Cache / Memory Attributes"] --> D
```

`volatile`만으로 모든 CPU·Bus·Cache Ordering 요구사항이 해결되지는
않는다.

------------------------------------------------------------------------

## 24. Interrupt와 Register

Interrupt 처리에서는 다음 Register들이 연결될 수 있다.

``` text
Interrupt Enable
Interrupt Pending
Interrupt Status
Interrupt Clear
```

``` mermaid
sequenceDiagram
    participant HW as Peripheral
    participant IC as Interrupt Controller
    participant CPU
    participant ISR
    HW->>IC: Event / Pending
    IC->>CPU: Interrupt Request
    CPU->>ISR: Execute
    ISR->>HW: Read / Acknowledge per Manual
```

Interrupt Flag를 **언제, 어떤 순서로 Clear해야 하는지**는 장치별 규칙에
따라 달라진다.

------------------------------------------------------------------------

## 25. Polling과 Register

Polling은 상태 Register를 반복적으로 확인한다.

``` mermaid
flowchart TD
    A["Read STATUS"] --> B{"READY?"}
    B -->|"No"| C{"Timeout?"}
    C -->|"No"| A
    C -->|"Yes"| D["Error"]
    B -->|"Yes"| E["Continue"]
```

주의점:

-   Flag가 실제로 Hardware에 의해 갱신되는가?
-   읽기 자체가 Flag를 Clear하는가?
-   READY 조건이 단일 Bit인가, 여러 상태의 조합인가?
-   Timeout 기준은 무엇인가?
-   오류 상태를 READY보다 먼저 확인해야 하는가?

------------------------------------------------------------------------

## 26. Register와 Driver 계층

``` mermaid
flowchart TD
    A["Application"] --> B["Driver API"]
    B --> C["Register Definitions"]
    C --> D["MMIO"]
    D --> E["Peripheral"]
```

Application이 Register 주소와 Bit Mask를 직접 다루기 시작하면 Hardware
교체와 유지보수가 어려워질 수 있다.

Driver는 다음 책임을 묶는다.

``` text
Register Address
Bit / Field Definition
Initialization Sequence
Read / Write Semantics
Timeout
Error Handling
Concurrency Policy
```

------------------------------------------------------------------------

## 27. Register 정의의 출처

실제 Firmware에서는 다음 자료를 함께 확인한다.

  자료                  확인 내용
  --------------------- -------------------------------------------
  Datasheet             Device/Pin/Electrical 특성
  Reference Manual      Register Map, Bit, Reset Value, 접근 규칙
  Errata                알려진 Silicon 문제와 우회 방법
  Device Header/CMSIS   C Register 타입, Base Address, Mask
  SDK/HAL               제조사가 제공하는 접근 API와 초기화 순서
  Board Schematic       Peripheral과 외부 회로의 연결

특히 **Errata**에는 Reference Manual만으로 알기 어려운 실제 Silicon
제약이 기록될 수 있다.

------------------------------------------------------------------------

## 28. C와 C++에서의 Register 접근

C와 C++ 모두 제조사 Header의 MMIO 정의를 사용할 수 있다.

  관점                C                          C++
  ------------------- -------------------------- -------------------------------
  Register 표현       `volatile` 포인터·구조체   동일한 저수준 표현 가능
  Driver 인터페이스   함수·구조체                함수·클래스
  Bit Mask            매크로·정수 상수           `constexpr`·강한 타입 등 가능
  추상화              모듈 중심                  모듈·클래스·템플릿 활용 가능

C++의 클래스나 템플릿도 **하드웨어 접근 규칙 자체를 바꾸지 않는다.**
추상화 아래에서는 동일한 Register Semantics를 지켜야 한다.

------------------------------------------------------------------------

## 29. 개념 간 연결

``` mermaid
flowchart TD
    A["Data Type"] --> F["Register"]
    B["Pointer"] --> F
    C["Struct"] --> F
    D["Bit Operation"] --> F
    E["volatile"] --> F
    F --> G["GPIO / UART / Timer"]
    G --> H["Driver"]
    H --> I["Firmware"]
```

  선행 문서            Register와의 연결
  -------------------- --------------------------------------
  `Data_Type.md`       Register 폭과 Signed/Unsigned
  `Pointer.md`         MMIO 주소 표현
  `Struct.md`          Register Block
  `Bit_Operation.md`   Bit/Field Mask
  `volatile.md`        Hardware Access Semantics
  `Embedded_C.md`      Driver·Peripheral·Firmware 전체 구조

------------------------------------------------------------------------

## 30. 핵심 정리

> **Register는 Firmware와 Hardware가 만나는 인터페이스다.**

> **MMIO에서는 Register가 주소 공간에 배치되고 Pointer로 표현될 수
> 있다.**

> **`volatile`은 접근 의미를 표현하지만 Atomicity, Cache Coherency,
> Memory Barrier를 대신하지 않는다.**

> **Bit Mask는 Register의 기능을 선택하는 도구이며, 실제 쓰기 규칙은
> Register Semantics를 따른다.**

> **W1C, Read-to-Clear, Write-only, Reserved Bit가 존재하므로 Register는
> 일반 RAM처럼 다루지 않는다.**

> **Register 접근 폭, Alignment, Reset Value, 순서와 부작용은 Reference
> Manual 및 Errata를 확인한다.**

> **Driver는 Register 수준의 세부사항을 Application과 분리한다.**

------------------------------------------------------------------------

## 다음 학습

**다음 문서:** `05_Embedded/Interrupt.md`

``` mermaid
flowchart LR
    A["Register"] --> B["Interrupt Status"]
    B --> C["Interrupt Controller"]
    C --> D["ISR"]
    D --> E["Main Loop / RTOS Task"]
```

다음에는 Interrupt 발생 과정, ISR, 우선순위, 공유 데이터, Flag 처리,
Latency, RTOS와의 연결을 개념 중심으로 정리한다.
