> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

调试是 mod 开发中耗时最长的环节之一。HOI4 提供了多个诊断工具：`-debug` 启动参数（启用控制台和详细日志）、`error.log`（错误记录）、`game.log`（完整运行记录）和丰富的控制台命令。本章提供一套完整的排错方法论——从快速定位到深度隔离，覆盖启动崩溃（CTD）、红字报错、逻辑错误三大类问题。

## 完整语法参考

### Debug 模式启动

**Steam 启动参数：** Steam 库 → 右键 Hearts of Iron IV → 属性 → 通用 → 启动选项：

```
-debug -crash_data_log -no_intro
```

| 参数 | 作用 |
|------|------|
| `-debug` | 启用控制台（~ 键）、错误弹窗、详细日志、`tdebug` 等调试命令 |
| `-crash_data_log` | 崩溃时生成详细堆栈和异常信息（写入 `logs/exceptions.log`） |
| `-no_intro` | 跳过开场动画，加快测试循环 |

### 日志文件位置和用途

所有日志文件位于 `Documents\Paradox Interactive\Hearts of Iron IV\logs\`：

```
Documents\Paradox Interactive\Hearts of Iron IV\logs\
├── error.log           # 错误日志——最常用的排错入口
├── game.log            # 完整运行记录——启动/加载/事件/错误全记录
├── exceptions.log      # 崩溃堆栈记录（需 -crash_data_log 参数）
├── ai.log              # AI 决策记录（仅在启用 AI 日志时生成）
├── graphics.log        # 图形/渲染错误
├── memory.log          # 内存占用记录
├── network.log         # 多人游戏连接日志
├── script.log          # 脚本执行追踪
├── setup.log           # Mod 加载顺序和路径解析记录
└── time.log            # 各阶段耗时统计
```

### error.log 解读

`error.log` 是每次启动/测试后首先检查的文件。其内容分为以下几类：

**语法错误（最常见）：**

```
[09:35:22][pdx_parser.cpp:123]: Syntax error near '}' in common/national_focus/my_focus.txt, line 45
[09:35:22][pdx_parser.cpp:456]: Unexpected token '==', expected '=' at line 67
```
含义：文件有语法错误（括号、运算符、引号等）。看文件名 + 行号定位。

**未定义引用：**

```
[09:35:30][localization.cpp:78]: Missing localization key: my_focus.1.t
[09:35:31][effect.cpp:200]: Invalid idea: my_missing_idea
```
含义：引用了不存在的 key（本地化文本或游戏对象）。检查拼写或确认相关对象文件已加载。

**重复定义冲突：**

```
[09:35:25][database.cpp:300]: Duplicate key: TAG_D01, already defined in history/countries/GER - Germany.txt
```
含义：两个文件定义了相同 ID（如 Tag、国策 ID、事件 ID）。修改其中一个的 ID。

**作用域错误：**

```
[09:36:00][effect.cpp:150]: Invalid scope for effect 'add_stability' in common/national_focus/my_focus.txt
```
含义：在错误的作用域上下文中使用了效果（如在省份作用域调用国家级效果）。

**注意事项（非致命）：**
```
[09:35:40][texturemanager.cpp:50]: Missing texture: GFX_my_custom_icon
```
这类日志不一定会崩溃，但会导致图标/精灵显示为问号。按优先级处理。

### game.log 解读

`game.log` 是完整运行记录，体积较大但信息全面。关键搜索模式：

| 搜索关键词 | 用途 |
|-----------|------|
| `[ERROR]` | 所有错误级别事件 |
| `[WARNING]` | 警告（不致命但需注意） |
| `Loading mod` | 确认你的 mod 被加载 |
| `Loaded mod` | Mod 加载完成的顺序 |
| `event` | 事件触发记录 |
| `decision` | 决议执行记录 |
| `focus` | 国策完成记录 |
| `crash` | 崩溃相关记录 |

**检查 Mod 是否正确加载：**
在 game.log 中搜索你的 mod 名称：
```
[08:15:00][mods.cpp:45]: Loading mod: MyModName (path: mod/mymod)
[08:15:01][mods.cpp:67]: Loaded mod: MyModName - 123 files loaded
```
如果找不到这些行，说明 mod 未被识别——检查 `.mod` 文件路径和 `supported_version`。

### CTD（Crash To Desktop）诊断流程

CTD 是最严重的问题。以下是系统化的诊断流程：

```
遭遇 CTD
  │
  ├─ ① 查看 exceptions.log 有无堆栈信息？
  │     ├─ 有 → 定位到具体函数/文件 → 修好 → 验证
  │     └─ 无 → 进入步骤 ②
  │
  ├─ ② CTD 发生在什么时候？
  │     ├─ 启动/加载画面 → 检查 error.log 语法错误
  │     │                   → 检查 .mod 文件编码（UTF-8 无 BOM）
  │     │                   → 检查本地化 .yml 编码（UTF-8 with BOM）
  │     └─ 游戏中特定操作 → 进入步骤 ③
  │
  ├─ ③ 移除最近增加/修改的内容，CTD 是否消失？
  │     ├─ 是 → 确定问题范围 → 进入步骤 ④
  │     └─ 否 → 二分法隔离（见下文）
  │
  ├─ ④ 检查相关文件：
  │     ├─ 括号闭合（每个 { 都有对应的 }）
  │     ├─ 引号配对（每个 " 都有对应的 "）
  │     ├─ 编码格式（.txt = UTF-8 无 BOM, .yml = UTF-8 with BOM）
  │     └─ 无用字符（不可见控制字符、中文标点混入代码）
  │
  └─ ⑤ 仍未解决 → 查看完整 game.log
                    → grep 原版类似结构对比
                    → 社区/论坛求助时附带 error.log + mod 压缩包
```

### 错误码速查表

| 错误码（Hex） | Windows 含义 | 常见场景 |
|--------------|-------------|---------|
| `0xC0000005` | ACCESS_VIOLATION | 内存访问违规，常见于空指针、野指针；删除正在引用的对象 |
| `0x3221225477` | STACK_BUFFER_OVERRUN | **最高频**：.yml 文件未使用 UTF-8 with BOM 编码 |
| `0xC0000374` | HEAP_CORRUPTION | 堆损坏；大型 mod 加载顺序冲突，或 replace_path 与其他 mod 冲突 |
| `0xC0000094` | INTEGER_DIVIDE_BY_ZERO | 除零错误，脚本中某计算公式除数为 0 |
| `0xC00000FD` | STACK_OVERFLOW | 无限递归 / 过深的嵌套调用（scripted effect 循环调用） |
| `0xE06D7363` | C++ Exception | 未处理的 C++ 异常；看 exceptions.log 详情 |
| 无错误码 | Silent CTD | 最常见原因：编码错误（.txt 文件有 BOM 或 .yml 无 BOM）、语法错误导致解析器崩溃 |

### 二分法隔离（Binary Search Isolation）

二分法是快速定位问题文件最可靠的方法，适用于 CTD 和逻辑错误：

```
流程：
① 备份当前 mod 文件夹
② 移除一半的文件/目录
③ 启动游戏测试 → CTD 消失？
     ├─ 是 → 问题在移除的那一半（恢复它们，再从中移除一半）
     └─ 否 → 问题在保留的这一半（从中再移除一半）
④ 重复步骤 ②-③，直到定位到具体文件
⑤ 在具体文件中，注释一半内容重复二分法，直到定位到具体行
```

**实用技巧**：
- 优先怀疑最近修改的文件
- 一次只改一个文件，保留操作记录
- 使用 Git 可快速回退到已知正常的版本：`git stash` 临时保存 → 测试 → `git stash pop` 恢复

### 逐行注释法（Line-by-Line Binary Search）

当二分法定位到具体文件后，用逐行注释法找到具体的错误行：

```
① 将文件后半部分全部注释掉（CTD 消失 → 问题在后半部分）
② 取消注释后半部分的一半（CTD 出现 → 问题在刚取消注释的这一半）
③ 继续二分注释，直到定位到具体行
④ 检查该行的括号闭合、引号、语法
```

### 控制台命令速查

在 `-debug` 模式下按 `~` 键打开控制台（键盘布局不同：美式键盘为 `~`，部分欧洲键盘为 `§` 或 `^`）。

**基础调试命令：**

| 命令 | 功能 |
|------|------|
| `tdebug` | 显示/切换调试信息（鼠标悬停显示省份 ID、地区 ID、战略区域等） |
| `debugtooltip` | 显示扩展提示（鼠标悬停显示更多内部数据） |
| `reloadfocus` | **重新加载国策树**——修改国策后无需重启游戏 |
| `reload events` | **重新加载事件文件**——修改事件后无需重启 |
| `reloadtechnologies` | 重新加载科技树 |
| `reload decisions` | 重新加载决议文件 |
| `reloadsupply` | 重新加载补给系统 |
| `reloadweather` | 重新加载天气数据 |
| `reload gfx` | 重新加载图形/纹理资源 |
| `reload text` | 重新加载本地化文本（部分文本可能需重启） |
| `reload_oob` | 重新加载部队序列 |
| `reload statics` | 重新加载静态修改器 |

**国家操作命令：**

| 命令 | 功能 | 用法 |
|------|------|------|
| `tag TAG` | 切换到指定国家 | `tag GER`（切换到德国） |
| `tag` (无参数) | 显示当前国家的 Tag | |
| `annex TAG` | 吞并指定国家 | `annex FRA` |
| `annex all` | 吞并所有其他国家（速通） | |
| `observe` | 观察者模式（脱离所有国家，纯观察） | |
| `human_ai` | 切换 AI 控制（当前国家由 AI 接管） | |
| `ai TAG` | 开关指定国家的 AI | `ai GER`（关闭德国 AI 控制） |
| `ai_invasion` | 开关 AI 入侵行为 | |

**资源和数值命令：**

| 命令 | 功能 | 用法 |
|------|------|------|
| `pp` | 增加政治点数 | `pp 1000` |
| `xp` | 增加所有类型经验 | `xp 1000`（陆军+海军+空军各 1000） |
| `army_xp` | 增加陆军经验 | `army_xp 500` |
| `navy_xp` | 增加海军经验 | `navy_xp 500` |
| `air_xp` | 增加空军经验 | `air_xp 500` |
| `st` | 增加稳定度 | `st 100`（最大 100%） |
| `ws` | 增加战争支持度 | `ws 100` |
| `manpower` | 增加人力 | `manpower 10000000` |
| `fuel` | 增加燃料 | `fuel 10000` |
| `cp` | 增加指挥点数 | `cp 100` |
| `add_latest_equipment` | 添加最新装备 | `add_latest_equipment 10000` |
| `add_equipment` | 添加指定装备 | `add_equipment 1000 infantry_equipment_2` |

**游戏状态命令：**

| 命令 | 功能 |
|------|------|
| `instantconstruction` / `ic` | 立即建造（切换开关） |
| `instantresearch` / `ir` | 立即研究（切换开关） |
| `instanttraining` / `it` | 立即训练（切换开关） |
| `instant_wargoal` | 立即制造战争借口（切换开关） |
| `allowdiplo` | 允许所有外交行动 |
| `nocb` | 允许无宣战理由开战 |
| `yesmen` | AI 接受所有外交提议 |
| `nomen` | AI 拒绝所有外交提议 |
| `fow` | 开关战争迷雾（开/关/开） |
| `debug_nuking` | 允许任意使用核弹 |
| `debug_events` | 在 game.log 中记录所有事件触发详情 |
| `event` | 手动触发事件 | `event my_event.1` |
| `event TAG event_id` | 对特定国家触发事件 | `event GER my_event.1` |
| `decision` | 执行指定决议 | `decision my_decision` |
| `focus` | 自动完成当前国策 | |
| `focus.autocomplete` | 国策瞬间完成（切换开关） |
| `nu` | 显示国家理念/精神 | |
| `research_on_icon_click` | 点击科技图标直接研究（跳过前置） |
| `research all` | 研究所有科技 |

**脚本调试命令：**

| 命令 | 功能 |
|------|------|
| `scripted_effects_dump` | 打印所有 scripted effects 列表到 game.log |
| `scripted_triggers_dump` | 打印所有 scripted triggers 列表到 game.log |
| `reload text` | 重新加载所有本地化文本 |
| `info` | 显示当前作用域/国家的详细数据 |
| `profile_data` | 显示性能分析数据（用于排查性能瓶颈） |
| `reloadtechnologies` | 重新加载科技树文件 |
| `dump_equipment_stats` | 打印装备统计数据到 game.log |
| `ai_log` | 开启 AI 行为详细日志记录 |
| `ai_step` | 让 AI 单步执行（用于观察 AI 决策过程） |

### 常见问题的快速诊断树

```
问题：Mod 未在启动器中显示
  ├─ .mod 文件在正确位置？（Documents/.../mod/）
  ├─ .mod 文件 path 字段正确？（格式："mod/yourfolder"）
  └─ supported_version 匹配当前游戏版本？

问题：Mod 勾选但游戏内无效果
  ├─ game.log 中能看到 "Loading mod: xxx" 吗？
  ├─ 文件放在正确的目录结构下？
  └─ 是否有 replace_path 意外屏蔽了你的文件？

问题：文本显示为 key 名称（如 my_event.1.t）
  ├─ .yml 文件编码是 UTF-8 with BOM？
  ├─ l_english: 声明行正确？
  └─ key 名称拼写完全一致？

问题：图标显示为问号
  ├─ 图片文件路径和名称与代码中 GFX_ 引用一致？
  ├─ spriteType 定义正确？（interface/*.gfx 文件）
  └─ 文件格式是 .dds（旗帜）/ .tga（UI）？

问题：事件/国策不触发
  ├─ trigger/available 条件检查通过了吗？（用 console 手动触发验证）
  ├─ 事件是 is_triggered_only = yes 吗？（只能用 effect 触发）
  └─ error.log 有相关语法错误吗？

问题：CTD 随机发生
  ├─ 移除所有 mod 后还崩溃吗？（判断是 vanilla 还是 mod 问题）
  ├─ 日志显示内存超标了吗？（HOI4 是 32 位程序，约 3.5GB 上限）
  └─ 大型 mod 组合冲突？（逐个启用 mod 测试）
```

## 实战示例

### 示例 1：定位启动 CTD（UTF-8 BOM 缺失）

**症状**：勾选 mod 后，HOI4 启动画面一闪而过，Windows 弹出错误码 `0x3221225477`（或 `3221225477` 十进制）。

**诊断步骤**：
1. 检查 exceptions.log：无堆栈信息（编码错误导致解析器在早期崩溃）
2. 使用二分法隔离：逐一移除文件夹测试
3. 定位到 `localisation/english/my_mod_l_english.yml`
4. 用 VS Code 打开文件，右下角显示编码为 "UTF-8"（无 BOM）
5. 点击右下角编码指示器 → "Save with Encoding" → 选择 "UTF-8 with BOM"
6. 重新启动游戏 → CTD 消失

**根本原因**：`.yml` 文件必须以 UTF-8 with BOM 编码保存。无 BOM 导致游戏解析器读取文件时无法正确识别编码边界，触发堆栈溢出（stack buffer overrun）。

### 示例 2：定位国策不显示的逻辑错误

**症状**：国策树的某个国策在游戏内不显示（图标区域为空，不可点击）。

**诊断步骤**：
1. 确认国策代码无语法错误（error.log 无相关报错）
2. 检查 `available` 条件：
```
available = {
    has_war_with = SOV    # ← 假设这里有问题
}
```
3. 在控制台用 `tag` 命令切换到该国家，用 `pp` 确保政治点数够用
4. 手动触发：检查 `has_war_with = SOV` 是否满足（当前存档国家确实在与苏联交战）
5. 发现问题：国策树中 `SOV` 不存在（该存档中苏联已不存在）→ 条件永远不满足
6. 修改条件：`OR = { has_war_with = SOV has_war_with = RUS }`（兼容苏联改名后的 Tag）

### 示例 3：使用二分法定位随机 CTD

**症状**：新增 500 行事件代码后，运行约 10 分钟随机 CTD，无明确错误信息。

**诊断步骤**：
1. 备份 mod 文件夹
2. 移除 `events/` 目录 → CTD 消失 → 问题确定在 events 目录
3. 恢复 events 文件，移除后 50% 事件文件 → CTD 消失 → 问题在这 50% 中
4. 从中再移除一半 → CTD 消失 → 重复二分
5. 最终定位到 `my_events_scenario_3.txt` 文件
6. 在该文件中二分注释 → 定位到第 187 行：`country_event = { id = my_event.99 ...`
7. 发现 `my_event.99` 效果中调用了 `puppet = SOV`，但该时刻 SOV 已不存在
8. 添加 `exists = SOV` 前置检查 → CTD 解决

## 常见陷阱

1. **不先看 error.log 就盲目改代码** — 永远先打开 error.log。90% 的问题在 error.log 中有明确提示（文件名 + 行号）。在大量代码中凭直觉猜问题来源是最低效的方法。

2. **忘记 `-debug` 参数** — 不用 `-debug` 启动就没法用控制台和 tdebug。在 Steam 启动参数没加、或者用桌面快捷方式直接启动时容易忽略。

3. **混淆 .txt 和 .yml 的编码要求** — `.txt`（国策/事件/决议等）→ UTF-8 无 BOM。`.yml`（本地化）→ UTF-8 with BOM。两者编码互换是最常见的 CTD 原因。

4. **`reloadfocus` 的局限性** — `reloadfocus` 只能重载国策树结构（focus 的位置、连接、可用条件），但不能重载 completion_reward 的修改（已完成的国策效果不会重新执行）。测试效果变更时必须重启或新开存档。

5. **不备份就大规模重构** — 在进行大规模修改前用 Git 提交或手动备份整个 mod 文件夹。一旦陷入复杂的 CTD，回退到已知正常的状态是最快的恢复方式。

## FAQ

**Q: error.log 完全没有报错但仍然 CTD 怎么办？**
A: 按以下顺序排查：(1) 检查 .yml 文件是否 UTF-8 with BOM；(2) 检查 .txt 文件是否 UTF-8 无 BOM；(3) 用二分法隔离所有 mod 文件；(4) 检查是否有无限循环/递归（如 scripted effect A 调用 B，B 又调用 A）；(5) 检查内存——大型 mod 总文件超过 2GB 可能导致 32 位程序溢出。

**Q: `reload events` 后改了的事件仍然不生效？**
A: 已触发过的事件不会被重定义覆盖。`reload events` 对新触发的事件有效，但不会回滚已生效的事件效果。另外 `country_event` 的 `id` 如果是 `is_triggered_only = yes`，必须通过 effect 触发——控制台 `event` 命令仍需 mod 代码中预先存在。

**Q: 如何确认 mod 加载了哪些文件、加载顺序？**
A: 查看 `Documents/Paradox Interactive/Hearts of Iron IV/logs/setup.log`。该文件记录了完整的 mod 加载序列和路径解析过程。如果 mod 文件未被列出，说明路径或 .mod 文件有问题。

**Q: 多人游戏测试时如何调试？**
A: 所有玩家都必须在启动选项中添加 `-debug`。主机可以在控制台中执行命令，但在多人游戏中部分命令（如 `annex`）可能导致同步错误（OOS）。建议先在单人模式中调试逻辑，多人模式只测试同步。

**Q: VS Code + CWTools 能替代手动查 error.log 吗？**
A: CWTools 可以捕获大部分语法错误（括号不匹配、未知字段），但无法检测逻辑错误（如作用域错误、运行时条件失败）、编码错误，以及 mod 加载顺序问题。CWTools + error.log 结合使用是最佳实践。

## 检查清单

- [ ] `-debug -crash_data_log` 启动参数已设置
- [ ] 每次启动测试后检查 `error.log`
- [ ] 知道日志目录位置（Documents/.../logs/）
- [ ] 掌握 `tdebug`、`tag`、`reloadfocus`、`reload events` 等核心命令
- [ ] 理解 .txt（UTF-8 无 BOM）和 .yml（UTF-8 with BOM）的编码要求
- [ ] 能用二分法在 5 分钟内定位问题文件
- [ ] 修改 mod 前先 Git 提交或备份
- [ ] 能在 game.log 中确认 mod 是否正确加载

> **相关文件**: Mod 项目结构（02-mod-structure.md）——许多 CTD 源于文件路径错误或 .mod 配置问题。PDXscript 语法基础（03-pdxscript.md）——作用域错误、括号不匹配是最常见语法问题。世界树编辑器（Nudge）使用说明见地图系统（11-map-states.md）。Steam Workshop 发布流程（15-workshop.md）介绍上传前的最终检查清单。
