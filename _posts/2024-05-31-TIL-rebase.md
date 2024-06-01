---
title: "git merge"
excerpt: " "
categories:
  - Programming Language
tags:
  - [Git]
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---

## merge

한 브랜치의 변경 사항을 모두 다른 브랜치에 적용하는 방식이다. `git merge <타겟 브랜치명>` 명령어를 통해 실행할 수 있으며, 실행하면 타겟 브랜치를 현재 checkout하고 있는 브랜치로 병합한다. 타겟 브랜치를 내 브랜치로 당겨온다고 생각할 수도 있을 것이다. merge 커맨드에는 브랜치 사이의 관계에 따라 두 종류의 방식이 존재한다.

### Fast-forward (FF) merge

가장 단순한 merge 방식이다. 현재 브랜치가 병합하려는 브랜치의 조상일 때 실행되는 merge 방식으로, 커밋 그래프가 선형인 상황이라고 생각하면 이해하기 쉬울 것이다. 병합하려는 타겟 브랜치가 내 브랜치보다 최신의 커밋일 경우 병합할 수 없고 (최신 브랜치를 과거 브랜치로 당겨올 수 없다.), 타겟 브랜치가 내 브랜치의 자식일 경우 타겟 브랜치가 가리키는 커밋을 내 브랜치가 가리키는 커밋으로 당겨오는 방식으로 merge가 이루어진다.

![Fast-forward merge]({{site.url}}{{site.baseurl}}/assets/img/git-ffmerge.png)

위 그림의 상황에서, B 브랜치를 checkout하고 git merge A 명령어를 실행하면 A 브랜치가 커밋 3으로 이동하는 방식으로 병합된다.

### 3-way merge

대부분의 merge 상황에서 발생하는 merge 방식이다. 현재 브랜치와 병합하려는 타겟 브랜치가 이전에 한 커밋에서 서로 다른 브랜치로 갈라져 나온 경우에 수행되는 merge 방식으로, Y자 형태의 커밋 그래프를 상상할 수 있다.

3-way merge는 **타겟 브랜치의 모든 변경사항을 현재 브랜치에 새로 커밋**하는 방식으로 이루어진다. 그래프를 보면 현재 브랜치에서 새 커밋이 생성되고, 타겟 브랜치의 HEAD가 현재 브랜치의 새 커밋으로 연결되는 형태로 그려짐을 알 수 있다. 이 때 생성되는 새 커밋을 Merge Commit이라고 한다.

![3-way merge]({{site.url}}{{site.baseurl}}/assets/img/git-3waymerge.png)

Commit 4에 위치해 있던 Branch A를 checkout한 상태에서, `git merge B` 커맨드를 사용해 Commit 5에 있던 Branch 5를 merge하였다. 두 브랜치는 이전에 다른 변경사항으로 갈라져 나왔으므로 3-way merge가 수행되고, 현재 브랜치인 A에 merge commit이 생성되며 B 브랜치의 변경사항을 당겨와 반영하게 된다. 이 때, 서로 다른 변경사항이 있는 두 브랜치를 병합하는 것이기 때문에 발생할 수 있는 conflict 상황에 유의해야 한다.

### merge conflict

3-way merge를 수행할 때는 이전에 서로 다른 분기로 갈라져 나온 두 브랜치를 병합하기 때문에, **같은 파일의 같은 라인을 서로 다르게 수정한 경우**가 생길 수 있다. 이럴 때 git은 merge 과정을 잠시 중단하고 conflict가 발생했다는 경고 메시지를 띄우며, 사용자에게 conflict 상황을 해결한 후 merge commit을 생성할 것을 요구한다. 이 때 git은 conflict가 발생한 부분을 쉽게 알 수 있도록 두 브랜치가 비교되게 자동으로 파일을 수정하는데, 그 구조는 다음과 같다.

> \<<<<<<< HEAD \
> 현재 브랜치의 변경 내용 \
> \======= \
> 병합할 브랜치의 변경 내용 \
> \>>>>>>> 병합할 브랜치명

conflict가 발생한 부분을 찾아 두 브랜치의 변경 내용을 비교하고, 충돌이 일어나지 않게 수정한 뒤 해당 파일을 커밋하면 merge가 완료된다.