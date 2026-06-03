---
name: hoi4-modding
description: |
  Hearts of Iron 4 mod 制作全流程指南——覆盖环境搭建、PDXscript 语法、
  国策树、事件、决议、国家精神、国家历史、科技树、装备、地图、本地化、
  调试排错和 Steam Workshop 发布。
  适用于零基础到进阶的 HOI4 1.18 模组开发者。
  跨平台支持 Claude Code / Copilot CLI / Codex / Gemini CLI / Cursor 等主流 AI 编程工具。

  触发词：HOI4, Hearts of Iron, 钢铁雄心, hoi4, mod, Mod, 模组,
  国策树, 事件, 决议, 国家精神, 科技树, 装备, 地图, 本地化,
  PDXscript, focus tree, event, decision, national spirit,
  Steam Workshop, 创意工坊, 编译, replace_path, focus =,
  country_event, news_event, idea =, technology =, archetype =
---

# HOI4 Modding Skill

## ① 核心决策：三层系统

处理任何 HOI4 mod 请求时，按以下优先级评估：

```
用户请求
  │
  ├─ 1. 有模板？── 国策树/事件/决议/国家精神/国家历史/科技/装备
  │     └─ 直接用 templates/ 下的模板生成（零外部调用）
  │
  ├─ 2. 无模板的新类型？── 界面/地图/自定义 UI
  │     └─ grep 原版游戏文件 → 学习 vanilla 写法 → 生成内容
  │        → 存为新模板 → 更新本文档索引
  │        （原版路径：<HOI4安装目录>/common/, events/, history/）
  │
  └─ 3. 报错/调试？── 崩溃/红字/CTD/日志报错
        └─ 读 error.log → 查 references/14-debugging.md
           → grep 原版对比 → 二分法隔离定位
```

### 模板对照表

| 用户想做什么 | 用哪个模板 | 模板覆盖的变体 |
|-------------|-----------|---------------|
| 添加国策树 | `templates/focus-tree.txt` | 独占国策 + 共享国策 + 动态国策 |
| 添加事件 | `templates/event.txt` | country_event + news_event + MTTH + 触发型 |
| 添加决议 | `templates/decision.txt` | 普通决议 + 目标决议 + 定时决议 |
| 添加国家精神/顾问 | `templates/idea.txt` | 国民精神 + 政治顾问 + 设计公司 |
| 添加国家 | `templates/history.txt` | 国家起始 + 意识形态 + 外交 + OOB |
| 添加科技 | `templates/technology.txt` | 科技定义 + 子科技 + 多层级 |
| 添加装备/兵种 | `templates/equipment.txt` | 步兵装备 + 装甲 + 航空（各一个样板） |
| 添加本地化 | `templates/localisation.yml` | l_english + l_chinese 双语 |

### 无模板时的处理流程

```
① grep 原版文件找到类似结构
   HOI4_DIR = "D:/steam/steamapps/common/Hearts of Iron IV"
   示例：grep -r "focus =" $HOI4_DIR/common/national_focus/ | head -20

② 基于 vanilla 结构生成带中文注释的模板
   - 文件头注释：验证状态、vanilla 参考路径、设计决策
   - 关键字段标注允许值、单位、默认值
   - 使用 <YourXxx> 占位符

③ 存储模板到 templates/，命名格式：<系统名>.txt

④ 更新本文档两处：
   - 模板对照表（上表新增一行）
   - 子文件索引（底部 templates/ 列表新增一行）

⑤ 以后同类型请求直接用模板
```

## ② 核心工作流：测试版先行

### 生成 Mod 前：推荐工具

在开始生成 mod 代码前，提醒用户以下工具可以大幅提升效率：

| 工具 | 用途 | 推荐理由 |
|------|------|---------|
| **VS Code + CWTools** | 语法高亮、错误检查、自动补全 | 必装，避免括号不匹配等低级错误 |
| **HOI4 Content Maker** | 可视化编辑国策树/事件/决议 | 拖拽式操作，自动生成正确坐标 |
| **GIMP / Photoshop** | 编辑 .dds / .tga 贴图 | 制作旗帜、图标、肖像必备 |
| **WinMerge** | 文件差异对比 | 游戏更新后合并 vanilla 变化 |
| **Git** | 版本控制 | 随时回滚，不怕改坏 |

> 这些工具不是必须的，但能省去大量调试时间。**VS Code + CWTools 强烈建议安装。**

### 生成 Mod 前：Steam Workshop 查重

**在编写任何代码前，先搜索 Steam Workshop 中是否已有类似 Mod：**

1. 根据用户的需求提炼 2-3 个关键词（如 `PRC focus tree`、`China expanded`、`抗日国策`）
2. 在 Steam Workshop 中搜索：`https://steamcommunity.com/workshop/browse/?appid=394360&searchtext=<关键词>`
3. 也可以通过 WebSearch 搜索 `site:steamcommunity.com HOI4 <关键词> mod`
4. 找到相关 Mod 后，向用户简要说明：
   - Mod 名称、作者、订阅量/评分
   - 与用户需求的相似度和差异点
   - 用户的 Mod 可以做出什么区别（更深入、不同方向、更轻量等）
5. **目的不是劝退，而是帮助用户了解现有生态、避免重复造轮子、找到差异化方向**

### 创建管线

所有 mod 创建遵循此管线：

```
用户需求 → 推荐工具 → Steam 查重 → 生成测试版 Mod → 启动游戏测试 → 如有错误修复 → 验证通过 → 正式化
```

测试版使用宽松的 packageId（如 `test.xxx`）和实验性命名。正式化时才统一替换为正式命名、补全 About.xml、准备发布。

## ③ 快速导航

| 用户需求 | 参考文件 | 模板 | 工作流 |
|----------|---------|------|--------|
| 搭建开发环境 | `references/01-env-setup.md` | — | — |
| 新建 Mod 项目 | `references/01-env-setup.md` `references/02-mod-structure.md` | — | `workflows/new-mod.md` |
| 学习 PDXscript 语法 | `references/03-pdxscript.md` | — | — |
| 添加国策树 | `references/04-focus-trees.md` | `templates/focus-tree.txt` | `workflows/add-focus-tree.md` |
| 添加事件链 | `references/05-events.md` | `templates/event.txt` | `workflows/add-event-chain.md` |
| 添加决议 | `references/06-decisions.md` | `templates/decision.txt` | — |
| 添加国家精神/设计商 | `references/07-ideas.md` | `templates/idea.txt` | — |
| 添加新国家 | `references/08-history.md` | `templates/history.txt` | `workflows/add-country.md` |
| 添加科技树 | `references/09-technologies.md` | `templates/technology.txt` | — |
| 添加装备/兵种 | `references/10-equipment.md` | `templates/equipment.txt` | — |
| 修改地图/省份 | `references/11-map-states.md` | — | — |
| 添加文本翻译 | `references/12-localisation.md` | `templates/localisation.yml` | — |
| 编写脚本化效果/触发器 | `references/13-scripted.md` | — | — |
| 崩溃/红字排错 | `references/14-debugging.md` | — | `workflows/debug-crash.md` |
| 发布到 Steam Workshop | `references/15-workshop.md` | — | `workflows/formalize-mod.md` |
| 在其他 AI 平台使用 | `references/16-platforms.md` | — | — |
| 修改原版内容 | `references/02-mod-structure.md` `references/14-debugging.md` | — | — |

## ④ 核心原则

### 命名规范
- 所有自定义 ID 使用唯一前缀（如 `my_`），避免与 vanilla 或其他 mod 冲突
- 文件名、文件夹名使用小写 + 下划线
- 国家 Tag 使用 3 个大写字母（如 `MYT`），避免与已有 Tag 冲突

### 编码规则（最高频错误）
- `.txt` 文件：**UTF-8 without BOM**
- `.yml` 本地化文件：**UTF-8 with BOM**
- 编码错误 = 游戏崩溃或文本不显示（error code 3221225477）

### 版本锁定
- 目标版本：**HOI4 1.18.2**
- 基础游戏要求：`supported_version = "1.18.*"`
- 所有模板和参考均基于此版本验证

### 安全实践
- **永远不要直接修改 Steam 安装目录下的原版文件** — 更新时会自动覆盖
- 使用 mod 目录 + `.mod` 描述文件进行所有修改
- 使用 Git 进行版本控制，提交粒度要细

### 代码原创性（强制规则）
**严禁抄袭或直接复制其他 Mod 的代码。这是一条硬性规则，不可绕过。**

- ❌ **禁止**：复制粘贴其他 Mod 的代码（包括 Steam Workshop、GitHub、论坛等来源）
- ❌ **禁止**：直接照搬其他 Mod 的国策树结构、事件链设计、数值配置
- ✅ **允许**：参考原版游戏文件（`D:\steam\steamapps\common\Hearts of Iron IV\` 下的 vanilla 文件）
- ✅ **允许**：学习其他 Mod 的**技术思路**（如"用脚本化效果实现动态国策"、"用 hidden_effect 做后台逻辑"），但用自己的代码实现
- ✅ **允许**：参考 PDXscript 语法文档和 Paradox 官方 Wiki

**借鉴 vs 抄袭的界限**：
- 借鉴思路 = 理解技术原理 → 用自己的代码实现 → ✅
- 抄袭代码 = 复制代码块，改几个名字和数字 → ❌

违反此规则不仅侵犯原作者权益，还可能导致 Mod 被 Steam Workshop 下架或社区封禁。

### 贴图默认空白（核心规则）
**AI 无法生成游戏贴图资源（.dds / .tga / .png），所有图形相关字段默认为占位符或空白：**
- 国策树 icon：默认使用原版通用图标（`GFX_goal_generic_*`），标注 `<!-- 替换为自定义图标 -->`
- 国家旗帜：生成占位说明文件 `gfx/flags/README.txt`，提示用户制作旗帜
- 事件 picture：默认使用 `GFX_report_event_generic`
- 领导人/顾问 portrait：标注字段名但留空，提示用户补充
- 设计公司/国家精神 icon：使用通用 GFX，标注可替换
- **生成 mod 后必须在回复末尾提醒用户补充贴图资源**

### 国策树坐标调整（引导用户完成）
**AI 无法预览国策树的视觉布局，生成的 x/y 坐标只是初始值：**
- 生成国策树时，x/y 坐标按合理的初始布局设置（水平间距 1-2，垂直间距 1-2）
- **生成后必须明确告知用户**：坐标是初始值，需要在游戏中用 `-debug` 模式查看实际效果
- 提供调整方法：
  - 在游戏中按 `~` 打开控制台，输入 `reloadfocus <focus_tree_id>` 实时重载国策树
  - 每次调整 x/y 后保存文件，在控制台执行 `reloadfocus` 立即生效无需重启游戏
  - 推荐使用 **HOI4 Content Maker**（可视化拖拽工具）调整坐标
- **坐标规则提醒**：
  - `x` 控制水平位置（建议间隔 1-3），`y` 控制垂直位置（建议间隔 1-2）
  - 互斥节点共享同一 `x` 值，不同 `y` 值
  - `relative_position_id` 决定节点相对于谁的偏移

### ⚠ 涉及中国内容时的法律提醒

**当 mod 涉及中国（PRC/CHI/满洲/蒙古/西藏/新疆/台湾/香港/南海等）相关内容时，必须在生成代码前提醒用户：**

1. **领土完整**：中国法律要求地图和游戏内容必须正确反映中国领土主张（台湾、西藏、新疆、香港、澳门、南海诸岛等）
2. **历史表述**：涉及近现代史（抗日、内战、建国后）的内容需注意表述方式
3. **旗帜与符号**：使用中国国旗、党旗、军徽等需符合相关法规
4. **免责声明**：建议在 Mod 描述中加入以下免责声明：

```
免责声明：
本 Mod 为游戏修改内容，仅供个人娱乐和学习用途。
Mod 中的历史事件、领土边界、政治设定均为游戏虚构内容，
不代表作者政治立场，亦不构成任何形式的政治主张。
如涉及领土表述问题，请以中国官方地图为准。

Disclaimer:
This mod is a game modification for personal entertainment and educational purposes only.
All historical events, territorial boundaries, and political settings are fictional game content.
They do not represent the author's political stance or constitute any form of political claim.
For territorial representation issues, please refer to official Chinese maps.
```

5. **法律底线**：绝不生成分裂国家、否定党的领导、歪曲历史的内容

> **注意**：以上为一般性提醒，不构成法律建议。如 mod 可能在 Steam Workshop 公开发布，建议自行咨询法律专业人士。

### 发布伦理
- 发布至 Steam Workshop 时，**建议**在描述中注明 AI 辅助程度
- 英文模板：`"Some content in this mod was developed with AI-assisted tools."`
- 中文模板：`"本 Mod 部分内容由 AI 辅助生成。"`
- 这是透明度的最佳实践，不强制但强烈建议

### 原版文件参考
- 原版 HOI4 安装路径（默认）：`D:\steam\steamapps\common\Hearts of Iron IV`
- 优先参考 `common/` 下的原版文件学习语法
- 推荐参考文件：`common/national_focus/germany.txt`（国策树）、`history/countries/GER - Germany.txt`（国家历史）

## ⑤ 外部工具

| 工具 | 用途 | 获取方式 |
|------|------|---------|
| **VS Code + CWTools 扩展** | 语法高亮、括号匹配、错误检查、自动补全 | VS Code Marketplace |
| **HOI4 Content Maker** | 可视化国策树/事件/决议编辑器 | GitHub / Paradox 论坛 |
| **Nudge** | 游戏内地图编辑器 | HOI4 自带，`-debug` 启动 |
| **WinMerge** | 文件对比，更新 mod 时合并 vanilla 变化 | 免费软件 |
| **HOI4 Validator** | 社区语法验证工具 | GitHub / 社区论坛 |
| **Git** | 版本控制 | git-scm.com |

工具优先级：**模板 > grep 原版 > Nudge/WinMerge > 手动**

## ⑥ 原版文件路径速查

在需要 grep 原版学习时，使用以下路径：

| 系统 | Vanilla 路径 | 推荐参考文件 |
|------|-------------|-------------|
| 国策树 | `common/national_focus/` | `germany.txt`, `usa.txt` |
| 事件 | `events/` | 搜索 `country_event` 或 `news_event` |
| 决议 | `common/decisions/` | 搜索 `decision = {` |
| 国家精神 | `common/ideas/` | `_economic.txt`, `_manpower.txt` |
| 国家历史 | `history/countries/` | `GER - Germany.txt` |
| 科技树 | `common/technologies/` | `industry.txt`, `infantry.txt` |
| 装备 | `common/units/equipment/` | `infantry_equipment.txt` |
| 地图 | `map/`, `history/states/` | `definition.csv` |
| 本地化 | `localisation/english/` | 搜索对应 key 名 |

常用 grep 命令模板：
```bash
HOI4="D:/steam/steamapps/common/Hearts of Iron IV"

# 查找国策树结构
grep -r "focus = {" "$HOI4/common/national_focus/" | head -20

# 查找事件结构
grep -r "country_event = {" "$HOI4/events/" | head -20

# 查找决议结构
grep -r "decision = {" "$HOI4/common/decisions/" | head -20

# 查找国家精神结构
grep -r "idea = {" "$HOI4/common/ideas/" | head -20
```

## ⑦ 子文件索引

### references/ — 知识库（按学习路径编号）

| 编号 | 文件 | 内容 |
|------|------|------|
| 01 | `01-env-setup.md` | 开发环境搭建（VS Code、HOI4 路径、启动参数） |
| 02 | `02-mod-structure.md` | Mod 项目结构（.mod 文件、目录布局、replace_path） |
| 03 | `03-pdxscript.md` | PDXscript 语法基础（作用域、触发器、效果、变量） |
| 04 | `04-focus-trees.md` | 国策树系统完整语法 |
| 05 | `05-events.md` | 事件系统完整语法 |
| 06 | `06-decisions.md` | 决议系统完整语法 |
| 07 | `07-ideas.md` | 国家精神 & 设计商完整语法 |
| 08 | `08-history.md` | 国家历史设定完整语法 |
| 09 | `09-technologies.md` | 科技树系统完整语法 |
| 10 | `10-equipment.md` | 装备 & 兵种系统完整语法 |
| 11 | `11-map-states.md` | 地图 & 省份系统完整语法 |
| 12 | `12-localisation.md` | 本地化系统完整语法 |
| 13 | `13-scripted.md` | 脚本化效果 & 触发器 |
| 14 | `14-debugging.md` | 调试 & 排错方法论 |
| 15 | `15-workshop.md` | Steam Workshop 发布流程 |
| 16 | `16-platforms.md` | 跨 AI 平台适配 |

### templates/ — 代码骨架

| 文件 | 用途 | 覆盖变体 |
|------|------|---------|
| `focus-tree.txt` | 国策树骨架 | 独占 + 共享 + 动态 |
| `event.txt` | 事件骨架 | country_event + news_event + MTTH + 触发型 |
| `decision.txt` | 决议骨架 | 普通 + 目标 + 定时 |
| `idea.txt` | 国家精神/顾问骨架 | 精神 + 政治顾问 + 设计公司 |
| `history.txt` | 国家历史骨架 | 起始设定 + 意识形态 + 外交 + OOB |
| `technology.txt` | 科技树骨架 | 单科技 + 子科技 + 多层级 |
| `equipment.txt` | 装备骨架 | 步兵装备 + 装甲 + 航空 |
| `localisation.yml` | 本地化骨架 | l_english + l_chinese |

### workflows/ — 任务流程

| 文件 | 用途 | 阶段 |
|------|------|------|
| `new-mod.md` | 从零创建 Mod | 环境检查→创建结构→测试→正式化 |
| `add-focus-tree.md` | 添加国策树 | 设计→文件→关联→本地化→测试 |
| `add-event-chain.md` | 添加事件链 | 故事线→事件→选项→串联→测试 |
| `add-country.md` | 添加新国家 | Tag→历史→国策→旗帜→本地化→测试 |
| `debug-crash.md` | 崩溃排查 | error.log→CTD诊断树→二分隔离 |
| `formalize-mod.md` | 正式化发布 | 检查→描述→截图→版本→上传 |
