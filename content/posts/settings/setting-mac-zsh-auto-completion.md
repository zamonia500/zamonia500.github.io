---
title: "Setting Mac Zsh Auto suggestion"
date: 2022-03-20T15:06:32+09:00
draft: false
categories: ["setting"]
tags: ["zsh"]
---

**[맥 zsh에서 명령어 자동완성 기능 추가(MacOS Oh My Zsh autosuggestions)](https://yeonfamily.tistory.com/15)**를 참고하여 zsh autosuggestion 설정을 했다.
zsh + oh-my-zsh + powerlevel 10K 설정은 이미 되어있는 상태

```
# git clone을 통해 plugin 다운로드
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

```
# ~/.zshrc 파일 설정, git plugin은 기존에 있던 녀석이였다.
plugins=(git zsh-autosuggestions)
```


