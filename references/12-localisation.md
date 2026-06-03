> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

本地化（Localisation）是 HOI4 中将内部 key 映射为玩家可见文本的系统。所有界面文字、国策名称、事件标题、决议描述等都通过本地化文件定义，而非硬编码在游戏逻辑中。支持多语言，每种语言一个单独的 `.yml` 文件。这是 mod 发布的最后一步——先保证逻辑正确，再补文本。

## 完整语法参考

### 文件结构

本地化文件必须放在 `localisation/<语言文件夹>/` 下（如 `localisation/english/`），文件扩展名为 `.yml`。

```
localisation/
├── english/                    # 英文
│   ├── my_mod_l_english.yml    # 文件名可以任意，但建议用描述性名称
│   └── my_events_l_english.yml
├── chinese/                    # 中文（简体）
│   ├── my_mod_l_chinese.yml
│   └── my_events_l_chinese.yml
└── replace/                    # 用于替换原版文本（同名文件覆盖合并）
    └── english/
        └── diplomacy_l_english.yml
```

### 基本语法

每个 `.yml` 文件以语言声明行开头，后跟 `key:版本号 "文本"` 对：

```
l_english:
 key_name:0 "This is the displayed text."
 key_two:0 "Second line of text."
```

- 第一行必须是 `l_english:`（或 `l_chinese:`、`l_french:` 等），告诉游戏这是哪种语言的本地化
- 每个条目格式：`<key>:<version> "<text>"`
- `<key>`：在代码中引用的 ID（国策、事件、决议等）
- `<version>`：版本号，通常写 `0`，游戏在版本变更时用它来标记需要重新翻译的条目
- `<text>`：双引号包裹的显示文本，可包含格式化代码和动态命令
- 缩进用一个 Tab 或两个空格（必须一致）

### 颜色和格式化代码

使用 `§` 字符开头后跟一个字母来改变文本颜色和样式。这些代码在显示时被解析，不会出现在最终文本中。

| 代码 | 效果 | 典型用途 |
|------|------|---------|
| `§R` | 红色 | 负面效果、警告 |
| `§G` | 绿色 | 正面效果、增益 |
| `§Y` | 黄色 | 国策名称、重要信息 |
| `§!` | 恢复默认颜色 | 在彩色文本后恢复正常 |
| `§B` | 蓝色 | 技术/科技相关 |
| `§W` | 白色 | 强调（在深色背景上） |
| `§H` | 黄绿色 | 特殊变量/数值 |
| `§S` | 暗橙色 | 次级警告 |
| `§r` | 浅红色 | 轻度负面 |
| `§g` | 金色 | 稀有/贵重物品 |
| `§b` | 蓝灰色 | 次要信息 |
| `§l` | 浅蓝色 | 链接/可交互提示 |
| `§t` | 淡色/灰色 | 不可用/灰色选项 |

**换行、Tab 和特殊字符：**
| 代码 | 效果 |
|------|------|
| `\n` | 换行（在同一个文本块中另起一行） |
| `\t` | Tab 缩进 |
| `\\` | 显示一个 `\` 字符 |
| `\"` | 显示一个双引号 |

示例：
```
my_focus_desc:0 "§YThis focus unlocks:§!\n§G+10%§! Factory Output\n§R−5%§! Stability"
```
显示效果：黄色标题 + 换行 + 绿色增益 + 红色惩罚。

### 动态命令（Bracket Commands）

运行时根据游戏状态动态替换文本，是本地化系统最强大的特性。语法为 `[作用域.命令]`。

**国家名称和属性：**

| 命令 | 说明 | 示例输出 |
|------|------|---------|
| `[TAG.GetName]` | 国家名称 | `Germany` |
| `[TAG.GetNameDef]` | 带定冠词的名词（英文专有） | `the German Reich` |
| `[TAG.GetNameDefCap]` | 首字母大写的定冠词名词 | `The German Reich` |
| `[TAG.GetAdjective]` | 形容词形式 | `German` |
| `[TAG.GetRulingParty]` | 执政党名称 | `NSDAP` |
| `[TAG.GetLeader]` | 国家领导人名字 | `Adolf Hitler` |
| `[TAG.GetFlag]` | 国旗图标字符串 | (显示国旗图标) |
| `[TAG.GetID]` | 国家 Tag 标识符 | `GER` |
| `[TAG.GetMotto]` | 国家格言 | `Ein Volk, ein Reich, ein Führer` |
| `[FROM.GetName]` | 上一级作用域的国家名 | |
| `[ROOT.GetName]` | 根作用域的国家名 | |

**数值和日期：**

| 命令 | 说明 |
|------|------|
| `[?scope.variable_name]` | 显示变量值（`?` 前缀表示值可能为 0） |
| `[?scope.variable_name|+.2]` | 带格式的变量值（正号 + 2 位小数） |
| `[?scope.variable_name|%]` | 百分比格式 |
| `[?scope.variable_name|H]` | 十六进制格式 |
| `[?scope.variable_name|*100]` | 乘以系数 100 |
| `[?scope.variable_name|-100]` | 减去 100 |
| `[?date]` | 当前游戏日期 |
| `[FROM.GetDateString]` | 格式化日期字符串 |

**其他常用命令：**

| 命令 | 说明 |
|------|------|
| `[SCOPE.GetToken]` | 获取自定义 Token 文本 |
| `[SCOPE.GetTokenIf]` | 条件性获取 Token 文本 |
| `[SCOPE.GetName]` | 通用名称获取（适用于国家、省份、角色等） |
| `[SCOPE.GetHerselfHimself]` | 根据领导人性别返回 herself/himself |
| `[SCOPE.GetSheHe]` | 根据领导人性别返回 She/He |
| `[SCOPE.GetHerHis]` | 根据领导人性别返回 her/his |

### 作用域在本地化中的使用

本地化中的作用域由触发条件代码的上下文决定：

- **国策树**：`ROOT` = 拥有该树的玩家国家
- **事件（country_event）**：`THIS` = 接收事件的国家，`FROM` = 触发事件的国家（通常 = ROOT）
- **决议**：`THIS` = 执行决议的国家
- **装备/科技描述**：作用域通常为全局，需要用变量传递

### Scripted Localisation（脚本化本地化）

当文本需要根据条件动态变化时，使用 scripted_localisation。定义在 `common/scripted_localisation/` 下。

```
# common/scripted_localisation/my_dynamic_text.txt
defined_text = {
    name = my_war_status_text
    text = {
        trigger = { has_war = yes }
        localization_key = "war_active_text"
    }
    text = {
        trigger = { has_war = no }
        localization_key = "peace_text"
    }
}
```

然后在 .yml 中用 `[SCOPE.my_war_status_text]` 引用。游戏会根据条件自动选择对应的本地化条目。

### 文本替换（replace 目录）

要替换原版某条文本，在 `localisation/replace/<语言>/` 下创建同名 .yml 文件，写入新值：

```
# localisation/replace/english/my_replacements_l_english.yml
l_english:
 german_fascism_party:0 "My Custom Party Name"
```

**注意**：replace 目录会完整覆盖同名原版文件中的所有条目，而不仅仅是修改的那几条。最好只放你要改的条目，使用与原版相同的文件名。

### 支持的本地化键后缀约定

| 后缀 | 含义 | 示例使用处 |
|------|------|-----------|
| 无后缀 | 显示名称 | 国策名称 `my_focus:0 "My Focus"` |
| `_desc` | 描述/说明文字 | 国策描述 `my_focus_desc:0 "This focus..."` |
| `.t` | 标题（title） | 事件标题 `my_event.1.t:0 "Event Title"` |
| `.d` | 描述（description） | 事件描述 `my_event.1.d:0 "Event Description"` |
| `.a` | 选项 A | 事件选项 `my_event.1.a:0 "Option A"` |
| `.b` | 选项 B | 事件选项 `my_event.1.b:0 "Option B"` |
| `_category` | 分类名称 | 决议分类 |
| `_modifier` | 修正名称 | 修正器显示文本 |

## 实战示例

### 示例 1：完整的国策 + 事件本地化

国策代码：
```
# common/national_focus/my_focus.txt
focus = {
    id = MY_industrial_boom
    icon = GFX_goal_generic_construct_civilian
    x = 0
    y = 0
    cost = 10
    completion_reward = {
        country_event = { id = my_event.1 }
    }
}
```

事件代码：
```
# events/my_events.txt
country_event = {
    id = my_event.1
    title = my_event.1.t
    desc = my_event.1.d
    picture = GFX_report_event_generic

    is_triggered_only = yes

    option = {
        name = my_event.1.a
        add_political_power = 50
    }
    option = {
        name = my_event.1.b
        add_stability = 0.05
    }
}
```

本地化文件：
```
# localisation/english/my_mod_l_english.yml
l_english:
 MY_industrial_boom:0 "Industrial Boom"
 MY_industrial_boom_desc:0 "Our factories work at full capacity."
 my_event.1.t:0 "Factory Expansion Decision"
 my_event.1.d:0 "The industrial boom program has exceeded expectations. How shall we proceed?"
 my_event.1.a:0 "§GInvest 50 Political Power§!"
 my_event.1.b:0 "§YConsolidate for Stability§!"
```

### 示例 2：动态文本（颜色 + 国家名称 + 变量）

```
# localisation/english/my_mod_l_english.yml
l_english:
 my_dynamic_title:0 "§Y[ROOT.GetName]§! Diplomatic Report"
 my_dynamic_desc:0 "Our relations with §B[FROM.GetName]§! have reached a critical point.\n\nCurrent trade volume: §G[?ROOT.trade_volume|+.1]§!\n\n§RFailure to act may result in war.§!"
 my_variable_report:0 "Factory Output: §Y[?ROOT.factory_output]§! units per day. Target: §B[?ROOT.factory_target]§!"
```
说明：`[ROOT.GetName]` 在游戏运行时会替换为玩家国家的实际名称；`[FROM.GetName]` 替换为目标国家名称；`[?ROOT.trade_volume|+.1]` 显示 `trade_volume` 变量的值，正数显示加号、1 位小数。

### 示例 3：Scripted Localisation 实现条件化文本

```
# common/scripted_localisation/dynamic_status.txt
defined_text = {
    name = get_economic_status
    text = {
        trigger = { check_variable = { var = gdp_index value = 80 compare = greater_than } }
        localization_key = "econ_boom"
    }
    text = {
        trigger = { check_variable = { var = gdp_index value = 50 compare = greater_than } }
        localization_key = "econ_normal"
    }
    text = {
        trigger = { check_variable = { var = gdp_index value = 20 compare = greater_than } }
        localization_key = "econ_recession"
    }
    text = {
        localization_key = "econ_depression"  # 默认兜底
    }
}
```

```
# localisation/english/my_mod_l_english.yml
l_english:
 econ_boom:0 "§GBooming§!"
 econ_normal:0 "Stable"
 econ_recession:0 "§SRecession§!"
 econ_depression:0 "§RDepression§!"
 economy_report:0 "Economic Status: [ROOT.get_economic_status]"
```

注意：trigger 按顺序从上到下评估，第一个满足条件的生效。如果都不满足，使用不带 trigger 的默认条目（必须放在最后）。

## 常见陷阱

1. **UTF-8 BOM 缺失（最高频崩溃原因）** — `.yml` 本地化文件必须以 UTF-8 with BOM 编码保存。如果保存为 UTF-8 without BOM，游戏加载时崩溃并弹出错误码 3221225477（代表 STATUS_STACK_BUFFER_OVERRUN）。VS Code 右下角编码指示器点击 → "Save with Encoding" → 选择 "UTF-8 with BOM"。

2. **缩进不一致** — `.yml` 文件对缩进要求严格。混用 Tab 和空格、缩进层级错误都会导致解析失败。使用 VS Code 右下角选择缩进模式，确保整个文件一致（推荐 2 个空格）。

3. **`l_english:` 声明行缺失或拼写错误** — 文件第一行必须是 `l_english:`（注意是下划线不是连字符）。如果写成了 `l-english:` 或遗漏冒号，整个文件的本地化都不会被加载。

4. **Key 不匹配** — 代码中引用的 key 必须与 .yml 中定义的 key 完全一致（大小写敏感）。常见的错误：代码中用 `MY_FOCUS` 但实际的 key 是 `my_focus`。HOI4 约定 key 使用小写字母 + 下划线。

5. **双引号未正确转义** — 文本内容中如果需要显示双引号，必须用 `\"`。`"He said "Hello""` 错误，应为 `"He said \"Hello\""`。

## FAQ

**Q: 为什么我的本地化文本显示为 key 名称（如 `my_focus_desc`）而不是实际文本？**
A: 三个可能原因：(1) .yml 文件的 `l_english:` 声明行错误或缺失；(2) 文件编码不是 UTF-8 with BOM；(3) key 名称拼写不一致（检查大小写和下划线）。

**Q: 可以一个 mod 中有多个 .yml 文件吗？**
A: 可以，而且推荐。按功能模块拆分为多个文件（如国策本地化、事件本地化、决议本地化各一个文件），方便维护。游戏会加载该语言目录下的所有 .yml 文件并合并。

**Q: 如何让 mod 支持中文？**
A: 在 `localisation/chinese/` 下创建同名 .yml 文件，使用 `l_chinese:` 声明行，key 与英文版本保持一致。游戏会根据玩家选择的语言自动加载对应文件夹下的 .yml。

**Q: `key:0` 中的数字 `0` 是什么？必须写 `0` 吗？**
A: 版本号标记，用于游戏大版本更新时标记哪些条目需要重新翻译。目前写 `0` 即可。如果游戏更新后某个 key 的文本需要修改，开发商会把数字改为 `1` 来标记。

**Q: 文本中可以嵌入图片/图标吗？**
A: 可以。使用 `[SCOPE.GetFlag]` 嵌入国旗图标，`[SCOPE.GetIdeologyIcon]` 嵌入意识形态图标，或者在 sprite 定义后用特殊语法嵌入自定义 GFX 图标。图标语法因上下文而异，建议 grep 原版事件文件学习。

**Q: 如何实现一个文本中显示不同的数字格式（百分比、千分位等）？**
A: 使用 `|` 管道格式化：`[?ROOT.my_var|%]` 百分比，`[?ROOT.my_var|+.2]` 带正号 2 位小数，`[?ROOT.my_var|H]` 十六进制，`[?ROOT.my_var|*100]` 乘以 100。

## 检查清单

- [ ] 文件编码为 UTF-8 with BOM（VS Code: Save with Encoding → UTF-8 with BOM）
- [ ] 第一行是 `l_english:`（或对应语言声明）
- [ ] 所有 key 与代码中的引用完全一致（大小写、下划线）
- [ ] 缩进统一（全部 Tab 或全部 2 空格）
- [ ] 双引号正确转义（`\"`）
- [ ] 动态命令语法正确（`[SCOPE.Command]`，注意是方括号不是花括号）
- [ ] 颜色代码使用 `§` 符号而非 `$` 或 `#`
- [ ] 文件放在正确的语言文件夹下（`localisation/english/` 等）
- [ ] 启动游戏验证所有文本正常显示

> **相关文件**: PDXscript 语法基础（03-pdxscript.md）介绍作用域概念，这是理解 `[FROM.GetName]` 等命令的前提。国策树（04-focus-trees.md）和事件（05-events.md）分别介绍如何正确引用本地化 key。脚本化效果/触发器（13-scripted.md）介绍 scripted_localisation 的完整用法。
