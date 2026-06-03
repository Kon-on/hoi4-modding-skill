> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

装备系统定义游戏中所有可生产的军事单位——从步兵武器到坦克、飞机、舰船。核心由三部分组成：**archetype**（装备原型/家族）、**equipment**（具体装备型号）、**upgrade**（升级路径）。装备文件也包含营/连定义（`subunit`），这些 definition 决定了陆军师的编成（combat_width, manpower, equipment_needs）。文件位于 `<mod>/common/units/equipment/` 下。所有装备通过科技树解锁（参见 09-technologies.md）。

## 完整语法参考

### 装备原型（archetype）

Archetype 定义了装备的"家族"——同一类型的装备共享相同的 stat 结构和升级路径：

```
archetype = {
    name = infantry_equipment            # 原型 ID（在 equipment 块中引用）
    type = infantry                      # 装备大类：infantry/support/armor/air/naval
    priority = 1                         # load order 优先级

    # ── 最大速度 ──
    max_speed = 4.0                      # 装备提供的最大速度 (km/h)

    # ── 战斗属性 ──
    defense = 20.0                       # 防御值（防守时）
    breakthrough = 2.0                   # 突破值（进攻时）
    soft_attack = 3.0                    # 对软目标攻击
    hard_attack = 0.5                    # 对硬目标攻击
    hardness = 0.0                       # 硬度（0=完全软目标, 1=完全硬目标=100%）
    armor_value = 0.0                    # 护甲值
    ap_attack = 0.0                      # 穿甲能力（piercing）
    piercing = 0.0                       # 穿甲值（同 ap_attack, 1.5 后字段名）

    # ── 空军属性（仅 air 类型）──
    air_defense = 0.0                    # 空中防御
    air_attack = 0.0                     # 空中攻击
    air_agility = 0.0                    # 空战机动性
    air_range = 0.0                      # 作战半径
    air_bombing = 0.0                    # 对地轰炸

    # ── 海军属性（仅 naval 类型）──
    surface_detection = 0.0
    sub_detection = 0.0
    surface_visibility = 0.0
    sub_visibility = 0.0
    light_attack = 0.0
    heavy_attack = 0.0
    torpedo_attack = 0.0
    depth_charge_attack = 0.0

    # ── 可靠性 ──
    reliability = 0.8                    # 基础可靠性（0.0~1.0, 1.0=100%）
    reliability_factor = 0.1             # 可靠性因子（影响损耗计算）

    # ── 生产属性 ──
    production_cost = 0.5                # 基准生产成本（IC 值）

    # ── 资源需求 ──
    build_cost_resources = {             # 生产所需资源（每工厂）
        steel = 1                        # 1 单位钢铁
    }
    resources = {                        # 使用所需资源（运营）
        # 通常留空，由 upgrade 路径定义
    }

    # ── 升级路径 ──
    upgrade = {                          # 装备可升级的属性路径
        reliability = {
            max_level = 5                # 最大升级等级
            cost = 1                     # 每级经验成本
        }
        gun = {
            max_level = 5
            cost = 1
        }
        engine = {
            max_level = 5
            cost = 1
        }
    }

    # ── 视觉 ──
    picture = GFX_infantry_equipment     # 图标引用
    sprite = generic_infantry            # 地图模型引用
    active = yes                         # 是否启用

    # ── 条件激活 ──
    is_buildable = no                     # no=由科技解锁后才可生产
}
```

### 装备定义（equipment）

`equipment` 块定义具体装备型号——通常是某 archetype 的一个具体实例：

```
equipment = {
    type = infantry_equipment            # 引用 archetype 的 name 字段
    # ── 版本标识 ──
    active = yes
    version_name = "Infantry Eq. I"      # 可选：显示名称
    # ── 属性覆盖 ──
    defense = 22.0
    breakthrough = 2.5
    soft_attack = 6.0
    hard_attack = 1.0
    hardness = 0.0
    armor_value = 0.0
    piercing = 1.0
    reliability = 0.9
    max_speed = 4.0
    # ── 生产 ──
    production_cost = 0.43               # 生产 IC 成本
    # ── 资源 ──
    resources = {
        steel = 2
    }
    # ── 视觉 ──
    picture = GFX_infantry_equipment_1
    # ── 条件 ──
    is_buildable = no
}
```

### 升级路径（upgrade）

`upgrade` 在 archetype 中定义，玩家可使用陆军/海军/空军经验来提升装备属性：

```
upgrade = {
    # 预定义升级槽位
    gun = {
        max_level = 5                   # 最多 5 级
        cost = 1
    }
    # 自定义升级槽位
    armor = {
        max_level = 5
        cost = 2                        # 每级消耗 2 经验
        equipment_upgrade = {
            armor_value = 5.0           # 每级 +5 护甲
            breakthrough = 1.0          # 每级 +1 突破
            hardness = 0.05             # 每级 +5% 硬度
        }
    }
    engine = {
        max_level = 5
        cost = 1
        equipment_upgrade = {
            max_speed = 0.5             # 每级 +0.5 km/h
            reliability = 0.02           # 每级 +2% 可靠性
        }
    }
    reliability_upgrade = {
        max_level = 3
        cost = 2
        equipment_upgrade = {
            reliability = 0.05
        }
    }
}
```

### 营/连定义（subunit）

陆军师的编成通过 `subunit` 定义（也存在于 `common/units/` 下）：

```
sub_unit = {
    # 快速参考：「连」「营」都使用 sub_unit 定义
    id = infantry_battalion

    # ── 战斗属性 ──
    combat_width = 2                    # 战斗宽度
    max_strength = 100                  # 最大兵力
    default_organisation = 60           # 基础组织度
    default_morale = 0.03               # 基础士气/恢复
    max_speed = 4.0                     # 营移动速度 (km/h)

    # ── 人力需求 ──
    manpower = 1000                     # 人

    # ── 装备需求 ──
    equipment = {
        infantry_equipment = 100        # 所需装备类型和数量
    }

    # ── 类型分类 ──
    category = infantry                 # 营类别（影响科技和国策的加成定位）
    group = infantry                    # 分组类别
    type = { infantry }                 # 类型标签

    # ── 地形修正 ──
    terrain_modifier = {
        forest = { attack = 0.10 }      # +10% 森林攻击
        urban = { attack = -0.10 }      # -10% 城市攻击
    }

    # ── 支援连特殊属性 ──
    need = { }                          # 支援连无此属性；前线营填写装备需求
    support = no                        # yes = 支援连（不占宽度, 用支援装备槽）
}
```

### 装备类型速查

| 类型 | archetype `type` | 文件示例 | 独特属性 |
|------|-----------------|---------|---------|
| 步兵装备 | `infantry` | `infantry_equipment.txt` | `defense`, `soft_attack`, `breakthrough` |
| 支援装备 | `support` | `support_equipment.txt` | 无战斗属性，用于支援连 |
| 轻/中/重/超重坦克 | `armor` | `light_tank_equipment.txt` | `hardness`, `armor_value`, `piercing`, `max_speed` |
| 战斗机/轰炸机/运输机 | `air` | `fighter_equipment.txt` | `air_attack`, `air_defense`, `air_agility`, `air_range` |
| 驱逐舰/巡洋舰/战列舰 | `naval` | `destroyer_equipment.txt` | 参见 naval stats 的完整字段列表 |
| 机械化/摩托化 | `mechanized` | `motorized_equipment.txt` | `hardness`, `max_speed` |
| 火炮/反坦克/防空 | `artillery` | `artillery_equipment.txt` | `soft_attack`, `hard_attack`, `piercing` |

## 实战示例

### 示例 1：新增步兵装备型号

文件：`mod/common/units/equipment/my_infantry.txt`

```
equipment = {
    type = infantry_equipment           # 沿用 vanilla archetype
    active = yes

    # ── 升级版属性（比原版步兵1略强）──
    defense = 30.0
    breakthrough = 5.0
    soft_attack = 10.0
    hard_attack = 2.0
    piercing = 5.0
    armor_value = 0.0
    hardness = 0.0
    reliability = 0.9
    max_speed = 4.0

    production_cost = 0.65              # 比原版贵
    resources = {
        steel = 3
    }

    picture = GFX_my_infantry_equipment_improved
    sprite = generic_infantry

    is_buildable = no                   # 由科技解锁
}
```

### 示例 2：新建坦克装备原型（含升级路径）

文件：`mod/common/units/equipment/my_tank.txt`

```
archetype = {
    name = my_medium_tank
    type = armor
    priority = 100

    max_speed = 8.0
    defense = 5.0
    breakthrough = 25.0
    soft_attack = 20.0
    hard_attack = 15.0
    hardness = 0.8
    armor_value = 60.0
    piercing = 75.0
    reliability = 0.8

    production_cost = 12.0
    build_cost_resources = {
        steel = 4
        tungsten = 1
    }

    upgrade = {
        gun = { max_level = 5 cost = 1
            equipment_upgrade = { soft_attack = 1.5 hard_attack = 1.5 piercing = 5.0 } }
        armor = { max_level = 5 cost = 1
            equipment_upgrade = { armor_value = 5.0 breakthrough = 1.0 } }
        engine = { max_level = 5 cost = 1
            equipment_upgrade = { max_speed = 0.5 reliability = 0.02 } }
    }

    picture = GFX_my_medium_tank
    active = yes
    is_buildable = no
}

# ── 配套营定义 ──
sub_unit = {
    id = my_medium_tank_battalion
    combat_width = 2
    max_strength = 100
    default_organisation = 10
    default_morale = 0.03
    max_speed = 8.0
    manpower = 500

    equipment = {
        my_medium_tank = 50
    }

    category = armor
    group = armor
    type = { armor }
}
```

### 示例 3：空军装备——新战斗机

文件：`mod/common/units/equipment/my_fighter.txt`

```
archetype = {
    name = my_advanced_fighter
    type = air
    priority = 100

    # ── 空军专属属性 ──
    air_attack = 30.0
    air_defense = 15.0
    air_agility = 65.0
    air_range = 1200.0
    air_bombing = 0.0                   # 战斗机无轰炸

    max_speed = 650.0
    reliability = 0.8

    production_cost = 28.0
    build_cost_resources = {
        aluminum = 3
        rubber = 1
    }

    upgrade = {
        gun = {
            max_level = 5
            cost = 1
            equipment_upgrade = { air_attack = 2.0 }
        }
        engine = {
            max_level = 5
            cost = 1
            equipment_upgrade = { max_speed = 15.0 air_agility = 1.0 }
        }
        range = {
            max_level = 5
            cost = 1
            equipment_upgrade = { air_range = 100.0 }
        }
    }

    picture = GFX_my_advanced_fighter
    active = yes
    is_buildable = no
}
```

## 常见陷阱

1. **`equipment` 的 `type` 必须精确匹配 `archetype` 的 `name`** — 这是最高频错误。即使大小写差异也会导致装备无法识别。验证方法：确认 `equipment = { type = infantry_equipment }` 中 `infantry_equipment` 与对应 archetype 文件中的 `archetype = { name = infantry_equipment }` 完全一致。

2. **忘记设置 `is_buildable = no`** — 如果不在至少一处设置 `is_buildable = no`（通常在 equipment 中），装备开局即对所有国家可用，无视科技限制。正确的管线：equipment 设 `is_buildable = no`，科技 `on_research_complete` 中通过效果解锁。

3. **`upgrade` 路径中的 `equipment_upgrade` 值累积叠加** — 每级升级的属性修改是累加的。如果 `equipment_upgrade = { armor_value = 5.0 }`，则升级第 5 级时总共获得 `5 × 5 = 25` 护甲加成。确保基础值和每次升级增量在经过 5 级后总属性在合理范围内。

4. **子单位定义遗漏重要分类标签** — `category`, `group`, `type` 三个标签决定此营能否获得科技加成和国策 buff。例如 `category = infantry` 的营会受益于所有"infantry"类别科技加成。如果一个新营的 `category` 设置错误，会导致预期的科技加成不生效。

5. **资源需求设置过高导致不可用** — 确保 `build_cost_resources` 和 `resources` 的需求与同类型 vanilla 装备的资源消耗水平一致。一个需要 5 单位铬 + 3 单位钨的早期坦克在实际游戏中可能因资源不足而无法生产。

## FAQ

**Q: archetype 和 equipment 的区别是什么？**
A: archetype 是"模板"——它定义了装备族类型的属性结构、默认值和升级路径。equipment 是"实例"——具体的装备型号引用 archetype 并可选地覆盖某些属性。一个 archetype 可以有多个 equipment 实例（如几种不同平衡的坦克变体）。

**Q: 如何让装备在科技研究后才解锁？**
A: 在 equipment 定义中设置 `is_buildable = no`，然后在对应科技的 `on_research_complete` 中使用 `add_equipment_to_stockpile` 或通过 `enable_equipment` 效果解锁。也可使用 archetype 内建的 `enable_equipment_type` 与科技 ID 的命名约定自动关联。

**Q: upgrade 路径的自定义槽位如何使用？**
A: 在 archetype 的 `upgrade = { ... }` 块中定义槽位（如 `gun`, `armor`, `engine`），每个槽位可含 `equipment_upgrade` 指定每级提升的属性。玩家在游戏中用陆军/海军/空军经验来升级这些槽位（在师设计器界面）。

**Q: `combat_width` 的标准值？** A: 步兵营=2，炮兵营=3，坦克营=2，防空/反坦克=1。宽度过大难以投入战场；过小则编成松散。

**Q: support = yes 的支援连有哪些特殊规则？**
A: 支援连不占 combat_width，不需要装备生产线中的"连"定义（直接在支援装备槽中分配），但需消耗 `support_equipment`。支援连的 `manpower` 通常少于同类前线营（约 100-500 人）。支援连不能单独构成一个师。

## 检查清单

- [ ] 所有 `equipment` 块的 `type` 精确匹配对应 `archetype` 的 `name`
- [ ] 战斗属性（defense, breakthrough, soft_attack 等）设置合理，与 vanilla 同类装备可比
- [ ] `is_buildable = no` 已设置（除非有意开局可用）
- [ ] `upgrade` 路径的 `equipment_upgrade` 值经过 5 级叠加后的上限检查
- [ ] `production_cost` 与 vanilla 同类装备可比（步兵装备约 0.4-0.5，早期坦克约 10-12）
- [ ] `build_cost_resources` 资源需求合理
- [ ] 营定义的 `combat_width` 在 1-3 范围内
- [ ] 营定义的 `category` / `group` / `type` 标签正确分类以匹配加成体系
- [ ] `picture` / `sprite` 引用存在或使用 vanilla 占位符
- [ ] 装备 ID 有唯一前缀以避冲突

> **相关文件**: 科技树系统（09-technologies.md）——装备通过科技的 `on_research_complete` 和 `equipment` 块解锁。国家精神系统（07-ideas.md）——装备设计商常通过 `add_ideas` 添加。本地化系统（12-localisation.md）——装备名称和描述需本地化 key。
