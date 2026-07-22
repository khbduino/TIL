# 모듈과 패키지

## 모듈

### 모듈의 정의 & 비유

```markdown
**한 파일로 묶인 변수와 함수의 모음**
특정한 기능을 하는 코드가 작성된 파이썬 파일(.py)
```

#### 모듈 예시

- math 내장 모듈
  - 파이썬이 미리 작성해 둔 수학 관련 변수와 함수가 작성된 모듈

```python
import math
print(math.pi) # 3.141592
print(math.sqrt(4)) # 2.0
```

### 모듈 활용

#### import문 사용

- 같은 이름의 함수가 여러 모듈에 있을 때 충돌을 방지할 수 있음
- `.(dot)`연산자
  - 점의 왼쪽 객체에서 점의 오른쪽 이름을 찾아라 라는 이름

```python
import math
print(math.pi) # 모듈명.변수명
print(math.sqrt(4)) # 모듈명.변수명
```

- 단점
  - 코드가 길어질 수 있음

#### from 절 사용

- 필요한 것만 콕 집어서 가져오는 방식

```python
from math import pi, sqrt
print(pi) # 변수명
print(sqrt(4))  # 함수명
```

- 실무에서 form절이 표준인 대표 사례

```python
from datetime import datetime
from collections import Count
from django.shrotcuts import render, redirect
```

| 방식 | 이럴 때 적합 |
| :---: | :---: |
| import 모듈 | 모듈 이름이 짧고, 어디서 온 함수인지 드러내고 싶을 때 |
| from 모듈 import 대상 | 특정 함수/변수를 자주 반복 호출할 때, 모듈 경로가 같을 때 |

> **TIP**
> 둘 중 무엇이 옳고 그른 것이 아닌 "이 이름의 출처를 코드에 남길것인가"를 선택하는 문제
> math.sqrt(4)는 출처가 보이고, sqrt(4)는 간결
> 팀 컨벤션에 따르되, 한 파일 안에서는 방식을 통일하는 것이 좋다

#### import 시 이름 충돌 다루기

- 같은 이름을 두 번 가져오면 나중 것이 이긴다

```python
from math import sqrt
from my_math import sqrt # 나중에 작성된 코드가 덮어씀

result = sqrt(9) # my_math의 sqrt가 호출됨
```

#### as 키워드

- as 키워드를 사용하여 별칭(alias)을 부여
  - 두 개 이상의 모듈에서 동일한 이름의 변수, 함수 클래스 등을 가져올 때 발생하는 이름 충돌 해결

```python
from math import sqrt
from my_math import sqrt as my_sqrt # 별칭으로 같이 공존이 가능

sqrt(4)
my_sqrt(4)
```

#### import * 는 별개의 문제이다

- 이름 충돌은 as로 해결, *는 무엇을 가져왔는지 몰라 충돌이 일어날 가능성이 높으며 원인을 찾지 못함
- 에디터의 자동완성과 코드 검사 도구도 무력해짐

```python
from math import *

e = 300
```

### 사용자 정의 모듈

#### 직접 정의한 모듈 사용하기

- my_math.py를 생성하여 add 함수를 작성

```python
# my_math.py
def add(x, y):
    return x + y
```

- 같은 위치에 sample.py를 생성후 my_math 모둘의 add 함수를 호출

```python
# sample.py
import my_math
print(my_math.add(1, 2)) # 3
```

## 패키지

### 패키지의 정의 & 비유

```markdown
연관된 모듈들을 하나의 디렉토리에 모아 놓은 것
```

> 유용한 도구(모듈)을 모아서 하나로 묶은 것이 패키지
> 패키지는 누군가 잘 만들어둔 **코드 꾸러미**
> 패키지 안에 패키지는 **서브패키지**

### 직접 패키지 만들어 보기

### 직접 만든 패키지 사용하기

### 패키지 사용 목적

모듈들의 이름공간을 구분하여 충돌을 방지
모듈들을 효율적으로 관리하고 할 수 있도록 돕는 역할

## 라이브러리

라이브러리 > 패키지 > 모듈

### 파이썬 표준 라이브러리

Python Standard Library

파이썬 언어와 함께 제공되는 다양한 모듈과 패키지의 모음

### pip

#### 패키지 설치

#### requests 외부 패키지 설치 및 사용 예시

- 실습 파일로 진행

## 제어문

코드의 실행 흐름을 제어하는 데 사용되는 구문
**조건**에 따라 코드 블록을 실행하거나 **반복**적으로 코드를 실행

> 제어문은 **상황에 따라 다른 코드를 실행**하거나 **같은 코드를 여러번 반복**할 수 있게 합니다.

## 조건문

주어진 조건식을 평가하여 해당 조건이 참(True)인 경우에만 코드 블록을 실행하거나 건너뜀

### 조건문의 종류

- if
- elif
- else

### 'if' statement

- if문
  - 조건문의 기본 형태
  - if문에 작성된 조건을 만족할 때 내부 코드를 실행
  - 작성되는 조건은 표현식으로 작성
  
- elif 문
  - 이전의 조건을 만족하지 못하고 추가로 다른 조건이 필요할 때 사용
  - 여러 개의 elif 문을 사용할 수 있음

- else 문
  - 모든 조건들을 만족하지 않으면 실행됨

#### 조건문의 네 가지 형태

1. if 단독 - 특정 상황에만 무언가를 덧붙일 때
   1. 조건이 거짓이면 그냥 넘어감
   2. else 가 필요 없는 가장 흔한 형태

```python
age = 10

if age < 20:
    print('미성년자')
```

- if-else 반드시 둘 중 하나

```python
age = 23

if age > 20:
    print('성인')
else:
    print('미성년자')
```

- if-elif

```python
rank = 5
if rank == 1:
    print('금메달')
elif rank == 2:
    print('은메달')
elif rank == 3:
    print('동메달')

print('경기 종료')
```

- if-elif-else

```python
score = 85

if score >= 90:
    grade = 'A'
elif score >= 80:
    grade = 'B'
elif score >= 70:
    grade = 'C'
else:
    grade = 'F'

print(grade)  # B
```

#### 독립 if 와 elif

#### 중첩 조건문

- 조건문 안에 조건문 가능

---

## 반복문

Loop Statement

주어진 코드 블록을 여러 번 반복해서 실행하는 구문

### 반복문의 종류

- for문
  - 반복 가능(iterable)한 객체의 요소들을 반복하는데 주로 사용
  - 주로 반복 가능(iterable)한 객체 요소의 개수만큼 반복
  - 특징: 반복 횟수가 정해져 있음

- while문
  - while 조건이 참(True)인 동안 반복
  - 반복 횟수가 정해지지 않은 경우 주로 사용

#### for 반복문

##### 반복 가능한 객체

iterable

시퀀스 자료형(list, tuple, str, range) 뿐만 아니라 비 시퀀스 자료형(dict, set) 등도 반복 가능한 객체

- for문 작동원리
  - 리스트 내 첫 항목이 반복 변수(item)에 할당되고 코드블록이 실행
  - 반복 변수에 리스트의 2번째 항목이 할당되고 코드블록이 실행
  - 위 항목을 계속 반복
  - 더 이상 방복 변수에 할당할 값이 없으면 반복 종료

> 반복 변수는 단수형으로 작성한는 것을 권장

## 반복 제어

### 반복 제어 키워드

- break
  - 해당 키워드를 만나게 되면 남은 코드를 무시하고 반복 즉식 종료
  - 반복을 끝내야 할 명확한 조건이 있을 때 사용
- continue
  - 해당 키워드를 만나게 되면 다음 코드를 무시하고 다음 반복을 수행

> TIP
> 반복 제어문은 반드시 반복문 내에서 사용
> 중첩 반복문인 경우 해당 키워드가 작성된 코드 블록의 반복 흐름만 제어함
> 과도하게 사용 시 가독성이 떨어짐

### 빈 코드 블록 키워드

- pass
  - '아무 동작도 하지 않음'을 명시적으로 나타내는 키워드
  - 반복 제어가 아닌 코드의 틀을 유지하거나 나중에 내용을 채우기 위한 용도

## 유용한 내장 함수 map

map(function, iterable)

- 반복 가능한 데이터구조(iterable)의 모든 요소에 function을 적용하고, 그 결과 값들을 map object로 묶어서 반환

```python
numbers = [1, 2, 3]
result = map(str, numbers)

print(result)  # <map object at 0x00000239C915D760>
print(list(result))  # ['1', '2', '3']
```

> **map object**
> 결과를 하나씩 꺼내 쓸 수 있는 반복 가능한 객체 자료형, 전체 값을 확인하려면 list나 tuple로 형변환을 해줘야 함

1. map 활용 1
   1. 리스트에 들어있는 데이터의 형변환을 위하여 사용
2. map 활용 2
   1. 람다(lambda)를 활용한 방법
   2. 일회용 함수를 사용할 때

## 참고

내장함수 help

for-else

enumerate

zip 함수

이터레이터, 제너레이터 찾아보기