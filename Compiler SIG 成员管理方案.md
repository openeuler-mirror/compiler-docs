# Compiler SIG 成员管理方案

本文简要描述了Compiler SIG 成员管理方案，包括：贡献者角色的权责划分，Maintainer与Committer的进入和退出机制等。


## 1、SIG成员角色描述及职责

### 1.1、Compiler SIG成员角色划分：

|角色| 职责范围（简要描述）|
|:-----------|:-------------------- |
| Developer | 社区最广泛的代码贡献者；在社区长期贡献可自荐或被推荐为仓库Contributor，甚至Committer；|
| Contributor| SIG组部分仓库的重要贡献者，也是这部分仓库Committer的后备力量；通常也是这部分代码仓库问题主要修复者和代码开发者；|
| Committer| SIG组部分仓库的看护者；作为这部分仓库的第一责任人；Committer也是SIG组Maintainer的后备力量；Committer 配置需要在sig-info.yaml中指定仓库的，权限覆盖yaml中明确的部分代码仓，权限默认不会扩展；|
| Maintainer| 定位是SIG组的牵引者、规划者，努力做好SIG组的发展和演进；负责定期组织SIG组会议、并代表SIG组向社区展示SIG组情况；Maintainer 是按sig组指定的，权限覆盖sig的所有代码仓，所有增加到sig的代码仓都默认获得权限；|
| RepoAdmin | 社区部分自研代码仓库需要git push权限，会对这些仓库设置Admin权限；通常是SIG组maintainer兼任，对这部分仓库的代码合入全量负责；|

### 1.2、SIG组成员权责划分

根据不同角色，在社区（特别是在gitee代码托管平台）承担不同的权责：

| 角色 | sig_info.yaml中是否记录 | 代码提交 | 通过评论approve合入 | 直接git push合入 |
|:-----|:------|:------|:-----|:-----|
| Developer | NO | YES | NO | NO |
| Contributor | YES | YES | NO | NO |
| Committer | YES | YES | YES[指定仓库] | NO |
| Maintainer | YES | YES | YES[sig组所有仓库] | NO |
| RepoAdmin | YES | YES | YES[指定仓库] | YES[指定仓库] |


### 1.3、审核者 Committer

#### 1.3.1、要求

+ 作为Compiler SIG贡献者至少3个月
+ 审阅或合并至少6个基本PR到代码库
+ 熟悉代码库
+ 可以自我提名，或由该SIG的审核者或维护者提名

#### 1.3.2、责任与权力

+  **评审PR**：对负责仓库的所有提交的PR完成评审，评审可以参考社区的[编程建议]()和[安全编程规范]()；评估负责仓库的新合入MR是否需要同步到不同分支。
+  **分发处理问题**:请参考“[问题处理流程]()“。
+  **跟踪依赖性问题**：在开发分支中，其他SIG组的软件包的更新可能会到导致破坏本SIG内软件包的依赖关系。此时Committer会收到告警提示，Committer应尽力重建软件包。依赖关系出错可能会使最终用户无法更新系统，打包团队也会介入并重建存在依赖性问题的软件包，但Maintainer不应依赖这些重建。
+  **如有接口变更，通知可能会影响到的SIG**：其他SIG或项目会依赖本SIG的软件包，对软件包接口的变更可能会对他们造成影响。Maintainer应了解并评审&决策变更造成的依赖影响，并公告和发送API或ABI变更的告警邮件。这类公告应在变更发生至少一周前完成，并应通知到所有可能受影响的SIG。具体请参考[接口变更通知流程]()。
+  **更新和维护软件包版本**：遵守社区的[软件包更新质量控制策略]()完成软件包的更新。
+  **和上游社区合作**，包括：
   +    将所有变更推送到上游社区
   +    参与上游社区邮件列表
   +    获取上游社区的bug跟踪器的账户，并跟踪上游社区的重要bug
   +    将严重的错误转发给上游社区以寻求帮助
         更多信息，请参考“[上游社区软件包管理建议]()”
+  **和测试团队合作**，包括：
   +  在提交软件包时，向质量检查人员提供如何调试/分类软件包的信息，以供问题的分类
   +  提供基本功能的测试用例，用于测试回归
   +  提交软件包更新时，提供有关更新中已经修复问题的测试用例，以供质量检查人员使用



### 1.4、 维护者 Maintainer

#### 1.4.1、 要求

+ 作为审核者至少3个月
+ 作为主要审阅者至少参与了12次PR的审阅
+ 审阅或合并至少30个基本PR到代码库
+ 熟悉代码库
+ 可以自我提名，也可以由子项目Maintainer提名，并且没有其他子项目Maintainer的反对

### 1.4.2、责任与权力

- **确定SIG所负责项目的技术路线**：包括规划和决策SIG技术方向、路标规划、架构演进
- **制定SIG所负责项目的发布计划**：确定SIG的关键需求和发布计划；参与社区的PM活动，并协调SIG计划和社区版本的里程碑时间表匹配
- **参与社区协调活动**：作为SIG的代表参与openEuler技术委员会或理事会组织的活动和特定会议等
- **召集SIG组会议**：定期召集SIG会议，决策SIG内上升的争议
- **制定SIG组运作机制/章程**：制定 SIG 的运作机制/章程、开发/维护规范、漏洞响应规范等
- **协调和指导工作**：协调和指导 Committer 的工作，规划各领域的版本与特性


## 2、SIG成员的产生与退出
### 2.1、Committer的角色设置
1、Committer 根据需要设置，暂不限制总人数，但相同仓库的 Committer 数量不宜超过**2**人。

### 2.2、Committer的产生
* 情形一：首任Committer由仓库发起者担任
* 情形二：开发者可以向 SIG 会议申请成为某个或多个仓库的Committer。经 SIG 会议评审通过，可以成
为 Committer。

满足下面条件之一，优先担任相应模块的 Committer：
* （1）模块或子系统的作者，或者作者所在公司指定开发维护人员。
* （2）上游社区对应模块的 Maintainer。
* （3）现任或担任过 openEuler 社区其他SIG的Committer。

如出现如上情形，SIG 会议决策通过，可以成为Committer。

### 2.3、Committer的退出
* 情形一：现任 Committer 向 SIG 组申请不再担任
* 情形二：现任 Committer 连续**6个月**没有参与社区活动

如出现如上情形，经 Maintainer 提出，SIG 会议决策通过，可以取消其 Committer 资格。

### 2.4、Maintainer的角色设置
* 1、Maintainer 数量一般不超过7人。
* 2、Maintainer 中包含一位常任 Maintainer，代表 Compiler SIG 参与技术委员会，并向技术委员会汇报。

### 2.5、Maintainer的产生
* 情形一：首任常任 Maintainer 由项目发起方产生。常任 Maintainer 申请不再担任时，可以提名一位继
任者，并报 TC 决策批准，继任者必须是现任 Maintainer。
* 情形二：现任 Maintainer 可以提名 Maintainer 候选人，候选人一般为SIG组内Committer。
* 情形三：现任 Maintainer 提出不再担任时，可以同时提名一位 Maintainer 候选人继任，候选人一般为SIG组内Committer。

如出现如上情形，SIG 会议决策通过，可以成为Maintainer。

### 2.6、Maintainer的退出
* 情形一：现任 Maintainer 向 SIG 组申请不再担任。
* 情形二：现任 Maintainer 连续**3个月**未参与 SIG 组工作。
* 情形三：不能胜任 Maintainer 的工作。

如出现如上情形，经 Maintainer 提出，经 SIG 会议决策，可以取消其 Maintainer 的资格。


# 参考
* [openEuler项目群开源治理制度](https://www.openeuler.org/zh/community/charter/)
* [openEuler项目群选举管理条例](https://www.openeuler.org/zh/community/vote/)
* [openEuler社区行为准则](https://www.openeuler.org/zh/community/conduct/)
* [openEule社区角色说明](https://www.openeuler.org/zh/sig/role-description/)
* [openEuler社区SIG组成员管理方案](https://www.openeuler.org/zh/blog/georgecao/openEuler-sig-member-management.html)
* [openEuler版本分支维护规范](https://atomgit.com/openeuler/release-management/blob/master/Goverance/openEuler%E7%89%88%E6%9C%AC%E5%88%86%E6%94%AF%E7%BB%B4%E6%8A%A4%E8%A7%84%E8%8C%83.md)
