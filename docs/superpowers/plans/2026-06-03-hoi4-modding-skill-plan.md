# HOI4 Modding Skill — 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在 `C:\Users\lxr20\.claude\skills\hoi4-modding\` 下创建完整的 HOI4 mod 制作 AI Skill，包含 1 个 SKILL.md、16 个 reference、8 个 template、6 个 workflow。

**Architecture:** 三层架构——SKILL.md 作为主入口协调 references/（知识库，按深度编号）、templates/（带注释 PDXscript 骨架）和 workflows/（步骤式任务流程）。二层决策用 grep 原版游戏文件替代 MCP。

**Tech Stack:** Markdown + PDXscript + YAML（仅 SKILL.md 前置元数据）

**Source of Truth:** 设计文档 `d:\WORK\docs\superpowers\specs\2026-06-03-hoi4-modding-skill-design.md`

---

## 文件结构总览

```
C:\Users\lxr20\.claude\skills\hoi4-modding\
├── SKILL.md
├── references/
│   ├── 01-env-setup.md
│   ├── 02-mod-structure.md
│   ├── 03-pdxscript.md
│   ├── 04-focus-trees.md
│   ├── 05-events.md
│   ├── 06-decisions.md
│   ├── 07-ideas.md
│   ├── 08-history.md
│   ├── 09-technologies.md
│   ├── 10-equipment.md
│   ├── 11-map-states.md
│   ├── 12-localisation.md
│   ├── 13-scripted.md
│   ├── 14-debugging.md
│   ├── 15-workshop.md
│   └── 16-platforms.md
├── templates/
│   ├── focus-tree.txt
│   ├── event.txt
│   ├── decision.txt
│   ├── idea.txt
│   ├── history.txt
│   ├── technology.txt
│   ├── equipment.txt
│   └── localisation.yml
└── workflows/
    ├── new-mod.md
    ├── add-focus-tree.md
    ├── add-event-chain.md
    ├── add-country.md
    ├── debug-crash.md
    └── formalize-mod.md
```

---

### Task 1: 创建目录结构并验证

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\` 及其所有子目录

- [ ] **Step 1: 创建目录结构**

```bash
mkdir -p "C:/Users/lxr20/.claude/skills/hoi4-modding/references"
mkdir -p "C:/Users/lxr20/.claude/skills/hoi4-modding/templates"
mkdir -p "C:/Users/lxr20/.claude/skills/hoi4-modding/workflows"
```

- [ ] **Step 2: 验证目录结构**

```bash
ls -R "C:/Users/lxr20/.claude/skills/hoi4-modding/"
```

预期输出：三个子目录（references, templates, workflows）均已存在。

---

### Task 2: SKILL.md — 主入口文件

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\SKILL.md`

**说明：** SKILL.md 是整个技能的大脑。它在 YAML 前置元数据中声明触发词，正文协调 references、templates 和 workflows 的使用。以下为完整内容：

- [ ] **Step 1: 写入 SKILL.md 完整内容**

```markdown
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

所有 mod 创建遵循此管线：

```
用户需求 → 生成测试版 Mod → 启动游戏测试 → 如有错误修复 → 验证通过 → 正式化
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
```

- [ ] **Step 2: 验证 SKILL.md 文件完整性**

```bash
wc -l "C:/Users/lxr20/.claude/skills/hoi4-modding/SKILL.md"
```

预期：行数 > 200。

- [ ] **Step 3: 检查 YAML 前置元数据格式**

```bash
head -20 "C:/Users/lxr20/.claude/skills/hoi4-modding/SKILL.md"
```

确认 `name:`、`description:` 字段存在，触发词列表完整。

---

### Task 3: References 01-03 — 基础层

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\01-env-setup.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\02-mod-structure.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\03-pdxscript.md`

**所有 reference 文件遵循统一头尾格式：**
- 文件头：`> **适用版本**: HOI4 1.18.2 | **最后更新**: 2026-06-03`
- 文件尾：`> **相关文件**: <交叉引用列表>`
- 章节顺序：概述 → 语法参考 → 实战示例 → 常见陷阱 → FAQ → 检查清单

- [ ] **Step 1: 写入 01-env-setup.md（环境搭建）**

```markdown
> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

搭建 HOI4 mod 开发环境，包括 VS Code + CWTools 扩展、HOI4 路径配置、启动参数和辅助工具安装。

## 开发环境配置

### 1. VS Code + CWTools 扩展

VS Code 是目前 HOI4 modding 的标准编辑器，配合 CWTools 扩展提供语法高亮、括号匹配、错误检查、自动补全和工具提示。

安装步骤：
1. 安装 VS Code（code.visualstudio.com）
2. 在扩展市场搜索 "CWTools" 并安装
3. 打开任意 .txt mod 文件，CWTools 自动激活

验证：打开一个原版国策文件（如 common/national_focus/germany.txt），应看到语法高亮和括号匹配。

### 2. HOI4 安装路径

默认 Steam 安装路径（Windows）：`D:\steam\steamapps\common\Hearts of Iron IV`

Mod 存放路径（Windows）：`C:\Users\<用户名>\Documents\Paradox Interactive\Hearts of Iron IV\mod\`

### 3. 启动参数

在 Steam 中设置启动参数或在桌面创建快捷方式：
- `-debug` — 开启调试模式，启用错误弹窗和控制台
- `-crash_data_log` — 崩溃时生成详细日志

### 4. 辅助工具

| 工具 | 用途 | 获取 |
|------|------|------|
| HOI4 Content Maker | 可视化编辑国策树/事件/决议 | GitHub |
| WinMerge | 对比文件差异 | winmerge.org |
| HOI4 Validator | 语法验证 | 社区论坛 |
| Git | 版本控制 | git-scm.com |

## 实战示例

### 示例 1：验证 CWTools 工作正常

1. 创建测试文件 test_mod/common/test.txt，写入：
   ```
   focus = {
       id = test_focus
       icon = GFX_test
   ```
   故意不闭合括号。

2. CWTools 应在编辑器中标注红色波浪线提示括号不匹配。

## 常见陷阱

1. **CWTools 不识别 mod 文件路径** — 确保在 VS Code 中打开了 mod 文件夹作为工作区根目录
2. **启动参数不生效** — 确认参数前有空格
3. **mod 目录不存在** — 至少运行一次 HOI4 才会自动创建 Documents 下的 mod 文件夹

## FAQ

**Q: 需要安装 Paradox 官方开发工具吗？**
A: 不需要。HOI4 使用纯文本脚本，任何文本编辑器都可以。

**Q: CWTools 提示的错误一定是错误吗？**
A: 不一定。CWTools 的规则库可能滞后于游戏更新，加载成功即为最终标准。

## 检查清单

- [ ] VS Code 已安装
- [ ] CWTools 扩展已安装并激活
- [ ] HOI4 安装路径确认
- [ ] Mod 目录路径确认
- [ ] `-debug` 启动参数已设置
```

- [ ] **Step 2: 写入 02-mod-structure.md（Mod 项目结构）**

```markdown
> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

每个 HOI4 mod 由两个必需部分组成：`.mod` 描述文件（告诉启动器关于 mod 的信息）和 mod 文件夹（镜像游戏内部目录结构）。理解这个结构是做 mod 的第一步。

## 完整语法参考

### .mod 描述文件

放在 `Documents/Paradox Interactive/Hearts of Iron IV/mod/` 目录下。

```
name = "Your Mod Name"
path = "mod/your_mod_folder"
tags = { "Alternative History" "Gameplay" }
supported_version = "1.18.*"
picture = "thumbnail.png"
```

可选字段：
- `replace_path = { "common/technologies" }` — 完全替换该路径，忽略 vanilla 文件（用于 total conversion mod）
- `user_dir = "YourMod"` — 创建独立的存档/配置目录
- `dependencies = { "Another Mod" }` — 依赖另一个 mod，确保加载顺序在后

### Mod 文件夹结构

Mod 文件夹镜像游戏内部目录结构。只需包含你要修改或新增的文件。

```
your_mod/
├── common/
│   ├── national_focus/     — 国策树文件
│   ├── decisions/          — 决议文件
│   │   └── categories/     — 决议分类
│   ├── ideas/              — 国家精神/设计商
│   ├── technologies/       — 科技树
│   ├── units/              — 装备/师编制
│   │   └── equipment/
│   ├── scripted_effects/   — 脚本化效果
│   └── scripted_triggers/  — 脚本化触发器
├── events/                 — 事件文件
├── history/
│   ├── countries/          — 国家起始设定
│   └── states/             — 省份/地区设定
├── localisation/
│   └── english/            — 英文文本（.yml）
├── gfx/
│   ├── flags/              — 国家旗帜
│   ├── leaders/            — 领导人/顾问肖像
│   └── interface/          — UI 图标和 GFX
└── map/                    — 地图文件（高级）
```

### 关键规则
- **文件路径必须完全匹配**游戏的内部路径
- **不要覆盖整个 vanilla 文件**——只添加新文件或新内容
- 游戏会加载目录下所有 .txt 文件，无需在某个地方注册
- 文件夹名和文件名区分大小写（Linux/Mac 用户）

## 实战示例

### 示例 1：最小可运行 Mod

1. 创建 `Documents/Paradox Interactive/Hearts of Iron IV/mod/mymod.mod`：
   ```
   name = "My First Mod"
   path = "mod/mymod"
   supported_version = "1.18.*"
   tags = { "Gameplay" }
   ```

2. 创建 `Documents/Paradox Interactive/Hearts of Iron IV/mod/mymod/` 文件夹

3. 在 HOI4 启动器中勾选该 mod

## 常见陷阱

1. **.mod 文件中 path 写错** — path 相对于 Documents 的 mod 文件夹
2. **mod 文件夹名与 .mod 文件中的 path 不一致**
3. **在没有 replace_path 的情况下覆盖整个文件** — 优先添加新文件
4. **目录结构层级错误** — 严格对照原版路径

## FAQ

**Q: 什么时候用 replace_path？**
A: 当你需要完全替代原版的某个系统（如所有科技、所有国策树），不希望原版文件被加载时。

**Q: 原版文件在哪？**
A: Steam 安装目录下的 Hearts of Iron IV/common/ 等文件夹中。

## 检查清单

- [ ] .mod 文件已创建且格式正确
- [ ] Mod 文件夹路径与 .mod 中 path 一致
- [ ] 目录结构镜像游戏内部路径
- [ ] 未修改原版 Steam 目录下的文件
- [ ] 启动器中可见并可勾选 mod
```

- [ ] **Step 3: 写入 03-pdxscript.md（PDXscript 语法基础）**

```markdown
> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

HOI4 使用 Paradox 自有的脚本语言（PDXscript），基于 `key = value` 和 `key = { ... }` 嵌套块结构。它是声明式的——你描述「有什么」和「什么时候触发」，而不是写过程式逻辑。

## 完整语法参考

### 基本元素

```
# 这是注释
key = value             # 简单赋值
key = { ... }           # 块/作用域
key = { value1 value2 } # 列表（空格分隔）
key = yes               # 布尔值
key = 100               # 数字
key = "string"          # 字符串
```

### 作用域系统

作用域决定了代码的「视角」——当前在操作哪个国家、哪个省份、哪个事件。

| 作用域 | 含义 | 示例用法 |
|--------|------|---------|
| `THIS` | 当前作用域 | `THIS = { add_stability = 0.1 }` |
| `FROM` | 上一个作用域 | 事件中 FROM 是触发者 |
| `ROOT` | 最顶层作用域 | 国策树中 ROOT 是拥有该树的国家 |
| `PREV` | 再上一个作用域 | 多层嵌套时逐层返回 |

### 触发器（Triggers）

放在 `trigger = { }`、`available = { }`、`allowed = { }` 块中。

```
# 复合逻辑
AND = { condition1 condition2 }    # 所有条件都满足
OR = { condition1 condition2 }     # 任一条件满足
NOT = { condition }                # 条件不满足

# 常用条件
has_political_power > 50
has_war = yes
date > 1939.1.1
controls_state = 123
has_technology = infantry_1
has_idea = my_national_spirit
exists = TAG
tag = GER
```

### 效果（Effects）

放在 `effect = { }`、`completion_reward = { }`、`complete_effect = { }` 块中。

```
# 常用效果
add_political_power = 100
add_stability = 0.05
add_war_support = 0.05
set_rule = { communist = 0.5 }
add_ideas = my_idea
remove_ideas = old_idea
declare_war_on = { target = TAG }
annex_country = { target = TAG }
set_country_flag = my_flag
country_event = { id = my_event.1 days = 7 }
```

### 条件分支

```
if = {
    limit = { has_war = yes }
    add_stability = -0.05
}
else_if = {
    limit = { has_war = no }
    add_stability = 0.05
}
else = {
    add_political_power = 50
}
```

### 变量系统

```
set_variable = { var = my_var value = 10 }
change_variable = { var = my_var value = 5 }
check_variable = { var = my_var value = 10 compare = greater_than }
```

## 实战示例

### 示例 1：一个简单的触发器

```
available = {
    has_political_power > 50
    has_war = no
    NOT = { has_idea = my_spirit }
}
```
效果：需要政治点数 > 50、不在战争状态、且没有 `my_spirit` 国家精神。

### 示例 2：一个带分支的效果

```
complete_effect = {
    if = {
        limit = { has_war = yes }
        add_war_support = 0.05
    }
    else = {
        add_stability = 0.05
    }
}
```
效果：如果在战争中 +5% 战争支持度，否则 +5% 稳定度。

## 常见陷阱

1. **括号不匹配** — 缺少闭合 `}` 是最常见的语法错误，导致文件无法加载
2. **作用域混淆** — 不确定当前是哪个国家/省份时，明确使用 THIS/FROM/ROOT
3. **trigger 和 effect 位置错误** — trigger 放效果块，effect 放触发器块，都不会生效
4. **列表格式错误** — 空格分隔的列表不能换行，除非用引号
5. **编码错误** — .txt 必须 UTF-8 no-BOM

## FAQ

**Q: if/else 和 trigger 有什么区别？**
A: trigger 控制行为是否可用（button 是否能点击），if/else 控制行为内的执行分支。

**Q: 如何调试作用域问题？**
A: 用 `log = "DEBUG: [THIS.GetName]"` 在游戏日志中打印当前作用域名称，需要 -debug 模式。

## 检查清单

- [ ] 理解 key = value 和 block = {} 语法
- [ ] 理解 THIS/FROM/ROOT/PREV 四个作用域
- [ ] 能区分 trigger 和 effect 的使用位置
- [ ] 了解 if/else_if/else 条件分支语法
```

- [ ] **Step 4: 验证三个文件创建成功**

```bash
ls -la "C:/Users/lxr20/.claude/skills/hoi4-modding/references/01-env-setup.md"
ls -la "C:/Users/lxr20/.claude/skills/hoi4-modding/references/02-mod-structure.md"
ls -la "C:/Users/lxr20/.claude/skills/hoi4-modding/references/03-pdxscript.md"
```

预期：三个文件均存在且大小 > 1KB。

---

### Task 4: References 04-06 — 核心系统：国策树、事件、决议

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\04-focus-trees.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\05-events.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\06-decisions.md`

- [ ] **Step 1: 写入 04-focus-trees.md（国策树系统）**

文件头 `> **适用版本**: HOI4 1.18.2 | **最后更新**: 2026-06-03`，内容覆盖：

```
## 概述
国策树是 HOI4 最核心的系统。每个国家可以有一个国策树，玩家通过完成国策来推进国家发展。

## 完整语法参考
- focus_tree = { } 块（id, country, default, reset_on_civil_war）
- focus = { } 节点（id, icon, x, y, cost, prerequisites, mutually_exclusive, available, bypass, cancel, completion_reward, ai_will_do, search_filters）
- 共享国策树（shared_focus = TAG_focus）
- 动态国策（dynamic = yes）

## 实战示例
- 示例 1：一个三节点线形国策树
- 示例 2：含独占分支的国策树

## 常见陷阱
## FAQ
## 检查清单

> **相关文件**: completion_reward 可触发事件（05-events.md），可通过效果添加国家精神（07-ideas.md）
```

- [ ] **Step 2: 写入 05-events.md（事件系统）**

文件头同上，内容覆盖：

```
## 概述
事件是 HOI4 中最灵活的叙事工具。可以是随机触发的（MTTH）、由国策/决议触发的、或通过触发器条件自动触发的。

## 完整语法参考
- country_event / news_event / state_event / unit_leader_event
- mean_time_to_happen（随机触发）
- is_triggered_only（强制触发）
- option = { } 块（name, trigger, effect, ai_chance）
- hidden = yes（后台事件）
- fire_only_once = yes
- major = yes（弹窗事件）

## 实战示例
- 示例 1：一个 MTTH 随机事件
- 示例 2：一个国策触发的三选项事件链
- 示例 3：一个 hidden 后台事件（自动检查条件并执行效果）

## 常见陷阱
## FAQ
## 检查清单

> **相关文件**: 通常由国策 completion_reward 触发（04-focus-trees.md），option 效果可添加决议或国家精神（06-decisions.md, 07-ideas.md）
```

- [ ] **Step 3: 写入 06-decisions.md（决议系统）**

文件头同上，内容覆盖：

```
## 概述
决议是玩家手动触发的行动——需要满足条件、消耗资源、产生效果。比事件更主动，比国策更轻量。

## 完整语法参考
- decision = { } 块（allowed, visible, available, cost, days_remove, remove_effect, cancel_trigger, complete_effect, ai_will_do, fire_only_once）
- targeted_decision 目标决议
- decision_category 分类定义
- on_map 地图决议

## 实战示例
- 示例 1：一个简单的政治决议
- 示例 2：一个定时决议（消耗天数后完成）

## 常见陷阱
## FAQ
## 检查清单

> **相关文件**: 可通过效果触发事件（05-events.md），决议效果可修改国家精神（07-ideas.md）
```

- [ ] **Step 4: 验证三个文件创建成功**

---

### Task 5: References 07-08 — 核心系统：国家精神、国家历史

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\07-ideas.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\08-history.md`

- [ ] **Step 1: 写入 07-ideas.md（国家精神 & 设计商）**

内容覆盖：idea 块语法、modifier 字段、political_advisor/high_command/theorist/design_team 类型、动态 modifier（add_ideas, remove_ideas, swap_ideas）、槽位系统。含 2 个实战示例。

- [ ] **Step 2: 写入 08-history.md（国家历史设定）**

内容覆盖：历史文件结构、ruling_party 与 ideology、起始科技/装备/师编制、外交关系、OOB 设定、Tag 命名规范。含 2 个实战示例。

- [ ] **Step 3: 验证文件创建成功**

---

### Task 6: References 09-11 — 进阶系统：科技、装备、地图

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\09-technologies.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\10-equipment.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\11-map-states.md`

- [ ] **Step 1: 写入 09-technologies.md（科技树系统）**

内容覆盖：technology 块语法、categories/tiers/prerequisites、research_cost、ahead_of_time_penalty、sub_technologies、on_research_complete 效果。含 2 个实战示例。

- [ ] **Step 2: 写入 10-equipment.md（装备 & 兵种系统）**

内容覆盖：archetype 范型、equipment 块（stats, production_cost, resources）、subunit 定义（combat_width, manpower, equipment_needs）、步兵/装甲/航空/海军装备类型差异、upgrade 路径。含 2 个实战示例。

- [ ] **Step 3: 写入 11-map-states.md（地图 & 省份系统）**

内容覆盖：provinces.bmp/definition.csv 格式、state 块（resources, buildings, victory_points, manpower）、strategic_region、Nudge 地图编辑器使用、铁路/补给节点/地形、地图修改的 CTD 常见陷阱。含 2 个实战示例。

- [ ] **Step 4: 验证文件创建成功**

---

### Task 7: References 12-14 — 辅助层：本地化、脚本化、调试

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\12-localisation.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\13-scripted.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\14-debugging.md`

- [ ] **Step 1: 写入 12-localisation.md（本地化系统）**

内容覆盖：.yml 文件格式（l_english/l_chinese 等多语言）、key:"text" 键值对、动态变量（§ 颜色代码、[FROM.GetName] 等）、scripted_loc 脚本化本地化、UTF-8 BOM 编码——最常见错误。含 2 个实战示例。

- [ ] **Step 2: 写入 13-scripted.md（脚本化效果 & 触发器）**

内容覆盖：scripted_effect 可复用效果块、scripted_trigger 可复用条件块、参数传递与调用约定、实战模式（全局变量、通用检查）。含 2 个实战示例。

- [ ] **Step 3: 写入 14-debugging.md（调试 & 排错）**

内容覆盖：debug 模式启动（-debug 参数）、错误日志位置与 reading（error.log, game.log）、CTD 诊断流程、常见错误速查表（括号不匹配、编码错误、路径错误、作用域混淆）、二分法隔离测试、开发者控制台命令（tdebug, reloadfocus, reload events, tag）。含完整的故障排除决策树。

- [ ] **Step 4: 验证文件创建成功**

---

### Task 8: References 15-16 — 发布层：Workshop、跨平台

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\15-workshop.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\references\16-platforms.md`

- [ ] **Step 1: 写入 15-workshop.md（Steam Workshop 发布）**

内容覆盖：发布前检查清单（tag、图片、描述、版本号）、手动上传 vs Launcher 上传、语义化版本控制、descriptor.mod 更新最佳实践、兼容性声明与更新流程。**必须包含 AI 辅助声明小节**（嵌入位置、中英模板文案、措辞原则）。

- [ ] **Step 2: 写入 16-platforms.md（跨平台适配）**

内容覆盖：Claude Code / Copilot CLI / Codex / Gemini CLI / Cursor / Windsurf 六平台工具映射表、各平台 Skill 安装路径、无 MCP 场景的 grep 原版文件回退方案、平台间差异说明。

- [ ] **Step 3: 验证文件创建成功**

---

### Task 9: Templates 1-4 — 国策树、事件、决议、国家精神

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\templates\focus-tree.txt`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\templates\event.txt`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\templates\decision.txt`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\templates\idea.txt`

**模板格式规范：**
- 文件头：`# <模板名> —— <用途>` + `# 验证状态: <说明>` + `# Vanilla 参考: <路径>` + `# 设计决策: <关键选择说明>`
- 关键字段中文行内注释，标注允许值、单位、默认值
- `<YourXxx>` 占位符
- 覆盖 2-3 种常见变体
- 文件末尾保留空行

- [ ] **Step 1: 写入 focus-tree.txt**

文件头注释块 + 完整 PDXscript 骨架。覆盖三种变体：独占国策（mutually_exclusive）、共享国策（shared_focus）、动态国策（dynamic = yes）。关键字段标注注释。

- [ ] **Step 2: 写入 event.txt**

文件头注释块 + 完整 PDXscript 骨架。覆盖四种变体：country_event（含 MTTH 和 is_triggered_only 各一个）、news_event、hidden 事件。

- [ ] **Step 3: 写入 decision.txt**

文件头注释块 + 完整 PDXscript 骨架。覆盖三种变体：普通决议、目标决议（targeted_decision）、定时决议（days_remove）。

- [ ] **Step 4: 写入 idea.txt**

文件头注释块 + 完整 PDXscript 骨架。覆盖三种变体：国民精神（含 modifier 样板）、政治顾问（political_advisor）、设计公司（design_team）。

- [ ] **Step 5: 验证四个模板文件创建成功**

---

### Task 10: Templates 5-8 — 国家历史、科技、装备、本地化

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\templates\history.txt`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\templates\technology.txt`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\templates\equipment.txt`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\templates\localisation.yml`

- [ ] **Step 1: 写入 history.txt**

文件头注释块 + 完整 PDXscript 骨架。覆盖：国家起始设定（ruling_party, ideology, flags, stability, war_support）、外交关系、OOB 设定。

- [ ] **Step 2: 写入 technology.txt**

文件头注释块 + 完整 PDXscript 骨架。覆盖：单层级科技、多层级科技（含子科技）、categories 组织。

- [ ] **Step 3: 写入 equipment.txt**

文件头注释块 + 完整 PDXscript 骨架。覆盖：步兵装备 archetype（一个完整样板）、装甲 archetype（精简）、航空 archetype（精简）。

- [ ] **Step 4: 写入 localisation.yml**

文件头注释块（用 # 格式） + 完整 .yml 骨架。覆盖 l_english 和 l_chinese 双语，含动态变量示例（§ 颜色、[FROM.GetName]）。

**注意：** .yml 文件必须使用 UTF-8 with BOM 编码。写入时需确保正确的编码和文件头格式（`l_english:` 开头）。

- [ ] **Step 5: 验证四个模板文件创建成功**

---

### Task 11: Workflows 1-3 — 新建 Mod、添加国策树、添加事件链

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\workflows\new-mod.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\workflows\add-focus-tree.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\workflows\add-event-chain.md`

**工作流文件结构：** 概述 → 阶段一 → 阶段二：测试与修复循环 → 阶段三 → 检查清单

- [ ] **Step 1: 写入 new-mod.md（从零创建 Mod）**

阶段：
- 阶段一：环境检查（确认 HOI4 安装路径、mod 目录位置）
- 阶段二：创建结构（.mod 文件 + 目录 + 最小测试内容）
- 阶段三：测试与修复（启动游戏验证 mod 加载）
- 阶段四：正式化（清理测试内容、准备生产文件）
- 检查清单

- [ ] **Step 2: 写入 add-focus-tree.md（添加国策树）**

阶段：
- 阶段一：设计国策树（节点规划、分支设计、数值平衡）
- 阶段二：创建文件（使用 focus-tree 模板生成）
- 阶段三：关联国家（history/countries 中添加国策树引用）
- 阶段四：添加本地化（所有节点名称和描述的 .yml 条目）
- 阶段五：测试与修复（加载测试、检查节点显示和效果执行）
- 检查清单

- [ ] **Step 3: 写入 add-event-chain.md（添加事件链）**

阶段：
- 阶段一：设计故事线（事件流程、分支、选项）
- 阶段二：编写事件（使用 event 模板，设置触发条件）
- 阶段三：编写选项（每个 option 的 effect）
- 阶段四：串联触发（事件触发下一个事件、国策触发事件）
- 阶段五：添加本地化
- 阶段六：测试与修复
- 检查清单

- [ ] **Step 4: 验证三个工作流文件创建成功**

---

### Task 12: Workflows 4-6 — 添加国家、崩溃排查、正式化发布

**Files:**
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\workflows\add-country.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\workflows\debug-crash.md`
- Create: `C:\Users\lxr20\.claude\skills\hoi4-modding\workflows\formalize-mod.md`

- [ ] **Step 1: 写入 add-country.md（添加新国家）**

阶段：
- 阶段一：选择 Tag（3 个大写字母，检查不与 vanilla 和其他 mod 冲突）
- 阶段二：创建历史文件（ideology, ruling_party, stability, war_support, 外交）
- 阶段三：创建国策树（可先用最简版本，后续扩展）
- 阶段四：创建旗帜（.tga 格式，三个变体：标准/法西斯/共产主义）
- 阶段五：添加本地化（国家名、执政党名、国策名）
- 阶段六：测试与修复
- 检查清单

- [ ] **Step 2: 写入 debug-crash.md（崩溃排查）**

阶段：
- 阶段一：定位错误来源（读 error.log，确认是哪个文件/哪个 ID）
- 阶段二：CTD 诊断决策树（检查编码→检查括号→检查路径→检查作用域→用二分法隔离）
- 阶段三：二分法隔离（移除一半内容测试→缩小范围→定位具体行）
- 阶段四：常见错误速查表（error code 对应原因和解决方案）
- 阶段五：寻求帮助时需提供的信息模板
- 检查清单

- [ ] **Step 3: 写入 formalize-mod.md（正式化 & 发布）**

阶段：
- 阶段一：命名检查（packageId、名称、Tag 统一为正式命名）
- 阶段二：补全 About.xml / .mod（author, version, supported_version, tags, description）
- 阶段三：描述与截图（描述撰写指南 + **AI 辅助声明检查项** + Steam 预览图规格）
- 阶段四：版本号设置（语义化版本 major.minor.patch）
- 阶段五：上传至 Steam Workshop
- 阶段六：Git tag 与管理
- 检查清单（**含 `[ ] 已添加 AI 辅助声明`**）

- [ ] **Step 4: 验证三个工作流文件创建成功**

---

### Task 13: 最终验证

- [ ] **Step 1: 确认所有文件已创建（31 个文件）**

```bash
find "C:/Users/lxr20/.claude/skills/hoi4-modding" -type f | wc -l
```

预期：31（1 SKILL.md + 16 references + 8 templates + 6 workflows）

- [ ] **Step 2: 确认 SKILL.md 触发词完整**

检查 `C:\Users\lxr20\.claude\skills\hoi4-modding\SKILL.md` 的 YAML 前置元数据中触发词列表是否包含所有设计文档指定的关键词。

- [ ] **Step 3: 确认所有 reference 文件有统一头尾格式**

```bash
grep -l "适用版本: HOI4 1.18.2" "C:/Users/lxr20/.claude/skills/hoi4-modding/references/"*.md | wc -l
```

预期：16（所有 reference 文件均有版本头）

- [ ] **Step 4: 确认交叉引用完整**

```bash
grep -l "相关文件" "C:/Users/lxr20/.claude/skills/hoi4-modding/references/"*.md | wc -l
```

预期：16

- [ ] **Step 5: 确认模板文件有文件头注释**

```bash
grep -l "验证状态" "C:/Users/lxr20/.claude/skills/hoi4-modding/templates/"*
```

预期：8 个文件均在输出中

- [ ] **Step 6: 确认 AI 辅助声明已嵌入三个位置**

```bash
grep -l "AI 辅助" "C:/Users/lxr20/.claude/skills/hoi4-modding/SKILL.md" \
            "C:/Users/lxr20/.claude/skills/hoi4-modding/references/15-workshop.md" \
            "C:/Users/lxr20/.claude/skills/hoi4-modding/workflows/formalize-mod.md"
```

预期：三个文件全部匹配

- [ ] **Step 7: 目录完整结构检查**

```bash
ls -R "C:/Users/lxr20/.claude/skills/hoi4-modding/"
```

对照设计文档 2.1 节文件结构，确认无一遗漏。

---

## 自审结果

1. **Spec 覆盖**：13 个 Task 覆盖了设计文档中的所有部分——SKILL.md 七大区块、16 个 reference、8 个 template、6 个 workflow、AI 辅助声明三处嵌入、编码规则验证
2. **无占位符**：所有任务均包含实际文件路径和内容规格
3. **类型一致性**：文件路径统一使用 `C:\Users\lxr20\.claude\skills\hoi4-modding\` 前缀，命名规范一致
