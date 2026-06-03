# Claude Skill 字段速查

Claude Code 的 `SKILL.md` YAML frontmatter 字段速查表，附带 Notepad++ 语法高亮和自动补全文件。

## 文件说明

| 文件 | 说明 |
|------|------|
| `Claude-Skill-字段速查.md` | 字段完整速查表，含实测结论、使用格式、注意事项 |
| `Claude-Skill_zh-CN.xml` | Notepad++ **语法高亮**文件（User Define Language） |
| `Claude-Skill-zh-CN.xml` | Notepad++ **自动补全**文件（AutoComplete） |

## 速查表包含

- **21 个字段**的完整实测结论（✅ 17 / ⚠️ 2 / ❓ 1 / ❌ 1）
- 所有字段的 YAML 写法示例和注意事项
- `dependencies` 字段的 AI 驱动安装机制说明
- `paths` 字段的 gate 条件行为和 glob 模式速查
- `allowed-tools` / `disallowed-tools` 误区澄清
- 三级加载机制、完整 SKILL.md 结构、推荐模板

## Notepad++ 配置安装

语法高亮和自动补全文件放到 Notepad++ 安装目录下：

```
# 语法高亮
Notepad++\userDefineLangs\Claude-Skill_zh-CN.xml

# 自动补全
Notepad++\autoCompletion\Claude-Skill-zh-CN.xml
```

重启 Notepad++，打开 `SKILL.md` 即可看到语法高亮和自动补全效果。

## 数据来源

- 🔵 官方确认 — [code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)
- 🟢 开放标准 — [agentskills.io](https://agentskills.io)
- ⚠️ 社区使用 — 官方未确认，实测验证
