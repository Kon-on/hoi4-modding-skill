> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

事件系统是 HOI4 叙事和机制驱动的核心——通过事件可以触发弹窗、执行效果、创建选项分支，以及控制游戏节奏。HOI4 支持三种主要事件类型：`country_event`（国家事件，最常用）、`news_event`（新闻事件，多个国家可见）和 `state_event`（地区事件）。事件可以随机触发（基于 MTTH 平均触发时间）或被其他系统主动触发（如国策完成效果）。

事件文件存放于 `<mod>/events/` 目录，文件名任意但必须为 `.txt` 后缀。文件头部必须声明命名空间（`add_namespace = my_namespace`），事件 ID 使用 `namespace.序号` 格式。

## 完整语法参考

### country_event（国家事件）—— 完整模板

```
add_namespace = my_mod                  # 命名空间声明（每个事件文件开头必须声明）

country_event = {
    id = my_mod.1                       # 事件 ID：namespace.序号（同 namespace 下序号不能重复）

    title = my_mod.1.t                  # 标题本地化 key（引用 localisation 文件中的 key）
    desc = my_mod.1.desc                # 描述文本本地化 key
    picture = GFX_report_event_xxx      # 事件配图（引用 gfx 目录下的图片资源）

    # === 触发方式（三选一或组合）===

    is_triggered_only = yes             # 仅被触发：不会随机发生，只能通过其他效果主动触发
    # 如果省略 is_triggered_only，则需要 mean_time_to_happen 定义随机触发

    fire_only_once = yes                # 全局只触发一次（整个游戏过程中此事件只发生一次）

    trigger = {                         # 触发条件（MTTH 事件的触发前提）
        tag = TAG                       # 事件发送给哪个 TAG 的国家
        has_war = yes                   # 必须满足的条件
        NOT = {
            has_country_flag = event_fired_flag
        }
    }

    mean_time_to_happen = {             # 平均触发时间（省略 is_triggered_only 时必须提供）
        days = 30                       # 基础平均天数（天数越短触发越频繁）
        months = 6                      # 也可用 months 指定月数

        modifier = {                    # 条件修正因子
            factor = 0.5               # 乘数（< 1 则触发更快，> 1 则更慢，0 = 永不触发）
            has_government = fascism
        }
        modifier = {
            factor = 2.0
            is_major = yes             # 主要国家触发概率降低
        }
    }

    # === 特殊属性 ===

    major = yes                         # 标记为主要事件（触发时弹出独立弹窗而非右侧通知）

    hidden = yes                        # 隐藏事件：效果在后台执行，不显示弹窗

    trigger = {                         # 注：trigger 仅用于 MTTH 事件
        # ...                           # 随机触发事件必须有 trigger 和 mean_time_to_happen
    }

    immediate = {                       # 直接效果块：事件弹出前立即执行的效果
        hidden_effect = {
            set_country_flag = event_is_firing
        }
    }

    after = {                           # 延迟效果：事件选项执行之后执行
        hidden_effect = {
            country_event = { id = my_mod.10 days = 30 }
        }
    }

    # === 选项（至少 1 个）===

    option = {
        name = my_mod.1.a               # 选项按钮文本本地化 key
        trigger = {                     # 选项可见条件（不满足时该选项灰掉但显示）
            has_political_power > 100
        }

        ai_chance = {                   # AI 选择权重
            base = 10                   # 基础权重（非概率，是相对权重）
            modifier = {                # 条件修正
                factor = 0             # 0 = 永不选择此选项
                has_government = democratic
            }
        }

        # 效果块：
        add_political_power = -100     # 简单效果
        add_stability = 0.10           # 稳定度 +10%
        set_country_flag = my_flag     # 设置国家标记

        TAG = {                         # 作用域切换：对其他国家执行效果
            add_opinion_modifier = {
                target = ROOT
                modifier = gratitude
            }
        }

        hidden_effect = {               # 隐藏效果：不显示在提示文本中
            country_event = { id = my_mod.2 days = 7 }
        }
    }

    option = {                          # 第二个选项
        name = my_mod.1.b
        ai_chance = { base = 0 }
        # 条件不满足时此选项仍可能显示但不可选
    }
}
```

### news_event（新闻事件）

```
news_event = {
    id = my_mod.100
    title = my_mod.100.t
    desc = my_mod.100.desc
    picture = GFX_report_event_generic_treaty

    is_triggered_only = yes

    major = yes                         # 新闻事件通常标记为 major

    option = {
        name = my_mod.100.a
        # 新闻事件通常只用于通知，不执行效果
    }
}

# 在效果中触发 news_event：
# news_event = { id = my_mod.100 }
# 或对特定国家：
# TAG = { news_event = { id = my_mod.100 } }
```

`news_event` 与 `country_event` 的主要区别：news_event 默认对所有满足条件的国家可见，而 country_event 只对指定的 THIS 国家可见。news_event 通常只用于展示信息（如"慕尼黑协定签署"），不在选项中执行效果。

### state_event（地区事件）

```
state_event = {
    id = my_mod.200
    title = my_mod.200.t
    desc = my_mod.200.desc
    picture = GFX_report_event_generic_strike

    is_triggered_only = yes

    option = {
        name = my_mod.200.a
        add_building_construction = {
            type = infrastructure
            level = 1
            instant_build = yes
        }
    }
}

# 触发方式：
# state_event = { id = my_mod.200 state = 123 }
```

### 事件触发方式对比

| 方式 | 说明 | 适用场景 |
|------|------|---------|
| `is_triggered_only = yes` | 仅被其他效果主动触发 | 国策奖励、决议链、脚本剧情 |
| `mean_time_to_happen = { ... }` | 基于 MTTH 随机触发 | 随机事件、周期性检查、动态反馈 |
| `trigger = { ... }` | 搭配 MTTH 定义触发条件 | 需要特定条件才能随机发生的事件 |
| `immediate = { ... }` | 事件弹出前立即执行 | 设置标记、保存事件目标 |
| `hidden = yes` | 后台执行不弹窗 | 纯逻辑事件、自动处理 |

### MTTH 计算原理

MTTH（Mean Time To Happen）不是固定间隔，而是概率计算：游戏每 20 天（默认检查间隔）计算一次事件触发概率 = 20 / (MTTH 天数 * 所有 modifier 的 factor 乘积)。例如 MTTH=30 天且无 modifier 时，每 20 天有 20/30 = 66.7% 的概率触发。

```
mean_time_to_happen = {
    days = 30                           # 基础 MTTH 30 天
    modifier = {
        factor = 0.5                   # 触发速度加倍（MTTH 变为 15 天）
        has_government = fascism
    }
    modifier = {
        factor = 2.0                   # 触发速度减半（MTTH 变为 60 天）
        has_war = no
    }
}
```

### 事件触发效果语法

```
# 触发国家事件：
country_event = { id = my_mod.1 }
country_event = { id = my_mod.1 days = 30 }     # 30 天后触发
country_event = { id = my_mod.1 hours = 12 }    # 12 小时后触发（极少用）

# 对特定国家触发：
TAG = { country_event = { id = my_mod.1 } }
FROM = { country_event = { id = my_mod.1 } }

# 触发新闻事件：
news_event = { id = my_mod.100 }

# 触发地区事件：
random_owned_controlled_state = {
    state_event = { id = my_mod.200 }
}
123 = { state_event = { id = my_mod.200 } }     # 对特定省份 123 触发

# 延迟触发（在其他效果块中）：
hidden_effect = {
    country_event = { id = my_mod.2 days = 14 }
}
```

## 实战示例

### 示例 1：MTTH 随机事件（周期性战争疲劳）

```
add_namespace = my_mod

# 战争疲劳随机事件：每约 90 天检查一次，影响稳定度
country_event = {
    id = my_mod.1
    title = my_mod.1.t
    desc = my_mod.1.desc
    picture = GFX_report_event_generic_war

    fire_only_once = no

    trigger = {
        has_war = yes
        surrender_progress < 0.5
        NOT = { has_country_flag = recent_war_fatigue }
    }

    mean_time_to_happen = {
        days = 90
        modifier = {
            factor = 0.7          # 更快触发
            surrender_progress > 0.2
        }
        modifier = {
            factor = 0.5          # 更快触发
            war_support < 0.3
        }
        modifier = {
            factor = 2.0          # 更慢触发
            has_government = fascism
        }
    }

    immediate = {
        hidden_effect = {
            set_country_flag = recent_war_fatigue
        }
    }

    # 选项 A：维持现状
    option = {
        name = my_mod.1.a
        ai_chance = {
            base = 10
            modifier = {
                factor = 2
                stability > 0.7
            }
        }
        add_stability = -0.05
        add_war_support = -0.02
    }

    # 选项 B：战争宣传（需要 PP）
    option = {
        name = my_mod.1.b
        trigger = {
            has_political_power > 50
        }
        ai_chance = {
            base = 5
            modifier = {
                factor = 3
                has_government = fascism
            }
        }
        add_political_power = -50
        add_war_support = 0.05
        hidden_effect = {
            set_country_flag = propaganda_used
        }
    }

    # 选项 C：寻求和平（仅民主政体）
    option = {
        name = my_mod.1.c
        trigger = {
            has_government = democratic
        }
        ai_chance = {
            base = 0               # AI 不选
        }
        add_stability = 0.05
        add_war_support = -0.10
        hidden_effect = {
            country_event = { id = my_mod.2 days = 30 }
        }
    }
}
```

### 示例 2：触发式多选项事件链（政治危机）

```
add_namespace = my_mod

# 事件 1：政治危机爆发（由国策或其他效果触发）
country_event = {
    id = my_mod.10
    title = my_mod.10.t
    desc = my_mod.10.desc
    picture = GFX_report_event_generic_political_crisis

    is_triggered_only = yes
    fire_only_once = yes
    major = yes

    # 选项 A：强硬镇压
    option = {
        name = my_mod.10.a
        ai_chance = {
            base = 10
            modifier = {
                factor = 5
                has_government = fascism
            }
            modifier = {
                factor = 0
                has_government = democratic
            }
        }
        add_stability = -0.10
        add_political_power = 50
        set_country_flag = crisis_suppressed
        hidden_effect = {
            country_event = { id = my_mod.11 days = 7 }
        }
    }

    # 选项 B：妥协让步
    option = {
        name = my_mod.10.b
        ai_chance = {
            base = 10
            modifier = {
                factor = 5
                has_government = democratic
            }
        }
        add_stability = 0.05
        add_political_power = -100
        add_popularity = {
            ideology = democratic
            popularity = 0.05
        }
        set_country_flag = crisis_compromised
        hidden_effect = {
            country_event = { id = my_mod.12 days = 7 }
        }
    }

    # 选项 C：请求外国调停
    option = {
        name = my_mod.10.c
        trigger = {
            any_major_country = {
                NOT = { tag = ROOT }
                has_government = democratic
            }
        }
        ai_chance = {
            base = 5
        }
        add_political_power = -50
        set_country_flag = crisis_mediation
        hidden_effect = {
            random_other_country = {
                limit = {
                    is_major = yes
                    has_government = democratic
                    NOT = { tag = ROOT }
                }
                save_event_target_as = mediator_country
                country_event = { id = my_mod.13 days = 3 }
            }
        }
    }
}

# 事件 11：镇压后续——军警权力扩大
country_event = {
    id = my_mod.11
    title = my_mod.11.t
    desc = my_mod.11.desc
    picture = GFX_report_event_generic_military_police

    is_triggered_only = yes

    option = {
        name = my_mod.11.a
        add_ideas = my_mod_martial_law
        custom_effect_tooltip = my_mod.11_effect_tt
        every_owned_state = {
            limit = {
                is_core_of = ROOT
            }
            add_building_construction = {
                type = bunker
                level = 1
                instant_build = yes
            }
        }
    }
}

# 事件 12：妥协后续——社会和解
country_event = {
    id = my_mod.12
    title = my_mod.12.t
    desc = my_mod.12.desc
    picture = GFX_report_event_generic_treaty

    is_triggered_only = yes

    option = {
        name = my_mod.12.a
        add_ideas = my_mod_national_reconciliation
        add_stability = 0.10
        add_war_support = -0.05
    }
}

# 事件 13：外国调停——调停国家视角
country_event = {
    id = my_mod.13
    title = my_mod.13.t
    desc = my_mod.13.desc
    picture = GFX_report_event_generic_diplomacy

    is_triggered_only = yes

    option = {
        name = my_mod.13.a
        ai_chance = { base = 10 }
        add_political_power = 25
        FROM = {
            add_stability = 0.10
            add_opinion_modifier = {
                target = ROOT
                modifier = grateful_for_mediation
            }
        }
    }

    option = {
        name = my_mod.13.b
        ai_chance = { base = 0 }
        add_political_power = -10
        FROM = {
            add_stability = -0.05
        }
    }
}
```

## 常见陷阱

1. **is_triggered_only 和 mean_time_to_happen 混用**——这两个属性是互斥的。`is_triggered_only = yes` 表示事件只能被其他效果触发，不应该有 `mean_time_to_happen` 块。如果同时指定两者，事件行为不确定。

2. **事件 ID 重复导致静默覆盖**——同一命名空间内的事件 ID 必须唯一。如果两个事件文件使用相同的 namespace 和相同的序号，后加载的文件会覆盖前者，且不会报错。建议为每个文件使用独立的 namespace。

3. **trigger 块位置错误**——在 `is_triggered_only = yes` 的事件中，`trigger` 块无效（触发方式决定事件何时发生，而不是条件）。`trigger` 仅用于 MTTH 事件的触发条件判断。

4. **ai_chance 的 base 值是非归一化相对权重**——`ai_chance = { base = 10 }` 不是 10% 的概率，而是与其他选项的相对权重。如果两个选项的 base 分别是 10 和 5，则 AI 选择第一个的概率是 10/(10+5) = 66.7%。

5. **遗忘 add_namespace 导致事件不加载**——每个事件文件开头的 `add_namespace = xxx` 是必须的。如果忘记声明，该文件中的所有事件将被跳过。日志中不会明确报错，事件简单地不触发。

## FAQ

**Q: 如何在一个国家的国策完成效果中触发事件？**
A: 在 `completion_reward` 中使用：`country_event = { id = my_mod.10 }`。如果要在 `completion_reward` 中触发事件给其他国家，使用：`TAG = { country_event = { id = my_mod.10 } }`。

**Q: fire_only_once 的"一次"是指每个国家一次还是全局一次？**
A: 全局一次——整个游戏进程中此事件 ID 只触发一次，无论哪个国家。如果需要每个国家触发一次，使用 `set_country_flag` + `NOT = { has_country_flag = xxx }` 的 trigger 组合。

**Q: hidden_effect 和普通的 effect 有什么区别？**
A: `hidden_effect` 内的效果不会显示在选项的工具提示中（鼠标悬停在选项上时不显示），而直接写在 option 中的效果会显示。用 `hidden_effect` 隐藏实现细节（如设置标记、触发后续事件），用直接效果展示玩家可见的变化。

**Q: 如何让事件延迟执行效果？**
A: 使用延迟触发：`country_event = { id = my_mod.2 days = 30 }` 在 30 天后触发另一个事件。或在 `after` 块中执行效果（选项被选择后执行）。

**Q: 一个事件文件可以有多个 namespace 吗？**
A: 技术上可以多次使用 `add_namespace` 声明不同命名空间，但推荐一个文件一个 namespace，保持清晰的对应关系。

## 检查清单

- [ ] 文件头部有 `add_namespace = xxx` 声明
- [ ] 事件 ID 格式为 `namespace.序号`，同 namespace 内不重复
- [ ] `title`、`desc`、选项的 `name` 都引用了本地化 key
- [ ] `is_triggered_only` 和 `mean_time_to_happen` 不同时出现
- [ ] MTTH 事件的 `trigger` 块正确放置（在最外层，不在 option 中）
- [ ] 每个事件至少有一个 `option`
- [ ] 每个 option 都有 `name` key
- [ ] `ai_chance` 使用 `base`（非 `factor`）作为基础值
- [ ] `hidden_effect` 嵌套正确（在 option 内）
- [ ] 事件配图 `picture` 引用了存在的 GFX 资源
- [ ] 文件编码为 UTF-8 with BOM

> **相关文件**: [04-focus-trees.md](04-focus-trees.md), [12-localisation.md](12-localisation.md)
