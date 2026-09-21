# Memory Model

> **학습 목표**
>
> C/C++ 프로그램의 데이터를 단순히 "변수"로만 보지 않고 **객체의 저장
> 위치(Storage), 저장 기간(Storage Duration), 수명(Lifetime),
> 주소(Address), 포인터(Pointer)** 관점으로 이해한다.
>
> 특히 C 학습에서는 **지역 변수, 전역 변수, `static`, 동적 메모리,
> Stack/Heap의 일반적인 구현 모델**을 구분하고, 포인터 버그가 왜
> 발생하는지 설명할 수 있어야 한다.
>
> 이 노트는 `C_Cpp_Study/00_Common/Memory_Model.md`에 위치하며
> `01_C/Pointer.md` 다음 단계로 학습한다.

------------------------------------------------------------------------

# 1. 왜 Memory Model을 배우는가?

다음 코드는 단순해 보인다.

``` c
int number = 10;
```

하지만 메모리 관점에서는 여러 질문이 생긴다.

``` text
number는 어디에 저장되는가?
언제 만들어지는가?
언제까지 존재하는가?
주소는 무엇인가?
함수가 끝나면 어떻게 되는가?
포인터로 계속 접근할 수 있는가?
```

``` mermaid
flowchart LR
    A["Variable"] --> B["Object"]
    B --> C["Storage"]
    B --> D["Lifetime"]
    B --> E["Address"]
    E --> F["Pointer"]
```

> **핵심 관점:** 포인터의 안전성은 주소값 자체만이 아니라 **그 주소가
> 가리키는 객체가 아직 유효한가**와 연결된다.

------------------------------------------------------------------------

# 2. 먼저 구분할 개념

초기 학습에서는 다음 개념을 구분한다.

  개념               질문
  ------------------ -------------------------------------------------
  Scope              이름을 어디에서 사용할 수 있는가?
  Storage Duration   객체를 위한 저장 공간이 얼마나 오래 유지되는가?
  Lifetime           객체가 언제부터 언제까지 존재하는가?
  Address            객체가 메모리 어디에 있는가?
  Pointer            그 주소를 어떻게 다루는가?

이 개념들은 서로 관련되지만 완전히 같은 것은 아니다.

``` mermaid
flowchart TD
    A["Object"] --> B["Scope"]
    A --> C["Storage Duration"]
    A --> D["Lifetime"]
    A --> E["Address"]
    E --> F["Pointer"]
```

------------------------------------------------------------------------

# 3. Scope와 Lifetime은 다르다

예:

``` c
void function(void)
{
    int value = 10;
}
```

`value`라는 **이름**은 해당 블록 안에서 사용할 수 있다.

``` text
{
    int value = 10;

    // value 사용 가능
}

// value 이름 사용 불가
```

이것은 Scope의 문제다.

반면 객체가 언제 생성되고 언제까지 유효한지는 Lifetime/Storage
Duration과 관련된다.

> \[!IMPORTANT\] **Scope는 이름을 사용할 수 있는 영역이고, Lifetime은
> 객체의 존재와 관련된 개념이다.**

------------------------------------------------------------------------

# 4. 프로그램 메모리의 개념적 구조

일반적인 실행 환경을 설명할 때 다음과 같은 그림을 자주 사용한다.

``` text
높은 주소
┌────────────────────┐
│ Stack              │
├────────────────────┤
│                    │
│ Free / Unused      │
│                    │
├────────────────────┤
│ Heap               │
├────────────────────┤
│ Static / Global    │
├────────────────────┤
│ Code / Read-only   │
└────────────────────┘
낮은 주소
```

``` mermaid
flowchart TD
    A["Program Memory"] --> B["Code / Read-only"]
    A --> C["Static / Global Storage"]
    A --> D["Heap"]
    A --> E["Stack"]
```

> \[!WARNING\] 이 그림은 **일반적인 구현을 이해하기 위한 개념도**다.
> C/C++ 언어 표준이 모든 시스템에서 정확히 이런 물리적 배치를 요구하는
> 것은 아니다.
>
> 특히 MCU, RTOS, linker script, 운영체제에 따라 실제 메모리 배치는
> 달라질 수 있다.

------------------------------------------------------------------------

# 5. 자동 저장 기간과 지역 변수

대표적인 지역 변수:

``` c
void function(void)
{
    int value = 10;
}
```

일반적인 구현에서는 이런 자동 저장 기간 객체가 함수 호출과 관련된 Stack
영역에 배치되는 경우가 많다.

개념:

``` mermaid
sequenceDiagram
    participant C as Caller
    participant F as function()
    participant V as value

    C->>F: 함수 호출
    F->>V: value 사용
    F-->>C: 함수 반환
    Note over V: 자동 저장 기간 종료
```

초기 학습에서는 흔히 다음처럼 기억한다.

``` text
함수 호출
 ↓
지역 자동 변수 사용
 ↓
함수 종료
 ↓
해당 객체에 더 이상 접근하면 안 됨
```

------------------------------------------------------------------------

# 6. Stack

Stack은 함수 호출과 지역 데이터 관리에 흔히 사용되는 메모리 영역이다.

개념적으로:

``` text
main()
┌────────────────────┐
│ main local data    │
└────────────────────┘

main() → funcA()

┌────────────────────┐
│ funcA local data   │
├────────────────────┤
│ main local data    │
└────────────────────┘
```

`funcA()`가 끝나면:

``` text
┌────────────────────┐
│ main local data    │
└────────────────────┘
```

``` mermaid
flowchart TD
    A["main()"] --> B["funcA() 호출"]
    B --> C["funcA용 실행 정보/지역 데이터"]
    C --> D["funcA() 종료"]
    D --> E["호출 이전 상태로 복귀"]
```

------------------------------------------------------------------------

# 7. Stack Frame 개념

함수 호출마다 필요한 정보를 묶어 설명할 때 **Stack Frame**이라는 표현을
자주 사용한다.

개념적으로 포함될 수 있는 것:

``` text
매개변수
지역 변수
반환과 관련된 정보
저장된 레지스터
```

``` mermaid
flowchart TD
    A["Function Call"] --> B["Stack Frame"]
    B --> C["Local Variables"]
    B --> D["Parameters"]
    B --> E["Return-related Data"]
```

> \[!NOTE\] 실제 컴파일러는 최적화에 따라 변수를 레지스터에 둘 수도 있고
> Stack Frame의 구체적 형태도 달라질 수 있다. Stack 그림은 실행 모델을
> 이해하기 위한 출발점으로 사용한다.

------------------------------------------------------------------------

# 8. 지역 변수의 주소를 반환하면 왜 위험한가?

잘못된 예:

``` c
int *create_value(void)
{
    int value = 10;

    return &value;
}
```

함수가 끝나면 `value`의 수명이 종료된다.

그러나 반환된 포인터에는 이전 주소가 남아 있을 수 있다.

``` mermaid
flowchart LR
    A["value 존재"] --> B["&value 반환"]
    B --> C["함수 종료"]
    C --> D["value 수명 종료"]
    D --> E["포인터는 이전 주소 보유"]
    E --> F["Dangling Pointer"]
```

이후:

``` c
int *ptr = create_value();

printf("%d\n", *ptr);
```

처럼 역참조하면 정의되지 않은 동작이다.

> \[!WARNING\] **주소값이 남아 있다는 것과 그 주소의 객체가 여전히 살아
> 있다는 것은 다른 문제다.**

------------------------------------------------------------------------

# 9. Dangling Pointer

### 한 문장 정의

**Dangling Pointer는 더 이상 유효하지 않은 객체를 가리키는 포인터다.**

대표적인 발생 원인:

``` text
지역 객체의 수명 종료
동적 메모리 free 후 계속 사용
객체의 수명이 끝났는데 주소를 보관
```

``` mermaid
flowchart TD
    A["Valid Pointer"] --> B["Target Lifetime Ends"]
    B --> C["Pointer Still Stores Address"]
    C --> D["Dangling Pointer"]
```

`ptr != NULL`이라고 해서 dangling 상태가 아닌 것은 아니다.

------------------------------------------------------------------------

# 10. 정적 저장 기간

전역 변수와 `static`으로 선언된 특정 객체는 정적 저장 기간을 가진다.

예:

``` c
int global_value = 10;

void function(void)
{
    static int count = 0;

    count++;
}
```

이러한 객체의 저장 공간은 프로그램 실행 전체 기간 동안 유지된다.

``` mermaid
flowchart LR
    A["Program Start"] --> B["Static Storage Objects"]
    B --> C["Program Running"]
    C --> D["Program End"]
```

------------------------------------------------------------------------

# 11. 전역 변수

함수 밖에서 선언한 예:

``` c
int global_value = 10;

int main(void)
{
    global_value++;

    return 0;
}
```

전역 객체는 정적 저장 기간을 가진다.

장점이 될 수 있는 점:

``` text
여러 함수에서 공유 가능
프로그램 실행 동안 상태 유지
```

하지만 과도하게 사용하면:

``` text
의존성 증가
변경 위치 추적 어려움
테스트 어려움
동시성 문제 가능
```

등의 문제가 생길 수 있다.

> **전역 변수는 "편리하니까 사용"보다 수명과 공유 범위를 의식하고
> 사용한다.**

------------------------------------------------------------------------

# 12. 지역 `static`

``` c
void counter(void)
{
    static int count = 0;

    count++;

    printf("%d\n", count);
}
```

호출:

``` c
counter();
counter();
counter();
```

개념적인 출력:

``` text
1
2
3
```

`count`의 이름은 함수 내부에서 사용되지만 저장 공간은 호출이 끝나도
유지된다.

``` mermaid
flowchart TD
    A["counter() 1회"] --> B["count = 1"]
    B --> C["함수 종료"]
    C --> D["count 저장 상태 유지"]
    D --> E["counter() 2회"]
    E --> F["count = 2"]
```

이 예는 Scope와 Storage Duration이 다른 개념임을 잘 보여준다.

------------------------------------------------------------------------

# 13. 자동 객체와 static 객체 비교

  항목                   일반 지역 자동 변수   지역 `static`
  ---------------------- --------------------- -----------------------
  선언 위치              함수/블록 내부        함수/블록 내부
  이름의 Scope           해당 블록             해당 블록
  저장 기간              자동                  정적
  호출 종료 후 값 유지   아니오                예
  일반적인 구현 이미지   Stack과 연관          Static storage와 연관

------------------------------------------------------------------------

# 14. 초기화의 차이

다음 지역 자동 변수는:

``` c
void function(void)
{
    int value;
}
```

초기화하지 않고 값을 읽으면 안 된다.

반면 정적 저장 기간 객체는 명시적 초기값이 없으면 언어 규칙에 따라 0으로
초기화된다.

``` c
static int count;
```

전역 변수도 마찬가지다.

``` c
int global_value;
```

> \[!IMPORTANT\] **"모든 변수는 자동으로 0이 된다"는 규칙은 없다.**
>
> 저장 기간과 초기화 규칙을 구분한다.

------------------------------------------------------------------------

# 15. 동적 메모리

프로그램 실행 중 필요한 크기의 저장 공간을 요청해서 사용하는 방식을 동적
메모리 할당이라고 한다.

C에서는 대표적으로:

``` c
malloc()
calloc()
realloc()
free()
```

를 사용한다.

간단한 예:

``` c
#include <stdlib.h>

int *ptr = malloc(sizeof(int));
```

``` mermaid
flowchart LR
    A["malloc"] --> B["Dynamic Storage"]
    B --> C["Address 반환"]
    C --> D["Pointer"]
```

이후 사용이 끝나면:

``` c
free(ptr);
```

를 호출한다.

------------------------------------------------------------------------

# 16. Heap

동적 메모리를 설명할 때 흔히 **Heap**이라는 구현상의 메모리 영역을
사용한다.

``` text
malloc()
   ↓
Heap에서 사용 가능한 저장 공간 확보
   ↓
주소 반환
   ↓
Pointer로 접근
```

``` mermaid
flowchart TD
    A["Request"] --> B["malloc"]
    B --> C["Heap / Dynamic Storage"]
    C --> D["Pointer"]
    D --> E["Use"]
    E --> F["free"]
```

> \[!NOTE\] C 언어의 동적 할당 규칙과 특정 시스템의 실제 Heap 구현은
> 구분해서 이해한다. 이후 `malloc_free.md`에서 더 자세히 다룬다.

------------------------------------------------------------------------

# 17. `malloc()` 기본 예제

``` c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *ptr = malloc(sizeof(int));

    if (ptr == NULL)
    {
        return 1;
    }

    *ptr = 100;

    printf("%d\n", *ptr);

    free(ptr);

    return 0;
}
```

흐름:

``` mermaid
flowchart TD
    A["malloc(sizeof(int))"] --> B{"ptr == NULL?"}
    B -->|"Yes"| C["할당 실패 처리"]
    B -->|"No"| D["*ptr = 100"]
    D --> E["사용"]
    E --> F["free(ptr)"]
```

------------------------------------------------------------------------

# 18. `free()` 이후

``` c
int *ptr = malloc(sizeof(int));

if (ptr != NULL)
{
    *ptr = 10;
    free(ptr);
}
```

`free(ptr)` 이후 해당 동적 할당 영역의 수명은 끝난다.

그 후:

``` c
printf("%d\n", *ptr);
```

처럼 접근하면 안 된다.

이것이 **Use-After-Free**다.

``` mermaid
flowchart LR
    A["malloc"] --> B["Valid Object"]
    B --> C["free"]
    C --> D["Lifetime End"]
    D --> E["*ptr 접근"]
    E --> F["Use-After-Free"]
```

------------------------------------------------------------------------

# 19. `free(ptr); ptr = NULL;`

다음 패턴을 볼 수 있다.

``` c
free(ptr);
ptr = NULL;
```

`free()` 후 포인터 변수에 `NULL`을 대입하면 해당 포인터가 이전 해제
주소를 계속 보유하지 않도록 할 수 있다.

하지만 주의해야 한다.

``` text
다른 포인터가 같은 동적 객체를 가리키고 있었다면?
```

예:

``` c
int *a = malloc(sizeof(int));
int *b = a;

free(a);
a = NULL;
```

이때 `b`는 자동으로 `NULL`이 되지 않는다.

``` mermaid
flowchart TD
    A["Heap Object"] --> B["a"]
    A --> C["b"]
    D["free(a)"] --> E["Heap Object 수명 종료"]
    E --> F["a = NULL"]
    E --> G["b는 이전 주소 보유"]
    G --> H["Dangling"]
```

------------------------------------------------------------------------

# 20. Memory Leak

동적으로 할당한 저장 공간을 더 이상 사용할 방법이 없는데 `free()`하지
않은 경우 메모리 누수(Memory Leak)가 발생할 수 있다.

예:

``` c
int *ptr = malloc(sizeof(int));

ptr = NULL;
```

할당에 성공했다면 원래 반환된 주소를 잃어버렸기 때문에 해당 메모리를
`free()`할 방법이 사라졌다.

``` mermaid
flowchart LR
    A["malloc"] --> B["Allocated Memory"]
    B --> C["ptr"]
    C --> D["ptr = NULL"]
    B --> E["접근 경로 상실"]
    E --> F["Memory Leak"]
```

------------------------------------------------------------------------

# 21. Double Free

같은 동적 할당을 두 번 해제하면 안 된다.

``` c
free(ptr);
free(ptr);
```

두 번째 `free()`는 잘못된 동작이다.

``` mermaid
flowchart LR
    A["Allocated"] --> B["free"]
    B --> C["Released"]
    C --> D["free again"]
    D --> E["Invalid"]
```

> \[!WARNING\] `free`, Use-After-Free, Double Free는 C에서 매우 중요한
> 메모리 버그다.

------------------------------------------------------------------------

# 22. Stack과 Heap 비교

초기 학습용 비교:

  -----------------------------------------------------------------------
  항목                    Stack과 연관된 자동     Heap과 연관된 동적 할당
                          객체                    
  ----------------------- ----------------------- -----------------------
  생성/확보               함수/블록 실행과 연관   `malloc()` 등

  종료/해제               저장 기간 종료          `free()` 필요

  크기                    일반적으로 제한적       시스템/Heap 상태에 의존

  관리                    자동                    프로그래머가 직접 관리

  대표 위험               지역 객체 주소의 잘못된 Leak, Use-After-Free,
                          사용, Stack overflow    Double Free
  -----------------------------------------------------------------------

> \[!NOTE\] "지역 변수는 무조건 Stack에 있다"처럼 절대적인 물리 배치
> 규칙으로 외우지 않는다. 최적화와 플랫폼 구현에 따라 달라질 수 있다.

------------------------------------------------------------------------

# 23. Stack Overflow

Stack 공간을 과도하게 사용하면 문제가 발생할 수 있다.

대표 원인:

``` text
너무 깊은 재귀 호출
매우 큰 지역 배열
제한된 MCU Stack
```

예:

``` c
void recursive(void)
{
    int buffer[1024];

    recursive();
}
```

이 코드는 재귀가 끝나지 않으며 호출이 계속 쌓인다.

``` mermaid
flowchart TD
    A["recursive()"] --> B["recursive()"]
    B --> C["recursive()"]
    C --> D["recursive()"]
    D --> E["..."]
    E --> F["Stack 사용 증가"]
    F --> G["Stack Overflow 가능"]
```

------------------------------------------------------------------------

# 24. 임베디드에서 Stack이 중요한 이유

MCU는 RAM이 제한적인 경우가 많다.

예를 들어 여러 함수가 큰 지역 배열을 사용하거나 RTOS의 각 Task가 별도
Stack을 가진다면 RAM 사용량이 빠르게 증가할 수 있다.

``` mermaid
flowchart TD
    A["Limited RAM"] --> B["Main / ISR Stack"]
    A --> C["RTOS Task Stack"]
    C --> D["Task A"]
    C --> E["Task B"]
    C --> F["Task C"]
```

따라서 펌웨어에서는 다음을 의식해야 한다.

``` text
지역 배열 크기
함수 호출 깊이
재귀 사용 여부
ISR Stack 사용량
RTOS Task Stack 크기
```

------------------------------------------------------------------------

# 25. 전역/static 메모리와 임베디드

임베디드에서는 정적 저장 기간 객체의 크기를 링크 시점에 비교적 명확하게
파악할 수 있는 경우가 많다.

예:

``` c
static unsigned char rx_buffer[256];
static unsigned char tx_buffer[256];
```

이런 버퍼는 프로그램 실행 전체 동안 저장 공간이 유지된다.

``` mermaid
flowchart LR
    A["Static Buffer"] --> B["Known Lifetime"]
    B --> C["UART RX/TX"]
    C --> D["Firmware"]
```

동적 메모리를 제한하거나 사용하지 않는 펌웨어 설계에서 정적 버퍼가 자주
등장하는 이유 중 하나다.

------------------------------------------------------------------------

# 26. 초기화된 전역/정적 데이터와 초기화되지 않은 데이터

실행 파일과 임베디드 메모리 맵을 공부하면 흔히 다음 용어를 만나게 된다.

``` text
.data
.bss
.rodata
.text
```

일반적인 개념:

  Section     대표 내용
  ----------- -------------------------------------------
  `.text`     실행 코드
  `.rodata`   읽기 전용 상수 데이터
  `.data`     초기값이 있는 writable static/global data
  `.bss`      0으로 초기화되는 static/global data

``` mermaid
flowchart TD
    A["Executable / Firmware Image"] --> B[".text"]
    A --> C[".rodata"]
    A --> D[".data"]
    A --> E[".bss"]
```

> \[!IMPORTANT\] 실제 Section 이름과 배치는 toolchain, linker script,
> 플랫폼에 따라 달라질 수 있다. 이 표는 매우 흔한 구조를 설명하기 위한
> 것이다.

------------------------------------------------------------------------

# 27. `.data` 예시

``` c
int global_value = 10;
static int mode = 1;
```

이처럼 명시적인 0이 아닌 초기값을 가진 정적 저장 기간 데이터는 일반적인
toolchain에서 `.data`와 연결되는 경우가 많다.

임베디드에서는 초기값을 Flash/ROM 이미지에 저장해 두었다가 시작 코드가
RAM으로 복사하는 구조를 흔히 볼 수 있다.

``` mermaid
flowchart LR
    A["Flash 초기값"] --> B["Startup Code"]
    B --> C["RAM .data"]
```

------------------------------------------------------------------------

# 28. `.bss` 예시

``` c
int global_counter;
static unsigned char buffer[128];
```

명시적 초기값이 없고 0으로 초기화되는 정적 저장 기간 데이터는 일반적인
toolchain에서 `.bss`와 연결된다.

시작 과정에서 해당 RAM 영역을 0으로 만드는 형태가 흔하다.

``` mermaid
flowchart LR
    A["Startup"] --> B[".bss 영역"]
    B --> C["Zero Initialize"]
    C --> D["main()"]
```

------------------------------------------------------------------------

# 29. 프로그램 시작 전에는 무슨 일이 일어나는가?

임베디드에서는 `main()`이 실행되기 전에 startup code가 동작하는 경우가
일반적이다.

개념:

``` mermaid
flowchart LR
    A["Reset"] --> B["Startup Code"]
    B --> C["Stack 설정"]
    C --> D[".data 초기화"]
    D --> E[".bss 초기화"]
    E --> F["System Init"]
    F --> G["main()"]
```

MCU와 toolchain에 따라 구체적인 과정은 달라질 수 있다.

이 내용은 이후 `Embedded_C.md`와 MCU startup/linker 학습에서 확장한다.

------------------------------------------------------------------------

# 30. Flash와 RAM

전형적인 MCU에서는 코드와 초기값이 비휘발성 메모리(예: Flash)에 저장되고
실행 중 변경되는 데이터는 RAM에 위치하는 구조가 흔하다.

개념:

``` mermaid
flowchart TD
    A["Flash"] --> B["Program Code"]
    A --> C["Constant / Initial Data"]
    D["RAM"] --> E["Stack"]
    D --> F["Static / Global Data"]
    D --> G["Heap if used"]
```

> \[!NOTE\] MCU마다 메모리 구조가 다르므로 실제 프로젝트에서는 반드시
> 데이터시트와 linker script를 확인한다.

------------------------------------------------------------------------

# 31. 포인터와 객체 수명

포인터 안전성을 판단할 때 다음 질문을 순서대로 한다.

``` mermaid
flowchart TD
    A["Pointer가 어떤 주소를 갖는가?"] --> B["그 주소에 대상 객체가 존재하는가?"]
    B --> C["객체의 Lifetime이 유효한가?"]
    C --> D["접근 범위가 유효한가?"]
    D --> E["타입에 맞게 접근하는가?"]
```

단순히:

``` c
ptr != NULL
```

만 검사하는 것으로 충분하지 않은 이유다.

------------------------------------------------------------------------

# 32. 지역 변수 주소 반환 문제 다시 보기

``` c
int *wrong(void)
{
    int value = 10;

    return &value;
}
```

문제를 Memory Model로 분석한다.

``` text
value
 ↓
자동 저장 기간
 ↓
함수 실행 중 유효
 ↓
함수 반환
 ↓
value 수명 종료
 ↓
반환된 주소를 역참조할 수 없음
```

``` mermaid
flowchart LR
    A["Pointer 문법"] --> B["Address 반환"]
    B --> C["Memory Model"]
    C --> D["Lifetime 종료"]
    D --> E["Dangling Pointer"]
```

이제 포인터 버그를 단순 문법 실수가 아니라 **수명 문제**로 볼 수 있다.

------------------------------------------------------------------------

# 33. 함수 호출과 메모리 추적

``` c
void foo(void)
{
    int a = 10;
}

int main(void)
{
    int x = 20;

    foo();

    return 0;
}
```

개념적인 호출 흐름:

``` text
main 시작
┌────────────┐
│ x = 20     │
└────────────┘

foo 호출
┌────────────┐
│ a = 10     │
├────────────┤
│ x = 20     │
└────────────┘

foo 종료
┌────────────┐
│ x = 20     │
└────────────┘
```

이 그림은 일반적인 Stack 기반 실행 모델을 이해하기 위한 것이다.

------------------------------------------------------------------------

# 34. 재귀와 Memory Model

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

각 호출은 별도의 실행 상태를 가진다.

``` mermaid
flowchart TD
    A["factorial(4)"] --> B["factorial(3)"]
    B --> C["factorial(2)"]
    C --> D["factorial(1)"]
    D --> E["return"]
    E --> C
    C --> B
    B --> A
```

재귀 호출 깊이가 커질수록 Stack 사용량도 증가할 수 있다.

임베디드에서는 이 점 때문에 재귀 사용을 제한하는 코딩 규칙을 적용하는
프로젝트도 있다.

------------------------------------------------------------------------

# 35. 배열과 메모리

``` c
int numbers[4] = {10, 20, 30, 40};
```

배열 요소는 연속적으로 배치된다.

``` text
┌────────────┐
│ numbers[0] │
├────────────┤
│ numbers[1] │
├────────────┤
│ numbers[2] │
├────────────┤
│ numbers[3] │
└────────────┘
```

``` mermaid
flowchart LR
    A["Array"] --> B["Contiguous Elements"]
    B --> C["Address"]
    C --> D["Pointer Arithmetic"]
```

`Array_String.md`와 `Pointer.md`의 내용이 Memory Model에서 다시
연결된다.

------------------------------------------------------------------------

# 36. 문자열 리터럴과 메모리

다음 두 코드를 구분한다.

``` c
char text[] = "Hello";
```

``` c
const char *text = "Hello";
```

첫 번째는 문자 배열 객체를 만든다.

두 번째는 문자열 리터럴을 가리키는 포인터를 만든다.

문자열 리터럴의 저장 위치를 특정 물리 영역 하나로 절대화하기보다,
**문자열 리터럴을 수정하려고 하면 안 된다**는 언어 규칙을 우선 기억한다.

``` c
const char *text = "Hello";
```

C 코드에서 문자열 리터럴을 수정하려는 동작은 정의되지 않는다.

------------------------------------------------------------------------

# 37. `const`와 Memory Model

`const`는 객체의 물리적 저장 위치를 의미하지 않는다.

``` c
const int value = 10;
```

`const`의 핵심은 해당 표현을 통해 객체를 수정할 수 없도록 타입 수준의
제약을 제공하는 것이다.

``` mermaid
flowchart TD
    A["const"] --> B["Modification Constraint"]
    A -.-> C["특정 물리 메모리 위치를 직접 의미하지 않음"]
```

임베디드에서 `const` 데이터가 Flash/ROM과 연관되는 경우가 있지만 실제
배치는 toolchain/linker 설정에 따라 확인해야 한다.

------------------------------------------------------------------------

# 38. `volatile`과 Memory Model 예고

임베디드에서는 다음과 같은 코드가 등장한다.

``` c
volatile unsigned int status;
```

또는 레지스터 포인터:

``` c
volatile unsigned int *reg;
```

`volatile`은 메모리 접근이 프로그램 외부 요인이나 하드웨어에 의해
관찰/변경될 수 있는 상황과 관련해 컴파일러 최적화에 영향을 주는 타입
한정자다.

``` mermaid
flowchart LR
    A["Memory"] --> B["volatile"]
    B --> C["Hardware Register"]
    B --> D["ISR와 공유되는 일부 객체"]
```

> \[!WARNING\] `volatile`은 thread synchronization이나 atomicity를
> 제공하는 기능이 아니다.

세부 내용은 `01_C/volatile.md`에서 학습한다.

------------------------------------------------------------------------

# 39. MMIO와 Memory Model

Memory-Mapped I/O에서는 특정 주소 범위가 일반 RAM이 아니라 하드웨어
주변장치 레지스터와 연결될 수 있다.

``` mermaid
flowchart TD
    A["CPU Address Space"] --> B["RAM"]
    A --> C["Flash"]
    A --> D["Peripheral Registers"]
    D --> E["GPIO"]
    D --> F["UART"]
    D --> G["Timer"]
```

따라서 임베디드에서 "주소"는 단순히 RAM 변수의 위치만 의미하지 않는다.

``` text
Address
├── RAM Object
├── Flash/Code/Data
└── Peripheral Register
```

------------------------------------------------------------------------

# 40. PC에서의 메모리와 MCU 메모리

일반 PC 프로그램에서는 운영체제가 프로세스에 가상 주소 공간을 제공하는
환경이 흔하다.

MCU bare-metal 환경에서는 물리적인 메모리 맵과 주변장치 주소를 직접
다루는 경우가 많다.

``` mermaid
flowchart TD
    A["Memory Model"] --> B["Hosted / OS Environment"]
    A --> C["Bare-metal MCU"]

    B --> D["Process Address Space"]
    C --> E["MCU Memory Map"]
    E --> F["Flash"]
    E --> G["SRAM"]
    E --> H["Peripheral"]
```

이 차이는 임베디드 학습에서 매우 중요하다.

------------------------------------------------------------------------

# 41. Linker Script로의 연결

임베디드 프로젝트에서는 Linker Script가 코드와 데이터를 어느 메모리
영역에 배치할지 결정하는 데 핵심 역할을 한다.

개념:

``` mermaid
flowchart LR
    A[".text / .data / .bss"] --> B["Linker"]
    C["Linker Script"] --> B
    B --> D["Flash / RAM 배치"]
```

현재는 다음 정도만 기억한다.

``` text
Compiler
 ↓
Object Files
 ↓
Linker
 ↓
Linker Script
 ↓
Final Memory Layout
```

컴파일/링크 단계는 별도 노트에서 확장한다.

------------------------------------------------------------------------

# 42. Memory Map 읽기 예고

MCU 데이터시트에서는 다음과 같은 메모리 맵을 볼 수 있다.

``` text
Address Range          Region
──────────────────     ─────────
0x00000000 ...         Flash
0x20000000 ...         SRAM
0x40000000 ...         Peripheral
...
```

정확한 주소는 MCU마다 다르다.

``` mermaid
flowchart TD
    A["MCU Memory Map"] --> B["Flash Region"]
    A --> C["SRAM Region"]
    A --> D["Peripheral Region"]
```

포인터와 Memory Model을 이해하면 데이터시트의 Memory Map이 실제 C 코드와
연결되기 시작한다.

------------------------------------------------------------------------

# 43. Memory Model에서 중요한 버그

``` mermaid
mindmap
  root((Memory Bugs))
    Lifetime
      Dangling Pointer
      Use After Free
      Local Address Return
    Dynamic Memory
      Memory Leak
      Double Free
    Range
      Buffer Overflow
      Out of Bounds
    Stack
      Stack Overflow
```

이 버그들은 대부분 다음 질문과 연결된다.

> **이 메모리는 지금도 유효한가?**

------------------------------------------------------------------------

# 44. 실습 1 --- 지역 변수 수명

다음 코드를 분석한다.

``` c
void function(void)
{
    int value = 10;
}
```

설명할 것:

``` text
value의 Scope
value의 Storage Duration
함수 종료 후 value 객체에 접근 가능한지
```

------------------------------------------------------------------------

# 45. 실습 2 --- 지역 `static`

``` c
#include <stdio.h>

void counter(void)
{
    static int count = 0;

    count++;

    printf("%d\n", count);
}

int main(void)
{
    counter();
    counter();
    counter();

    return 0;
}
```

출력을 예상한 뒤 이유를 Scope와 Storage Duration으로 설명한다.

------------------------------------------------------------------------

# 46. 실습 3 --- 잘못된 주소 반환

``` c
int *get_value(void)
{
    int value = 100;

    return &value;
}
```

다음 질문에 답한다.

``` text
왜 컴파일이 된다고 해서 안전한 코드가 아닌가?
value의 수명은 언제 끝나는가?
반환된 포인터는 어떤 상태가 되는가?
```

------------------------------------------------------------------------

# 47. 실습 4 --- 동적 메모리

다음 프로그램을 직접 작성한다.

``` text
1. int 하나를 malloc
2. 실패 여부 확인
3. 100 저장
4. 값 출력
5. free
```

기본 형태:

``` c
int *ptr = malloc(sizeof(int));

if (ptr == NULL)
{
    /* 실패 처리 */
}
```

------------------------------------------------------------------------

# 48. 실습 5 --- 배열 동적 할당 예고

정수 `count`개를 저장할 공간을 동적으로 확보하는 형태를 확인한다.

``` c
int *numbers = malloc(sizeof(int) * count);
```

사용 후:

``` c
free(numbers);
```

현재는 구조만 익히고 overflow, `size_t`, `calloc`, `realloc` 등의 세부
안전성은 `malloc_free.md`에서 다룬다.

------------------------------------------------------------------------

# 49. 실습 6 --- 메모리 영역 분류

다음 코드의 각 객체를 분류한다.

``` c
int global_value = 10;
static int global_static;

void function(void)
{
    int local_value = 20;
    static int local_static = 30;
}
```

표를 채운다.

  객체              Scope   Storage Duration   일반적인 구현 이미지
  ----------------- ------- ------------------ ----------------------
  `global_value`                               
  `global_static`                              
  `local_value`                                
  `local_static`                               

------------------------------------------------------------------------

# 50. 실습 7 --- Memory Leak 찾기

``` c
int *ptr = malloc(sizeof(int));

if (ptr == NULL)
{
    return;
}

*ptr = 100;

ptr = NULL;
```

무엇이 문제인지 설명한다.

------------------------------------------------------------------------

# 51. 실습 8 --- Use-After-Free 찾기

``` c
int *ptr = malloc(sizeof(int));

if (ptr != NULL)
{
    *ptr = 100;

    free(ptr);

    printf("%d\n", *ptr);
}
```

잘못된 접근 위치를 찾고 이유를 설명한다.

------------------------------------------------------------------------

# 52. 실습 9 --- Stack 사용량 생각하기

다음 두 함수를 비교한다.

``` c
void a(void)
{
    int value = 0;
}
```

``` c
void b(void)
{
    unsigned char buffer[4096];
}
```

제한된 RAM을 가진 MCU에서 어떤 차이를 고려해야 하는지 설명한다.

------------------------------------------------------------------------

# 53. 코드 추적 연습

``` c
int global_value = 1;

void function(void)
{
    static int count = 0;
    int local = 10;

    count++;
    global_value++;
}
```

`function()`을 세 번 호출한다고 가정한다.

    호출   `count`   `global_value`   `local`
  ------ --------- ---------------- ---------
       1                            
       2                            
       3                            

`local`과 `count`의 차이를 설명한다.

------------------------------------------------------------------------

# 54. 반드시 설명할 수 있어야 하는 것

-   [ ] Scope와 Lifetime이 다른 개념임을 설명할 수 있다.
-   [ ] Storage Duration의 의미를 대략 설명할 수 있다.
-   [ ] 일반적인 Stack의 역할을 설명할 수 있다.
-   [ ] Stack Frame의 개념을 설명할 수 있다.
-   [ ] 지역 자동 객체의 주소를 함수 밖에서 잘못 사용하는 문제를 설명할
    수 있다.
-   [ ] Dangling Pointer를 설명할 수 있다.
-   [ ] 전역 객체의 저장 기간을 설명할 수 있다.
-   [ ] 지역 `static`이 일반 지역 변수와 어떻게 다른지 설명할 수 있다.
-   [ ] 정적 저장 기간 객체의 기본 0 초기화를 설명할 수 있다.
-   [ ] 동적 메모리 할당의 목적을 설명할 수 있다.
-   [ ] `malloc()`과 `free()`의 기본 관계를 설명할 수 있다.
-   [ ] Memory Leak을 설명할 수 있다.
-   [ ] Use-After-Free를 설명할 수 있다.
-   [ ] Double Free를 설명할 수 있다.
-   [ ] Stack Overflow가 발생할 수 있는 상황을 설명할 수 있다.
-   [ ] `.text`, `.data`, `.bss`, `.rodata`의 일반적인 역할을 구분할 수
    있다.
-   [ ] 실제 Section 배치가 linker/toolchain에 의존함을 설명할 수 있다.
-   [ ] MCU에서 Flash, SRAM, Peripheral 영역이 존재할 수 있음을 설명할
    수 있다.
-   [ ] MMIO가 Memory Model과 어떻게 연결되는지 설명할 수 있다.
-   [ ] 임베디드에서 Stack 크기를 의식해야 하는 이유를 설명할 수 있다.
-   [ ] Linker Script가 메모리 배치와 연결된다는 것을 설명할 수 있다.

------------------------------------------------------------------------

# 55. 복습 문제

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q1. Scope와 Lifetime의 차이는?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** Scope는 이름을 사용할 수 있는 영역이고, Lifetime은 객체가
유효하게 존재하는 기간과 관련된다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q2. 함수 내부의 일반 지역 변수는 함수가 끝난 뒤
포인터로 계속 접근해도 되는가?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 해당 자동 객체의 수명이 종료되므로 그 객체를 가리키던 포인터로
접근하면 안 된다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q3. 지역 static 변수는 함수 호출이 끝나면 값이
사라지는가?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 아니다. 정적 저장 기간을 가지므로 저장 상태가 프로그램 실행
동안 유지된다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q4. malloc으로 얻은 메모리는 언제
해제되는가?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 프로그램이 명시적으로 `free()`하는 방식으로 관리해야 한다.
정확한 동적 할당 규칙은 `malloc_free.md`에서 확장한다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q5. Memory Leak이란?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 동적으로 확보한 메모리가 더 이상 필요하지 않지만 적절히
해제되지 않아 사용 가능한 메모리가 낭비되는 상황이다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q6. Use-After-Free란?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** `free()`로 수명이 끝난 동적 할당 영역을 포인터로 다시 접근하는
오류다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q7. ptr != NULL이면 항상 안전한가?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 아니다. `NULL`이 아니어도 수명이 끝난 객체를 가리키는 dangling
pointer일 수 있다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q8. .bss는 일반적으로 어떤 데이터와
연결되는가?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** 0으로 초기화되는 정적/전역 데이터와 연결되는 경우가
일반적이다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q9. 임베디드에서 main() 전에 수행될 수 있는 초기화
작업은?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** Stack 설정, `.data` 초기화, `.bss` 0 초기화, 시스템 초기화
등이 일반적인 예다. 실제 과정은 MCU/toolchain에 따라 달라진다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q10. Memory-Mapped I/O란?`</strong>`{=html}
```{=html}
</summary>
```
**정답:** CPU 주소 공간의 특정 주소 범위를 주변장치 레지스터에 대응시켜
메모리 주소 접근 방식으로 하드웨어를 제어하는 구조다.

```{=html}
</details>
```

------------------------------------------------------------------------

# 56. Bug Log

`99_Bug_Log/Memory_Bugs.md`에 다음 형태로 기록한다.

```` markdown
## 지역 변수 주소 반환

### 잘못 작성한 코드

```c
int *get_value(void)
{
    int value = 10;

    return &value;
}
```

### 문제

`value`는 자동 저장 기간을 가지며 함수 반환 시 수명이 끝난다.

### 결과

반환된 포인터는 더 이상 유효한 `value` 객체를 가리키지 않는다.

### 재발 방지 규칙

포인터를 저장하거나 반환할 때 반드시 대상 객체의 Lifetime을 확인한다.
````

추가 기록 항목:

``` text
Dangling Pointer
Use-After-Free
Memory Leak
Double Free
Stack Overflow
Large Local Buffer
잘못된 전역 상태 공유
```

------------------------------------------------------------------------

# 57. Memory 문제를 볼 때 사고 방식

``` mermaid
flowchart TD
    A["어떤 객체인가?"] --> B["어디에서 선언됐는가?"]
    B --> C["Storage Duration은?"]
    C --> D["Lifetime은 아직 유효한가?"]
    D --> E["누가 주소를 보관하는가?"]
    E --> F["동적 메모리라면 누가 free하는가?"]
    F --> G["범위는 안전한가?"]
```

항상 다음 질문을 한다.

``` text
이 객체는 지금 존재하는가?
이 포인터가 가리키는 대상은 살아 있는가?
누가 이 메모리를 관리하는가?
언제 해제되는가?
```

------------------------------------------------------------------------

# 58. 지금까지의 개념 연결

``` mermaid
flowchart LR
    A["Data Type"] --> B["Operator"]
    B --> C["Control Flow"]
    C --> D["Function"]
    D --> E["Array / String"]
    E --> F["Pointer"]
    F --> G["Memory Model"]
```

  기존 개념      Memory Model에서의 역할
  -------------- -------------------------
  Data Type      객체의 표현과 크기
  Function       지역 객체와 호출
  Array          연속된 저장 공간
  Pointer        주소와 객체 접근
  Memory Model   저장 위치, 기간, 수명

------------------------------------------------------------------------

# 59. C 학습으로의 연결

Memory Model 이후 C 학습은 다음과 같이 확장된다.

``` mermaid
flowchart TD
    A["Memory Model"] --> B["Struct"]
    A --> C["malloc / free"]
    A --> D["Bit Operation"]
    D --> E["volatile"]
    E --> F["Embedded C"]
```

특히 다음 주제에서 현재 내용이 직접 사용된다.

``` text
Struct Pointer
Dynamic Allocation
Linked Data Structure
Buffer
Register
Driver
```

------------------------------------------------------------------------

# 60. C++로의 연결

C++에서는 직접적인 메모리 관리 실수를 줄이기 위해 다음 개념을 학습하게
된다.

``` mermaid
flowchart LR
    A["C Memory Model"] --> B["C++ Object Lifetime"]
    B --> C["Constructor / Destructor"]
    C --> D["RAII"]
    D --> E["Smart Pointer"]
```

C에서 `malloc/free`, lifetime, dangling pointer를 이해하면 C++의 RAII가
해결하려는 문제가 무엇인지 더 명확해진다.

------------------------------------------------------------------------

# 61. 임베디드로의 연결

``` mermaid
flowchart TD
    A["Memory Model"] --> B["Flash"]
    A --> C["SRAM"]
    A --> D["Stack"]
    A --> E["Static Buffer"]
    A --> F["MMIO"]

    F --> G["Register"]
    G --> H["GPIO / UART / Timer"]

    D --> I["ISR / Task Stack"]
    E --> J["UART / Ring Buffer"]
```

임베디드에서 중요한 흐름:

``` text
Memory Model
 ↓
MCU Memory Map
 ↓
Flash / SRAM
 ↓
Stack / Static Data
 ↓
Pointer
 ↓
MMIO
 ↓
Register
 ↓
Peripheral
```

------------------------------------------------------------------------

# 62. 핵심 요약

``` mermaid
mindmap
  root((Memory Model))
    Concepts
      Scope
      Storage Duration
      Lifetime
      Address
    Automatic
      Local Variable
      Stack
      Stack Frame
    Static
      Global
      static
      data
      bss
    Dynamic
      malloc
      Heap
      free
    Bugs
      Dangling
      Leak
      Use After Free
      Double Free
      Stack Overflow
    Embedded
      Flash
      SRAM
      MMIO
      Startup
      Linker Script
```

## 최종적으로 기억할 여덟 문장

> **1. Scope는 이름을 사용할 수 있는 영역이고 Lifetime은 객체가 유효하게
> 존재하는 기간과 관련된다.**

> **2. 일반적인 구현에서 함수의 자동 지역 데이터는 Stack과 밀접하게
> 연결된다.**

> **3. 전역 변수와 지역 `static` 객체는 정적 저장 기간을 가지며 프로그램
> 실행 동안 저장 공간이 유지된다.**

> **4. 동적 메모리는 `malloc()` 등으로 확보하고 더 이상 필요하지 않으면
> `free()`해야 한다.**

> **5. 포인터가 `NULL`이 아니더라도 대상 객체의 수명이 끝났다면 안전하지
> 않다.**

> **6. Dangling Pointer, Memory Leak, Use-After-Free, Double Free는 모두
> 메모리 수명과 관리 문제다.**

> **7. `.text`, `.data`, `.bss`, `.rodata`는 흔한 Section 구조지만 실제
> 배치는 toolchain과 linker script에 의존한다.**

> **8. 임베디드에서는 Memory Model이
> `Flash/SRAM → Stack/Static Data → MMIO → Register → Hardware`로
> 연결된다.**

------------------------------------------------------------------------

# 63. 현재 학습 진행 상태

``` mermaid
flowchart LR
    A["Data_Type.md<br/>완료"] --> B["Operator.md<br/>완료"]
    B --> C["Control_Flow.md<br/>완료"]
    C --> D["Function.md<br/>완료"]
    D --> E["Array_String.md<br/>완료"]
    E --> F["Pointer.md<br/>완료"]
    F --> G["Memory_Model.md<br/>현재"]
    G --> H["Struct.md"]
```

``` text
C_Cpp_Study/
├── 00_Common/
│   ├── Data_Type.md       ✓
│   ├── Operator.md        ✓
│   ├── Control_Flow.md    ✓
│   ├── Function.md        ✓
│   └── Memory_Model.md    ← 현재
│
└── 01_C/
    ├── Array_String.md    ✓
    └── Pointer.md         ✓
```

------------------------------------------------------------------------

# 64. 다음 학습

Memory Model을 이해했다면 다음에는 서로 관련된 여러 데이터를 하나의
사용자 정의 타입으로 묶는 **구조체(Struct)**를 학습한다.

``` mermaid
flowchart LR
    A["Data Type"] --> B["Struct"]
    C["Pointer"] --> B
    D["Memory Model"] --> B
    B --> E["Struct Pointer"]
    E --> F["Driver Context"]
    E --> G["Linked Data Structure"]
```

다음 질문을 해결하게 된다.

``` text
서로 다른 타입의 데이터를 하나로 어떻게 묶는가?
구조체는 메모리에 어떻게 배치되는가?
구조체 포인터와 -> 연산자는 무엇인가?
padding과 alignment는 무엇인가?
임베디드에서 레지스터/드라이버 상태를 구조체로 어떻게 표현하는가?
```

**다음 문서:** `01_C/Struct.md`
