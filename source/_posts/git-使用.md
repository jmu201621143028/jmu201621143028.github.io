---
title: git 使用
date: 2024-03-11 16:58:14
tags:
---
# 一：提交代码流程

1. 同步最新代码
```shell
git fetch
git merge --ff-only
```
2. 创建新分支 version_{bug-id}
```shell
git checkout -b 8.2-PBT-1234
```
3. 修改代码...
4. 添加
5. 提交
```shell
git commit
```
6. 推送远程仓库
```
git push origin 8.2-PBT-1234
```
7. code review
8. 删除分支
```
git branch -D 8.2-PBT-1234
```
9. 同步其他分支子模块更新后,主模块需要更新添加子模块
```
git add iso-sources/kylin-v10
```
# 二： 特殊处理
## 模块管理
```
vim .gitmodules  # view git modules
git submodule update --init bt-ganesha/src/nfs-ganesha # 拉取子模块的代码
```

## 修改commit日志
```
git commit --amend
```
## 强推
```
git push origin 8.2-PBT-5703 --force
```
## 合并提交
```
git rebase -i HEAD~2 # 记得选s.
```
## 提交后测试不通过
```
git rebase origin/virtualstor-8.3
```
## cherry-pick
```
git cherry-pick <commitHash>
git cherry-pick <branch>
```
## 同步主分支
```
git fetch
git rebase origin/scaler/8.3/12.2.13/master
git push origin PBT-5804 --force
```
## 回退到指定位置
```
git reset --hard hash
```
## 主项目合并子项目
```
cd ./主项目路径
git fetch
git merge --ff-only
git submodule update # 非常重要
cd ./子项目路径
git checkout 子项目的分支
git fetch
git merge --ff-only
git add 子项目
git commit # [子项目名] message, for example: [ezs3-ceph] add s3 tier related
commit
git checkout -b 8.2-s3-tier-fix-v3
git push origin 8.2-s3-tier-fix-v3
```