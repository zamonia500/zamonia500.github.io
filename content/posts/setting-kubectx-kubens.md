---
title: "kubectx, kubens 설정하기"
date: 2022-03-13T16:35:16+09:00
draft: false
tag: ["kubernetes"]
category: [“setting”]
---

## kubectx, kubens란?

**[What are kubectx and kubens](https://github.com/ahmetb/kubectx#what-are-kubectx-and-kubens)**
> kubectx is a tool to switch between contexts (clusters) on kubectl faster.
kubens is a tool to switch between Kubernetes namespaces (and configure them for kubectl) easily.

- kubectx : Multi-Kubernetes-Cluster 환경에서 kubectl cli tool이 바라보는 cluster를 쉽게 switch 할 수 있게 해 주는 툴
- kubens : Cluster에 존재하는 Multi-Namespaces들 중 default namespace를 쉽게 switch 해 주는 툴

## Install kubectx, kubens

공식 가이드에서는 brew를 통해 install 할 경우 자동완성(auto-completion) 설정이 진행된다고 했지만 잘 안되었다.
그래서 brew uninstall로 다시 지우로 [manual-installation-macos-and-linux](https://github.com/ahmetb/kubectx#manual-installation-macos-and-linux)을 따라 다시 설치했다.

```
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

## Setup auto completion

[zsh](https://github.com/ahmetb/kubectx#completion-scripts-for-plain-zsh)를 위한 자동완성을 설정해 주었다.

```
mkdir -p ~/.oh-my-zsh/completions
chmod -R 755 ~/.oh-my-zsh/completions
ln -s /opt/kubectx/completion/_kubectx.zsh ~/.oh-my-zsh/completions/_kubectx.zsh
ln -s /opt/kubectx/completion/_kubens.zsh ~/.oh-my-zsh/completions/_kubens.zsh
```

## Test kubens

minikube에서 kubens를 테스트 해 보자

```
❯ kubens
default
istio-system
kube-node-lease
kube-public
kube-system
```

자동완성 테스트

```
$ kubens default
-                default          istio-system     kube-node-lease  kube-public      kube-system
```

**자동완성이 동작하지 않을 때**

```
echo "autoload -U compinit && compinit" >> ~/.zshrc
source ~/.zshrc
```
