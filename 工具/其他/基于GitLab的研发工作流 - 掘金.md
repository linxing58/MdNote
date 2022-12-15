# 基于GitLab的研发工作流 - 掘金
> 本文工作流模式，是我担任`LIZI UI Design`团队 Leader 时，基于 GitLab 的工具集，创建的一套标准的研发工作流。当前文档是对这套工作流的拆解和说明。

## 背景

由于团队成员分属不同业务线，日常碰面、交流的机会比较少，不能用早会、日报等普通的项目管理方式，对项目研发进度进行把控，所以需要一种全新的管理模式。

主要的痛点有：

1.  项目的研发目标、里程碑不明确
2.  任务的分解不清晰
3.  团队成员之间无法获知对方目前的研发状态
4.  团队成员之间协作，缺乏信息记录

基于以上痛点，选择了 GitLab 提供的工具集，来一一解决。

接下来，开始这套工作流的讲解。

## 关键要素说明

### 确定项目的里程碑

小组成员，通过开会讨论，确定一个周期内的目标，包括有哪些交付物，需要的研发时长，再由此反推，确定好相应的里程碑。

进入 GitLab 的小组项目（以后的语境，均在此项目下，后续不进行累述），打开 Milestones 进行里程碑设置。

如下图所示，设置

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43aaedb7d278409e882dc08f74b6093a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

-   里程碑代号（Title）
-   工作内容描述（Description）
-   开始日期（Start Date）
-   截止日期（Due Date）

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/656b875e22b44c2da99a1e21fe6d52cf~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### 确定信息标签

#### 标签分类

主要的信息标签类别有：

-   **待准入**
-   **待认领**
-   **Doing**
-   **待 review**
-   **待关闭**
-   **developer: 名字**
-   **reviewer: 名字**

#### 设置

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cdf08d274ab04469b0726cc530878560~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

进入 Labels 设置页面，点击 New Label 按钮进行创建即可。

### 看板的设置

看板的作用是可以清晰地、透明化地体现当前项目的进度情况和研发人员的工作状态。

#### 阶段说明

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b41791f9876479c9d3928e705f57c15~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

如图所示，看板的阶段前后顺序依次为：

1.  Open （默认的打开阶段）
2.  待准入
3.  待认领
4.  Doing
5.  待 Review
6.  待关闭
7.  Closed　（默认的关闭阶段）

#### 设置

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39297c6fb4804b498dad54cf7245e0f6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

进入 Boards 界面，可以选择 Create new board 创建新的看板。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/214fb476d6c342019c0f82ee04f4dc7c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

并通过 Add List 按钮，添加新的看板阶段。

### Issue 的设置

#### Issue 说明

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a57f563ab1894c1f80a0fe8470473ec6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

如上图所示，Issue 的信息有：

-   任务名称（Title）
-   任务的工作内容（Description）
-   任务的执行者（Assignee）
-   任务所属的里程碑（Milestone）
-   任务当前的状态（Labels）
-   任务的截止日期（Due date）

其中，Assigness 的设置，可以在该 Issue 改变的时候，让任务执行者及时收到邮件通知。

另外，在 Description 中输入　/estimate（估时）　和　/spend（耗时） 进行工时的记录。

#### 任务评估时长示例

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a92c143ee7942a4841605874d133cd0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

如图，输入估时后，在 Issue 的信息面板中，Time tracking 会有信息的展示。

#### 任务消耗时长示例

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d80919e481644e39a417d12a65b62fea~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

如图，输入任务耗时后，在 Issue 的信息面板中，Time tracking 会根据任务估时和当前的消耗时长，进行百分比的进度展示。

## 任务工作流讲解

### 初始任务

通过 New Issue 的方式，将任务的信息记录到 Issue 中，并打上信息标签待准入。

经过团队成员确认后（可通过开会的方式，集中确认），将该 Issue，通过看板面，移动到待认领阶段。

并且，为该 Issue 打上 Milestone（里程碑）的标记，正式确认当前任务所属的里程碑。

### 认领任务

在认领阶段的任务，团队成员只要将该 Issue 打上 developer：名字的标签，即可完成认领。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/206e26052a904da7b02fd8e33cda8fa2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### 确定任务的工时信息和开发周期

需要操作以下 3 个步骤：

1.  设置 Assignee 为自己，方便接收邮件提醒
2.  设置 Due date（截止日期），大概预估任务的完成日期
3.  通过 Description 评论，设置任务的估时和耗时

如下图所示，操作完成后的 Issue 面板信息。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7159af86cc93469e86162620c8393d4a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

另外，如果想要利用 GitLab 的 To Do 功能，可以在该 Issue 面板的最上方，点击 Add To Do 按钮，即可。

### 任务状态改变

任务状态的改变，都是通过看板，对 Issue 进行移动，来完成更改的。

比如，当前的任务正在编码中，就将 Issue 移动到 Doing 阶段。

具体如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3902f7b2469f45c8a45ef8d868232429~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### Review 阶段

#### 改变 Issue 状态

当任务编码完成后，Issue 移动到待 Review 阶段。

这个时候，可以由 Issue 的研发者去找团队其他成员进行代码 review，也可以由其他成员，主动进行 review。

review 的人员，需要将该 Issue 打上　reviewer: 名字　的标签，表示自己负责当前任务的 review。

打标签后，如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/814f95493dfe419cb3ca7282752f9445~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

#### 提交 MR

另外，需要研发者发起 Merge Request 请求，将 feature-\*\*\*分支的代码，合并到对应的 dev-\*\*\*分支。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00ac981b6f2148caa2d7d886590be74f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

如上图所示，需要做以下几件事：

-   确定好要合并的分支
-   填写合并的信息（Title）
-   在 Description 中，输入 #，会有相应的提示，选择对应的 Issue，关联到当前的 MR

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d826b7dc96d14007a0c104d720179749~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

-   设置好 Assignee 和 Milestone

### 代码分支合并

负责 review 的人员，在 review 完代码后，对 MR 请求进行 resolve，合并进 release 分支。

然后，将看板中的 Issue 移动到待关闭阶段。

> 需要注意的地方：当 feature 分支的代码，与要合并的 dev 分支产生冲突时，需要研发在本地解决冲突，并 push 到远程的 feature 分支，才可以合并。

### 版本发布

所有当前里程碑的任务完成后，将 `dev-***`分支的代码合并到 dev 主分支，再由 dev 分支合并到 master 分支和 release 分支，并进行版本发布操作。

然后，将看板中，待关闭阶段的 Issue 移动到 Closed 阶段。 
 [https://juejin.cn/post/7073399615579979813](https://juejin.cn/post/7073399615579979813)
