# x-to-obsidian-auto-categorize

用 Claude Code 自动拉取、总结、分类你的 X/Twitter 书签，写入 Obsidian 知识库。

[English](README.md) | 中文

## 它做什么

```
X 书签 → feedgrab 自动拉取 → AI 生成结构化 Obsidian 笔记 → 自动分流到领域目录
```

三阶段流水线：

| 阶段 | 内容 | 是否必须 |
|------|------|---------|
| **Phase 0** | 通过 feedgrab 拉取 X 书签 | 可选——也可手动粘贴 URL |
| **Phase 1** | 逐条抓取推文内容，生成结构化 Obsidian 笔记 | 核心 |
| **Phase 2** | 将笔记从收件箱自动分流到各领域目录 | 核心 |

每条笔记包含：核心命题、来源背景、内容摘要、核心要点、深度提炼、行动建议、关联笔记——全部使用 Obsidian 原生格式（callout + wikilink）。

feedgrab 支持单条推文、Thread、引用推文（QT）和 X Article，完整抓取全部内容。

## 前置要求

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)（CLI）
- [feedgrab](https://github.com/iBigQiang/feedgrab)——通用内容抓取工具（Setup 向导自动安装）
- [browser-cookie3](https://pypi.org/project/browser-cookie3/)——从 Chrome 提取 X 认证 cookie（自动安装）
- Chrome 已登录 x.com（认证用）

## 快速开始

### 1. 安装 x-to-obsidian-auto-categorize

克隆到 Claude Code skills 目录：

```bash
# 放到你的 vault skills 目录
git clone https://github.com/scratchy1234/x-to-obsidian ~/my-vault/.agents/skills/x-to-obsidian

# 或独立存放
git clone https://github.com/scratchy1234/x-to-obsidian ~/x-to-obsidian
```

### 2. 运行——Setup 向导自动启动

直接调用 skill，首次运行会自动进入配置向导：

```
/x-to-obsidian-auto-categorize
```

向导会引导你完成：
1. **Vault 路径**——你的 Obsidian vault 在哪？
2. **收件箱目录**——新笔记放哪个文件夹？
3. **领域目录**——哪些目录用于扫描关联笔记和分流？
4. **你的视角**——决定每条笔记的"为什么值得关注"和行动建议
5. **feedgrab + browser-cookie3**——未安装则自动安装
6. **X 认证**——自动从 Chrome 提取 cookie（auto / manual 两种方式）
7. **前缀系统**——使用默认 8 种前缀，或自定义分流规则

向导会验证输入、创建缺失目录、生成 `config.yaml` 和 `prefix-rules.md`，并可选运行测试。

### 3. 日常使用

配置完成后，同样的命令：

```
/x-to-obsidian-auto-categorize
```

或指定模式：

```
/x-to-obsidian-auto-categorize summarize only     # 只处理 Phase 0+1，不分流
/x-to-obsidian-auto-categorize distribute only    # 只做 Phase 2 分流
/x-to-obsidian-auto-categorize skip bookmarks     # Phase 1+2，手动填 URL
```

### 4. （可选）定时任务

通过 scheduled task 每天自动运行，参考 `scheduled-task/SKILL.md`。

## 工作原理

### Phase 0：书签抓取

feedgrab 执行：
1. 用你的 X session cookie 拉取 `x.com/i/bookmarks`
2. 解析所有书签 `.md` 文件
3. 对比 `processed_ids.json` 去重
4. 将新 URL 追加到 `{inbox}/Twitter-URLs.md`

**不想自动拉取？** 手动往文件里粘贴 URL 即可：`- 2026-03-22 [标题](https://x.com/user/status/123)`

> **关于认证**：X 会拒绝 `feedgrab login twitter` 的自动登录。Setup 向导通过 browser-cookie3 直接从 Chrome 本地存储提取 cookie，无需手动复制 token。

### Phase 1：抓取 & 总结

每条 URL 使用两级降级链：

```
feedgrab（独立 tmpdir + FORCE_REFETCH）→ WebFetch（兜底）
```

feedgrab 输出结构化 Markdown，包含完整 Thread 内容、引用推文和丰富的 frontmatter（作者、发布时间、互动数据）。每条 URL 使用独立临时目录，绕过 feedgrab 内部的去重缓存。

生成的 Obsidian 笔记包含：
- **核心命题**——作者的核心主张 + 为什么值得你关注（基于你的视角）
- **来源背景**——作者背景、发布时间
- **内容摘要**——2-4 句概述；有 QT 时额外用 `[!info]` callout 说明
- **核心要点**——`[!tip]` callout，按逻辑重要性降序排列
- **深度提炼**——`[!success]` 主张 + 论据组合
- **触发的想法**——`[!example]` 具体下一步行动，关联你的活跃项目
- **已有关联**——自动从 vault 索引缓存中匹配的 wikilink

### Phase 2：自动分流

读取每条笔记的领域前缀和内容，按 `prefix-rules.md` 路由到对应目录：

- 更新 frontmatter（`status: inbox` → `status: 未开始`）
- 同步更新来源和目标的索引文件
- 自动创建缺失目录和索引

## 项目结构

```
x-to-obsidian/
├── SKILL.md                    # 主 skill——流水线调度器
├── config/
│   ├── config.example.yaml     # 路径配置模板
│   ├── prefix-rules.example.md # 前缀和分流规则模板
│   └── note-template.example.md # 笔记模板自定义说明
├── templates/
│   ├── Twitter_Note.md         # 默认笔记模板
│   └── Index_Template.md       # 目录索引模板
├── examples/
│   └── sample-note.md          # 生成笔记示例
├── scheduled-task/
│   └── SKILL.md                # 每日自动运行任务模板
└── docs/
    ├── setup.md                # 详细安装指南
    └── customization.md        # 自定义教程
```

## 常见问题

**Q：feedgrab login twitter 失败怎么办？**
A：X 会拒绝自动登录。Setup 向导通过 browser-cookie3 从 Chrome 提取 cookie 解决这个问题。确保 Chrome 已打开并登录 x.com，然后在向导里选择 "auto" 认证方式。

**Q：我不想自动拉书签，可以关闭吗？**
A：可以。在 `config/config.yaml` 里设置 `phase0_enabled: false`，然后手动往 `{inbox}/Twitter-URLs.md` 加 URL，用 "skip bookmarks" 模式运行。

**Q：支持 Thread 和引用推文吗？**
A：支持。feedgrab 可以完整抓取 Thread 内容和引用推文。有 QT 时生成的笔记会包含 QT 摘要 callout。

**Q：可以添加新的前缀类型吗？**
A：可以。编辑 `config/prefix-rules.md`，按现有格式新增一段，更新分流规则即可。

**Q：支持非 Twitter 的 URL 吗？**
A：feedgrab 支持任意 URL，内置 Jina 兜底。笔记模板针对推文优化，但对一般网页内容同样适用。

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=scratchy1234/x-to-obsidian-auto-categorize&type=Date)](https://star-history.com/#scratchy1234/x-to-obsidian-auto-categorize&Date)

## License

MIT
