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

# 三：理解Git

在计算机的世界里，考虑问题一般有两种思路。一种是自顶向下，一种是自下向上。个人观点：如果是做应用，那么自顶向下的思路非常好，如果是做研究，那么自下向上的思路会更好。今天我们从更底层的角度去考虑Git。  

## Git's data model
```
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | blob>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```
## Objects and content-addressing
```
type object = blob | tree | commit

objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```
## 例子
我们可以使用`git cat-file -p <hash>`去理解：**Git's data model** and **Objects and content-addressing**  

```shell
$git cat-file -p 6dcc7249a05b18591354926bfa7259b9f03baa7f
tree 19421a887b2ce8cdd1e18e79ccd970da223a738a
parent c28082e80d7262f20f61aa5d17df71451ff26cb4
author demo-zhang <demo-zhang@qq.com> 1706843273 +0800
committer demo-zhang <demo-zhang@qq.com> 1709125816 +0800
[update] 修改~

[笔记] 性能测试
$git cat-file -p 19421a887b2ce8cdd1e18e79ccd970da223a738a
040000 tree 22b768bcfa6ace79bc888dffca09bad950b74929    .github
100644 blob c8677bde9b5b228af47f4451bbf2f621239065bd    .gitignore
100644 blob 49f46de3ebf96c3d189f7e7f3dd469477c3ea36e    .gitmodules
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    _config.landscape.yml
100644 blob 86ac462512e3128e001ad2a0dcfcde2b8e5b1ecb    _config.yml
100644 blob e290158a9fd84184e0bbb4f9942a6aa2116fdb7d    package-lock.json
100644 blob 79d10fdaa92d541429bf3f8d9f0084612a3a0f4c    package.json
040000 tree fadf7ab7818d6053c708c5133a560cb3b4759281    scaffolds
040000 tree d212beb8a5145b3c52b0d55770947b1e1072f8d2    source
040000 tree 0ef98a955dee3dc7da1ef212a39fe91443dd7622    themes

$git cat-file -p d212beb8a5145b3c52b0d55770947b1e1072f8d2
040000 tree 8fcd384c88c75735dc52d942b8455a1ac83e7f0a    _posts
040000 tree 0f6f9b7f175ff6374ccebdb9743f641012a20f20    images

$git cat-file -p 8fcd384c88c75735dc52d942b8455a1ac83e7f0a
...

```