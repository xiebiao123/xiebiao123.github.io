---
title: Git
date: 2019-09-10
categories:
    - 学习
tags:
    - git
---

### 常用命令

* 本地一直提示没有权限或账户密码错误
  
``` shell
git config --system --unset credential.helper
```

* 删除保存在本地的git账户
  
``` shell
git credential-manager uninstall
```

* 本地仓库与远程仓库关联
  
``` shell
# 创建本地仓库
git init
git add *
git commit -m "init test"

# 创建远程仓库

# 关联远程仓库
git remote add origin https://仓库地址
git push origin master
```
