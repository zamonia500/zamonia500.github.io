---
title: "Three Git Flows"
date: 2022-08-28T20:33:27+09:00
draft: false
categories: [""]
tags: ["git"]
---

협업, 출시 형상 관리는 어렵다.
개발은 개발대로 해야 하고, 출시 준비는 준비대로 해야 한다.

QA는 해야 겠는데 계속 추가 기능이 반영되는 브랜치에서 QA를 진행하게 된다면 지속적으로 병합되는
다른 기능들 때문에 이미 수행된 테스트 케이스들이 여전히 정상 동작하는지 알 수 없다.

이런 어려움을 잘 해쳐 나가기 위해서 `git flow`, `github flow`, `gitlab flow` 따위의 git branch 사용 전략이 많이 제시되었다. 이 전략들에 대해서 알아보자.

# Git flow

![](/images/posts/git-flow_overall_graph.png)

기본 아이디어 : 외부 코드 변화에 영향을 받고싶지 않으면 브랜치를 따서 작업한다.

5 종류의 브랜치가 존재한다.
- master : 배포 브랜치
  - release, hotfix 브랜치에서 머지될 수 있다
- develop : 개발 브랜치
  - feature 브랜치에서 머지될 수 있다
- feature : 신규 기능 개발 브랜치
  - develop 브랜치에서 feature 브랜치를 생성하여 작업한다
- release : 배포 준비 브랜치
  - develop 브랜치에서 release 브랜치를 생성하여 작업한다.
  - release 브랜치에서 QA를 수행하면서 버그가 발견될 경우, release 브랜치에 직접 반영하고 develop 브랜치에 반영한다.
- hotfix : master 브랜치에 이슈가 발견되었을 경우 이슈 해결을 위한 브랜치
  - master 브랜치에서 직업 hotfix 브랜치를 생성하여 작업한다.
  - master / develop 브랜치에 각각 반영한다.

실 개발에 적용했을 때 다음과 같은 흐름이 될 것 같다.

1. main, develop 브랜치를 생성한다. develop 브랜치가 default 브랜치가 된다.
2. develop에서 feature 브랜치를 생성하여 기능 개발을 하고, 작업 결과를 develop 브랜치에 반영한다.
3. 배포하고자 하는 기능의 개발이 모두 완료되었다면 release 브랜치를 생성하고 기능 검증을 시작한다.
4. release 브랜치에서 버그가 발견되었을 경우 release 브랜치에서 부터 bugfix 브랜치를 생성하여 버그를 수정하는 변경을 release 브랜치에 반영한다. 이 버그 픽스는 develop 브랜치에도 반영되어야 하므로, release -> develop 브랜치로 반영한다.
5. release 브랜치에서 발견된 모든 이슈가 해결되었다면 release -> master로 머지하고 tag를 생성한다. 이 태그에 대해서 배포를 진행한다.
6. 배포가 된 코드에서 버그가 발견되었을 경우, master 브랜치에서 hotfix 브랜치를 생성한 뒤 변경 내역을 master, develop 브랜치에 각각 반영한다.
7. hotfix에 의해 master 브랜치가 변경되었기 때문에 tag를 새로 생성하고 다시 배포한다.

### 곰곰히 생각해보자

**단점**
- 관리해야 하는 브랜치가 너무 많다
- 개발 내역을 서로 다른 두 개의 브랜치에 반영해야 하는 경우가 있다. 사람이 실수로 누락하기 쉬워 보인다
- Pull request에 대한 가이드는 없지만 "Merge는 Pull request를 통해 진행한다"라는 컨벤션을 가져가면 될 것 같다


# Github flow

![](/images/posts/github-flow.png)

단 하나의 원칙만 기억하면 되는 아주 간단한 워크플로우다.

"main 브랜치에서 새 브랜치를 생성한 뒤 main 브랜치로 머지해라."

개발 단계롤 조금 자세히 들여다 보자.

**Create a branch**
- 브랜치 이름은 의미있는 것을 사용한다

**Make Changes**
- 필요한 개발을 한다
- 각각의 커밋은 독립적이고 완결한 작업인 것이 좋다
- push도 마음껏 해라
- 브랜치를 잘 생성해라. 하나의 브랜치에서는 하나의 작업만 하는 것이 리뷰 받기에 좋다

**Create a PR**
- PR 리뷰는 중요한 작업이므로, 어떤 작업에 대한 PR인지 잘 적어주자

**Merge your pull request**

**Delete your branch**

### 곰곰히 생각해보자

**장점**
- 간단하다
- github의 자동화 기능을 이용하여 CICD를 적용할 수 있다.

**단점**
- 고정된 형상을 대상으로 QA를 한다는 개념이 없다. "항상 main은 배포 가능한 상태"를 가정하고 있기 때문이다
- monorepo 구조에서 특정 컴포넌트에 대한 기능 개발만 추적하면서 배포를 준비할 수 없다
- release에 대한 개념이 없다. release는 CICD를 구축한 뒤 신경쓰지 마세요~ 라고 하는건가?


# Gitlab flow

### Git flow의 단점

잘 만들어졌지만 복잡해서 2가지 문제점이 있다.

main이 아닌 develop 브랜치를 사용해야 한다. 보통 main을 default branch로 사용하기 때문에 default branch를 develop으로 변경하는 번거로운 작업을 해야 한다 → _minor 한 이슈로 아닙니꽈_

hotfix / release 브랜치 때문에 복잡성이 너무 높다. 최신 형상 관리는 Continuous delivery와 연동하여 관리하는 경우가 많고, 이는 당신의 default 브랜치가 배포 가능한 상태 라는 소리다. CD가 있다면 hotfix / release 브랜치는 필요 없다.

hotfix에 대한 변경을 main 에만 반영하고 develop 브랜치에는 반영하지 않는 실수가 잦다.

### 간단한 Github flow

git flow 보다 더 간단한 Github flow는 main / feature branch만 존재한다. 하지만 deployment / environments(배포 환경) / release 등의 내용이 빠져있다.


## Gitlab flow

기본 아이디어 : development / production 브랜치를 두고, 배포할 때 development 브랜치를 production 브랜치에 머지한다.

### Environment branched with Gitlab flow

- QA / production과 같은 environment가 있을 수 있다. 
  - environment를 ‘배포 환경’으로 이해했다. QA는 release candidate를 기반으로 배포한 형상을 검증하기 위해 배포한 개발 서버이고, production은 QA이슈가 해결된 코드를 실 release 한 뒤, 해당 형상을 기반으로 배포한 운영 서버이다.
- production environment에서 문제가 발생했을 때, hotfix branch를 생성해서 production branch에 반영하는 경우가 종종 있는데 이 때 머지, 이슈 해결 확인, development 브랜치 반영 작업이 필요하다.

### Release branched with Gitlab flow

![](/images/posts/gitlab_flow.png)

- release가 필요하면 release 브랜치를 생성한다, `main` 브랜치에서 생성하되, 최대한 늦게 생성하는게 좋다. 이 브랜치에는 bugfix만 들어갈 수 있다. bugfix는 main에 반영한 다음에 cherry-pick 해서 반영한다.
- release 브랜치에 bugfix에 대한 변경이 있을 때 마다 patch version을 올려라.

### Merge/Pull requests with Gitlab flow

- _Pull request와 리뷰 과정은 널리 익숙한 작업이기 때문에 스킵_

### Issue tracking(ex. Jira, Github issue) with Gitlab flow

- 1시간 이상의 작업이 필요하다면 (Jira) 이슈를 생성하자
- 하나의 이슈는 최대 하나의 branch만 가질 수 있다.
- 하나의 branch가 하나 이상의 이슈를 해결할 수 있다.

### Squashing commits with rebase

- rebase? 가능
- push된 commit에 대한 rebase? 음 contributor가 있다면 하지 않는것이 좋다
- 남의 코드를 rebase 했다가는 뚝배기가 깨질 수 있습니다 휴먼. 절대 동의 없이 하지 마십쇼
- Squash-and-Merge를 사용할 수 있는데, 이 때 최대 2개의 commit이 머지될 수 있다. (squashed commit & merge commit)
  - Github에는 Squash-and-Merge가 없다. Squash Merge는 squashed commit 단 하나만 들어간다. Squash-and-Merge를 하기 위해서는 리뷰를 마치고 머지를 하기 전에 로컬에서 squash를 하고 Merge commit을 해야한다

### Reducing merge commits in feature branches

- feature 브랜치에 머지 커밋이 많이 들어가는 건 좋지 않다 
- main을 feature branch에 반영하고 싶은 상황은 다음과 같은 것들이 있다.
  1. main에 반영된 새 코드가 필요할 때
      - `cheery-pick`으로 해결할 수 있다면 cherry-pick을 해라
  2. merge conflict를 해결하고 싶을 때
      - merge conflict가 있다면 main merge를 해도 좋다. (rebase 지옥이라면..)
  3. 오래된 브랜치를 업데이트 하고 싶을 때.
      - 브랜치가 오래 된 것 부터 문제다. 언능 개발하고 머지해라 ㅋㅋㅋㅋ


## 11가지 gitlab flow best practices

1. main에 바로 commit하지 말고 feature branch를 써라
2. 메인을 테스트 하는 것이 아니라 모든 커밋을 테스트 해라. ( 머지 전에 테스트 해라)
3. 모든 커밋에 모든 테스트를 돌려라 (테스트가 5분 이상 걸리면, 병렬처리해라)
4. 메인에 머지하기 전에 코드리뷰 해라
5. branch 또는 tag가지고 자동으로 배포해라
6. tag는 CI가 아니라 사람이 찍어라
7. tag를 사용하여 release 해라 (하나의 tag는 하나의 release를 의미한다.)
8. push한 commit은 rebase 하지 마라
9. main에서 따서 main으로 넣는다. (우리는 이렇게 사용하지 않는 부분이 있는듯?)
10. 버그 픽스는 main에 먼저 반영한다.
11. 커밋 메세지에는 “무엇을 했는가” 뿐만이 아니라 “왜 했는가?”에 대한 내용도 같이 적어라.
  -  다른 여러가지 선택지들 중에서 왜 이러한 방법을 사용했는지 설명하는게 더 좋다. (음 귀찮겠군)


**Refs**
- [우린 Git-flow를 사용하고 있어요](https://techblog.woowahan.com/2553/)
- [Github flow](https://docs.github.com/en/get-started/quickstart/github-flow)
- [Git flow, GitHub flow, GitLab flow](https://ujuc.github.io/2015/12/16/git-flow-github-flow-gitlab-flow/)
- [Introduction to GitLab Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html)
- [What are GitLab Flow best practices?](https://about.gitlab.com/topics/version-control/what-are-gitlab-flow-best-practices/)
