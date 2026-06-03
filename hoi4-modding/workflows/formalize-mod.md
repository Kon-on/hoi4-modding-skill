# 工作流：正式化 & 发布 Mod

> **适用版本**: HOI4 1.18.2 | **最后更新**: 2026-06-03
>
> 参考: `references/15-workshop.md` `references/02-mod-structure.md`
>
> 适用场景：测试版 Mod 功能验证完毕，准备清理临时命名、补全元数据并发布到 Steam Workshop

## 概述

将测试版 Mod 升级为正式版本：统一命名、补全元数据、准备描述与截图、设置版本号、通过 Git 管理并上传至 Steam Workshop。核心原则：**先测试后正式化——确保所有功能已在测试版中验证通过。**

| 属性 | 测试版 | 正式版 |
|------|--------|--------|
| Mod 名称 | `My Mod (Test)` | 正式名称，无后缀 |
| packageId | `test.my_mod` | `author.my_mod`（去掉 test. 前缀） |
| 描述 | 简短/临时 | 完整的 Steam Workshop 描述 |
| 截图 | 无 | 至少 3-5 张 |
| 版本号 | 0.1.0 或未设置 | 语义化版本（1.0.0） |
| Git | 本地提交 | 已提交 + 打 tag |

---

## 阶段一：命名统一

### 目标
替换所有测试占位符为正式 ID、packageId 和名称。

### 步骤

1. **确定正式前缀** — 格式 `<作者名>_` 或 `<缩写>_`（如 `mye_`），所有自定义 ID 使用此前缀

2. **替换 packageId**：`test.my_empire` → `author_name.my_empire`（在 `.mod` 和 `About.xml` 中同步修改）。**一旦发布 packageId 不可再改！**

3. **批量替换临时 ID**：

   | 位置 | 搜索 | 替换为 |
   |------|------|--------|
   | 国策树 | `test_xxx` `tmp_xxx` | `mye_xxx` |
   | 事件 | `test_event.1` | `mye_event.1` |
   | 国家精神 | `test_idea_xxx` | `mye_idea_xxx` |
   | 决议 | `test_decision_xxx` | `mye_decision_xxx` |
   | 科技 | `test_tech_xxx` | `mye_tech_xxx` |
   | 本地化 key | 含 test/tmp | 对应正式 id |

4. **验证清除**：
   ```bash
   grep -r "test_\|tmp_\|TST_\|placeholder" "$MOD_DIR/" --include="*.txt" --include="*.yml" | grep -v "#"
   ```
   预期输出为空。

### 输出物
- 所有 ID 统一为正式前缀，packageId 已更新

---

## 阶段二：补全元数据

### 目标
补全 `.mod` 文件和/或 `About.xml` 的所有元数据字段。

### 步骤

1. **更新 .mod 文件**：
   ```ini
   name = "My Empire"                                    # 正式名称
   path = "mod/my_empire"                                # 确认路径
   supported_version = "1.18.*"                          # 目标版本
   tags = { "Alternative History" "Gameplay" "National Focuses" "Events" }
   picture = "thumbnail.png"                             # 缩略图
   # 可选: dependencies = { }, replace_path = { ... }, user_dir = "MyEmpire"
   ```

   **可用 Steam Workshop 标签**：`"Alternative History"` `"Gameplay"` `"National Focuses"` `"Events"` `"Decisions"` `"Ideologies"` `"Technologies"` `"Graphics"` `"Map"` `"Sound"` `"Total Conversion"` `"Balance"` `"Historical"` `"Fixes"` `"Translation"`

2. **About.xml**（如使用）：
   ```xml
   <ModMetaData>
     <packageId>author.my_empire</packageId>
     <name>My Empire</name>
     <author>Your Name</author>
     <version>1.0.0</version>
     <supportedVersions><li>1.18.*</li></supportedVersions>
     <tags><li>Alternative History</li><li>Gameplay</li></tags>
   </ModMetaData>
   ```

### 输出物
- 完整的 .mod 文件和/或 About.xml

---

## 阶段三：描述与截图

### 目标
编写 Steam Workshop 描述，准备缩略图和 3-5 张截图。

### 步骤

1. **编写描述**（Steam Workshop 支持 Markdown 风格标记）：

   ```
   [h1]Mod Name[/h1]

   [h2]Overview[/h2]
   Brief description of what the mod does.

   [h2]Features[/h2]
   [list]
   [*] Feature 1
   [*] Feature 2
   [*] Feature 3
   [/list]

   [h2]AI Assistance Disclosure[/h2]
   Some content in this mod was developed with AI-assisted tools.
   本 Mod 部分内容由 AI 辅助生成。

   [h2]Compatibility[/h2]
   Compatible with HOI4 1.18.*. Not compatible with total conversion mods.

   [h2]Changelog[/h2]
   [b]v1.0.0[/b] - Initial release

   [h2]Credits[/h2]
   Created by [Your Name]
   ```

   **描述必备段落**：Overview / Features / Compatibility / Changelog / Credits

2. **CRITICAL -- AI 辅助声明（如适用）**：
   - [ ] 如在开发中使用了 AI 工具（Claude Code、Copilot、ChatGPT 等），**必须在描述中加入声明**
   - 英文：`"Some content in this mod was developed with AI-assisted tools."`
   - 中文：`"本 Mod 部分内容由 AI 辅助生成。"`
   - 这是透明度的最佳实践，强烈建议

3. **准备缩略图（thumbnail.png）**：
   - 尺寸：512 x 512 像素，格式：PNG
   - 路径：`<mod>/thumbnail.png`
   - 在 `.mod` 中添加：`picture = "thumbnail.png"`

4. **准备截图**（3-5 张，最多 10 张）：
   - 内容建议：国策树全景、地图位置、国家信息面板、特色系统、对比图
   - 分辨率：1920x1080（原生）
   - 用 F12（Steam 截图）或 Print Screen 截取

### 输出物
- Steam Workshop 描述文本、thumbnail.png、3-5 张截图

---

## 阶段四：版本号设置

### 目标
使用语义化版本号标记版本。

### 语义化版本（MAJOR.MINOR.PATCH）

| 版本位 | 何时递增 | 示例 |
|--------|---------|------|
| MAJOR | 不兼容的大改动（存档不可用） | 1.0.0 → 2.0.0 |
| MINOR | 向后兼容的新功能 | 1.0.0 → 1.1.0 |
| PATCH | 向后兼容的 bug 修复 | 1.0.0 → 1.0.1 |

| Mod 阶段 | 版本号 |
|---------|--------|
| 首次发布 | 1.0.0 |
| 新增国策/事件 | 1.1.0 |
| 修复 bug | 1.1.1 |
| 重大重构 | 2.0.0 |

### 步骤
1. 在 `.mod` 中添加版本注释：`# Version: 1.0.0`（.mod 文件无官方 version 字段，用注释记录）
2. 在 Workshop 描述的 Changelog 中标明版本号
3. 在 Git 中创建对应 tag（见阶段五）

### 输出物
- 版本号已确定并记录

---

## 阶段五：Git 提交与 Tag

### 目标
将正式化变更提交到 Git，创建版本 tag。

### 步骤

1. **（如尚未初始化）**：
   ```bash
   cd path/to/your/mod
   git init && git add . && git commit -m "Initial commit"
   ```

2. **提交正式化变更**：
   ```bash
   git add .
   git commit -m "formalize: version 1.0.0 - official release"
   ```

3. **创建 tag**：
   ```bash
   git tag -a v1.0.0 -m "Release version 1.0.0"
   ```
   格式 `vMAJOR.MINOR.PATCH`，使用 `-a` 创建带注释 tag。

4. **（可选）推送**：
   ```bash
   git push origin main
   git push origin v1.0.0    # tag 需单独推送
   ```

### 输出物
- Git 提交完成，版本 tag 已创建

---

## 阶段六：上传 Steam Workshop

### 目标
将 Mod 上传到 Steam Workshop 并配置页面。

### 前提
- Steam 已登录，HOI4 已购买；Mod 在启动器中可正常加载；以上五阶段全部完成

### 步骤

1. **上传**：
   - 打开 HOI4 启动器 → Mods → 找到你的 Mod → "上传到 Steam 创意工坊"
   - 首次上传创建新 Workshop 页面，更新上传覆盖已有版本

2. **配置页面**：
   - 粘贴阶段三描述
   - 上传 thumbnail.png 作为封面
   - 上传截图
   - 设置可见性：Public（推荐）/ Friends Only（测试）/ Hidden（私密）
   - 选择分类和标签（与 .mod 文件中 tags 一致）

3. **测试已发布版本**：
   - 取消本地 Mod 勾选 → 在 Workshop 中订阅 → 勾选 Workshop 版本 → 启动验证

4. **更新 .mod 文件**（上传后 Steam 自动添加 `remote_file_id`）：
   ```bash
   git add my_mod.mod
   git commit -m "Add remote_file_id from Steam Workshop"
   git push
   ```

### 输出物
- Mod 已发布，Workshop 页面完整，已通过 Workshop 版本测试

---

## 检查清单

### 命名统一
- [ ] packageId 已去掉 `test.` 前缀
- [ ] 所有自定义 ID 使用统一正式前缀
- [ ] 无 test_ / tmp_ / TST_ / placeholder 残留
- [ ] Mod 名称已去掉 (Test) 后缀

### 元数据
- [ ] name 为正式名称
- [ ] path 指向正确的 mod 文件夹
- [ ] supported_version 为 `"1.18.*"`
- [ ] tags 包含正确分类标签
- [ ] author 信息完整

### 描述与截图
- [ ] Workshop 描述已编写（Overview、Features、Compatibility、Changelog）
- [ ] **[ ] 已在 Mod 描述中加入 AI 辅助声明（如适用）**
  - 英文：`"Some content in this mod was developed with AI-assisted tools."`
  - 中文：`"本 Mod 部分内容由 AI 辅助生成。"`
- [ ] 512x512 thumbnail.png 已放置于 mod 根目录
- [ ] 3-5 张 1920x1080 截图已准备，展示核心功能

### 版本号
- [ ] 使用语义化版本（MAJOR.MINOR.PATCH）
- [ ] 版本号与 Changelog 一致

### Git
- [ ] 正式化变更已提交
- [ ] 版本 tag（vX.Y.Z）已创建

### Steam Workshop
- [ ] Mod 已成功上传
- [ ] Workshop 页面信息完整（描述、截图、封面、标签）
- [ ] AI 辅助声明在页面中可见
- [ ] 已订阅 Workshop 版本并验证功能正常
- [ ] remote_file_id 已更新到 .mod 文件

---

> **下一步**：发布后通过 Steam Workshop 更新工具推送新版本，更新 Changelog，根据玩家反馈维护。
