---
title: "Git 용어, 명령어 정리"
excerpt: "Git에서 사용하는 기본적인 용어 및 command를 간단하게 정리한다."
categories: 
  - Programming Language
tags:
  - Git
toc: true
toc_sticky: true
---

Git은 버전 관리 시스템(VCS)의 일종으로 파일의 변경 이력을 저장해 버전을 관리하고 다른 사람과의 협업을 쉽게 해 준다. 이 글에서는 Git에서 사용하는 기본적인 용어와 명령어를 간단하게 알아보고, 세부적인 사항은 추후 별도의 글로 정리할 예정이다.

## 용어
- ``repository`` : 파일이 저장되어 있는 저장소. 원격 서버 저장소는 remote repository, 개인 디바이스 내 저장소는 local repository라고 한다.
- ``commit`` : 파일/폴더의 변경 사항을 로컬 저장소에 기록한다. commit을 실행하면 직전 commit부터 현재까지의 변경 사항이 기록된 새로운 commit이 만들어진다. commit을 버그 수정, 기능 추가와 같은 유의미한 작업 단위로 실행하면 변경 사항을 찾기 쉽다. commit은 저장소에 이력을 남기기 때문에 commit에 대한 메시지를 첨부해야 한다.
- ``index / stage`` : 실제 작업 공간과 원격 저장소 사이에 존재하는 공간이다. ``git add`` 명령을 통해 실제로 commit할 파일을 선택해서 stage에 추가 (staging) 한 뒤 commit한다. 이를 통해 commit이 필요 없는 파일을 제외하거나 파일의 일부 변경 사항만 staging해 commit할 수 있게 된다.
- ``branch`` : 여러 개발자가 서로 독립적으로 작업할 수 있게 해 주는 기능으로, 프로젝트를 여러 갈래로 나눌 수 있다. 각각의 branch에서 파일을 원하는 대로 변경한 뒤 원래의 branch에 합치는 방법으로 독립적인 작업을 수행할 수 있다.

## 명령어
- ``git init`` : 현재 디렉토리를 git repository로 설정한다.
- ``git clone <url>`` : ``url``에 존재하는 git 원격 repository를 로컬에 복사한다.
- ``git status`` : 현재 git repository의 상태를 확인한다.
- ``git add <directory/file>`` : 특정 파일이나 디렉토리 전체를 지정하여 staging한다. 만일 아직 git에서 추적하지 않던 파일/디렉토리라면 추적을 시작한다.
- ``git commit -m <msg>`` : staging된 파일을 ``msg`` 메시지를 첨부해 commit한다.
- ``git push <remote> <branch>`` : commit을 ``remote`` 저장소의 ``branch`` 브랜치에 업로드한다.
- ``git fetch <remote>`` : ``remote`` 저장소의 파일을 가져온다.
- ``git pull <remote> <branch>`` : ``remote`` 저장소의 ``branch`` 브랜치를 가져와 로컬 저장소에 합친다. 
-- 일반적으로 처음 repository를 clone하면 저장소 이름은 ``origin``으로, 메인 브랜치의 이름은 ``master``로 지정된다. 
- ``git merge <branch_name>`` : 현재 브랜치에 ``br_name`` 브랜치를 병합한다.
- ``git branch <br_name>`` : ``br_name`` 이름을 가진 새 브랜치를 생성한다.
- ``git checkout <br_name>`` : ``br_name`` 브랜치를 선택해 작업을 시작한다.
- ``git log`` : 현재 브랜치의 수정 내역을 확인한다.

## 출처
- [누구나 쉽게 이해할 수 있는 Git 입문](https://backlog.com/git-tutorial/kr/)
- [Git Documentation](https://git-scm.com/book/ko/v2/)
