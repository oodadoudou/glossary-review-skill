# glossary-review

小说术语表 AI 审查工具 — Claude Code Skill。

针对韩中 / 日中翻译场景，逐词审查 `.xlsx` 术语表，结合小说背景与参考原文，对每个术语执行网络搜索 + 上下文一致性分析，输出修改/删除提案供用户确认后覆盖写回。

## 功能特性

- **逐词审查**：对术语表中每个词条独立执行网络搜索与上下文分析，不做批量跳过
- **一致性扫盘**：审查完毕后全局横扫，确保同一角色/组织的不同指代形式译名同源
- **Tier 分级**：按词频与背景出现情况分为 S/A/B/C 四级，精准控制删改力度
- **交互式确认**：所有变更以提案形式展示，用户逐条确认或质疑后才写回
- **双语支持**：韩语（默认）/ 日语，可自动检测

## 前置要求

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI 或桌面端
- 目标目录中需包含：
  - `glossary_output_final.xlsx` — 待审查的术语表（或任意 `.xlsx`）
  - `output_detail.txt` — 参考原文（必需）
  - `background.txt` — 小说背景描述（缺失时会提示输入并自动创建）

## 安装

将 `SKILL.md` 复制到 Claude Code 的 skill 目录：

```bash
# 用户级安装（所有项目可用）
cp SKILL.md ~/.claude/skills/glossary-review.md

# 或项目级安装（仅当前项目可用）
cp SKILL.md .claude/skills/glossary-review.md
```

## 使用方法

在 Claude Code 中输入：

```
/glossary-review
```

### 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `目录路径` | 术语表所在目录 | 当前对话上下文中 `@` 引用的目录 |
| `--lang ko` | 韩语模式 | 默认 |
| `--lang ja` | 日语模式 | - |
| `--lang auto` | 自动检测源语言 | - |

### 示例

```bash
# 当前目录，韩语（默认）
/glossary-review

# 指定目录，日语
/glossary-review ./novels/some_novel --lang ja

# 自动检测语言
/glossary-review --lang auto
```

## 工作流程

```
Phase 1  文件发现与初始化
  ↓      定位 xlsx、加载 background.txt 与 output_detail.txt
Phase 2  语言判定
  ↓      根据参数或自动检测确定源语言（ko/ja）
Phase 3  加载与分组
  ↓      解析术语表，按分类 + 词频排序
Phase 4  逐词审查
  ↓      每个术语：网络搜索 → Tier 判定 → 规则校验 → 生成建议
Phase 5  全局一致性扫盘
  ↓      角色组 / 组织组聚类，修正不一致译名
Phase 6  提案输出
  ↓      Markdown 表格 + review_proposal.md 详细报告
Phase 7  用户反馈循环
  ↓      逐条确认 / 质疑 / 调整，直至用户满意
Phase 8  写回 xlsx
         覆盖原文件，保留所有列与数据类型
```

## 术语表格式

xlsx 列结构按位置 + 关键词自动识别：

| 列 | 识别规则 | 说明 |
|----|----------|------|
| 第 1 列 | `src` | 原文 |
| 第 2 列 | `dst` | 译文 |
| `info` / `备注` | 分类 | 术语分类标签 |
| `regex` | 正则 | 保留不动 |
| `frequency` / `次数` / `freq` / `count` | 词频 | 用于 Tier 判定 |
| 其他列 | — | 原样保留 |

## 审查规则概要

- **人名一致性**：同一角色的全名/省略名/昵称中文译名同源，组内性别一致
- **组织一致性**：全称/简称/俗称中译同源
- **精简原则**：通用词、助词、无歧义日常词建议删除
- **网文语境优先**：译名符合现代中文网络文学习惯
- **分类标准化**：统一为"大类/子类"格式

详细规则参见 [SKILL.md](./SKILL.md)。

## 输出文件

| 文件 | 说明 |
|------|------|
| `review_proposal.md` | 完整审查提案（含理由、搜索结果、上下文引用） |
| `glossary_output_final.xlsx` | 用户确认后覆盖写回的术语表 |

## 注意事项

- 单轮审查 + 用户确认即终止，不做多轮自动迭代
- 写回时直接覆盖原文件，不生成额外变更日志
- 大型术语表（>200 条）会分组给出阶段性进度提示
- xlsx 被 Excel 占用时会提示关闭后重试

## License

MIT
