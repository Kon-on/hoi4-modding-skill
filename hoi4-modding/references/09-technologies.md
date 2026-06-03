> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

科技树系统定义游戏中可研究的科技。每个科技是一个 `technology = { ... }` 块，通过 `categories` 分类、`tier` 层级和 `prerequisites` 前置条件组织成树状结构。科技可以解锁装备（通过 `equipment` 引用）、添加国家精神、触发事件，或通过 `on_research_complete` 执行任意效果。子科技（`sub_technologies`）支持同一科技槽位下的多层级研究。文件位于 `<mod>/common/technologies/` 下，推荐按类别分文件（如 `my_industry.txt`、`my_infantry.txt`）。

## 完整语法参考

### 科技定义块

```
technology = {
    # ── 基本信息 ──
    enable_tech_tab_section = yes       # 在科技树界面显示此科技（默认 yes）

    # ── 分类与层级 ──
    categories = {                      # 所属科技类别（必须）
        infantry_tech                   # 需匹配 common/technology_tags/ 中的定义
    }
    tier = 1                            # 科技层级：0=开局已研究, 1=最早可研究
                                        # 决定图标在科技树中的 Y 轴位置

    # ── 前置条件 ──
    prerequisites = { infantry_0 }      # 必须研究完的前置科技列表
                                        # 多个时为 AND 关系

    # ── 研究属性 ──
    research_cost = 2.0                 # 基础研究点数成本（天数×日产出）
                                        # 标准：tier1=1.7, tier2=2.0, tier3=2.5, tier4=3.0
    start_year = 1936                   # 最早可开始研究的年份
    ahead_of_time_penalty = yes         # yes=提前研究有惩罚（默认 yes）
    ahead_of_time_penalty_factor = 2.0  # 惩罚倍率（默认 2.0, 每年提前×2）

    # ── 国家限制 ──
    allow_branch = {                    # 可选：AI 研究此分支的权重条件
        # trigger 条件
    }

    # ── 子科技（同一槽位多层级）──
    sub_technologies = {                # 可选：一个 UI 槽位包含多个连续科技
        infantry_1                      # 按列表顺序研究，上一个完成才能研究下一个
        infantry_2
        infantry_3
    }
    # 注意：使用 sub_technologies 后，外层 technology={} 不再直接作为科技；
    # 所有属性（cost, year 等）应在子科技中定义，外层仅作为容器。

    # ── 图标与界面 ──
    folder = {                          # 科技文件夹（组织 UI 分组）
        name = infantry_folder          # 文件夹名称 key
        position = { x = 0 y = 0 }      # 在科技树中的位置
        # tiers 在外层定义
    }

    # ── 完成效果 ──
    on_research_complete = {            # 研究完成时执行的效果
        # 可包含任意 effect 语句
    }

    # ── 特殊属性 ──
    path = {                            # 可选：升级路径（如火炮升级）
        leads_to_tech = improved_artillery
        research_cost_coeff = 1.0       # 成本系数
    }

    equipment = {                       # 可选：解锁的装备类型
        infantry_equipment = {
            # 装备解锁定义
        }
    }

    # ── AI 行为 ──
    ai_will_do = {                      # AI 研究此科技的倾向
        factor = 1.0                    # 基础权重
        modifier = {                    # 修正条件
            factor = 2.0
            has_war = yes               # 战时 AI 更有兴趣
        }
    }

    # ── 互斥科技 ──
    mutually_exclusive = { some_tech }  # 与另一科技互斥（二选一）
}
```

### 子科技（sub_technologies）语法

当多个科技在 UI 上占据同一槽位时使用：

```
technology = {
    enable_tech_tab_section = yes       # 在 UI 中显示此组

    path = { leads_to_tech = next_tier_tech }

    # 子科技的公共设置（可选）
    categories = { infantry_tech }
    tier = 1
    folder = { name = infantry_folder position = { x = 0 y = 0 } }

    sub_technologies = {
        infantry_weapons = {
            # 这个子科技的独立属性
            start_year = 1936
            research_cost = 1.7
            on_research_complete = { ... }
            equipment = { ... }
        }
        infantry_weapons1 = {
            start_year = 1939
            research_cost = 2.0
            prerequisites = { infantry_weapons }  # 子科技也可以有独立前置
            on_research_complete = { ... }
        }
    }
}
```

### 科技文件夹（folder）语法

独立于 `technology` 块定义，用于组织科技树的 UI 布局：

```
folder = {
    name = infantry_folder
    position = { x = 2 y = 0 }
    categories = { infantry_tech }
    tiers = { 0 1 2 3 4 }              # 此文件夹包含的层级
}
```

### 装备解锁（通过科技）

科技研究完成后可以解锁新装备类型或升级装备属性：

```
on_research_complete = {
    # 方式 1：直接引用装备 archetype 使其可用
    # （装备 archetype 中设置 is_buildable = yes 且与科技 ID 关联）

    # 方式 2：使用 effect 解锁
    add_equipment_to_stockpile = {
        type = infantry_equipment_1
        amount = 1000
    }
}

equipment = {
    infantry_equipment = {
        # 此块定义科技如何提升现有装备
        # 引用装备 archetype 中的 upgrade 路径
    }
}
```

### 学说科技（Doctrine Techs）

学说科技有特殊属性，通常使用类别 `doctrine` 或 `land_doctrine` / `naval_doctrine` / `air_doctrine`：

```
technology = {
    categories = { land_doctrine }
    tier = 1
    start_year = 1936
    research_cost = 1.7

    doctrine = yes                      # 标记为学说科技（影响部分 UI 逻辑）

    on_research_complete = {
        add_doctrine_cost_reduction = {
            name = mobile_warfare
            cost = 0.5
        }
        add_ideas = doctrine_idea_mobile_warfare
    }
}
```

### 科技标签分类

在 `common/technology_tags/` 中定义可用的科技类别：

```
infantry_tech = {
    name = INFANTRY_TECH_CATEGORY
    icon = GFX_tech_category_infantry
    position = { x = 0 y = 0 }
}
```

## 实战示例

### 示例 1：单一新科技——解锁步兵装备升级

文件：`mod/common/technologies/my_infantry.txt`

```
technology = {
    categories = { infantry_tech }
    tier = 2

    prerequisites = { basic_infantry_weapons }

    start_year = 1940
    research_cost = 2.5

    folder = {
        name = infantry_folder
        position = { x = 0 y = 3 }
    }

    on_research_complete = {
        # 提升步兵装备属性（通过 upgrade 系统）
        add_tech_bonus = {
            bonus = 1.0
            uses = 1
            category = infantry_tech
        }
        # 直接给国家精神表示完成
        add_ideas = my_improved_weapons
    }

    ai_will_do = {
        factor = 1.0
        modifier = {
            factor = 1.5
            has_war = yes
        }
    }
}
```

### 示例 2：三层级科技树（子科技）

文件：`mod/common/technologies/my_tank_tech.txt`

```
# ── 坦克科技组 ──
technology = {
    categories = { armor_tech }
    tier = 1
    folder = { name = armor_folder position = { x = 2 y = 0 } }

    sub_technologies = {
        basic_tank = {
            start_year = 1936
            research_cost = 1.7
            equipment = {
                light_tank_chassis = {
                    # 可在此定义 archetype 关联
                }
            }
            on_research_complete = {
                add_ideas = tank_manufacturer_basic
            }
        }
        improved_tank = {
            start_year = 1939
            research_cost = 2.0
            prerequisites = { basic_tank }       # 子科技间的前置关系
            on_research_complete = {
                add_ideas = tank_manufacturer_improved
            }
        }
        advanced_tank = {
            start_year = 1943
            research_cost = 2.5
            prerequisites = { improved_tank }
            on_research_complete = {
                add_ideas = tank_manufacturer_advanced
            }
        }
    }
}
```

### 示例 3：学说科技——附带条件互斥

文件：`mod/common/technologies/my_doctrine.txt`

```
technology = {
    categories = { land_doctrine }
    tier = 1

    prerequisites = { basic_training }

    start_year = 1938
    research_cost = 2.0

    doctrine = yes

    mutually_exclusive = { alternative_doctrine }  # 与另一学说二选一

    on_research_complete = {
        add_ideas = my_offensive_doctrine
        add_army_experience = 25
    }

    ai_will_do = {
        factor = 1.0
        modifier = {
            factor = 2.0
            tag = GER             # 德国 AI 倾向选择进攻学说
        }
        modifier = {
            factor = 0
            has_war = no          # 和平时不研究
            date > 1942.1.1       # 但 1942 年后即使和平也考虑
        }
    }
}
```

## 常见陷阱

1. **子科技中 prerequisites 引用错误** — 子科技内的 `prerequisites` 必须引用该 `sub_technologies` 列表内的另一个子科技 ID，不能引用外部科技。跨组前置关系需在主 `technology` 块的 `prerequisites` 中声明。

2. **tier=0 的科技不会出现在科技树** — tier=0 表示开局已研究科技，这类科技不需要 `start_year`、`research_cost`，且不会在 UI 中显示为可点击节点。如果想让科技可见但已研究，使用 `start_tech = yes` 属性。

3. **research_cost 和 ahead_of_time_penalty 的叠加** — 如果设置 `ahead_of_time_penalty_factor = 2.0`，在 `start_year - 1` 年研究时实际成本 = `research_cost × 2.0`；在 `start_year - 2` 年时为 `research_cost × 4.0`（指数增长）。确保基础成本合理以免出现天文数字的研究时间。

4. **文件夹位置重叠导致 UI 错乱** — 两个不同的 folder 如果 `position` 相同或接近，科技树 UI 会出现节点重叠。使用不同的 `x` 值按类别排列，`y` 值对应 tier 来避免冲突。

5. **装备 archetype 与科技 ID 命名不匹配** — 游戏通过 naming convention 自动关联科技和装备 archetype。如果科技 ID 是 `infantry_weapons_2`，装备 archetype 应命名为 `infantry_weapons_2`（或在其 `enable_equipment_type` 中显式引用）。命名不一致是装备不解锁的常见原因。

## FAQ

**Q: 如何让一个科技在开局就已完成？**
A: 设置 `start_tech = yes` 或在国家历史的 `set_technology = { my_tech = 1 }` 中设为已研究。注意这仍然需要 `tier = 0` 以使该科技不在科技树中显示为可研究节点。也可以设置 `enable_tech_tab_section = no` 完全隐藏。

**Q: 子科技和普通科技的区别是什么？**
A: 子科技在 UI 上占据同一个"格子"——它们按顺序排列，玩家必须依次研究。普通科技在科技树中占据独立的格子。子科技适合同一装备的逐步升级（如"步枪 Mk I → Mk II → Mk III"），普通科技适合概念性分支（如"闪电战 vs 大纵深"）。

**Q: 如何让科技只对特定国家可用？**
A: 使用 `allow_branch` 块添加条件触发器，或在 `on_research_complete` 中使用 `hidden_effect` 配合 `limit`。对于国家专属科技树，更推荐的做法是使用 `categories` 中定义专属分类，并通过国策树的 `enable_tech_tab_section` 来控制科技树标签页的开启。

**Q: ahead_of_time_penalty_factor 的值代表什么？**
A: 默认值为 2.0。每提前一年研究（相对于 `start_year`），研究成本乘以该因子。公式：实际成本 = `research_cost × (ahead_of_time_penalty_factor ^ 提前年数)`。设置 `ahead_of_time_penalty = no` 则完全忽略此惩罚。

**Q: 学说科技需要特殊标记吗？**
A: 需要设置 `doctrine = yes` 来告知游戏此科技为学说。学说科技在研究槽位和 UI 中有特殊行为（如学说科技使用独立的陆军/海军/空军经验加速机制）。学说科技通常使用 `land_doctrine`、`naval_doctrine` 或 `air_doctrine` 类别，也可自定义但需在 `technology_tags` 中声明。

## 检查清单

- [ ] 科技 ID 使用了唯一前缀，不与 vanilla 或其他 mod 冲突
- [ ] `categories` 正确引用了 `common/technology_tags/` 中定义的标签
- [ ] `tier` 值合理（0=开局已研究，1+=可研究）
- [ ] `prerequisites` 指向的科技 ID 确实存在
- [ ] `start_year` 在合理的范围内（通常是 1936-1945）
- [ ] `research_cost` 与同 tier vanilla 科技成本一致（参考：tier1≈1.5-1.7, tier2≈2.0-2.2, tier3≈2.5-3.0）
- [ ] 如有子科技，子科技间的 `prerequisites` 在 `sub_technologies` 列表内部
- [ ] `folder` 的 `position` 与同类科技不重叠
- [ ] `on_research_complete` 中引用的 idea/event ID 已正确定义
- [ ] AI 行为通过 `ai_will_do` 合理配置
- [ ] 学说科技已设置 `doctrine = yes`

> **相关文件**: 装备系统（10-equipment.md）——科技通过 `equipment` 和 archetype 解锁装备。国家精神系统（07-ideas.md）——`on_research_complete` 常配合 `add_ideas` 使用。本地化系统（12-localisation.md）——科技名称和描述需对应 key。
