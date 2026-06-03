> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

HOI4 地图系统是最基础也最容易出错的模组领域。地图由位图文件（`provinces.bmp`、`definition.csv`、`terrain.bmp`）、州份定义（`history/states/*.txt`）、战略区域、胜利点、资源和建筑构成。修改地图涉及多个环节的精确对应——一处不一致就可能导致 CTD（Crash To Desktop）或地图显示错误。Nudge 是内建的地图编辑工具，可大幅降低操作复杂性。

## 完整语法参考

### 地图文件基础

| 文件 | 位置 | 用途 |
|------|------|------|
| `provinces.bmp` | `map/provinces.bmp` | 省份 ID 色表：每个像素=一个省份，颜色=R+G+B 索引到 definition.csv |
| `definition.csv` | `map/definition.csv` | 省份 ID→RGB 映射表：格式 `0;0;0;0;海` 或 `1;255;0;0;陆地` |
| `terrain.bmp` | `map/terrain.bmp` | 地形分布图：每个像素→所在省份的地形类型 |
| `rivers.bmp` | `map/rivers.bmp` | 河流分布图：绿色像素=河流 |
| `heightmap.bmp` | `map/heightmap.bmp` | 高度图：灰度值=地形高度 |
| `states/` | `history/states/*.txt` | 州份定义文件：每个州包含省份列表、资源、建筑、人口等 |

### 州份定义（state）

州份文件位于 `history/states/` 下，每个文件定义一个州：

```
state = {
    id = 1                              # 州 ID（必须唯一）
    name = "STATE_1"                    # 本地化 key

    # ── 资源 ──
    resources = {
        oil = 10                        # 原油（单位：/日产出基数）
        steel = 20                      # 钢铁
        aluminum = 5                    # 铝
        rubber = 0                      # 橡胶
        tungsten = 0                    # 钨
        chromium = 0                    # 铬
    }

    # ── 省份列表 ──
    provinces = {
        1 2 3 4 5                       # 该州包含的省份 ID（空格分隔）
    }

    # ── 归属与控制 ──
    owner = SOV                         # 初始拥有国 Tag
    controller = SOV                    # 初始控制国 Tag（可与 owner 不同）
    add_core_of = SOV                   # 核心领土标记
    add_core_of = GER                   # 可以有多个核心国

    # ── 人力 ──
    manpower = 100000                   # 该州总人力池

    # ── 建筑 ──
    buildings = {
        infrastructure = 3              # 基础设施等级 (0-10)
        industrial_complex = 2          # 民用工厂
        arms_factory = 1                # 军工工厂
        naval_base = 3                  # 海军基地
        air_base = 2                    # 空军基地
        bunker = 0                      # 地面堡垒
        coastal_bunker = 0              # 海岸堡垒
        anti_air_building = 0           # 防空炮
        synthetic_refinery = 0          # 合成炼油厂
        nuclear_reactor = 0             # 核反应堆
        rocket_site = 0                 # 火箭基地
        fuel_silo = 0                   # 储油罐
        radar_station = 0               # 雷达站
    }

    # ── 胜利点 ──
    victory_points = {
        1 5                             # 省份 ID=1 的胜利点值为 5
    }
    victory_points = {
        2 1                             # 省份 ID=2 的胜利点值为 1（次要城市）
    }

    # ── 分类 ──
    state_category = rural              # 州类别：wasteland / rural / town / city / megalopolis
                                        # 影响建筑槽位数量和资源产出

    # ── 历史设定 ──
    history = {
        owner = SOV                     # 历史 owner（可随时间变化）
        # add_core_of 在上层定义，不在此处
    }
}
```

### 战略区域（strategic_region）

位于 `map/strategicregions/` 下，每个文件定义一个战略区域：

```
strategic_region = {
    id = 1
    name = "STRATEGICREGION_1"

    provinces = {
        1 2 3 4 5 6 7 8 9 10
    }

    naval_terrain = ocean               # 海军地形类型（仅海上区域）

    weather = {
        # 天气定义（通常引用全局天气模板）
    }
}
```

### 地形类型

在 `common/terrains/` 中定义。标准地形类型速查：

| 地形 ID | 名称 | 移动耗费 | 攻击修正 | 战斗宽度缩减 |
|--------|------|---------|---------|------------|
| `plains` | 平原 | 1.0 | 0 | 0% |
| `forest` | 森林 | 1.5 | -10% attacker | -20% |
| `hills` | 丘陵 | 1.2 | -5% attacker | 0% |
| `mountain` | 山地 | 2.0 | -25% attacker | -50% |
| `desert` | 沙漠 | 1.2 | 0 | 0% |
| `marsh` | 沼泽 | 2.0 | -20% attacker | -25% |
| `jungle` | 丛林 | 1.5 | -25% attacker | -25% |
| `urban` | 城市 | 1.0 | -20% attacker | -33% |
| `ocean` | 海洋 | N/A | N/A | N/A |

### Nudge 工具

Nudge 是 HOI4 内建的地图编辑工具，通过 `-debug` 启动后在控制台输入 `nudge` 打开：

| 快捷键 | 功能 |
|--------|------|
| `nudge` | 打开/关闭 Nudge 面板 |
| 左键 | 选择省份 |
| 右键 | 绘制/修改 |
| `Ctrl+S` | 保存修改 |
| `Find Error` | **每次修改地图后必须点击**——自动检查并修复 bitmap 和 CSV 的一致性错误 |

**关键：** 修改 `provinces.bmp`、`definition.csv` 或 `terrain.bmp` 后，必须在 Nudge 中点击 **"Find Error"** 按钮并保存。跳过此步 = 极高概率 CTD。

### 补给节点与铁路

在州份定义中添加（1.18 的铁路供给系统）：

```
state = {
    id = 1
    # ...
    railways = {
        hub_1 = {
            level = 3                  # 铁路等级（影响补给流量和部队战略移动速度）
            location = 1                # 补给枢纽所在省份 ID
        }
    }
    supply_nodes = {
        5 10 15                        # 补给节点省份 ID 列表
    }
}
```

### 省份属性

省份属性在 `common/province_names/` 中管理。额外的省份级别效果可通过 `modifier` 定义：

```
# 省份 modifier 示例
my_province_modifier = {
    local_resources = 2.0              # 双倍本地资源
    local_manpower = 10000              # 额外人力
    local_building_slots = 2           # 额外建筑槽位
}
```

## 实战示例

### 示例 1：修改州份资源

文件：`mod/history/states/1.txt`（覆盖原版）

```
state = {
    id = 1
    name = "STATE_1"

    provinces = {
        1 2 3 4 5
    }

    manpower = 150000                    # 从 100000 提升到 150000

    resources = {
        oil = 20                         # 从 10 翻倍到 20
        steel = 30                       # 从 20 提升到 30
        aluminum = 10
    }

    owner = SOV
    add_core_of = SOV
    add_core_of = MYT                    # 添加自定义国家核心

    buildings = {
        infrastructure = 4               # 从 3 提升到 4
        industrial_complex = 3
        arms_factory = 2
    }

    victory_points = {
        1 10                             # 从 5 提升到 10
    }

    state_category = town                # 从 rural 升级为 town（更多建筑槽位）

    history = {
        owner = SOV
    }
}
```

### 示例 2：新建州份

文件：`mod/history/states/900.txt`

```
state = {
    id = 900
    name = "STATE_MY_NEW_STATE"          # 对应本地化 key

    provinces = {
        10001 10002 10003                # 在 definition.csv 中定义的新省份
    }

    manpower = 50000

    resources = {
        steel = 15
        chromium = 8
    }

    owner = MYT                          # 自定义国家 Tag
    add_core_of = MYT

    buildings = {
        infrastructure = 2
        industrial_complex = 1
    }

    victory_points = {
        10001 3                          # 省份 10001 为 3 分胜利点
    }

    state_category = rural

    history = {
        owner = MYT
    }
}
```

配套步骤（不可跳过）：
1. 在 `map/definition.csv` 中添加新省份行：`10001;128;64;0;my_province`
2. 在 `map/provinces.bmp` 中为该省份着色（RGB=128,64,0）
3. 在 Nudge 中点击 **Find Error** 并保存
4. 在 `localisation/` 中为 `STATE_MY_NEW_STATE` 添加名称
5. 在战略区域文件（`map/strategicregions/`）中将新省份添加到对应区域

### 示例 3：通过 Nudge 修改省份归属

操作流程（无需手动写代码）：

1. 启动 HOI4 带 `-debug` 参数
2. 进入主菜单后按 `~` 打开控制台
3. 输入 `nudge` 并回车，Nudge 面板出现
4. 使用 `tdebug` 查看省份 ID（鼠标悬停显示）
5. 点击目标省份 → 在面板中修改 owner/controller/core
6. 修改完成后点击 **Find Error**
7. `Ctrl+S` 保存
8. 退出游戏，Nudge 会自动将修改写入 `history/states/` 文件
9. 将生成的文件从 Documents 的 mod 目录复制到你的 mod 项目目录

## 常见陷阱

1. **maps/provinces.bmp 和 definition.csv 不一致** — 这是最高频 CTD 原因。bmp 中存在的颜色在 CSV 中没有对应行，或 CSV 中有定义的行在 bmp 中找不到对应颜色——都会导致 CTD。Nudge 的 **Find Error** 功能可自动检测并报告这类不一致。

2. **省份 ID 跨越战略区域边界** — 一个州的所有省份必须在同一个战略区域内。如果将一个州的省份拆分到两个不同的战略区域，游戏会报错。验证：检查 `map/strategicregions/` 中各区域的 `provinces` 列表。

3. **terrain.bmp 未与新 provinces.bmp 同步** — 修改或新增省份后，`terrain.bmp` 中的对应像素区域必须更新为正确的颜色（参见 `common/terrains/` 中各地形定义的颜色映射）。如果 `terrain.bmp` 保持旧有颜色或全黑，会导致地形显示错误或 CTD。

4. **add_core_of 忘记添加** — 新建州份后如果不添加核心领土标记，该州将显示为占领领土（高抵抗度、低顺从度），即使 owner 是该国。务必为每个州的合法拥有者添加 `add_core_of`。

5. **bitmap 文件格式错误** — `provinces.bmp` 和 `terrain.bmp` 必须是 **24-bit RGB BMP** 格式（不支持 32-bit RGBA），且分辨率在整个地图范围内精确匹配。使用 GIMP 或 Photoshop 编辑时，导出前务必确认格式。任何格式偏差都会导致游戏无法加载地图。

## FAQ

**Q: Nudge 生成的 state 文件放在哪里？**
A: 默认放在 `Documents/Paradox Interactive/Hearts of Iron IV/mod/<your_mod>/history/states/`。如果你的 mod 没有在启动器中正确注册，Nudge 会将修改直接写入 `Documents/.../Hearts of Iron IV/history/states/`（这很危险——会直接修改用户数据）。

**Q: 添加新省份需要修改多少个文件？**
A: 至少 5 个文件：(1) `map/definition.csv` 添加新行，(2) `map/provinces.bmp` 添加对应颜色，(3) `map/terrain.bmp` 设置地形，(4) `history/states/<id>.txt` 将新省份加入某个州，(5) `map/strategicregions/<id>.txt` 将省份加入对应战略区域。跳过任何一个都会导致 CTD。

**Q: 如何安全地修改原版州份？**
A: 不要直接修改 `D:\steam\...\history/states/` 原文件。在 mod 的 `history/states/` 下创建同名文件（如也命名为 `1.txt`），mod 文件会完全覆盖原版（非 merge）。确保 mod 的 state 文件中包含该州的所有原始省份和设置，否则会丢失原版数据。

**Q: CTD 但 error.log 没有线索怎么办？**
A: 地图相关 CTD 是少数不会在 error.log 中留下明确线索的错误类型。诊断步骤：(1) 确认最近一次修改了哪个地图文件，(2) 在 Nudge 中运行 Find Error，(3) 用 WinMerge 对比修改前后的 definition.csv 和 bmp 文件，(4) 回退到上一个工作的版本进行二分法隔离。

**Q: state_category 会影响什么？**
A: `state_category` 决定该州的建筑槽位数量（wasteland=0, rural=2, town=4, city=6, megalopolis=8）和人口增长系数。城市的州份在抵抗度、顺从度增长、资源产出方面有修正。升级州类别可显著提升省份工业价值。

## 检查清单

**一般修改检查（修改资源/建筑/归属）：**
- [ ] `state id` 与文件名匹配（如 `1.txt` → `state = { id = 1 }`）
- [ ] `provinces` 列表中的所有省份在 `definition.csv` 中存在
- [ ] `owner` 和 `add_core_of` 的 Tag 已定义（通常需配合 08-history.md）
- [ ] `state_category` 值合法（wasteland/rural/town/city/megalopolis）
- [ ] 资源值（oil, steel 等）为正整数且合理

**新建省份检查：**
- [ ] `definition.csv` 中已添加新省份行（ID;R;G;B;名称）
- [ ] `provinces.bmp` 中已为新省份着色（颜色与 CSV 一致）
- [ ] `terrain.bmp` 中已为新省份设置地形颜色
- [ ] 新省份已添加到某州的 `provinces` 列表中
- [ ] 新省份已添加到对应战略区域的 `provinces` 列表中
- [ ] 已在 Nudge 中点击 **Find Error** 并保存
- [ ] 本地化文件已添加州名和省份名

**地图文件格式检查：**
- [ ] `provinces.bmp` 和 `terrain.bmp` 为 **24-bit RGB BMP**
- [ ] `definition.csv` 编码为 UTF-8 without BOM
- [ ] 所有 map bmp 文件分辨率一致
- [ ] 游戏可正常加载至主菜单（无 CTD）

> **相关文件**: Mod 项目结构（02-mod-structure.md）——地图修改需配置 `replace_path` 路径。调试排错（14-debugging.md）——地图 CTD 的专项诊断流程。国家历史设定（08-history.md）——州份归属需与国家历史匹配。
