# Claude Code 宠物系统（BUDDY）完整深度解析

基于 2026年3月31日 Claude Code 源码泄露事件，对内置宠物伙伴系统的全面分析。

---

## 真实的 18 种宠物（ASCII 精灵）

Claude Code 的 BUDDY 宠物在终端中以 **ASCII 字符画**形式呈现（使用 React/Ink 渲染）。以下是从泄露源码 `buddy/sprites.ts` 中提取的全部 18 种宠物：

```
   \^^^/         .----.        n  ____  n      n______n        /\_/\         /\    /\  
}~(______)~{    ( °  ° )      | |°  °| |     ( ×    × )      ( ·   ·)      ( ×    × ) 
}~(× .. ×)~{   (      )      |_|    |_|     (   Oo   )      (  ω  )       (   ..   ) 
  ( .--. )       `----´         |    |        `------´       (")_(")        `------´  
  (_/  \_)                                                                            
  AXOLOTL         BLOB          CACTUS        CAPYBARA         CAT           CHONK     
```

```
  /^\  /^\        __           .----.          (°>          . o  .         .----.    
 <  °  °  >     <(- )___     / °  ° \          ||        .-o-OO-o-.     ( °  ° )   
 (   ~~   )      (  ._>      |      |        _(__)_     (__________)   (______)    
  `-vvvv-´        `--´       ~`~``~`~         ^^^^        |°  °|       /\/\/\/\    
                                                          |____|                   
  DRAGON          DUCK         GHOST          GOOSE       MUSHROOM      OCTOPUS     
```

```
   /\  /\        .---.        (\__/)         .[||].        °    .--.      _,--._    
  ((@)(@))      (×>×)       ( ◉  ◉ )       [ ×  × ]        \  ( @ )    ( ·  · )   
  (  ><  )     /(   )\     =(  ..  )=       [ ==== ]         \_`--´    /[______]\  
   `----´       `---´       (")__(")        `------´        ~~~~~~~     ``    ``   
                                                                                   
    OWL         PENGUIN       RABBIT          ROBOT          SNAIL        TURTLE    
```

### 宠物定制选项

| 系统 | 选项 |
|------|------|
| **眼睛** | `·` `✦` `×` `◉` `@` `°` — 6种样式 |
| **帽子** | 无、皇冠、礼帽、螺旋桨帽、光环、巫师帽、毛线帽、小鸭子 — 8种 |
| **闪光** | 普通 / Shiny✦（特殊发光效果） |

### 稀有度

| 等级 | 概率 |
|------|------|
| Common（普通） | 60% |
| Uncommon（罕见） | 25% |
| Rare（稀有） | 10% |
| Epic（史诗） | 4% |
| Legendary（传说） | 1% |

### 性格属性（程序化生成）

每只宠物有 5 个独立属性值（0-100），由你的编码行为决定：

- **DEBUGGING（调试力）**
- **PATIENCE（耐心）**
- **CHAOS（混乱值）**
- **WISDOM（智慧）**
- **SNARK（毒舌值）**

---

## 在线体验

打开 `buddy-pet-demo/index.html` 即可体验完整的 BUDDY 宠物系统（无需服务器，直接在浏览器中打开）。

功能包括：
- 18 种宠物的实时 ASCII 动画（500ms/tick，3帧循环 + 眨眼）
- 抽卡系统（输入用户名 hash 生成，与官方 mulberry32 PRNG 一致）
- 6 种眼睛 + 8 种帽子定制
- 5 种稀有度 + 闪光变体
- 5 大属性条（DEBUGGING/PATIENCE/CHAOS/WISDOM/SNARK）
- 对话气泡（10 秒后淡出）
- 抚摸互动（心形飘浮动画）
- 物种图鉴（底部 18 种缩略图点击切换）

---

## 项目内容

```
.
├── README.md                           # 本文件
├── Claude-Code-Pet-System-Analysis.md  # 完整深度分析报告
├── buddy-pet-demo/                     # 可体验的 Web 宠物系统
│   └── index.html                      # 单文件应用，直接打开即可
├── assets/                             # 可视化展示图 (15张)
│   ├── sprites_3_frames_all.png        # 18种宠物x3帧动画全展示
│   ├── sprites_blink_states.png        # 眨眼状态对比
│   ├── sprites_hats_capybara.png       # 8种帽子展示
│   ├── sprites_eyes_cat.png            # 6种眼睛展示
│   ├── sprites_rarity_shiny.png        # 稀有度+闪光对比
│   ├── sprites_interaction_states.png  # 交互状态展示
│   ├── real_18_species_ascii.png       # 真实18种宠物ASCII图鉴
│   ├── real_buddy_customization.png    # 宠物定制系统展示
│   ├── buddy_18_species.png            # 18种宠物像素风图鉴
│   ├── buddy_stats_rarity.png          # 性格属性与稀有度系统
│   ├── buddy_system_overview.png       # Buddy系统概览
│   ├── pet_evolution_system.png        # 进化系统展示
│   ├── pet_stats_and_moods.png         # 宠物属性与情绪界面
│   ├── pet_ecosystem_overview.png      # 宠物生态系统全览
│   └── clawd_12_states.png             # Clawd on Desk 12种状态
└── claw-code-main/                     # instructkr/claw-code 源码
    └── src/buddy/                      # buddy子系统占位包
```

---

## 宠物系统工作原理

宠物以 ASCII 精灵形式**坐在用户输入框旁边**，通过对话气泡偶尔发表评论。当用户直接叫宠物名字时，气泡会回应。

### 交互命令

| 命令 | 功能 |
|------|------|
| `/buddy` | 呼出宠物系统 |
| `/pet:status` | 查看状态 |
| `/pet:feed` | 喂食（+30 饥饿值） |
| `/pet:play` | 玩耍（+25 快乐值） |
| `/pet:sleep` | 切换睡眠 |

### XP 获取

| 编码行为 | XP |
|---------|-----|
| 写入文件 | +10 |
| 修复错误 | +25 |
| 完成会话 | +50 |

---

## 背景

2026年3月31日，Anthropic 通过 npm source map 意外泄露了 Claude Code 完整源码（512,000行 TypeScript）。BUDDY 宠物系统被发现隐藏在 `buddy/` 目录中，锁在编译时功能开关后面，原计划 4月1日预热、5月正式发布。

### 源码模块结构

| 文件 | 功能 |
|------|------|
| `CompanionSprite.tsx` | React/Ink 精灵渲染 |
| `companion.ts` | 伙伴核心逻辑 |
| `prompt.ts` | 对话模板 |
| `sprites.ts` | ASCII 精灵数据 |
| `types.ts` | 类型定义 |
| `useBuddyNotification.tsx` | 通知系统 |

---

## 社区衍生项目

| 项目 | 类型 | 链接 |
|------|------|------|
| Clawd on Desk | 桌面宠物（12种动画状态） | [GitHub](https://github.com/rullerzhou-afk/clawd-on-desk) |
| Claude Pet Companion | PyPI 插件（10阶段进化） | [PyPI](https://pypi.org/project/claude-pet-companion/) |
| Claude Quest | 像素RPG伙伴 | [Blog](https://michaellivs.com/blog/claude-quest) |
| Claude Code Crabgotchi | VS Code ASCII螃蟹 | [GitHub](https://github.com/panuhen/claude-code-crabgotchi) |

---

## 参考资料

- [Tech Startups: Claude Code 泄露分析](https://techstartups.com/2026/03/31/anthropics-claude-source-code-leak-goes-viral-again-after-full-source-hits-npm-registry-revealing-hidden-capybara-models-and-ai-pet/)
- [CyberKendra: 详细技术分析](https://www.cyberkendra.com/2026/03/anthropics-claude-code-source-code.html)
- [DEV.to: 512,000 行源码曝光](https://dev.to/evan-dong/claude-codes-entire-source-code-just-leaked-512000-lines-exposed-3139)
- [Claude Buddy Gallery](https://claude-buddy.vercel.app/) — 18种宠物交互展示
- [instructkr/claw-code](https://github.com/instructkr/claw-code)

---

*分析日期：2026年4月1日*
