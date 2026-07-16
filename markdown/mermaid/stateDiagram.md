## State Diagram
### 상태
```markdown
stateDiagram-v2
Idle
Running
Stop
```
```mermaid
stateDiagram-v2
Idle
Running
Stop
```
### 상태전이
```markdown
Idle --> Running
Running --> Stop
```
```mermaid
stateDiagram-v2

Idle --> Running
Running --> Stop
```

### 시작 상태
```markdown
[*] --> Idle
```
```mermaid
stateDiagram-v2

[*] --> Idle
```
### 종료 상태
```markdown
Idle --> [*]
```
```mermaid
stateDiagram-v2

Idle --> [*]
```

### 전이 이름(Event)
```markdown
Idle --> Running : Start 버튼

Running --> Stop : Stop 버튼
```
```mermaid
stateDiagram-v2

Idle --> Running : Start 버튼

Running --> Stop : Stop 버튼
```
### 자기자신을 이동
```markdown
Running --> Running : Timer
```
```mermaid
stateDiagram-v2

Running --> Running : Timer
```
### 상태 설명
```markdown
state Running {
    Execute
    Save
    Display
}
```
```mermaid
stateDiagram-v2

state Running {
    Execute
    Save
    Display
}
```
### Composite State (중첩 상태)
```markdown
state Login {

    [*] --> Input

    Input --> Check

    Check --> Success

}
```
```mermaid
stateDiagram-v2

state Login {

    [*] --> Input

    Input --> Check

    Check --> Success

}
```

### Choice (분기)
```markdown
[*] --> Check

Check --> Success : OK

Check --> Fail : Error
```
```mermaid
stateDiagram-v2

[*] --> Check

Check --> Success : OK

Check --> Fail : Error
```
### 방향 변경
```markdown
direction LR , TB

[*] --> Idle

Idle --> Running

Running --> Stop
```
```mermaid
stateDiagram-v2

direction LR

[*] --> Idle

Idle --> Running

Running --> Stop
```

### Note
```markdown
[*] --> Idle

note right of Idle
대기 상태
end note
```
```mermaid
stateDiagram-v2

[*] --> Idle

note right of Idle
대기 상태
end note
```
---
### Fork / Join
병렬 상태를 표현
```markdown
state fork_state <<fork>>

state join_state <<join>>
```
```mermaid
stateDiagram-v2

state fork_state <<fork>>

state join_state <<join>>
```
---
## 예시안
### 버튼 제어
```mermaid
stateDiagram-v2

[*] --> Released

Released --> Pressed : 버튼 누름

Pressed --> Released : 버튼 뗌
```
---
### LED 제어
```mermaid
stateDiagram-v2

[*] --> OFF

OFF --> ON : Switch

ON --> OFF : Switch
```
---
#### UART 통신
```mermaid
stateDiagram-v2

[*] --> Idle

Idle --> Receive : 데이터 수신

Receive --> Process : CRC 확인

Process --> Send : 응답 생성

Send --> Idle : 완료
```
---
#### 로그인 상태
```mermaid
stateDiagram-v2

[*] --> Logout

Logout --> Login : 로그인 성공

Login --> Logout : 로그아웃
```