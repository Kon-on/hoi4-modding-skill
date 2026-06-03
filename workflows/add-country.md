# 工作流：添加新国家

> **适用版本**: HOI4 1.18.2 | **最后更新**: 2026-06-03
>
> 参考: `references/08-history.md` `references/04-focus-trees.md` `references/12-localisation.md`
>
> 模板: `templates/history.txt` `templates/focus-tree.txt` `templates/localisation.yml`

## 概述

向 HOI4 添加一个全新的可玩国家，包含 Tag 分配、历史起始设定、国策树、旗帜和本地化文本。遵循测试版先行原则——先用临时名快速迭代，全部通过测试后再正式化。

**前置条件：** Mod 项目已创建（`workflows/new-mod.md`）、VS Code + CWTools 已安装、`-debug` 启动参数已设置。

**命名规范：** 国家 Tag 为 3 个大写字母（全 mod 唯一）；所有自定义 ID 使用统一前缀（如 `my_`）；文件名/文件夹用小写 + 下划线；`.txt` 文件 UTF-8 without BOM，`.yml` 文件 UTF-8 with BOM。

---

## 阶段一：选择 Tag

### 目标
确定一个 3 个大写字母的 Tag，确保不与原版和其他 Mod 冲突。

### 步骤

1. **生成候选 Tag** — 根据国家名缩写，优先含 Y/Q/Z/X 等低频字母（冲突概率更低），如 "My Empire" → `MYE`

2. **检查冲突**（将 `MYE` 替换为你的候选 Tag）：
   ```bash
   HOI4="D:/steam/steamapps/common/Hearts of Iron IV"
   MODS="$HOME/Documents/Paradox Interactive/Hearts of Iron IV/mod"

   # 原版 history/countries/ 中是否有同名文件
   ls "$HOI4/history/countries/" | grep -i "MYE"

   # 原版文件中是否有此 Tag 引用
   grep -r "tag = MYE" "$HOI4/history/" --include="*.txt"
   grep -r "MYE" "$HOI4/common/national_focus/" --include="*.txt"

   # 已安装 Mod 中是否有冲突
   grep -r "tag = MYE" "$MODS/" --include="*.txt"
   grep -r "MYE" "$MODS/" --include="*.txt" | grep -v "localisation"
   ```

3. **最终确认** — 以上全部无输出即安全。冲突则换候选 Tag 重新检查。

### 输出物
- 确认可用的 3 字母 Tag（如 `MYE`）

---

## 阶段二：创建 history/countries/ 文件

### 目标
创建国家历史文件，设置起始状态（1936 年剧本为最低要求）。

### 步骤

1. **创建文件**：`<mod>/history/countries/MYE - My Empire.txt`（命名格式：`TAG - 国家名.txt`，参考 `templates/history.txt`）

2. **必设字段**：
   - `capital = <省份ID>` — 用 `tdebug` 在游戏中查找
   - `ideology = democratic` / `communism` / `fascism` / `neutrality`
   - `ruling_party = <party_id>`
   - `stability = <0~1>` — 如 `0.6`
   - `war_support = <0~1>` — 如 `0.4`
   - `set_rule = { democratic = 60 communism = 10 fascism = 10 neutrality = 20 }`

3. **强烈建议设定**：
   - `set_technology = { infantry_weapons1 support_weapons electronic_mechanical_engineering }` — 起始科技
   - `set_politics = { ... }` — 内阁、法律
   - `create_country_leader = { ... }` — 领导人
   - `diplomacy = { ... }` — 外交关系（guarantee、non_aggression_pact、puppet 等）

4. **支持双剧本**（可选）：在文件顶部用 `if = { limit = { has_game_rule = { rule = start_date option = 1936_1_1 } } ... }` 包裹 1936 年设定，或创建独立 1939 年文件。

### 关键验证
- [ ] 文件编码 UTF-8 without BOM
- [ ] capital 省份 ID 正确
- [ ] 所有 `}` 正确闭合（CWTools 无红色报错）

### 输出物
- `<mod>/history/countries/TAG - 国家名.txt`

---

## 阶段三：创建国策树（最简版本）

### 目标
创建 1 个起点 + 2-3 个后续节点的最小可用国策树。

### 步骤

1. **创建文件**：`<mod>/common/national_focus/my_focus_tree.txt`

2. **最小骨架**（参考 `templates/focus-tree.txt`）：
   ```
   focus_tree = {
       id = MYE_focus
       country = { tag = MYE }

       focus = {
           id = MYE_first_step
           icon = GFX_goal_generic_construct_civilian
           x = 0
           y = 0
           cost = 10
           completion_reward = {
               add_stability = 0.05
               add_political_power = 50
           }
       }

       focus = {
           id = MYE_industrial_push
           icon = GFX_goal_generic_construct_civilian
           prerequisite = { focus = MYE_first_step }
           x = 0
           y = 1
           cost = 10
           completion_reward = {
               add_offsite_building = { type = arms_factory level = 2 }
           }
       }

       focus = {
           id = MYE_military_expansion
           icon = GFX_goal_generic_build_armor
           prerequisite = { focus = MYE_first_step }
           x = 1
           y = 1
           cost = 10
           completion_reward = {
               add_army_experience = 20
           }
       }
   }
   ```

   **关键规则**：`prerequisite` 即使只有一项也须用大括号 `{ focus = xxx }`；互斥用 `mutually_exclusive = { focus = A focus = B }`；x/y 坐标相邻节点差 1；icon 可先用原版通用图标。

3. **加载国策树**：在 history 文件中添加 `load_focus_tree = MYE_focus`。

### 输出物
- `<mod>/common/national_focus/my_focus_tree.txt`

---

## 阶段四：创建旗帜

### 目标
创建 .tga 格式旗帜——标准、法西斯、共产主义三个变体，每种 82x52 和 41x26 两个尺寸。

### 步骤

1. **创建目录结构**：
   ```
   <mod>/gfx/flags/
   ├── MYE.tga              # 标准旗帜（82×52）
   ├── MYE_medium.tga       # 标准旗帜（41×26）
   ├── MYE_fascism.tga      # 法西斯变体（82×52）
   ├── MYE_fascism_medium.tga
   ├── MYE_communism.tga    # 共产主义变体（82×52）
   ├── MYE_communism_medium.tga
   ├── MYE_neutrality.tga   # 中立变体（可选）
   └── MYE_neutrality_medium.tga
   ```

2. **旗帜规格**：

   | 文件 | 尺寸 | 格式 |
   |------|------|------|
   | `TAG.tga` | 82×52 | 32-bit TGA（含 Alpha 通道） |
   | `TAG_medium.tga` | 41×26 | 32-bit TGA（含 Alpha 通道） |
   | `TAG_fascism.tga` / `_medium.tga` | 82×52 / 41×26 | 32-bit TGA（含 Alpha 通道） |
   | `TAG_communism.tga` / `_medium.tga` | 82×52 / 41×26 | 32-bit TGA（含 Alpha 通道） |
   | `TAG_neutrality.tga` / `_medium.tga` | 82×52 / 41×26 | 32-bit TGA（可选，含 Alpha） |

3. **制作要点**：使用 GIMP/Photoshop 导出 32-bit TGA（含 Alpha）；小旗需确保元素清晰可辨（非简单缩放）；测试阶段可先用纯色 .tga 验证加载逻辑。

### 关键验证
- [ ] 扩展名为 `.tga`（非 `.png` `.jpg`）
- [ ] 32-bit TGA 含 Alpha 通道
- [ ] 文件名与 Tag 严格一致（区分大小写）

### 输出物
- `<mod>/gfx/flags/` 下 6-8 个 .tga 文件

---

## 阶段五：添加本地化

### 目标
为所有新增内容添加英文和中文双语本地化。

### 步骤

1. **创建文件**（均为 **UTF-8 with BOM**）：
   - `<mod>/localisation/english/my_country_l_english.yml`
   - `<mod>/localisation/chinese/my_country_l_chinese.yml`

2. **英文本地化**（参考 `templates/localisation.yml`）：
   ```yml
   l_english:
    MYE:0 "My Empire"
    MYE_DEF:0 "the My Empire"
    MYE_ADJ:0 "My Imperial"
    MYE_democratic:0 "My Empire"
    MYE_fascism:0 "Imperial Authority"
    MYE_communism:0 "People's Republic of My Empire"
    MYE_neutrality:0 "Kingdom of My Empire"
    MYE_democratic_party:0 "National Congress"
    MYE_fascism_party:0 "Imperial Vanguard"
    MYE_communism_party:0 "People's Front"
    MYE_neutrality_party:0 "Royal Council"
    MYE_first_step:0 "First Steps Forward"
    MYE_first_step_desc:0 "Our nation must take its first steps into a new era."
    MYE_industrial_push:0 "Industrial Push"
    MYE_industrial_push_desc:0 "Invest heavily in military industry."
    MYE_military_expansion:0 "Military Expansion"
    MYE_military_expansion_desc:0 "Expand and modernize our armed forces."
   ```

3. **中文本地化**：
   ```yml
   l_chinese:
    MYE:0 "我的帝国"
    MYE_DEF:0 "我的帝国"
    MYE_ADJ:0 "我的帝国"
    MYE_democratic:0 "我的帝国"
    MYE_fascism:0 "帝国威权"
    MYE_communism:0 "我的帝国人民共和国"
    MYE_neutrality:0 "我的帝国王国"
    MYE_democratic_party:0 "国民议会"
    MYE_fascism_party:0 "帝国先锋党"
    MYE_communism_party:0 "人民阵线"
    MYE_neutrality_party:0 "皇家议会"
    MYE_first_step:0 "迈出第一步"
    MYE_first_step_desc:0 "我们的国家必须迈入新时代的第一步。"
    MYE_industrial_push:0 "工业推动"
    MYE_industrial_push_desc:0 "大力投资军事工业以保障未来。"
    MYE_military_expansion:0 "军事扩张"
    MYE_military_expansion_desc:0 "扩大和现代化我们的武装力量。"
   ```

   **命名规范**：`TAG:0 "名称"` / `TAG_DEF:0 "the 名称"` / `TAG_ADJ:0 "形容词"` / `TAG_<ideology>:0 "政体名称"` / `TAG_<ideology>_party:0 "政党名"` / `<focus_id>:0 "国策名"` / `<focus_id>_desc:0 "描述"`。每个 key 后须有 `:0`。

### 关键验证
- [ ] .yml 文件编码为 UTF-8 with BOM（非 UTF-8 without BOM！）
- [ ] 所有新 ID 都有对应的本地化条目
- [ ] 英文和中文两套齐全

### 输出物
- `<mod>/localisation/english/my_country_l_english.yml`
- `<mod>/localisation/chinese/my_country_l_chinese.yml`

---

## 阶段六：测试

### 目标
加载 Mod 并验证国家存在、国策可用、旗帜正确显示。

### 步骤

1. **启动前检查** — 确认文件编码、CWTools 无报错、启动器中 Mod 已勾选

2. **启动游戏**（带 `-debug`），检查 `logs/error.log` 中是否有你 Tag 的错误：
   ```bash
   grep "MYE" "$DOCUMENTS/Paradox Interactive/Hearts of Iron IV/logs/error.log"
   ```

3. **游戏中逐项验证**：
   - 1936 剧本中地图上国家可见
   - 国策树面板（F6）正确显示，节点可点击
   - 旗帜在顶部状态栏、外交界面、师级列表均正确显示
   - 本地化文本无 "missing string" 占位符
   - （可选）1939 剧本正常加载
   - （可选）控制台 `tag MYE` 切换后功能正常

### 常见问题速查

| 问题 | 原因 | 解决 |
|------|------|------|
| 国家不显示 | capital 省份 ID 不正确 | tdebug 重新确认省份 ID |
| 显示原始 key（如 MYE_DEF） | 本地化文件编码不是 UTF-8 BOM | 用 VS Code 重新保存 |
| 红旗/黑旗 | .tga 格式不正确或无 Alpha | 重新导出 32-bit TGA |
| "missing focus tree" | 国策树文件路径或 id 不匹配 | 检查目录和 id 拼写 |
| "Could not find country" | history 文件未被加载 | 检查文件名格式和编码 |

### 输出物
- 所有测试项通过的验证记录

---

## 检查清单

### Tag
- [ ] Tag 为 3 个大写字母
- [ ] 与原版 `history/countries/` 无冲突
- [ ] 与已安装 Mod 无冲突

### 国家历史
- [ ] `history/countries/TAG - 国家名.txt` 已创建
- [ ] capital、ideology、ruling_party、stability、war_support 已设定
- [ ] 至少含 infantry_weapons1 起始科技
- [ ] 文件编码 UTF-8 without BOM

### 国策树
- [ ] `common/national_focus/my_focus_tree.txt` 已创建
- [ ] 含 1 起点 + 2-3 后续节点
- [ ] 所有括号正确闭合，prerequisite 引用正确
- [ ] focus_tree id 与 history 中 load_focus_tree 一致

### 旗帜
- [ ] TAG.tga（82x52）和 TAG_medium.tga（41x26）
- [ ] TAG_fascism + TAG_communism 变体各两种尺寸
- [ ] 32-bit TGA 含 Alpha 通道

### 本地化
- [ ] l_english 和 l_chinese 文件已创建（UTF-8 with BOM）
- [ ] 国家名、政党名、形容词、DEF 形式全部覆盖
- [ ] 所有国策 ID 有对应名称 + 描述

### 测试
- [ ] 游戏启动无 error.log 相关报错
- [ ] 1936 剧本中国家可见
- [ ] 国策树面板正确显示
- [ ] 旗帜在 UI 中正确显示
- [ ] 本地化文本全部正常

---

> **下一步**：全部通过后，进入正式化流程（`workflows/formalize-mod.md`），清理测试占位符、补全元数据、准备发布。
