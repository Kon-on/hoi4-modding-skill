> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

国策树（National Focus Tree）是 HOI4 最核心的内容系统——玩家通过完成国策节点来推动国家发展。每个国策树文件包含一个或多个 `focus_tree` 块（内嵌 `focus` 节点），定义国策的图标、位置、前置条件、互斥分支、完成奖励和 AI 行为。一个完整的国策树还涉及 `shared_focus`（共享国策）、`dynamic=yes`（动态国策布局）、`continuous_focus_position`（持续国策位置）等辅助元素。

国策树文件存放于 `<mod>/common/national_focus/` 目录，文件名任意但必须为 `.txt` 后缀。文件编码为 UTF-8 with BOM（Paradox 标准的 `UTF-8-BOM`）。

## 完整语法参考

### focus_tree 块

```
focus_tree = {
    id = my_custom_focus_tree              # 唯一标识符，用于 event_target 和引用

    country = {                             # 哪些国家使用此国策树
        factor = 0                         # 基础权重为 0（即默认不分配）
        modifier = {                        # 修正条件：满足时权重变为 factor + add
            add = 10                       # 权重增加值
            tag = TAG                      # 目标国家 TAG
            has_dlc = "Expansion Name"     # 需要特定 DLC
        }
    }

    default = no                            # 是否为默认国策树（通用树才设为 yes）

    focus = { ... }                         # 国策节点（可包含无限多个）
}
```

### focus 节点核心属性

```
focus = {
    id = TAG_focus_name                     # 唯一 ID，命名惯例：TAG_描述性名称

    icon = GFX_focus_TAG_name               # 图标引用，对应 gfx/interface/goals/ 下的 dds 文件

    x = 5                                   # 横坐标（相对于显示起点，正数向右）
    y = 2                                   # 纵坐标（正数向下）
    # 通常通过 prerequisite 的 relative_position_id 自动计算位置；
    # 手动设置 x/y 用于根节点或特殊布局

    cost = 10                               # 完成所需天数（必须是 7 的倍数）
    # 常见值：5(35天)、10(70天)、15(105天)、20(140天)、30(210天)

    prerequisite = { focus = TAG_parent }   # 前置国策（必须先完成该国策才能开始本节点）
    # 可列出多个 focus：prerequisite = { focus = A focus = B }
    # 此时需要完成 A 或 B 中任意一个（OR 关系）

    mutually_exclusive = {                  # 互斥分支：选择一个后其他永久锁定
        focus = TAG_branch_a
        focus = TAG_branch_b
    }

    available = {                           # 可见条件：条件满足时国策在面板中显示
        has_war = yes
    }

    allow_branch = {                        # 分支可见控制（配合游戏规则的 obsolete focus branches）
        IF = {
            limit = {
                has_game_rule = {
                    rule = obsolete_focus_branches_visibility
                    option = HIDE
                }
            }
            NOT = { has_completed_focus = TAG_mutually_exclusive_focus }
        }
    }

    bypass = {                              # 跳过条件：为 true 时自动完成（不消耗时间）
        has_war = yes                       # 例如战争爆发时自动解锁动员国策
    }

    cancel = {                              # 取消条件（极少使用）
        NOT = { has_idea = my_idea }
    }

    available_if_capitulated = yes          # 即使国家投降也可选择

    cancel_if_invalid = no                  # 默认 yes；no 表示条件失效后国策不取消

    continue_if_invalid = no                # 默认 no；yes 表示即使条件失效仍继续完成

    relative_position_id = TAG_parent       # 布局参考点：本节点定位以此节点为相对基准

    search_filters = {                      # 搜索筛选标签（游戏内国策搜索用）
        FOCUS_FILTER_POLITICAL
        FOCUS_FILTER_INDUSTRY
        FOCUS_FILTER_HISTORICAL
    }

    # 可用搜素标签：
    # FOCUS_FILTER_POLITICAL        — 政治
    # FOCUS_FILTER_RESEARCH         — 科研
    # FOCUS_FILTER_INDUSTRY         — 工业
    # FOCUS_FILTER_STABILITY        — 稳定度
    # FOCUS_FILTER_WAR_SUPPORT      — 战争支持度
    # FOCUS_FILTER_MANPOWER         — 人力
    # FOCUS_FILTER_ANNEXATION       — 吞并/领土
    # FOCUS_FILTER_POLITICAL_CHARACTER — 政治人物
    # FOCUS_FILTER_MILITARY_CHARACTER — 军事人物
    # FOCUS_FILTER_INTERNAL_AFFAIRS — 内政
    # FOCUS_FILTER_HISTORICAL       — 历史路线

    ai_will_do = {                          # AI 选择权重
        factor = 1                          # 基础权重
        modifier = {                        # 条件修正
            factor = 2                     # 权重乘数（0 = 永不选择，负数降低优先）
            has_war = yes
        }
    }

    completion_reward = {                   # 完成效果（同 event/decision 的 effect 语法）
        add_political_power = 120
        add_stability = 0.05
        add_tech_bonus = {
            name = my_bonus                 # 研究加成名称
            bonus = 1.0                    # 加成倍率（1.0 = 100%）
            uses = 2                       # 可用次数
            category = industry            # 限定类别
        }
    }
}
```

### shared_focus（共享国策）

```
# 在 focus_tree 中引用：
focus_tree = {
    id = my_tree

    shared_focus = SMB_army                # 引用全局共享国策
    shared_focus = SMB_air_force
    shared_focus = SMB_navy
    # shared_focus 定义在单独的共享文件中
}
```

```
# 共享国策定义（通常在共享文件中）：
shared_focus = {
    id = SMB_army
    icon = GFX_goal_generic_army_experience
    cost = 10

    ai_will_do = {
        factor = 1
    }

    available = {
        has_army_experience > 50
    }

    completion_reward = {
        add_army_experience = 25
    }
}
```

### dynamic = yes（动态国策布局）

当国策的 x/y 坐标需要根据前置节点动态计算时使用：

```
focus = {
    id = TAG_dynamic_focus
    icon = GFX_focus_generic
    dynamic = yes                          # 位置从 prerequisite 自动推导
    prerequisite = { focus = TAG_parent }
    x = -1                                 # 相对父节点的横偏移
    y = 1                                  # 相对父节点的纵偏移
    cost = 10

    completion_reward = {
        # 效果
    }
}
```

动态国策需要 `prerequisite` 来确定位置基准。`x` 和 `y` 是相对于前置节点的偏移量，而非绝对坐标。

### continuous_focus_position（持续国策按钮位置）

```
focus_tree = {
    id = my_tree

    continuous_focus_position = { x = 200 y = 2300 }
    # 定义持续国策（如"建造工业"）按钮在国策面板上的显示位置
}
```

### initial_show_position（初始视图位置）

```
focus_tree = {
    id = my_tree

    initial_show_position = {
        x = 25                             # 游戏打开国策树时镜头中心点的 X 坐标
        y = 0                              # 游戏打开国策树时镜头中心点的 Y 坐标
    }
}
```

## 实战示例

### 示例 1：3 节点线性国策树（工业发展路线）

```
focus_tree = {
    id = MYTAG_industrial_focus_tree

    country = {
        factor = 0
        modifier = {
            add = 10
            tag = MYTAG
        }
    }

    default = no

    # 节点 1：基础工业（根节点，无前置）
    focus = {
        id = MYTAG_basic_industry
        icon = GFX_goal_generic_construction
        x = 0
        y = 0
        cost = 10

        ai_will_do = {
            factor = 1
        }

        completion_reward = {
            add_building_construction = {
                type = industrial_complex
                level = 2
                instant_build = yes
            }
        }
    }

    # 节点 2：扩展工厂（前置 = 节点 1）
    focus = {
        id = MYTAG_expand_factories
        icon = GFX_goal_generic_production
        prerequisite = { focus = MYTAG_basic_industry }
        relative_position_id = MYTAG_basic_industry
        x = 0
        y = 1
        cost = 10

        ai_will_do = {
            factor = 1
        }

        completion_reward = {
            add_extra_state_shared_building_slots = 2
            add_tech_bonus = {
                name = MYTAG_industry_bonus
                bonus = 1.0
                uses = 1
                category = industry
            }
        }
    }

    # 节点 3：军事工业化（前置 = 节点 2）
    focus = {
        id = MYTAG_military_industry
        icon = GFX_goal_generic_military_industry
        prerequisite = { focus = MYTAG_expand_factories }
        relative_position_id = MYTAG_expand_factories
        x = 0
        y = 1
        cost = 10

        available = {
            NOT = { has_war = yes }
        }

        ai_will_do = {
            factor = 1
        }

        completion_reward = {
            every_owned_state = {
                limit = {
                    is_core_of = ROOT
                }
                add_building_construction = {
                    type = arms_factory
                    level = 1
                    instant_build = yes
                }
            }
        }
    }
}
```

### 示例 2：互斥分支国策树（外交选择路线）

```
focus_tree = {
    id = MYTAG_diplomacy_focus_tree

    country = {
        factor = 0
        modifier = {
            add = 10
            tag = MYTAG
        }
    }

    default = no

    # 根节点：外交方向
    focus = {
        id = MYTAG_diplomatic_direction
        icon = GFX_goal_generic_improve_relations
        x = 0
        y = 0
        cost = 10

        mutually_exclusive = {
            focus = MYTAG_join_allies
            focus = MYTAG_join_axis
        }

        ai_will_do = {
            factor = 1
        }

        completion_reward = {
            add_political_power = 100
        }
    }

    # 分支 A：与 Allies 结盟
    focus = {
        id = MYTAG_join_allies
        icon = GFX_goal_generic_allies
        prerequisite = { focus = MYTAG_diplomatic_direction }
        mutually_exclusive = { focus = MYTAG_join_axis }
        relative_position_id = MYTAG_diplomatic_direction
        x = -1
        y = 1
        cost = 10

        available = {
            ENG = { exists = yes }
        }

        allow_branch = {
            IF = {
                limit = {
                    has_game_rule = {
                        rule = obsolete_focus_branches_visibility
                        option = HIDE
                    }
                }
                NOT = { has_completed_focus = MYTAG_join_axis }
            }
        }

        ai_will_do = {
            factor = 1
            modifier = {
                factor = 10
                has_government = democratic
            }
            modifier = {
                factor = 0
                has_government = fascism
            }
        }

        search_filters = { FOCUS_FILTER_POLITICAL }

        completion_reward = {
            add_opinion_modifier = {
                target = ENG
                modifier = mytag_pro_allies
            }
            if = {
                limit = {
                    ENG = {
                        is_in_faction = yes
                    }
                }
                add_to_faction = ENG
            }
        }
    }

    # 分支 B：与 Axis 结盟
    focus = {
        id = MYTAG_join_axis
        icon = GFX_goal_generic_axis
        prerequisite = { focus = MYTAG_diplomatic_direction }
        mutually_exclusive = { focus = MYTAG_join_allies }
        relative_position_id = MYTAG_diplomatic_direction
        x = 1
        y = 1
        cost = 10

        available = {
            GER = { exists = yes }
        }

        allow_branch = {
            IF = {
                limit = {
                    has_game_rule = {
                        rule = obsolete_focus_branches_visibility
                        option = HIDE
                    }
                }
                NOT = { has_completed_focus = MYTAG_join_allies }
            }
        }

        ai_will_do = {
            factor = 1
            modifier = {
                factor = 10
                has_government = fascism
            }
            modifier = {
                factor = 0
                has_government = democratic
            }
        }

        search_filters = { FOCUS_FILTER_POLITICAL }

        completion_reward = {
            add_opinion_modifier = {
                target = GER
                modifier = mytag_pro_axis
            }
            if = {
                limit = {
                    GER = {
                        is_in_faction = yes
                    }
                }
                add_to_faction = GER
            }
        }
    }

    # 分支 A 的后续节点
    focus = {
        id = MYTAG_allied_trade
        icon = GFX_goal_generic_trade
        prerequisite = { focus = MYTAG_join_allies }
        relative_position_id = MYTAG_join_allies
        x = 0
        y = 1
        cost = 10

        completion_reward = {
            add_ideas = MYTAG_allied_trade_idea
            every_other_country = {
                limit = {
                    is_in_faction_with = ROOT
                }
                add_opinion_modifier = {
                    target = ROOT
                    modifier = mytag_allied_trade
                }
            }
        }
    }

    # 分支 B 的后续节点
    focus = {
        id = MYTAG_axis_support
        icon = GFX_goal_generic_military_sphere
        prerequisite = { focus = MYTAG_join_axis }
        relative_position_id = MYTAG_join_axis
        x = 0
        y = 1
        cost = 10

        completion_reward = {
            GER = {
                add_opinion_modifier = {
                    target = ROOT
                    modifier = mytag_axis_volunteer
                }
            }
            add_manpower = 50000
        }
    }
}
```

## 常见陷阱

1. **cost 不是 7 的倍数导致 UI 显示异常**——游戏以周为时间单位，cost=10 实际是 10\*7=70 天。虽然任意整数理论上可以工作，但非 7 倍数可能导致 UI 显示不准确。始终使用 5、10、15、20、30 等值（对应 35, 70, 105, 140, 210 天）。

2. **prerequisite 的 OR 与 AND 混淆**——`prerequisite = { focus = A focus = B }` 是 OR 关系（完成 A 或 B 任一即可），不是 AND。如果需要同时完成 A 和 B，必须使用两个独立的 prerequisite 块或使用条件检查：`available = { has_completed_focus = A has_completed_focus = B }`。实际上，原版引擎不支持直接的 AND 前置需求——通常的做法是将前置设为链式关系或使用 available 条件。

3. **mutually_exclusive 双向未声明**——互斥必须在所有涉及节点中互相声明。如果 A 声明 `mutually_exclusive = { focus = B }`，B 也必须声明 `mutually_exclusive = { focus = A }`，否则可能出现一方可完成而另一方的锁定逻辑不完整。

4. **缺少 allow_branch 导致过时分支不隐藏**——当玩家启用"隐藏过时国策分支"游戏规则时，没有 `allow_branch` 块的互斥分支将不会自动隐藏。始终为互斥分支添加 `allow_branch` 块。

5. **坐标冲突/国策重叠**——多个国策使用相同 (x, y) 坐标会导致 UI 重叠。使用 `relative_position_id` 时确保偏移量不冲突，并在完成后运行游戏验证国策面板无重叠节点。

## FAQ

**Q: 如何让国策在特定日期前不可见？**
A: 在 `available` 条件中使用日期检查：`available = { date > 1939.1.1 }`。注意 `available` 控制可见性和可点击性，如果只是想让按钮灰掉但仍可见，需用其他方式。

**Q: 如何让国策自动完成而不消耗时间？**
A: 使用 `bypass` 块。例如：`bypass = { has_war = yes }`——战争爆发时该节点自动跳过（立即获得完成奖励）。

**Q: 同一个 focus_tree 可以给多个国家使用吗？**
A: 可以，在 `country` 块中为每个 TAG 添加 modifier。但注意 completion_reward 中的效果会将 ROOT 设为拥有该树的国家。

**Q: shared_focus 和普通 focus 有什么区别？**
A: `shared_focus` 是可以在多个国策树中复用的国策节点（如原版的 SMB 军事分支）。在 `focus_tree` 中通过 `shared_focus = ID` 引入即可，无需重新定义节点属性。

**Q: dynamic=yes 的实际效果是什么？**
A: 当 `dynamic = yes` 时，国策的最终位置会根据 `prerequisite` 指向的前置节点动态计算。x/y 坐标变为相对于前置节点的偏移而非绝对坐标。这主要用于从共享分支派生的国策。

## 检查清单

- [ ] 每个 `focus` 都有唯一的 `id`（命名规范：TAG_描述性名称）
- [ ] 所有 `cost` 值是 7 的倍数（或至少为整数）
- [ ] 每个 `focus` 都有 `icon` 引用（对应 gfx 目录下的 dds）
- [ ] 坐标无冲突：所有节点的 (x, y) 或 (relative_position_id + offset) 不重叠
- [ ] `mutually_exclusive` 在所有参与节点中双向声明
- [ ] 互斥分支包含 `allow_branch` 块（兼容 obsolete focus branches 规则）
- [ ] `ai_will_do` 不为空（至少 `factor = 1`）
- [ ] 文件编码为 UTF-8 with BOM
- [ ] `focus_tree` 有唯一的 `id`
- [ ] `country` 块正确配置权重和 TAG

> **相关文件**: [05-events.md](05-events.md), [07-ideas.md](07-ideas.md)
