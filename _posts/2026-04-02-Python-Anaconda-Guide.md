---
title: "Anaconda 설치 및 다중 사용자 설정 가이드"
date: 2026-04-02
description: Anaconda 다중 사용자 환경(공용 폴더, 그룹 권한, 환경 변수) 구축 및 유의사항
categories: [devops]
tags: [anaconda, conda, python, multi-user, 환경설정, 그룹권한]
---

## Anaconda 설치

[Free Download | Anaconda](https://www.anaconda.com/download#downloads)  
위 공식 사이트에서 운영체제에 맞는 Anaconda 설치 bash 파일을 다운받습니다.

받은 bash 파일 실행 시, **설치 디렉터리를 명확히 지정**해야 하며, 지정하지 않으면 현재 계정의 홈 디렉터리에 설치됩니다.

```bash
# bash 파일 실행하여 설치 디렉터리 지정 (예시)
/bin/bash [받은 Anaconda 설치 파일]

# 예시: bash Anaconda3-2023.07-1-Linux-x86_64.sh
```

---

## 설치 폴더 권한 및 그룹 설정

- 여러 사용자가 Anaconda 환경을 공유할 경우, **설치 폴더에 그룹/권한 설정**을 통해 폴더 접근 및 패키지 설치 권한을 관리합니다.

```bash
# 그룹 생성 (예시: mygroup)
sudo addgroup [그룹명]

# 설치 폴더의 그룹을 변경 (예시: /PATH/TO/ANACONDA/INSTALL)
sudo chgrp -R mygroup /PATH/TO/ANACONDA/INSTALL

# 그룹에 읽기/쓰기/실행 권한 부여
sudo chmod 770 -R /PATH/TO/ANACONDA/INSTALL

# 해당 그룹에 사용자를 추가
sudo adduser [유저명] [그룹명]
```

---

## 환경 변수 설정 (.bashrc 적용)

- 각 사용자 홈 디렉터리의 `.bashrc` 파일에 Anaconda 환경 변수를 등록해야 conda를 사용할 수 있습니다.
- (예시) `/home/{Path}/anaconda3`에 설치된 경우, 아래 코드를 `.bashrc` 하단에 추가

```bash
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/home/{Path}/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup";
else
    if [ -f "/home/{Path}/anaconda3/etc/profile.d/conda.sh" ]; then
        . "/home/{Path}/anaconda3/etc/profile.d/conda.sh"
    else
        export PATH="/home/{Path}/anaconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```

- 적용 후 파일을 실행하여 반영

```bash
source ~/.bashrc
```

---

## 환경 생성 및 다중 사용자 유의사항

- conda 환경 이름이 중복되지 않도록 **`conda env list`** 를 통해 현재 환경 리스트 확인 권장
- base 환경에는 모든 유저가 설치 가능한 라이브러리가 존재
- **다른 사용자의 환경에 접근 시 권한이 없다면 base 환경만 사용가능**
    - 모든 사용자가 접근해야 하는 환경 및 패키지는 별도 공용 폴더 및 그룹 권한 부여 필요

---

## 권한/공용폴더 예시 현황

- {Group} 그룹으로 폴더/권한 관리
- `/home/{Path}` 폴더를 공용 디렉터리로 사용

---

## Reference

- [Installing for multiple users — Anaconda documentation](https://docs.anaconda.com/free/anaconda/install/multi-user/)
