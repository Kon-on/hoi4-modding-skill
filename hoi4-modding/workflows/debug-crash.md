# 工作流：崩溃排查

> **适用版本**: HOI4 1.18.2 | **最后更新**: 2026-06-03
>
> 参考: `references/14-debugging.md`
>
> 适用场景：mod 加载时报错、游戏启动时崩溃（CTD）、游戏中弹出红字、功能行为异常

## 概述

HOI4 mod 开发中遇到错误和崩溃是常态。本工作流提供了一个结构化的排查方法论，从最快速的信息收集到最深入的二分隔离，覆盖 90% 以上的常见问题。核心原则：**先读日志，再做假设——不要凭直觉猜测错误原因。**

**关键路径速查：**
```
logs/error.log  →  第一个错误  →  定位文件+行号  →  分类诊断  →  修复  →  验证
         ↕                                            ↕
    二分法隔离  ←  错误不明确/无error.log  ←  范围太大  ←  难以定位
```

**工具准备：**
- `Documents/Paradox Interactive/Hearts of Iron IV/logs/error.log` — 错误日志
- `Documents/Paradox Interactive/Hearts of Iron IV/logs/game.log` — 游戏详细日志
- VS Code + CWTools — 语法检查和括号匹配
- `-debug` 启动参数 — 开启调试模式

---

## 阶段一：定位错误

### 目标
从 error.log 中提取第一个错误信息，定位到具体文件和行号。

### 步骤

1. **定位日志文件**
   ```
   C:\Users\<用户名>\Documents\Paradox Interactive\Hearts of Iron IV\logs\error.log
   ```
   此文件在每次启动游戏时被覆盖，崩溃后立即读取。

2. **打开 error.log**
   - 使用 VS Code 或任意文本编辑器打开
   - 如果文件超过 100 行，通常意味着存在系统性问题（如 replace_path 缺失、编码错误）

3. **提取第一个错误**
   - **第一个错误最重要**——后续错误往往是第一个错误导致的连锁反应
   - 忽略与你的 mod 无关的错误（原版游戏也可能产生一些非致命 error）
   - 找到带有你 mod ID 或自定义 Tag 前缀的错误行

4. **解析错误信息**
   HOI4 error.log 的常见格式：
   ```
   [时间戳] [错误级别] [错误描述] [文件路径] [行号]

   示例：
   [22:15:30][trigger.cpp:1234]: Error: "Unknown trigger type: has_warm, near line: 42" in file: "common/national_focus/my_focus.txt"
   ```

   提取关键信息：
   - **文件路径**：哪个文件出错了
   - **行号**：near line: N（注意：有时行号因嵌套而偏移 ±5 行）
   - **错误描述**：具体什么不对（未知 key、类型错误、缺失引用等）

5. **假如 error.log 为空或只包含非致命错误**
   - 检查 `game.log`：搜索 `"Error"` 或你的 mod 名称
   - 检查 `exceptions.log`（如果存在）：包含 C++ 层面的异常信息
   - 使用 `-crash_data_log` 启动参数增强崩溃日志输出

### 输出物
- 第一个错误的具体描述、出错文件、大致行号

---

## 阶段二：分类诊断

### 目标
根据错误描述将问题归类，使用对应的诊断策略快速定位根本原因。

### 决策树

```
错误类型判定
│
├─ 编码错误？
│   特征：游戏启动即崩溃（CTD before main menu）、error.log 显示乱码
│   常见编码：error code 3221225477
│   → 检查 .txt 文件是否为 UTF-8 without BOM
│   → 检查 .yml 文件是否为 UTF-8 with BOM
│   → 用 VS Code 右下角编码指示器确认，另存为正确编码
│
├─ 括号/语法错误？
│   特征："Unexpected token" "Expected" "Missing closing bracket"
│   → 用 VS Code + CWTools 查看红色波浪线位置
│   → 点击 { 查看对应的 } 是否正确闭合
│   → 从错误行号附近，向上逐层检查是否有未闭合的块
│   → 注意：缺少一个 } 可能导致几十行之后才报错
│
├─ 路径/文件未找到？
│   特征："Could not find file" "Could not open file" "missing definition"
│   → 检查文件是否在正确的子目录中
│   → 检查文件名大小写（同一目录下文件名不能仅靠大小写区分）
│   → 检查 replace_path 是否覆盖了不该覆盖的路径
│   → grep 原版文件确认 vanilla 的目录结构
│
├─ 作用域错误？
│   特征："Invalid scope" "This trigger does not accept that scope" "Cannot execute effect in this scope"
│   → 检查 trigger/effect 是否放在了正确的块中
│   → 触发器放在 trigger/available/allowed/limit 块中；效果放在 effect/complete_effect 块中
│   → 检查 THIS / FROM / ROOT / PREV 是否指向了错误的实体
│   → 必要时在 game.log 中打印作用域信息：
│       log = "DEBUG: [THIS.GetName] [FROM.GetName]"
│
├─ 缺失引用？
│   特征："Could not resolve reference" "Unknown <type>: <id>"
│   → 检查引用的 id 是否拼写正确（区分大小写！）
│   → 检查被引用的对象是否在游戏加载范围内：
│       * 国策树 id 需要在 common/national_focus/ 下的文件中定义
│       * 国家 Tag 需要在 history/countries/ 下存在
│       * 装备 archetype 需要在 common/units/equipment/ 下定义
│   → 如果是跨 mod 引用，确认依赖 mod 已启用
│
├─ 重复定义？
│   特征："Duplicate <type>: <id>" "Already defined"
│   → 检查是否定义了与 vanilla 或其他 mod 相同的 id
│   → 检查是否在同一个 mod 中重复定义了同一 id
│   → 解决方案：修改 id 加入唯一前缀
│
└─ 游戏逻辑错误（不报错但行为异常）？
    特征：无 error.log 条目，但功能不符合预期
    → 在相关代码块中使用 log 命令输出关键变量值
    → 检查 trigger 条件是否过于宽松/严格
    → 检查 effect 是否真的在预期时机执行
    → 使用控制台命令手动触发事件/国策来验证逻辑
```

### 输出物
- 错误分类 + 对应的诊断结论

---

## 阶段三：二分法隔离

### 目标
当错误信息不明确、或怀疑是 mod 冲突、或无法定位到具体文件时，使用二分法逐步缩小范围。

### 步骤

1. **备份当前 mod**
   ```bash
   cp -r "path/to/your/mod" "path/to/your/mod_backup_$(date +%Y%m%d_%H%M)"
   ```

2. **禁用其他所有 mod**
   - 在启动器中只留你的 mod
   - 这样可以确定问题在你的 mod 中还是与其他 mod 冲突
   - 如果问题消失 → mod 冲突，跳到第 6 步
   - 如果问题仍在 → 你的 mod 本身存在问题

3. **第一轮二分：删除一半内容**
   - 临时将 mod 文件夹的一半子目录移出（建议从 `common/` `events/` `history/` 中选择）
   - 重新启动游戏
   - 结果判定：
     - 问题消失 → 问题在删除的那一半中
     - 问题仍在 → 问题在保留的那一半中

4. **第二轮二分：继续缩小**
   - 在问题范围内，继续删除一半文件
   - 重新测试
   - 重复此过程，直到缩小到 1-2 个文件

5. **定位具体行**
   - 在问题文件中，注释掉一半内容（块注释或使用 `#` 行注释）
   - 重新测试
   - 最终定位到具体行或块

6. **Mod 冲突排查**（如果第 2 步确认是冲突）
   - 恢复你的 mod 到完整版本
   - 逐步启用其他 mod：先启用一半 → 测试 → 进一步二分
   - 找到冲突 mod 后，分析两者的修改范围是否有交集
   - 常见冲突类型：
     - 两个 mod 修改了同一个原版文件（将其中一个改为 replace_path 或使用 patch 方式）
     - 两个 mod 定义了相同的 id（修改 id 前缀）
     - 两个 mod 对同一国家 Tag 做不同修改（协商或合并）

### 计时与效率
- 每次测试循环的耗时主要在游戏启动（30-60 秒）
- 二分法通常能在 5-7 次启动内定位问题
- 如果 HOI4 启动太慢，可考虑用 `-no_intro` 参数跳过开场动画

### 输出物
- 锁定的具体文件和行号（或冲突 mod 名称）

---

## 阶段四：常见错误速查表

### 文件/语法

| 错误信息 | 原因 | 解决方案 |
|---------|------|---------|
| `Unexpected token: XXX` | 关键字拼写错误或使用了不支持的 key | 对照原版文件确认正确拼写（区分大小写） |
| `Expected "="` | 赋值语法错误或缺少等号 | 检查 key = value 格式，确保 = 前后有空格 |
| `Missing closing bracket` | 花括号未闭合 | 用 CWTools 括号匹配功能逐层排查 |
| `Could not find file: "gfx/flags/XXX.tga"` | 旗帜文件缺失或路径错误 | 确认文件名与 Tag 一致，确认扩展名正确 |
| `Could not open file: "localisation/XXX.yml"` | 本地化文件编码错误（非 UTF-8 BOM） | 用 VS Code 重新保存为 UTF-8 with BOM |

### 引用/作用域

| 错误信息 | 原因 | 解决方案 |
|---------|------|---------|
| `Unknown trigger type: XXX` | 触发器名拼写错误或不支持 | 在 references/03-pdxscript.md 查看支持的触发器列表 |
| `Unknown effect type: XXX` | 效果名拼写错误或不支持 | 在 references/03-pdxscript.md 查看支持的效果列表 |
| `Invalid scope for trigger XXX` | 在不支持的作用域中使用触发器 | 检查当前代码块在哪个作用域中（国家/省份/全局） |
| `Could not resolve reference: XXX` | 引用了一个不存在的 id | 检查 id 拼写、确认被引用对象已定义 |
| `Tag XXX is not defined` | 国家 Tag 未定义 | 检查是否有对应的 `history/countries/TAG - xxx.txt` 文件 |

### 启动/崩溃

| 错误/现象 | 原因 | 解决方案 |
|----------|------|---------|
| 游戏启动即崩溃（error code 3221225477） | 文件编码错误 | .txt → UTF-8 without BOM；.yml → UTF-8 with BOM |
| 启动器列表中不显示 mod | .mod 文件格式错误或 path 不正确 | 检查 .mod 文件是否在正确目录，path 是否指向正确的 mod 文件夹 |
| Mod 显示在列表中但勾选后不生效 | mod 文件夹结构错误或 .mod 中 path 与文件夹名不一致 | 检查文件夹名是否与 path 字段一致 |
| 游戏加载卡在"初始化游戏" | replace_path 错误或 common 文件夹结构损坏 | 临时注释掉 replace_path 测试 |
| 启动后 error.log 大量报错 | replace_path 缺失导致双重加载（见 02-mod-structure.md） | 确认是否需要 replace_path |

### 运行时行为异常

| 现象 | 可能原因 | 排查方法 |
|------|---------|---------|
| 国策树不显示 | focus_tree 未正确绑定国家 | 检查 history 文件中的 load_focus_tree 是否指向正确的 focus_tree id |
| 事件不触发 | trigger 条件永远不满足 | 用控制台手动触发事件验证事件本身是否正常 |
| 国家精神不生效 | idea 定义错误或添加时机不对 | 检查 add_ideas 是否在正确的作用域中执行 |
| 文本显示 "missing string" | 对应的本地化条目缺失 | 检查本地化 key 拼写，确认在 .yml 文件中定义了条目 |

---

## 阶段五：无法解决时——需要提供的信息清单

当自行排查仍无法解决、需在社区（贴吧/论坛/Discord/Reddit）求助时，提供以下信息可大幅提高获得有效回复的概率。

### 必须提供

1. **error.log 关键片段**
   - 提供第一个错误的完整行 + 前后 5 行
   - 用代码块格式呈现，不要截图（文本可被搜索和引用）
   - 如果 error.log 很长，用 Pastebin 等工具上传完整文件并附链接

2. **game.log 关键信息**
   - 搜索你的 mod 名称或 Tag，提供相关行
   - 包含游戏启动到崩溃之间的完整日志段

3. **出错相关代码**
   - 提供完整的错误文件和前后关联文件
   - 说明你做了什么修改导致问题出现

4. **Mod 基本信息**
   - .mod 文件内容（尤其是 replace_path 和 dependencies 配置）
   - mod 文件夹结构概览（tree 命令输出或截屏）
   - 支持的 HOI4 版本

### 建议提供

5. **复现步骤**
   - 详细描述你做了什么操作触发了问题
   - 例如："启动游戏 → 选择 1936 剧本 → 选择中国 → 点击第一个国策 → 5 秒后崩溃"

6. **已尝试的排查步骤**
   - 说明你已经做了哪些尝试（缩小范围/二分法/修改了哪些内容），以免社区重复建议

7. **系统环境**
   - 操作系统（Windows/Mac/Linux）及版本
   - HOI4 版本（启动器左下角）
   - 其他已加载的 mod 列表

### 模板

```
**问题描述：**
（一句话概括问题）

**error.log 关键片段：**
```
（粘贴第一个错误及前后 5 行）
```

**相关代码：**
（粘贴出错文件的关键代码段）

**复现步骤：**
1.
2.
3.

**已尝试的方案：**
- [ ] 检查了文件编码
- [ ] 已进行二分法排查
- [ ] 已确认无 mod 冲突
- [ ] 已查阅 references/14-debugging.md

**环境信息：**
- HOI4 版本：1.18.2
- 操作系统：Windows 11
- 其他 mod：（列出或说明只开了一个 mod）
```

---

## 检查清单

### 信息收集
- [ ] 已读取 `logs/error.log` 并提取第一个错误
- [ ] 已确认错误与哪个文件有关（文件路径 + 大致行号）
- [ ] 已检查 `logs/game.log` 是否有额外信息
- [ ] 已确认 -debug 启动参数已开启

### 分类诊断
- [ ] 已判定错误属于哪种类型（编码/语法/路径/作用域/引用/重复/逻辑）
- [ ] 已根据分类执行对应的诊断策略
- [ ] 已用 CWTools 检查括号匹配
- [ ] 已确认文件编码正确（.txt UTF-8 without BOM, .yml UTF-8 with BOM）

### 二分法（如需要）
- [ ] 已备份当前 mod
- [ ] 已测试只开你的 mod（排除冲突）
- [ ] 已通过删除/注释逐步缩小到具体文件和行号
- [ ] 如涉及 mod 冲突，已定位冲突 mod

### 修复与验证
- [ ] 已修复根本原因（非仅仅是症状）
- [ ] 已重新启动游戏验证修复有效
- [ ] error.log 不再出现该错误
- [ ] 修复未引入新问题

### 求助准备（如需要）
- [ ] 已收集 error.log 关键片段
- [ ] 已整理相关代码
- [ ] 已编写复现步骤
- [ ] 已列出已尝试的方案

---

> **相关文件**：`references/14-debugging.md` 提供调试编码和工具的完整参考。`references/02-mod-structure.md` 解释 replace_path 和目录结构规则。`references/03-pdxscript.md` 提供 PDXscript 语法完整参考。
