## GIT이란?
Working Director: 실제 작업 중인 파일들이 위치하는 영역

Staging Area: Working Directory에서 변경된 파일 중 다음 버전에 포함시킬 파일들을 선택적으로 추가하거나 제외할 수 있는 중간 준비 영역

Repository: 버전 이력과 파일들이 영구적으로 저장되는 영역 모든 버전과변경  이력이 기록됨

```
Working Directory -> Staging Area -> Repository
```

## Commit
변경 파일들을 저장하는 행위
사진을 찍듯이 기록한다 하여 ‘snapshot’ 이라고도 함

## Git의 동작

- git init
    - 로컬 저장소 설정 (초기화)
    - git의 버전 관리를 시작할 디렉토리에서 진행
- git add
    - 변경사항이 있는 파일을 staging area에 추가
- git commit
    - staging area 에 있는 파일들을 저장소에 기록
        - 해당 시점의 버전을 생성하고 변경 이력을 남기는
    
    ```bash
    git commit -m "변경 정보"
    ```
- git status
    - git의 상태를 출력
- git log
    - git의 로그를 출력

#### git 사용하기
- 로컬 저장소 초기화
    - git init
    - Git의 관리를 받기 시작한 디렉토리 내에서는 (master)가 표시됨
- git의 상태를 확인
    - git status
    - git의 상태를 확인 후 해당 내용의 변경점을 확인
- git의 변경점을 저장
    - git commit -m "변경 정보"

git을 commit을 진행할 시 작성자의 정보가 설정이 안되어있을 경우 사용자 정보를 작성하라는 명령어 도움말과 내용을 제공

```bash
SSAFY@2PC067 MINGW64 ~/Desktop/git-practice (master)
$ git commit -m "first commit"
Author identity unknown

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'SSAFY@2PC067.(none)')
```

#### git 기타 명령어

- git 로그를 한줄씩만 간단히 보기

```jsx
git log — oneline
```

- git global 설정 정보 보기

```jsx
git config —global -l
```

#### 로컬

- 현재 사용자가 직접 접속하고있는 기기 또는 시스템
- 개인 컴퓨터 등 사용자가 직접 조작하는 환경

#### git init 주의사항

- Git 로컬 저장소 내에 또다른 Git 로컬 저장소를 만들지 말것
- 최상위의 git은 하위 git 설정을 인식하지 못함
    - git이 관리하는 폴더 내부에는 또다른 git 저장소를 생성하지 말아야 함
    - 확실한 git 관리를 위해서는 세부적으로 작업할 것
- 문제 발생 시 복구 방법
    - 폴더에 있는 .git 폴더를 삭제하면 해당 폴더에서의 git 설정은 삭제됨

#### 마크다운

- 일반 텍스트로 문서를 작성하는 간단한 방법
- 주로 개발자들이 텍스트와 코드를 작성해 문서화하기 위해 사용

#### 마크다운 특징

- 작성된 마크다운은 다른 프로그램에 의해서 변환되어 출력됨
    - 

#### 마크업

!image.png

마크업 언어: 태그 등을 사용하여 문서나 데이터의 논리적인 구조와 의미를 정의하고 컴퓨터가 이를 어떻게 해석하고 보여줘야 할지 지시하는 언어

#### 제목

```markdown
# 안녕하세요 마크다운 테스트입니다.

# H1
## H2
### H3
#### H4

일반 텍스트

### 순서가 없는 리스트
- 첫번쨰
- 두번쨰
- 세번쨰

### 순사가 있는 리스트
1. 첫번째
2. 두번쨰
    1. 두번쨰의 첫번쨰
3. 세번째
    1. 세번쨰의 첫번쨰
    2. 세번쨰의 두번쨰

### 텍스트 강조
**굵게**

*기울임*

~~취소선~~

```

#### README.md

!image.png

### 원격 저장소 Remote Repository

!image.png

#### git 추가 정리

- git remote add origin remote_repo_url
- git remote/add/이름/주소
    - 이름보다는 주소가 더 중요하며 이름은 간단하게 작성하는게 편하기때문에 간단히 작성함
    - origin을 주로 개발자들이 첫번쨰 원격저장소에 많이 쓰임
    - 추가하는 원격저장소 주소 URL을 정확히 작성

#### git push / pull & clone

- push
    - 변경사항만 보냄
    - 원격 저장소에 로컬 저장소의 변경을 보내는 역할
- pull & clone
    - pull
        - 
    - clone

- git push origin master
    - git push/별칭/브랜치명
    - “git아 push해줘 origin이라는 이름의 원격저장소에 master라는 이름의 브랜치를 “ 라는 의미의 명령어
- git pull origin master
    - push와 사용법은 같으며 push가 아닌 pull 만 변경해주면 됩니다.
- git clone remote_urepo_url
    - 원격 저장소 전체를 복제하기 위하여 해당 하는 주소를 입력
    - git clone 한 프로젝트는 이미 git init이 되어있습니다.

!image.png

#### gitignore

- Git에서 파일이나 디렉토리를 추적하지 않도록 설정하는데 사용되는 텍스트 파일
- 중요 파일이나 키를 올리지 않도록 하기 위함

git remote rename 변경전이름 변경후이름

- 

---

TIL(Today I Learned)

- 단순히 배운것 필기
- 배운 것 기반해서 + a
- 막힌 부분이 있었으며 이를 어떻게 해결했다 (과정.)


#### forking workflow
fork
fork를 진행 한 이후 내 저장소로 가져와서 해당 저장소에 변경 이후 이 저장소에서 fork 수신자에게 PR을 할 수 있음

#### 바이브 코딩시 나타나는 버그
AI를 사용하여 기능을 추가하거나 반복적인 수정시 기존의 기능이 망가지는 경우가 생김
-> 회귀버그

#### 유닛테스트 unit test(단위테스트)
컴퓨터에게 기능 단위로 검증 시키는 방법
모든 기능들이 정상적으로 동작하는지 편하게 테스트하 수 있음
테스트 코드가 동일한 입력을 넣었을 때, 동일한 출력값을 확인하는 것

3단계 준비단계 - 실행단계 - 검증단계

- 언제든지 빠르게 기존 기능이 올바르게 동작하는지 확인할 수 있음
- 많은 기능들에 대해서 각각의 테스트코드를 작성해야 하므로 시간이 단축