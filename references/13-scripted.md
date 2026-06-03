> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

HOI4 提供三种脚本化（Scripted）系统来减少重复代码：**scripted_effects**（可复用效果块）、**scripted_triggers**（可复用条件块）和 **scripted_localisation**（条件化动态文本）。它们是实现 DRY（Don't Repeat Yourself）原则的核心工具——把在多处使用相同逻辑抽取为一个命名块，一处定义、多处调用。大型 mod（如 Kaiserreich、TNO）大量使用脚本化系统来保持代码可维护性。

## 完整语法参考

### Scripted Effects（脚本化效果）

定义在 `common/scripted_effects/` 目录下的 `.txt` 文件中。用于封装可复用的效果块。

**基本语法：**

```
# common/scripted_effects/my_effects.txt
scripted_effect_my_boost = {
    add_political_power = 50
    add_stability = 0.05
    add_war_support = 0.03
}
```

**调用方式（在使用者代码中）：**

```
# 在任何 effect 块中调用
completion_reward = {
    scripted_effect_my_boost = yes  # ← 直接作为一条效果调用
}
```

**命名约定**：虽然语法上可以任意命名，但社区惯例是使用 `scripted_effect_` 前缀或按功能分组命名（如 `my_event_effect_01`、`stabalize_nation_effect`），以清晰区分于普通效果命令。

### 参数化（$PARAM$ 替换）

Scripted effects 支持参数替换，这是其最强大的特性。在定义中用一个以 `$` 包裹的占位符，调用时传入实际值。

**定义（带参数）：**

```
# common/scripted_effects/my_effects.txt
add_custom_stability = {
    if = {
        limit = { $CONDITION$ }
        add_stability = $AMOUNT$
    }
    else = {
        add_stability = $FALLBACK$
    }
}
```

**调用（传入参数）：**

```
# 在效果块中调用
completion_reward = {
    add_custom_stability = {
        CONDITION = has_war = yes     # 替换 $CONDITION$
        AMOUNT = 0.05                 # 替换 $AMOUNT$
        FALLBACK = 0.02               # 替换 $FALLBACK$
    }
}
```

**参数规则**：参数名用 `$NAME$` 包裹。调用时提供 `NAME = <值>`，值可以是数值、布尔、作用域引用、甚至整个条件/效果块。参数按名称匹配（非位置匹配）。未提供的参数将被替换为空（可能导致语法错误）。

**作用域传递**：Scripted effects/triggers 在调用时**不改变**当前作用域。调用发生时的 THIS 会被原封不动地传入内部代码。如需操作其他作用域，用参数传入：

```
# 定义
affect_target = {
    $TARGET$ = { add_political_power = 50 }
}

# 调用
completion_reward = {
    affect_target = { TARGET = FROM }  # 操作 FROM
    affect_target = { TARGET = GER }   # 操作德国
}
```

### Scripted Triggers（脚本化触发器）

定义在 `common/scripted_triggers/` 目录下，用于封装可复用的条件块。

```
# 定义
is_major_power = {
    OR = {
        num_of_factories > 50
        original_tag = GER   original_tag = SOV   original_tag = USA
        original_tag = ENG   original_tag = FRA   original_tag = ITA  original_tag = JAP
    }
}

is_at_peace = { has_war = no }
is_stable = { stability > 0.5 }

# 调用
available = { is_major_power = yes   has_political_power > 100 }
trigger = { is_major_power = yes }
if = { limit = { is_major_power = yes }   add_stability = 0.05 }
```

**带参数的 Scripted Triggers：**
```
has_more_factories_than = { num_of_factories > $TARGET_FACTORIES$ }
# 调用：available = { has_more_factories_than = { TARGET_FACTORIES = 100 } }
```

### Scripted Localisation（脚本化本地化）

定义在 `common/scripted_localisation/` 目录下。详见 12-localisation.md 的 Scripted Localisation 章节。这里补充技术细节。

**完整语法：**

```
# common/scripted_localisation/my_dynamic_text.txt
defined_text = {
    name = get_war_status                       # 引用时用的名称
    text = {
        trigger = { has_war = yes }              # 条件 1
        localization_key = "war_active_text"     # 满足条件 1 时显示的 key
    }
    text = {
        trigger = { has_war = no }
        localization_key = "peace_text"
    }
    text = {
        localization_key = "unknown_text"        # 没有 trigger = 默认兜底
    }
}
```

**在本地化文件中引用：**

```
my_event.1.d:0 "Current Status: [ROOT.get_war_status]"
```

**触发条件评估规则**：游戏从第一个 `text` 条目开始按序评估 `trigger`。第一个返回 true 的条目被使用。如果所有带 trigger 的都不满足，使用不带 trigger 的兜底条目。无兜底 → 文本显示为空。

### 文件组织最佳实践

```
common/scripted_effects/
├── 00_vanilla_overrides.txt   # 覆盖原版效果
├── MY_economy_effects.txt     # 按功能模块拆分
├── MY_political_effects.txt
└── MY_war_effects.txt
common/scripted_triggers/
├── 00_vanilla_overrides.txt
├── MY_country_checks.txt
└── MY_war_checks.txt
common/scripted_localisation/
└── MY_dynamic_text.txt
```
命名：`00_` 前缀最先加载（与 vanilla 一致）；用 mod 前缀避免冲突；按功能拆分保持每个文件 < 200 行。

## 实战示例

### 示例 1：可复用的稳定度修正效果（带条件分支）

```
# common/scripted_effects/MY_stability_effects.txt

# 统一稳定度调整：战争期间惩罚减半
adjust_stability_contextual = {
    if = {
        limit = { has_war = yes }
        add_stability = $WAR_VALUE$
    }
    else = {
        add_stability = $PEACE_VALUE$
    }
}

# 标准化紧急稳定度干预（扣政治点数换稳定度）
emergency_stability_boost = {
    if = {
        limit = {
            has_political_power > $PP_COST$
            stability < 0.5
        }
        add_political_power = $PP_COST_NEG$
        add_stability = $STAB_GAIN$
        set_country_flag = stability_emergency_used
    }
}
```

调用示例：
```
# 国策完成奖励
completion_reward = {
    adjust_stability_contextual = {
        WAR_VALUE = 0.05
        PEACE_VALUE = 0.10
    }
}

# 决议效果
effect = {
    emergency_stability_boost = {
        PP_COST = 100
        PP_COST_NEG = -100
        STAB_GAIN = 0.15
    }
}
```

### 示例 2：可复用的「列强检查」触发器（多条件组合）

```
# common/scripted_triggers/MY_country_checks.txt

# 是否为主要国家（工厂数量 + 原版主要国家）
is_major_or_industrial = {
    OR = {
        num_of_factories > 50
        original_tag = GER
        original_tag = SOV
        original_tag = USA
        original_tag = ENG
        original_tag = FRA
        original_tag = ITA
        original_tag = JAP
    }
}

# 是否处于危险状态
is_in_danger = {
    AND = {
        stability < 0.4
        OR = {
            has_war = yes
            surrender_progress > 0.2
            has_country_flag = internal_crisis
        }
    }
}

# 是否可以接收外交援助
can_receive_diplomatic_aid = {
    AND = {
        is_major_or_industrial = yes
        is_in_danger = yes
        NOT = { has_war_with = $AID_PROVIDER$ }
    }
}
```

调用示例：
```
# 国策可用条件
available = {
    is_major_or_industrial = yes
    has_political_power > 150
}

# 决议目标筛选
target_trigger = {
    can_receive_diplomatic_aid = { AID_PROVIDER = ROOT }
}
```

### 示例 3：批量效果 + 复杂参数传递

```
# common/scripted_effects/MY_war_effects.txt

# 战后处理：战胜国得奖励，战败国受惩罚
apply_post_war_effects = {
    $LOSER$ = {
        add_stability = -0.20
        add_war_support = -0.10
        add_political_power = -50
    }
    THIS = {
        add_stability = 0.10
        add_political_power = 100
        set_country_flag = post_war_bonus
    }
}

# 条件性吞并
conditional_annexation = {
    if = {
        limit = {
            $TARGET$ = { check_variable = { var = resistance_score value = 30 compare = less_than } }
        }
        annex_country = { target = $TARGET$ }
    }
    else = {
        puppet = $TARGET$
    }
}
```

## 常见陷阱

1. **作用域泄漏** — Scripted effect 不自动切换作用域。如果在 effect 中写了 `add_stability = 0.05`，它作用于调用时的 THIS，而非你"以为"的对象。如果需要操作其他作用域，必须显式用 `SCOPE = { ... }` 包裹。常见错误：在 peace conference 的 effect 中调用 scripted effect，忘记 THIS 在战后协议中可能不是你想操作的国家。

2. **参数名拼写不一致** — 定义中用 `$AMOUNT$`，调用时写 `AMMOUNT = 0.05`（多了一个 M）→ 参数不会被替换，`$AMOUNT$` 保留为字面量，导致语法错误。

3. **缺少兜底条目** — Scripted localisation 的 `defined_text` 中所有带 `trigger` 的条目都不满足时，如果没有不带 trigger 的兜底条目，文本会空显示。始终在最末尾放一个无条件的兜底条目。

4. **循环依赖** — Scripted effect A 调用 scripted effect B，B 又调用 A → 游戏崩溃或无限递归。保持调用链为单向的树状结构。

5. **参数值包含 `=` 号** — 如果参数值本身包含 `=`（如 `COND = { stability > 0.5 }`），需要用花括号包裹但不要额外处理。但如果在 if/else 的 limit 中传递包含复杂结构的参数，确保花括号对匹配。

## FAQ

**Q: Scripted effect 和普通 macro（复制粘贴）有什么区别？**
A: Scripted effect 在 -debug 模式的 `scripted_effects_dump` 中可见，便于调试。修改一处、所有引用自动更新。调用开销可忽略。

**Q: 可以在 scripted effect 内部使用 `if/else` 吗？**
A: 可以。Scripted effect 中可以使用任何合法的效果语法，包括 `if/else`、变量操作、触发事件、调用其他 scripted effect 等。

**Q: 如何在 scripted trigger 中否定一个复杂的组合条件？**
A: 使用 `NOT = { my_scripted_trigger = yes }`。Scripted trigger 返回布尔值（true/false），与其他条件一样可以被 `NOT` 包裹。

**Q: 可不可以先检查条件再执行效果——在 scripted effect 中实现 guard clause？**
A: 可以。在 scripted effect 内部用 `if = { limit = { <条件> } ... }` 包裹效果。注意 scripted effect 本身不能阻止调用——调用始终发生，只是内部可能因条件不满足而什么都不做。

**Q: Scripted localisation 的 trigger 可以使用脚本化触发器吗？**
A: 可以。`trigger = { my_scripted_trigger = yes }` 在 scripted localisation 的 `text` 条目中完全有效。这是复用复杂条件的最佳方式。

## 检查清单

- [ ] Scripted effects 文件放在 `common/scripted_effects/` 下
- [ ] Scripted triggers 文件放在 `common/scripted_triggers/` 下
- [ ] Scripted localisation 文件放在 `common/scripted_localisation/` 下
- [ ] 所有命名使用唯一前缀避免冲突（如 mod 缩写）
- [ ] 参数名使用 `$NAME$` 包裹，调用时正确传入
- [ ] Scripted localisation 有兜底条目（无 trigger 的最后一条）
- [ ] 确认作用域在调用时正确传递
- [ ] 无循环依赖
- [ ] 文件按功能模块分组，命名清晰

> **相关文件**: PDXscript 语法基础（03-pdxscript.md）介绍作用域、if/else 和变量系统。事件系统（05-events.md）大量使用 scripted effects 简化事件链。本地化（12-localisation.md）详细介绍 scripted_localisation 的文本侧写法。调试排错（14-debugging.md）教你如何使用控制台 dump 脚本化效果列表。
