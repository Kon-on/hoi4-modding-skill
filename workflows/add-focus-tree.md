# 添加国策树

> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03
> **前置要求**: Mod 骨架已创建（`workflows/new-mod.md`），目标国家已存在或准备创建
> **关联参考**: `references/04-focus-trees.md` `references/12-localisation.md`
> **关联模板**: `templates/focus-tree.txt`

## 概述

本工作流指导 AI 代理为一个国家创建或扩展完整的国策树（National Focus Tree）。流程覆盖设计布局、使用模板生成代码、关联国家、添加本地化和测试五个阶段。完成后你将拥有一个加载正常、逻辑正确、文本完整的国策树。

HOI4 的国策树是游戏中最核心的系统之一——它驱动国家发展方向、触发历史事件、解锁决策和国家精神。一个设计良好的国策树应平衡历史真实性与游戏可玩性，提供有趣且有意义的战略选择。

适用场景：
- 为新国家创建立国的完整的国策树
- 为现有国家添加替代历史的国策分支
- 扩展原版国策树（通过 mod 添加额外国策文件）

---

## 阶段一：设计国策树布局

目标：在编写代码前，先用纸笔或思维导图规划国策树的结构。这一步决定了最终的玩法和平衡性。

### 步骤 1.1：确定国策树范围和主题

回答以下设计问题：

1. **国家背景**：这个国家在 1936/1939 年起点是什么状态？工业能力？军事实力？政治体制？
2. **核心主题**：国策树围绕什么主题？（工业化、军事扩张、意识形态转变、外交联盟、经济复苏等）
3. **分支数量**：2 条分支（简单，适合小国）到 9+ 条分支（复杂，适合大国）。一般建议 3-6 条。
4. **历史 vs 替代历史**：是否包含非历史路径（如德国民主化、日本共产主义革命）？

### 步骤 1.2：设计国策树骨架

规划以下结构参数：

| 参数 | 说明 | 建议范围 |
|------|------|----------|
| 总 node 数 | 所有国策节点的总数 | 20-40（小国）/ 50-100（大国） |
| 分支数 | 顶层互斥分支（工业/军事/政治/外交等） | 3-6 条 |
| 每分支深度 | 从第一个节点到最后一个节点的步数 | 5-12 步 |
| 互斥组数 | 二选一/三选一的决策组数 | 3-8 组 |
| 共享分支 | 多分支共用节点数（用于汇聚关键国策） | 2-5 个 |

### 步骤 1.3：设计成本与奖励平衡

使用以下原则分配成本：

| 国策类型 | 建议 cost | 对应天数 | 典型奖励 |
|----------|-----------|----------|----------|
| 快速决策 | 5-7 | 35-49 天 | +50 PP, +0.03 stability, 小型精神 |
| 标准国策 | 10 | 70 天 | 1 个工厂, 1 个研究槽位, 中型精神 |
| 重大国策 | 15-21 | 105-147 天 | 多工厂, 宣战目标, 核心领土, 重大精神 |
| 极其重要 | 28-35 | 196-245 天 | 重大意识形态转变, 吞并, 新国策树加载 |

奖励设计原则：
- 早期节点给予低价值但急需的奖励（如 +50 PP、+1 民用工厂）
- 中期节点给予中等回报（如研究加成精神、额外研究槽位）
- 后期节点给予高价值奖励（如领土宣称、战争目标、大规模工业化）
- 避免在早期节点堆叠过多奖励——这会破坏游戏平衡

### 步骤 1.4：绘制布局草图

在编写代码前，绘制国策树的视觉布局：

```
每个节点格式：
  position: (x, y)  相对位置
  cost: 10           完成天数（70 天）
  prerequisite: 前置节点 ID 列表
  mutually_exclusive: 互斥节点 ID

示例布局（简单工业化分支）：
  ind_start (x=0, y=0)     ──→  ind_arms (x=1, y=0)
       │                              │
       └──→  ind_civ (x=1, y=1)  ──→  ind_mega (x=2, y=1)

x 方向表示进度（从左到右），y 方向表示分支选择（从上到下）
```

使用 HOI4 Content Maker 工具可视化布局（推荐），或在纸上/思维导图工具中草绘。

注意：
- `x, y` 坐标决定节点在国策树视图中的位置。水平间距建议 1-3，垂直间距建议 1-2
- 互斥节点应共享同一 x 坐标（在同一列），但不同 y
- `relative_position_id` 决定节点在树中的排列参考。一般指向其唯一的前置节点（如果只有一个前置），或指向自身（根节点）

### 阶段一检查点

- [ ] 国家背景和核心主题已明确
- [ ] 总 node 数、分支数、深度、互斥组数已确定
- [ ] 成本与奖励平衡方案已设计
- [ ] 布局草图已完成（含所有节点的 x,y 坐标和前置关系）

---

## 阶段二：使用模板生成国策文件

目标：基于阶段一的设计，使用 `templates/focus-tree.txt` 模板生成完整、语法正确的国策树代码。

### 步骤 2.1：准备模板

确认 `templates/focus-tree.txt` 模板可用。该模板覆盖以下变体：
- **独占国策**（mutually_exclusive）：二选一或三选一
- **共享国策**（无 mutually_exclusive）：任意组合
- **动态国策**（含 available 条件块）：根据国家状态显示/隐藏
- **重复国策**（允许重复完成的国策）

如果模板缺失，参考原版文件自行构建。推荐参考：`common/national_focus/germany.txt`（大型树）和 `common/national_focus/belgium.txt`（中型树）。

### 步骤 2.2：生成 focus_tree 外壳

创建文件 `common/national_focus/<MOD_PACKAGE>_focus_tree.txt`：

```
focus_tree = {
    id = <MOD_PACKAGE>_focus_tree

    country = {
        factor = 0

        modifier = {
            add = 10
            tag = <TARGET_TAG>
        }
    }

    # 所有 focus 节点放在这里
}
```

关键字段说明：
- `id`：全局唯一标识，被 `history/countries/` 中的 `load_focus_tree` 引用
- `country`：决定哪些国家加载此树。`factor = 0` 作为基准，`modifier` 中的 `add = 10` 使特定 tag 获得此树。如果有多个国家使用同一棵树，为每个 tag 添加一个 modifier
- `initial_show_position`（可选）：`{ x = 0 y = 0 }` 设定打开国策树时的初始视口位置

### 步骤 2.3：逐个生成国策节点

对每个设计好的节点，使用模板生成代码。以下是完整节点结构：

```
focus = {
    id = <MOD_PACKAGE>_<node_name>
    icon = <GFX_icon_path>

    x = <x_coordinate>
    y = <y_coordinate>

    relative_position_id = <reference_focus_id>

    cost = <cost_value>

    available_if_capitulated = <yes|no>

    prerequisite = {
        focus = <prerequisite_focus_id>
    }

    mutually_exclusive = {
        focus = <mutex_focus_id_1>
        focus = <mutex_focus_id_2>
    }

    available = {
        # 条件块——国策何时可见/可选
        <trigger_conditions>
    }

    cancel_if_invalid = <yes|no>
    cancel = {
        # 当此条件不再满足时自动取消国策
        <trigger_conditions>
    }

    will_lead_to_war_with = <TAG>

    completion_reward = {
        # 完成效果
        <effects>
    }

    search_filters = {
        FOCUS_FILTER_POLITICAL
        FOCUS_FILTER_INDUSTRY
    }
}
```

字段使用指南：

**必填字段（每个 focus 必须包含）：**
- `id`：唯一标识，格式 `<MOD_PACKAGE>_<descriptive_name>`
- `icon`：GFX 图标路径。可用原版通用图标（如 `GFX_goal_generic_construct_civilian`、`GFX_goal_generic_allies_build_infantry`、`GFX_goal_generic_attack_allies` 等），或自定义图标
- `x, y`：在树中的位置
- `relative_position_id`：参考节点 ID，决定此节点在树中的相对排列位置
- `cost`：完成所需天数 = cost × 7
- `completion_reward`：完成时执行的效果块

**常用可选字段：**
- `prerequisite`：前置国策列表。可指定多个（ALL 必须完成）
- `mutually_exclusive`：互斥国策列表。选择此节点后，列表中的其他节点将不可用
- `available`：显示条件。不满足时节点灰显但可见；如果要完全隐藏，需使树中无路径可达
- `available_if_capitulated = yes`：允许在投降后仍可使用（默认 no）
- `cancel_if_invalid = yes`：如果 `available` 条件不再满足，自动取消进行中的此国策
- `cancel`：取消条件块。与 `cancel_if_invalid` 类似但更灵活
- `will_lead_to_war_with = <TAG>`：提示玩家此国策将导致与某国开战（UI 警示）
- `search_filters`：在国策树搜索栏中的过滤分类

### 步骤 2.4：添加互斥关系

互斥国策是国策树设计的核心机制。两种实现方式：

**方式一：使用 mutually_exclusive（推荐）**
```
focus = {
    id = my_army_focus
    # ...
    mutually_exclusive = {
        focus = my_navy_focus
    }
}

focus = {
    id = my_navy_focus
    # ...
    mutually_exclusive = {
        focus = my_army_focus
    }
}
```

**方式二：使用 completion_reward 中设置 flag + available 检查**
```
# 在完成奖励中设置标记
completion_reward = {
    set_country_flag = chose_army_path
}

# 在互斥节点的 available 中检查
available = {
    NOT = { has_country_flag = chose_army_path }
}
```

推荐方式一用于简单互斥，方式二用于复杂门控逻辑（多个前置条件组合）。

### 步骤 2.5：添加动态节点（条件性显示）

某些节点应根据国家状态动态显示/隐藏：

```
focus = {
    id = my_war_focus
    # ...

    available = {
        has_war = yes
    }

    cancel_if_invalid = yes  # 战争结束后自动取消此国策
    cancel = {
        NOT = { has_war = yes }
    }

    # ...
}
```

常用动态条件：
- `has_war = yes/no`：是否在战争中
- `has_government = <ideology>`：当前政体
- `date > 1939.1.1`：日期条件
- `has_country_flag = <flag>`：自定义标记
- `has_idea = <spirit_id>`：拥有某国家精神

### 步骤 2.6：代码审查和语法检查

生成所有节点后，在 VS Code 中：
1. 检查 CWTools 的 PROBLEMS 面板是否有红色错误（括号不匹配、未知字段等）
2. 检查所有 `id` 一致性：focus_tree.id、focus.id、prerequisite.focus、mutually_exclusive.focus 中的所有引用
3. 检查所有坐标是否有重叠（两个节点在同一 x,y 位置）
4. 确认所有 `completion_reward` 块中的效果使用正确的语法和作用域

### 阶段二检查点

- [ ] focus_tree 外壳已生成（含 id 和 country 块）
- [ ] 所有设计中的节点已转换为完整 focus 块
- [ ] 每个节点包含：`id` `icon` `x` `y` `relative_position_id` `cost` `completion_reward`
- [ ] 互斥关系已正确设置（双向引用）
- [ ] 动态节点的 `available` 和 `cancel` 条件已添加
- [ ] CWTools 无语法错误

---

## 阶段三：在历史文件中关联国家

目标：将国策树与国家关联，使其在游戏中生效。

### 步骤 3.1：编辑国家历史文件

打开 `history/countries/<TAG> - <Country Name>.txt`，确保包含：

```
load_focus_tree = <MOD_PACKAGE>_focus_tree
```

说明：
- `<MOD_PACKAGE>_focus_tree` 必须与步骤 2.2 中 focus_tree 的 `id` 完全一致
- 如果国家历史文件中有多条 `load_focus_tree` 语句（通过 if/else 分支），确保目标分支正确指向你的树
- 如果这是该国家的唯一国策树，应无条件加载；如果有替代树（按意识形态分支），应放在合适的条件块中

### 步骤 3.2：验证 Tag 匹配

在 focus_tree 的 `country` 块中：
```
country = {
    factor = 0
    modifier = {
        add = 10
        tag = <TARGET_TAG>
    }
}
```

确保 `<TARGET_TAG>` 与历史文件的 Tag 完全一致。如果使用 `add = 10` 作为唯一正权重，且 `factor = 0`，则只有此 Tag 获得该树。

### 步骤 3.3：处理通用国策树（可选）

如果要让多个国家使用同一棵树：
```
country = {
    factor = 0
    modifier = {
        add = 10
        tag = TAG_A
    }
    modifier = {
        add = 10
        tag = TAG_B
    }
    modifier = {
        add = 10
        tag = TAG_C
    }
}
```

或者使用通配条件：
```
country = {
    factor = 0
    modifier = {
        add = 10
        OR = {
            tag = TAG_A
            tag = TAG_B
            tag = TAG_C
        }
    }
}
```

### 阶段三检查点

- [ ] 国家历史文件包含 `load_focus_tree = <正确的 focus_tree id>`
- [ ] focus_tree 的 `country` 块中 Tag 与国家文件一致
- [ ] 如果通用树，所有目标 Tag 已列出

---

## 阶段四：添加本地化

目标：为所有国策节点添加英文（必须）和中文（推荐）的显示文本。

### 步骤 4.1：创建或扩展本地化文件

在 `localisation/english/` 下创建 `<MOD_PACKAGE>_focus_l_english.yml`（或追加到已有文件）：

```yml
l_english:
 # 国策树名称
 <MOD_PACKAGE>_focus_tree:0 "<Country> National Focus Tree"

 # 国策节点
 <MOD_PACKAGE>_industrial_expansion:0 "Industrial Expansion"
 <MOD_PACKAGE>_industrial_expansion_desc:0 "Our nation must expand its industrial base to compete on the world stage. New factories will be constructed across the heartland."

 <MOD_PACKAGE>_army_reform:0 "Army Reform"
 <MOD_PACKAGE>_army_reform_desc:0 "The armed forces require modernization. We will reform our military doctrine and expand our officer corps."

 # ... 每个 focus id 对应一个名称和一个描述
```

### 步骤 4.2：本地化命名规范

| 条目类型 | 命名格式 | 示例 |
|----------|----------|------|
| 国策名称 | `<focus_id>:0 "<Name>"` | `my_army_focus:0 "Strengthen the Army"` |
| 国策描述 | `<focus_id>_desc:0 "<Description>"` | `my_army_focus_desc:0 "Our army..."` |
| 国策树名称 | `<tree_id>:0 "<Name>"` | `my_focus_tree:0 "My Country Focus Tree"` |

描述写作指南：
- 长度：1-3 句，在国策详情窗口中舒适显示
- 内容：描述战略目标+历史背景，避免啰嗦
- 风格：符合 HOI4 原版的客观、冷静的叙述语气
- 使用 `§Y<text>§!` 包裹高亮颜色（黄色高亮关键术语）：`§YBerlin§! will be...`

### 步骤 4.3：添加中文本地化（推荐）

在 `localisation/simp_chinese/` 下创建对应的中文 `.yml` 文件：

```yml
l_simp_chinese:
 <MOD_PACKAGE>_focus_tree:0 "<国家名>国策树"
 <MOD_PACKAGE>_industrial_expansion:0 "工业扩张"
 <MOD_PACKAGE>_industrial_expansion_desc:0 "我国必须扩张工业基础以在世界舞台上竞争。新的工厂将在心脏地带建立。"
```

注意：
- 中文本地化文件也必须是 UTF-8 with BOM
- 如果 mod 只有中文玩家受众，可以将中文放在 `localisation/english/` 作为默认

### 步骤 4.4：编码验证

本地化编码是最常见的错误来源。验证：
1. 英文 `.yml` 文件：UTF-8 with BOM
2. 中文 `.yml` 文件：UTF-8 with BOM
3. 在 VS Code 右下角状态栏检查编码标签
4. 如果编码错误：点击编码标签 → "Save with Encoding" → 选择正确的 UTF-8 with BOM

### 阶段四检查点

- [ ] 每个 focus 节点有对应的 `<id>:0` 名称条目和 `<id>_desc:0` 描述条目
- [ ] focus_tree 有对应的名称条目
- [ ] 所有 `.yml` 文件为 UTF-8 with BOM
- [ ] 中文本地化已添加（如需要）

---

## 阶段五：测试

目标：验证所有国策显示、执行和互斥逻辑正确。

### 步骤 5.1：启动测试环境

1. 以 `-debug` 模式启动 HOI4
2. 勾选你的 mod
3. 加载包含目标国家的剧本（如 1936 年）
4. 进入游戏后按 `~` 打开控制台
5. 输入 `tdebug` 开启调试信息

### 步骤 5.2：基础显示测试

切换到目标国家（`tag <TAG>`），打开国策树界面，检查：

- [ ] 国策树名正确显示
- [ ] 所有节点名称正确显示（非空白、非 `<missing string>`）
- [ ] 所有节点描述在悬停时正确显示
- [ ] 所有节点位置正确（无重叠、无孤立节点，布局与设计一致）
- [ ] GFX 图标正常渲染（非紫色方框）

如果节点名显示为 `<id>_desc` 而非实际文本，说明本地化 key 拼写不一致或本地化文件未正确加载。

### 步骤 5.2a：引导用户调整坐标（关键）

**AI 生成的 x/y 坐标是初始值，用户需要根据游戏内实际效果微调：**

1. **告知用户**：国策树坐标需要游戏中验证，当前坐标是初始布局，可能需要调整
2. **提供调整方法**：
   - 游戏内按 `~` 打开控制台
   - 输入 `reloadfocus <focus_tree_id>` 实时重载国策树（无需重启游戏）
   - 修改 `.txt` 文件中的 `x`/`y` 值 → 保存 → 控制台 `reloadfocus` → 立即生效
3. **推荐工具**：使用 **HOI4 Content Maker** 可视化拖拽节点，自动生成正确坐标
4. **坐标规则**：
   - `x` 水平位置（间距建议 1-3），`y` 垂直位置（间距建议 1-2）
   - 互斥节点共享同一 `x`、不同 `y`
   - `relative_position_id` 决定相对位置参照节点
5. **生成后必须提醒用户**：「请用 `reloadfocus` 命令在游戏中检查国策树布局，调整 x/y 坐标直到满意」

### 步骤 5.3：功能测试

对每个节点，使用以下方法快速测试（在 `-debug` 模式下）：

**快速完成国策（控制台命令）：**
```
fa <focus_id>      # 立即完成指定国策（无视前提条件）
focus.autocomplete # 切换自动完成模式——所有国策 1 天完成
```

**测试流程：**
1. 使用 `fa <root_focus_id>` 完成根节点
2. 使用 `fa <child_focus_id>` 逐级测试每个子节点
3. 观察每次 `fa` 后是否正确执行 `completion_reward`
4. 检查互斥节点：完成互斥组中一个节点后，尝试 `fa` 另一个——应为不可用

### 步骤 5.4：互斥逻辑测试

互斥关系是国策树最容易出错的地方。逐组测试：

1. 解除所有已完成国策的控制台命令（如有）或重新加载存档
2. 正常完成互斥组中的节点 A
3. 确认节点 B 变为灰显（不可选）
4. 同样，新开档正常完成节点 B，确认节点 A 变为灰显

如果互斥失败（两个互斥节点都能完成）：检查 `mutually_exclusive` 中的 focus id 是否正确，确保双向引用。

### 步骤 5.5：效果验证

使用以下方法验证效果正确执行：

| 效果类型 | 验证方法 |
|----------|----------|
| 政治点数 | 查看顶部 PP 数值变化 |
| 稳定度/战争支持度 | 查看顶部百分比变化 |
| 国家精神 | 打开政治界面（F5），检查精神列表 |
| 工厂 | 打开建造界面，检查可用工厂数 |
| 研究加成/槽位 | 打开科技界面（F4） |
| 领土宣称/核心 | 在外交地图模式检查 |
| 战争目标 | 打开外交界面检查宣战理由 |
| 变量 | 控制台输入 `get_var <var_name>` |
| 标记 (flag) | 控制台输入 `check_flag <flag_name>` |
| 事件触发 | 检查事件弹窗是否出现 |

### 步骤 5.6：错误日志检查

完成测试后，打开 `Documents\Paradox Interactive\Hearts of Iron IV\logs\error.log`：

1. 搜索你的 `<MOD_PACKAGE>` 前缀
2. 记录所有相关错误
3. 分类处理：

| 错误类型 | 处理方法 |
|----------|----------|
| `"<id>" is not a valid focus` | 检查 prerequisite 或 mutually_exclusive 中的 focus id 拼写 |
| `"<key>" is not a valid localisation key` | 检查本地化文件中的 key 拼写和编码 |
| `Unknown trigger/effect` | 检查 `available` 或 `completion_reward` 中的语法 |
| `Invalid scope` | 检查作用域（THIS/FROM/ROOT 使用错误） |

### 步骤 5.7：修复循环

```
测试 → 发现错误 → 查看 error.log → 定位问题文件/行 → 修复 → 重启游戏 → 测试
```

重复此循环直到：
- error.log 中无 `<MOD_PACKAGE>` 相关条目
- 所有节点显示、执行、互斥均正常

### 阶段五检查点

- [ ] 所有节点名称和描述正确显示
- [ ] GFX 图标正常渲染
- [ ] 节点位置符合设计布局
- [ ] 互斥逻辑全部正确
- [ ] completion_reward 效果正确执行
- [ ] 动态节点的 available 条件正确生效
- [ ] cancel_if_invalid 节点在条件变化时正确取消
- [ ] error.log 中无相关错误

---

## 检查清单（总）

### 设计与结构
- [ ] 国策树主题和分支结构已确定
- [ ] 节点总数合适（小国 20-40, 大国 50-100）
- [ ] 成本与奖励平衡合理
- [ ] 布局草图已完成

### 代码与语法
- [ ] focus_tree `id` 全局唯一
- [ ] 所有聚焦 `id` 前缀一致（`<MOD_PACKAGE>_`）
- [ ] `x, y` 坐标无重叠
- [ ] 前置和互斥引用正确
- [ ] `completion_reward` 块语法正确
- [ ] CWTools 无报错

### 国家关联
- [ ] `history/countries/<TAG>.txt` 有正确的 `load_focus_tree`
- [ ] focus_tree 的 `country` 块中 tag 正确

### 本地化
- [ ] 所有 `<focus_id>:0` 名称条目存在
- [ ] 所有 `<focus_id>_desc:0` 描述条目存在
- [ ] `.yml` 文件编码为 UTF-8 with BOM
- [ ] 名称和描述在游戏中正确显示

### 测试
- [ ] 国策树在游戏中可见
- [ ] 所有节点可正常完成
- [ ] 效果执行正确
- [ ] 互斥逻辑正确
- [ ] error.log 无相关错误

---

> **下一步**: 国策树完成后，可以考虑：
> - 通过事件触发国策树加载切换 → `workflows/add-event-chain.md`
> - 添加国策关联的决议 → `references/06-decisions.md` + `templates/decision.txt`
> - 添加国策解锁的国家精神 → `references/07-ideas.md` + `templates/idea.txt`
> - 为 mod 添加自定义 GFX 图标 → `references/11-map-states.md`（gfx 路径）
