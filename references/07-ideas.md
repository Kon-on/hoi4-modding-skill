> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

国家精神系统是 HOI4 中最核心的 buff/debuff 机制，涵盖国民精神、政治顾问、军事高层指挥、理论家和设计商。所有 idea 定义在 `common/ideas/` 下的 `.txt` 文件中，通过 `add_ideas` / `remove_ideas` / `swap_ideas` 在运行时动态管理。整个系统基于 modifier 数组影响国家属性，支持超过 200 种修饰符。

---

## 完整语法参考

### 1. 通用 Idea 块结构

```
idea = {
    name = my_national_spirit           # 唯一 ID（必须）

    picture = generic_industrial_bonus  # GFX 图标名
    category = industrial               # 可选分类

    cost = 150                          # 激活花费（PP），0 或不写 = 免费
    removal_cost = 75                   # 移除花费（PP），默认 0

    # --- 四种触发器 ---
    allowed = { tag = GER }             # 能否启用（硬条件）
    visible = { ... }                   # 何时在 UI 中可见（纯显示控制）
    available = { has_war = yes }       # 持续检查，不满足则自动移除
    cancel = { ... }                    # 手动"取消"按钮的可用条件
    cancel_if_not_available = yes       # available 不满足时自动移除
    allowed_civil_war = { always = yes }
    allowed_to_remove = { always = yes }

    slot = political_advisor            # 槽位类型（不写 = 普通国民精神）

    modifier = { ... }                  # 修饰符数组（见下文）

    ai_will_do = {
        factor = 1                      # 1 = 默认权重
        modifier = { factor = 0 has_war = no }
    }
}
```

### 2. Modifier 速查表

**数值规则：小数 = 百分比**（0.10 = +10%），**整数 = 绝对量**（500 = +500）。

| 类别 | 常用 modifier | 说明 |
|------|-------------|------|
| 政治 | `stability_factor` | 稳定度 0.10 = +10% |
| 政治 | `war_support_factor` | 战争支持度 0.10 = +10% |
| 政治 | `political_power_factor` | 政治点数获取系数 |
| 政治 | `political_power_gain` | 每日 PP 绝对值（如 0.20） |
| 意识形态 | `fascism_drift` / `communism_drift` / `democratic_drift` / `neutrality_drift` | 每月意识形态变化 |
| 意识形态 | `drift_defence_factor` | 意识形态防御系数 |
| 科研 | `research_speed_factor` | 全研究速度 |
| 科研 | `research_speed_industrial_engineering_factor` | 工业工程研究速度 |
| 工业 | `production_factory_efficiency_factor` | 工厂产出 |
| 工业 | `production_factory_start_efficiency_factor` | 新产线起始效率 |
| 工业 | `production_factory_max_efficiency_factor` | 最大工厂效率 |
| 工业 | `production_factory_efficiency_gain_factor` | 效率保持 |
| 工业 | `production_speed_arms_factory_factor` | 军工建造速度 |
| 工业 | `production_speed_industrial_complex_factor` | 民工建造速度 |
| 工业 | `production_speed_dockyard_factor` | 船坞建造速度 |
| 工业 | `production_speed_buildings_factor` | 建筑速度 |
| 工业 | `industrial_capacity_factory` | 军工绝对产出（整数） |
| 工业 | `industrial_capacity_dockyard` | 船坞绝对产出（整数） |
| 工业 | `line_change_production_efficiency_factor` | 产线切换保持 |
| 工业 | `equipment_conversion_speed` | 装备改装速度 |
| 工业 | `lack_of_resource_penalty_factor` | 资源缺乏惩罚（负数 = 减少惩罚） |
| 工业 | `conversion_cost_civ_to_mil_factor` | 民工转军工花费系数 |
| 经济 | `consumer_goods_factor` | 生活消费品需求（负数 = 减少） |
| 经济 | `economy_cost_factor` | 经济法案费用 |
| 经济 | `trade_laws_cost_factor` | 贸易法案费用 |
| 资源 | `local_resources_factor` | 本地资源产出 |
| 资源 | `production_oil_factor` | 炼油产出 |
| 人力 | `conscription` | 适役人口绝对百分比（如 0.02 = +2%） |
| 人力 | `recruitable_population` | 可招募人口绝对比例 |
| 人力 | `recruitable_population_factor` | 可招募人口系数 |
| 人力 | `weekly_manpower` | 每周人力（整数） |
| 人力 | `minimum_training_level` | 训练时间（负数 = 缩短） |
| 人力 | `mobilization_speed` | 动员速度 |
| 陆军 | `army_org_factor` | 组织度 |
| 陆军 | `army_morale_factor` | 士气 |
| 陆军 | `army_attack_factor` / `army_defence_factor` | 全军攻击/防御 |
| 陆军 | `army_core_attack_factor` / `army_core_defence_factor` | 核心领土攻击/防御 |
| 陆军 | `army_speed_factor` | 移动速度 |
| 陆军 | `infantry_attack_factor` / `armor_attack_factor` / `artillery_attack_factor` / `motorized_attack_factor` | 具体兵种攻击 |
| 陆军 | `armor_speed_factor` | 装甲速度 |
| 陆军 | `supply_consumption_factor` | 补给消耗（负数 = 节省） |
| 陆军 | `army_fuel_consumption_factor` | 陆军油耗（负数 = 节省） |
| 陆军 | `planning_speed` | 计划速度 |
| 陆军 | `max_planning_factor` | 最大计划加成 |
| 陆军 | `org_loss_when_moving` | 移动组织度损失（负数 = 减少损失） |
| 陆军 | `max_dig_in_factor` / `dig_in_speed_factor` | 堑壕值 |
| 陆军 | `initiative_factor` / `coordination_bonus` | 主动权/协同性 |
| 陆军 | `equipment_capture` | 装备缴获率 |
| 陆军 | `air_superiority_bonus_in_combat` | 空优战斗加成 |
| 陆军 | `land_night_attack` | 夜战攻击 |
| 陆军 | `army_experience_gain_factor` | 陆军经验获取 |
| 陆军 | `land_doctrine_cost_factor` | 陆军学说花费（负数 = 减费） |
| 海军 | `navy_org_factor` / `navy_attack_factor` / `navy_defense_factor` | 海军组织度/攻击/防御 |
| 海军 | `naval_hit_chance` | 命中率 |
| 海军 | `screening_efficiency` | 屏护效率 |
| 海军 | `convoy_raiding_efficiency_factor` | 破交效率 |
| 海军 | `navy_max_range_factor` | 最大航程 |
| 海军 | `navy_fuel_consumption_factor` | 海军油耗（负数 = 节省） |
| 空军 | `air_attack_factor` / `air_defence_factor` | 空军攻击/防御 |
| 空军 | `air_agility_factor` | 机动性 |
| 空军 | `air_range_factor` | 航程 |
| 空军 | `air_accidents_factor` | 事故率（负数 = 减少） |
| 空军 | `air_cas_efficiency` | 近地支援效率 |
| 空军 | `air_nav_efficiency` | 海军轰炸效率 |
| 空军 | `air_ace_generation_chance_factor` | 王牌生成率 |
| 空军 | `air_strategic_bomber_bombing_factor` | 战略轰炸 |
| 情报 | `encryption_factor` / `decryption_factor` | 加密/破译 |
| 情报 | `intel_network_gain_factor` | 情报网获取 |
| 情报 | `civilian_intel_to_others` / `army_intel_to_others` / `navy_intel_to_others` / `airforce_intel_to_others` | 情报泄露（负数 = 减少泄露） |
| 占领 | `resistance_growth` | 抵抗增长（负数 = 压制） |
| 占领 | `compliance_growth` | 顺从度增长 |
| 傀儡 | `subjects_autonomy_gain` | 傀儡自治度增长（负数 = 延缓独立） |

### 3. 槽位类型一览

| 槽位值 | 中文 | 特殊字段 |
|--------|------|---------|
| (无) | 国民精神 | 永久生效，不占槽位；`cost > 0` 需手动购买 |
| `political_advisor` | 政治顾问 | — |
| `army_chief` | 陆军总司令 | — |
| `navy_chief` | 海军总司令 | — |
| `air_chief` | 空军总司令 | — |
| `theorist` | 理论家 | — |
| `tank_manufacturer` | 坦克设计商 | `equipment_bonus = { tank = { ... } }` |
| `naval_manufacturer` | 舰船设计商 | `equipment_bonus = { ship = { ... } }` |
| `aircraft_manufacturer` | 飞机设计商 | `equipment_bonus = { fighter = { ... } cas = { ... } }` |
| `materiel_manufacturer` | 装备设计商 | `equipment_bonus = { infantry = { ... } }` |

**设计商模板（含 equipment_bonus）：**

```
idea = {
    name = MYT_tank_designer
    picture = generic_armor_manufacturer_2
    slot = tank_manufacturer
    cost = 150
    removal_cost = 75
    allowed = { tag = MYT }
    visible = { has_technology = basic_light_tank }
    modifier = {
        armor_attack_factor = 0.05
        armor_speed_factor = 0.05
    }
    equipment_bonus = {
        tank = {                    # 匹配 archetype = tank 的装备
            armor_value = 0.10      # 装甲 +10%
            max_speed = 0.05
            reliability = 0.05
        }
    }
    design_team_type = tank_manufacturer
    ai_will_do = { factor = 1 }
}
```

### 4. 动态管理

```pdxscript
add_ideas = MYT_spirit                    # 单个
add_ideas = { MYT_1 MYT_2 MYT_3 }         # 批量
remove_ideas = { MYT_old }                # 移除

swap_ideas = {                            # 原子交换：remove 不存在则 add 也不执行
    remove = MYT_old
    add = MYT_new
}

if = { limit = { has_idea = MYT_spirit } ... }
```

---

## 实战示例

### 示例 1：国家精神 —— "五年计划"

```
idea = {
    name = MYT_five_year_plan
    picture = generic_industrial_bonus
    category = industrial
    allowed = { tag = MYT }
    modifier = {
        production_speed_buildings_factor = 0.15
        industrial_capacity_factory = 3
        consumer_goods_factor = 0.05
        stability_factor = -0.05
        war_support_factor = 0.05
    }
    removal_cost = 150
    allowed_to_remove = { always = yes }
    ai_will_do = { factor = 1 }
}
```

### 示例 2：政治顾问 —— "财政部长"

```
idea = {
    name = MYT_finance_minister
    picture = generic_political_advisor_asia_1
    slot = political_advisor
    cost = 150
    removal_cost = 50
    allowed = { tag = MYT has_government = neutrality }
    visible = { NOT = { has_idea = MYT_finance_minister } }
    modifier = {
        political_power_factor = 0.15
        economy_cost_factor = -0.15
        consumer_goods_factor = -0.05
    }
    ai_will_do = { factor = 1 }
}
```

### 示例 3：坦克设计商 —— "皇家兵工厂"

```
idea = {
    name = MYT_royal_arsenal
    picture = generic_armor_manufacturer_2
    slot = tank_manufacturer
    cost = 150
    removal_cost = 75
    allowed = { tag = MYT }
    visible = { has_technology = basic_light_tank }
    modifier = {
        armor_attack_factor = 0.05
        armor_defence_factor = 0.05
        production_speed_arms_factory_factor = 0.05
    }
    equipment_bonus = {
        tank = { armor_value = 0.10 max_speed = 0.05 reliability = 0.05 }
    }
    design_team_type = tank_manufacturer
    ai_will_do = { factor = 1 }
}
```

---

## 常见陷阱

1. **未指定 `slot` 的顾问类 idea 会变成永久国民精神**。如果没有 `slot = political_advisor`，它不会出现在顾问栏，而是无代价永久生效。

2. **`visible` 仅控制 UI 显示，`available` 控制自动移除**。只设 `available` 不设 `visible` 会使 idea 在条件不满足时静默消失，玩家困惑。

3. **`equipment_bonus` 只在设计商槽位中生效**。写在普通国民精神里不报错但完全无效。装备匹配依赖 `archetype` 字段。

4. **`swap_ideas` 的 remove 不存在则 add 也不会执行**。如果旧 idea 已被其他途径移除，新 idea 也丢失——这不是事务回滚。

5. **`cost` 单位是政治点数**。cost = 0 或不写 = 免费。注意：政治点数是一次性扣除，不是持续消耗。

---

## FAQ

**Q: 如何让国家开局就拥有 idea？**
A: 在 `history/countries/XXX - CountryName.txt` 中写 `add_ideas = { ... }`。参考 Chile: `add_ideas = { CHL_the_mapuche_conflict ... }`。

**Q: 国策完成时如何替换国家精神？**
A: `completion_reward = { swap_ideas = { remove = old_id add = new_id } }`。

**Q: 如何让设计商在特定科技完成后才出现？**
A: `visible = { has_technology = basic_medium_tank }`。

**Q: 如何创建需要政治点数购买的精神？**
A: 设置 `cost = 150` 不写 slot。精神不会自动生效，需玩家手动购买。

**Q: 一个 idea 可以有多个 slot 吗？**
A: 不可以。每个 idea 只能属于一种 slot 或完全不写。

---

## 检查清单

- [ ] idea `name` 使用唯一前缀（如 `MYT_`）
- [ ] `slot` 类型正确
- [ ] 带 slot 的 idea 填写了 `cost` 和 `removal_cost`
- [ ] 设计商 `equipment_bonus` 的 archetype 匹配装备定义
- [ ] `allowed` 触发器限制使用国家
- [ ] `visible` / `available` 语义区分清楚
- [ ] modifier 数值单位正确（小数 = 百分比，整数 = 绝对量）
- [ ] `ai_will_do` > 0
- [ ] 本地化 key 已添加

---

> **相关文件**: `references/04-focus-trees.md`、`references/08-history.md`
