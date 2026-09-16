# malloc / free

> **학습 목표**
>
> C의 동적 메모리 관리에서 `malloc()`, `calloc()`, `realloc()`,
> `free()`의 역할을 이해하고 안전하게 사용할 수 있어야 한다.
>
> 핵심 흐름: **크기 계산 → 할당 → 실패 검사 → 초기화/사용 → 수명·소유권
> 관리 → 해제**
>
> 위치: `C_Cpp_Study/01_C/malloc_free.md`

------------------------------------------------------------------------

# 1. 왜 동적 메모리가 필요한가?

고정 배열은 크기를 미리 정한다.

``` c
int numbers[100];
```

하지만 실행 중 필요한 크기가 결정될 수도 있다.

``` text
사용자 입력 개수
수신 데이터 크기
자료구조 Node 개수
가변 길이 데이터
```

``` mermaid
flowchart LR
    A["Runtime Size"] --> B["Dynamic Allocation"]
    B --> C["Pointer"]
    C --> D["Use"]
    D --> E["free"]
```

**동적 메모리 할당은 실행 중 필요한 저장 공간을 요청하고 반환된 주소를
포인터로 관리하는 방식이다.**

------------------------------------------------------------------------

# 2. 관련 함수

``` c
#include <stdlib.h>
```

  함수          역할
  ------------- --------------------------------------------------
  `malloc()`    지정한 byte 수 할당
  `calloc()`    요소 개수 × 크기 할당 + 모든 byte를 0으로 초기화
  `realloc()`   기존 allocation 크기 변경
  `free()`      allocation 해제

``` mermaid
flowchart TD
    A["Dynamic Memory"] --> B["malloc"]
    A --> C["calloc"]
    A --> D["realloc"]
    A --> E["free"]
```

------------------------------------------------------------------------

# 3. `malloc()`

원형:

``` c
void *malloc(size_t size);
```

정수 하나:

``` c
int *ptr = malloc(sizeof(int));
```

또는 타입 중복을 줄여:

``` c
int *ptr = malloc(sizeof *ptr);
```

``` mermaid
flowchart LR
    A["sizeof *ptr"] --> B["malloc"]
    B --> C["Dynamic Storage"]
    C --> D["Address"]
    D --> E["ptr"]
```

성공하면 확보한 저장 공간의 주소를 반환하고, 실패하면 `NULL`을 반환한다.

------------------------------------------------------------------------

# 4. 반드시 실패를 검사한다

``` c
int *ptr = malloc(sizeof *ptr);

if (ptr == NULL)
{
    return 1;
}
```

이후에만 역참조한다.

``` c
*ptr = 100;
```

전체 예:

``` c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *ptr = malloc(sizeof *ptr);

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

------------------------------------------------------------------------

# 5. C에서 `malloc()` 캐스팅

C에서는 일반적으로:

``` c
int *ptr = malloc(sizeof *ptr);
```

처럼 작성한다.

`malloc()`의 `void *` 반환값은 C에서 객체 포인터 타입으로 변환될 수
있으므로 다음 캐스팅은 필요하지 않다.

``` c
int *ptr = (int *)malloc(sizeof *ptr);
```

> C++에서는 변환 규칙과 권장 자원 관리 방식이 다르다. C++에서는 이후
> RAII와 컨테이너를 학습한다.

------------------------------------------------------------------------

# 6. `malloc()`은 0으로 초기화하지 않는다

``` c
int *ptr = malloc(sizeof *ptr);
```

할당 직후 `*ptr`이 `0`이라고 가정하면 안 된다.

값을 읽기 전에 초기화한다.

``` c
*ptr = 0;
```

배열도 마찬가지다.

``` c
int *numbers = malloc(sizeof *numbers * 10);
```

각 요소를 사용 전에 필요한 값으로 초기화한다.

------------------------------------------------------------------------

# 7. 동적 배열

``` c
size_t count = 10;

int *numbers = malloc(sizeof *numbers * count);

if (numbers == NULL)
{
    return 1;
}
```

사용:

``` c
for (size_t i = 0; i < count; i++)
{
    numbers[i] = (int)i;
}
```

해제:

``` c
free(numbers);
```

``` mermaid
flowchart LR
    A["malloc"] --> B["연속된 int 저장 공간"]
    C["numbers"] --> B
    B --> D["numbers[0]"]
    B --> E["numbers[1]"]
    B --> F["..."]
```

------------------------------------------------------------------------

# 8. 요소 개수와 byte 수

정수 `count`개:

``` c
malloc(sizeof *numbers * count);
```

구조체 `count`개:

``` c
malloc(sizeof *points * count);
```

항상 구분한다.

``` text
Element Count
×
Element Size
=
Required Bytes
```

> \[!IMPORTANT\] `malloc(count)`는 `count`개의 정수를 할당한다는 의미가
> 아니다. `count` **byte**를 요청한다.

------------------------------------------------------------------------

# 9. 크기 계산 Overflow

다음 계산도 정수 연산이다.

``` c
sizeof *numbers * count
```

매우 큰 `count`에서는 `size_t` 범위를 넘는 곱셈을 고려해야 한다.

``` mermaid
flowchart LR
    A["Element Size"] --> C["Multiply"]
    B["Count"] --> C
    C --> D{"Representable?"}
    D -->|"Yes"| E["Allocate"]
    D -->|"No"| F["Error"]
```

초기 단계에서는 우선 **몇 byte를 요청하는지 정확하게 계산하는 습관**을
만든다.

------------------------------------------------------------------------

# 10. `calloc()`

원형:

``` c
void *calloc(size_t count, size_t size);
```

예:

``` c
int *numbers = calloc(10, sizeof *numbers);
```

의미:

``` text
10 elements
×
sizeof(int)
```

`calloc()`은 할당한 모든 byte를 0으로 초기화한다.

------------------------------------------------------------------------

# 11. `malloc()` vs `calloc()`

  항목        `malloc()`        `calloc()`
  ----------- ----------------- ----------------------
  인자        전체 byte 수      요소 개수, 요소 크기
  초기 상태   초기화되지 않음   모든 byte가 0
  실패        `NULL`            `NULL`
  해제        `free()`          `free()`

> \[!NOTE\] "모든 byte가 0"과 "모든 C 타입에서 의미상 원하는 초기값"을
> 무조건 동일시하지 않는다. 현재는 정수 배열 등의 기본 사례에 집중한다.

------------------------------------------------------------------------

# 12. `calloc()` 예제

``` c
size_t count = 5;

int *numbers = calloc(count, sizeof *numbers);

if (numbers == NULL)
{
    return 1;
}

for (size_t i = 0; i < count; i++)
{
    printf("%d\n", numbers[i]);
}

free(numbers);
```

------------------------------------------------------------------------

# 13. `free()`

원형:

``` c
void free(void *ptr);
```

더 이상 필요하지 않은 동적 allocation을 해제한다.

``` c
free(ptr);
```

``` mermaid
flowchart LR
    A["Allocated"] --> B["Use"]
    B --> C["free"]
    C --> D["Lifetime End"]
```

`free()` 이후 해당 allocation을 다시 역참조하면 안 된다.

------------------------------------------------------------------------

# 14. `free(NULL)`

다음은 허용된다.

``` c
free(NULL);
```

따라서 `ptr`이 `NULL` 또는 적절한 allocation을 가리키는 상태라면 단순히:

``` c
free(ptr);
```

라고 호출할 수 있다.

------------------------------------------------------------------------

# 15. Use-After-Free

잘못된 코드:

``` c
int *ptr = malloc(sizeof *ptr);

if (ptr != NULL)
{
    *ptr = 10;

    free(ptr);

    printf("%d\n", *ptr);
}
```

``` mermaid
flowchart LR
    A["malloc"] --> B["Valid"]
    B --> C["free"]
    C --> D["Lifetime End"]
    D --> E["*ptr"]
    E --> F["Use-After-Free"]
```

`free()` 후 해당 allocation의 수명은 끝났다.

------------------------------------------------------------------------

# 16. `free(ptr); ptr = NULL;`

다음 패턴을 자주 사용한다.

``` c
free(ptr);
ptr = NULL;
```

이는 `ptr`이 이전 해제 주소를 계속 보유하지 않게 할 수 있다.

하지만 다른 포인터가 같은 allocation을 가리키고 있다면 자동으로 `NULL`이
되지 않는다.

``` c
int *a = malloc(sizeof *a);
int *b = a;

free(a);
a = NULL;
```

이후 `b`는 해제된 allocation의 이전 주소를 보유하는 dangling pointer가
될 수 있다.

------------------------------------------------------------------------

# 17. Aliasing

여러 포인터가 같은 객체를 가리킬 수 있다.

``` mermaid
flowchart TD
    A["Allocation"] --> B["a"]
    A --> C["b"]
    A --> D["c"]
```

`free(a)`가 수행되면 allocation 자체의 수명이 종료된다.

``` mermaid
flowchart TD
    A["free(a)"] --> B["Allocation Lifetime End"]
    B --> C["b dangling"]
    B --> D["c dangling"]
```

------------------------------------------------------------------------

# 18. Double Free

잘못된 코드:

``` c
free(ptr);
free(ptr);
```

같은 allocation을 두 번 해제하려는 동작은 잘못된 메모리 관리다.

``` mermaid
flowchart LR
    A["Allocated"] --> B["free"]
    B --> C["Released"]
    C --> D["free again"]
    D --> E["Double Free"]
```

------------------------------------------------------------------------

# 19. Memory Leak

``` c
int *ptr = malloc(sizeof *ptr);

if (ptr == NULL)
{
    return;
}

ptr = NULL;
```

기존 allocation의 주소를 잃었다.

``` mermaid
flowchart LR
    A["malloc"] --> B["Allocation"]
    B --> C["ptr"]
    C --> D["ptr = NULL"]
    B --> E["접근 경로 상실"]
    E --> F["Memory Leak"]
```

장시간 실행되는 프로그램이나 펌웨어에서는 반복적인 누수가 특히 문제가
된다.

------------------------------------------------------------------------

# 20. 포인터 덮어쓰기 Leak

``` c
int *ptr = malloc(100);

ptr = malloc(200);
```

첫 번째 allocation을 해제하지 않은 채 `ptr`을 덮어쓰면 첫 번째 주소를
잃을 수 있다.

``` text
malloc(100)
 ↓
ptr

malloc(200)
 ↓
ptr 덮어쓰기
 ↓
첫 allocation 접근 경로 상실
```

------------------------------------------------------------------------

# 21. `realloc()`

원형:

``` c
void *realloc(void *ptr, size_t new_size);
```

기존 allocation의 크기를 변경할 때 사용한다.

예:

``` c
int *numbers = malloc(sizeof *numbers * 5);
```

5개에서 10개로 늘리고 싶다면 `realloc()`을 사용할 수 있다.

------------------------------------------------------------------------

# 22. `realloc()`의 핵심 규칙

`realloc()`은 성공 시 기존 주소와 다른 주소를 반환할 수 있다.

또한 실패하면 `NULL`을 반환하며 기존 allocation은 그대로 유지된다.

따라서 다음 코드는 위험하다.

``` c
numbers = realloc(numbers, sizeof *numbers * 10);
```

실패하면 기존 주소를 잃을 수 있기 때문이다.

------------------------------------------------------------------------

# 23. 안전한 `realloc()` 기본 패턴

``` c
int *temp = realloc(numbers,
                    sizeof *numbers * new_count);

if (temp == NULL)
{
    /* 기존 numbers는 여전히 유효 */
}
else
{
    numbers = temp;
}
```

``` mermaid
flowchart TD
    A["Existing Allocation"] --> B["realloc"]
    B --> C{"Success?"}
    C -->|"No"| D["NULL"]
    D --> E["기존 allocation 유지"]
    C -->|"Yes"| F["새 주소 반환 가능"]
    F --> G["numbers = temp"]
```

------------------------------------------------------------------------

# 24. `realloc()` 후 기존 포인터 주의

성공한 `realloc()`이 allocation을 다른 위치로 옮겼다면 이전 allocation
내부를 가리키던 별도 포인터는 더 이상 유효하지 않을 수 있다.

``` text
numbers
 ↓
old allocation

element_ptr = &numbers[2]

realloc 성공 + 이동
 ↓
new allocation

element_ptr
 ↓
이전 위치를 계속 가리킬 수 있음
```

따라서 내부 요소 주소를 별도로 보관하는 코드에서는 특히 주의한다.

------------------------------------------------------------------------

# 25. 확장된 영역은 자동 0 초기화가 아니다

5개에서 10개로 확장했다고 가정한다.

``` text
기존 영역 : 5 elements
추가 영역 : 5 elements
```

새로 추가된 영역이 자동으로 0이라고 가정하면 안 된다.

필요한 값을 직접 초기화한다.

------------------------------------------------------------------------

# 26. 구조체 동적 할당

``` c
typedef struct
{
    int x;
    int y;
} Point;
```

할당:

``` c
Point *point = malloc(sizeof *point);

if (point == NULL)
{
    return 1;
}

point->x = 10;
point->y = 20;

free(point);
```

``` mermaid
flowchart LR
    A["malloc"] --> B["Point Object"]
    C["Point *point"] --> B
    C --> D["point->x"]
    C --> E["point->y"]
```

------------------------------------------------------------------------

# 27. 구조체 배열 동적 할당

``` c
size_t count = 10;

Point *points = malloc(sizeof *points * count);

if (points == NULL)
{
    /* failure */
}
```

접근:

``` c
points[0].x = 10;
points[0].y = 20;
```

해제:

``` c
free(points);
```

------------------------------------------------------------------------

# 28. Linked List와 동적 메모리

``` c
typedef struct Node
{
    int value;
    struct Node *next;
} Node;
```

Node 하나:

``` c
Node *node = malloc(sizeof *node);

if (node == NULL)
{
    /* failure */
}

node->value = 10;
node->next = NULL;
```

``` mermaid
flowchart LR
    A["malloc"] --> B["Node"]
    B --> C["value"]
    B --> D["next"]
```

------------------------------------------------------------------------

# 29. Linked List 해제 순서

``` mermaid
flowchart LR
    A["Node 1"] --> B["Node 2"]
    B --> C["Node 3"]
    C --> D["NULL"]
```

안전한 기본 패턴:

``` c
Node *current = head;

while (current != NULL)
{
    Node *next = current->next;

    free(current);

    current = next;
}
```

순서:

``` text
1. next 주소 저장
2. current 해제
3. 저장한 next로 이동
```

------------------------------------------------------------------------

# 30. 잘못된 Linked List 해제

``` c
free(current);
current = current->next;
```

`free(current)` 후 `current->next`를 읽으려 한다.

즉, Use-After-Free다.

``` mermaid
flowchart LR
    A["current"] --> B["next 먼저 저장"]
    B --> C["free(current)"]
    C --> D["current = next"]
```

------------------------------------------------------------------------

# 31. Ownership

C가 allocation의 소유자를 자동으로 추적해 주지는 않는다.

따라서 API 설계에서 다음을 정한다.

``` text
누가 malloc하는가?
누가 free하는가?
함수는 포인터를 빌려 쓰는가?
소유권을 전달받는가?
반환된 포인터는 누가 해제하는가?
```

``` mermaid
flowchart TD
    A["Allocation"] --> B["Owner"]
    B --> C["Use"]
    C --> D["Ownership Transfer?"]
    D --> E["Final Owner"]
    E --> F["free"]
```

------------------------------------------------------------------------

# 32. API와 소유권

예:

``` c
char *create_buffer(size_t size);
```

이 함수가 allocation을 반환한다면 API 문서에 다음과 같은 계약이
필요하다.

``` text
반환된 메모리는 호출자가 free한다.
```

반대로:

``` c
void process_buffer(const unsigned char *buffer,
                    size_t size);
```

처럼 외부 버퍼를 잠시 사용하는 함수라면 함수가 임의로 `free(buffer)`하지
않는다는 계약을 둘 수 있다.

------------------------------------------------------------------------

# 33. 여러 자원과 실패 처리

``` c
int *a = malloc(sizeof *a);

if (a == NULL)
{
    return 1;
}

int *b = malloc(sizeof *b);

if (b == NULL)
{
    free(a);
    return 1;
}
```

두 번째 할당이 실패하면 이미 확보한 첫 번째 자원을 정리해야 한다.

``` mermaid
flowchart TD
    A["malloc a"] --> B{"Success?"}
    B -->|"No"| C["Error"]
    B -->|"Yes"| D["malloc b"]
    D --> E{"Success?"}
    E -->|"No"| F["free(a)"]
    F --> C
    E -->|"Yes"| G["Use"]
```

------------------------------------------------------------------------

# 34. Cleanup Pattern

자원이 여러 개일 때 정리 경로를 한곳에 모으는 C 코드도 볼 수 있다.

``` c
int function(void)
{
    int result = -1;
    int *a = NULL;
    int *b = NULL;

    a = malloc(sizeof *a);

    if (a == NULL)
    {
        goto cleanup;
    }

    b = malloc(sizeof *b);

    if (b == NULL)
    {
        goto cleanup;
    }

    result = 0;

cleanup:
    free(b);
    free(a);

    return result;
}
```

> \[!NOTE\] 모든 `goto`를 권장한다는 뜻이 아니다. 저수준 C 코드에서는
> 여러 자원을 하나의 cleanup 경로에서 정리하기 위해 제한적으로 사용하는
> 패턴을 볼 수 있다.

------------------------------------------------------------------------

# 35. 임베디드에서 동적 메모리

임베디드에서도 Heap을 사용할 수 있지만 프로젝트에 따라 제한하거나 금지할
수 있다.

주요 고려사항:

``` text
제한된 RAM
할당 실패
Fragmentation
실행 시간 예측성
장시간 안정성
안전 규칙
```

``` mermaid
flowchart TD
    A["Embedded Heap"] --> B["RAM"]
    A --> C["Fragmentation"]
    A --> D["Failure"]
    A --> E["Timing"]
```

------------------------------------------------------------------------

# 36. Memory Fragmentation

할당과 해제를 반복하면 Free 공간이 여러 조각으로 나뉠 수 있다.

``` text
초기:
[              Free              ]

반복 후:
[Used][Free][Used][Free][Used][Free]
```

전체 Free 크기는 남아 있어도 큰 연속 블록을 얻기 어려울 수 있다.

``` mermaid
flowchart LR
    A["Repeated Allocation"] --> B["Free Blocks 분산"]
    B --> C["Fragmentation"]
    C --> D["Large Allocation 어려움"]
```

------------------------------------------------------------------------

# 37. 임베디드에서의 대안

``` text
Static Array
Fixed-size Buffer
Memory Pool
Object Pool
Ring Buffer
Compile-time Capacity
```

``` mermaid
flowchart TD
    A["Memory Strategy"] --> B["General Heap"]
    A --> C["Static Buffer"]
    A --> D["Memory Pool"]
    A --> E["Ring Buffer"]
```

시스템의 메모리 크기, 실시간성, 실패 허용 여부에 따라 선택한다.

------------------------------------------------------------------------

# 38. UART Buffer 예

``` c
#define UART_RX_BUFFER_SIZE 128

static unsigned char rx_buffer[UART_RX_BUFFER_SIZE];
```

``` mermaid
flowchart LR
    A["UART RX"] --> B["Static Buffer"]
    B --> C["Parser"]
```

고정 크기 메모리를 사용하면 필요한 RAM을 미리 파악하기 쉽다는 장점이
있다.

------------------------------------------------------------------------

# 39. ISR과 동적 할당

ISR에서는 일반적으로 짧고 예측 가능한 처리를 목표로 한다.

동적 할당은 다음 요소를 고려해야 한다.

``` text
allocator 실행 시간
동시 접근
fragmentation
실시간 요구사항
```

따라서 ISR 내부에서 일반 Heap allocation을 피하는 설계가 흔하다.

실제 정책은 RTOS, allocator, 프로젝트 요구사항에 따라 결정한다.

------------------------------------------------------------------------

# 40. RTOS와 메모리

RTOS에서는 객체를 동적으로 또는 정적으로 생성하는 API가 제공될 수 있다.

예를 들어 FreeRTOS에서는 이후 다음 계열의 차이를 접하게 된다.

``` text
Dynamic Creation
Static Creation
```

``` mermaid
flowchart TD
    A["RTOS Object"] --> B["Dynamic"]
    A --> C["Static"]
```

핵심은 **메모리 할당 정책도 시스템 설계의 일부**라는 점이다.

------------------------------------------------------------------------

# 41. C++ RAII와 연결

C:

``` c
int *ptr = malloc(sizeof *ptr);

/* use */

free(ptr);
```

C++에서는 객체 수명과 자원 해제를 연결하는 RAII를 학습한다.

``` mermaid
flowchart LR
    A["C Manual Management"] --> B["Leak / Lifetime Risk"]
    B --> C["C++ RAII"]
    C --> D["Container / Smart Pointer"]
```

C의 수동 메모리 관리를 이해하면 RAII의 목적을 이해하기 쉬워진다.

------------------------------------------------------------------------

# 42. 대표적인 메모리 버그

``` mermaid
mindmap
  root((Dynamic Memory Bugs))
    Allocation
      NULL Check Missing
      Wrong Size
      Size Overflow
    Lifetime
      Use After Free
      Dangling Pointer
    Release
      Memory Leak
      Double Free
    Resize
      realloc Pointer Loss
      Stale Interior Pointer
```

------------------------------------------------------------------------

# 43. 디버깅 체크리스트

-   [ ] 필요한 요소 개수는 몇 개인가?
-   [ ] 필요한 byte 수는 몇 개인가?
-   [ ] `sizeof` 대상은 올바른가?
-   [ ] 크기 곱셈 overflow 가능성이 있는가?
-   [ ] 반환값을 `NULL` 검사했는가?
-   [ ] 읽기 전에 초기화했는가?
-   [ ] 배열 범위를 지키는가?
-   [ ] 누가 allocation을 소유하는가?
-   [ ] 누가 `free()`하는가?
-   [ ] `free()` 후 접근하지 않는가?
-   [ ] Double Free 가능성이 없는가?
-   [ ] alias pointer가 dangling이 되지 않는가?
-   [ ] `realloc()` 결과를 안전하게 처리했는가?

------------------------------------------------------------------------

# 44. AddressSanitizer

PC 학습 환경에서는 Sanitizer가 메모리 버그 탐지에 도움을 줄 수 있다.

``` bash
gcc -std=c17 -Wall -Wextra -Wpedantic -g \
    -fsanitize=address,undefined \
    main.c -o main
```

실행:

``` bash
./main
```

환경이 지원하는 경우 다음 문제 탐지에 유용하다.

``` text
Use-After-Free
Heap Buffer Overflow
Stack Buffer Overflow
일부 Undefined Behavior
```

> 도구 사용 전에 Lifetime과 Ownership을 먼저 추적하는 습관을 만든다.

------------------------------------------------------------------------

# 45. 실습 1 --- 정수 하나

다음 순서로 구현한다.

``` text
int 하나 malloc
→ NULL 검사
→ 100 저장
→ 출력
→ free
```

------------------------------------------------------------------------

# 46. 실습 2 --- 동적 배열

사용자로부터 `count`를 입력받는다.

``` text
count 입력
→ int count개 할당
→ 값 저장
→ 전체 출력
→ free
```

------------------------------------------------------------------------

# 47. 실습 3 --- `calloc()`

정수 10개를 `calloc()`으로 할당한다.

1.  초기 요소 출력
2.  값을 변경
3.  다시 출력
4.  `free()`

------------------------------------------------------------------------

# 48. 실습 4 --- 구조체

``` c
typedef struct
{
    int id;
    float value;
} Sensor;
```

`Sensor` 하나를 동적으로 할당해:

``` text
id = 1
value = 25.5
```

를 저장하고 출력한 뒤 해제한다.

------------------------------------------------------------------------

# 49. 실습 5 --- 구조체 배열

`Sensor` 5개를 동적으로 만든다.

``` c
Sensor *sensors = malloc(...);
```

각 요소를 초기화하고 반복문으로 출력한 뒤 해제한다.

------------------------------------------------------------------------

# 50. 실습 6 --- `realloc()`

``` text
int 5개 할당
→ 값 저장
→ 10개로 realloc
→ 추가 영역 초기화
→ 전체 출력
→ free
```

반드시 임시 포인터를 사용한다.

------------------------------------------------------------------------

# 51. 실습 7 --- Leak 찾기

``` c
void function(void)
{
    int *ptr = malloc(sizeof *ptr);

    if (ptr == NULL)
    {
        return;
    }

    *ptr = 10;
}
```

누락된 작업을 찾는다.

------------------------------------------------------------------------

# 52. 실습 8 --- Use-After-Free

``` c
int *ptr = malloc(sizeof *ptr);

if (ptr != NULL)
{
    *ptr = 10;
    free(ptr);

    printf("%d\n", *ptr);
}
```

allocation의 Lifetime이 끝나는 줄을 표시한다.

------------------------------------------------------------------------

# 53. 실습 9 --- Linked List

``` c
typedef struct Node
{
    int value;
    struct Node *next;
} Node;
```

Node 3개를 동적으로 만들어:

``` text
10 → 20 → 30 → NULL
```

로 연결하고 모든 Node를 안전하게 해제한다.

------------------------------------------------------------------------

# 54. 코드 추적

``` c
int *a = malloc(sizeof *a);

if (a != NULL)
{
    *a = 10;

    int *b = a;

    free(a);
    a = NULL;
}
```

  단계            `a`   `b`   allocation
  --------------- ----- ----- ------------
  `malloc` 성공         \-    
  `b = a`                     
  `free(a)`                   
  `a = NULL`                  

질문:

> 마지막 시점에 `b`를 역참조할 수 있는가?

------------------------------------------------------------------------

# 55. 반드시 설명할 수 있어야 하는 것

-   [ ] 동적 메모리가 필요한 이유
-   [ ] `malloc()`의 역할
-   [ ] `NULL` 검사
-   [ ] `malloc()`의 초기화 특성
-   [ ] `sizeof *ptr` 패턴
-   [ ] 동적 배열
-   [ ] 요소 수와 byte 수의 차이
-   [ ] `calloc()`의 역할
-   [ ] `malloc()`과 `calloc()`의 차이
-   [ ] `free()`의 역할
-   [ ] `free(NULL)`
-   [ ] Dangling Pointer
-   [ ] Use-After-Free
-   [ ] Double Free
-   [ ] Memory Leak
-   [ ] `realloc()`의 기본 규칙
-   [ ] 임시 포인터를 사용하는 이유
-   [ ] 구조체 동적 할당
-   [ ] Linked List 해제 순서
-   [ ] Ownership
-   [ ] API의 해제 책임
-   [ ] Fragmentation
-   [ ] 임베디드에서 Heap을 제한하는 이유
-   [ ] Static Buffer/Memory Pool이라는 대안
-   [ ] C++ RAII와의 연결

------------------------------------------------------------------------

# 56. 복습 문제

```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q1. malloc()은 무엇을 반환하는가?`</strong>`{=html}
```{=html}
</summary>
```
성공 시 동적 저장 공간을 가리키는 포인터를 반환하고 실패 시 `NULL`을
반환한다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q2. malloc()으로 얻은 메모리는 자동으로
0인가?`</strong>`{=html}
```{=html}
</summary>
```
아니다. 사용 전에 필요한 값을 초기화한다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q3. calloc()의 특징은?`</strong>`{=html}
```{=html}
</summary>
```
요소 개수와 요소 크기를 받아 공간을 할당하고 모든 byte를 0으로
초기화한다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q4. free(ptr) 후 \*ptr을 사용할 수
있는가?`</strong>`{=html}
```{=html}
</summary>
```
안 된다. 해당 allocation의 수명이 종료되었다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q5. free(NULL)은 허용되는가?`</strong>`{=html}
```{=html}
</summary>
```
허용된다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q6. Memory Leak이란?`</strong>`{=html}
```{=html}
</summary>
```
필요 없어진 allocation을 해제하지 않거나 주소를 잃어 더 이상 해제할 수
없게 되는 메모리 관리 문제다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q7. realloc 결과를 기존 포인터에 바로 저장하면 왜
위험한가?`</strong>`{=html}
```{=html}
</summary>
```
실패 시 `NULL`이 반환되지만 기존 allocation은 유지되므로, 기존 포인터를
덮어쓰면 그 주소를 잃을 수 있기 때문이다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q8. ptr = NULL로 만들면 같은 allocation을 가리키던 다른
포인터도 NULL이 되는가?`</strong>`{=html}
```{=html}
</summary>
```
아니다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q9. 임베디드에서 Heap 사용을 제한하는
이유는?`</strong>`{=html}
```{=html}
</summary>
```
RAM 제한, fragmentation, 실패 가능성, 실행 시간 예측성, 장시간 안정성
등을 고려하기 때문이다.

```{=html}
</details>
```
```{=html}
<details>
```
```{=html}
<summary>
```
`<strong>`{=html}Q10. C++ RAII와 어떤 관계가 있는가?`</strong>`{=html}
```{=html}
</summary>
```
C에서 수동 자원 관리로 발생할 수 있는 해제 누락과 Lifetime 문제를
이해하면, 자원 수명을 객체 수명과 연결하는 RAII의 목적을 이해하기
쉬워진다.

```{=html}
</details>
```

------------------------------------------------------------------------

# 57. Bug Log

`99_Bug_Log/Memory_Bugs.md`에 기록한다.

```` markdown
## realloc 결과 직접 덮어쓰기

### 잘못된 코드

```c
ptr = realloc(ptr, new_size);
```

### 문제

realloc 실패 시 NULL이 반환되지만 기존 allocation은 유지된다.
기존 포인터를 바로 덮어쓰면 기존 주소를 잃을 수 있다.

### 수정

```c
void *temp = realloc(ptr, new_size);

if (temp != NULL)
{
    ptr = temp;
}
```

### 재발 방지

realloc 결과는 임시 포인터로 받은 뒤 성공 여부를 확인한다.
````

추천 항목:

``` text
NULL Check 누락
Wrong Allocation Size
Memory Leak
Use-After-Free
Double Free
Dangling Alias
realloc Pointer Loss
Linked List Free Order
```

------------------------------------------------------------------------

# 58. 메모리 관리 사고 흐름

``` mermaid
flowchart TD
    A["무엇을 저장하는가?"] --> B["몇 개인가?"]
    B --> C["몇 byte인가?"]
    C --> D["Allocate"]
    D --> E["NULL Check"]
    E --> F["Initialize"]
    F --> G["Use"]
    G --> H["Owner 확인"]
    H --> I["Lifetime End"]
    I --> J["free"]
```

항상 다음 질문을 반복한다.

``` text
누가 만들었는가?
누가 소유하는가?
언제까지 유효한가?
누가 해제하는가?
```

------------------------------------------------------------------------

# 59. 지금까지의 연결

``` mermaid
flowchart LR
    A["Pointer"] --> D["Dynamic Memory"]
    B["Memory Model"] --> D
    C["Struct"] --> D
    D --> E["malloc / calloc"]
    E --> F["realloc"]
    F --> G["free"]
```

  기존 개념      역할
  -------------- -----------------
  Data Type      요소 크기
  Array          동적 배열
  Pointer        allocation 주소
  Memory Model   Lifetime
  Struct         동적 객체/Node
  Function       Ownership/API

------------------------------------------------------------------------

# 60. 코딩 테스트와 연결

C++ 코딩 테스트에서는 직접 `malloc/free`보다 `std::vector`,
`std::string` 등을 주로 사용한다.

``` mermaid
flowchart LR
    A["C Dynamic Array"] --> B["malloc / realloc / free"]
    C["C++"] --> D["std::vector"]
```

C 동적 배열을 이해하면 `vector`의 size/capacity와 재할당 개념을 이해하는
기반이 된다.

------------------------------------------------------------------------

# 61. 임베디드와 연결

``` mermaid
flowchart TD
    A["Memory Management"] --> B["Heap"]
    A --> C["Static Buffer"]
    A --> D["Memory Pool"]
    A --> E["RTOS Allocation"]

    B --> F["Fragmentation"]
    C --> G["Predictable Memory"]
    D --> H["Fixed-size Objects"]
```

펌웨어에서는 다음을 함께 판단한다.

``` text
RAM은 충분한가?
실행 시간은 예측 가능한가?
Fragmentation을 허용할 수 있는가?
할당 실패 시 복구 가능한가?
정적 할당으로 대체할 수 있는가?
```

------------------------------------------------------------------------

# 62. 핵심 요약

``` mermaid
mindmap
  root((Dynamic Memory))
    malloc
      Size
      NULL Check
      Uninitialized
    calloc
      Count
      Size
      Zero Bytes
    realloc
      Resize
      Temp Pointer
      Address Change
    free
      Lifetime End
    Bugs
      Leak
      Use After Free
      Double Free
      Dangling
    Design
      Ownership
      Cleanup
    Embedded
      Fragmentation
      Static Buffer
      Memory Pool
```

## 최종적으로 기억할 아홉 문장

> **1. `malloc()`은 필요한 byte 수의 동적 저장 공간을 요청한다.**

> **2. 동적 할당은 실패할 수 있으므로 `NULL`을 처리한다.**

> **3. `malloc()`은 저장 공간을 0으로 초기화하지 않는다.**

> **4. `calloc()`은 개수와 크기를 받아 공간을 확보하고 모든 byte를 0으로
> 초기화한다.**

> **5. `free()` 이후 해당 allocation의 수명은 끝난다.**

> **6. Memory Leak, Use-After-Free, Double Free는 대표적인 동적 메모리
> 버그다.**

> **7. `realloc()`은 주소가 바뀔 수 있고 실패 시 기존 allocation이
> 유지되므로 임시 포인터 패턴이 중요하다.**

> **8. C에서는 누가 메모리를 소유하고 해제하는지 명확히 정해야 한다.**

> **9. 임베디드에서는 Heap뿐 아니라 RAM, Fragmentation, 실패 가능성,
> 실행 시간 예측성을 함께 고려한다.**

------------------------------------------------------------------------

# 63. 현재 학습 진행 상태

``` mermaid
flowchart LR
    A["Memory_Model.md<br/>완료"] --> B["Struct.md<br/>완료"]
    B --> C["malloc_free.md<br/>현재"]
    C --> D["Bit_Operation.md"]
```

``` text
C_Cpp_Study/
└── 01_C/
    ├── Array_String.md    ✓
    ├── Pointer.md         ✓
    ├── Struct.md          ✓
    ├── malloc_free.md     ← 현재
    ├── Bit_Operation.md
    ├── volatile.md
    └── Embedded_C.md
```

------------------------------------------------------------------------

# 64. 다음 학습

다음 단계는 **Bit Operation**이다.

``` mermaid
flowchart LR
    A["Integer"] --> B["Binary"]
    B --> C["& | ^ ~ << >>"]
    C --> D["Bit Mask"]
    D --> E["Set / Clear / Toggle / Test"]
    E --> F["Register"]
    F --> G["Hardware"]
```

다음 질문을 해결한다.

``` text
비트 연산자는 무엇인가?
특정 bit를 어떻게 Set/Clear/Toggle하는가?
특정 bit 상태를 어떻게 Test하는가?
Mask란 무엇인가?
Shift 연산은 어떻게 사용하는가?
Register 제어와 어떻게 연결되는가?
```

**다음 문서:** `01_C/Bit_Operation.md`
