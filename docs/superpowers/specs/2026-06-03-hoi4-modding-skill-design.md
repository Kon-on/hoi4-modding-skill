# HOI4 Modding Skill — 设计文档

> **日期**: 2026-06-03  
> **版本**: v1.0  
> **目标**: 面向零基础用户，覆盖 HOI4 1.18.2 全流程 mod 制作的 Claude Code Skill

---

## 1. 概述

### 1.1 项目定位

创建一个覆盖 Hearts of Iron 4 (HOI4) 全流程 mod 制作的 AI Skill，采用与 rimworld-modding 技能相同的三层架构（知识库 + 代码骨架 + 任务流程），面向零基础用户，支持从环境搭建到 Steam Workshop 发布的完整链路。

### 1.2 市场空白

经网络搜索确认，目前**不存在**同类项目：
- 没有现成的 HOI4 modding AI Skill（Claude Code、Copilot CLI、Codex 等平台均无）
- 没有 Paradox 游戏（HOI4/CK3/EU4/Stellaris）的 AI 辅助 modding Skill
- 社区中有可视化工具（HOI4 Content Maker、Focus Tree Creation Tool）但没有 LLM 驱动的智能辅助工具

这是一个全新的原创项目。

### 1.3 设计参考

基于已验证的 rimworld-modding 技能架构，适配 HOI4 modding 的以下关键差异：

| 维度 | RimWorld 1.6 | HOI4 1.18.2 |
|------|-------------|-------------|
| 脚本语言 | XML | PDXscript（自定义语法） |
| 编码规则 | UTF-8 | .txt UTF-8 no-BOM / .yml UTF-8 BOM |
| 外部查询 | MCP 服务器 | 无 MCP，grep 原版文件 |
| 系统耦合 | 低（各 Def 相对独立） | 高（国策树→事件→决议→国家精神互联） |
| 工具链 | IDE + 反编译器 | VS Code+CWTools, Nudge, Content Maker |

---

## 2. 架构设计

### 2.1 文件结构

```
hoi4-modding/
├── SKILL.md                        # 主入口：决策树、快速导航、原则
├── references/                     # 知识库（16 个文件，按深度递增编号）
│   ├── 01-env-setup.md             # 环境搭建
│   ├── 02-mod-structure.md         # Mod 项目结构
│   ├── 03-pdxscript.md             # PDXscript 语法基础
│   ├── 04-focus-trees.md           # 国策树系统
│   ├── 05-events.md                # 事件系统
│   ├── 06-decisions.md             # 决议系统
│   ├── 07-ideas.md                 # 国家精神 & 设计商
│   ├── 08-history.md               # 国家历史设定
│   ├── 09-technologies.md          # 科技树系统
│   ├── 10-equipment.md             # 装备 & 兵种系统
│   ├── 11-map-states.md            # 地图 & 省份系统
│   ├── 12-localisation.md          # 本地化系统
│   ├── 13-scripted.md              # 脚本化效果 & 触发器
│   ├── 14-debugging.md             # 调试 & 排错
│   ├── 15-workshop.md              # Steam Workshop 发布
│   └── 16-platforms.md             # 跨平台适配
├── templates/                      # 代码生成骨架（8 个）
│   ├── focus-tree.txt              # 国策树模板
│   ├── event.txt                   # 事件模板
│   ├── decision.txt                # 决议模板
│   ├── idea.txt                    # 国家精神模板
│   ├── history.txt                 # 国家历史模板
│   ├── technology.txt              # 科技树模板
│   ├── equipment.txt               # 装备模板
│   └── localisation.yml            # 本地化模板
└── workflows/                      # 任务导向流程（6 个）
    ├── new-mod.md                  # 从零创建 Mod
    ├── add-focus-tree.md           # 添加国策树
    ├── add-event-chain.md          # 添加事件链
    ├── add-country.md              # 添加国家
    ├── debug-crash.md              # 崩溃排查
    └── formalize-mod.md            # 正式化 & 发布
```

**总计约 30 个文件**。

### 2.2 核心决策树（三层系统）

```
用户请求
  │
  ├─ 1. 有模板？
  │     └─ 直接用模板生成（零外部调用，最快路径）
  │
  ├─ 2. 无模板的新类型？
  │     └─ grep 原版游戏文件 → 学习 vanilla 写法 → 生成内容
  │        → 存为新模板 → 更新 SKILL.md 索引
  │        （原版路径：<HOI4安装目录>/common/, events/, history/）
  │
  └─ 3. 报错/调试？
        └─ 读 error.log → 查 references/14-debugging
           → grep 原版对比 → 二分法隔离定位
```

与 rimworld-modding 的关键区别：第二层使用 `grep` 搜索原版游戏文件，因为 HOI4 没有对应的 MCP 服务器。原版游戏文件本身就是最好的参考文档。

### 2.3 模板自扩展循环

```
用户请求新类型 → 无对应模板
  → grep 原版文件找到类似结构
  → 基于 vanilla 生成带注释的模板
  → Write 到 templates/
  → Edit SKILL.md 更新两处（对照表 + 索引）
  → 下次同类型请求直接用模板，无需重复搜索
```

---

## 3. SKILL.md 主文件设计

### 3.1 YAML 前置元数据

```yaml
name: hoi4-modding
description: |
  Hearts of Iron 4 mod 制作全流程指南——覆盖环境搭建、PDXscript 语法、
  国策树、事件、决议、国家精神、国家历史、科技树、装备、地图、本地化、
  调试排错和 Steam Workshop 发布。
  适用于零基础到进阶的 HOI4 1.18 模组开发者。

触发词：HOI4, Hearts of Iron, 钢铁雄心, hoi4, mod, Mod, 模组,
  国策树, 事件, 决议, 国家精神, 科技树, 装备, 地图, 本地化,
  PDXscript, focus tree, event, decision, national spirit,
  Steam Workshop, 创意工坊, 编译
```

### 3.2 七大区块

| 序号 | 区块 | 内容 |
|------|------|------|
| ① | 核心决策 | 三层决策树流程图 + 模板对照表 |
| ② | 核心工作流 | 测试版先行管线（生成→测试→修复→验证→正式化） |
| ③ | 快速导航 | 「我想做 X」→ 需要哪些文件的对照表 |
| ④ | 核心原则 | 命名规范、编码规则、版本锁定、安全实践、发布伦理 |
| ⑤ | 外部工具 | VS Code+CWTools、Nudge、HOI4 Content Maker、WinMerge、Git |
| ⑥ | 原版文件参考 | 每个系统对应的 vanilla 文件路径 + grep 命令模板 |
| ⑦ | 子文件索引 | references/、templates/、workflows/ 完整列表 |

### 3.3 快速导航对照表（示例）

| 用户需求 | 参考文件 | 模板 | 工作流 |
|----------|---------|------|--------|
| 新建 Mod 项目 | 01, 02 | — | new-mod |
| 添加国策树 | 04 | focus-tree.txt | add-focus-tree |
| 添加事件链 | 05 | event.txt | add-event-chain |
| 添加决议 | 06 | decision.txt | — |
| 添加国家 | 08 | history.txt | add-country |
| 修改原版内容 | 02, 14 | — | — |
| 添加科技树 | 09 | technology.txt | — |
| 添加装备 | 10 | equipment.txt | — |
| 添加国家精神 | 07 | idea.txt | — |
| 崩溃/红字排错 | 14 | — | debug-crash |
| 发布到 Steam | 15 | — | formalize-mod |

---

## 4. Reference 文件设计

### 4.1 编号逻辑

- **01-03**：基础层（环境、结构、语法）—— 所有后续系统的前提
- **04-08**：核心系统层（国策树、事件、决议、国家精神、历史）—— 最常见的 mod 类型
- **09-11**：进阶系统层（科技、装备、地图）—— 技术难度更高
- **12-14**：辅助层（本地化、脚本化、调试）—— 贯穿所有系统的支撑
- **15-16**：发布层（Workshop、跨平台适配）

### 4.2 每个文件的统一结构

```
> 适用版本: HOI4 1.18.2  |  最后更新: 2026-06-03

## 概述（2-3 句话说明该系统的作用）

## 完整语法参考（核心内容）
  - 每个语法元素含 PDXscript 代码示例
  - 关键字段标注允许值、单位、默认值
  - 中文注释解释每个块的作用

## 实战示例（2-3 个从简单到复杂的完整例子）

## 常见陷阱（3-5 条）
  - 编码错误（UTF-8 BOM/no-BOM 混淆）
  - 括号不匹配
  - 路径错误
  - 作用域混淆（THIS/FROM/ROOT/PREV）
  - 跨系统引用断裂

## FAQ（3-5 条）

## 检查清单（3-5 项）
```

### 4.3 交叉引用

由于 HOI4 系统间耦合度远高于 RimWorld，每个 reference 文件末尾标注：

```
> 相关文件: 国策树的 completion_reward 可触发事件（详见 05-events.md），
>   事件 option 可添加国家精神（详见 07-ideas.md）
```

### 4.4 原版文件路径速查

在 SKILL.md 和 01-env-setup.md 中包含 vanilla 文件路径表：

| 系统 | Vanilla 路径 | 推荐参考文件 |
|------|-------------|-------------|
| 国策树 | common/national_focus/ | germany.txt, usa.txt |
| 事件 | events/ | various |
| 决议 | common/decisions/ | various |
| 国家精神 | common/ideas/ | _economic.txt, _manpower.txt |
| 国家历史 | history/countries/ | GER - Germany.txt |
| 科技树 | common/technologies/ | industry.txt, infantry.txt |
| 装备 | common/units/equipment/ | infantry_equipment.txt |
| 地图 | map/, history/states/ | definition.csv |
| 本地化 | localisation/english/ | various |

---

## 5. Templates 设计

### 5.1 模板格式规范

每个模板文件遵循：

```
# <模板名称> —— <用途>
# 验证状态: 已对照 vanilla <文件> 验证
# Vanilla 参考: <路径>
# 设计决策: <关键结构选择的原因>

<完整的 PDXscript 骨架代码，带中文注释>
```

关键原则：
- `<YourXxx>` 占位符用于所有用户自定义值
- 关键字段附加中文行内注释，包含允许值范围
- 每个模板覆盖该类型的 2-3 种常见变体（如 event.txt 含 country_event 和 news_event 两种）
- 文件末尾保留一个空行

### 5.2 8 个模板清单

| 模板 | 覆盖的变体 | Vanilla 参考 |
|------|-----------|-------------|
| focus-tree.txt | 独占国策 + 共享国策 + 动态国策 | germany.txt |
| event.txt | country_event + news_event + MTTH + 触发型 | events/ 多个文件 |
| decision.txt | 普通决议 + 目标决议 + 定时决议 | decisions/ 多个文件 |
| idea.txt | 国民精神 + 政治顾问 + 设计公司 | ideas/ 多个文件 |
| history.txt | 国家起始 + 意识形态 + 外交 + OOB | history/countries/* |
| technology.txt | 科技定义 + 子科技 + 多层级 | technologies/ 多个文件 |
| equipment.txt | 步兵装备 + 装甲 + 航空（各一样板） | units/equipment/ |
| localisation.yml | l_english + l_chinese 双语 | localisation/english/ |

---

## 6. Workflows 设计

### 6.1 工作流结构

每个 workflow 文件遵循：

```
# <工作流名称>

## 概述（适用场景、前置条件）

## 阶段一：<阶段名>
  - 步骤 1.1
  - 步骤 1.2
  - ...

## 阶段二：测试与修复循环
  - 启动游戏测试
  - 如有错误 → 查 error.log → 修复 → 再测试
  - 循环直到无错误

## 阶段三：<最终阶段>
  - ...

## 检查清单（3-5 项可勾选）
```

### 6.2 6 个工作流清单

| 工作流 | 阶段 | 适用场景 |
|--------|------|---------|
| new-mod.md | 环境检查→创建结构→测试→正式化 | 从零开始 |
| add-focus-tree.md | 设计→文件→关联→本地化→测试 | 添加或修改国策树 |
| add-event-chain.md | 故事线→事件→选项→串联→测试 | 添加事件链 |
| add-country.md | Tag→历史→国策→旗帜→本地化→测试 | 添加新国家 |
| debug-crash.md | error.log定位→CTD诊断树→二分隔离 | 任何崩溃/错误 |
| formalize-mod.md | 检查→描述→截图→版本→上传 | 发布前最终检查 |

---

## 7. AI 辅助声明

在发布层面嵌入 AI 辅助透明声明：

### 7.1 嵌入位置

1. **SKILL.md ④ 核心原则** — 新增「发布伦理」条目：建议在描述中注明 AI 辅助
2. **references/15-workshop.md** — 描述撰写小节提供中英模板文案
3. **workflows/formalize-mod.md** — 发布前检查清单中加入 `[ ] 已添加 AI 辅助声明`

### 7.2 推荐模板文案

- 中文：`"本 Mod 部分内容由 AI 辅助生成。"`
- 英文：`"Some content in this mod was developed with AI-assisted tools."`

### 7.3 措辞原则

- 始终使用「建议」而非「必须」，尊重创作者自由
- 提供默认文案但允许修改
- 中英双语以覆盖 Steam Workshop 国际用户

---

## 8. 技术要点

### 8.1 编码规则（最高频错误）

- `.txt` 文件：UTF-8 **without** BOM
- `.yml` 文件：UTF-8 **with** BOM
- 编码错误 = 游戏崩溃或文本不显示（error code 3221225477）

### 8.2 HOI4 版本锁定

- 目标版本：**HOI4 1.18.2**（2026-05-20 发布，"Peace For Our Time" DLC 周期）
- 所有模板和 reference 均基于此版本验证

### 8.3 跨平台

- Skill 安装在 `~/.claude/skills/hoi4-modding/`
- Skill 本体知识平台中立
- 16-platforms.md 提供各 AI 平台工具映射
- 无 MCP 场景直接 grep 原版文件，不依赖外部服务

---

## 9. 后续扩展规划

### 9.1 v1.0 范围（当前设计）

- 16 个 reference 文件覆盖所有主要 modding 系统
- 8 个模板覆盖最常见的内容类型
- 6 个工作流覆盖完整开发生命周期

### 9.2 v1.1+ 可能扩展

- 界面/GUI modding（interface/*.gui 文件）
- 音乐/音效添加
- 多人游戏兼容性
- 子 mod 和补丁 mod 工作流
- 性能优化（AI 计算负载、事件频率控制）
- 自动化测试脚本（批量验证语法）

---

## 10. 成功标准

- [ ] 零基础用户能跟随 new-mod 工作流创建第一个可运行的 mod
- [ ] 所有模板生成的代码通过 CWTools 语法检查
- [ ] 所有模板中的示例在 HOI4 1.18.2 中无错误加载
- [ ] 每个 reference 文件包含至少 2 个不同复杂度的实战示例
- [ ] Skill 在 Claude Code、Copilot CLI、Codex 三平台可用
