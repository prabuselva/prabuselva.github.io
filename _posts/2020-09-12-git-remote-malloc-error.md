---
title: "Git Pull - remote malloc error"
related: true
categories:
  - linux
  - hacks
tags:
  - git
  - repository
toc: false  
---

git pull sometimes causes issue,

```shell
remote: fatal: Out of memory, malloc failed (tried to allocate xxxx bytes)
```

This may be caused due to low memory in the git server side (ex: running gitea/gogs server in arm processor).So, we need to configure the ~/.gitconfig file with the following parameters

```shell
git config --global core.packedGitLimit 64m
git config --global core.packedGitWindowSize 64m
git config --global pack.deltaCacheSize 64m
git config --global pack.packSizeLimit 64m
git config --global pack.windowMemory 64m
git config --global http.postBuffer 52428800 (= 50m)
git config --global pack.threads 1 (not mandatory)
```
