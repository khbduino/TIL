# Embedded C

> **학습 목표**
>
> 지금까지 학습한 C 개념을 임베디드/펌웨어 관점에서 연결한다.
>
> **Data Type → Pointer → Memory → Bit Operation → `volatile` → Register
> → Peripheral → Driver → Firmware**
>
> 이 문서부터는 별도의 실습 문제 없이 **개념 정리와 개념 간 연결**에
> 집중한다.
>
> 위치: `C_Cpp_Study/01_C/Embedded_C.md`

------------------------------------------------------------------------

# 1. Embedded C란?

Embedded C는 별도의 새로운 C 언어라기보다 **하드웨어를 제어하는 임베디드
시스템에서 C를 사용하는 방식과 관점**을 의미한다.

``` mermaid
flowchart TD
    A["C Language"] --> B["Embedded Environment"]
    B --> C["Hardware"]
    B --> D["Memory"]
    B --> E["Timing"]
    B --> F["Peripheral"]
```

일반 응용 프로그램보다 다음 요소의 중요성이 커진다.

``` text
제한된 Memory
Hardware Register
정확한 Data Width
Bit Operation
Memory-Mapped I/O
Interrupt
Timing
Deterministic Behavior
Peripheral Driver
```

------------------------------------------------------------------------

# 2. 전체 구조

``` mermaid
flowchart LR
    A["C Language"] --> B["Memory / Address"]
    B --> C["MMIO Register"]
    C --> D["Peripheral"]
    D --> E["Driver"]
    E --> F["Application"]
```

핵심 질문은 다음과 같다.

``` text
데이터는 어디에 저장되는가?
주소는 RAM인가 Register인가?
각 bit는 어떤 Hardware 기능인가?
값은 CPU와 Hardware 중 누가 변경하는가?
함수는 어떤 Hardware 계층을 추상화하는가?
```

------------------------------------------------------------------------

# 3. Fixed-width Integer

``` c
#include <stdint.h>

uint8_t
uint16_t
uint32_t
int8_t
int16_t
int32_t
```

임베디드에서는 Register나 Protocol의 데이터 폭이 명확하게 정해지는
경우가 많다.

``` text
UART Data     → 8 bit
Protocol ID   → 16 bit
Register      → 32 bit
Timer Counter → MCU에 따라 16/32 bit 등
```

``` mermaid
flowchart LR
    A["Hardware Data Width"] --> B["stdint.h"]
    B --> C["uint8_t"]
    B --> D["uint16_t"]
    B --> E["uint32_t"]
```

정확한 폭의 정수형은 해당 구현에서 그 폭을 지원할 때 제공된다.

------------------------------------------------------------------------

# 4. Hardware Register

Peripheral은 여러 Register를 통해 제어되는 경우가 많다.

``` text
UART
├── CONTROL
├── STATUS
├── DATA
└── BAUDRATE
```

``` mermaid
flowchart TD
    A["Peripheral"] --> B["Control Register"]
    A --> C["Status Register"]
    A --> D["Data Register"]
    A --> E["Configuration Register"]
```

CPU는 Register를 읽고 쓰면서 Peripheral의 상태를 확인하고 동작을
설정한다.

------------------------------------------------------------------------

# 5. Memory-Mapped I/O

MMIO는 Peripheral Register를 CPU 주소 공간에 배치하는 방식이다.

``` mermaid
flowchart TD
    A["CPU Address Space"] --> B["Flash"]
    A --> C["RAM"]
    A --> D["Peripheral Registers"]
```

개념적인 주소:

``` text
0x2000_0000 → RAM 영역의 예
0x4000_0000 → Peripheral 영역의 예
```

실제 주소 배치는 MCU의 Memory Map이 기준이다.

------------------------------------------------------------------------

# 6. Pointer + MMIO + `volatile`

개념적인 Register 접근:

``` c
volatile uint32_t *reg =
    (volatile uint32_t *)0x40000000u;
```

``` mermaid
flowchart LR
    A["Address"] --> B["Pointer"]
    B --> C["volatile Register"]
    C --> D["Hardware"]
```

핵심 연결:

> **Pointer는 주소를 표현하고, `volatile`은 Hardware와 연결된 접근
> 의미를 표현한다.**

실제 MCU에서는 제조사의 CMSIS/SDK 헤더와 Reference Manual을 우선
사용한다.

------------------------------------------------------------------------

# 7. Bit Operation과 Register

Register의 각 bit가 서로 다른 기능을 나타낼 수 있다.

``` text
CONTROL REGISTER

bit 0 → ENABLE
bit 1 → TX_ENABLE
bit 2 → RX_ENABLE
bit 3 → IRQ_ENABLE
```

``` c
#define ENABLE_MASK    (UINT32_C(1) << 0)
#define TX_MASK        (UINT32_C(1) << 1)
#define RX_MASK        (UINT32_C(1) << 2)
```

``` mermaid
flowchart LR
    A["Register"] --> B["Bit"]
    B --> C["Mask"]
    C --> D["Hardware Feature"]
```

기본 공식:

  목적     형태
  -------- ----------------------
  Set      `reg |= mask;`
  Clear    `reg &= ~mask;`
  Toggle   `reg ^= mask;`
  Test     `(reg & mask) != 0u`

------------------------------------------------------------------------

# 8. Register는 일반 RAM이 아니다

Hardware Register는 다음과 같은 특수 동작을 가질 수 있다.

``` text
Read-only
Write-only
Write-1-to-Clear
Read-to-Clear
Self-clearing
Reserved Bits
```

``` mermaid
flowchart TD
    A["Register"] --> B{"Semantics"}
    B --> C["Normal R/W"]
    B --> D["W1C"]
    B --> E["Read-only"]
    B --> F["Special Behavior"]
```

따라서 `REG |= MASK` 같은 Read-Modify-Write를 모든 Register에 기계적으로
적용하면 안 된다.

**Reference Manual의 Register 설명이 최종 기준이다.**

------------------------------------------------------------------------

# 9. Read-Modify-Write

``` c
REG |= MASK;
```

는 개념적으로:

``` text
Read → Modify → Write
```

이다.

``` mermaid
sequenceDiagram
    participant CPU
    participant REG as Register
    CPU->>REG: Read
    REG-->>CPU: Current Value
    Note over CPU: Modify with Mask
    CPU->>REG: Write
```

Interrupt나 Hardware가 같은 상태를 변경할 수 있다면 경쟁 상태와 원자성도
검토해야 한다.

`volatile` 자체는 이 과정을 atomic하게 만들지 않는다.

------------------------------------------------------------------------

# 10. 주요 Peripheral

``` mermaid
mindmap
  root((MCU Peripheral))
    GPIO
    UART
    SPI
    I2C
    Timer
    ADC
    PWM
    DMA
    Watchdog
```

대표 역할:

  Peripheral   역할
  ------------ -----------------------------
  GPIO         Digital Input/Output
  UART         비동기 Serial Communication
  SPI          동기식 Serial Communication
  I2C          Address 기반 Serial Bus
  Timer        시간 측정, 주기 이벤트, PWM
  ADC          Analog → Digital 변환
  DMA          CPU 개입을 줄인 데이터 전송
  Watchdog     시스템 동작 감시

------------------------------------------------------------------------

# 11. GPIO

GPIO는 General-Purpose Input/Output이다.

``` text
Pin Mode
Digital Output
Digital Input
Pull-up / Pull-down
Alternate Function
Interrupt
```

``` mermaid
flowchart LR
    A["GPIO Register"] --> B["Pin Configuration"]
    B --> C["Physical Pin"]
    C --> D["LED / Button / Sensor"]
```

------------------------------------------------------------------------

# 12. UART

UART의 주요 개념:

``` text
TX
RX
Baud Rate
Data Register
Status Flag
Interrupt
Buffer
```

``` mermaid
flowchart LR
    A["Application"] --> B["UART Driver"]
    B --> C["UART Register"]
    C --> D["TX / RX"]
```

UART는 이후 Buffer, Ring Buffer, Packet Parser와 연결된다.

------------------------------------------------------------------------

# 13. SPI와 I2C

SPI의 대표 Signal:

``` text
SCLK
MOSI
MISO
CS
```

I2C의 대표 Signal:

``` text
SCL
SDA
```

``` mermaid
flowchart TD
    A["MCU"] --> B["SPI"]
    A --> C["I2C"]
    B --> D["Sensor / Flash / Display"]
    C --> E["Sensor / EEPROM"]
```

두 방식 모두 실제 통신에서는 Timing, 상태 Flag, Buffer, Error Handling이
중요하다.

------------------------------------------------------------------------

# 14. Timer와 Clock

Timer는 MCU Clock을 기반으로 동작한다.

``` mermaid
flowchart LR
    A["Oscillator"] --> B["Clock Tree"]
    B --> C["Peripheral Clock"]
    C --> D["Prescaler"]
    D --> E["Timer Counter"]
    E --> F["Interrupt / PWM"]
```

대표 개념:

``` text
Clock
Prescaler
Counter
Period
Compare
Capture
PWM
Interrupt
```

UART Baud Rate와 SPI 속도 등도 Clock 설정과 연결된다.

------------------------------------------------------------------------

# 15. Polling

Polling은 CPU가 Peripheral 상태를 반복적으로 확인하는 방식이다.

``` c
while ((STATUS_REG & READY_MASK) == 0u)
{
}
```

``` mermaid
flowchart TD
    A["Read Status"] --> B{"Ready?"}
    B -->|"No"| A
    B -->|"Yes"| C["Continue"]
```

장점은 구조가 단순하다는 것이고, 단점은 CPU 시간을 계속 소비하거나 무한
대기에 빠질 수 있다는 것이다.

------------------------------------------------------------------------

# 16. Interrupt

Interrupt는 Hardware Event가 발생했을 때 CPU가 ISR을 실행하도록 하는
방식이다.

``` mermaid
sequenceDiagram
    participant M as Main
    participant H as Hardware
    participant I as ISR
    M->>M: Normal Execution
    H->>I: Interrupt
    I->>I: Handle Event
    I-->>M: Return
```

Polling과 Interrupt는 상호 배타적인 개념이 아니라 시스템 요구사항에 따라
함께 사용될 수도 있다.

------------------------------------------------------------------------

# 17. ISR

ISR은 Interrupt Service Routine이다.

주요 고려사항:

``` text
짧은 실행 시간
Interrupt Latency
공유 데이터
Atomicity
Priority
Reentrancy
```

``` mermaid
flowchart TD
    A["Interrupt"] --> B["ISR"]
    B --> C["Minimal Work"]
    C --> D["Flag / Buffer Update"]
    D --> E["Main / Task Processing"]
```

ISR에서 무거운 처리를 최소화하고 후속 작업을 main loop나 RTOS Task로
넘기는 구조가 흔하다.

------------------------------------------------------------------------

# 18. ISR과 `volatile`

ISR이 변경하고 main이 읽는 상태에는 `volatile`이 관련될 수 있다.

``` c
volatile uint32_t event_flag;
```

그러나:

``` text
volatile ≠ atomic
volatile ≠ mutex
volatile ≠ synchronization
```

공유 데이터의 읽기/쓰기 폭, Read-Modify-Write, Interrupt 우선순위 등을
별도로 분석해야 한다.

------------------------------------------------------------------------

# 19. Driver

Driver는 Register 수준의 하드웨어 세부사항을 상위 코드에서 분리하는
계층이다.

``` mermaid
flowchart TD
    A["Application"] --> B["Driver API"]
    B --> C["Register Access"]
    C --> D["Peripheral"]
```

개념적인 API:

``` c
void uart_init(void);
int uart_write(const uint8_t *data, size_t size);
int uart_read(uint8_t *data, size_t size);
```

상위 Application이 Register bit 위치를 직접 알 필요가 없도록 만드는 것이
핵심이다.

------------------------------------------------------------------------

# 20. Hardware Abstraction

``` mermaid
flowchart TD
    A["Application"] --> B["Service / Middleware"]
    B --> C["Driver / HAL"]
    C --> D["MCU Specific Layer"]
    D --> E["Register"]
    E --> F["Hardware"]
```

추상화의 목적은 Hardware를 무조건 숨기는 것이 아니라 **변경 가능성이
높은 세부사항을 적절한 계층에 가두는 것**이다.

지나친 추상화는 코드 크기와 복잡도를 늘릴 수 있으므로 시스템 규모와
요구사항에 맞춰야 한다.

------------------------------------------------------------------------

# 21. Header / Source 분리

대표적인 Driver 구조:

``` text
uart.h
uart.c
```

Header:

``` text
Public Type
Public Constant
Public Function Declaration
```

Source:

``` text
Register Access
Internal State
static Function
Public Function Definition
```

``` mermaid
flowchart LR
    A["Application"] --> B["uart.h"]
    B --> C["uart.c"]
    C --> D["Hardware"]
```

------------------------------------------------------------------------

# 22. `static`과 Module 내부 구현

파일 내부에서만 사용할 함수:

``` c
static void uart_configure_baudrate(void);
```

파일 내부 상태:

``` c
static uint8_t rx_buffer[128];
```

파일 범위의 `static`은 이름에 internal linkage를 부여하여 모듈 외부에서
직접 접근하지 못하도록 구성하는 데 사용될 수 있다.

------------------------------------------------------------------------

# 23. Firmware Main Loop

Bare-metal Firmware의 대표적인 구조:

``` c
int main(void)
{
    system_init();
    gpio_init();
    uart_init();
    timer_init();

    while (1)
    {
        /* application processing */
    }
}
```

``` mermaid
flowchart TD
    A["Reset"] --> B["Initialization"]
    B --> C["Main Loop"]
    C --> D["Process"]
    D --> C
```

------------------------------------------------------------------------

# 24. Startup Code

`main()` 이전에도 여러 초기화 과정이 존재할 수 있다.

``` mermaid
flowchart TD
    A["Reset"] --> B["Reset Handler"]
    B --> C["Runtime Initialization"]
    C --> D["System Initialization"]
    D --> E["main"]
```

대표 개념:

``` text
Stack 설정
.data 초기화
.bss 초기화
System Clock 초기화
main 호출
```

정확한 과정은 MCU, ABI, Runtime, Toolchain에 따라 달라진다.

------------------------------------------------------------------------

# 25. Memory Section

대표적인 Section 개념:

  Section     대표 내용
  ----------- ---------------------------------------
  `.text`     실행 코드
  `.rodata`   읽기 전용 데이터
  `.data`     초기값이 있는 writable static storage
  `.bss`      zero-initialized static storage

``` mermaid
flowchart TD
    A["Firmware"] --> B[".text"]
    A --> C[".rodata"]
    A --> D[".data"]
    A --> E[".bss"]
```

실제 Section 이름과 배치는 Linker Script와 Toolchain에 따라 달라질 수
있다.

------------------------------------------------------------------------

# 26. Linker Script

Linker Script는 코드와 데이터를 Flash/RAM의 어느 위치에 배치할지
결정하는 데 사용된다.

``` mermaid
flowchart LR
    A["Object Files"] --> B["Linker"]
    C["Linker Script"] --> B
    B --> D["Firmware Memory Layout"]
```

개념:

``` text
Flash
├── .text
├── .rodata
└── 초기화 데이터

RAM
├── .data
├── .bss
├── Heap
└── Stack
```

------------------------------------------------------------------------

# 27. Stack과 Heap

Stack에서는 일반적으로 함수 호출과 자동 저장 기간 객체 등이 중요한
부분을 차지한다.

임베디드에서 Stack 사용량을 키울 수 있는 요소:

``` text
큰 Local Array
깊은 Call Chain
Recursion
ISR Nesting
RTOS Task Stack
```

Heap에서는 동적 메모리를 관리한다.

``` text
malloc
calloc
realloc
free
```

Heap 사용 시:

``` text
Fragmentation
Allocation Failure
Timing Predictability
Long-term Stability
```

를 함께 고려한다.

------------------------------------------------------------------------

# 28. Static Allocation

``` c
static uint8_t rx_buffer[128];
```

장점:

``` text
Memory 사용량 예측이 쉬움
Runtime allocation failure 없음
Heap fragmentation 없음
```

단점:

``` text
최대 크기를 미리 결정해야 함
필요 이상으로 Memory를 예약할 수 있음
```

Embedded에서는 시스템 요구사항에 따라 Static Allocation을 적극 사용하는
경우가 많다.

------------------------------------------------------------------------

# 29. Buffer

대표 Buffer:

``` text
UART RX/TX Buffer
SPI Buffer
ADC Sample Buffer
Network Packet Buffer
DMA Buffer
```

``` mermaid
flowchart LR
    A["Peripheral"] --> B["Buffer"]
    B --> C["Processing"]
```

Array, Pointer, Size, Lifetime, Concurrency 개념이 모두 연결된다.

------------------------------------------------------------------------

# 30. Ring Buffer

연속적으로 들어오는 데이터를 고정 크기 Buffer에서 관리하는 대표 구조다.

``` mermaid
flowchart LR
    A["UART RX"] --> B["Ring Buffer"]
    B --> C["Parser"]
```

핵심 요소:

``` text
Fixed-size Array
Read Index
Write Index
Wrap-around
Full / Empty State
```

세부 내용은 `05_Embedded/Ring_Buffer.md`에서 확장한다.

------------------------------------------------------------------------

# 31. DMA

DMA는 Hardware가 Peripheral과 Memory 사이의 데이터 전송을 수행하도록
하는 기능이다.

``` mermaid
flowchart LR
    A["Peripheral"] --> B["DMA"]
    B --> C["RAM Buffer"]
    D["CPU"] --> C
```

중요한 개념:

``` text
Buffer Lifetime
Alignment
Transfer Completion
Interrupt
Cache Coherency
Memory Ordering
```

`volatile`만으로 DMA 관련 Cache/Ordering 문제가 모두 해결되지는 않는다.

------------------------------------------------------------------------

# 32. Error Handling과 Timeout

Firmware에서 대표적으로 처리해야 하는 오류:

``` text
Timeout
Invalid Parameter
Peripheral Busy
Communication Error
CRC Error
Memory Failure
Hardware Fault
```

``` mermaid
flowchart TD
    A["Operation"] --> B{"Success?"}
    B -->|"Yes"| C["Continue"]
    B -->|"No"| D["Error Handling"]
    D --> E["Retry"]
    D --> F["Report"]
    D --> G["Safe State"]
```

Hardware가 특정 상태가 되기를 기다릴 때는 무한 대기 대신 Timeout 정책이
필요한 경우가 많다.

------------------------------------------------------------------------

# 33. Driver Result

Driver API는 상태를 반환하도록 설계할 수 있다.

``` c
typedef enum
{
    DRIVER_OK = 0,
    DRIVER_ERROR,
    DRIVER_BUSY,
    DRIVER_TIMEOUT
} DriverResult;
```

``` mermaid
flowchart LR
    A["Driver"] --> B["Result"]
    B --> C["Application Policy"]
```

Driver는 저수준 상태를 전달하고, 재시도/복구/안전 상태 진입 같은 정책은
적절한 상위 계층에서 결정할 수 있다.

------------------------------------------------------------------------

# 34. Watchdog

Watchdog Timer는 Firmware가 정상적으로 동작하는지 감시하는 Hardware
기능이다.

``` mermaid
flowchart TD
    A["Firmware"] --> B["Watchdog Refresh"]
    B --> C{"Refresh 지속?"}
    C -->|"Yes"| A
    C -->|"No"| D["Timeout"]
    D --> E["Reset / Fault Action"]
```

Watchdog은 오류 복구 전략의 한 구성 요소이며 모든 Software 오류를
자동으로 해결하는 기능은 아니다.

------------------------------------------------------------------------

# 35. State Machine

Firmware는 상태 기반으로 동작하는 경우가 많다.

``` mermaid
stateDiagram-v2
    [*] --> IDLE
    IDLE --> RECEIVING
    RECEIVING --> PROCESSING
    PROCESSING --> IDLE
    RECEIVING --> ERROR
    ERROR --> IDLE
```

C에서는 `enum`과 `switch`를 조합해 FSM을 표현하는 방식이 흔하다.

``` text
Control Flow
+
enum
+
Function
+
Event
=
FSM
```

------------------------------------------------------------------------

# 36. Datasheet / Reference Manual / Schematic

Embedded 개발에서는 언어 문서 외에 Hardware 문서가 필수적이다.

``` mermaid
flowchart LR
    A["Firmware"] --> B["C / Compiler Documentation"]
    A --> C["Datasheet"]
    A --> D["Reference Manual"]
    A --> E["Errata"]
    A --> F["Board Schematic"]
```

일반적으로:

``` text
Datasheet
→ Device 특성, Pin, Electrical Specification

Reference Manual
→ Peripheral과 Register 동작 상세

Errata
→ 알려진 Silicon 문제와 Workaround

Schematic
→ MCU Pin과 실제 Board 회로의 연결
```

제조사마다 문서 구성과 이름은 달라질 수 있다.

------------------------------------------------------------------------

# 37. Cross Compilation

개발 PC와 실행 대상 CPU가 다른 경우가 많다.

``` mermaid
flowchart LR
    A["Host PC"] --> B["Cross Compiler"]
    B --> C["Target Machine Code"]
    C --> D["MCU"]
```

예:

``` text
Host   → x86-64 PC
Target → ARM Cortex-M MCU
```

------------------------------------------------------------------------

# 38. Firmware Build 결과

환경에 따라 다음 파일을 볼 수 있다.

``` text
ELF
BIN
HEX
MAP
```

``` mermaid
flowchart TD
    A["Linker"] --> B["ELF"]
    A --> C["BIN"]
    A --> D["HEX"]
    A --> E["MAP"]
```

MAP 파일은 Symbol과 Section의 Memory 배치를 분석하는 데 유용하다.

------------------------------------------------------------------------

# 39. Debugging

Embedded Debugger에서 자주 확인하는 요소:

``` text
Breakpoint
Step
CPU Register
Memory
Watch
Call Stack
Peripheral Register
```

``` mermaid
flowchart LR
    A["Debugger"] --> B["CPU"]
    B --> C["Memory"]
    B --> D["Register"]
    B --> E["Peripheral"]
```

JTAG/SWD 같은 Debug Interface가 사용될 수 있다.

------------------------------------------------------------------------

# 40. Logging과 Timing

UART나 Debug Channel을 통해 Logging할 수 있지만 Logging 자체에도 실행
비용이 있다.

``` mermaid
flowchart LR
    A["Firmware"] --> B["Logging"]
    B --> C["Debug Output"]
    B --> D["Execution Time"]
```

실시간성이 중요한 경로에서는 Logging이 Timing에 미치는 영향도 고려한다.

------------------------------------------------------------------------

# 41. Determinism

Real-Time System에서는 평균 실행 시간뿐 아니라 요구 Deadline 안에
동작하는지와 실행 시간의 예측 가능성이 중요하다.

``` mermaid
flowchart LR
    A["Execution"] --> B["Timing"]
    B --> C["Predictability"]
    C --> D["Real-time Requirement"]
```

Embedded에서 성능은 단순히 "빠름"만을 의미하지 않는다.

------------------------------------------------------------------------

# 42. Bare-metal과 RTOS

``` mermaid
flowchart TD
    A["Embedded Firmware"] --> B["Bare-metal"]
    A --> C["RTOS"]
    B --> D["Main Loop + ISR"]
    C --> E["Task + Scheduler"]
```

Bare-metal의 대표 구조:

``` text
Initialization
↓
Main Loop
+
Interrupt
```

RTOS에서는 이후 다음 개념이 추가된다.

``` text
Task
Queue
Semaphore
Mutex
Timer
Scheduler
```

------------------------------------------------------------------------

# 43. Embedded C 핵심 사고 방식

``` mermaid
flowchart TD
    A["Requirement"] --> B["Hardware Resource"]
    B --> C["Register / Peripheral"]
    C --> D["Driver API"]
    D --> E["Application Logic"]
    E --> F["Memory / Timing / Error 검토"]
```

항상 다음을 함께 본다.

``` text
Memory는 충분한가?
주소와 타입은 올바른가?
Register 동작은 문서와 일치하는가?
공유 접근이 가능한가?
Timeout이 필요한가?
ISR이 너무 오래 실행되지 않는가?
오류 발생 시 시스템은 어떻게 동작하는가?
```

------------------------------------------------------------------------

# 44. 지금까지의 C 개념 통합

``` mermaid
flowchart TD
    A["Data Type"] --> B["Fixed-width Integer"]
    C["Pointer"] --> D["MMIO Address"]
    E["Struct"] --> F["Register Block"]
    G["Bit Operation"] --> H["Register Mask"]
    I["volatile"] --> J["Hardware Access"]
    K["Function"] --> L["Driver API"]

    B --> M["Embedded C"]
    D --> M
    F --> M
    H --> M
    J --> M
    L --> M
```

  C 개념           Embedded 연결
  ---------------- ----------------------------
  Data Type        Register / Protocol 폭
  Array            Buffer
  Pointer          MMIO Address
  Struct           Register Block / Packet
  Dynamic Memory   Heap 정책
  Bit Operation    Register Mask
  `volatile`       Hardware Access
  Function         Driver API
  Control Flow     Polling / FSM
  Memory Model     Flash / RAM / Stack / Heap

------------------------------------------------------------------------

# 45. 개념 체크리스트

-   [ ] Embedded C의 의미를 설명할 수 있다.
-   [ ] Fixed-width Integer의 목적을 설명할 수 있다.
-   [ ] MMIO의 의미를 설명할 수 있다.
-   [ ] Pointer와 Register의 관계를 설명할 수 있다.
-   [ ] `volatile`과 Register의 관계를 설명할 수 있다.
-   [ ] Bit Mask와 Register Control의 관계를 설명할 수 있다.
-   [ ] Register가 일반 RAM과 다를 수 있음을 이해한다.
-   [ ] Read-Modify-Write를 설명할 수 있다.
-   [ ] GPIO/UART/SPI/I2C/Timer의 역할을 구분할 수 있다.
-   [ ] Polling과 Interrupt의 차이를 설명할 수 있다.
-   [ ] ISR의 기본 설계 관점을 설명할 수 있다.
-   [ ] Driver Layer의 목적을 설명할 수 있다.
-   [ ] Header/Source 분리의 의미를 설명할 수 있다.
-   [ ] Startup Code의 역할을 개념적으로 이해한다.
-   [ ] Linker Script와 Memory Layout의 관계를 이해한다.
-   [ ] Stack과 Heap의 임베디드 관점을 설명할 수 있다.
-   [ ] Static Buffer가 사용되는 이유를 설명할 수 있다.
-   [ ] DMA와 Buffer Lifetime의 관계를 이해한다.
-   [ ] Timeout과 Error Handling의 필요성을 설명할 수 있다.
-   [ ] Watchdog의 역할을 설명할 수 있다.
-   [ ] FSM이 Firmware 구조에 자주 사용되는 이유를 설명할 수 있다.
-   [ ] Bare-metal과 RTOS의 기본 차이를 설명할 수 있다.

------------------------------------------------------------------------

# 46. 핵심 요약

``` mermaid
mindmap
  root((Embedded C))
    Types
      Fixed Width
    Memory
      Flash
      RAM
      Stack
      Heap
    Hardware
      MMIO
      Register
      volatile
      Bit Mask
    Peripheral
      GPIO
      UART
      SPI
      I2C
      Timer
      DMA
    Execution
      Polling
      Interrupt
      ISR
      Main Loop
    Architecture
      Driver
      HAL
      Application
      FSM
    Reliability
      Timeout
      Error Handling
      Watchdog
      Determinism
    Build
      Compiler
      Linker
      Linker Script
      Firmware Image
```

## 기억할 핵심

> **Embedded C는 C 문법을 Hardware, Memory, Timing 제약과 연결해
> 사용하는 방식이다.**

> **MMIO에서는 Hardware Register가 CPU Address Space에 배치된다.**

> **Pointer는 Register Address를 표현하고 `volatile`은 Hardware와 연결된
> 접근 의미를 표현한다.**

> **Bit Operation은 Register의 기능을 선택하고 제어하는 핵심 도구다.**

> **Hardware Register의 실제 동작은 Reference Manual을 기준으로
> 판단한다.**

> **Polling과 Interrupt는 Hardware Event를 처리하는 대표적인 방식이다.**

> **Driver는 Register 수준의 세부사항과 Application을 분리한다.**

> **Firmware에서는 Memory, Timing, Error Handling, Concurrency를 함께
> 고려한다.**

> **Startup, Linker, Memory Section을 이해하면 C 코드가 MCU Memory에
> 어떻게 배치되는지 연결할 수 있다.**

> **Pointer, Struct, Bit Operation, `volatile`, Function은 Embedded
> Driver 구조에서 하나로 합쳐진다.**

------------------------------------------------------------------------

# 47. 현재 학습 진행 상태

``` text
C_Cpp_Study/
├── 01_C/
│   ├── Array_String.md    ✓
│   ├── Pointer.md         ✓
│   ├── Struct.md          ✓
│   ├── malloc_free.md     ✓
│   ├── Bit_Operation.md   ✓
│   ├── volatile.md        ✓
│   └── Embedded_C.md      ← 현재
└── 02_CPP/
    ├── Reference.md
    ├── Class.md
    ├── RAII.md
    ├── Template.md
    ├── STL.md
    └── Smart_Pointer.md
```

``` mermaid
flowchart LR
    A["C Fundamentals"] --> B["Embedded C"]
    B --> C["C++ Reference"]
    C --> D["Class"]
    D --> E["RAII"]
    E --> F["STL"]
```

------------------------------------------------------------------------

# 48. 다음 학습

C 기반 저수준 개념을 정리했으므로 다음부터 C++ 핵심 개념으로 이동한다.

**다음 문서:** `02_CPP/Reference.md`

``` mermaid
flowchart LR
    A["C Pointer"] --> B["C++ Reference"]
    B --> C["const Reference"]
    C --> D["Class"]
    D --> E["RAII"]
    E --> F["STL"]
```

다음 문서의 핵심 개념:

``` text
Reference란 무엇인가?
Pointer와 Reference의 차이는 무엇인가?
lvalue reference란 무엇인가?
const reference는 왜 사용하는가?
함수 Parameter에서 Reference는 어떤 의미인가?
C Pointer Parameter와 C++ Reference Parameter는 어떻게 다른가?
Reference가 Class, RAII, STL과 어떻게 연결되는가?
```
