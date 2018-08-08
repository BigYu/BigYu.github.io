---
title: 如何配置github ssh连接
date: 2017-01-19 16:24:06
tags:
---

## 测试ssh连接是否正常

```bash
ssh -T git@coding.net
```

## 配置ssh连接

### 1. 打开gitbash

### 2. 生成public key

用以下命令 邮箱换成自己的

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

一路回车就行了

### 3. 找到生成的public key添加到github

首先找到公钥文件id_rsa.pub，默认目录为C:/Users/[username]/.ssh，用文本编辑器打开并复制

[然后参考这里](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)


### 到这里就ok了

可以找repo pull/push一下试一试了

