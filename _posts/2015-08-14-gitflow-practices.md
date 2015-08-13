---
layout: post
title: "Gitflow 在客户端开发中的实践"
description: ""
category: 
tags: [gitflow practices]
---
{% include JB/setup %}

       为了规范开发流程，当存在多次预发布情况时，不至于因代码不同步，遗漏Bug修复。并让流程更清晰，避免出现多个混淆分支。所以将实践方案提出供大家一起讨论。

### 先看看标准的 Gitflow 流程

![552844C8_FCF4_49CE_8A78_2A12865A9DAF](http://img4.tbcdn.cn/L1/461/1/5319ce74d44bc3c59ea05fd47aebfff9a867563c)

---

#### 原则 

* 新建feature、bugfix , release，全部用 git flow start 或 SourceTree 右上角的 Gitflow 流程
* feature, Bugfix 命名规范：feature/用户名/功能名 ,  bugfix/用户名/修复点
* 完成feature 或 Bugfix 时，用 git flow  finish 或 SourceTree 右上角的 Gitflow 流程
* BugFix 仅在 release 分支上进行，HotFix 仅在 master 分支上进行

---

#### 关于Code Review

* 如果在使用Gitlab 管理代码仓库时，可以使用 Gitlab 自带的 Merge Request 工具进行代码 Review 申请
* ![_2015_08_14_3_20_33](http://img4.tbcdn.cn/L1/461/1/8749eaaedcbf84fbe1318cb0746568af8bac4792)
* 创建 Merge Request ，并指定从哪个分支合并到哪个分支，并指定到Review 人
* Review 者仔细审核过代码后，如果没有疑问，切到对应的来源分支，进行上述的 git flow 的 finish 操作。
* 如果只是 feature -> develop ，可以直接点 Accept Merge Request

--- 

### 代码管理实践

![Gitflow_1](http://img2.tbcdn.cn/L1/461/1/01a46faf75eb853e8e69d856f4d53c86bfeb0691)

`Hotfix 后，会有版本号后加上一个小版本号，表示有一个 hot fix 修复. x.x.x.1`

***当有多个并行预发布版本正在进行时，需要注意的是***

- `Hotfix 在完成后，除了执行标准Gitflow 流程，还需同时合到正在并行的 release ​分支`

### 更多资料参考

- [Github flow](https://guides.github.com/introduction/flow/)
- [Gitlab flow](https://about.gitlab.com/2014/09/29/gitlab-flow/)