---
layout: post
title: R 세션 시작 시 초기화 과정
categories:
  - R
---

R 세션 시작 시 초기화 과정
======================

리눅스에서 환경 변수를 설정하기 위해 사용자 홈디렉터리에 있는 '.bashrc'파일을 수정하곤 했습니다.
RStudio로 R 코딩을 하던 차에 환경 변수 설정이 필요하여 이번에도 '.bashrc'를 수정했는데 아무리 해도 환경 변수가 R 세션에서 보이지 않았습니다.
한동안은 지금까지 했던 것처럼 환경 변수 정의 코드를 복사해서 붙여놓고 재실행했지만 매번 이렇게 하는 것이 거추장스럽기도 하고 "기술 부채"가 늘어나는 것처럼 보여 그다지 마음이 좋지는 않았습니다.

이번 기회에 R에서 환경 변수 설정하는 방법을 제대로 알아두자고 생각했고 다행히도 문서가 잘 작성되어 있어서 필요한 정보를 얻을 수 있었습니다:
- [R: Initialization at Start of an R Session](https://stat.ethz.ch/R-manual/R-devel/library/base/html/Startup.html)
- [R: Environment Variables](https://stat.ethz.ch/R-manual/R-devel/library/base/html/EnvVar.html)
- [Advanced R: Environments](http://adv-r.had.co.nz/Environments.html)


결과적으로 저는 '~/.Renviron' 파일에 필요한 환경 변수를 정의했습니다.


## 시작 시 초기화 과정

1. 환경 변수 파일 읽기
    1. site 환경 변수 파일
    2. user 환경 변수 파일
2. 프로필 파일 읽기
    1. site 프로필 파일 읽기
    2. user 프로필 파일 읽기
3. 사용자 작업 디렉터리에서 저장된 이미지 읽기
4. 'search path'에서 `.First` 함수 찾아 실행
5. base 패키지에 있는 `.First.sys()` 함수 실행

이 과정 중에서 환경 변수 파일과 프로필 파일을 읽는 과정을 좀 더 상세하게 보겠습니다.
'search path'에 대해서 궁금하신 분은 위 참고 문서 중에서 [Advanced R: Environments](http://adv-r.had.co.nz/Environments.html)를 참고해주십시오.
아래에 있는 `.First` 함수 실행은 저에게는 필요하지 않아서 그냥 넘어가겠습니다.


## 1. 환경 변수 파일 읽기

- 환경 변수 파일은 다음과 같이 환경 변수를 정의한 파일입니다.
```bash
R_LIBS=~/R/library
```

### 1-1. site 환경 변수 파일

커맨드 라인에서 `--no-environ` 옵션 지정하지 않았을 때만 읽습니다.

파일을 찾는 순서는 아래와 같습니다.
1. `R_ENVIRON` 변수가 가리키는 파일
2. `R_ENVIRON` 변수가 없으면 `R_HOME/etc/Renviron.site` (기본 설치에서는 만들지 않음)

### 1-2. user 환경 변수 파일

커맨드 라인에서 `--no-environ` 옵션 지정하지 않았을 때만 읽습니다.

파일을 찾는 순서는 아래와 같습니다.
1. `R_ENVIRON_USER` 변수가 가리키는 파일
2. `R_ENVIRON_USER` 변수가 없으면 현재 디렉터리와 사용자 홈디렉터리 순으로 `.Renviron` 파일을 찾음


## 2. 프로필 파일 읽기

- 프로필 파일은 환경 변수 파일과는 다르게 R 코드가 포함된 파일입니다.
- site와 user 프로필은 base 패키지를 로드한 후 읽어들입니다.
- 'tilde expansion'은 '~' 기호를 사용자 홈디렉터리로 변환하는 과정을 말합니다.

### 2-1. site 프로필 파일

커맨드 라인에서 `--no-site-file` 옵션 지정하지 않았을 때만 읽습니다.

파일을 찾는 순서는 아래와 같습니다.
1. `R_PROFILE` 변수가 가리키는 파일 (tilde expansion 적용함)
2. `R_PROFILE` 변수가 없으면 `R_HOME/etc/Rprofile.site` 파일을 찾음 (기본 설치에서는 만들지 않음)

읽은 코드는 base 패키지에 포함됩니다.
그러므로 기존에 있던 base 패키지의 내용을 덮어씌우지 않도록 조심해야 합니다.
[이 문서](https://stat.ethz.ch/R-manual/R-devel/library/base/html/Startup.html)에서는 `local`을 사용하는 것을 추천한다고 합니다.
`local`을 사용한 예가 맨 아래 예제 코드에 있습니다.

### 2-2. user 프로필 파일 읽기

커맨드 라인에서 `--no-init-file` 옵션을 지정하지 않았을 때만 읽습니다.
읽은 코드는 workspace에 포함됩니다.

1. `R_PROFILE_USER` 변수가 가리키는 파일 (tilde expansion 적용함)
2. `R_PROFILE_USER` 변수가 없으면 현재 디렉터리와 사용자 홈디렉터리 순으로 `.Rprofile` 파일을 찾음

