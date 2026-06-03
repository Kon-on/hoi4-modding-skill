> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

每个 HOI4 mod 由两个必需部分组成：`.mod` 描述文件（告诉启动器 mod 的基本信息）和 mod 文件夹（镜像游戏内部目录结构）。理解这个结构是 mod 制作的第一步，也是避免「改了不生效」问题的关键。

## 完整语法参考

### .mod 描述文件

放置在 `Documents\Paradox Interactive\Hearts of Iron IV\mod\` 目录下，文件名即 mod 标识。

```
name = "Your Mod Name"
path = "mod/your_mod_folder"
supported_version = "1.18.*"
tags = { "Alternative History" "Gameplay" }
picture = "thumbnail.png"
```

**必填字段：**

| 字段 | 说明 | 示例 |
|------|------|------|
| `name` | Mod 显示名称 | `"My Overhaul Mod"` |
| `path` | Mod 文件夹相对路径 | `"mod/mymod"` |
| `supported_version` | 支持的游戏版本 | `"1.18.*"` |
| `tags` | Steam Workshop 分类标签 | `{ "Alternative History" "Gameplay" }` |

**可选字段：**

| 字段 | 说明 | 示例 |
|------|------|------|
| `picture` | 缩略图文件名 | `"thumbnail.png"` |
| `replace_path` | 完全替换某路径，忽略 vanilla 文件 | `replace_path = { "common/technologies" }` |
| `user_dir` | 独立存档/配置目录 | `user_dir = "MyModSave"` |
| `dependencies` | 依赖其他 mod | `dependencies = { "Kaiserreich" }` |
| `remote_file_id` | Steam Workshop ID（上传后自动填写） | `remote_file_id = "1234567890"` |

### replace_path 详解

`replace_path` 告诉游戏**完全忽略**原版对应目录下的所有文件，只加载你 mod 中的版本。用于 total conversion mod。

```
# 替换整个科技树——原版 common/technologies/ 下的所有文件被忽略
replace_path = { "common/technologies" }

# 同时替换多个路径
replace_path = { "common/national_focus" "history/countries" "history/states" }
```

**不使用时**：游戏会加载原版文件 + 你的 mod 文件，两者合并。这是大多数 mod 的做法。

### Mod 文件夹结构

```
your_mod/
├── common/
│   ├── national_focus/     — 国策树（.txt，每个文件定义一个 focus_tree）
│   ├── decisions/          — 决议（.txt）
│   │   └── categories/     — 决议分类定义
│   ├── ideas/              — 国家精神/设计商/顾问
│   ├── technologies/       — 科技树
│   ├── units/              — 师编制/装备
│   │   └── equipment/      — 装备定义
│   ├── scripted_effects/   — 可复用效果块
│   ├── scripted_triggers/  — 可复用触发器块
│   └── scripted_localisation/ — 脚本化本地化
├── events/                 — 事件文件（.txt）
├── history/
│   ├── countries/          — 国家起始设定（一个文件一个国家）
│   └── states/             — 省份/地区数据
├── localisation/
│   └── english/            — 英文文本（.yml，UTF-8 BOM）
├── gfx/
│   ├── flags/              — 旗帜（.tga 格式）
│   ├── leaders/            — 领导人/顾问肖像（.dds）
│   ├── interface/          — UI 图标
│   └── ideas/              — 国家精神/设计商图标
└── map/                    — 地图文件（高级，仅 total conversion）
```

### 关键规则

1. **文件路径必须完全匹配**游戏内部路径。游戏 `common/national_focus/` 对应你的 `common/national_focus/`。
2. **不要覆盖整个 vanilla 文件**——mod 中添加新文件（如 `my_focus_tree.txt`），游戏会同时加载原版和你的文件。
3. **游戏加载目录下所有 .txt 文件**——无需在某个地方注册新文件。
4. **Windows 不区分大小写，Mac/Linux 区分**——为了兼容，始终使用与原版一致的大小写。

## 实战示例

### 示例 1：最小可运行 Mod

目标是让启动器能识别、勾选、进入游戏不报错。

**步骤 1**：创建 .mod 文件

`Documents\Paradox Interactive\Hearts of Iron IV\mod\test_mod.mod`：
```
name = "Test Mod"
path = "mod/test_mod"
supported_version = "1.18.*"
tags = { "Gameplay" }
```

**步骤 2**：创建 mod 文件夹

`Documents\Paradox Interactive\Hearts of Iron IV\mod\test_mod\`（空文件夹即可）

**步骤 3**：在 HOI4 启动器中验证

打开启动器 → "播放集" → "添加更多 Mod" → 应能看到 "Test Mod"。勾选后启动游戏，无报错即成功。

### 示例 2：替换科技的 Total Conversion 骨架

```
name = "Total Overhaul"
path = "mod/overhaul"
supported_version = "1.18.*"
tags = { "Alternative History" "Total Conversion" }
replace_path = { "common/technologies" "common/national_focus" "history/countries" "history/states" }
```

然后创建对应的 mod 文件夹，在其中放置你自己的科技树、国策树、国家文件和省份文件。

## 常见陷阱

1. **.mod 文件 path 写错** — path 是相对于 `Documents\Paradox Interactive\Hearts of Iron IV\` 的路径。正确的 path 是 `mod/yourfolder`，不是 `yourfolder` 或绝对路径。
2. **mod 文件夹名与 path 不一致** — .mod 里写 `path = "mod/my_mod"`，但实际文件夹叫 `my mod`（空格/大小写不一致），导致无法加载。
3. **忘记 replace_path 导致双重内容** — 做 total conversion 替换科技时，不加 replace_path 会导致你的科技和原版科技都被加载。
4. **在 Steam 安装目录改文件** — 游戏更新时会被覆盖。始终在 mod 文件夹中操作。
5. **编码问题** — .mod 文件也必须是 UTF-8 无 BOM。

## FAQ

**Q: 如何让 mod 依赖另一个 mod（如 Kaiserreich）？**
A: 在 .mod 文件中添加 `dependencies = { "Kaiserreich" }`。名称必须完全匹配启动器中显示的 mod 名。

**Q: 测试时不想每次都手动勾选 mod？**
A: 在启动器中创建"播放集"，添加你的 mod。之后该播放集会记住选择。

**Q: mod 文件夹名字有空格会怎样？**
A: 可能导致启动器识别问题。使用下划线或连字符：`my_mod` 而非 `my mod`。

## 检查清单

- [ ] .mod 文件已创建且格式正确
- [ ] path 字段指向正确的 mod 文件夹
- [ ] mod 文件夹名称与 .mod 中 path 一致
- [ ] 目录结构镜像游戏内部路径
- [ ] 未修改 Steam 安装目录下的原版文件
- [ ] 启动器中可见并可勾选 mod
- [ ] 启动游戏无错误

> **相关文件**: 环境搭建（01-env-setup.md）告诉你如何配置 VS Code 和 debug 模式。调试排错（14-debugging.md）教你查看 error.log 定位加载问题。
