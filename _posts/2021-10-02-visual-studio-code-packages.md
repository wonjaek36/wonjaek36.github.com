---
layout: post
title: "Visual Studio Code Packages"
date: 2021-10-02 22:44+0900
tags: [vscode]
---

개인적으로 사용하는 vscode packages 설명 글  
업무에 python, docker를 사용하기에 git, python, docker 관련 패키지를 많이 사용  

## Environment
* Visual Studio Code 1.60.2

## 1. Git
### 1.1 Git History
* Object: GUI for git history
* Provider: Don Jayamanne
* Link: https://marketplace.visualstudio.com/items?itemName=donjayamanne.githistory
* Functionaliy:
  * Git log를 Visualization(branch, file)
  * branches, commit 별 비교
  * 더 자세한 기능은 링크 참조

### 1.2 Git Lens
* Object: Change previous commit easily
* Provider: Eric Amodio
* Link: https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens
* Functionality:
  * 기능이 다양하고, Git History보다 더 많은 기능을 제공함
  * 여기서 주로 사용하는 기능은 "Open Changes with Previous Revision", 이전 git commit과 현재 파일을 직접 비교
  * Line by Line으로 누가 commit하였는지 Annotation 해줌


## 2. Docker
### 2.1 Docker
* Object: Docker extension for vscode
* Provider: Microsoft
* Link: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker
* Functionality:
  * vscode에서 dockerfile을 쉽게 작성할 수 있게 도와주고, 직접 실행할 수 있게 해줌
  * Editor 내에 Docker interface 제공하여, container, image, registry 대한 인지 및 조작이 가능

### 2.2 Docker Linter
* Object: Linter for python, ruby, etc in container 
* Provider: Henrik Sjööh
* Link: https://marketplace.visualstudio.com/items?itemName=henriiik.docker-linter
* Functionality:
  * Container 내 4가지 언어 Linter
  * 참고: Linter는 언어별 코딩 컨벤션(Coding Convention)과 에러 체크를 도와주는 툴 

### 2.3 Remote - Containers
* Object: Access Remote Container File System
* Provider: Microsoft
* Link: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers
* Functionality:
  * Container 내 파일 시스템에 직접 접근하고 VS Code로 구현

### 2.4 Remote - WSL (Only Window)
* Object: Access WSL
* Provider: Microsoft
* Link: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl
* Functionality:
  * WSL 내 파일 시스템에 직접 접근하고 Vs code로 구현
  * WSL 내에서 code .로 Vscode를 실행해서 사용하는 것이 편함

## 3. Python
### 3.1 Python Extension Pack
* Object: Python 개발에 자주 사용하는 패키지 설치
  * Python
  * Visual Studio IntelliCode
  * Django
  * MagicPython
  * Jinja
* Provider: Don Jayamanne
* Link: https://marketplace.visualstudio.com/items?itemName=donjayamanne.python-extension-pack
* Functionaliy
  * Vscode로 파이선 개발에 추천하는 패키지들
  * 특히, 웹개발자들에게 적합

### 3.2 Python Docstring Generator
* Object: Make easy to create docstring
* Provider: Nils Werner
* Link: https://marketplace.visualstudio.com/items?itemName=njpwerner.autodocstring
* Functionality:
  * Function 바로 다음 줄에 doc-string을 작성하기 위해 """ 하고 newline을 넣으면 doc string template 제공
  * template은 demo를 참고
  
### 3.3 Python Preview
* Object: Python Preview
* Provider: Dong li
* Link: https://marketplace.visualstudio.com/items?itemName=dongli.python-preview
* Functionality
  * 아직 잘 모르지만, Python이 실행되는 과정을 line by line으로 보여줌
  * 특히, 메모리 레벨의 상황을 그림으로 보여주는 것이 매우 직관적

### 3.4 Jupyter & Jupyter Keymap
* Object: Jupyter Interface for vscode
* Provider: Microsoft
* Link
  * Jupyter: https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter
  * Jupyter Keymap: https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter-keymap
* Functionality
  * Jupyter Interface를 vscode에서 제공


## 4. Tools
### 4.1 Prettier - Code Formatter
* Object: Reformat your code more prettier(JSON, YAML, etc)
* Provider: Prettier
* Link: https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode
* Functionality
  * 아직 옵션 설정 때문에 사용하지 못 해봄
  * JSON File을 Reformat 해주면, 설정 파일 작성이 매우 편해질 듯

### 4.2 SSH FS
* Object: SSH Code
* Provider: Kelvin Schoofs
* Link: https://marketplace.visualstudio.com/items?itemName=Kelvin.vscode-sshfs
* Functionality
  * Remote Server File System, Terminal 접근
  * 한 번 설정해놓으면, 매우 편하게 접근 가능



## 5. Theme
### 5.1 Material Icon Theme
* Object: Change Icon as material style
* Provider: Philipp Kief
* Link: https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme
* Functionality
  * file-tree에서 icon을 예쁜 material style로 바꿔줌
  * Colorful하고 직관적이기 때문에 편함

### 5.2 Linux Themes for VS Code
* Object: Change color scheme for vscode
* Provider: SolarLiner
* Link: https://marketplace.visualstudio.com/items?itemName=SolarLiner.linux-themes
* Functionality
  * Linux style theme를 제공
  * 취향 저격