# Feature: 移除与 tmux 重复的 kitty 功能

## Feature Description

精简 oh-my-kitty 配置，移除与 tmux 功能重复的部分，将 kitty 还原为纯粹的终端模拟器角色。保留 kitty 特有的功能（hints、主题、diff 等），让 tmux 专门负责终端复用、会话管理和分屏布局。

## User Story

作为使用 tmux 进行终端复用的用户
我想要精简 kitty 配置，移除与 tmux 重复的功能
这样配置更清晰，职责分离更明确，减少功能冲突和键绑定混乱

## Problem Statement

当前 oh-my-kitty 配置模拟了大量 tmux 功能：
- 分屏管理（水平/垂直分割、窗口导航、大小调整）
- 会话持久化（保存/恢复会话）
- 窗口/标签页复杂管理（创建、重命名、切换）
- tmux 风格的 `Ctrl+a` 前缀键系统

这些功能与 tmux 完全重复，造成：
1. 配置文件冗余复杂（splits.conf 完全是 tmux 替代）
2. 键绑定冲突风险（tmux 和 kitty 都响应类似操作）
3. 功能职责不清晰（不确定该用 tmux 还是 kitty 的分屏）
4. 维护成本高（需要同时维护两套类似的配置）

## Solution Statement

系统性地移除 kitty 中与 tmux 重复的功能，保留 kitty 的核心职能和特有功能：

**移除范围**：
- 完整删除 splits.conf（所有分屏管理键绑定）
- 删除 zoom_toggle.py（窗口缩放功能）
- 移除会话管理相关配置和键绑定
- 移除所有 `Ctrl+a` 前缀键绑定
- 简化布局系统配置

**保留范围**：
- 基础终端设置（字体、颜色、主题、透明度）
- Kitty 特有功能（hints、diff、ssh kitten、icat）
- 基础操作（字体调整、配置管理、全屏、退出）
- Shell 集成（保留，不影响 tmux 使用）

**重新分配键绑定**：
- 将保留的 kitty 特有功能改用非 `Ctrl+a` 前缀的快捷键
- 使用 `kitty_mod`（Ctrl+Shift）作为主要修饰键
- 保持简洁，避免与 tmux 冲突

## Feature Metadata

**Feature Type**: Refactor
**Estimated Complexity**: Medium
**Primary Systems Affected**: kitty.conf, splits.conf, zoom_toggle.py, CLAUDE.md
**Dependencies**: 无外部依赖，纯配置重构

---

## CONTEXT REFERENCES

### Relevant Codebase Files

**需要修改的文件**：
- `kitty.conf` (行 49-50, 130, 162-163, 200-289) - 主配置文件，需要移除 tmux 相关配置和键绑定
- `splits.conf` (全部) - 完整的 tmux 分屏管理替代，需要删除
- `zoom_toggle.py` (全部) - tmux zoom 功能替代，需要删除
- `CLAUDE.md` (行 186-295) - 项目文档，需要更新以反映新的配置架构

**保留不变的文件**：
- `custom-hints.py` - Kitty 特有的智能提示功能，保留
- `diff.conf` - Kitty diff 工具配置，保留
- `open-actions.conf` - 文件打开动作，保留
- `current-theme.conf` - 当前主题，保留
- `.gitignore` - 版本控制，保留

### Files to Remove

- `splits.conf` - 完整的 tmux 风格分屏管理配置文件
- `zoom_toggle.py` - 窗口缩放切换脚本（tmux zoom 替代）

### Relevant Documentation

- [Kitty 配置文档](https://sw.kovidgoyal.net/kitty/conf/)
  - Why: 验证配置项的正确性
- [Kitty 键盘快捷键](https://sw.kovidgoyal.net/kitty/conf/#keyboard-shortcuts)
  - Why: 重新分配键绑定时参考
- [Kitty Kittens 文档](https://sw.kovidgoyal.net/kitty/kittens/hints/)
  - Why: 保留的 hints 功能参考

### Patterns to Follow

**配置注释风格**（从 kitty.conf）：
```conf
#: 多行注释说明配置项的作用
#: 提供额外的上下文和示例
option_name value
```

**键绑定格式**（从 kitty.conf）：
```conf
# 单行注释描述快捷键作用
map modifier+key action
```

**Include 语法**（从 kitty.conf:56, 275）：
```conf
include filename.conf
# BEGIN_KITTY_THEME
include current-theme.conf
# END_KITTY_THEME
```

**配置禁用模式**（从 kitty.conf:162, 202）：
```conf
#option_name disabled_value  # 使用注释保留默认值参考
option_name actual_value      # 实际使用的值
```

---

## IMPLEMENTATION PLAN

### Phase 1: 备份和准备

**Tasks:**
- 创建当前配置的备份
- 记录当前所有键绑定（用于对比验证）
- 确认 tmux 配置正常工作

### Phase 2: 移除 tmux 重复功能

**Tasks:**
- 删除 splits.conf 文件
- 删除 zoom_toggle.py 文件
- 从 kitty.conf 中移除对 splits.conf 的引用
- 移除所有 `Ctrl+a` 前缀键绑定
- 移除会话管理相关配置
- 简化布局系统配置

### Phase 3: 重新分配保留功能的键绑定

**Tasks:**
- 为配置管理功能分配新快捷键
- 为 kitty 特有功能（hints, themes）分配新快捷键
- 确保新键绑定不与 tmux 冲突

### Phase 4: 配置验证和文档更新

**Tasks:**
- 测试 kitty 配置加载无错误
- 验证保留功能的键绑定工作正常
- 更新 CLAUDE.md 文档
- 更新 README.md（如果需要）

---

## STEP-BY-STEP TASKS

### BACKUP kitty configuration files

- **IMPLEMENT**: 创建配置备份目录并复制所有配置文件
- **VALIDATE**: `ls -la backup/ && diff kitty.conf backup/kitty.conf`

```bash
mkdir -p backup
cp kitty.conf backup/
cp splits.conf backup/
cp zoom_toggle.py backup/
cp current-theme.conf backup/
```

### REMOVE splits.conf file

- **IMPLEMENT**: 删除完整的 tmux 风格分屏配置文件
- **GOTCHA**: 这个文件包含所有 splits 相关的键绑定，删除后这些功能将完全由 tmux 接管
- **VALIDATE**: `test ! -f splits.conf && echo "splits.conf removed successfully"`

```bash
rm splits.conf
```

### REMOVE zoom_toggle.py file

- **IMPLEMENT**: 删除窗口缩放脚本（tmux zoom 功能替代）
- **VALIDATE**: `test ! -f zoom_toggle.py && echo "zoom_toggle.py removed successfully"`

```bash
rm zoom_toggle.py
```

### UPDATE kitty.conf - Remove splits.conf include

- **IMPLEMENT**: 移除对 splits.conf 的引用
- **PATTERN**: kitty.conf:56
- **VALIDATE**: `grep -q "include splits.conf" kitty.conf && echo "FAILED: splits.conf still referenced" || echo "PASS: reference removed"`

在 kitty.conf 中删除或注释掉：
```conf
# https://sw.kovidgoyal.net/kitty/layouts/#the-splits-layout
include splits.conf
```

### UPDATE kitty.conf - Simplify enabled_layouts

- **IMPLEMENT**: 简化布局系统，移除 splits 布局
- **PATTERN**: kitty.conf:49
- **GOTCHA**: tmux 将负责所有分屏管理，kitty 只需要基础布局
- **VALIDATE**: `grep "^enabled_layouts" kitty.conf`

修改：
```conf
# 修改前：enabled_layouts splits,stack
# 修改后：使用 kitty 默认值或简单布局
enabled_layouts tall,stack
```

### UPDATE kitty.conf - Disable startup_session

- **IMPLEMENT**: 禁用会话自动加载（会话管理由 tmux 负责）
- **PATTERN**: kitty.conf:163
- **VALIDATE**: `grep "^startup_session" kitty.conf`

修改：
```conf
# 修改前：startup_session session.conf
# 修改后：
startup_session none
```

### UPDATE kitty.conf - Remove Ctrl+a prefix keybindings

- **IMPLEMENT**: 移除所有 tmux 风格的 Ctrl+a 前缀键绑定
- **PATTERN**: kitty.conf:207-287
- **VALIDATE**: `grep "^map ctrl+a>" kitty.conf | wc -l | grep -q "^0$" && echo "PASS" || echo "FAILED: Ctrl+a bindings still exist"`

删除以下键绑定：
- `map ctrl+a>x close_window`
- `map ctrl+a>] next_window`
- `map ctrl+a>[ previous_window`
- `map ctrl+a>period move_window_forward`
- `map ctrl+a>comma move_window_backward`
- `map ctrl+a>c launch --cwd=last_reported --type=tab`
- `map ctrl+a>, set_tab_title`
- `map ctrl+a>e no-op`
- `map ctrl+a>shift+e launch --type=tab nvim ~/.config/kitty/kitty.conf`
- `map ctrl+a>shift+r combine : load_config_file : launch --type=overlay sh -c 'echo "kitty config reloaded."; echo; read -r -p "Press Enter to exit"; echo ""'`
- `map ctrl+a>shift+d debug_config`
- `map ctrl+a>space kitten hints --alphabet asdfqwerzxcvjklmiuopghtybn1234567890 --customize-processing custom-hints.py`
- `map ctrl+a>ctrl+a send_text all \x01`

### UPDATE kitty.conf - Remove unnecessary window/tab mappings

- **IMPLEMENT**: 移除 tmux 功能重复的窗口/标签页快捷键
- **PATTERN**: kitty.conf:226
- **VALIDATE**: `grep "kitty_mod+t launch --location=hsplit" kitty.conf`

删除或修改：
```conf
# 删除：map kitty_mod+t launch --location=hsplit
# 这个键绑定是分屏功能，应该用 tmux 的 prefix+% 或 prefix+"
```

### ADD kitty.conf - New keybindings for kitty-specific features

- **IMPLEMENT**: 为保留的 kitty 特有功能分配新的快捷键
- **PATTERN**: 使用 kitty_mod (Ctrl+Shift) 避免与 tmux 冲突
- **VALIDATE**: `grep "^map kitty_mod" kitty.conf | grep -E "(hints|themes|reload|edit)" | wc -l`

添加新的键绑定：
```conf
# === Kitty Configuration Management ===

# Reload kitty configuration
map kitty_mod+f5 combine : load_config_file : launch --type=overlay sh -c 'echo "kitty config reloaded."; echo; read -r -p "Press Enter to exit"; echo ""'

# Edit kitty configuration
map kitty_mod+, launch --type=overlay nvim ~/.config/kitty/kitty.conf

# Debug kitty configuration
map kitty_mod+shift+d debug_config

# === Kitty-Specific Features ===

# Smart hints (copy URLs, paths, filenames)
map kitty_mod+h kitten hints --alphabet asdfqwerzxcvjklmiuopghtybn1234567890 --customize-processing custom-hints.py

# Theme selector
map kitty_mod+f1 kitten themes

# Generic hints with default program
# (保留原有的 f3 绑定)
# map f3 kitten hints --program '*'

# === Font Size Adjustment ===
# (保留原有的 ctrl+=/- 绑定)

# === Fullscreen and Exit ===
# (保留原有的 f11 和 ctrl+q 绑定)
```

### UPDATE kitty.conf - Add configuration comments

- **IMPLEMENT**: 添加清晰的注释说明配置哲学（kitty 作为基础终端，tmux 负责复用）
- **PATTERN**: 在文件顶部添加说明
- **VALIDATE**: `grep "tmux" kitty.conf | head -3`

在 kitty.conf 顶部添加：
```conf
# vim:fileencoding=utf-8:foldmethod=marker

# oh-my-kitty - Minimalist Configuration for tmux Users
#
# Philosophy: kitty as a pure terminal emulator, tmux for multiplexing
# - kitty handles: fonts, colors, themes, terminal features (hints, diff, icat)
# - tmux handles: window/pane management, sessions, layouts, splits
#
# This configuration removes all tmux-duplicate features to avoid conflicts
# and maintain clear separation of concerns.

# https://sw.kovidgoyal.net/kitty/conf/
```

### VALIDATE kitty.conf - Test configuration loading

- **IMPLEMENT**: 验证配置文件语法正确，无错误
- **VALIDATE**: `kitty --debug-config 2>&1 | grep -i error`

```bash
# 测试配置加载
kitty --debug-config
```

### UPDATE CLAUDE.md - Update project overview

- **IMPLEMENT**: 更新项目概览，反映新的设计哲学
- **PATTERN**: CLAUDE.md:188-200
- **VALIDATE**: `grep "tmux" CLAUDE.md | grep "专门负责"`

修改项目概览部分：
```markdown
## 项目概览

**oh-my-kitty** - 简洁的 kitty 终端配置，专为 tmux 用户设计

这是一个精简的 kitty 终端模拟器配置，遵循"单一职责"原则：
- **kitty**: 纯粹的终端模拟器，负责字体、颜色、主题和终端特有功能
- **tmux**: 专门负责终端复用、会话管理、分屏布局

### 核心特性

- **轻量配置**: 移除与 tmux 重复的功能，保持配置简洁
- **清晰职责**: kitty 专注终端渲染，tmux 专注会话管理
- **Kitty 特有功能**: 智能提示（hints）、主题切换、diff 工具、icat 图片查看
- **无冲突**: 键绑定设计避免与 tmux 冲突
```

### UPDATE CLAUDE.md - Update architecture section

- **IMPLEMENT**: 更新架构说明，移除已删除的文件
- **PATTERN**: CLAUDE.md:204-229
- **VALIDATE**: `grep "splits.conf" CLAUDE.md`

修改架构说明：
```markdown
## 架构说明

### 配置文件组织

```
kitty.conf              # 主配置文件（终端设置 + kitty 特有功能）
├── current-theme.conf  # 当前主题配色
├── diff.conf           # diff 工具配置
└── open-actions.conf   # 文件打开动作配置
```

### Python 脚本

- **custom-hints.py**: 自定义 hints kitten 的匹配逻辑
  - 匹配 URL、文件路径、常见文件扩展名
  - 使用正则表达式: `RE_URL_OR_PATH`
  - 选中内容自动复制到剪贴板

### 核心设计

1. **职责分离**: kitty 不再处理分屏/会话管理，完全交给 tmux
2. **Shell 集成**: 保留 `shell_integration enabled` 用于增强终端功能
3. **远程控制**: 保留 `allow_remote_control` 支持 kitty 扩展和脚本
4. **Kitty 特有功能**: 充分利用 hints、主题、diff、icat 等 kitty 独有能力
```

### UPDATE CLAUDE.md - Update keybindings section

- **IMPLEMENT**: 更新常用命令部分，只保留 kitty 相关的快捷键
- **PATTERN**: CLAUDE.md:244-275
- **VALIDATE**: `grep "Ctrl+a" CLAUDE.md | wc -l`

替换为新的键绑定说明：
```markdown
### 常用命令

**配置管理**
- 重载配置: `Ctrl+Shift+F5`
- 编辑配置: `Ctrl+Shift+,`
- 调试配置: `Ctrl+Shift+D`

**Kitty 特有功能**
- 智能提示(复制): `Ctrl+Shift+H`
- 主题切换: `Ctrl+Shift+F1`
- 通用提示: `F3`

**字体调整**
- 放大: `Ctrl+=` 或 `Ctrl++`
- 缩小: `Ctrl+-`
- 重置: `Ctrl+0`

**基础操作**
- 全屏切换: `F11`
- 退出 kitty: `Ctrl+Q`

**分屏/会话管理**: 使用 tmux 的快捷键
- 水平分屏: `Ctrl+B "` (tmux 默认)
- 垂直分屏: `Ctrl+B %` (tmux 默认)
- 窗口导航: `Ctrl+B 方向键` (tmux)
- 会话管理: `tmux` 命令
```

### UPDATE CLAUDE.md - Update development notes

- **IMPLEMENT**: 更新开发注意事项，移除 splits 相关内容
- **PATTERN**: CLAUDE.md:277-288
- **VALIDATE**: `grep "布局下正常工作" CLAUDE.md`

修改开发注意事项：
```markdown
## 开发注意事项

### 修改配置时

1. **键绑定冲突检测**: 使用 `kitty --debug-input` 检测按键符号
2. **配置验证**: 修改后使用 `Ctrl+Shift+D` 进行配置调试
3. **测试重载**: 使用 `Ctrl+Shift+F5` 热重载，无需重启 kitty
4. **避免与 tmux 冲突**: 不使用 `Ctrl+B` (tmux prefix) 和 `Ctrl+A` 相关组合键

### 扩展功能

- **自定义 kitten**: Python 脚本放在配置目录，通过 `kitten` 命令调用
- **正则匹配**: 修改 `custom-hints.py` 中的 `RE_*` 常量调整匹配规则
- **主题管理**: 使用 `kitten themes` 或编辑 `current-theme.conf`

### 设计原则

- **保持简洁**: kitty 只做终端该做的事
- **不重复造轮子**: tmux 已经解决的问题（分屏、会话）不在 kitty 中实现
- **发挥特长**: 充分利用 kitty 的 hints、主题、图形能力等特有功能
```

### VALIDATE - Test all preserved features

- **IMPLEMENT**: 手动测试所有保留的功能是否正常工作
- **VALIDATE**: 逐一验证每个功能

测试清单：
```bash
# 1. 配置重载
# 按 Ctrl+Shift+F5，应显示 "kitty config reloaded."

# 2. 配置编辑
# 按 Ctrl+Shift+,，应打开 nvim 编辑 kitty.conf

# 3. 调试配置
# 按 Ctrl+Shift+D，应显示配置调试信息

# 4. 智能提示
# 在有 URL 或路径的窗口按 Ctrl+Shift+H，应高亮可复制项

# 5. 主题切换
# 按 Ctrl+Shift+F1，应显示主题选择界面

# 6. 字体调整
# Ctrl+=, Ctrl+-, Ctrl+0 应正常调整字体大小

# 7. Diff 功能
# 在终端运行: kitty +kitten diff file1 file2
# 应显示 diff 界面

# 8. SSH kitten
# 在终端运行: kitty +kitten ssh user@host
# 应正常建立 SSH 连接

# 9. icat 图片查看
# 在终端运行: kitty +kitten icat image.png
# 应在终端内显示图片
```

### VALIDATE - Verify no tmux conflicts

- **IMPLEMENT**: 在 tmux 会话中测试 kitty，确认无键绑定冲突
- **VALIDATE**: `tmux list-keys | grep -E "(C-S-|C-q|F11|F3)"`

测试步骤：
```bash
# 1. 启动 kitty
# 2. 在 kitty 中启动 tmux
tmux new -s test

# 3. 测试 tmux 基础功能
# Ctrl+B " (水平分屏)
# Ctrl+B % (垂直分屏)
# Ctrl+B 方向键 (切换 pane)
# Ctrl+B d (detach)

# 4. 测试 kitty 功能不影响 tmux
# 在 tmux 会话中按 Ctrl+Shift+H (kitty hints)
# 在 tmux 会话中按 Ctrl+Shift+F1 (kitty theme)
# 应该 kitty 功能正常工作，tmux 不受影响

# 5. 清理测试会话
tmux kill-session -t test
```

### CLEANUP - Remove backup if satisfied

- **IMPLEMENT**: 如果测试通过，可选择性删除备份
- **GOTCHA**: 建议保留备份一段时间，确认无问题后再删除
- **VALIDATE**: `ls -la backup/`

```bash
# 可选：删除备份
# rm -rf backup/
# 或保留备份以备不时之需
```

---

## TESTING STRATEGY

### Manual Testing

由于这是配置重构，主要通过手动测试验证：

**1. 配置加载测试**
- 启动 kitty，观察是否有错误消息
- 运行 `kitty --debug-config` 检查配置解析

**2. 功能完整性测试**
- 逐一测试所有保留的键绑定
- 验证 hints、themes、diff、icat 等功能正常

**3. Tmux 集成测试**
- 在 kitty 中启动 tmux
- 测试 tmux 分屏、会话管理功能
- 确认 kitty 和 tmux 键绑定无冲突

**4. 长期使用测试**
- 日常使用 1-2 天
- 记录任何缺失的功能或不便之处
- 根据实际使用反馈微调

### Edge Cases

**1. 配置重载边界情况**
- 在有多个 kitty 窗口时重载配置
- 在 tmux 会话中重载 kitty 配置

**2. 主题切换**
- 切换主题后配置重载是否保持
- current-theme.conf 是否正确更新

**3. Hints 功能**
- 在复杂的终端输出中测试 hints
- 中文路径、特殊字符路径的识别

---

## VALIDATION COMMANDS

### Level 1: Configuration Syntax

```bash
# 检查配置文件语法
kitty --debug-config 2>&1 | grep -i error

# 验证 splits.conf 已删除
test ! -f splits.conf && echo "PASS: splits.conf removed" || echo "FAIL"

# 验证 zoom_toggle.py 已删除
test ! -f zoom_toggle.py && echo "PASS: zoom_toggle.py removed" || echo "FAIL"

# 检查 Ctrl+a 键绑定已全部移除
grep "^map ctrl+a>" kitty.conf && echo "FAIL: Ctrl+a bindings exist" || echo "PASS"

# 检查新键绑定已添加
grep "kitty_mod+f5" kitty.conf && echo "PASS: new bindings added" || echo "FAIL"
```

### Level 2: Functional Testing

```bash
# 启动 kitty 无错误
kitty --start-as=normal 2>&1 | grep -i error

# 测试配置重载（需要手动验证界面反馈）
# 在 kitty 中按 Ctrl+Shift+F5

# 测试主题切换（需要手动验证界面）
# 在 kitty 中按 Ctrl+Shift+F1

# 测试 hints 功能
# echo "https://example.com /path/to/file.txt"
# 然后按 Ctrl+Shift+H

# 测试 diff 功能
echo "test1" > /tmp/test1.txt
echo "test2" > /tmp/test2.txt
kitty +kitten diff /tmp/test1.txt /tmp/test2.txt
```

### Level 3: Integration Testing with tmux

```bash
# 启动 tmux 会话
tmux new -s validation_test

# 在 tmux 中测试分屏（应使用 tmux 功能）
# Ctrl+B " (水平分屏)
# Ctrl+B % (垂直分屏)

# 在 tmux 中测试 kitty hints（应正常工作）
# echo "https://example.com"
# Ctrl+Shift+H

# 退出测试会话
# tmux kill-session -t validation_test
```

### Level 4: Documentation Validation

```bash
# 验证 CLAUDE.md 已更新
grep "专门负责" CLAUDE.md && echo "PASS: CLAUDE.md updated" || echo "FAIL"

# 检查文档中不再提及已删除的功能
grep "splits.conf" CLAUDE.md && echo "WARN: splits.conf mentioned" || echo "PASS"
grep "zoom_toggle" CLAUDE.md && echo "WARN: zoom_toggle mentioned" || echo "PASS"
```

---

## ACCEPTANCE CRITERIA

- [ ] splits.conf 文件已完全删除
- [ ] zoom_toggle.py 文件已完全删除
- [ ] kitty.conf 中移除了对 splits.conf 的引用
- [ ] 所有 `Ctrl+a` 前缀键绑定已移除
- [ ] 会话管理配置已禁用（startup_session none）
- [ ] 布局系统已简化（移除 splits 布局）
- [ ] 保留功能的新键绑定已添加并正常工作
- [ ] kitty 配置加载无错误
- [ ] Kitty 特有功能（hints、themes、diff、icat）工作正常
- [ ] 在 tmux 会话中使用 kitty 无键绑定冲突
- [ ] CLAUDE.md 文档已更新，准确反映新架构
- [ ] 配置文件包含清晰的注释说明设计哲学
- [ ] 所有验证命令执行通过

---

## COMPLETION CHECKLIST

- [ ] 备份已创建
- [ ] splits.conf 已删除
- [ ] zoom_toggle.py 已删除
- [ ] kitty.conf 已更新（移除 tmux 功能）
- [ ] kitty.conf 已更新（添加新键绑定）
- [ ] kitty.conf 已添加配置说明注释
- [ ] 配置语法验证通过
- [ ] 手动功能测试全部通过
- [ ] Tmux 集成测试无冲突
- [ ] CLAUDE.md 项目概览已更新
- [ ] CLAUDE.md 架构说明已更新
- [ ] CLAUDE.md 键绑定说明已更新
- [ ] CLAUDE.md 开发注意事项已更新
- [ ] 所有验证命令执行成功
- [ ] 日常使用测试无问题（建议至少 1-2 天）

---

## NOTES

### Design Rationale

**为什么移除分屏管理？**
Tmux 的分屏管理更成熟、更强大（支持嵌套布局、面板缩放、面板同步等）。Kitty 的 splits 布局功能有限，维护两套分屏系统只会增加复杂度。

**为什么保留 shell_integration？**
Shell integration 提供的功能（命令输出标记、提示符检测、CWD 追踪）不与 tmux 冲突，且对终端体验有实质提升。即使在 tmux 中，这些功能依然有用。

**为什么保留 allow_remote_control？**
远程控制能力是 kitty 的扩展接口，支持脚本化控制终端。这不与 tmux 功能重复，且可用于自定义工作流（如从外部脚本打开新 kitty 窗口、调整字体等）。

**新键绑定选择原则**：
1. 使用 `kitty_mod` (Ctrl+Shift) 作为主要修饰键，与大多数终端应用兼容
2. 避免 `Ctrl+B` (tmux prefix) 和 `Ctrl+A` (备选 tmux prefix)
3. 功能键 (F1-F12) 用于次要功能，避免占用常用快捷键
4. 保持简洁，只为真正需要的功能分配快捷键

### Potential Issues

**1. 肌肉记忆调整**
如果用户已习惯 oh-my-kitty 原有的 `Ctrl+a` 键绑定，需要一段时间适应新的键绑定或纯 tmux 工作流。

**解决方案**: 保留备份配置，可以随时切换回来。同时建议逐步过渡，先禁用部分功能，适应后再完全移除。

**2. 会话恢复功能丢失**
移除 kitty 会话管理后，依赖 `startup_session` 的用户需要改用 tmux 会话管理。

**解决方案**: 使用 `tmux attach` 或 `tmux new -A -s session_name` 在 shell 启动脚本中实现类似功能。或使用 tmux 插件如 tmux-resurrect、tmux-continuum。

**3. 标签页快速跳转**
移除 `Ctrl+a>1-0` 后，快速跳转到特定标签页需要改用 tmux 窗口跳转。

**解决方案**: 在 tmux 配置中设置窗口快速跳转（tmux 默认支持 `prefix+0-9`）。

### Migration Path

如果用户暂时不想完全移除某些功能，可以分阶段迁移：

**阶段 1**: 仅移除分屏管理（splits.conf, zoom_toggle.py），保留标签页管理
**阶段 2**: 移除标签页管理的 Ctrl+a 绑定，改用 kitty 默认绑定
**阶段 3**: 移除会话管理，完全使用 tmux
**阶段 4**: 完全清理，只保留 kitty 特有功能

### Future Enhancements

**1. 集成 tmux 和 kitty 的剪贴板**
可以配置 tmux 和 kitty 共享剪贴板，实现无缝的复制粘贴体验。

**2. 自定义 hints 规则**
根据实际工作流，在 custom-hints.py 中添加更多匹配规则（如 Git SHA、Issue ID、Docker 容器 ID 等）。

**3. 主题同步**
可以创建脚本同步 kitty 主题和 tmux 主题，保持视觉一致性。

**4. Kitty + Tmux 启动脚本**
创建启动脚本自动启动 kitty 并 attach 到 tmux 会话，简化工作流。

<!-- EOF -->
