> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

HOI4 modding skill 的**核心知识是平台无关的**——国策树语法、事件结构、决议系统在所有 AI 编程平台上完全一致。差异仅在于各平台提供的**工具能力**（文件读写、搜索、终端执行），这决定了你在该平台上能多高效地完成 modding 操作。本章提供 6 个主流 AI 编程平台的工具映射、安装路径和降级方案。

## 平台无关的核心知识

以下内容在所有平台上完全相同——你可以在任何平台学习这些章节：

| 章节 | 内容 | 平台依赖 |
|------|------|---------|
| 02-mod-structure | .mod 文件结构、文件夹镜像规则 | 无 |
| 03-pdxscript | PDXscript 语法、作用域、触发器、效果 | 无 |
| 04-focus-trees | 国策树：focus = { ... } 完整语法 | 无 |
| 05-events | 事件：country_event、news_event | 无 |
| 06-decisions | 决议：decision = { ... } 及分类 | 无 |
| 07-ideas | 国家精神、设计商、顾问 | 无 |
| 08-history | 国家起始设定、省份数据 | 无 |
| 09-technologies | 科技树定义和分类 | 无 |
| 10-equipment | 装备、部队、变体 | 无 |
| 11-map | 地图修改（高级） | 无 |
| 12-localisation | 本地化 .yml 文件 | 无（需 UTF-8 BOM 编辑器） |
| 13-graphics | .dds/.tga 图像、图标、旗帜 | 需图像编辑软件 |
| 14-debugging | error.log 解读、控制台调试 | 无 |
| 15-workshop | Steam Workshop 发布和更新 | 需 Steam 启动器 |

**仅 01-env-setup（环境搭建）和本章（16-platforms）是平台相关的。**

## 平台 1：Claude Code（原生支持）

Claude Code 是 hoi4-modding skill 的**主要开发和测试平台**，拥有最完整的工具链。

### 安装路径

```
~/.claude/skills/hoi4-modding/
```

### 可用工具

| 工具 | 用途 | HOI4 Modding 场景 |
|------|------|------------------|
| **Read** | 读取文件内容 | 查阅 vanilla 文件语法参考、查阅 mod 文件、读取 error.log |
| **Write** | 创建/覆盖文件 | 创建新的国策树文件、事件文件 |
| **Edit** | 精确字符串替换 | 修改单个国策的某个字段、批量修正拼写 |
| **Bash** | 执行终端命令 | 复制 vanilla 文件到 mod 目录、启动 VS Code |
| **Grep** | 内容搜索（ripgrep） | 搜索 vanilla 中的关键字用法、搜索 error.log 中的错误 |
| **Glob** | 文件名匹配 | 查找所有 .txt mod 文件、列出某个目录下的所有国策 |
| **Skill** | 调用子技能 | 调用 rimworld-modding skill 参考跨游戏经验 |

### 典型工作流

```
1. Grep: 搜索 vanilla 中 "has_war_with" 的所有用法 → 学习语法
2. Glob: 列出 common/national_focus/ 下的所有文件 → 找到要修改的目标
3. Read: 读取目标文件 → 理解原有结构
4. Edit: 精确修改/添加国策节点
5. Bash: 启动 HOI4 -debug 测试
6. Grep: 在 error.log 中搜索 mod 文件名 → 定位错误行
```

### 注意事项

- 直接在 mod 目录操作（`D:\steam\steamapps\common\Hearts of Iron IV\` 的 vanilla 文件只读不写）
- 大文件（如 defines）使用 Read 的 offset/limit 分段读取
- 跨平台路径：Linux/Mac 上 HOI4 路径不同，但工具用法一致

## 平台 2：Copilot CLI

### 安装路径

```
~/.copilot/skills/hoi4-modding/
```

### 可用工具

| 工具 | 对应 Claude Code | 说明 |
|------|-----------------|------|
| **view** | Read | 查看文件内容 |
| **create** | Write | 创建新文件 |
| **edit** | Edit | 修改文件 |
| **grep** | Grep | 内容搜索 |
| **glob** | Glob | 文件名匹配 |
| **bash** | Bash | 终端命令 |
| **task** | Skill | 调用子任务 |
| **skill** | - | 加载领域知识 |

### 注意事项

- Copilot CLI 的 `skill` 加载机制与 Claude Code 不同——确保 skill 文件已正确放置在 `~/.copilot/skills/hoi4-modding/` 下
- `view` 默认分页显示，大文件可能需要 `| cat` 管道
- 如果 `grep` 不可用，降级用 `bash` 调 `grep` 命令

## 平台 3：Codex（OpenAI）

### 安装路径

```
~/.codex/skills/hoi4-modding/
```

### 可用工具

| 工具 | 对应 Claude Code | 说明 |
|------|-----------------|------|
| **native files** | Read/Write/Edit | 内置文件系统访问 |
| **shell** | Bash | 终端命令 |
| **spawn_agent** | Skill | 启子 Agent 并行处理 |
| **skill** | - | 加载领域知识 |

### 注意事项

- Codex 的 `native files` 可能不如 Claude Code 的 Read/Edit 精确——文件编辑后需验证完整性
- `spawn_agent` 适合并行任务：同时创建国策树和对应事件
- Codex 的 PDXscript 理解能力取决于模型版本——推荐使用最新的 GPT-5 或 o3 系列
- 遇到语法问题可先让 Codex 搜索 vanilla 参考文件，再生成代码

## 平台 4：Gemini CLI

### 安装路径

```
~/.gemini/skills/
```

### 可用工具

Gemini CLI 通过 **MCP（Model Context Protocol）** 集成文件工具：

| 能力 | 说明 | HOI4 Modding 场景 |
|------|------|------------------|
| **MCP filesystem** | 通过 MCP 服务器访问文件系统 | 读写 mod 文件 |
| **MCP tools** | 可配置的 MCP 工具集 | grep、glob 等需要额外配置 MCP 服务器实现 |
| **Built-in shell** | 终端命令执行 | 对比文件差异、启动游戏测试 |

### 关键 MCP 服务器配置建议

```
# gemini MCP 配置示例（在 Gemini CLI 的 mcp_config.json 中）
{
    "mcpServers": {
        "filesystem": {
            "command": "npx",
            "args": ["-y", "@anthropic/mcp-server-filesystem", "/path/to/mod"]
        },
        "ripgrep": {
            "command": "npx",
            "args": ["-y", "mcp-server-ripgrep", "--path", "/path/to/game"]
        }
    }
}
```

### 注意事项

- MCP 配置是 Gemini CLI 使用此 skill 的**前提条件**——未配置时将只能使用 shell 命令
- 没有 MCP 时，降级使用 `shell` + 标准 Unix 工具（`grep`, `find`, `cat`）
- Gemini 的上下文窗口较大，可一次性读取较多文件，适合「通读 vanilla 目录学习模式」

## 平台 5：Cursor / Windsurf（IDE 集成）

### 安装路径

Cursor 和 Windsurf 不通过 `~/.claude/skills/` 路径加载 skill，而是通过**项目级知识库引用**。

### 使用方式

**方式 A：作为参考文档加载**

1. 将 hoi4-modding skill 的 reference 文件放入项目 `.cursor/rules/` 或 `.windsurf/rules/` 目录
2. 在设置中注册为 always-applied rules
3. AI 助手在对话中自动引用这些规则

**方式 B：手动引用**

在对话中手动指定：
```
Please follow the HOI4 modding reference at:
/path/to/hoi4-modding/references/03-pdxscript.md
```

### 工具能力

| 工具 | 对应 Claude Code | 说明 |
|------|-----------------|------|
| **内置编辑器** | Read/Write/Edit | 直接在 IDE 中编辑文件 |
| **终端** | Bash | 内置终端 |
| **搜索** | Grep/Glob | IDE 内搜索（vs 项目级搜索） |

### 关键限制：无 MCP

Cursor 和 Windsurf **不支持 MCP**。但这不是 blocker——核心降级方案如下。

**降级方案（所有无 MCP 平台通用）**：

直接用 shell 调用标准 Unix 工具，在 vanilla 目录搜索：

```bash
# 搜索 vanilla 中 has_war_with 的用法
grep -r "has_war_with" "D:/steam/steamapps/common/Hearts of Iron IV/common/" | head -20

# 查找所有国策文件
find "D:/steam/steamapps/common/Hearts of Iron IV/common/national_focus/" -name "*.txt" | head -20

# 读取某个文件
cat "D:/steam/steamapps/common/Hearts of Iron IV/common/ideas/zzz_vanilla_ideas.txt" | head -50

# 搜索 error.log 中的错误
grep "my_mod" "C:/Users/$(whoami)/Documents/Paradox Interactive/Hearts of Iron IV/logs/error.log"
```

### 注意事项

- Cursor 的「codebase indexing」可用于索引 vanilla 文件，提升搜索效率
- Windsurf 的「Cascade」模式更适合多文件编辑任务
- 两者都支持 VS Code 终端——可直接使用上面的降级命令

## 平台 6：通用降级方案

所有平台的最小公分母方案——只要有**终端访问**就可以使用。

### 核心原则

在 vanilla 安装目录搜索和学习，在 mod 目录创作：

```
# Vanilla: 只读搜索、学习参考
D:/steam/steamapps/common/Hearts of Iron IV/

# Mod: 读写创作
C:/Users/<用户名>/Documents/Paradox Interactive/Hearts of Iron IV/mod/<YourMod>/
```

### 必备 Unix 命令速查

| 操作 | Windows (bash) | 说明 |
|------|---------------|------|
| 搜索内容 | `grep -r "关键词" 路径/` | 递归搜索，`-i` 忽略大小写 |
| 查找文件 | `find 路径/ -name "*.txt"` | 按文件名查找 |
| 读取文件 | `cat 文件路径` | 或 `head -50` 只读前 50 行 |
| 比较文件 | `diff file1 file2` | 或 `diff -u` 输出统一格式 |
| 统计行数 | `wc -l 文件路径` | 快速了解文件大小 |
| 创建目录 | `mkdir -p 路径` | `-p` 自动创建父目录 |
| 复制文件 | `cp 源文件 目标路径` | 从 vanilla 复制模板到 mod |

### 注意事项

- Windows 用户建议使用 Git Bash（随 Git 安装）或 WSL，而不是 cmd/PowerShell
- 路径中的空格需要转义或用引号包裹：`"D:/steam/steamapps/common/Hearts of Iron IV/"`
- 大型 vanilla 目录（如 `history/states/` 含数百个文件）使用 `head` 限制输出，避免终端被刷屏

## 实战示例

### 示例 1：在任何平台添加一个新国策

以「为德国添加一个叫 『Rhineland 再武装』的国策」为例，展示平台无关的工作流。

**步骤 1：查找 vanilla 参考（搜索）**

```bash
grep -r "focus =" "D:/steam/steamapps/common/Hearts of Iron IV/common/national_focus/germany.txt" | head -5
```

输出示例（展示了 vanilla 中国策的声明格式）——AI 可据此生成符合 vanilla 风格的代码。

**步骤 2：确认文件结构（Glob/查找）**

```bash
ls "D:/steam/steamapps/common/Hearts of Iron IV/common/national_focus/"
```

**步骤 3：读取目标位置（Read/cat）**

```bash
cat "D:/steam/steamapps/common/Hearts of Iron IV/common/national_focus/germany.txt" | head -30
```

**步骤 4：编写代码**

在 mod 中创建 `common/national_focus/my_germany_focus.txt`，添加新国策代码（代码本身与平台无关）。

**步骤 5：测试**

```bash
# 复制 mod 到 Documents（如果是直接在 Steam 目录开发的）
# 启动 HOI4 -debug，检查 error.log
grep "error" "C:/Users/$(whoami)/Documents/Paradox Interactive/Hearts of Iron IV/logs/error.log"
```

### 示例 2：跨平台协作——一个人用 Claude Code，另一个人用 Gemini CLI

**场景**：Alice 用 Claude Code（Windows），Bob 用 Gemini CLI（Mac）。他们合作开发一个 mod。

**协作策略**：

1. **共享 Git 仓库**：mod 文件放在 Git 中，核心知识一致——两人看到的国策语法、事件结构完全相同
2. **不同工具，相同流程**：
   - Alice：`Grep → Read → Edit → Bash (-debug)`
   - Bob：`MCP ripgrep → MCP filesystem read → MCP filesystem edit → shell (-debug)`
3. **冲突点处理**：
   - 路径差异：`.gitignore` 中统一用相对路径，不提交平台绝对路径
   - 编码：都用 UTF-8 无 BOM，通过 Git 属性 `*.txt text working-tree-encoding=UTF-8` 确保
4. **知识同步**：两人使用同一套 reference 文件（本 skill），遇到 modding 问题查阅同一章节

## 常见陷阱

1. **在 vanilla 目录下直接改文件** — 这是所有平台的头号陷阱。Steam 更新 HOI4 时会覆盖你的修改。始终在 `Documents\Paradox Interactive\Hearts of Iron IV\mod\` 下操作。

2. **路径分隔符混乱** — Windows 用 `\`，Linux/Mac 用 `/`。在脚本和命令中统一使用 `/`（HOI4 引擎内部也使用 `/`）。MCP 工具和 Git 都接受 `/` 作为跨平台路径分隔符。

3. **Gemini CLI 未配置 MCP 就尝试使用** — 导致所有文件操作失败。首次使用时，花 5 分钟配置 `mcp_config.json`（见上方平台 4 的配置示例），之后效率和 Claude Code 基本持平。

4. **Cursor/Windsurf 用户误以为 MCP 是必需的** — Cursor/Windsurf 的降级方案（shell + grep/find/cat）完全够用。核心工作流：在 vanilla 目录搜索（shell）→ 在 mod 目录编辑（IDE 内置编辑器）→ 测试（shell 启动游戏）。

5. **跨平台团队中编码不一致** — Mac/Linux 用户保存 UTF-8 文件默认**不带 BOM**；Windows 用户用记事本保存会**自动加 BOM**，可能导致 HOI4 读取本地化文件失败。统一使用 VS Code，设置 `"files.encoding": "utf8"` 和 `"files.insertFinalNewline": true`。

## FAQ

**Q: 哪个平台最适合 HOI4 modding？**
A: Claude Code（原生 Read/Write/Edit/Grep/Glob 全部可用，hoi4-modding skill 在此平台上开发和测试）。其次推荐 Gemini CLI（MCP 配置后能力接近 Claude Code）。Coder/Windsurf 适合习惯 IDE 界面的开发者，但需要额外降级处理。

**Q: 我只有 Cursor，没有终端访问权限怎么办？**
A: Cursor 自带内置终端（Ctrl+\` 或 Cmd+\`），可以直接使用降级方案中的 shell 命令。如果终端被禁用，联系管理员开启。

**Q: 在不同平台间切换时，需要重新学习 modding 知识吗？**
A: **不需要**。所有平台的核心 PDXscript 语法、HOI4 modding 概念完全一致。唯一需要调整的是工具名称和调用方式（见各平台工具映射表）。

**Q: 如何判断当前平台的工具是否支持某个操作？**
A: 对照各平台的工具映射表。如果映射表中没有对应的工具，使用「通用降级方案」中的 shell 命令。例如：未知平台不支持 `Glob` → 用 `find 路径/ -name "*.txt"` 替代。

**Q: MCP 服务器需要自己写代码吗？**
A: 不需要。社区已有现成的 MCP 文件系统服务器（`@anthropic/mcp-server-filesystem`）和 ripgrep 服务器（`mcp-server-ripgrep`）。通过 npm 安装即可，配置只需几行 JSON。

## 检查清单

- [ ] 确认当前使用的 AI 编程平台（Claude Code / Copilot CLI / Codex / Gemini CLI / Cursor / Windsurf）
- [ ] Skill 文件已放置到对应平台的安装路径下
- [ ] 文件读写工具可用（Read/Write/Edit 或等效工具）
- [ ] 搜索工具可用（Grep/Glob 或降级为 shell grep/find）
- [ ] 终端/Bash 可用（用于测试和批量操作）
- [ ] 确认 vanilla 安装路径：`D:/steam/steamapps/common/Hearts of Iron IV/`
- [ ] 确认 mod 目录路径：`Documents/Paradox Interactive/Hearts of Iron IV/mod/`
- [ ] （Gemini CLI 用户）MCP 配置文件已设置
- [ ] （Cursor/Windsurf 用户）降级 shell 方案已验证（`grep -r "test" vanilla_path/` 可正常输出）
- [ ] 跨平台时编码统一为 UTF-8 无 BOM

> **相关文件**: 环境搭建（01-env-setup.md）配置 VS Code + CWTools, 这对所有平台都是通用前提。Mod 项目结构（02-mod-structure.md）告诉你文件放在哪里——文件操作路径在所有平台一致。
