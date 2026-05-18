# 项目入门指南（写给全体队员）

> 协作工具 → AI 辅助 → 项目路线 → 第一阶段深挖 → 分工方式

---

## 零、项目启动会——介绍整体项目

### 是什么

> "我们做一个 3D 五子棋游戏。不是控制台跑的那种黑白方块，是 [Unity](#g-unity) 做的真 3D 棋盘，你能转视角看棋。玩家可以跟 AI 下——AI 有六个难度，从智障到高手。也可以两个人联机下。"
>
> 关键词：**3D、AI、联机**。

- "市面上的五子棋"：2D，AI 只有三个难度（本质同一套算法调参数），不能联机
- "我们的五子棋"：3D 棋盘，六个 AI 策略（算法内核不同，不是调参数），联机对战 + 观战

### 为什么

**为什么选五子棋**：规则简单（大一就能写），但策略深度足够——从菜鸟打分到高手搜索，算法强弱在棋盘上一目了然。它是研究博弈算法的完美载体。

**为什么用 [Unity](#g-unity) + [Python](#g-python) 两个语言**：Unity 做 3D 画面最方便，并且适配工作应聘需求。Python 做 AI 有现成库（easyAI、pymcts）。一个负责好看，一个负责聪明，各干各的最擅长的事。

**为什么我们做比自己做有价值**：这是一整年、五个人的团队项目。每个人在真实的工程架构里干活——有人写 3D、有人写 AI、有人做联机。最后交出去的不是一个课程作业，是一个能写进简历的项目经历。

### 怎么做

**技术架构——我们的系统怎么运转**：

整个系统分成两块：[前端](#g-frontend)（看得见的）和[后端](#g-backend)（看不见的脑子），中间用网络连着。下面用一个完整的"人 vs AI"场景串起来：

<a id="from-frontend"></a><a id="from-backend"></a>

**第一步：玩家在[前端](#g-frontend)落子**

玩家在 [Unity](#g-unity) 3D 棋盘上点了一个交叉点 (7,7)。Unity 做了三件事：

- 3D 场景层：在棋盘 (7,7) 位置渲染一颗黑子，播放粒子特效
- 逻辑层：把落子记录到本地棋盘数组 `board[7][7] = BLACK`，检查玩家是否连五获胜
- 还没赢 → [前端](#g-frontend)把棋盘状态打包成 [JSON](#g-json)：`{"board": [...],"last_move":{"r":7,"c":7},"mode":"pvp_ai","strategy":"mcts"}`

<a id="from-json"></a><a id="from-websocket"></a><a id="from-api"></a>

**第二步：数据通过网络传到[后端](#g-backend)**

这段 [JSON](#g-json) 通过 [WebSocket](#g-websocket) 发给 [Python](#g-python) 服务器。服务器收到后做了两件事：

- 在自己维护的这局棋盘上更新：`board[7][7] = BLACK`
- 判断模式是 `pvp_ai`，调用 MCTS 策略模块，传入当前棋盘
- MCTS 模块跑 5000 次模拟，算出最优落子 (8,8)

**第三步：AI 结果返回[前端](#g-frontend)**

[Python](#g-python) 把结果打包成 [JSON](#g-json)：`{"ai_move":{"r":8,"c":8},"score":5000,"candidates":[...],"time_ms":3420}`

[Unity](#g-unity) 收到后：在 (8,8) 渲染白子 → 更新本地棋盘 → 显示"AI 思考用时 3.4s" → 轮到你下。

> 上面这三步里，前后端之间传的 JSON 消息有固定格式——哪些字段、什么类型、谁发谁收。这套约定就是我们的 **[API](#g-api)**（应用程序接口）。你可以把它理解为"前后端对话的语言规则"：前端说 `{"board":..., "last_move":...}`，后端听懂、算完、回一句 `{"ai_move":...}`。谁不按约定来，对方就听不懂。

**三种对战模式的区别**：

| 模式 | 数据怎么流 |
|---|---|
| 人 vs AI | 玩家落子 → 发服务器 → AI 算完返回 → 显示 |
| 人 vs 人 | 玩家A落子 → 发服务器 → 服务器转发给玩家B → B 的棋盘更新 |
| AI vs AI | 服务器定时器触发 → 调策略A算 → 返回结果 → 调策略B算 → 返回，循环直到终局 |

**房间怎么管理**：

人在服务器上，每个房间是一个数据对象：

```
Room {
  room_id: "3742",           // 四位房间号
  board: [[0,0,1,...],...],  // 当前棋盘
  players: [A_连接, B_连接],  // 两个玩家的网络连接
  current_turn: BLACK,       // 轮到谁
  mode: "pvp",               // 对战模式
  spectators: [C_连接]        // 观战者（可多个）
}
```

玩家A点"创建房间" → 服务器生成这个对象，返回 `3742`。玩家B 点"加入房间"，输入 `3742` → 服务器把 B 挂到 `players[1]` → 双方开始下棋。每步棋都经过服务器验证再广播，保证两人看到的棋盘完全一致。

**观战怎么实现**：

C 点"观战"，输入 `3742` → 服务器把 C 挂到 `spectators[]`。之后每步棋的广播多加一份给 C——C 只能看棋盘更新，不能落子。代码上看就是服务器推送多一个接收方，不额外写逻辑。

**总览图**：

```
┌────────── Unity 前端 ──────────┐          ┌────── Python 后端 ─────────────┐
│                                │          │                               │
│  [3D 场景] 棋盘棋子灯光摄像机    │ JSON     │ [AI 模块]                     │
│  [逻辑层]  落子/胜负/悔棋        │←─────→   │  规则引擎 → Minimax → MCTS    │
│  [UI 层]   按钮/菜单/状态文字    │WebSocket │ [房间管理] 创建/加入/列表/观战  │
│                                │          │ [棋盘管理] 每房间一份状态       │
└────────────────────────────────┘          └───────────────────────────────┘
      ▲ 玩家A的exe     ▲ 玩家B的exe             ▲ 云服务器（阿里云/腾讯云）
```

> 一句话：[前端](#g-frontend)画画面、收操作，[后端](#g-backend)算 AI、管房间，中间 [JSON](#g-json) 来回传。

**分工方式**：不搞流水线——不搞"张三只做 3D、李四只做 UI"。每个阶段拆成十几个子任务，挂在 [GitHub](#g-github) Issues 上，两三人认领一个，下次换组合。最终每个人都摸过 3D、逻辑、UI，知识不偏食。

### 现在做什么

第一周：

1. 注册 [GitHub](#g-github) 账号，队长加你进仓库
2. 装 [Git](#g-git)，装 [VSCode](#g-vscode) Python 插件
3. 克隆仓库，在 README 后加一行你的名字，commit + push，验证能走通
4. 装 Unity Hub → [Unity](#g-unity) 编辑器

第二周：

1. Roll-a-Ball 教程（3 小时）
2. 用 Cube 拼 15×15 棋盘原型
3. 学基础 [C#](#g-csharp) 语法（变量、函数、if/for）

### 问答（2 分钟）

> "我什么都不会怎么办？"

- 不会 [C#](#g-csharp) → 第一周就学变量和 if，够用
- 不会 [Unity](#g-unity) → Roll-a-Ball 零基础 3 小时做完
- 数学系不会写代码 → 数据分析、胜率图表、结题报告、PPT。不需要 push 代码，但要能看 [GitHub](#g-github) 上的文档

---

## 前置：软件工程的基本流程

写一个小项目可以上来就写代码。但五个人的团队、一年的周期，不按流程来会乱——做完了发现做偏了、做完了不知道怎么测、测完了不知道怎么交。

软件工程原理就是教你这些流程的。我们用这套标准流程来管自己的项目，分五步：

### 1. 需求分析——搞清楚"到底要做什么"

先不要写一行代码。坐下来把需求写清楚。就拿我们项目举例：

- 用户打开程序 → 看到 3D 棋盘 → 点菜单选"人机对战" → 选难度 → 开始下棋 → 悔棋/认输 → 有人连五弹出胜利
- 用户打开程序 → 点"联机对战" → 创建房间 → 分享房间号 → 对方加入 → 开始下棋

每一个"用户怎么用"的路径都写下来。写完需求文档，你对要做什么就有底了。我们第一阶段的需求拆出来就是 14 个子任务（见第四节）。

### 2. 系统设计——搞清楚"怎么做"

需求说的是"要什么"，设计说的是"用什么技术实现、模块怎么划分"。

我们项目的设计就是：[Unity](#g-unity) 做[前端](#g-frontend)渲染 + [Python](#g-python) 做 AI [后端](#g-backend) + [WebSocket](#g-websocket) 做通信。一盘棋子拆成 3D 场景、游戏逻辑、UI 三层。设计阶段的关键产出是一张架构图。

### 3. 编码实现——动手写代码

你实际分到最多的就是这一步。但关键点是：**不是在 main 里写到一万行**。按照设计阶段的模块划分，一个模块一个模块地写，写一个测一个，不要攒到最后一起测——那时候报错多到修不完。

### 4. 测试——不是"跑起来没崩就行"

五子棋怎么测：

- **单元测试**：悔棋逻辑单独测一下——连走三步，悔两步，查棋盘是不是该回到第一步。胜负判定单独测一下——模拟活四、冲四、连五各种局面
- **集成测试**：[Unity](#g-unity) 发棋盘 [JSON](#g-json) 给 [Python](#g-python) → Python 返回坐标 → Unity 画棋子。这条链路打通了没
- **验收测试**：按需求文档里的那几条用户路径，一条条走一遍

### 5. 部署与交付——交出去

我们把 [Unity](#g-unity) 打包成 .exe，[Python](#g-python) 部署到云服务器，代码上传 [GitHub](#g-github) 开源，文档写好操作手册。结题时老师打开就能玩，不用配环境。

> **一句话贯穿**：需求说"要什么" → 设计说"怎么做" → 编码去做 → 测试验证 → 部署交付。每一步都有文档，每一步做完再进入下一步。

---

## 一、团队协作靠什么

一个团队写同一个项目，不能靠 U 盘拷来拷去。我们需要统一的工具链。

<a id="from-vscode"></a>

### 1. [VSCode](#g-vscode) + [Git](#g-git) + [GitHub](#g-github)：怎么写代码不打架 (+markdown语法)

**VSCode基础配置教程**
【VScode使用教程——更加全面，每个人都能看懂的VScode基础配置！安装、设置、运行代码以及各种问题的解决——大一新生必看！】 https://www.bilibili.com/video/BV15kavzWExk/?share_source=copy_web&vd_source=10b34eedb03cdd37052d6deba3516864

**VSCode优化插件**
【【教程】vscode优化体验篇（推荐设置 && 推荐插件）】 https://www.bilibili.com/video/BV1Hd4y1o7CN/?share_source=copy_web&vd_source=10b34eedb03cdd37052d6deba3516864

<a id="from-git"></a><a id="from-github"></a>

**[Git](#g-git)** 是装在你电脑上的版本控制工具，负责记录每次改动。
**[GitHub](#g-github)** 是网站，全球最大的代码管理平台。负责存代码、管任务、给队友看。

**日常操作流程（每个人都这样走）**：

```
早上开工：
  git pull              → 把队友昨晚的更新拉下来

开始写一个新功能：
  git checkout -b my-task  → 开一条自己的分支（不影响主代码）

写完了一段，保存：
  git add .               → 告诉 Git "这些文件我要提交"
  git commit -m "修复了悔棋bug"  → 拍一张快照，写上说明

收工，把今天的活上传：
  git push                → 推到 GitHub 上
  去 GitHub 网页上点 "Create Pull Request"  → 告诉队友"我改完了，来审一下"

队友审核通过，合并到主分支：
  git checkout main       → 切回主分支
  git pull                → 拉下来最新版本，接着写下一个任务
```

**要记住的核心原则**：
- 不要直接在 main 分支上写代码，每次新功能开一条 branch
- commit 说明用中文写清楚改了什么，方便自己和队友看
- 每天开工第一件事 pull，收工最后一件事 push

> 每次写完了可以用markdown语法，写一下复盘，你干了什么，做出来了什么功能，有什么想法，都可以加载branch里面

【给傻子的Git教程】 https://www.bilibili.com/video/BV1Hkr7YYEh8/?share_source=copy_web&vd_source=10b34eedb03cdd37052d6deba3516864

<a id="from-ssh"></a>

### 2. [SSH](#g-ssh) 要不要配？

**可以不配。** 我们用 HTTPS 地址就能 push/pull，不需要折腾 [SSH](#g-ssh) key。[SSH](#g-ssh) 是给熟练开发者省事的工具，对新手来说是额外的坑。以后熟了再配，现在可以不碰。但是如果大家以后还有搞项目的准备的话，建议学一下这种工具。

### 3. 不想敲命令？用 GitHub Desktop

上节 `git pull` / `git push` 是在[终端](#g-cli)里敲的。如果不想记命令，可以装 **GitHub Desktop**——一个带按钮的桌面软件，pull、commit、push 全做成按钮，点一下就行。clone 仓库时选 HTTPS 链接（`https://github.com/xxx/xxx.git`），push 时浏览器弹窗登录一次 [GitHub](#g-github) 账号，之后自动记住。

### 4. [VSCode](#g-vscode) 自带的 [Git](#g-git) 面板

[VSCode](#g-vscode) 左边栏有个**源代码管理**按钮，快捷键 `Ctrl+Shift+G`。你在 VSCode 里改了代码，点这个按钮 → 输入 commit 说明 → 点 ✓ 提交 → 点 "同步更改" 就 push 上去了。不用切窗口，不用开[终端](#g-cli)，全部在 VSCode 里完成。推荐大家用这个，比 GitHub Desktop 更轻量。

### 4.5 怎么把代码传到队长的仓库里？

**第一步：队长建仓库**

1. 打开 github.com → 登录 → 右上角 `+` → New repository
2. 仓库名填 `Gomoku3D`，勾选 `Add a README file`，点 Create
3. 进仓库 → Settings → Collaborators → Add people → 输入队友的 [GitHub](#g-github) 用户名 → 邀请
4. 队友在邮箱里点确认

**第二步：所有人把仓库拉到本地**

打开 [VSCode](#g-vscode) → `Ctrl+Shift+P` → 输入 `Git: Clone` → 粘贴仓库 HTTPS 链接（队长在 [GitHub](#g-github) 仓库页面点绿色 `Code` 按钮复制 `https://github.com/你的用户名/Gomoku3D.git`）→ 选一个本地文件夹 → 打开。

**第三步：日常改代码提交**

改了代码 → `Ctrl+Shift+G` 打开源代码管理 → 输入 commit 说明 → 点 ✓ → 点 "同步更改"。第一次 push 会弹窗让你登录 [GitHub](#g-github)，浏览器授权一次，之后自动记住。

> 队长建仓库加人 → 所有人 clone → 改代码 → commit → 同步。全程 VSCode 里完成。

下面有一个协作开发视频可以看一下：
【和傻子一起写代码】 https://www.bilibili.com/video/BV1udEuzrEa7/?share_source=copy_web&vd_source=10b34eedb03cdd37052d6deba3516864

### 5. [GitHub](#g-github) Issues：任务分配看板

我们把每个阶段拆成子任务，挂在 [GitHub](#g-github) 的 Issues 标签下。比如第一阶段拆出 14 个子任务（棋盘模型、棋子 [Prefab](#g-prefab)、落子逻辑……），每个 Issue 标题写清楚做什么，assign 给当轮负责的人，完成就 close。

Issue 标签用三种颜色：🟦 3D 场景 / 🟩 逻辑 / 🟨 UI，一眼就知道属于哪个方向。

### 6. [GitHub](#g-github) 还能干什么

除了存代码和管任务，[GitHub](#g-github) 是你的资源库：搜开源项目（关键词 + 按 Stars 排序）、看 README（作者的说明书）、搜代码片段（选 Code 标签）、看 Issues（别人踩过的坑）。

<a id="from-ide"></a>

### 7. [VSCode](#g-vscode)：统一 [IDE](#g-ide)

[IDE](#g-ide) = 写代码的软件，把"写代码 + 运行 + 报错提示"集成在一个窗口。就像 Word 写文档、PS 修图，IDE 专门写代码。我们选 [VSCode](#g-vscode)——轻量、插件多、什么语言都支持。不需要装 PyCharm 或 Visual Studio。

搭环境和搭 C/C++ 一样：装解释器（相当于 MinGW） + VSCode 里装插件。[Python](#g-python) 去 python.org 下载 3.12，勾选"Add Python to PATH"。VSCode 里装 Python 插件。装完[终端](#g-cli)敲 `python3 --version` 确认。

---

## 二、AI 编程助手

[VSCode](#g-vscode) 可以装 AI 插件。我们主要用 Claude Code，背后是四个概念：

<a id="from-model"></a><a id="from-prompt"></a><a id="from-agent"></a><a id="from-skill"></a>

**[Model](#g-model)（模型）**：AI 的大脑。GPT、Claude、DeepSeek 都是模型。喂它文字就吐文字。

**[Prompt](#g-prompt)（提示词）**：你问什么、AI 回什么。问得越具体，答得越精准。核心能力是提炼信息、清晰表达。

**[Agent](#g-agent)（智能体）**：模型 + 能操作电脑的工具。光聊天不够——Agent 能读代码、改文件、在[终端](#g-cli)跑编译。Claude Code、Cursor、[GitHub](#g-github) Copilot、codex 都是 Agent。

**[Skill](#g-skill)（技能）**：Agent 的预装工作流程。比如 `/grill-me` 就是一整套方案审问流程。队长接入了 Matt Pocock 开源的一套 Skill（github.com/mattpocock/skills），队员不需要配。

**浅谈在VScode里配置deepseek(也可以选择其他ai模型)**
【在VScode中使用Claude Code agent并配置DeepSeek v4 model【闲谈】】 https://www.bilibili.com/video/BV1ia9UBPESQ/?share_source=copy_web&vd_source=10b34eedb03cdd37052d6deba3516864

**好用的skill**
【为了不让AI瞎写代码，大神程序员把自己蒸馏了！GitHub星标6万+】 https://www.bilibili.com/video/BV1UpR9BBEf5/?share_source=copy_web&vd_source=10b34eedb03cdd37052d6deba3516864

---

## 三、项目分五步走

| 阶段 | 目标 | 产出 |
|---|---|---|
| 第一步 | 单机 3D 五子棋系统 | 能打开、能下棋、能判胜负的 exe |
| 第二步 | 接入 AI 对战功能 | 能跟 AI 对弈，AI 有多个难度等级 |
| 第三步 | 实现联网对战 | 两个人通过网络下同一盘棋 |
| 第四步 | 双端分离部署 | 服务器 + 客户端独立运行 |
| 第五步 | 整理材料 | 实验数据、结题报告、答辩 PPT |

---

### 前置概念：[前端](#g-frontend)和[后端](#g-backend)是什么

拿我们这个项目举例：

**[前端](#g-frontend)（客户端）** = 玩家电脑上双击打开的 [Unity](#g-unity) exe。负责画面、交互、按钮、3D 棋盘——你看得到摸得着的全是[前端](#g-frontend)。

**[后端](#g-backend)（服务端）** = 用 [Python](#g-python) 写的 AI 程序。玩家看不见它，它躲在远处（云服务器上）算 AI 该下哪步。[前端](#g-frontend)发棋盘数据给它，它算完返回落子坐标。

类比：你点外卖——[前端](#g-frontend)是手机上的美团 App，[后端](#g-backend)是商家后厨。你点的菜（棋盘数据）发给后厨，后厨做好（AI 算完）骑手送回给你（[Unity](#g-unity) 显示棋子）。

> 我们项目第一阶段只做[前端](#g-frontend)（单机本地），第二阶段开始写[后端](#g-backend)（AI），第三四阶段把前后端用网络连起来。

---

## 四、第一阶段：单机 3D 五子棋

<a id="from-unity"></a>

### [Unity](#g-unity) 是什么

[Unity](#g-unity) 是一个游戏引擎——帮你画 3D 画面、处理鼠标点击、播放动画，不用从零写底层代码。我们用它做 3D 五子棋。它既能处理**3D 场景**（立体棋盘棋子），也能用 [C#](#g-csharp) 写**客户端逻辑**（落子判定、胜负校验），还能搭**UI**（按钮、菜单、文字）。

### 三个关键词的关系

| 概念 | 是什么 | 例子 |
|---|---|---|
| **3D 场景** | 游戏世界里的立体东西 | 棋盘模型、棋子材质、灯光、阴影 |
| **客户端** | 游戏程序本身 | 落子规则、胜负判定、悔棋逻辑 |
| **UI** | 贴在屏幕上的平面图层 | 按钮（悔棋/认输）、手数显示、菜单 |

关系：3D 场景是内容，客户端是逻辑，UI 是操作面板。你转摄像机视角，棋盘跟着转，但 UI 按钮不动。三个图层叠在一起就是最终玩家看到的画面。

### 第一阶段的任务拆解

分析"单机 3D 五子棋"这个目标，拆出三个大方向，每个方向再细分成子任务：

**方向一：3D 场景搭建**

1. 棋盘模型——15×15 网格，木质材质
2. 棋子 [Prefab](#g-prefab)——黑白棋子预制体，带反光
3. 灯光环境——场景照明，让棋盘看起来立体
4. 摄像机——默认视角、鼠标滚轮缩放、拖拽旋转
5. 棋盘坐标标尺——棋盘上的网格线和星位标记

**方向二：游戏核心逻辑**

<a id="from-raycast"></a>

1. 落子规则——鼠标点击 → [射线检测](#g-raycast) → 吸附最近交叉点 → 落子
2. 胜负判定——每次落子后四个方向扫描，连五即胜
3. 悔棋逻辑——栈结构弹出上两步，恢复棋盘
4. 手数记录——显示当前第几手、轮到谁

**方向三：交互层 + UI**

1. 鼠标点击落子——[射线检测](#g-raycast) + 交叉点吸附
2. 按钮——悔棋、认输、重新开始
3. 状态显示——当前回合、手数、胜利提示
4. 菜单——主菜单、难度选择、模式切换

### 怎么分工

不做"张三只做 3D、李四只做 UI"那种模式——那样每个人只学了自己那一小块。我们是**一起学、一起做**。每个子任务两三个人一组去完成，下次任务换组合。最终每个人都摸过 3D、逻辑、UI，知识不偏食。

具体做法：把子任务挂到 [GitHub](#g-github) Issues 上，每两周认领一轮。

<a id="from-csharp"></a><a id="from-gameobject"></a><a id="from-prefab"></a>

### 前两周每个人要做什么

1. 学[GitHub](#g-github)基础知识，先实现协同在一个仓库里添加文件
2. 配置好[VSCode](#g-vscode)插件，中文 + Github用法 + claude code(选配，建议配一个) + Python系列插件(现在用不到，可以不着急) 
3. 下载 Unity Hub → 安装 [Unity](#g-unity) 编辑器
4. 完成官网 Roll-a-Ball 教程（3 小时），做完就懂 [GameObject](#g-gameobject)、[Prefab](#g-prefab)、脚本是什么
5. 在场景里用 Cube 拼出 15×15 网格阵列（棋盘原型）
6. 了解基础 [C#](#g-csharp) 语法：变量、函数、if/for(不需要精通，会用就行，先不学类和对象)


### 目录结构

```

Gomoku3D/
├── .gitignore                    # Unity + Python 忽略规则
├── .vscode/
│   └── settings.json             # 编辑器统一设置
├── Intro.md
├── LICENSE
├── README.md
│
├── docs/
│   ├── design/                   # 设计文档
│   └── notes/                    # 队员复盘
│
├── client/
│   └── Gomoku3D/
│       └── Assets/
│           ├── Scenes/           # 场景文件
│           ├── Scripts/
│           │   ├── GameLogic/    #   落子、胜负、悔棋
│           │   ├── UI/           #   按钮、菜单、状态
│           │   ├── Camera/       #   摄像机控制
│           │   └── Network/      #   网络客户端
│           ├── Prefabs/          # 棋子预制体
│           ├── Materials/        # 材质
│           ├── Models/           # 3D 模型
│           └── Textures/         # 贴图
│
└── server/
    ├── ai/                       # AI 引擎
    ├── network/                  # 网络服务
    ├── tests/                    # 测试
    └── requirements.txt          # Python 依赖

```

---

## 附录：术语速查

> 点击正文中任一 **高亮术语** 可跳转到此处的详细解释。点击条目末尾的 **↩** 可跳回原文位置。

---

### <a id="g-unity"></a>Unity

制作游戏和 3D 交互内容的跨平台引擎。你把模型拖进去、写脚本控制行为，Unity 帮你渲染画面、处理输入、打包成 exe/apk。我们用它做五子棋的 3D 棋盘、棋子和 UI。

> 类比：Unity 是 "游戏界的 PowerPoint"——拖拽组件、搭场景，不用从零写图形代码。

[↩ 返回](#from-unity)

---

### <a id="g-csharp"></a>C#

微软开发的面向对象编程语言，语法类似 Java。它是 Unity 的官方脚本语言——你在 Unity 里写的所有游戏逻辑代码都是 C#。我们用它写落子规则、胜负判定、UI 交互。

> 你不需要精通 C#。变量、函数、if/for 够写五子棋逻辑。

[↩ 返回](#from-csharp)

---

### <a id="g-python"></a>Python

语法简洁的解释型语言，拥有丰富的 AI/数据科学库（numpy、easyAI、pymcts）。在我们的项目里，Python 负责写 AI 对弈算法和网络服务端，不涉及画面渲染。

[↩ 返回](#from-websocket)

---

### <a id="g-json"></a>JSON

全称 JavaScript Object Notation，一种轻量级数据交换格式。长得像 Python 字典：

```json
{"board": [[0,0,1],[2,0,0]], "last_move": {"r":7, "c":7}}
```

我们的 [Unity](#g-unity) 和 [Python](#g-python) 用不同语言写，但它们都能读写 JSON——所以用 JSON 做"通用语言"，两端互传棋盘数据。

[↩ 返回](#from-json)

---

### <a id="g-websocket"></a>WebSocket

一种网络通信协议，特点是**建立连接后两端可以随时互发消息**，不需要每次传输都重新握手。对比 HTTP（一问一答、每次重新连接），WebSocket 适合需要实时双向通信的场景——比如网络对战五子棋，你下一步棋，对方要立刻看到。

> 类比：HTTP 是写信——寄一封等回信，过程慢。WebSocket 是打电话——接通后随时说随时听。

[↩ 返回](#from-websocket)

---

### <a id="g-api"></a>API

应用程序接口（Application Programming Interface）。两段程序之间"对话"的约定规则——A 发什么格式的数据给 B，B 回什么格式给 A，双方事先说好、照章办事。

在我们的项目里：前后端之间传的 [JSON](#g-json) 消息格式就是 API。比如前端必须发：

```json
{"board": [[0,0,1,...]], "last_move": {"r":7,"c":7}, "mode": "pvp_ai"}
```

后端收到后必须回：

```json
{"ai_move": {"r":8,"c":8}, "score": 5000, "time_ms": 3420}
```

谁不按这个格式来，对方就报错。这就是 API 的核心——**约定好了，照格式来**。

> 类比：API 是餐厅的点菜单。你按菜单格式写"红烧肉×1"，后厨就做红烧肉。你写"给我来盘好吃的"，后厨不知道你要啥。

[↩ 返回](#from-api)

---

### <a id="g-frontend"></a>前端（客户端）

用户直接看到和交互的部分。在我们的项目里：Unity 打包出来的 exe 就是前端——3D 棋盘、棋子动画、按钮菜单、鼠标点击落子，全是前端的事。

> 简记：前端 = 看得见、摸得着的。

[↩ 返回](#from-frontend)

---

### <a id="g-backend"></a>后端（服务端）

运行在远处服务器上、用户看不见的逻辑部分。在我们的项目里：[Python](#g-python) 程序跑在云服务器上，负责 AI 计算、房间管理、数据验证和中转。玩家电脑上的 exe 关掉了，后端还在跑。

> 简记：后端 = 看不见、算得多的。

[↩ 返回](#from-backend)

---

### <a id="g-ide"></a>IDE

集成开发环境（Integrated Development Environment）。把"代码编辑 + 自动补全 + 运行 + 调试器 + 报错提示 + 图形用户界面等等"集成在一个窗口的软件。就像 Word 写文档、PS 修图，IDE 专门用来写代码。

[↩ 返回](#from-ide)

---

### <a id="g-vscode"></a>VSCode

微软出品的轻量级代码编辑器，通过装插件变成全功能 [IDE](#g-ide)。免费、启动快、支持几乎所有语言。我们团队统一用它写 C# 和 Python。

[↩ 返回](#from-vscode)

---

### <a id="g-git"></a>Git

分布式版本控制系统。它记录你每次"保存"（commit），让你能回到任意历史版本、开分支实验新功能而不破坏主代码、多人协作时自动合并改动。

> 类比：Git 是"游戏存档"——打 Boss 前存个档，打输了读档重来。

[↩ 返回](#from-git)

---

### <a id="g-github"></a>GitHub

基于 [Git](#g-git) 的代码托管网站，全球最大的程序员社区。功能：存代码（云端备份）、管任务（Issues 看板）、审代码（Pull Request）、搜开源项目。我们的项目仓库就托管在 GitHub 上。

[↩ 返回](#from-github)

---

### <a id="g-ssh"></a>SSH

安全外壳协议（Secure Shell），一种加密的网络连接方式。在 Git 场景下，配置 SSH key 后 push/pull 不需要反复输入密码。缺点是配置步骤多，新手容易踩坑——我们用 HTTPS 代替，暂时不碰。

[↩ 返回](#from-ssh)

---

### <a id="g-cli"></a>终端 / CLI

命令行界面（Command-Line Interface）。一个黑底白字的窗口，通过敲命令来操作电脑（而不是点鼠标）。VSCode 自带内置终端，我们用它敲 `git pull`、`python main.py` 等命令。

[↩ 返回](#from-ide)

---

### <a id="g-model"></a>Model（AI 模型）

AI 的"大脑"。GPT-4、Claude、DeepSeek 都是模型。你输入文字（Prompt），它输出文字。模型的能力取决于训练数据和参数规模——越大越聪明，但也越慢越贵。

[↩ 返回](#from-model)

---

### <a id="g-prompt"></a>Prompt（提示词）

你发给 AI 模型的那段文字。Prompt 的质量决定回答的质量："写一个五子棋 AI" → 回答泛泛。"用 Minimax 算法写一个五子棋 AI，15×15 棋盘，搜索深度 4，输出 C# 代码" → 回答精准可用。

> 写好 Prompt 的核心能力：想清楚你到底要什么，然后精准描述。

[↩ 返回](#from-prompt)

---

### <a id="g-agent"></a>Agent（AI 智能体）

[Model](#g-model) + 工具 = Agent。光聊天不够——Agent 能读你的代码文件、修改代码、在[终端](#g-cli)跑编译和测试。Claude Code、Cursor、GitHub Copilot 都是 Agent。你给它任务，它自己去查文件、写代码、跑测试，最后告诉你结果。

> Agent = AI 大脑 + 双手（能操作电脑）。

[↩ 返回](#from-agent)

---

### <a id="g-skill"></a>Skill（技能）

[Agent](#g-agent) 的预装工作流程。比如 `/grill-me` 就是一整套方案审问流程——Agent 按固定剧本一步步质问你、帮你完善设计。Skill 把"每次都重复说的那套话"固化成一个命令。

[↩ 返回](#from-skill)

---

### <a id="g-gameobject"></a>GameObject

[Unity](#g-unity) 里一切东西的基础容器。棋盘、棋子、灯光、摄像机，全是 GameObject。它本身是空的，往上面挂组件（模型、脚本、碰撞器）才变成具体的东西。

> 类比：GameObject 是"积木底座"，上面插什么组件就变成什么。

[↩ 返回](#from-gameobject)

---

### <a id="g-prefab"></a>Prefab（预制体）

[Unity](#g-unity) 里可复用的 [GameObject](#g-gameobject) 模板。比如你做好一颗黑子的外观、材质、碰撞体，存成 Prefab。之后要生成 225 颗棋子？不用一个个做——脚本一行 `Instantiate(blackPiecePrefab)` 就行。改 Prefab 一处，所有实例自动更新。

[↩ 返回](#from-prefab)

---

### <a id="g-raycast"></a>射线检测（Raycast）

从鼠标位置向 3D 场景发射一条看不见的"射线"，碰到哪个物体就返回那个物体的信息。我们用它实现"鼠标点击棋盘 → 检测点在哪条线交叉 → 在那个位置落子"。

> 类比：射线检测 = 激光笔。你指哪里，程序就知道你指了什么。

[↩ 返回](#from-raycast)
