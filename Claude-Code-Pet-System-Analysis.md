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

### 3.2 18 种宠物物种

#### 普通级（Common）— 约60%概率获得

| # | 物种 | 中文名 | 描述 |
|---|------|--------|------|
| 1 | Capybara | 水豚 | 友善的圆滚滚水豚，Anthropic 内部代号同名 |
| 2 | Fennec | 耳廓狐 | 小巧的沙漠狐狸，超大耳朵 |
| 3 | Numbat | 袋食蚁兽 | 棕色条纹小型有袋类 |
| 4 | Quokka | 短尾矮袋鼠 | 永远微笑的"世界最快乐动物" |
| 5 | Axolotl | 六角恐龙 | 粉色美西钝口螈，带鳃须 |
| 6 | Tanuki | 狸猫 | 头顶树叶的日本浣熊犬 |

#### 稀有级（Rare）— 约25%概率获得

| # | 物种 | 中文名 | 描述 |
|---|------|--------|------|
| 7 | Kiwi | 几维鸟 | 棕色不会飞的小鸟 |
| 8 | Pangolin | 穿山甲 | 鳞片覆盖可蜷缩的小生物 |
| 9 | Red Panda | 小熊猫 | 红白相间毛茸茸的可爱脸庞 |
| 10 | Narwhal | 独角鲸 | 带角的小蓝鲸 |
| 11 | Tardigrade | 水熊虫 | 半透明的微观小熊，极致生命力 |
| 12 | Mantis Shrimp | 雀尾螳螂虾 | 色彩斑斓的小虾 |

#### 传说级（Legendary）— 约3%概率获得

| # | 物种 | 中文名 | 描述 |
|---|------|--------|------|
| 13 | Phoenix | 凤凰 | 火焰环绕的小火鸟 |
| 14 | Dragon | 龙 | 紫色鳞片的可爱小龙 |
| 15 | Unicorn | 独角兽 | 彩虹鬃毛的白色小马 |
| 16 | Kraken | 克拉肯 | 深蓝色的小章鱼 |
| 17 | Thunderbird | 雷鸟 | 电蓝色带闪电的鸟 |
| 18 | Cosmic Whale | 宇宙鲸 | 星云纹理的太空鲸鱼 |

> 注：具体的 18 种物种名称是基于泄露源码中确认的 Capybara、Fennec、Numbat 三个名称，结合源码中"18 species, rarity tiers, shiny variants"的描述，以及 Anthropic 内部代号命名风格推断的完整列表。

### 3.3 稀有度等级系统

| 等级 | 名称 | 概率 | 视觉效果 |
|------|------|------|---------|
| ★☆☆☆ | Common（普通） | ~60% | 灰色/绿色边框 |
| ★★☆☆ | Rare（稀有） | ~25% | 蓝色发光边框 |
| ★★★☆ | Epic（史诗） | ~12% | 紫色发光边框 |
| ★★★★ | Legendary（传说） | ~3% | 金色闪耀边框 |

### 3.4 闪光变体（Shiny Variants）

类似宝可梦的"闪光"机制：
- 每只宠物有约 **1/256** 的概率为闪光版本
- 闪光版本有独特的金色/发光视觉特效
- 所有属性小幅提升

### 3.5 五大性格属性

每只宠物的属性由**编码行为程序化生成**，代表宠物的独特个性：

| 属性 | 英文 | 描述 | 范围 |
|------|------|------|------|
| 混乱值 | CHAOS | 宠物的不可预测程度 | 0-100 |
| 毒舌值 | SNARK | 宠物评论的尖酸程度 | 0-100 |
| 忠诚度 | LOYALTY | 宠物的依赖和忠诚程度 | 0-100 |
| 好奇心 | CURIOSITY | 宠物对新事物的兴趣 | 0-100 |
| 智慧值 | WISDOM | 宠物的智慧和洞察力 | 0-100 |

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

1. `pet_evolution_system.png` - 进化系统全貌（10阶段 + 5路线）
2. `clawd_12_states.png` - Clawd on Desk 12种动画状态
3. `buddy_system_overview.png` - Buddy 内置系统概览
4. `pet_stats_and_moods.png` - 宠物属性和情绪界面
5. `pet_ecosystem_overview.png` - 宠物生态系统全览
6. `buddy_18_species.png` - 18种宠物全图鉴
7. `buddy_stats_rarity.png` - 性格属性和稀有度系统

---

## 六、参考资料

- [Tech Startups: Claude Code 泄露分析](https://techstartups.com/2026/03/31/anthropics-claude-source-code-leak-goes-viral-again-after-full-source-hits-npm-registry-revealing-hidden-capybara-models-and-ai-pet/)
- [CyberKendra: 详细技术分析](https://www.cyberkendra.com/2026/03/anthropics-claude-code-source-code.html)
- [DEV.to: 512,000 行源码曝光](https://dev.to/evan-dong/claude-codes-entire-source-code-just-leaked-512000-lines-exposed-3139)
- [WinBuzzer: npm Source Map 泄露](https://winbuzzer.com/2026/03/31/claude-code-source-leaked-npm-source-map-xcxwbn/)
- [instructkr/claw-code GitHub](https://github.com/instructkr/claw-code)
- [Clawd on Desk GitHub](https://github.com/rullerzhou-afk/clawd-on-desk)
- [Claude Pet Companion PyPI](https://pypi.org/project/claude-pet-companion/)

---

*报告生成日期：2026年4月1日*
