# Claude Code 宠物系统（BUDDY System）完整深度解析

> 基于 2026年3月31日 Claude Code 源码泄露事件的分析报告

---

## 一、背景

2026年3月31日，Anthropic 意外通过 npm registry（版本 2.1.88）的 source map 文件泄露了 Claude Code 的完整源代码（512,000 行 TypeScript，1,906 个文件）。社区开发者在分析源码时发现了一个**从未公开发布的隐藏功能** —— 内置宠物伙伴系统 **BUDDY**，通过隐藏的 `/buddy` 命令触发。

该功能原计划于 **2026年4月1日-7日** 作为预热发布（愚人节彩蛋），**5月** 正式上线，首先对 Anthropic 内部员工开放。

---

## 二、本地 claw-code 项目分析

`claw-code-main`（来自 [instructkr/claw-code](https://github.com/instructkr/claw-code)）是 Python 重写/移植版本，宠物系统仅保留元数据占位。

### buddy.json 元数据

```json
{
  "archive_name": "buddy",
  "package_name": "buddy",
  "module_count": 6,
  "sample_files": [
    "buddy/CompanionSprite.tsx",
    "buddy/companion.ts",
    "buddy/prompt.ts",
    "buddy/sprites.ts",
    "buddy/types.ts",
    "buddy/useBuddyNotification.tsx"
  ]
}
```

### 原始 TypeScript 源码模块结构

| 文件 | 功能 |
|------|------|
| `CompanionSprite.tsx` | 宠物精灵的 React/Ink 渲染组件 |
| `companion.ts` | 宠物伙伴核心逻辑 |
| `prompt.ts` | 宠物相关的提示词和对话模板 |
| `sprites.ts` | 精灵数据（外观、动画帧定义） |
| `types.ts` | 类型定义（物种、属性、稀有度） |
| `useBuddyNotification.tsx` | 宠物通知系统 React Hook |

---

## 三、BUDDY 宠物系统完整解析

### 3.1 概述

BUDDY 是一个类似 Tamagotchi（电子宠物）的 AI 伙伴系统，宠物以像素风格精灵形态显示在用户输入框旁边的对话气泡中。它具有以下核心特性：

- **18 种宠物物种**
- **4 个稀有度等级**
- **闪光（Shiny）变体**
- **5 大性格属性**（程序化生成）
- **帽子和装饰品系统**
- **像素动画**
- **编译时功能开关**（默认锁定）

### 3.2 真实的 18 种宠物物种

以下是从泄露源码 `buddy/sprites.ts` 中确认的全部 18 种宠物的 **ASCII 精灵**（终端中的实际样子）：

> 参考来源：[Claude Buddy Gallery](https://claude-buddy.vercel.app/) — 基于泄露源码重建的交互展示

#### 完整物种列表

| # | 物种 | 中文名 | ASCII 精灵预览 | 描述 |
|---|------|--------|---------------|------|
| 1 | **Axolotl** | 六角恐龙 | `}~(× .. ×)~{` | 带波浪鳃须的美西钝口螈 |
| 2 | **Blob** | 果冻怪 | `( °  ° )` | 圆滚滚的简单生物 |
| 3 | **Cactus** | 仙人掌 | `\|°  °\|` | 盆栽仙人掌，带小脸 |
| 4 | **Capybara** | 水豚 | `( ×    × )  (   Oo   )` | 友善的圆脸水豚，Anthropic 内部模型代号同名 |
| 5 | **Cat** | 猫 | `( ·   ·)  (  ω  )` | 带猫耳和ω嘴的可爱猫咪 |
| 6 | **Chonk** | 胖墩 | `( ×    × )  (   ..   )` | 圆润的胖胖生物 |
| 7 | **Dragon** | 龙 | `<  °  °  >  (   ~~   )` | 带翅膀和角的小飞龙 |
| 8 | **Duck** | 鸭子 | `<(- )___` | 简笔画风格的小鸭子 |
| 9 | **Ghost** | 幽灵 | `/ °  ° \  ~\`~\`\`~\`~` | 飘浮的可爱小幽灵 |
| 10 | **Goose** | 鹅 | `(°>  \|\|  _(__)_` | 站立的小鹅，经典"Untitled Goose"风 |
| 11 | **Mushroom** | 蘑菇 | `.-o-OO-o-.  \|°  °\|` | 带圆点的蘑菇头，有小脸 |
| 12 | **Octopus** | 章鱼 | `( °  ° )  /\\/\\/\\/\\` | 触手章鱼 |
| 13 | **Owl** | 猫头鹰 | `((@)(@))  (  ><  )` | 大眼睛的猫头鹰 |
| 14 | **Penguin** | 企鹅 | `(×>×)  /(   )\\` | 系围巾的小企鹅 |
| 15 | **Rabbit** | 兔子 | `( ◉  ◉ )  =(  ..  )=` | 竖耳朵的小兔子 |
| 16 | **Robot** | 机器人 | `[ ×  × ]  [ ==== ]` | 方方正正的小机器人 |
| 17 | **Snail** | 蜗牛 | `°  .--.  \\_\`--´` | 背着壳慢慢爬的蜗牛 |
| 18 | **Turtle** | 乌龟 | `( ·  · )  /[______]\\` | 背着龟壳的小乌龟 |

#### ASCII 精灵完整展示

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

### 3.3 宠物定制系统

每只宠物可以通过以下选项定制外观：

#### 眼睛样式（6种）

| 符号 | 名称 |
|------|------|
| `·` | dot（圆点） |
| `✦` | sparkle（闪光） |
| `×` | cross（叉叉） |
| `◉` | target（靶心） |
| `@` | at |
| `°` | degree（度） |

#### 帽子（8种）

| 帽子 | 英文 | 说明 |
|------|------|------|
| 无 | none | 默认无帽子 |
| 皇冠 | crown | 小皇冠 |
| 礼帽 | tophat | 绅士高帽 |
| 螺旋桨帽 | propeller | 童趣螺旋桨帽 |
| 光环 | halo | 天使光环 |
| 巫师帽 | wizard | 魔法师尖帽 |
| 毛线帽 | beanie | 冬季毛线帽 |
| 小鸭子 | tinyduck | 头顶一只小鸭子 |

### 3.4 稀有度等级系统

| 等级 | 名称 | 概率 |
|------|------|------|
| ★ | Common（普通） | 60% |
| ★★ | Uncommon（罕见） | 25% |
| ★★★ | Rare（稀有） | 10% |
| ★★★★ | Epic（史诗） | 4% |
| ★★★★★ | Legendary（传说） | 1% |

### 3.5 闪光变体（Shiny Variants）

类似宝可梦的"闪光"机制：
- 宠物可以切换为 Shiny✦ 闪光模式
- 闪光版本有独特的视觉特效

### 3.6 五大性格属性

每只宠物的属性由**编码行为程序化生成**，代表宠物的独特个性：

| 属性 | 英文 | 描述 | 范围 |
|------|------|------|------|
| 调试力 | DEBUGGING | 宠物帮助调试的能力 | 0-100 |
| 耐心 | PATIENCE | 宠物的耐心程度 | 0-100 |
| 混乱值 | CHAOS | 宠物的不可预测程度 | 0-100 |
| 智慧 | WISDOM | 宠物的智慧和洞察力 | 0-100 |
| 毒舌值 | SNARK | 宠物评论的尖酸程度 | 0-100 |

### 3.6 养成属性

| 属性 | 说明 | 恢复方式 |
|------|------|---------|
| Hunger（饥饿值） | 会随时间衰减 | `/pet:feed` +30 |
| Happiness（快乐值） | 会随时间衰减 | `/pet:play` +25 |
| Energy（能量值） | 会随时间衰减 | `/pet:sleep` 恢复 |
| XP（经验值） | 通过编码积累 | 自动获取 |

### 3.7 XP 经验获取

| 编码行为 | 获得 XP |
|---------|--------|
| 写入文件 | +10 XP |
| 修复错误 | +25 XP |
| 完成会话 | +50 XP |

### 3.8 帽子和装饰品系统

宠物可以佩戴各种帽子和装饰品（hats），这是源码中明确提到的定制化系统。

### 3.9 交互命令

| 命令 | 功能 |
|------|------|
| `/buddy` | 呼出宠物系统（隐藏命令） |
| `/pet:status` | 查看宠物状态、等级、心情 |
| `/pet:feed` | 喂食宠物 |
| `/pet:play` | 和宠物玩耍 |
| `/pet:sleep` | 切换睡眠模式 |

### 3.10 宠物会动吗？

**会！** BUDDY 系统使用 React/Ink 在终端中渲染像素动画精灵：
- `CompanionSprite.tsx` 负责精灵渲染，使用 React 组件化方式
- `sprites.ts` 包含精灵帧数据（各状态的像素图案）
- 宠物位于用户输入框旁的对话气泡中
- 根据编码活动实时变化动画状态

---

## 四、社区衍生宠物项目

### 4.1 Clawd on Desk

**GitHub**: [rullerzhou-afk/clawd-on-desk](https://github.com/rullerzhou-afk/clawd-on-desk)（500+ Stars）

桌面宠物应用，12 种动画状态：Idle、Thinking、Typing、Building、Juggling、Conducting、Error、Happy、Notification、Sweeping、Carrying、Sleeping

核心特性：
- 眼球追踪（跟随光标）
- 睡眠/惊醒序列
- 多代理支持（Claude Code / Codex CLI / Copilot CLI / Gemini CLI / Cursor Agent）
- 权限审批气泡
- 支持 Windows / macOS / Linux

### 4.2 Claude Pet Companion

**PyPI**: `pip install claude-pet-companion`

最完善的社区实现：
- 10 阶段进化系统（Egg → Ancient）
- 5 条进化路线（Coder / Warrior / Social / Night Owl / Balanced）
- 3D 伪渲染
- 守护进程模式
- 30+ 成就
- 小游戏（接球、记忆游戏）

### 4.3 Claude Pet

**GitHub**: [scm1400/claude-pet](https://github.com/scm1400/claude-pet)

轻量级桌面宠物小部件

### 4.4 Claude Code Crabgotchi

**GitHub**: [panuhen/claude-code-crabgotchi](https://github.com/panuhen/claude-code-crabgotchi)

VS Code 扩展，ASCII 螃蟹伙伴

### 4.5 Claude Quest

像素风 RPG 伙伴（Go + Raylib），将编码会话可视化为冒险

---

## 五、生成的可视化图片

以下图片位于 `assets/` 目录：

1. `real_18_species_ascii.png` - **真实18种宠物ASCII图鉴**（来自泄露源码）
2. `real_buddy_customization.png` - **真实宠物定制系统**（眼睛/帽子/稀有度）
3. `buddy_18_species.png` - 18种宠物像素风图鉴
4. `buddy_stats_rarity.png` - 性格属性与稀有度系统
5. `buddy_system_overview.png` - Buddy 内置系统概览
6. `pet_evolution_system.png` - 进化系统全貌（社区版）
7. `pet_stats_and_moods.png` - 宠物属性和情绪界面
8. `pet_ecosystem_overview.png` - 宠物生态系统全览
9. `clawd_12_states.png` - Clawd on Desk 12种动画状态

---

## 六、参考资料

- [Tech Startups: Claude Code 泄露分析](https://techstartups.com/2026/03/31/anthropics-claude-source-code-leak-goes-viral-again-after-full-source-hits-npm-registry-revealing-hidden-capybara-models-and-ai-pet/)
- [CyberKendra: 详细技术分析](https://www.cyberkendra.com/2026/03/anthropics-claude-code-source-code.html)
- [DEV.to: 512,000 行源码曝光](https://dev.to/evan-dong/claude-codes-entire-source-code-just-leaked-512000-lines-exposed-3139)
- [WinBuzzer: npm Source Map 泄露](https://winbuzzer.com/2026/03/31/claude-code-source-leaked-npm-source-map-xcxwbn/)
- [Claude Buddy Gallery](https://claude-buddy.vercel.app/) — 18种宠物交互展示（基于泄露源码重建）
- [instructkr/claw-code GitHub](https://github.com/instructkr/claw-code)
- [Clawd on Desk GitHub](https://github.com/rullerzhou-afk/clawd-on-desk)
- [Claude Pet Companion PyPI](https://pypi.org/project/claude-pet-companion/)

---

*报告生成日期：2026年4月1日*
