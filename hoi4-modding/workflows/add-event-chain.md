# 添加事件链

> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03
> **前置要求**: Mod 骨架已创建（`workflows/new-mod.md`），目标国家已存在
> **关联参考**: `references/05-events.md` `references/12-localisation.md`
> **关联模板**: `templates/event.txt`

## 概述

本工作流指导 AI 代理创建完整的事件链（Event Chain）——多个事件通过触发关系串联，讲述故事或驱动游戏机制。流程覆盖故事设计、事件编写、事件串联、本地化和测试五个阶段。完成后你将拥有一个逻辑完整、触发正确、文本流畅的事件链系统。

HOI4 的事件系统分为两类：
- **country_event**（国家事件）：针对特定国家，有独立弹窗，玩家做出选项决策
- **news_event**（新闻事件）：全局广播，所有国家可见（如二战爆发、政府崩溃等）

大多数 mod 事件链使用 country_event 驱动故事线，用 news_event 广播重大结果。

适用场景：
- 新增历史事件链（如某国内战、外交危机、经济改革）
- 为国策树添加配套事件（国策触发事件，事件返回效果）
- 创建替代历史剧情线（如德国民主化、日本转向共产主义）
- 添加随机事件池（通过 MTTH 机制触发）

---

## 阶段一：设计事件流程

目标：在编写代码前设计故事线、分支逻辑和选项后果。

### 步骤 1.1：确定事件链结构

定义事件链的全局参数：

| 参数 | 说明 | 示例 |
|------|------|------|
| 事件总数 | 链中事件个数 | 5-15 个 |
| 起始触发 | 第一个事件的触发方式 | 国策完成、日期到达、条件满足（MTTH）、手动决议 |
| 链式传递 | 事件间传递方式 | `country_event = { id = X days = Y }` |
| 分支数量 | 玩家的选择分支数 | 2-4 条 |
| 结束条件 | 链的终止条件 | 最后一个事件完成、设置全局标记 |

### 步骤 1.2：设计故事线

为每个事件填写以下设计卡片：

```
事件 #N: <事件名称>
├── 触发条件：<条件>
├── 事件类型：country_event / news_event
├── 触发方式：is_triggered_only / MTTH / on_actions
├── 事件描述：<1-2 句描述>
├── 选项 A：<名称> → <效果>
├── 选项 B：<名称> → <效果>
└── 后续：触发事件 #N+1（条件：<触发选项>）
```

### 步骤 1.3：设计分支逻辑

事件链的分支由选项的 `country_event = { id = X }` 驱动：不同选项可以触发不同后续事件，从而实现故事分支。

**线性链（无分支）：**
```
事件 1 → 事件 2 → 事件 3 → 事件 4（结束）
```

**分支链（并行路径）：**
```
             ┌→ 事件 2a（路径 A）→ 事件 3a
事件 1 ─────┤
             └→ 事件 2b（路径 B）→ 事件 3b
```

**汇聚链（分支后合并）：**
```
             ┌→ 事件 2a ─┐
事件 1 ─────┤             ├→ 事件 3（汇聚）
             └→ 事件 2b ─┘
```

### 步骤 1.4：设计选项后果平衡

每个选项应提供不同的利弊权衡：

| 选项类型 | 典型利 | 典型弊 | 示例场景 |
|----------|--------|--------|----------|
| 激进选项 | 大量 PP/工厂/领土 | 稳定度 -10%, 战争支持度 -5% | 军事冒险 |
| 温和选项 | 少量 PP, 稳定度 +5% | 无负面, 但错失激进奖励 | 外交妥协 |
| 延迟选项 | 额外研究加成 | 延迟核心机制触发 | 先做准备 |
| 道德选项 | 意识形态支持 +10% | 关系惩罚, 国际反感 | 人道主义 |
| 实用选项 | 立即工厂, 装备 | 意识形态支持 -5% | 经济利益优先 |

### 阶段一检查点

- [ ] 事件总数和触发链结构已确定
- [ ] 每个事件的设计卡片已填写完成
- [ ] 分支逻辑清晰（线性/分支/汇聚）
- [ ] 选项后果平衡合理

---

## 阶段二：使用模板编写事件

目标：基于设计卡片，使用 `templates/event.txt` 模板生成每个事件的完整代码。

### 步骤 2.1：准备模板

确认 `templates/event.txt` 模板可用。该模板覆盖以下变体：
- **country_event**（国家事件）——最常用
- **news_event**（新闻事件）——用于全局广播
- **is_triggered_only 事件**——仅被其他事件或国策触发
- **MTTH 事件**——根据条件随机触发

如果模板缺失，参考原版文件自行构建。推荐参考：`events/WTT_Germany.txt` 或 `events/WTT_Japan.txt`。

### 步骤 2.2：声明命名空间

在事件文件的**第一行**声明命名空间：

```
add_namespace = <MOD_PACKAGE>_<namespace>
```

命名空间规范：
- 使用小写+下划线
- 每个 mod 可以定义多个命名空间（按模块分：`my_mod_political`、`my_mod_economic`）
- 每个命名空间内的事件 ID 从 1 开始编号
- 事件全局 ID 格式：`<namespace>.<n>`（如 `my_mod_political.1`）

示例：
```
add_namespace = my_mod_political   # 政治事件
add_namespace = my_mod_economic    # 经济事件
add_namespace = my_mod_war         # 战争事件
```

如果所有事件放在同一个文件中，可以只用 `add_namespace = <MOD_PACKAGE>`。

### 步骤 2.3：编写 country_event

生成每个事件的完整代码块。以下是完整结构：

**is_triggered_only 事件（被其他事件/国策触发）：**

```
country_event = {
    id = <namespace>.<event_number>
    title = <namespace>.<event_number>.t
    desc = <namespace>.<event_number>.d
    picture = GFX_report_event_<picture_name>

    is_triggered_only = yes

    trigger = {
        # 接收事件的条件（通常比较简单，因为触发者已经筛选过）
        <conditions>
    }

    immediate = {
        # 事件弹出前立即执行的效果（可选）
        <pre_effects>
    }

    option = {
        name = <namespace>.<event_number>.a
        <effects_or_triggers>
        ai_chance = {
            factor = <base_weight>
            modifier = {
                factor = <modifier_value>
                <conditions>
            }
        }
    }

    option = {
        name = <namespace>.<event_number>.b
        <effects_or_triggers>
    }
}
```

**MTTH 事件（随机触发）：**

```
country_event = {
    id = <namespace>.<event_number>
    title = <namespace>.<event_number>.t
    desc = <namespace>.<event_number>.d
    picture = GFX_report_event_<picture_name>

    trigger = {
        # 事件检查的前提条件
        <conditions>
    }

    mean_time_to_happen = {
        days = <base_days>
        modifier = {
            factor = <factor_value>
            <conditions>
        }
    }

    option = {
        name = <namespace>.<event_number>.a
        <effects>
    }
}
```

### 步骤 2.4：字段详解

**事件基本字段：**

| 字段 | 说明 | 必填 |
|------|------|------|
| `id` | 全局唯一事件 ID，格式 `<namespace>.<n>` | 是 |
| `title` | 本地化键：事件标题 | 是 |
| `desc` | 本地化键：事件描述文本 | 是 |
| `picture` | 事件配图 GFX 引用 | 是 |
| `is_triggered_only = yes` | 仅被其他代码触发（不会自动/MTTH 触发） | 条件必填 |
| `trigger` | 事件触发条件（MTTH 事件中为检查条件；triggered 事件中为接收条件） | 是 |
| `mean_time_to_happen` | MTTH 定义（与 is_triggered_only 互斥） | 条件必填 |
| `fire_only_once = yes` | 此事件仅触发一次 | 否 |
| `major = yes` | 标记为重大事件（存储于历史日志） | 否 |
| `immediate` | 事件弹出前立即执行的效果 | 否 |
| `after` | 事件关闭后（选项处理后）执行的效果 | 否 |

**事件配图常用 GFX 引用：**

```
GFX_report_event_generic               # 通用事件（最常用）
GFX_report_event_ger_civil_war         # 内战
GFX_report_event_coup                  # 政变
GFX_report_event_election              # 选举
GFX_report_event_treaty                # 条约/外交
GFX_report_event_economic_crisis       # 经济危机
GFX_report_event_world_tension        # 世界紧张度
GFX_report_event_generic_war           # 战争
GFX_report_event_munich_conference     # 慕尼黑会议
```

### 步骤 2.5：编写选项块

每个 option 是玩家可以在事件弹窗中点击的按钮：

```
option = {
    name = <namespace>.<event_number>.<option_letter>   # 本地化键：选项文本
    trigger = {
        # 选项可用条件（可选）——不满足时灰显
        <conditions>
    }
    <effects>                                          # 选择后的效果
    ai_chance = {
        factor = <base_weight>
        modifier = {
            factor = <multiplier>
            <conditions>
        }
    }
}
```

**ai_chance 系统：**
- `factor = 1` 是基准权重
- `factor = 0` 表示 AI 绝不会选择（除非无其他可用选项）
- AI 选择概率 = 此选项的最终 factor / 所有可用选项的 factor 之和
- modifier 中的 factor 是乘数：`factor = 2` 使概率加倍，`factor = 0.5` 使概率减半
- modifier 条件常用：AI 意识形态、是否有战争、是否有某国家精神

**AI 选择权重示例：**

```
ai_chance = {
    factor = 1
    modifier = {
        factor = 10        # 法西斯 AI 极大概率选激进选项
        has_government = fascism
    }
    modifier = {
        factor = 0.1       # 民主 AI 几乎不选
        has_government = democratic
    }
}
```

### 步骤 2.6：编写 news_event

news_event 用于全局广播重大消息：

```
news_event = {
    id = <namespace>.<event_number>
    title = <namespace>.<event_number>.t
    desc = <namespace>.<event_number>.d
    picture = GFX_report_event_<picture_name>

    is_triggered_only = yes

    major = yes

    option = {
        name = <namespace>.<event_number>.a
        trigger = {
            tag = ROOT          # 仅触发者可见此选项
        }
        <effects_on_triggerer>
    }

    option = {
        name = <namespace>.<event_number>.b
        trigger = {
            NOT = { tag = ROOT } # 其他国家仅看到此选项
        }
        <effects_on_others>
    }
}
```

news_event 的关键特点：
- 所有国家都会收到（除非通过 option trigger 过滤）
- 被 `major = yes` 标记的 news_event 会出现在时间线/历史日志中
- option 的 `trigger` 决定哪些国家看到哪个选项：`tag = ROOT` 仅触及发者，`NOT = { tag = ROOT }` 仅限其他国家
- ROOT 在 news_event 中等于事件的触发者

### 阶段二检查点

- [ ] 命名空间已声明（`add_namespace = <MOD_PACKAGE>_xxx`）
- [ ] 每个事件有完整的 `id` `title` `desc` `picture` `option` 结构
- [ ] 所有事件要么 `is_triggered_only = yes`，要么有 `mean_time_to_happen` 块
- [ ] 每个 option 有 `name` 本地化键和一个或多个效果
- [ ] 需要 AI 决策的 option 有 `ai_chance` 块
- [ ] CWTools 无语法错误

---

## 阶段三：串联事件

目标：将独立的事件串联成完整的事件链。

### 步骤 3.1：链式触发（事件→事件）

在事件 A 的 option 效果中触发事件 B：

```
option = {
    name = my_event.1.a
    add_stability = 0.05
    country_event = { id = my_event.2 }          # 立即触发
}
```

带延迟的链式触发：

```
option = {
    name = my_event.1.a
    country_event = { id = my_event.2 days = 7 }     # 7 天后触发
    country_event = { id = my_event.3 days = 30 }    # 30 天后触发
}
```

使用小时作为延迟单位（用于快速连续事件）：

```
country_event = { id = my_event.2 hours = 12 }       # 12 小时后
```

### 步骤 3.2：条件分支触发

同一事件选项 A 和选项 B 触发不同的后续事件：

```
# 事件 1
option = {
    name = my_event.1.a    # "激进"
    country_event = { id = my_event.2a days = 7 }
}
option = {
    name = my_event.1.b    # "温和"
    country_event = { id = my_event.2b days = 7 }
}
```

或根据条件触发：

```
option = {
    name = my_event.1.a
    if = {
        limit = { has_war = yes }
        country_event = { id = my_event.2_war }
    }
    else = {
        country_event = { id = my_event.2_peace }
    }
}
```

### 步骤 3.3：国策触发事件

在国策的 `completion_reward` 中触发事件链的第一个事件：

```
focus = {
    id = my_focus_trigger
    # ...
    completion_reward = {
        add_political_power = 50
        country_event = { id = my_event_chain.1 }  # 启动事件链
    }
}
```

注意作用域：
- 在 `completion_reward` 中操作的作用域是 ROOT（拥有国策的国家）
- `country_event = { id = X }` 默认将事件发送给 THIS，在国策完成奖励中 THIS = ROOT
- 如果要发送给其他国家：`TAG = { country_event = { id = X } }`

### 步骤 3.4：决议触发事件

在决议的 `complete_effect` 中触发事件：

```
decision = {
    name = my_decision
    # ...
    complete_effect = {
        country_event = { id = my_event_chain.1 hours = 4 }
    }
}
```

### 步骤 3.5：全局标记门控

使用全局标记（global_flag）防止事件链重复触发：

```
# 在事件链起始检查
trigger = {
    NOT = { has_global_flag = my_chain_completed }
    <other_conditions>
}

# 在事件链结束时设置标记
option = {
    name = my_chain_end.1.a
    set_global_flag = my_chain_completed
}
```

### 步骤 3.6：事件链追踪图

创建事件链追踪表，用于验证：

```
| 事件 ID          | 触发者              | 触发方式              | 后续事件                       |
|------------------|---------------------|-----------------------|--------------------------------|
| my_event.1       | 国策 my_focus       | is_triggered_only     | my_event.2 (days=7)            |
| my_event.2       | my_event.1 选项 A   | is_triggered_only     | my_event.3a 或 my_event.3b     |
| my_event.3a      | my_event.2 选项 A   | is_triggered_only     | my_event.4 (days=14)           |
| my_event.3b      | my_event.2 选项 B   | is_triggered_only     | my_event.4 (days=14)           |
| my_event.4       | my_event.3a 或 .3b  | is_triggered_only     | (结束)                          |
```

### 阶段三检查点

- [ ] 事件链的起始触发器已确定（国策/决议/MTTH/其他事件）
- [ ] 每个选项正确使用 `country_event = { id = X days = Y }` 链式触发
- [ ] 分支和汇聚逻辑正确
- [ ] 使用 global_flag 防止重复触发（如需要）
- [ ] 事件链追踪表完整且无循环引用

---

## 阶段四：添加本地化

目标：为事件标题、描述和选项文本添加英文和中文本地化。

### 步骤 4.1：创建事件本地化文件

在 `localisation/english/` 下创建 `<MOD_PACKAGE>_events_l_english.yml`：

```yml
l_english:
 # 事件 <namespace>.1 - 起始事件
 <namespace>.1.t:0 "The Crisis Begins"
 <namespace>.1.d:0 "Our nation faces a critical moment. The decisions we make in the coming days will shape our future for generations. The council has gathered to debate our course of action."
 <namespace>.1.a:0 "We must act decisively."
 <namespace>.1.b:0 "Caution is the better part of valor."
 <namespace>.1.c:0 "Let us seek a diplomatic solution."

 # 事件 <namespace>.2 - 后续事件
 <namespace>.2.t:0 "The Aftermath"
 <namespace>.2.d:0 "Our decision has set events in motion. Now we must deal with the consequences."
 <namespace>.2.a:0 "So be it."
```

### 步骤 4.2：本地化命名规范

| 条目类型 | 命名格式 | 说明 |
|----------|----------|------|
| 事件标题 | `<namespace>.<n>.t:0 "<Title>"` | 事件弹窗顶部标题 |
| 事件描述 | `<namespace>.<n>.d:0 "<Description>"` | 事件正文 |
| 选项文本 | `<namespace>.<n>.<letter>:0 "<Option Text>"` | 每个选项按钮的文字 |

### 步骤 4.3：文本写作指南

**标题（.t）：**
- 长度：5-10 个词
- 格式：短语/名词化（如 "The Crisis"、"A New Dawn"），不是完整句子
- 使用标题大写（Title Case）
- 示例：`"The Berlin Conference"`、`"Economic Reforms Begin"`、`"A Fateful Decision"`

**描述（.d）：**
- 长度：2-5 句
- 风格：HOI4 原版的客观、略带文学感的叙述语气
- 结构：情景描述 + 背景信息 + 可选的历史脚注
- 使用 `§Y<text>§!` 高亮关键术语（国家名、领袖名、重要概念）
- 使用 `§R<text>§!` 高亮危险/警告信息
- 使用 `\n\n` 创建段落分隔
- 示例：
  ```
  "The industrial heartland of §YGermany§! lies exposed. Without adequate air defenses, our factories remain vulnerable to strategic bombing campaigns.\n\nMilitary advisors recommend immediate allocation of resources to anti-aircraft installations."
  ```

**选项文本（.a/.b/.c）：**
- 长度：3-8 个词
- 语气：果断、行动导向
- 反映玩家选择的立场/态度
- 示例：`"Crush the Opposition"`、`"Seek a Compromise"`、`"Wait and See"`

### 步骤 4.4：添加中文本地化

在 `localisation/simp_chinese/` 下创建对应的中文 `.yml` 文件：

```yml
l_simp_chinese:
 <namespace>.1.t:0 "危机开始"
 <namespace>.1.d:0 "我们的国家正面临关键时刻。未来几天我们做出的决策将影响几代人的命运。议会已召集起来讨论我们的行动方针。"
 <namespace>.1.a:0 "我们必须果断行动。"
 <namespace>.1.b:0 "谨慎是勇敢的一部分。"
 <namespace>.1.c:0 "让我们寻求外交解决方案。"
```

### 阶段四检查点

- [ ] 每个事件的 `.t`（标题）、`.d`（描述）本地化条目存在
- [ ] 每个选项的 `.<letter>`（选项文本）本地化条目存在
- [ ] 文本长度和语气符合 HOI4 风格
- [ ] 颜色标记（`§Y`、`§R`）使用正确且闭合（`§!`）
- [ ] 所有 `.yml` 文件为 UTF-8 with BOM

---

## 阶段五：测试

目标：验证所有事件触发、显示和执行正确。

### 步骤 5.1：启动测试环境

1. 以 `-debug` 模式启动 HOI4
2. 勾选你的 mod
3. 加载包含目标国家的剧本
4. 按 `~` 打开控制台
5. 输入 `tdebug` 开启调试信息

### 步骤 5.2：事件触发测试

**方式一：通过正常渠道触发**
如果事件由国策触发：切换 Tag → 完成对应国策 → 检查事件是否弹出。

**方式二：通过控制台强制触发（快速测试）**
```
event <namespace>.<event_number>              # 对当前国家触发事件
event <namespace>.<event_number> <TAG>        # 对指定国家触发事件
```

**方式三：测试 MTTH 事件**
```
# 无法直接触发 MTTH 事件（is_triggered_only = no 的事件），需满足 trigger 条件
# 使用以下命令快速检查 MTTH 事件是否在事件池中：
event <namespace>.<event_number>              # 如果事件是 MTTH 型但不满足条件，会无效果
```

### 步骤 5.3：显示测试

对每个已触发的事件：

- [ ] 事件标题正确显示（非空白、非 `<namespace>.<n>.t` 原样）
- [ ] 事件描述正确显示且排版正常
- [ ] 事件图片正确显示（非紫色方框）
- [ ] 所有选项文本正确显示
- [ ] 不可用选项（含 trigger 但条件不满足的选项）灰显
- [ ] AI 选项带 AI 标记显示

### 步骤 5.4：功能测试

对每个选项：

1. 点击选项
2. 检查效果是否正确执行：

| 效果类型 | 验证方法 |
|----------|----------|
| 政治点数 | PP 数值变化 |
| 稳定度/战争支持度 | 顶部百分比变化 |
| 国家精神 | 政治界面（F5）精神列表 |
| 宣战/和平 | 外交界面 |
| 工厂 | 建造界面 |
| 事件触发 | 下一个事件是否弹出（检查 ID） |
| 标记 | 控制台 `check_flag <flag_name>` |
| 变量 | 控制台 `get_var <var_name>` |

### 步骤 5.5：事件链完整性测试

从事件链起点开始，遍历所有分支路径：

```
测试路径 A：事件 1 → 选项 A → 事件 2a → 选项 A → 事件 3a → 结束
测试路径 B：事件 1 → 选项 B → 事件 2b → 选项 A → 事件 3b → 结束
（以此类推，覆盖所有分支和所有选项组合）
```

验证：
- [ ] 链中每个事件依次触发（无跳过、无重复）
- [ ] 延迟触发的时间正确（`days=7` 约在 7 天后触发）
- [ ] 分支选择正确导向不同后续事件
- [ ] 汇聚点在两个分支后都能正确到达
- [ ] 事件链结束后不再有后续事件触发（无循环/幽灵事件）

### 步骤 5.6：重复触发防护测试

测试 global_flag 和 fire_only_once 的防护效果：
1. 完成整个事件链一次
2. 尝试重新满足起始触发条件
3. 确认事件链不再启动（应由 `NOT = { has_global_flag = xxx }` 阻止）
4. 如果期望可重复触发：确认起始事件正确重复弹出

### 步骤 5.7：错误日志检查

打开 `Documents\Paradox Interactive\Hearts of Iron IV\logs\error.log`：

1. 搜索你的 `<namespace>` 前缀
2. 分类处理所有相关错误：

| 错误类型 | 处理方法 |
|----------|----------|
| `"<namespace>.<n>" is not a valid event` | 检查 `country_event = { id = X }` 引用中的 id 拼写 |
| `"<namespace>.<n>.t" is not a valid localisation key` | 检查本地化文件中标题 key 的拼写和编码 |
| `Unknown trigger` in event | 检查 `trigger = {}` 块中的条件语法 |
| `Unknown effect` in option | 检查 option 中的效果语法 |
| `Invalid scope for effect` | 检查效果块中的作用域（是否需要 FROM/ROOT 嵌套） |
| `Event has no options` | 检查 option 块的 trigger 是否过于严格导致所有选项灰显 |

### 步骤 5.8：修复循环

```
测试 → 发现错误 → 查看 error.log → 定位问题事件和行号 → 修复 → 重启游戏 → 重新测试
```

重复此循环直到：
- error.log 中无 `<namespace>` 相关条目
- 所有事件触发、显示、执行、链式传递均正常
- 所有分支路径都无错误

### 阶段五检查点

- [ ] 每个事件可通过预期方式触发（控制台或正常渠道）
- [ ] 所有标题、描述、选项文本正确显示
- [ ] 事件配图正常渲染
- [ ] 所有选项的效果正确执行
- [ ] 事件链传递正确（线性/分支/汇聚）
- [ ] 延迟触发时间正确
- [ ] 重复触发防护生效（如适用）
- [ ] error.log 中无相关错误

---

## 检查清单（总）

### 设计
- [ ] 故事线设计完整（事件卡片所有字段填写）
- [ ] 分支逻辑清晰（线性/分支/汇聚）
- [ ] 选项后果平衡合理

### 代码
- [ ] 命名空间已声明（`add_namespace`）
- [ ] 所有事件 ID 格式正确（`<namespace>.<n>`）
- [ ] 所有事件有 `title` `desc` `picture`
- [ ] `is_triggered_only` 或 `mean_time_to_happen` 正确设置
- [ ] 每个 option 有 `name` 和一个或多个效果
- [ ] AI 决策的 option 有 `ai_chance` 块
- [ ] 链式触发使用 `country_event = { id = X days = Y }`
- [ ] 全局标记用于防止重复触发（如需要）
- [ ] CWTools 无语法错误

### 本地化
- [ ] 所有 `.t` `.d` `.<letter>` 本地化条目存在
- [ ] 文本风格符合 HOI4 原版
- [ ] `.yml` 文件编码为 UTF-8 with BOM

### 测试
- [ ] 所有事件可正常触发
- [ ] 所有文本显示正确
- [ ] 所有效果执行正确
- [ ] 事件链完整性验证通过（所有分支路径）
- [ ] 重复触发防护生效
- [ ] error.log 无相关错误

---

> **下一步**: 事件链完成后，可以考虑：
> - 通过决议扩展事件触发能力 → `references/06-decisions.md` + `templates/decision.txt`
> - 使用脚本化效果简化重复逻辑 → `references/13-scripted.md`
> - 添加声音或特效到关键事件 → `references/05-events.md`（音效章节）
> - 发布 mod 到 Steam Workshop → `workflows/formalize-mod.md`
