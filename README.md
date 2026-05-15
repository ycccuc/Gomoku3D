# Gomoku3D
A 3D Gomoku game with multi-strategy AI, built with Unity.

# 基于多策略博弈算法的 3D 五子棋人机对战系统

---

## 项目简介

一个 3D 五子棋桌面游戏，支持六种不同博弈算法AI，实现棋力逐级递增的 AI 对手、联网对战和观战。Unity 3D 做前端渲染，Python 做 AI 后端，WebSocket 做网络通信。

## 功能

-  3D 棋盘，鼠标点击落子，自由旋转缩放视角
-  六种 AI 策略：规则引擎 → Minimax → Alpha-Beta → 迭代加深+历史 → MCTS → 神经网络+MCTS
-  联网对战：人 vs 人、人 vs AI、AI vs AI，支持观战
-  AI 决策可视化：候选点热度图、评分柱状图、搜索路径展示
-  悔棋、认输、手数显示

## 技术栈

| 模块 | 技术 |
|---|---|
| 3D 前端 | Unity (C#) |
| AI 后端 | Python + easyAI / pymcts |
| 网络通信 | WebSocket |
| 版本管理 | Git + GitHub |

## 进度

- [ ] 第一阶段：单机 3D 五子棋系统
- [ ] 第二阶段：接入 AI 对战功能
- [ ] 第三阶段：联网对战
- [ ] 第四阶段：双端分离部署
- [ ] 第五阶段：整理材料与结题

## 本地运行（开发中）

```bash
# Python AI 后端
cd server
pip install easyai pymcts websockets
python main.py

# Unity 前端
# 用 Unity Hub 打开 unity/ 文件夹，点 Play
```
