# 从零创建 Mod

> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03
> **前置要求**: 已安装 Hearts of Iron IV，至少启动过一次游戏（确保 Documents 下生成了 mod 目录）
> **关联参考**: `references/01-env-setup.md` `references/02-mod-structure.md` `references/04-focus-trees.md`
> **关联模板**: `templates/focus-tree.txt`

## 概述

本工作流指导 AI 代理从零创建一个可加载、可玩的最小 HOI4 mod。流程覆盖环境确认、文件创建、测试内容（一个简单国策）和正式命名四个阶段。完成后你将拥有一个通过启动器验证、游戏内功能正常的 mod 骨架。

适用场景：
- 首次创建 HOI4 mod 项目
- 为新国家/新机制搭建独立 mod 容器
- 需要一个干净的基础骨架来承载后续国策树、事件、决议等

前置条件：
- HOI4 已安装（默认路径 `D:/steam/steamapps/common/Hearts of Iron IV`）
- 游戏至少运行过一次（生成 Documents 下的 mod 和 logs 目录）
- VS Code 已安装（如使用 CWTools 语法检查）
- 了解 PDXscript 基本语法（`key = value`, `block = { }`，见 `references/03-pdxscript.md`）

---

## 阶段一：环境确认

目标：确保 HOI4 安装正确、mod 目录存在、开发工具可用。此阶段无产出文件，仅做验证。

### 步骤 1.1：验证 HOI4 游戏安装

确认以下路径存在：

| 检查项 | 预期路径 | 验证命令 (bash) |
|--------|----------|-----------------|
| 游戏根目录 | `D:/steam/steamapps/common/Hearts of Iron IV/` | `ls "D:/steam/steamapps/common/Hearts of Iron IV/hoi4.exe"` |
| 原版国策目录 | `D:/steam/steamapps/common/Hearts of Iron IV/common/national_focus/` | `ls "D:/steam/steamapps/common/Hearts of Iron IV/common/national_focus/germany.txt"` |
| 原版事件目录 | `D:/steam/steamapps/common/Hearts of Iron IV/events/` | `ls "D:/steam/steamapps/common/Hearts of Iron IV/events/"` |
| 原版本地化目录 | `D:/steam/steamapps/common/Hearts of Iron IV/localisation/english/` | `ls "D:/steam/steamapps/common/Hearts of Iron IV/localisation/english/"` |

如果游戏安装在其他路径，请在后续所有步骤中替换 `HOI4_DIR` 变量。

### 步骤 1.2：验证 Mod 目录

确认用户 Documents 下存在 mod 和 logs 目录：

```
# Windows 路径模式
C:\Users\<用户名>\Documents\Paradox Interactive\Hearts of Iron IV\mod\
C:\Users\<用户名>\Documents\Paradox Interactive\Hearts of Iron IV\logs\
```

验证命令（bash）：
```bash
ls "/c/Users/$USER/Documents/Paradox Interactive/Hearts of Iron IV/mod/"
ls "/c/Users/$USER/Documents/Paradox Interactive/Hearts of Iron IV/logs/"
```

如果 mod 目录不存在：至少运行一次 HOI4 游戏并进入主菜单，然后退出。游戏会自动创建该目录。

### 步骤 1.3：验证 VS Code + CWTools（可选但强烈推荐）

1. 确认 VS Code 已安装：`code --version`
2. 确认 CWTools 扩展已安装：在 VS Code 扩展面板搜索 "CWTools"，确认为"已安装"状态
3. 验证 CWTools 激活：用 VS Code 打开任一原版国策文件（如 `germany.txt`），查看是否出现语法高亮和悬停提示

如果 CWTools 未安装：VS Code → 扩展 (Ctrl+Shift+X) → 搜索 "CWTools" → 安装。

### 步骤 1.4：确认 Steam 启动参数

在 Steam 库中右键 Hearts of Iron IV → 属性 → 通用 → 启动选项，确保包含：
```
-debug -crash_data_log
```
- `-debug`：开启控制台（`~` 键）、错误弹窗、详细日志
- `-crash_data_log`：崩溃时生成详细堆栈信息

可选添加 `-no_intro` 跳过开场动画以加快测试迭代。

### 阶段一检查点

- [x] HOI4 游戏安装路径已验证
- [x] Documents/.../mod/ 目录已存在
- [x] VS Code + CWTools 已确认可用（或已记录不用）
- [x] Steam 启动参数已包含 `-debug`

---

## 阶段二：创建 Mod 文件结构

目标：创建 `.mod` 描述文件和 mod 文件夹，含一个最小测试国策来验证加载链路。

### 步骤 2.1：确定 Mod 命名

在开始创建文件前，确定以下命名参数：

| 参数 | 说明 | 示例值 |
|------|------|--------|
| `MOD_NAME` | 启动器中显示的 mod 名称（支持中文） | `测试模组` |
| `MOD_FOLDER` | mod 文件夹名称（小写+下划线，无空格） | `test_mod` |
| `MOD_PACKAGE` | 内部 ID 前缀（小写+下划线，用于所有自定义 ID） | `test_mod` |
| `MOD_TAG` | 测试国家 Tag（3 个大写字母，不与原版冲突） | `TST` |

命名规范：
- `MOD_FOLDER` 必须使用小写字母、数字和下划线，禁止空格和特殊字符
- `MOD_TAG` 必须是 3 个大写字母，推荐使用不与任何原版或主流 mod 冲突的组合。快速检查：`grep -r "MOD_TAG" "D:/steam/steamapps/common/Hearts of Iron IV/history/countries/" | head -5`（应无输出）
- `MOD_PACKAGE` 作为所有自定义 ID 的前缀，避免与其他 mod 冲突

### 步骤 2.2：创建 .mod 描述文件

在 `Documents\Paradox Interactive\Hearts of Iron IV\mod\` 下创建 `<MOD_FOLDER>.mod`：

```
name = "<MOD_NAME>"
path = "mod/<MOD_FOLDER>"
supported_version = "1.18.*"
tags = { "Gameplay" "Alternative History" }
```

字段说明：
- `name`：启动器中显示的 mod 名称，可与后续阶段更改
- `path`：相对于 `Documents\Paradox Interactive\Hearts of Iron IV\` 的 mod 文件夹路径。**必须**以 `mod/` 开头
- `supported_version`：支持的游戏版本，`"1.18.*"` 覆盖 1.18.x 所有小版本
- `tags`：Steam Workshop 分类标签。常用标签包括 `"Gameplay"` `"Alternative History"` `"National Focuses"` `"Events"` `"Graphics"` `"Total Conversion"` 等

暂时不添加 `replace_path`、`dependencies` 等可选字段——它们将在后续扩展中按需加入。

### 步骤 2.3：创建 Mod 文件夹和目录结构

在 `Documents\Paradox Interactive\Hearts of Iron IV\mod\` 下创建以下目录树：

```
<MOD_FOLDER>/
├── common/
│   └── national_focus/      # 国策树目录
├── localisation/
│   └── english/             # 英文（必须，游戏以此为默认）
└── history/
    └── countries/           # 国家历史目录
```

创建命令（bash）：
```bash
MOD_DIR="/c/Users/$USER/Documents/Paradox Interactive/Hearts of Iron IV/mod/<MOD_FOLDER>"
mkdir -p "$MOD_DIR/common/national_focus"
mkdir -p "$MOD_DIR/localisation/english"
mkdir -p "$MOD_DIR/history/countries"
```

注意：
- 路径名必须与游戏内部路径完全一致（区分大小写——虽 Windows 不敏感，但为兼容 Mac/Linux 发布，始终使用与原版一致的大小写）
- 只创建当前阶段需要的目录。后续添加事件时创建 `events/`，添加决议时创建 `common/decisions/`，以此类推
- 不要在 Steam 安装目录下创建或修改任何文件

### 步骤 2.4：创建最小测试国策

在 `common/national_focus/` 下创建 `<MOD_PACKAGE>_test_focus.txt`。这个国策仅用于验证 mod 加载链路，后续阶段将被正式内容替换。

使用 `templates/focus-tree.txt` 模板，生成一个最小 focus_tree：

```
focus_tree = {
    id = <MOD_PACKAGE>_test_focus_tree

    country = {
        factor = 0

        modifier = {
            add = 10
            tag = <MOD_TAG>
        }
    }

    focus = {
        id = <MOD_PACKAGE>_test_focus
        icon = GFX_goal_generic_construct_civilian

        x = 0
        y = 0

        relative_position_id = <MOD_PACKAGE>_test_focus

        cost = 10

        completion_reward = {
            add_political_power = 50

            # 标记 mod 加载成功——测试用
            set_country_flag = <MOD_PACKAGE>_mod_loaded_flag
        }
    }
}
```

关键点：
- `id` 在 focus_tree 层级唯一标识国策树，在单个 focus 层级唯一标识国策
- `country` 块决定哪个国家加载此国策树。`factor = 0` + 一个带 `tag = <MOD_TAG>` 的 `modifier` 确保只有目标国家获得
- `x = 0, y = 0` 将国策放在国策树视口原点（左上角）
- `cost = 10` 即 70 天（10 × 7 天），足够短的测试时间
- `icon` 使用 vanilla 通用图标 `GFX_goal_generic_construct_civilian`——无需制作自定义图标

### 步骤 2.5：创建测试国家历史文件

在 `history/countries/` 下创建 `<MOD_TAG> - Test Nation.txt`：

```
capital = 1

set_politics = {
    ruling_party = neutrality
    last_election = "1936.1.1"
    election_frequency = 48
    elections_allowed = no
}

set_research_slots = 4

set_convoys = 100

set_stability = 0.6
set_war_support = 0.6

oob = "<MOD_TAG>_1936"
```

如果不需要完整的部队模板 (OOB)，可以简化为：

```
capital = 1
set_research_slots = 3
set_stability = 0.6
set_war_support = 0.6

set_technology = {
    infantry_weapons = 1
    basic_machine_tools = 1
    construction1 = 1
    electronic_mechanical_engineering = 1
}

load_focus_tree = <MOD_PACKAGE>_test_focus_tree
```

关键点：
- `capital = 1` 将首都设为省份 ID 1（原版德国首都柏林所在地——仅用于测试）
- `load_focus_tree` 行将国策树关联到此国家，**必须**与步骤 2.4 中 focus_tree 的 `id` 一致
- 如果没有 `load_focus_tree`，国家将没有国策可用

### 步骤 2.6：创建最小本地化文件

在 `localisation/english/` 下创建 `<MOD_PACKAGE>_l_english.yml`：

注意：`.yml` 本地化文件必须使用 **UTF-8 with BOM** 编码。

```yml
l_english:
 <MOD_PACKAGE>_test_focus:0 "Test Focus"
 <MOD_PACKAGE>_test_focus_desc:0 "A test focus to verify mod loading."
 <MOD_PACKAGE>_test_focus_tree:0 "Test Nation Focus Tree"
 <MOD_TAG>:0 "Test Nation"
 <MOD_TAG>_DEF:0 "the Test Nation"
 <MOD_TAG>_ADJ:0 "Testian"
```

本地化键命名规范：
- 国策名称：`<focus_id>:0 "<Display Name>"`
- 国策描述：`<focus_id>_desc:0 "<Description>"`
- 国家名称：`<TAG>:0 "<Country Name>"`
- 国家从属：`<TAG>_DEF:0 "the <Country Name>"`（用于 "ally of the <Name>" 等句式）
- 国家形容词：`<TAG>_ADJ:0 "<Adjective>"`（用于 "<Adjective> Army" 等句式）
- `:0` 后缀是本地化版本标记（一般用 0）

### 阶段二检查点

- [ ] `.mod` 文件已创建，`name` `path` `supported_version` `tags` 字段正确
- [ ] mod 文件夹已创建，包含 `common/national_focus/` `history/countries/` `localisation/english/` 三个子目录
- [ ] 最小测试国策文件已创建（含一个 focus_tree + 一个 focus）
- [ ] 国家历史文件已创建（含 `load_focus_tree` 指向测试国策树）
- [ ] 本地化文件已创建（含国策名、国家名的英文条目），编码为 UTF-8 BOM

---

## 阶段三：启动游戏测试

目标：验证 mod 在启动器中可见、可加载、国策可显示和执行。这是质量关口——未通过此阶段不得进入正式化。

### 步骤 3.1：启动器验证

1. 启动 Hearts of Iron IV
2. 在启动器中点击"播放集" → "添加更多 Mod"
3. 搜索 `<MOD_NAME>`
4. 勾选该 mod，点击"添加到播放集"

预期结果：
- Mod 出现在播放集列表中
- 无红色感叹号（表示 path 正确，文件夹存在）

常见问题：
- **找不到 Mod**：检查 `.mod` 文件名与 `path` 值的最后一部分是否一致；检查 `.mod` 文件是否在正确的 `mod/` 目录下
- **红色感叹号**：`path` 路径写错或 mod 文件夹不存在。验证 `path` 以 `mod/` 开头，确认文件夹确实存在于该路径

### 步骤 3.2：加载验证

1. 在启动器中，确保 mod 已勾选
2. 选择任意官方剧本（如 1936 年德国）
3. 启动游戏

预期结果：
- 游戏正常进入主菜单
- 选择剧本后正常加载到地图界面
- 无红色错误弹窗

如果出现红色错误弹窗：
1. 记录错误信息的确切文本
2. 打开 `Documents\Paradox Interactive\Hearts of Iron IV\logs\error.log`
3. 搜索与你的 `MOD_FOLDER` 或 `MOD_PACKAGE` 相关的条目
4. 根据错误类型修复（参见下方"常见错误与修复"）

### 步骤 3.3：国策验证

1. 在游戏中打开控制台（`~` 键）
2. 输入 `tdebug` 开启调试信息显示
3. 输入 `tag <MOD_TAG>` 切换到测试国家
4. 打开国策树界面
5. 点击 `<MOD_PACKAGE>_test_focus` 国策
6. 等待 10 个游戏天（70 天 / ~3 秒在速度 5）

预期结果：
- 国策树界面显示一个国策节点，名称为 "Test Focus"
- 点击后开始执行（绿色进度条）
- 完成后获得 50 政治点数
- 未出现 Error 弹窗或 error.log 新增相关错误

### 步骤 3.4：常见错误与修复

| 错误现象 | error.log 典型条目 | 根本原因 | 修复方法 |
|----------|--------------------|----------|----------|
| 启动器找不到 mod | 无（启动器级别错误） | `.mod` 文件 path 不一致或缺失 | 检查 `.mod` 文件名与 path 值一致性 |
| 游戏加载时红字错误 | `"<ID>" is not a valid focus tree` | 国策树文件语法错误（括号不匹配、字段名错误） | 用 CWTools 检查括号，逐块核对字段 |
| 切换 tag 后无国策树 | `"<ID>" is not a valid ...` 或 `load_focus_tree` 错误 | 国家历史文件未正确引用 focus_tree id，或 id 拼写不一致 | 检查 `load_focus_tree` 与 focus_tree 的 `id` 是否完全一致（包括大小写） |
| 国策名称空白 | `"<key>" is not a valid localisation key` | 本地化文件编码错误（非 UTF-8 BOM）或 key 拼写不一致 | 检查 `.yml` 文件为 UTF-8 with BOM；检查 key 拼写 |
| 国策无法点击 | `focus has no available trigger` 或 `prerequisite not met` | 缺少 `available` 条件或前置条件不满足 | 检查 `available = {}` 块或 `prerequisite` 字段 |
| 游戏崩溃 (CTD) | error.log 末尾无明确错误 | 编码错误（.txt 含 BOM）或非法字符 | 检查所有 `.txt` 文件为 UTF-8 without BOM；移除特殊字符 |

### 阶段三检查点

- [ ] Mod 在启动器中可见并可勾选
- [ ] 游戏加载无错误弹窗
- [ ] error.log 中无与 `<MOD_PACKAGE>` 相关的条目（允许无关的原版 warning）
- [ ] 国策树界面正常显示测试国策
- [ ] 国策名称正确显示（非空白）
- [ ] 国策可点击并完成执行
- [ ] 完成后获得预期奖励（50 PP）

---

## 阶段四：正式化

目标：将测试命名替换为正式命名，准备承载后续内容。

### 步骤 4.1：更新正式命名

确定最终 mod 名称和内部标识。按以下映射全局替换：

| 测试值 | 替换为 | 出现位置 |
|--------|--------|----------|
| `<测试 MOD_NAME>` | 正式的 Mod 显示名称 | `.mod` 文件的 `name` 字段 |
| `<测试 MOD_FOLDER>` | 正式的 mod 文件夹名 | `.mod` 文件的 `path` 字段；磁盘上的文件夹名 |
| `<测试 MOD_PACKAGE>` | 正式的 package 前缀 | 所有自定义 ID 前缀（国策、事件、决议、flags 等） |
| `<测试 MOD_TAG>` | 正式的目标国家 Tag | 国策树 modifier、国家历史文件 Tag |

如果目标国家尚不存在：保留测试 MOD_TAG 作为占位，待 `workflows/add-country.md` 完成后再替换。

### 步骤 4.2：移除测试标记

删除或注释掉测试内容：
1. 移除 `set_country_flag = <MOD_PACKAGE>_mod_loaded_flag`（或保留为调试用——标记为 `# DEBUG` 注释）
2. 将国策 `cost = 10` 调整为正式值（一般国策为 35-70 天，即 `cost = 5` 到 `cost = 10`）
3. 将国策 `icon` 替换为自定义 GFX 条目（如果已有图标）

### 步骤 4.3：最终验证

重复阶段三的所有验证步骤，确认重命名后一切正常：
1. 启动器可见正式名称
2. 游戏加载正常
3. 国策显示和执行正常

### 步骤 4.4：版本控制初始化（强烈推荐）

在 mod 文件夹根目录初始化 Git 仓库：

```bash
cd "/c/Users/$USER/Documents/Paradox Interactive/Hearts of Iron IV/mod/<MOD_FOLDER>"
git init
git add .
git commit -m "Initial mod skeleton: verified loading chain"
```

后续每完成一个功能模块（国策树、事件链、决议等）提交一次，便于回溯和协作。

如需 `.gitignore` 模板：
```
# HOI4 mod .gitignore
*.dds
*.tga
*.png
*.psd
Thumbs.db
desktop.ini
```

### 阶段四检查点

- [ ] 测试命名已全局替换为正式命名
- [ ] 测试标记已清理或标注为 DEBUG
- [ ] 最终验证通过（重复阶段三全部检查项）
- [ ] Git 仓库已初始化（可选）

---

## 检查清单（总）

使用以下清单确认 mod 骨架创建完成：

### 文件存在性
- [ ] `.mod` 描述文件存在于 `Documents/.../mod/`
- [ ] mod 文件夹包含 `common/national_focus/` 子目录
- [ ] mod 文件夹包含 `history/countries/` 子目录
- [ ] mod 文件夹包含 `localisation/english/` 子目录
- [ ] 国策树文件存在于 `common/national_focus/`
- [ ] 国家历史文件存在于 `history/countries/`
- [ ] 本地化 `.yml` 文件存在于 `localisation/english/`

### 内容正确性
- [ ] `.mod` 文件：`name` `path` `supported_version` `tags` 四个必填字段完整
- [ ] `path` 以 `mod/` 开头，指向正确的文件夹名
- [ ] focus_tree `id` 与 `load_focus_tree` 值完全一致
- [ ] 国策树 `country` 块中 `tag = <MOD_TAG>` 正确
- [ ] 本地化 key 与代码中引用的 ID 完全一致
- [ ] 所有 `.txt` 文件为 UTF-8 without BOM
- [ ] 所有 `.yml` 文件为 UTF-8 with BOM

### 运行时验证
- [ ] 启动器中可见并可勾选 mod
- [ ] 无加载错误（error.log 无相关条目）
- [ ] 国策树可显示
- [ ] 国策可点击并可执行完成
- [ ] 效果执行正确（获取 PP、flag 等）

---

> **下一步**: Mod 骨架就绪后，根据需求选择后续工作流：
> - 添加国策树 → `workflows/add-focus-tree.md`
> - 添加事件链 → `workflows/add-event-chain.md`
> - 添加新国家 → `workflows/add-country.md`
> - 添加决议 → 参考 `references/06-decisions.md` + `templates/decision.txt`
> - 添加国家精神 → 参考 `references/07-ideas.md` + `templates/idea.txt`
