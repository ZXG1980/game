# 余烬回廊 · Ember Corridor

10 分钟一局的元素 Roguelike 网页原型。分层工程结构，纯原生 JS（无框架、无构建工具），双击 `余烬回廊.html` 即可运行。

## 运行

- 直接双击 `余烬回廊.html`（file:// 可用，脚本均以普通 `<script>` 加载，不依赖模块化 CORS）
- 若浏览器对本地文件限制较严，也可在该目录执行 `python3 -m http.server 8000` 后访问 `http://localhost:8000/余烬回廊.html`
- 存档写在 `localStorage`（键 `ember_corridor_v1`）

## 操作

| 操作           | 键位                           |
| -------------- | ------------------------------ |
| 移动           | WASD / 方向键 / 鼠标或手指拖拽 |
| 冲刺（短无敌） | 空格                           |
| 释放技能       | 1 – 5                          |

## 目录结构

```
余烬回廊-工程/
├── 余烬回廊.html            入口页：画布、HUD、遮罩层、脚本加载顺序
├── css/
│   └── style.css            全部样式（HUD、卡牌、面板、图鉴）
└── js/
    ├── core/
    │   ├── core.js          基础工具：画布、随机、取整、距离、DOM 取用
    │   ├── save.js          存档读写、解锁价格、全局状态 g
    │   ├── player.js        玩家构建、模组加成、主元素判定
    │   └── combat.js        伤害/印记/反应结算、施法分发、敌人 AI、受击
    ├── data/
    │   ├── elements.js      四元素、六反应表 REACT、被动词条 PASK
    │   ├── skills.js        技能池 SKILLS（16 主动 + 6 规则型）
    │   └── stages.js        敌人属性 ETYPE、房间配置 ROOMS
    ├── game/
    │   ├── run.js           开局、房间推进、波次、清房、Boss 启动
    │   └── draft.js         三选一抽卡、卡牌渲染、重掷
    ├── ui/
    │   ├── panels.js        抉择室 / 商店 / 篝火 / 宝箱 / 结算 / 大厅 / 图鉴
    │   └── render.js        背景、区域、敌人、玩家、特效、浮字
    └── main.js              主循环、HUD 刷新、键盘与触屏输入、启动
```

脚本按 `余烬回廊.html` 底部的顺序串联加载，顶层声明在同一个全局词法作用域内共享（普通脚本的既有行为），所以不需要任何打包步骤。

## 数据驱动：怎么加内容

### 加一个技能

在 `js/data/skills.js` 的 `SKILLS` 数组里追加一条，`kind` 决定行为：

| kind      | 行为                         | 关键参数                                                     |
| --------- | ---------------------------- | ------------------------------------------------------------ |
| `shot`    | 朝最近敌人发射弹道           | `power` `count` `splash` `zone`                              |
| `aoe`     | 以自身为中心的瞬发范围       | `power` `radius`                                             |
| `zone`    | 地面持续区域，按 `tick` 结算 | `power` `radius` `dur` `tick`                                |
| `dash`    | 向目标突进并在路径上判定     | `power` `len`                                                |
| `chain`   | 连锁跳跃                     | `power` `jumps`                                              |
| `strike`  | 直接落点打击                 | `power` `radius`                                             |
| `passive` | 规则型被动，只带 `mod`       | `mod: {dmg, react, cdr, hp, spd, vamp, basic, prism, chain}` |

`pas` 字段填 `dmg / react / cdr / hp / spd / vamp / basic`，对应 `elements.js` 里的 `PASK` 词条。

### 加一种反应

`elements.js` 里 `addR(a, b, { name, color, kind, mul, txt })`，`kind` 目前支持
`vapor / over / burn / super / freeze / echain`，具体结算在 `combat.js` 的 `doReact()` 中分支。

### 调关卡

`js/data/stages.js` 的 `ROOMS`：

- `hp`：该房间内各类敌人的血量
- `waves`：波次数组，`{t: 敌人类型, n: 数量}`
- `lim`：软时限，超时后玩家每秒掉 1.2% 最大生命（逼推进）
- `choice: true`：该房间改为抉择室
- `crystals: true`：该房间额外生成元素晶石

## 数值现状

| 项         | 值                                           |
| ---------- | -------------------------------------------- |
| 玩家生命   | 130（被动加成另算）                          |
| 普攻       | 12 伤害 / 0.55s                              |
| 每技能增益 | 约 +20%~35% 有效输出                         |
| 单局时长   | 快速通关约 7 分钟，正常游玩 8–11 分钟        |
| 尘晶       | 击杀 + 通关奖励，未花部分按 60% 折算入局外   |
| 核心       | 精英 1 / Boss 2 / 宝箱 1–2，用于解锁蓝紫技能 |

## 已实现 / 未实现

已实现：5 房间 + 抉择室 + 双阶段 Boss、6 种元素反应、三选一与重掷、商店/篝火/宝箱、图腾破防、局内与局外双货币、永久解锁与偏好元素、本地存档、键鼠与触屏操作、Canvas 特效。

未实现：音效、手柄、排行榜、成就、技能升阶独立数值表（目前统一 +30%）。
