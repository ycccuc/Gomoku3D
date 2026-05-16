# 项目入门指南（写给全体队员）

> 协作工具 → AI 辅助 → 项目路线 → 第一阶段深挖 → 分工方式

## 零、介绍整体项目

### 是什么

> 做一个3D五子棋带有多维度多策略的AI算法的小游戏，支持双端

*核心亮点：*
1. **3D** -- 基于Unity来开发3D环境，为什么选Unity -- 招聘的时候，基本都是要学会Unity，像什么Unreal（虚幻游戏开发引擎 -- 3A），其他的引擎要么太小了，要么就是招聘没多少要求

> 同级的项目一般都是做软件 我们是写出来别人没有的3D 
> 流程--> 找具体问题(生活/社会/前沿) --> 进行针对性的开发软件
> 科研流程 --> 选题 --> 去找知名文章 --> 根据目前已有成果，在这个基础上面，找到一些你想出来的新点子，根据新点子再进行针对性的科研 --> 根据你这个方向，来泛读几篇相关文章 

2. **联机** -- Websockt -- 市面上的五子棋软件，只能支持随机联机，还没做出来自定义开房间

3. **AI算法** -- 基于多策略AI人机博弈算法的五子棋系统，核心点是多种算法，像什么阿尔法狗AI，其实本质也是算法，加上神经网络这种东西来训练他



### 为什么

因为市面上的五子棋游戏，UI界面一坨，联机功能不全，特别是算法，他的算法难度梯度其实没变，只优化了效率，核心算法没改。

### 怎么做

**整体架构**

```

Untiy (客户端 + 脚本C#)
  |---3D场景  (直接用Unity写)
  |---交互逻辑(本质就是C#写脚本)
  |---UI/按钮 (审美 + 素材)


Python (后端 网上的算法库就是Python写的)
  |---AI算法  (网上的算法库)
  |---房间管理(解决自定义开房间问题)
  |---游戏框架(自动保存棋盘状态，自动对局)

```

**问答**

> “什么都不会怎么办？”

- 其实我也不会

> 不会 Unity 和 C#？

- 学 Roll-a-Ball 大概几个小时
- 暂时用不到C#语言，建议学个基础(if/else | while-for)

> 基础语法都是互通的，只是说每个语言都有每个语言独特的地方，比如说C偏向底层，是“中级语言”，C++，Java偏向于工程师，面向对象的思想，Python也是C语言写的，他的特点就是有很多库，其实他算一个工具语言，专门用来爬数据之类的

---

## 一、团队协作靠什么

一个团队写同一个项目，不能靠 U 盘拷来拷去。我们需要统一的工具链。

### 1. VSCode向 Git + GitHub：怎么写代码不打架 (+markdown语法)

**VSCode基础配置教程**
【VScode使用教程——更加全面，每个人都能看懂的VScode基础配置！安装、设置、运行代码以及各种问题的解决——大一新生必看！】 https://www.bilibili.com/video/BV15kavzWExk/?share_source=copy_web&vd_source=10b34eedb03cdd37052d6deba3516864

**VSCode优化插件**
【【教程】vscode优化体验篇（推荐设置 && 推荐插件）】 https://www.bilibili.com/video/BV1Hd4y1o7CN/?share_source=copy_web&vd_source=10b34eedb03cdd37052d6deba3516864


**Git** 是装在你电脑上的版本控制工具，负责记录每次改动。
**GitHub** 是网站，全球最大的代码管理平台。负责存代码、管任务、给队友看。

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

### 2. SSH 要不要配？

**可以不配。** 我们用 HTTPS 地址就能 push/pull，不需要折腾 SSH key。SSH 是给熟练开发者省事的工具，对新手来说是额外的坑。以后熟了再配，现在可以不碰。但是如果大家以后还有搞项目的准备的话，建议学一下这种工具。

### 3. 不想敲命令？用 GitHub Desktop

上节 `git pull` / `git push` 是在终端里敲的。如果不想记命令，可以装 **GitHub Desktop**——一个带按钮的桌面软件，pull、commit、push 全做成按钮，点一下就行。clone 仓库时选 HTTPS 链接（`https://github.com/xxx/xxx.git`），push 时浏览器弹窗登录一次 GitHub 账号，之后自动记住。

### 4. VSCode 自带的 Git 面板

VSCode 左边栏有个**源代码管理**按钮，快捷键 `Ctrl+Shift+G`。你在 VSCode 里改了代码，点这个按钮 → 输入 commit 说明 → 点 ✓ 提交 → 点 "同步更改" 就 push 上去了。不用切窗口，不用开终端，全部在 VSCode 里完成。推荐大家用这个，比 GitHub Desktop 更轻量。

### 4.5 怎么把代码传到队长的仓库里？

**第一步：队长建仓库**

1. 打开 github.com → 登录 → 右上角 `+` → New repository
2. 仓库名填 `Gomoku3D`，勾选 `Add a README file`，点 Create
3. 进仓库 → Settings → Collaborators → Add people → 输入队友的 GitHub 用户名 → 邀请
4. 队友在邮箱里点确认

**第二步：所有人把仓库拉到本地**

打开 VSCode → `Ctrl+Shift+P` → 输入 `Git: Clone` → 粘贴仓库 HTTPS 链接（队长在 GitHub 仓库页面点绿色 `Code` 按钮复制 `https://github.com/你的用户名/Gomoku3D.git`）→ 选一个本地文件夹 → 打开。

**第三步：日常改代码提交**

改了代码 → `Ctrl+Shift+G` 打开源代码管理 → 输入 commit 说明 → 点 ✓ → 点 "同步更改"。第一次 push 会弹窗让你登录 GitHub，浏览器授权一次，之后自动记住。

> 队长建仓库加人 → 所有人 clone → 改代码 → commit → 同步。全程 VSCode 里完成。

下面有一个协作开发视频可以看一下：
【和傻子一起写代码】 https://www.bilibili.com/video/BV1udEuzrEa7/?share_source=copy_web&vd_source=10b34eedb03cdd37052d6deba3516864

### 5. GitHub Issues：任务分配看板

我们把每个阶段拆成子任务，挂在 GitHub 的 Issues 标签下。比如第一阶段拆出 14 个子任务（棋盘模型、棋子 Prefab、落子逻辑……），每个 Issue 标题写清楚做什么，assign 给当轮负责的人，完成就 close。

Issue 标签用三种颜色：🟦 3D 场景 / 🟩 逻辑 / 🟨 UI，一眼就知道属于哪个方向。

### 6. GitHub 还能干什么

除了存代码和管任务，GitHub 是你的资源库：搜开源项目（关键词 + 按 Stars 排序）、看 README（作者的说明书）、搜代码片段（选 Code 标签）、看 Issues（别人踩过的坑）。

### 7. VSCode：统一 IDE

IDE = 写代码的软件，把"写代码 + 运行 + 报错提示"集成在一个窗口。就像 Word 写文档、PS 修图，IDE 专门写代码。我们选 VSCode——轻量、插件多、什么语言都支持。不需要装 PyCharm 或 Visual Studio。

搭环境和搭 C/C++ 一样：装解释器（相当于 MinGW） + VSCode 里装插件。Python 去 python.org 下载 3.12，勾选"Add Python to PATH"。VSCode 里装 Python 插件。装完终端敲 `python3 --version` 确认。

---

## 二、AI 编程助手

VSCode 可以装 AI 插件。我们主要用 Claude Code，背后是三个概念：

**Model（模型）**：AI 的大脑。GPT、Claude、DeepSeek 都是模型。喂它文字就吐文字。

**Prompt**：提示词，问的简单，AI答得泛面，问的具体，AI答得具体。核心是个人提炼信息，清晰的表达能力。

**Agent（智能体）**：模型 + 能操作电脑的工具。光聊天不够——Agent 能读代码、改文件、在终端跑编译。Claude Code、Cursor、GitHub Copilot 都是 Agent。ycccuc电脑上配好了。

**Skill（技能）**：Agent 的预装工作流程。比如 `/grill-me` 就是一整套方案审问流程。队长接入了 Matt Pocock 开源的一套 Skill（github.com/mattpocock/skills），队员不需要配。

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

### 前置概念：前端和后端是什么

拿我们这个项目举例：

**前端（客户端）** = 玩家电脑上双击打开的 Unity exe。负责画面、交互、按钮、3D 棋盘——你看得到摸得着的全是前端。

**后端（服务端）** = 用 Python 写的 AI 程序。玩家看不见它，它躲在远处（云服务器上）算 AI 该下哪步。前端发棋盘数据给它，它算完返回落子坐标。

类比：你点外卖——前端是手机上的美团 App，后端是商家后厨。你点的菜（棋盘数据）发给后厨，后厨做好（AI 算完）骑手送回给你（Unity 显示棋子）。

> 我们项目第一阶段只做前端（单机本地），第二阶段开始写后端（AI），第三四阶段把前后端用网络连起来。

---

## 四、第一阶段：单机 3D 五子棋

### Unity 是什么

Unity 是一个游戏引擎——帮你画 3D 画面、处理鼠标点击、播放动画，不用从零写底层代码。我们用它做 3D 五子棋。它既能处理**3D 场景**（立体棋盘棋子），也能用 C# 写**客户端逻辑**（落子判定、胜负校验），还能搭**UI**（按钮、菜单、文字）。

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
2. 棋子 Prefab——黑白棋子预制体，带反光
3. 灯光环境——场景照明，让棋盘看起来立体
4. 摄像机——默认视角、鼠标滚轮缩放、拖拽旋转
5. 棋盘坐标标尺——棋盘上的网格线和星位标记

**方向二：游戏核心逻辑**

1. 落子规则——鼠标点击 → 射线检测 → 吸附最近交叉点 → 落子
2. 胜负判定——每次落子后四个方向扫描，连五即胜
3. 悔棋逻辑——栈结构弹出上两步，恢复棋盘
4. 手数记录——显示当前第几手、轮到谁

**方向三：交互层 + UI**

1. 鼠标点击落子——射线检测 + 交叉点吸附
2. 按钮——悔棋、认输、重新开始
3. 状态显示——当前回合、手数、胜利提示
4. 菜单——主菜单、难度选择、模式切换

### 怎么分工

不做"张三只做 3D、李四只做 UI"那种模式——那样每个人只学了自己那一小块。我们是**一起学、一起做**。每个子任务两三个人一组去完成，下次任务换组合。最终每个人都摸过 3D、逻辑、UI，知识不偏食。

具体做法：把子任务挂到 GitHub Issues 上，每两周认领一轮。

### 前两周每个人要做什么

1. 学GitHub基础知识，先实现协同在一个仓库里添加文件
2. 配置好VSCode插件，中文 + Github用法 + claude code(选配，建议配一个) + Python系列插件(现在用不到，可以不着急) 
3. 下载 Unity Hub → 安装 Unity 编辑器
4. 完成官网 Roll-a-Ball 教程（3 小时），做完就懂 GameObject、Prefab、脚本是什么
5. 在场景里用 Cube 拼出 15×15 网格阵列（棋盘原型）
6. 了解基础 C# 语法：变量、函数、if/for(不需要精通，会用就行，先不学类和对象)


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