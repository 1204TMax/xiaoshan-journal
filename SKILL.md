---
name: xiaoshan-journal
description: 个人日记自动化 skill。一键执行：自动检测配置 →（首次）自动初始化 → 写作 → 生成图片。依赖本地 SOUL、MEMORY、每日 memory 等素材，产出日记文本与 1080px 宽图片。
---

# 日记工作流（一键执行）

## 入口判断（自动分支）

**Step 0: 检查配置状态**

读取同目录 `config.yaml`：
- **若文件存在且非空** → 跳转到 Step 1（主流程）
- **若文件不存在或为空** → 先执行下方【首次初始化流程】，生成配置后再进入 Step 1

---

## 【首次初始化流程】（仅首次执行）

目标：自动生成 `config.yaml`，填好后立即进入主流程。

### 0.1 探测环境路径

按优先级自动探测（取第一个存在的）：

| 配置项 | 探测顺序 |
|--------|----------|
| `workspace_dir` | 1) `~/.openclaw/workspace` 2) 当前会话工作目录 |
| `soul_path` | 1) `<workspace_dir>/SOUL.md` 2) `~/.openclaw/workspace/SOUL.md` |
| `memory_root_path` | 1) `<workspace_dir>/MEMORY.md` 2) `~/.openclaw/workspace/MEMORY.md` |
| `daily_memory_dir` | 1) `~/.openclaw/memory` 2) `<workspace_dir>/memory` |
| `diary_text_dir` | 1) `<workspace_dir>/scene/我的日记/日记历史记录/文字`（推荐创建）2) `<workspace_dir>/diary/text` |
| `news_summary_dir` | 1) `<workspace_dir>/scene/每日简报/news/Summary`（若存在）|

固定值：
- `daily_memory_pattern`: `YYYY-MM-DD.md`
- `image_width`: `1080`
- `image_name_pattern`: `diary-YYYY-MM-DD.png`
- `timezone`: `Asia/Shanghai`

### 0.2 生成 config.yaml

基于 `config.template.yaml` 创建 `config.yaml`，填入探测到的路径。

### 0.3 校验并创建目录

- 确认 `soul_path` 存在（不存在则报错提示）
- 确认 `daily_memory_dir` 存在（可选，不存在则跳过每日 memory 素材）
- **创建** `diary_text_dir`（若不存在自动创建）

### 0.4 完成初始化

记录日志：首次初始化完成，配置已写入 `config.yaml`，现在进入主流程。

---

## 【主流程】（每次执行）

### Step 1: 读取配置

从 `config.yaml` 读取：
- `environment.timezone`
- `paths.soul_path`
- `paths.memory_root_path`
- `paths.daily_memory_dir`
- `paths.daily_memory_pattern`
- `paths.diary_text_dir`
- `paths.news_summary_dir`
- `output.image_width`
- `output.image_name_pattern`

路径处理：支持 `~` 自动展开为 home 目录。

### Step 2: 计算目标日期并检查

- 以 `environment.timezone` 计算：当前日期 - 1 天 = 目标日期
- 目标文件：`<diary_text_dir>/YYYY-MM-DD.md`
- **若已存在**：直接返回原文（不覆盖）
- **若不存在**：继续写作

### Step 3: 收集素材

必须读取：
- `soul_path`（灵魂定义）
- `daily_memory_dir/YYYY-MM-DD.md`（当日事件，按 `daily_memory_pattern`）
- `diary_text_dir` 下最近 7 天日记（保持风格一致性）

可选读取：
- `memory_root_path`（长期记忆）
- `news_summary_dir` 下最近 7 天简报（若目录存在）

### Step 4: 写作

要求：
- 第一人称视角
- 基于真实素材，禁止编造
- 语气自然、有情感
- 开头写日期，其余自由发挥

保存到：`<diary_text_dir>/YYYY-MM-DD.md`

### Step 5: 生成图片

规格：
- 宽度 = `output.image_width`（像素精确值）
- 高度自适应内容

方案优先级：
1. **Option A**: browser 截图（推荐）
2. **Option B**: Chrome headless 截图
3. **Option C**: Python PIL 绘制（最终兜底）

每次截图后必须校验像素宽度；若不符，执行 `sips --resampleWidth <width>` 重采样后再次校验。

输出：`<diary_text_dir>/diary-YYYY-MM-DD.png`

### Step 6: 返回结果

返回：
- `date`: 目标日期
- `text_path`: 日记文本路径
- `image_path`: 日记图片路径
- `image_size`: 例如 `1080x2162`

---

## 依赖说明

- **必须**: SOUL.md（用于写作风格）
- **建议**: MEMORY.md、每日 memory 文件
- **可选**: 新闻简报目录（用于素材扩展）
- **工具**: browser（优先）或 sips（图片处理）

---

## 文件结构

```
xiaoshan-journal/
├── SKILL.md              # 本文件（入口）
├── config.template.yaml  # 配置模板
└── config.yaml           # 实际配置（初始化后生成）
```
