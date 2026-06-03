> **适用版本**: HOI4 1.18.2  |  **最后更新**: 2026-06-03

## 概述

将 HOI4 mod 发布到 Steam Workshop（创意工坊）并管理后续更新。涵盖发布前检查清单、上传方式、语义化版本号、descriptor.mod 更新流程、兼容性声明、HOI4 版本更新后的 mod 适配流程，以及 AI 辅助内容声明建议。

## 发布前检查清单

在点下「上传」按钮之前，确认以下每一项都已就绪：

### 1. 缩略图

- 文件名：`thumbnail.png`（放置在 mod 文件夹根目录）
- 尺寸：**512x512** 像素（Steam 要求，小于此尺寸会被自动拉伸，大于则被压缩）
- 格式：PNG，建议使用 32-bit RGBA（带透明通道）
- 内容：清晰可辨识的 mod 图标/封面，避免过于复杂的细节（Steam 列表中缩略图显示很小）
- 文件大小：无硬限制，但建议 < 2 MB

### 2. 标签（tags）

在 `.mod` 文件中设置：

```
tags = { "Alternative History" "Gameplay" "Events" "National Focuses" }
```

**规则**：至少 1 个、最多 5 个——从 Steam 预定义标签中选择（自定义标签不会被检索到），按重要性排序。常用标签还有 `"Technologies"` `"Balance"` `"Total Conversion"` `"Graphics"` `"Sound"` `"Translation"` `"Utilities"` `"Historical"` `"Military"` `"Map"` `"Ideologies"`。

### 3. 描述（Description）

Steam Workshop 页面支持 BBCode 格式（`[b]`, `[i]`, `[u]`, `[list]`, `[h1]` 等）。建议结构：`[center][b]标题[/b][/center]` → `[h1]概述[/h1]`（2-4 句摘要） → `[h1]特性[/h1]`（`[list][*]特性[/list]`） → `[h1]兼容性[/h1]`（游戏版本、铁人模式、已知冲突 Mod） → `[h1]已知问题[/h1]` → `[h1]致谢[/h1]`。

### 4. 版本号

在描述或更新日志中声明版本号，方便用户对比。详见下方「语义化版本号」部分。

### 5. Mod 文件最终检查

- `.mod` 文件的 `name`、`path`、`supported_version` 确认无误
- `thumbnail.png` 在 mod 文件夹根目录
- 无调试遗留（如 `set_country_flag = debug_test`、事件自动触发等）
- 所有本地化的 key 都有对应文本（无 `loc_missing` 占位符）
- 游戏中至少启动一次确认无报错（查看 `error.log`）

### 6. AI 辅助声明

见下方「AI 辅助声明」专节。

## 上传方法

### 方法 1：HOI4 启动器内上传（推荐）

1. 打开 Paradox 启动器
2. 进入「播放集」→ 找到你的 Mod
3. 点击 Mod 右侧的「更多」(...) →「上传到 Steam Workshop」
4. 启动器自动处理：
   - 压缩 mod 文件夹为 .zip
   - 读取 `.mod` 文件获取 name、tags
   - 上传到 Steam Workshop
5. 上传完成后，`.mod` 文件中自动写入 `remote_file_id = "xxxxxxxxxx"`

**注意**：启动器会**同时更新** `Documents\...\mod\` 中的 `.mod` 文件和 mod 文件夹中的 `descriptor.mod` 文件。

### 方法 2：SteamCMD 手动上传（备用）

启动器上传失败时（大型 mod 或网络问题），使用 SteamCMD：

1. 安装 SteamCMD 并登录
2. 创建 `workshop.vdf` 配置文件，设置 `appid = "394360"`、`contentfolder`、`previewfile`、`visibility`（0=公开）、`title`、`description`、`changenote`
3. 首次上传 `publishedfileid = "0"`，更新时填写已有 Workshop ID。运行 `steamcmd +login USERNAME +workshop_build_item workshop.vdf +quit`

## 语义化版本号

遵循 **major.minor.patch**（主版本.次版本.修订号）规范：

| 级别 | 变更类型 | 示例 |
|------|---------|------|
| **Major** | 破坏性变更——旧存档不可用、核心机制重做、总转化重构 | `1.0.0 → 2.0.0` |
| **Minor** | 新功能、新国策树、新事件链——向后兼容 | `1.0.0 → 1.1.0` |
| **Patch** | Bug 修复、平衡调整、文本修正 | `1.0.0 → 1.0.1` |

**版本号格式要点**：
- 始终使用数字，不要用字母（`1.0a` 是不规范的）
- 先行版可用后缀：`0.1.0-alpha`、`0.2.0-beta`
- 不要在版本号前加 `v`（Steam 显示会重复）
- 每个（非 patch）版本发布时写一条 changenote

**版本号声明位置**：
1. 在 `.mod` 文件中添加注释（不会显示给用户，但方便自己追踪）
2. 在 Steam Workshop 描述顶部声明
3. 在 Changelog/更新日志中逐版本记录

## descriptor.mod 更新流程

### 两个 .mod 文件的区别

一个 HOI4 mod 实际涉及**两个** `.mod` 文件：

| 位置 | 文件名 | 用途 |
|------|--------|------|
| `Documents\Paradox Interactive\Hearts of Iron IV\mod\` | `your_mod.mod` | 启动器读取——告诉启动器 mod 存在、路径、版本 |
| Mod 文件夹根目录 | `descriptor.mod` | 游戏本体读取——游戏运行时加载此文件获取 mod 元数据 |

### 标准更新流程

1. **修改 mod 文件夹根目录的 `descriptor.mod`**
   - 更新 `name`（如果改名）
   - 更新 `supported_version`（如果游戏版本升级后兼容）
   - 更新 `dependencies`（如果新增/移除依赖）

2. **同步修改 Documents 下的 `.mod` 文件**
   - `name` 必须一致
   - `path` 保持不变
   - `supported_version` 与 descriptor.mod 一致
   - `tags` 可根据需要调整

3. **在启动器内执行上传**
   - 启动器会检测到修改，提示「Mod 已更改」
   - 点击上传后，`remote_file_id` 自动保持（因为 .mod 中已有）

### 关键规则

- `name` 在两个文件中**必须完全一致**（包括大小写、空格），否则游戏内 mod 列表显示异常
- `remote_file_id` **只存在于** Documents 下的 `.mod` 文件中，descriptor.mod 中不应包含此字段
- 如果手动删除并重建了 mod 文件夹，确保 descriptor.mod 也已重新创建
- 上传到 Workshop 后，**不要手动编辑 `remote_file_id`**——删除或修改会导致更新链断裂

## 兼容性声明

### supported_version 字段

```
# 兼容 1.18 所有子版本
supported_version = "1.18.*"

# 仅兼容特定子版本
supported_version = "1.18.2"

# 兼容 1.17 和 1.18（不推荐，范围过大）
supported_version = "1.1*"
```

**最佳实践**：
- 测试过就写具体版本，如 `"1.18.2"`
- 如果机制简单（仅添加国策/事件），可大胆写 `"1.18.*"`
- 涉及 `replace_path` 或修改原版文件引用时，必须写具体版本
- 每次 HOI4 大版本更新后，优先用 `-debug` 测试一遍，再更新 `supported_version`

### 描述中的兼容性说明

在 Workshop 描述中补充：
- 铁人模式支持情况
- 已知兼容/不兼容的知名 Mod
- 是否需要特定 DLC
- 存档兼容性（旧版本存档能否在新版本继续使用）

## HOI4 补丁后的 Mod 更新流程

Paradox 约每 3-6 个月发布一次大版本更新。每次更新后按以下流程操作：

### 阶段 1：自查

阅读 Paradox 更新日志，用 WinMerge 对比新旧 vanilla 文件——重点关注 mod 中修改过的目录（`common/national_focus/`、`common/ideas/` 等），记录字段增删改。

### 阶段 2：测试

`-debug` 启动仅含你 mod 的游戏 → 检查 `error.log` → 控制台手动触发 mod 事件/国策 → `focus.autocomplete` 快速完成国策树测试 → 加载旧存档（如果声称兼容）。重点关注补丁前不存在的**新错误**。

### 阶段 3：修复

字段重命名 → 批量替换。旧字段新增必填字段 → 参考 vanilla 默认值补充。文件结构变化 → 重新拆分/合并。废弃 API → 用新方案重写。`replace_path` 冲突 → 更新路径列表。

### 阶段 4：发布

更新 `supported_version` + 递增版本号 + 编写 changenote → 启动器上传。在 Workshop 物品页面开启「评论通知」；补丁周关注用户反馈（bug 报告集中在 48 小时内）。优先级：崩溃 > 功能异常 > 文本/显示。

## 实战示例

### 示例 1：首次发布完整流程

以 mod「East Asia Expanded（东亚扩展）」为例。

**步骤 1**：发布前检查——`thumbnail.png` 512x512 就位；`.mod` 文件写好 `name`/`path`/`supported_version`/`tags`；`descriptor.mod` 同步（不含 `path` 和 `remote_file_id`）；游戏内无报错；本地化文本完整覆盖。

**步骤 2**：编写 Workshop 描述——BBCode 格式，包含概述（为东亚主要国家添加 200+ 国策/50+ 事件/30+ 决议）、特性列表、兼容性（铁人模式支持情况、已知冲突 Mod）、AI 辅助声明、致谢。

**步骤 3**：启动器上传——找到 Mod → 右键 →「上传到 Steam Workshop」。

**步骤 4**：确认——Steam 创意工坊页面缩略图/描述/标签正确显示；`.mod` 中 `remote_file_id` 已自动写入；请朋友在一台干净机器上订阅测试。

### 示例 2：HOI4 补丁适配更新（1.17 → 1.18）

**背景**：你的 mod 在 1.17 下发布，HOI4 更新到 1.18。

**步骤 1**：阅读更新日志 → 发现国策新增 `ai_will_do` 字段、`add_equipment_to_stockpile` 参数顺序改变。

**步骤 2**：WinMerge 对比 mod 修改过的 vanilla 文件（如 `germany.txt`）→ 记录差异。

**步骤 3**：修复——为所有国策补充 `ai_will_do = { base = 1 }`；调整 equipment stockpile 调用。

**步骤 4**：`-debug` 测试 → 无新增 error.log 报错。

**步骤 5**：更新 `supported_version = "1.18.2"`、版本号 `1.3.0 → 1.3.1`、写 changenote → 上传。

## 常见陷阱

1. **descriptor.mod 与 .mod 不同步** — 上传时只改了 descriptor.mod，忘了同步 Documents 下的 .mod 文件。结果：启动器显示旧名称/版本，但游戏内读取新数据。每次修改 descriptor.mod 后，**立即同步更新** Documents 下的 .mod。

2. **remote_file_id 丢失** — 手动删除并重新创建 .mod 文件时，`remote_file_id` 被清除。重新上传后 Steam 会当作**新物品**创建，导致旧订阅者收不到更新、评论区丢失。始终备份 .mod 文件中的 `remote_file_id`。

3. **thumbnail.png 尺寸错误** — 使用非 512x512 的图片作为缩略图。Steam 不拒绝上传，但图片会变模糊或在列表中显示异常。确保在图像编辑软件中确认画布尺寸为 512x512。

4. **上传时忘记选中正确的播放集** — 如果播放集中包含多个 mod（如依赖/子 mod），上传可能打包额外文件或缺失文件。上传前建议创建**仅含目标 mod** 的临时播放集。

5. **标签滥用导致搜索降权** — 超出 5 个标签（启动器限制了但 Steam 不限制），或使用不相关的热门标签（如明明是国策 mod 却标「Map」），会被 Steam 算法降权，搜索排名降低。

## FAQ

**Q: 上传后修改描述，需要重新上传整个 mod 吗？**
A: 不需要。Steam Workshop 描述和缩略图可以独立编辑——在 Steam 创意工坊物品页面直接编辑，不触发 mod 文件更新。

**Q: 如何删除已上传的 mod？**
A: 在 Steam 创意工坊 → 你的物品 → 编辑 → 可见性 → 设为「隐藏」。完全删除需联系 Steam 客服。隐藏后现有订阅者仍可使用，但新用户搜索不到。

**Q: 多人合作时如何管理 Workshop 上传权限？**
A: Steam Workshop 只允许原始上传者更新。可在物品页面添加「贡献者」——贡献者可以编辑描述、缩略图和可见性，但**不能上传文件更新**。多开发者方案：(a) 一人负责上传，(b) 使用 Git 协作，上传前合并分支。

**Q: 可以同时发布一个 mod 的多个版本吗（如稳定版和测试版）？**
A: 可以，但需要创建两个独立的 Steam Workshop 物品（两个 .mod 文件，不同 remote_file_id）。惯例命名：`Mod Name`（稳定版）+ `Mod Name - Beta`（测试版）。在各自的描述中互相链接。

**Q: Subscribers 数字突然下降是怎么回事？**
A: 可能原因：(a) Steam 缓存波动（每周二维护时常发生），几小时后自动恢复；(b) 你的 mod 在某些国家/地区被隐藏（检查可见性设置）；(c) 其他热门 mod 声明与你的 mod 冲突，用户被迫二选一。

## AI 辅助声明

当 mod 使用了 AI 工具（包括但不限于 Claude、ChatGPT、Copilot、Gemini、Midjourney/DALL-E 生成图像、语音合成等）辅助开发时，**建议**在 Workshop 描述中注明。

### 建议理由

透明度让订阅者了解创作过程；提前告知比被发现后质疑更有利；部分社区对 AI 内容有披露要求；「已人工审核」声明可回应质量质疑。

### 推荐声明文本

英文：
> **AI Disclosure**: Some content in this mod was developed with AI-assisted tools. All AI-generated content has been reviewed by a human and tested in-game.

中文：
> **AI 辅助声明**：本 Mod 部分内容由 AI 辅助生成。所有 AI 生成内容已通过人工审核和游戏内测试。

### 建议放置位置

- Workshop 描述末尾（在「致谢」之前）
- 如果 mod 有独立文档/GitHub README，同样建议注明

### 注意事项

- 声明是**建议**而非**必须**——Steam 目前不强制要求 AI 披露
- 如果 AI 仅用于代码格式检查、拼写修正等常规工具性质的任务，则无需声明
- 不夸大也不隐瞒 AI 的参与程度——准确描述

## 检查清单

- [ ] `thumbnail.png` 512x512 已放置在 mod 文件夹根目录
- [ ] `.mod` 文件的 `name`、`path`、`supported_version`、`tags` 确认无误
- [ ] `descriptor.mod` 与 `.mod` 内容同步（name、supported_version、tags）
- [ ] Workshop 描述已编写（含兼容性说明）
- [ ] 版本号已声明（major.minor.patch 格式）
- [ ] 游戏中已测试，无报错
- [ ] 调试遗留代码已清除
- [ ] 本地化文本完整，无 `loc_missing`
- [ ] AI 辅助声明已添加（如适用）
- [ ] 上传后 Steam Workshop 页面显示正常
- [ ] `remote_file_id` 已备份

> **相关文件**: Mod 项目结构（02-mod-structure.md）介绍了 .mod 和 descriptor.mod 的区别与基础结构。调试排错（14-debugging.md）讲解在上传前如何查看 error.log 确认无报错。
