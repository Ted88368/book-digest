# Book Digest —— 本地知识中枢

[English](./README_EN.md)

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)
[![Language: Python](https://img.shields.io/badge/Language-Python-blue.svg)](https://www.python.org/)

> 基于 [the-knowledge-guy](https://github.com/Ted88368/the-knowledge-guy)
> 
> 把 `raw/` 里的任意 PDF 或 EPUB 变成结构化的两层 AI 技能 —— 然后在整个书架上提问、学习。

<p align="center">
  <img src="docs/hero-pipeline.png" alt="管线示意图：/book-to-skill 通过五个 map-reduce 阶段把 PDF 摄入为一个两层 Claude Code 技能；/the-knowledge-guy 将任意问题路由到所有已安装的技能，同时写出一条聊天回复和一个 HTML 产物。" width="900">
</p>

*从磁盘上的 PDF 到可查询的知识技能，一条命令；
从一个问题到跨领域的答案，另一条。*

---

## 这是什么

一本长书有 40 万以上 token；读到结尾你已忘掉中间；跨三本书提问则根本做不到。
现有工具要么把一切塞进上下文（昂贵、健忘），要么手写摘要（有损、立刻过期）。

`/book-to-skill` 是一条 **map-reduce 摄入管线**，把 PDF 或 EPUB 变成一个
两层 Claude Code 技能：一份常驻加载的概念地图（约 3K token —— 论点、
6-10 个承重框架、章节与主题索引）加按需调用的章节工具箱（每份约 1K token
—— 框架、技法、反模式、实例）。一本 600 页的书变成一个咨询成本几千 token
而非四十万 token 的技能。

`/the-knowledge-guy` 是跨所有已安装技能的**路由器与互动教师**。它在每次
调用时从文件系统自动发现技能，把任意问题并行路由到每一本可能相关的书，
并综合出带内联引用的一个答案。它可以带测验、可恢复进度地一步步教你一个
主题 —— 也可以把一个章节变成一个交互式**边做边学**网站：带可操作 SVG
插图的理论、自动判分的测验、浏览器内代码实验，以及按评分量表判分的
开放任务。

把它拧在一起的架构规则：**Python 负责管道，Claude 负责智能。**
`extract.py` 只做机械的 PDF → 文本 + 切片 + 图片清单，仅此而已。
每一次理解 —— 框架提取、概念映射、综合、出题 —— 都是一次 LLM 调用。

两个技能都遵循 Anthropic 的 `SKILL.md` 开放标准 —— 被 Claude Code、
Claude Desktop、claude.ai、Claude API、OpenAI Codex CLI 和 GitHub Copilot
原生消费。见下文 *在你的平台上安装*。

## 快速上手

```bash
/book-to-skill /path/to/book.pdf                                  # 摄入（加 --course → 完整覆盖 + 练习）
/the-knowledge-guy what do my books say about margin of safety?   # 提问
/the-knowledge-guy walk me through Kerberos                       # 教学
/the-knowledge-guy course <skill-slug>                            # 边做边学
/the-knowledge-guy nutshell <skill-slug>                          # 略读
/the-knowledge-guy resume                                         # 继续
```

每次调用还会向 `artifacts/` 写一个自包含的 HTML 产物。课程页是交互式的
—— 直接在浏览器中运行。

## 系统由两部分构成

### `/book-to-skill` —— 摄入管线

六个阶段的 map-reduce（第六个 **Stage 3 PRACTICE** 为可选），外加可选的
逐章**覆盖度审计**。值得知道的要点：

- **两层输出。** `SKILL.md` 常驻加载；`chapters/<book_number>-<slug>.md`
  按需调入。第一层文件前重后轻，上下文压缩时能安全保留开头。
- **`book_number` 是规范的章节标签** —— `ch07`、`intro`、`appendix-a`、
  `part-1`、`fm`、`bm`。用书自身的章节编号，绝不用抽取顺序。manifest 另带
  一个内部 `index` —— 永不面向用户。
- **`schema_version: 2`** 加三个幂等的辅助脚本，让旧技能可以就地升级，
  无需重新抽取。
- **恢复靠文件系统驱动。** 对一次半成品运行重跑永远安全；`chapters/` 里
  的章节文件就是检查点。
- **`raw/` 随技能保留** —— 全文、切片、图片、元数据、Pass-0 脊柱。
  重新抽取一个章节从不重跑 Stage 0。
- **七种体裁画像**（技术 / 漏洞挖掘 / 金融 / 科学 / 效率 / 叙事非虚构 /
  通用）调校分块边界、章节 schema 与 reduce 的侧重。
- **完整覆盖模式（可选）** 用"捕获每章每一个承重元素"替代默认的密集但
  限量工具箱，然后跑一次逐章**覆盖度审计**，凡有缺漏的章节重跑直到越过
  95% 门槛 —— 保证没有任何内容被静默丢弃。（CS:APP 就是这样摄入的：
  130 个 section，逐个审计到 100%。）
- **Stage 3 PRACTICE（可选）** 把每个章节变成一个练习集 —— 它提取书自带的
  练习、生成新练习，对安全/技术章节还能联网调研出真实感实验。产物
  （`practice/<book_number>-<slug>.json`）就是 `/the-knowledge-guy
  course` 渲染成交互式边做边学网站的内容。最适合技术 / 教材 / 漏洞挖掘
  类书籍。
- **选项 —— 标志或交互。** 传 `--complete`（完整覆盖）、`--practice`
  （Stage 3）、`--course`（两者 → 直接可出课程）或 `--regenerate`（重建
  已有技能）—— 或者都不传，book-to-skill 会逐项询问，且*在花费前给出
  成本估算*。相同标志也可穿过路由器：`/the-knowledge-guy <path>.pdf --course`。

### `/the-knowledge-guy` —— 路由器 + 教师

自动发现，无注册表。每次调用读取 `.claude/skills/*/SKILL.md` 的 frontmatter；
放一个技能进来，路由器下次调用就会捡起。每次都把自己和 `book-to-skill`
排除在路由之外。

| 模式 | 做什么 | 触发方式 |
| ---- | ------ | ------- |
| **ask** | 带内联引用的跨领域综合长文 | 开放式问题 |
| **walk** | 带测验的互动课程，进度跨会话保存 | `walk me through <topic>` |
| **course** | 每章一个交互式边做边学网站 —— 理论 + 自动判分测验 + 浏览器内代码实验 + Claude 判分的开放任务 | `course <book> [<chapter>]` |
| **check** | 按量表判分一个开放式练习答案 | `check <book> <ch> <id>` |
| **nutshell** | 整书逐章略读（每章约 100 词） | `nutshell <book>` |
| **library** | 书架总览 | `library` |
| **comparison** | 一个概念跨多本书，标注 一致 / 延伸 / 张力 | `compare <topic>` |
| **cheatsheet** | 每书一页操作速查 | `cheatsheet <book>` |
| **glossary** | A-Z 术语查询，按书或跨全库 | `glossary [<book>]` |
| **concept-map** | 一本书的第一层框架图 | `concept-map <book>` |
| **toolkit** | 对一个章节的第二层深挖 | `toolkit <book> <chapter>` |
| **ingest** | 把 PDF/EPUB 移交给 `book-to-skill` | `add <path>.pdf` |
| **resume** | 接续一次中断的 walk | `resume` |

ask 模式自己从不读领域 `SKILL.md` —— 它只读 40 行的 frontmatter 用于路由，
然后为每个匹配技能扇出一个并行 subagent。每个 subagent 恰好加载一本书，
用 200-400 词带章节引用作答。编排者把这些报告综合成一篇统一长文。

### 边做边学 —— `course` 体验

对技术、教材与漏洞挖掘类书籍，阅读只是学习的一半。
`/the-knowledge-guy course <book>` 为每章渲染一个交互式网站（外加一份
课程大纲索引），既教又让你练。整个闭环都在一个可从磁盘直接打开的
自包含 HTML 页面里：

- **理论** —— 用从业者口吻讲这个章节，由设计系统组件拼成，可选配一个
  **交互式 SVG 概念部件**：切换一个 sanitizer，看着污点传播止于此处；把
  写入长度拖过缓冲区容量，看它转为 critical；逐步推进一条管线；对比两种
  做法。五种部件类型（`flow`、`toggle-state`、`stepper`、`slider`、
  `compare`），全部由现有 `.plate`/`.illus` 类构成，因此暗色模式自动反转，
  动效受 `prefers-reduced-motion` 约束。教学 subagent 只在概念确实结构性
  时才加一个 —— 默认纯文字。
- **练习** —— 自动判分的测验（单选、预测输出、修 bug、填空、重排步骤、
  找反模式）即时反馈；**浏览器内可运行的代码实验**（JavaScript 原生支持；
  Python 走 Pyodide）在沙箱 iframe 中执行学习者代码并跑确定性检查；以及
  开放式任务。
- **判分 + 进度** —— 开放任务的 *"Check with Claude"* 按钮会复制一条
  `check …` 命令；粘回来，Claude 按该任务的量表判分并记录结果。测验/实验
  进度存进浏览器 `localStorage`；持久的 `course-<slug>.md` 记忆文件才是
  事实源，索引为每章显示一个掌握度仪表。

练习内容来自 `book-to-skill` 的可选 **Stage 3 PRACTICE**：它提取书自带的
练习、生成新练习，并可为实验做联网调研（`practice/<book_number>-<slug>.json`）。
两个 lint 守护它：`lint_practice.py` *执行* 每个实验，证明其解答能过、
其起点会挂；`lint_concept_widgets.py` 校验每个部件的 schema，并拒绝任何
硬编码颜色的定制 SVG（引擎自身从不设置颜色）。

## 每个输出同时也是 HTML 产物

每次调用都向聊天写文本*并*向 `artifacts/` 写一个自包含 HTML 文件，
使用共享设计系统（"Knowledge Guide · Modern" —— Bricolage Grotesque +
JetBrains Mono，单一 cobalt 强调色，亮/暗 + 密度开关持久化在
`localStorage`）。目录 `artifacts/index.html` 在每次写入时自动更新。

```
artifacts/
├── index.html                              ← 自动更新的目录
├── library.html                            ← 书架总览
├── nutshells/<book-slug>.html              ← 缓存，确定性
├── synthesis/YYYY-MM-DD-<query-slug>.html  ← 带日期，永不复用
├── walks/<topic>-step-<N>.html             ← 每步覆盖
├── walks/<topic>-recap.html                ← 持久保留
├── courses/<book-slug>/index.html          ← 课程大纲，重新生成
├── courses/<book-slug>/<book_number>.html  ← 交互课程页，缓存
└── comparisons/  toolkits/  cheatsheets/
    concept-maps/  glossaries/              ← 按 slug 缓存
```

确定性输出（nutshell、toolkit、cheatsheet、concept-map、按书 glossary、
library、课程页）被缓存复用；非确定性输出（synthesis、comparison、
walk-recap）以带日期的文件累积。设计系统位于
[`.claude/skills/the-knowledge-guy/design-system/`](./.claude/skills/the-knowledge-guy/design-system/)
—— `shell.html`（带静态 CSS 与两个受保护的引擎：练习**实验引擎**与
**概念部件引擎**）、`layouts.md`、`widgets.md`（部件 schema）、以及
`reference/full-demo-light.html` 里的完整视觉契约。

## 诚实的局限

- 针对 50–500 页的**技术或非虚构散文**优化。菜谱、参考手册和密集数学
  教材抽出来的效果较差。
- Stage 0 依赖 PyMuPDF；无 OCR 层的扫描 PDF 退回纯文本（无图）。
- 图片密集的书在抽取时额外费钱 —— 管线对每张保留的图都发一次 vision 调用。
- **course 模式**需要现代浏览器打开生成的页面。测验和 JavaScript 实验
  完全离线可跑；**Python** 实验首次 Run 时从 CDN 拉 Pyodide，离线则退回
  "check with Claude"。开放式任务回到聊天里判分，不在页面里。练习生成
  （Stage 3）最适合技术 / 教材 / 漏洞挖掘类书籍；叙事与金融类书籍得到的是
  测验和反思任务，而非代码实验。

## 本地模型 / 低并发环境

两个技能在进行扇出处理（摄入各阶段、多技能解答、多步骤教学）时都会启动子代理。默认批次宽度为 6 个并行子代理 —— 云端后端运行良好，但在本地大模型（LM Studio、Ollama 等）环境下，每个子代理都是长多轮循环，容易导致超时或并发过载。提供两档控制手段：

- **项目级** —— 在项目根目录下创建 `.claude/kg-settings.json`（已被 git 忽略；按项目独立配置）：

  ```json
  {"max_concurrency": 2}
  ```

  该项目中的所有扇出都会以最多 2 个一组进行分批，整批执行完成后才启动下一批。

- **单次运行** —— 在调用命令后传入 `--serial`（严格单线程执行）或 `--concurrency <N>`；命令行标志优先于配置文件：

  ```
  /book-to-skill ./book.pdf --serial
  /the-knowledge-guy add ./book.pdf --concurrency 2
  ```

  无配置文件且无标志时 → 保持默认行为不变。

## 设计原则

1. **自动发现优于配置。**
2. **每技能两层 —— 第一层常驻加载，第二层按需调入。**
3. **Python 负责管道，Claude 负责智能。**
4. **默认并行，预算受限** —— 扇出按 ≤ `MAX_CONCURRENCY` 分批执行（默认 6；可通过 `--serial` / `--concurrency <N>` 或 `.claude/kg-settings.json` 覆盖）。
5. **raw 随技能保留** —— 重新抽取从不重跑 Stage 0。
6. **文件系统驱动的恢复；幂等的升级脚本。**

## 仓库结构

```
the-knowledge-guy/
├── README.md · CLAUDE.md
├── artifacts/                  ← 所有 HTML 输出落在这里
└── .claude/skills/
    ├── book-to-skill/
    │   ├── SKILL.md            ← 管线运行手册（Stage 0-3）
    │   ├── reference/          ← 模板、体裁画像、concept-map 规格、
    │   │                         practice-template.md（Stage 3 契约）
    │   └── scripts/            ← extract.py · detect_chapters.py · lint_chapters.py
    │                             lint_practice.py · lint_concept_widgets.py
    │                             backfill_book_numbers.py · relabel_nutshell.py
    │                             upgrade_walk_memory.py · upgrade_course_memory.py
    ├── the-knowledge-guy/
    │   ├── SKILL.md            ← 模式分发 + 全部 13 个模式（含 course / check）
    │   ├── walk-mode.md        ← 互动课程 + 测验 + 课程记忆
    │   └── design-system/      ← shell.html（+ 实验与部件引擎）· layouts.md
    │                             widgets.md · reference/
    └── <book-derived skills>/  ← 每本摄入的书一个（+ 可选 practice/）
```

## 在你的平台上安装

这个仓库带**两个技能** —— `book-to-skill` 和 `the-knowledge-guy`。两个都
必须安装。生成的图书技能（运行 `/book-to-skill` 的产物）与它们并排存放，
路由器会自动捡起。每个技能的 Python venv 需要
[`uv`](https://docs.astral.sh/uv/)（PyMuPDF + ebooklib + beautifulsoup4 + pypdf）。

### 通用安装 —— 克隆一次，到处软链

一步覆盖 Claude Code + Claude Desktop：

```bash
git clone https://github.com/Ted88368/the-knowledge-guy.git ~/the-knowledge-guy
mkdir -p ~/.claude/skills
ln -s ~/the-knowledge-guy/.claude/skills/book-to-skill      ~/.claude/skills/
ln -s ~/the-knowledge-guy/.claude/skills/the-knowledge-guy  ~/.claude/skills/
~/the-knowledge-guy/.claude/skills/book-to-skill/scripts/setup.sh
```

软链（而非拷贝）意味着 `git pull` 会就地升级两个技能。其他平台（web、API、
Codex、Copilot）各加一条链接、一次 ZIP 上传或一次 API 注册 —— 见下。

### Claude Code（CLI）—— `~/.claude/skills/`

上面的通用安装已覆盖这里。项目级替代方案：把两个技能放进
`<your-project>/.claude/skills/`。技能热加载 —— 无需重启。
文档：<https://code.claude.com/docs/en/skills>

### Claude Desktop（macOS / Windows）—— 共享的用户级目录

桌面应用读与 CLI 相同的 `~/.claude/skills/`。跑过通用安装的话，应用下次
启动即捡起技能。自 2026 年 4 月 13 日起 Free / Pro / Max / Team / Enterprise
均可用。
文档：<https://support.claude.com/en/articles/12512180-use-skills-in-claude>

### claude.ai（web）—— 经 Settings 上传 ZIP

```bash
cd ~/the-knowledge-guy/.claude/skills
zip -r book-to-skill.zip      book-to-skill
zip -r the-knowledge-guy.zip  the-knowledge-guy
```

然后在 claude.ai 里：**Settings → Customize → Skills → Create skill →
Upload .zip**。每个 ZIP 重复一次。需要启用 Code Execution + File Creation。
注意：claude.ai 无法写入本地 `artifacts/` 目录 —— web 会话里的 HTML 产物
改为在聊天容器内联渲染。
文档：<https://support.claude.com/en/articles/12512180-use-skills-in-claude>

### Anthropic API / Agent SDK —— `container.skills`

把每个技能注册为 custom skill，然后在 `container.skills` 中引用两者，带
beta header（需启用代码执行；每请求至多 8 个技能）：

```python
client.messages.create(
    model="claude-opus-4-7", max_tokens=4096,
    container={"skills": [{"type": "custom", "skill_id": "<id-1>"},
                          {"type": "custom", "skill_id": "<id-2>"}]},
    extra_headers={"anthropic-beta": "skills-2025-10-02,code-execution-2025-08-25"},
    messages=[{"role": "user", "content": "/the-knowledge-guy library"}],
)
```

文档：<https://platform.claude.com/docs/en/build-with-claude/skills-guide>

### OpenAI Codex CLI —— `~/.agents/skills/`

Codex CLI（2025 年 12 月起）从 `~/.agents/skills/` 原样消费 `SKILL.md`：

```bash
mkdir -p ~/.agents/skills
ln -s ~/the-knowledge-guy/.claude/skills/book-to-skill      ~/.agents/skills/
ln -s ~/the-knowledge-guy/.claude/skills/the-knowledge-guy  ~/.agents/skills/
```

Codex 的工具运行时与 Claude Code 不同（无 `Skill` 工具、无 `AskUserQuestion`），
因此 **walk** 和 **ingest** 模式退化为提示词式文本回退；**ask**、
**nutshell**、**library** 及其他只读模式可直接工作。
文档：<https://developers.openai.com/codex/skills>

### GitHub Copilot —— 每仓库 `.github/skills/`

Copilot 的技能支持（2026 年 4 月）是项目级的。对每个希望启用技能的仓库：

```bash
cd <your-repo>
mkdir -p .github/skills
cp -R ~/the-knowledge-guy/.claude/skills/book-to-skill      .github/skills/
cp -R ~/the-knowledge-guy/.claude/skills/the-knowledge-guy  .github/skills/
```

在 Copilot 的 agent 模式生效。`book-to-skill` 假设的 Python venv 在
Copilot 容器里不存在 —— 摄入模式在那里跑不了；查询模式可以。
文档：<https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills>
与 <https://code.visualstudio.com/docs/copilot/customization/agent-skills>

### 移植到无原生技能支持的平台

- **Cursor IDE** —— 用 `.cursor/rules/*.mdc`；适配是机械性的（frontmatter
  转换 + 按文件拆分）。[文档](https://cursor.com/docs/rules)。
- **Zed 编辑器** —— 原生支持待定
  ([zed#49057](https://github.com/zed-industries/zed/issues/49057))；过渡期
  把 `SKILL.md` 内容粘进系统提示词。
- **Cline / Roo / Aider / Continue** —— 截至 2026 年 5 月无标准化技能机制；
  权宜之计是把 `book-to-skill/SKILL.md` 和 `the-knowledge-guy/SKILL.md` 的
  运行手册文本粘进平台的系统提示词或规则文件。

### 兼容矩阵（2026 年 5 月）

| 平台 | 原生技能？ | 安装位置 | 原样消费 `SKILL.md`？ |
| --- | --- | --- | --- |
| **Claude Code（CLI）** | ✓ | `~/.claude/skills/` 或 `./.claude/skills/` | 是 |
| **Claude Desktop**（mac/win） | ✓ | `~/.claude/skills/` | 是 |
| **claude.ai**（web） | ✓ | Settings → Customize → Skills（ZIP 上传） | 是 |
| **Anthropic API / Agent SDK** | ✓ | `container={"skills":[…]}` + beta header | 是 |
| **OpenAI Codex CLI**（2025 年 12 月起） | ✓ | `~/.agents/skills/` | 是 |
| **GitHub Copilot**（2026 年 4 月起） | ✓ | 每仓库 `.github/skills/` | 是 |
| **Cursor IDE** | 需适配 | `.cursor/rules/*.mdc` | 否 —— 手动转换 |
| **Zed 编辑器** | 待定 | — | — |
| **Cline / Roo / Aider / Continue** | 否 | — | — |

装好后用 `/book-to-skill /path/to/book.pdf` 摄入你的第一本书（体裁提示 →
成本估算 → 命名）。一本 600 页的书约需 10-20 分钟墙钟时间，API 成本约
$1-3，视模型而定。

## 升级旧技能

四个幂等的辅助脚本处理旧技能：`backfill_book_numbers.py` 补写
`book_number` 并重命名章节文件；`relabel_nutshell.py` 在回填之后修正已缓存
nutshell 的标题；`upgrade_walk_memory.py` 改写 walk 记忆里的过期
`<slug>/chNN` 简写；`upgrade_course_memory.py` 在重新回填之后修复
`practice/*.json` 文件与 `course-<slug>.md` 记忆中的 `book_number` 漂移。
四个脚本对已是最新状态的技能都是无操作（no-op）。

## 贡献

欢迎 PR。先读 [`CLAUDE.md`](./CLAUDE.md) 和两个 `SKILL.md` 文件 —— 三者合
起来就是权威的架构简报，其余文档都派生自它们。

**常见贡献**

- **给 `the-knowledge-guy` 加一个新模式** —— 在 `the-knowledge-guy/SKILL.md`
  的模式分发里加触发词，写模式章节，在 `design-system/layouts.md` 加对应的
  HTML 布局。每个模式都产出产物（按 Step 0.5）。
- **新设计系统组件** —— 加进 `design-system/shell.html`，在
  `design-system/reference/full-demo-light.html` 里演示，并在用到它的布局中
  引用。不要引入第二种强调色 —— cobalt 是承重结构。
- **新的练习类型或概念部件类型** —— 扩展冻结契约（练习用
  `book-to-skill/reference/practice-template.md`，部件用
  `the-knowledge-guy/design-system/widgets.md`），在 `design-system/shell.html`
  对应引擎里加渲染分支，并教会 lint（`lint_practice.py` /
  `lint_concept_widgets.py`）校验它。部件必须保持主题安全 —— 只换 class，
  永不设 `fill`/`stroke`。
- **给 `book-to-skill` 加新的体裁画像** —— 扩展
  `book-to-skill/reference/genre-profiles.md`。体裁调校分块边界、章节 schema
  与 reduce 的侧重。
- **新的辅助脚本** —— 放进 `book-to-skill/scripts/`，做成幂等的，在
  `book-to-skill/SKILL.md` 记录一行调用。
- **hero 示意图** —— 在浏览器打开 `artifacts/readme-hero-pipeline.html`，
  需要时翻转主题开关，对 `.plate` 块截图，保存为 `docs/hero-pipeline.png`。
  源 HTML 用的是设计系统的实时 token，调色板变了图也会跟着同步。

**风格**

- Markdown 按 80 列折行；命令与路径放围栏代码块；行尾不留空格。
- Python 遵循 PEP-8，不引入 `setup.sh` 之外的新依赖（PyMuPDF、ebooklib、
  beautifulsoup4、pypdf）。
- 文档语气是编辑式的自信 —— 陈述句，不用营销语言，不用 emoji。
- commit 标题用祈使句（`Add X`、`Fix Y`）；正文解释这个改动*为什么*存在，
  而非*改了什么*。

**开 PR 之前**

- 动过章节文件，跑 `book-to-skill/scripts/lint_chapters.py <skill-dir>`。
- 动过 `extract.py`，摄入一本小的测试书，确认 `chapters_manifest.json` 为
  `schema_version: 2` 且每个条目都带 `book_number`。
- 动过布局，重新生成一个缓存产物（例如
  `/the-knowledge-guy nutshell <slug> --regenerate`），并在亮、暗两种主题下
  打开。
- 动过练习 schema、练习渲染器或实验，跑
  `book-to-skill/scripts/lint_practice.py <skill-dir>` —— 它会执行每个实验，
  证明解答能过、起点会挂。
- 动过部件 schema、部件引擎或课程页，跑
  `book-to-skill/scripts/lint_concept_widgets.py --page <course.html>`，并在
  两种主题下（以及减少动效设置下）打开页面。

**Issues**

- 报 bug：附上受影响的技能 slug、失败的斜杠命令，以及有助于复现的
  `.claude/skills/<slug>/raw/metadata.json`。
- 功能请求：先描述使用场景，再勾勒实现。

## 下一步

已设计但未发布：语音模式 walk、对记录的错题做间隔重复、远端通道（桌面 /
claude.ai）、以及跨多个重叠 walk 的掌握度视图。
