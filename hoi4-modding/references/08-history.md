> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

国家历史文件（`history/countries/XXX - CountryName.txt`）定义每个国家在 1936 年（及 1939 年剧本）的所有初始状态。每个国家有一个独立文件，按 Tag 一对一命名。

---

## 完整语法参考

### 1. 顶级设定（所有国家共用）

```
capital = 123                            # 首都省份 ID（必填）

# --- OOB ---
set_oob = "MYT_1936"                     # 陆军战斗序列

# --- 学说 ---
add_breakthrough_progress = {            # 突破进度
    specialization = specialization_land # specialization_land / _naval / _air
    value = 0.5                          # 0~1
}
set_grand_doctrine = new_mobile_warfare  # 主学说路径

# --- 科技 ---
set_technology = {
    infantry_weapons = 1
    tech_support = 1
    tech_engineers = 1
    tech_trucks = 1
    motorised_infantry = 1
    gw_artillery = 1
}

# --- 基础设定 ---
starting_train_buffer = 2                # 火车库存
set_fuel_ratio = 0.8                     # 燃油储量（0~1）
set_convoys = 200                        # 运输船
set_research_slots = 3                   # 科研槽数

# --- 国家状态（0~1，非百分比！） ---
set_stability = 0.55
set_war_support = 0.30

# --- 政治 ---
set_politics = {
    ruling_party = neutrality            # democratic / fascism / communism / neutrality
    last_election = "1933.3.5"
    election_frequency = 48              # 月
    elections_allowed = no
}
set_popularities = {                     # 百分比，总和应 = 100
    democratic = 15
    fascism = 10
    communism = 20
    neutrality = 55
}

# --- 初始 idea ---
add_ideas = { MYT_national_spirit MYT_advisor }

# --- 动态修正器 & 变量 ---
add_dynamic_modifier = { modifier = MYT_economy_modifier }
set_variable = { MYT_factory_factor = 0.1 }

# --- 外交 ---
diplomatic_relation = { country = GER relation = non_aggression_pact }

# --- 角色 ---
recruit_character = MYT_general_wang
```

### 2. DLC 条件分叉

几乎所有新 DLC 都需要在 OOB 和科技块中做条件分叉。关键 DLC 对照表：

| DLC key | 影响的系统 |
|---------|-----------|
| `No Step Back` | 坦克设计器（chassis 分离） |
| `By Blood Alone` | 飞机设计器（airframe 分离） |
| `Man the Guns` | 舰船设计器（hull 分离） |
| `La Resistance` | 情报机构、装甲车 |
| `Gotterdammerung` | 特殊项目（内圈、突破） |

分叉模式：
```
IF = {
    limit = { has_dlc = "No Step Back" }
    set_oob = "MYT_1936_nsb"
    set_technology = { gwtank_chassis = 1 basic_light_tank_chassis = 1 ... }
}
ELSE = {
    set_oob = "MYT_1936_legacy"
    set_technology = { gwtank = 1 basic_light_tank = 1 ... }
}
```

**空军 DLC 分叉示例：**
```
IF = {
    limit = { has_dlc = "By Blood Alone" }
    set_air_oob = "MYT_1936_air_bba"
    set_technology = { engines_1 = 1 iw_small_airframe = 1 basic_small_airframe = 1 }
}
ELSE = {
    set_air_oob = "MYT_1936_air_legacy"
    set_technology = { early_fighter = 1 fighter1 = 1 }
}
```

**海军 DLC 分叉示例：**
```
IF = {
    limit = { has_dlc = "Man the Guns" }
    set_naval_oob = "MYT_1936_naval_mtg"
    set_technology = { early_ship_hull_light = 1 basic_ship_hull_light = 1 basic_battery = 1 sonar = 1 }
}
ELSE = {
    set_naval_oob = "MYT_1936_naval_legacy"
    set_technology = { early_destroyer = 1 basic_destroyer = 1 }
}
```

### 3. 外交关系参照

```pdxscript
# 保证独立
diplicatic_relation = { country = BEL relation = guarantee }

# 互不侵犯
diplomatic_relation = { country = SOV relation = non_aggression_pact }

# 阵营成员
diplomatic_relation = { country = HUN relation = faction_member }

# 单向军事通行权
give_military_access = FRA

# 傀儡/附庸（自治等级设置）
set_autonomy = {
    target = SLO
    autonomous_state = autonomy_puppet
}
# 自治等级: autonomy_dominion / autonomy_colony / autonomy_puppet /
#            autonomy_integrated_puppet / autonomy_supervised_state / autonomy_annex / autonomy_free

# 态度修正（引用 common/opinion_modifiers/）
add_opinion_modifier = { target = GER modifier = MR_pact }

# 贸易
create_import = { resource = oil factories = 2 exporter = SOV }
```

### 4. 标记与条件设定

```pdxscript
set_country_flag = MYT_reform_done                   # 永久标记
set_country_flag = { flag = MYT_temp days = 365 }    # 365 天后自动清除
set_global_flag = MYT_conference_completed           # 全局标记
clr_country_flag = MYT_reform_done                   # 手动清除
```

### 5. Tag 命名规范

- **3 个大写字母**（如 `GER`, `MYT`），VANILLA TAG 不可复用
- 文件名格式：`XXX - EnglishCountryName.txt`
- Tag 必须在 `common/country_tags/` 注册
- 已知 Tag 查 [HOI4 Wiki](https://hoi4.paradoxwikis.com/Countries)

### 6. 1939 剧本块

```
1939.1.1 = {
    set_war_support = 0.70
    add_political_power = 800
    add_command_power = 50
    set_research_slots = 4

    add_ideas = { war_economy extensive_conscription }

    complete_national_focus = MYT_industrial_expansion  # 已完成（可跳过前置条件）
    unlock_national_focus = MYT_army_reform              # 仅解锁（不跳前置）

    set_autonomy = { target = SLO autonomous_state = autonomy_puppet }
    set_country_flag = MYT_1939_scenario_initialized

    diplomatic_relation = { country = ENG relation = faction_member }
}
```

---

## 实战示例

### 示例 1：新建小国 —— "明阳"（MYG）

```
# history/countries/MYG - Mingyang.txt

capital = 1200
OOB = "MYG_1936"

set_technology = {
    infantry_weapons = 1
    tech_support = 1
    tech_engineers = 1
    tech_trucks = 1
    gw_artillery = 1
}

set_convoys = 10
set_fuel_ratio = 0.3
set_research_slots = 2
set_stability = 0.60
set_war_support = 0.20

set_politics = {
    ruling_party = neutrality
    last_election = "1934.6.15"
    election_frequency = 48
    elections_allowed = no
}
set_popularities = { democratic = 20 fascism = 10 communism = 15 neutrality = 55 }

add_ideas = { MYG_traditional_agriculture MYG_limited_industry }

recruit_character = MYG_general_liu
recruit_character = MYG_advisor_chen

1939.1.1 = {
    set_war_support = 0.40
    add_political_power = 300
    set_research_slots = 3
    add_ideas = { limited_exports }
    complete_national_focus = MYG_industrial_expansion
}
```

### 示例 2：复杂外交国家 —— "北风共和国"（NFR）

```
# history/countries/NFR - Northwind Republic.txt

capital = 850
IF = {
    limit = { has_dlc = "No Step Back" }
    set_oob = "NFR_1936_nsb"
}
ELSE = { set_oob = "NFR_1936" }

set_grand_doctrine = new_grand_battleplan

set_technology = {
    infantry_weapons = 1 infantry_weapons1 = 1
    tech_recon = 1 tech_support = 1 tech_engineers = 1
    tech_trucks = 1 motorised_infantry = 1
    gw_artillery = 1 interwar_antiair = 1 basic_train = 1
    synth_oil_experiments = 1 fuel_silos = 1 fuel_refining = 1
}

IF = {
    limit = { has_dlc = "By Blood Alone" }
    set_air_oob = "NFR_1936_air_bba"
    set_technology = { engines_1 = 1 iw_small_airframe = 1 basic_small_airframe = 1 early_fighter = 1 }
}
ELSE = {
    set_air_oob = "NFR_1936_air_legacy"
    set_technology = { early_fighter = 1 fighter1 = 1 }
}

IF = {
    limit = { has_dlc = "Man the Guns" }
    set_naval_oob = "NFR_1936_naval_mtg"
    set_technology = {
        early_ship_hull_light = 1 basic_ship_hull_light = 1
        early_ship_hull_cruiser = 1 early_ship_hull_submarine = 1
        basic_battery = 1 basic_light_battery = 1 basic_depth_charges = 1
    }
}
ELSE = {
    set_naval_oob = "NFR_1936_naval_legacy"
    set_technology = { early_destroyer = 1 basic_destroyer = 1 }
}

set_convoys = 80
set_fuel_ratio = 0.7
set_research_slots = 3
set_stability = 0.50
set_war_support = 0.35

set_politics = {
    ruling_party = democratic
    last_election = "1936.4.12"
    election_frequency = 48
    elections_allowed = yes
}
set_popularities = { democratic = 55 fascism = 15 communism = 20 neutrality = 10 }

add_ideas = { NFR_democratic_tradition NFR_naval_heritage }
add_dynamic_modifier = { modifier = NFR_economy_modifier }
set_variable = { NFR_economy_consumer_goods_factor = 0.05 }

# 外交关系
diplomatic_relation = { country = GER relation = guarantee }
diplomatic_relation = { country = FRA relation = non_aggression_pact }
give_military_access = FRA
set_autonomy = { target = NFP autonomous_state = autonomy_puppet }
diplomatic_relation = { country = NFP relation = faction_member }

recruit_character = NFR_president_dubois
recruit_character = NFR_general_moreau
recruit_character = NFR_admiral_leroux
recruit_character = NFR_advisor_dupont

# 1939 剧本
1939.1.1 = {
    set_war_support = 0.65
    add_political_power = 800
    set_research_slots = 4
    add_ideas = { limited_exports NFR_wartime_industry }

    complete_national_focus = NFR_industrial_expansion
    complete_national_focus = NFR_naval_rearmament
    unlock_national_focus = NFR_join_allies

    set_country_flag = NFR_1939_war_preparation
    diplomatic_relation = { country = ENG relation = faction_member }
}
```

---

## 常见陷阱

1. **Tag 冲突静默灾难**。HOI4 不报错——两个同名 Tag 互相覆盖数据，行为随机。必须在 `common/country_tags/` 中验证唯一性。

2. **`IF` / `ELSE` 关键字大小写**。必须是 `IF = { limit = { ... } }`（大写），效果/触发器名是蛇形小写。混淆导致文件解析失败但无报错。

3. **`set_technology` 中拼错科技 key 静默失败**。不存在的科技名不会报错，只是不会生效。始终用 CWTools 或 HOI4 Validator 预检。

4. **`set_stability` / `set_war_support` 值域是 0~1**。`set_stability = 55` 会造成 5500% 稳定度溢出。始终用 0.55 格式。

5. **OOB 文件路径**。`set_oob = "MYT_1936"` 引用 `history/units/MYT_1936.txt`。文件不存在则陆军完全空营；同理 `set_air_oob` 和 `set_naval_oob`。

---

## FAQ

**Q: 添加新国家后闪退，排查顺序？**
A: (1) Tag 与 `common/country_tags/` 匹配；(2) OOB 文件存在；(3) 引用的 idea/technology key 正确；(4) 语法和编码（UTF-8 without BOM）。开 `-debug` 查 error.log。

**Q: 开局陆军师怎么配？**
A: 在 `history/units/MYT_1936.txt` 定义 OOB（`division_template` + 部署），然后 `set_oob = "MYT_1936"` 引用。详见 `references/10-equipment.md`。

**Q: `diplomatic_relation` 的 military_access 和 `give_military_access` 区别？**
A: `diplomatic_relation` 是双向声明，`give_military_access` 是单向授予。A 单方给 B 通行权用后者。

**Q: 如何让开局就处于战争状态？**
A: 通过 `1939.1.1` 剧本块 + `create_wargoal` + 事件触发。不能在顶级直接声明战争。

**Q: `complete_national_focus` 和 `unlock_national_focus` 区别？**
A: `complete_national_focus` 表示该焦点已完成（包括其 `completion_reward` 效果）；`unlock_national_focus` 仅移除锁（不执行奖励，且仍受前置条件限制）。

---

## 检查清单

- [ ] Tag 3 大写字母且唯一，已在 `common/country_tags/` 注册
- [ ] 文件名格式：`XXX - CountryEnglishName.txt`
- [ ] `capital` 指向存在的省份 ID
- [ ] OOB 文件存在（`history/units/XXX_1936.txt`）
- [ ] 空军/海军 OOB 有 DLC 分叉（`By Blood Alone` / `Man the Guns`）
- [ ] 所有引用的科技/idea/character key 有效
- [ ] `set_stability` / `set_war_support` 值 0~1
- [ ] `set_politics.ruling_party` 是四种基本意识形态之一
- [ ] `set_popularities` 四项之和 = 100
- [ ] `1939.1.1` 剧本块已添加（如需要）
- [ ] 本地化 key 已添加

---

> **相关文件**: `references/02-mod-structure.md`、`references/04-focus-trees.md`
