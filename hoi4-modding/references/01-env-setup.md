> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

搭建 HOI4 mod 开发环境，包括 VS Code + CWTools 扩展、HOI4 路径配置、启动参数和辅助工具安装。完成本章后即可开始编写和测试 mod。

## 开发环境配置

### 1. VS Code + CWTools 扩展

VS Code 是目前 HOI4 modding 的标准编辑器，配合 CWTools 扩展提供语法高亮、括号匹配、错误检查、自动补全和工具提示。

安装步骤：
1. 安装 VS Code（code.visualstudio.com）
2. 在扩展市场搜索 **CWTools** 并安装
3. 打开任意 .txt mod 文件，CWTools 自动激活

验证：打开一个原版国策文件（如 `common/national_focus/germany.txt`），应看到：
- 语法高亮（key = value 不同颜色）
- 括号匹配（点击 `{` 高亮对应的 `}`）
- 悬停提示（鼠标悬停在字段上显示说明）

### 2. HOI4 安装路径

| 路径类型 | Windows |
|---------|---------|
| 游戏安装目录 | `D:\steam\steamapps\common\Hearts of Iron IV` |
| Mod 存放目录 | `C:\Users\<用户名>\Documents\Paradox Interactive\Hearts of Iron IV\mod\` |
| 日志文件目录 | `C:\Users\<用户名>\Documents\Paradox Interactive\Hearts of Iron IV\logs\` |

### 3. 启动参数

在 Steam 库中右键 Hearts of Iron IV → 属性 → 通用 → 启动选项：

| 参数 | 作用 |
|------|------|
| `-debug` | 开启调试模式：启用控制台（~ 键）、错误弹窗、详细日志 |
| `-crash_data_log` | 崩溃时生成详细堆栈日志 |
| `-no_intro` | 跳过开场动画，加快测试迭代 |

推荐组合：`-debug -crash_data_log`

### 4. 辅助工具

| 工具 | 用途 | 获取方式 |
|------|------|---------|
| **HOI4 Content Maker** | 可视化编辑国策树/事件/决议（拖拽式 GUI） | GitHub: MillenniumDawn/focus-tree-creation-tool |
| **WinMerge** | 对比文件差异，更新 mod 时合并 vanilla 变化 | winmerge.org |
| **HOI4 Validator** | 语法验证工具 | GitHub 社区 |
| **Git** | 版本控制 | git-scm.com |
| **GIMP/Photoshop** | 编辑 .dds / .tga 图像资源 | gimp.org |

## 实战示例

### 示例 1：验证 CWTools 工作正常

1. 创建测试文件 `test_mod/common/test.txt`，写入以下不闭合括号的代码：
```
focus = {
    id = test_focus
    icon = GFX_test
```
2. CWTools 应在编辑器底部 PROBLEMS 面板标注红色错误，第 3 行提示缺少闭合 `}`。
3. 补上 `}` 后错误消失。

### 示例 2：验证 debug 模式

1. 在 Steam 启动选项中添加 `-debug`
2. 启动 HOI4
3. 进入主菜单后按 `~` 键（波浪号），应出现控制台输入框
4. 输入 `tdebug` 回车——鼠标悬停在地图省份上会显示省份 ID 等调试信息

## 常见陷阱

1. **CWTools 不识别 mod 文件** — 确保在 VS Code 中打开了 mod 文件夹（或父目录）作为工作区根目录。如果使用 File → Open Folder 打开 mod 文件夹本身，CWTools 应识别 `.mod` 文件并激活。
2. **启动参数不生效** — 确认参数前有空格分隔，且没有中文引号。正确：`-debug -no_intro`。错误：`-debug-no_intro`。
3. **mod 目录不存在** — 至少运行一次 HOI4 后，Documents 下才会自动创建 `Paradox Interactive\Hearts of Iron IV\mod\` 文件夹。
4. **VS Code 其他扩展冲突** — 某些通用语法扩展可能与 CWTools 冲突。如果语法高亮异常，尝试禁用其他 Paradox 相关扩展。
5. **编码问题导致文件无法读取** — 确保 .txt 文件用 UTF-8 无 BOM 编码保存（VS Code 右下角状态栏可查看和更改编码）。

## FAQ

**Q: CWTools 提示的错误一定是真正的错误吗？**
A: 不一定。CWTools 的规则库可能滞后于游戏更新（1.18 较新），加载成功（启动器无报错、游戏内功能正常）即为最终标准。

**Q: 必须用 VS Code 吗？**
A: 不必须，但强烈推荐。Notepad++ 设语言为 Perl 可基本高亮，Sublime Text 也有类似扩展，但 CWTools 的自动补全和错误检查是独有的。

**Q: 如何知道 CWTools 需要更新？**
A: 检查扩展版本——CWTools 维护者通常在游戏大版本更新后 1-2 周更新规则库。如果大量 vanilla 文件报错，说明规则库过期。

## 检查清单

- [ ] VS Code 已安装
- [ ] CWTools 扩展已安装并激活
- [ ] HOI4 安装路径已确认（`D:\steam\steamapps\common\Hearts of Iron IV`）
- [ ] Mod 目录路径已确认（Documents 下）
- [ ] `-debug` 启动参数已设置
- [ ] 控制台可正常打开（`~` 键）

> **相关文件**: Mod 项目结构（02-mod-structure.md）告诉你创建什么文件放在哪里。
