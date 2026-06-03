> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

决议系统（Decisions）是 HOI4 中玩家主动触发的操作面板——区别于国策（自动推进的时间线），决议是玩家在满足条件后手动点击执行的即时操作。决议分为三类：普通 `decision`（对国家自身执行）、`targeted_decision`（对目标国家或省份执行）、和地图决议（`on_map` 模式）。所有决议归属于 `decision_category`（决议类别），在游戏内的决议与事件面板中按类别分组显示。

决议文件存放于 `<mod>/common/decisions/` 目录，类别定义通常在 `categories` 文件中。文件编码为 UTF-8 with BOM。

## 完整语法参考

### decision_category（决议类别）

```
decision_category = {
    id = my_category                    # 唯一的类别标识符

    name = my_category_name             # 类别显示名称本地化 key

    icon = GFX_decision_cat_my_cat      # 类别图标（可选）
    priority = 10                       # 排序优先级（数字越小排在越前面，默认 0）

    allowed_always = yes                # 是否始终显示在面板中（即使无可用决议）

    visible = {                         # 类别可见条件
        has_dlc = "Expansion Name"
    }

    desc = my_category_desc             # 类别描述本地化 key（可选）
}
```

### decision（普通决议）—— 完整模板

```
decision_category = {
    id = my_category
    name = my_category_name
}

decision = {
    name = my_decision                  # 决议名称本地化 key
    icon = GFX_decision_my_icon         # 决议图标

    # === 可见性和可用性（三重过滤）===

    allowed = {                         # 允许条件：决议在面板中显示的前提
        tag = TAG                       # 条件不满足时决议不可见
        has_political_power > 50
    }

    available = {                       # 可用条件：决议可点击的前提
        has_war = yes                   # 条件不满足时决议灰掉但可见
        NOT = { has_country_flag = decision_used }
    }

    visible = {                         # 额外可见条件（独立于 allowed）
        has_government = fascism        # 不满足时隐藏
    }
    # 通常 allowed 和 visible 二者选一即可。
    # allowed 控制"是否存在"，visible 额外过滤"可见性"。

    # === 消耗 ===

    cost = 100                          # 执行所需政治点数（PP）
    # cost = 50 意味着支付 50 政治点数后执行

    # === 定时和取消 ===

    days_remove = 30                    # N 天后自动移除（不执行，直接消失）
    # 通常配合 remove_effect 使用

    remove_effect = {                   # 决议被移除时执行的效果（不管是否被手动执行）
        set_country_flag = decision_expired
    }

    cancel_trigger = {                  # 取消触发条件（条件满足时自动取消决议）
        has_war = no                    # 例如战争结束时取消
    }

    # === 执行效果 ===

    on_map_mode = no                    # 默认 no；yes 表示地图模式决议

    # 地图模式决议专用：
    # on_map_mode = yes
    # states = { 123 456 }             # 限定可选的省份列表
    # 当 on_map_mode=yes 时，complete_effect 中使用 FROM 引用目标省份

    complete_effect = {                 # 决议执行后的效果
        add_political_power = -50      # 注意：cost 和 complete_effect 中的 PP 支出是独立的
        add_stability = 0.05           # cost 是入场费，complete_effect 是执行效果
        set_country_flag = decision_used

        every_owned_state = {
            limit = {
                is_core_of = ROOT
            }
            add_building_construction = {
                type = industrial_complex
                level = 1
            }
        }
    }

    # === AI 行为 ===

    ai_will_do = {                      # AI 选择权重
        factor = 1                      # 基础权重
        modifier = {
            factor = 0                 # 0 = AI 不选
            has_government = democratic
        }
        modifier = {
            factor = 3                 # 权重乘数
            has_war = yes
        }
    }

    # === 定时自动触发（可选）===

    days_mission_timeout = 90           # N 天后决议自动消失（不同于 days_remove）
    # days_remove 和 days_mission_timeout 的区别：
    # - days_remove：出现 N 天后消失
    # - days_mission_timeout：常用于任务型决议（到期后可选择不同的到期效果）

    timeout_effect = {                  # 超时效果（配合 days_mission_timeout 使用）
        add_stability = -0.05
    }
}
```

### targeted_decision（目标决议）

```
targeted_decision = {
    name = my_targeted_decision
    icon = GFX_decision_my_targeted

    # === 目标选择 ===

    target = yes                        # 需要选择目标（必须）
    # target_root = yes                 # 可以以自身为目标（可选）

    target_trigger = {                  # 目标筛选条件
        FROM = {                        # FROM = 执行决议的国家
            NOT = { tag = ROOT }       # ROOT = 目标国家
        }
        has_war_with = FROM             # 目标必须与执行者处于战争
    }

    state_target = yes                  # yes = 目标是省份而非国家
    # target_array 包含可选的多个目标类型

    # === 条件和消耗 ===

    allowed = {
        is_major = yes
    }

    available = {
        has_political_power > 50
    }

    cost = 100

    # === 执行效果 ===

    complete_effect = {
        # ROOT = 目标国家/省份
        # FROM = 决议执行国家

        FROM = {
            add_political_power = -50
        }

        add_opinion_modifier = {
            target = FROM
            modifier = targeted_opinion
        }
    }

    ai_will_do = {
        factor = 0
    }
}
```

### 决议触发方式汇总

```
# 在事件/国策效果中激活决议：
activate_decision = { decision = my_decision }

# 在效果中移除决议：
remove_decision = { decision = my_decision }

# 对特定国家添加决议：
TAG = {
    activate_decision = { decision = my_decision }
}

# 有条件的可见决议（在事件中激活后进入面板）：
# 1. 先 activate_decision
# 2. 在决议的 allowed/available 中设置条件
```

### on_map 地图决议详细语法

```
decision = {
    name = my_map_decision
    icon = GFX_decision_map_icon

    on_map_mode = yes

    # 限定可选省份：
    states = { 123 456 789 }

    # 或排除省份：
    # all_provinces_except = { 123 }

    allowed = {
        tag = TAG
    }

    available = {
        has_war = yes
    }

    highlight = {                       # 目标省份高亮条件
        is_controlled_by = ROOT
    }

    complete_effect = {
        FROM = {                        # FROM = 所选省份
            add_building_construction = {
                type = bunker
                level = 3
            }
        }
        add_political_power = -100
    }
}
```

## 实战示例

### 示例 1：PP 消耗型决议（可重复使用的政策决议）

```
decision_category = {
    id = my_policy_category
    name = my_policy_category_name
    priority = 5
}

decision = {
    name = my_war_propaganda
    icon = GFX_decision_war_propaganda

    allowed = {
        has_war = yes
    }

    available = {
        has_political_power > 50
        NOT = { has_country_flag = propaganda_active }
    }

    cost = 50

    complete_effect = {
        set_country_flag = propaganda_active
        add_war_support = 0.05
        hidden_effect = {
            country_event = { id = my_mod.20 days = 180 }
        }
    }

    ai_will_do = {
        factor = 1
        modifier = {
            factor = 5
            war_support < 0.5
        }
        modifier = {
            factor = 0
            war_support > 0.8
        }
    }
}

# 对应事件：180天后 propaganda 过期
# country_event = {
#     id = my_mod.20
#     is_triggered_only = yes
#     hidden = yes
#     immediate = {
#         clr_country_flag = propaganda_active
#     }
# }
```

### 示例 2：定时决议（期限内完成可获得奖励）

```
decision_category = {
    id = my_timed_category
    name = my_timed_category_name
    priority = 2
}

decision = {
    name = my_industrial_drive
    icon = GFX_decision_industrial_drive

    allowed = {
        tag = MYTAG
        NOT = { has_completed_focus = MYTAG_full_industrialization }
    }

    available = {
        has_political_power > 100
        num_of_factories < 30
    }

    cost = 100

    days_remove = 365                   # 一年后自动消失
    remove_effect = {
        # 移除时添加失败效果
        add_stability = -0.05
        set_country_flag = industrial_drive_failed
    }

    cancel_trigger = {
        has_completed_focus = MYTAG_full_industrialization
    }

    complete_effect = {
        add_stability = 0.10
        add_extra_state_shared_building_slots = 3
        every_owned_state = {
            limit = {
                is_core_of = ROOT
                free_building_slots > 0
            }
            add_extra_state_shared_building_slots = 1
        }
        set_country_flag = industrial_drive_completed
        hidden_effect = {
            remove_decision = { decision = my_industrial_drive }
        }
    }

    ai_will_do = {
        factor = 1
        modifier = {
            factor = 5
            num_of_factories < 20
        }
    }
}
```

## 常见陷阱

1. **allowed 和 available 的语义混淆**——这是最常犯的错误。`allowed` 控制决议是否"存在"（不满足时决议不可见），`available` 控制决议是否可点击（不满足时决议灰色但可见）。通常的做法是：`allowed` 用于永久性条件（如 tag、has_dlc），`available` 用于临时性条件（如 PP、war status）。如果只想用其中一个，用 `allowed` 即可。

2. **cost 的 PP 消耗和 complete_effect 中的 PP 消耗叠加**——`cost = 50` 是点击决议的门票费，`complete_effect` 中如果还有 `add_political_power = -30`，则总共消耗 80 PP。不要让 cost 和效果中的支出意外叠加。

3. **days_remove 触发 remove_effect 但不阻止执行**——如果在 days_remove 到期前玩家点击执行，`remove_effect` 仍然会在到达截止日时触发。如果要避免这种情况，在 `complete_effect` 中先移除决议（`remove_decision = { decision = my_decision }`），或使用 `if` 判断标记。

4. **targeted_decision 中 FROM 和 ROOT 的作用域混合**——在 `targeted_decision` 中，`ROOT` = 目标国家/省份，`FROM` = 执行决议的国家。这与事件中的作用域习惯相反，容易写反。建议始终在 `targeted_decision` 的 `complete_effect` 中注释标记 FROM/ROOT 的指向。

5. **on_map 决议的 states 必须存在于当前地图**——如果 `states = { 9999 }` 是一个无效省份 ID，游戏不会报错但决议永远无法使用。始终用游戏内的 `tdebug` 命令确认省份 ID。

## FAQ

**Q: 如何让决议只能执行一次？**
A: 在 `complete_effect` 中设置一个 country_flag，并在 `available` 中检查 `NOT = { has_country_flag = decision_done }`。或者直接移除决议：`hidden_effect = { remove_decision = { decision = my_decision } }`。

**Q: 如何让决议在特定国策完成后才能使用？**
A: 在 `available` 中使用 `has_completed_focus = TAG_focus_id`。如果决议要在该 `focus_tree` 的国策完成前不可见，使用 `allowed = { has_completed_focus = TAG_focus_id }`。

**Q: days_remove 和 cancel_trigger 有什么区别？**
A: `days_remove = 30` 是绝对时间限制——30 天后无论条件如何都移除决议。`cancel_trigger = { has_war = no }` 是条件性取消——当条件满足时移除决议。两者可以共存。

**Q: targeted_decision 和普通 decision 可以在同一类别中混用吗？**
A: 可以。`decision_category` 可以同时包含 `decision` 和 `targeted_decision`，它们会在面板中使用不同的 UI 布局（普通决议显示为按钮列表，目标决议需要额外的目标选择步骤）。

**Q: on_map 决议如何在实际地图上选择目标省份？**
A: 当 `on_map_mode = yes` 时，点击决议按钮后游戏会进入地图选择模式。玩家点击地图上的省份来执行效果。`states` 列表限定可选省份，`highlight` 用于高亮符合条件的省份。执行时 `FROM` 指向所选省份。

## 检查清单

- [ ] 所有决议归属于有效的 `decision_category`
- [ ] 每个 `decision` 有唯一的 `name` key（决议名称不会与其他决议冲突）
- [ ] `icon` 引用存在于 gfx 目录
- [ ] `allowed` / `available` / `visible` 使用正确的语义
- [ ] 如果有 `cost`，确认金额合理且不与其他消耗叠加
- [ ] `days_remove` 搭配 `remove_effect` 或不搭配（根据需要）
- [ ] `cancel_trigger` 的逻辑与其他条件不冲突
- [ ] `targeted_decision` 中 `target_trigger` 的 FROM/ROOT 作用域正确
- [ ] `targeted_decision` 有 `target = yes` 声明
- [ ] `on_map_mode = yes` 时 `states` 列表包含有效省份 ID
- [ ] 文件编码为 UTF-8 with BOM
- [ ] 所有本地化 key（`name`）在 localisation 文件中有定义

> **相关文件**: [05-events.md](05-events.md), [07-ideas.md](07-ideas.md)
