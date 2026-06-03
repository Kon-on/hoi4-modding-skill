> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

HOI4 使用 Paradox 自有的脚本语言（社区常称 PDXscript），基于 `key = value` 和 `key = { ... }` 的嵌套块结构。它是声明式的——你描述「有什么」和「在什么条件下触发」，而不是写过程式逻辑。掌握 PDXscript 是做任何 HOI4 mod 的前提。

## 完整语法参考

### 基本元素

```
# 这是注释——# 之后到行尾的内容被忽略
key = value             # 简单赋值（数字、布尔、字符串）
key = { ... }           # 块/作用域（嵌套结构）
key = { a b c }         # 列表（空格分隔的值）
key = yes               # 布尔值（yes/no）
key = 100               # 整数
key = 0.05              # 小数
key = "string"          # 字符串（可含空格）
key = TAG                # 国家 Tag 引用（不加引号）
```

### 作用域系统（THIS / FROM / ROOT / PREV）

作用域决定了代码的「视角」——当前在操作哪个国家、哪个省份、哪个事件。这是 PDXscript 最核心也最容易混淆的概念。

| 作用域 | 含义 | 典型场景 |
|--------|------|---------|
| `THIS` | 当前所在的作用域 | 事件选项中 THIS 是接收事件的国家 |
| `FROM` | 上一层作用域 | 国策触发事件：FROM 是拥有国策的国家 |
| `ROOT` | 最顶层作用域 | 国策树中 ROOT 始终是拥有该树的国家 |
| `PREV` | 再上一层作用域 | 多层嵌套时逐层向外返回 |

**作用域传递链（国策树触发事件的典型案例）：**

```
ROOT = 拥有国策树的国家（最顶层，始终不变）
  └─ 国策 completion_reward 中:
       country_event = { id = X }  →  事件里:
         THIS = 接收事件的国家（通常 = ROOT）
         FROM = ROOT（触发者）
```

### 触发器（Triggers）

放在 `trigger`、`available`、`allowed`、`limit` 块中。条件满足返回 true。

**逻辑组合：**
```
AND = { 条件1 条件2 条件3 }     # 全部满足 → true
OR = { 条件1 条件2 }             # 任一满足 → true
NOT = { 条件 }                   # 条件不满足 → true
```

**常用条件速查：**

| 条件 | 说明 | 示例 |
|------|------|------|
| `has_political_power > 50` | 政治点数 > 50 | 数值比较：`>` `<` `>=` `<=` `=` |
| `has_war = yes` | 处于战争状态 | |
| `has_war_with = TAG` | 与某国交战 | `has_war_with = GER` |
| `date > 1939.1.1` | 日期晚于 1939.1.1 | 格式：年.月.日 |
| `controls_state = 123` | 控制省份 ID 123 | 用 tdebug 查看省份 ID |
| `owns_state = 123` | 拥有省份 123 | |
| `has_technology = infantry_1` | 已研究某科技 | |
| `has_idea = my_spirit` | 拥有某国家精神 | |
| `exists = TAG` | 某国家存在 | |
| `tag = GER` | 当前国家是德国 | |
| `has_government = democratic` | 政体为民主 | 也可用 `communism` `fascism` `neutrality` |
| `is_in_faction = yes` | 在阵营中 | |
| `is_in_faction_with = TAG` | 与某国同一阵营 | |
| `stability > 0.5` | 稳定度 > 50% | 0~1 的小数 |
| `war_support > 0.5` | 战争支持度 > 50% | 0~1 的小数 |
| `surrender_progress > 0.5` | 投降进度 > 50% | |
| `has_army_experience > 10` | 陆军经验 > 10 | |
| `num_of_factories > 20` | 工厂数量 > 20 | |
| `has_country_flag = my_flag` | 有自定义标记 | 通过 set_country_flag 设置 |
| `check_variable = { var = x value = 5 compare = greater_than }` | 变量比较 | |

### 效果（Effects）

放在 `effect`、`complete_effect`、`completion_reward`、`remove_effect` 块中。改变游戏状态。

**常用效果速查：**

| 效果 | 说明 | 示例 |
|------|------|------|
| `add_political_power = 100` | 增加政治点数 | |
| `add_stability = 0.05` | 增加稳定度 | 0~1 范围 |
| `add_war_support = 0.05` | 增加战争支持度 | 0~1 范围 |
| `set_rule = { communist = 0.5 }` | 设置意识形态支持度 | |
| `add_ideas = my_idea` | 添加国家精神 | |
| `remove_ideas = old_idea` | 移除国家精神 | |
| `swap_ideas = { remove = old add = new }` | 替换国家精神 | |
| `declare_war_on = { target = TAG }` | 宣战 | 需配合 wargoal |
| `annex_country = { target = TAG }` | 吞并国家 | |
| `puppet = TAG` | 傀儡某国 | |
| `set_country_flag = my_flag` | 设置自定义标记 | 用于触发器检查 |
| `clr_country_flag = my_flag` | 清除标记 | |
| `country_event = { id = X days = 7 }` | 延迟触发事件 | |
| `country_event = { id = X hours = 24 }` | 延迟触发（小时） | |
| `country_event = { id = X }` | 立即触发事件 | |
| `add_manpower = 10000` | 增加人力 | |
| `add_offsite_building = { type = arms_factory level = 1 }` | 添加建筑 | |
| `add_resource = { type = oil amount = 10 }` | 添加资源 | |
| `load_focus_tree = TAG_focus` | 加载国策树 | |
| `hidden_effect = { ... }` | 执行但不显示提示 | 用于后台逻辑 |

### 条件分支（if / else_if / else）

```
if = {
    limit = { has_war = yes }      # 条件
    add_war_support = -0.05        # 满足时执行
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
set_variable = { var = my_counter value = 0 }         # 创建/赋值
change_variable = { var = my_counter value = 1 }       # 增减（value 可为负）
check_variable = { var = my_counter value = 5 compare = greater_than }  # 比较
subtract_variable = { var = my_counter value = 3 }     # 减法
multiply_variable = { var = my_counter value = 2 }     # 乘法
```

## 实战示例

### 示例 1：一个带条件的国策完成奖励

```
completion_reward = {
    if = {
        limit = { has_war = yes }
        add_war_support = 0.05
        add_army_experience = 20
    }
    else = {
        add_stability = 0.05
    }
    # 无论条件如何都执行
    add_political_power = 50
}
```

说明：如果在战争中，获得 +5% 战争支持度和 20 陆军经验；如果和平，获得 +5% 稳定度。无论如何都获得 50 政治点数。

### 示例 2：使用 FROM 在事件中操作触发者

事件文件：
```
country_event = {
    id = my_event.1
    title = my_event.1.t
    desc = my_event.1.d
    picture = GFX_report_event_generic

    is_triggered_only = yes

    option = {
        name = my_event.1.a
        # THIS = 接收事件的国家, FROM = 触发此事件的国家（ROOT）
        FROM = {
            add_political_power = 100  # 给触发者加 PP
        }
        add_stability = 0.05  # 给自己加稳定度
    }
}
```

### 示例 3：使用变量实现计数器

```
# 初始化
set_variable = { var = invasion_count value = 0 }

# 每次入侵 +1
change_variable = { var = invasion_count value = 1 }

# 检查
if = {
    limit = { check_variable = { var = invasion_count value = 3 compare = greater_than } }
    country_event = { id = too_many_invasions.1 }  # 入侵太多触发事件
}
```

## 常见陷阱

1. **括号不匹配** — 缺少闭合 `}` 是最常见且最难排查的语法错误。使用 VS Code + CWTools 的括号匹配功能可大幅减少此类问题。
2. **作用域混淆** — 不确定当前代码在操作谁时，使用 `THIS` `FROM` `ROOT` 显式指定作用域。常见错误：在事件 option 中直接写 `add_stability`，但想给 FROM 加——应该写成 `FROM = { add_stability = ... }`。
3. **trigger 和 effect 用错位置** — 把效果写在 available 块中不会执行；把条件写在 complete_effect 中也不会检查。
4. **编码错误** — .txt 文件必须 UTF-8 无 BOM（有 BOM 的 .txt 文件可能导致游戏崩溃或文件完全被忽略）。
5. **列表格式错误** — `replace_path = { "a" "b" }` 正确（空格分隔加引号），`replace_path = { a, b }` 错误（不支持逗号分隔）。

## FAQ

**Q: if/else 和 trigger 有什么区别？**
A: trigger（available/allowed）控制**操作是否可选**（按钮是否可点击），if/else 控制**操作内的执行分支**（点击后的不同结果）。

**Q: 如何调试作用域问题？**
A: 在 -debug 模式下，可以用 `log = "DEBUG: [THIS.GetName] [FROM.GetName]"` 在 game.log 中打印当前和上层作用域名称。

**Q: from 和 prev 什么时候不同？**
A: 在两层嵌套中相同。三层以上时：假设 A→B→C，在 C 中 FROM=B, PREV=B；但如果有 A→B→C→D，在 D 中 FROM=C, PREV=C。实际使用中大多数情况 FROM 就够了。

**Q: 比较字符串怎么写？**
A: `tag = GER` 而非 `tag = "GER"`。PDXscript 中 Tag 引用、ID、key 通常不加引号，显示文本才用引号。

## 检查清单

- [ ] 理解 key = value 和 block = { } 语法
- [ ] 理解 THIS / FROM / ROOT / PREV 四个作用域及其传递链
- [ ] 能区分 trigger 和 effect 的使用位置
- [ ] 掌握 if / else_if / else 条件分支
- [ ] 了解常用 trigger（has_war, has_political_power, date, tag 等至少 5 个）
- [ ] 了解常用 effect（add_stability, add_ideas, country_event, declare_war_on 等至少 5 个）

> **相关文件**: 国策树（04-focus-trees.md）、事件（05-events.md）、决议（06-decisions.md）分别详细介绍各系统的特有语法。
