**05课：OpenClaw Skill 开发实践：基于MinerU 的文档灵活问答**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>简介</strong></p>
<p>MinerU Document Extractor 是上海人工智能实验室 OpenDataLab
出品的文档智能提取工具MinerU的官方Skill ，支持
PDF、Word、PPT、图片、HTML、网页 URL 等多格式一键转
Markdown/HTML/LaTeX/DOCX/JSON，内置表格识别、公式识别、OCR 扫描件识别和
80+ 语言支持。</p>
<p>该 Skill 有两种模式：flash-extract（免 Token、10MB/20页限制、快速输出
Markdown）和 extract（需要 Token、无文件限制、vlm
高精度模型、多格式输出、批量处理）。上述两种模式都能够一键将文档提取成干净的结构化文本。</p>
<p><strong>ClawHub
托管平台：</strong>https://clawhub.ai/mineru-extract/mineru-document-extractor</p>
<p><strong>GitHub
源码下载：</strong>https://github.com/opendatalab/MinerU-Ecosystem/tree/main/skills</p></td>
</tr>
</tbody>
</table>

**一、为什么要做这个 Skill？**

1\. **Q：Skill 之前，用户怎么用 MinerU CLI？**

**A：**MinerU 提供了一个非常好用的命令行工具
[mineru-open-api](https://mineru.net/ecosystem)，可以把
PDF、图片、Word、PPT、网页提取成 Markdown。但对于普通用户，直接用 CLI
有几个门槛：

**不知道装什么**：CLI 名字叫 mineru-open-api，但 npm/pip
包叫什么？怎么验证安装成功？

**命令记不住**：flash-extract 和 extract 的区别是什么？什么时候需要
Token？

**不知道怎么判断**：文件超大怎么办？出现 429 限流该切哪个模式？

**每次都要重新查**：边看文档边操作，效率极低。

**有了 Skill 之后**，OpenClaw
会自动处理上述所有问题——识别文件类型、选择正确的模式、处理错误、引导用户——用户只需要说"帮我把这个
PDF 转成 Markdown"。

这就是 Skill 的核心价值：**把零散的 CLI
命令变成标准化的智能流程，降低使用门槛，保证结果稳定。**

2\. **安装方式**

MinerU Document Extractor skill 目前支持在 Openclaw等Agent工具上的安装。

2.1 **Openclaw 的安装方式一（手动安装）【推荐】**

从下方链接下载 zip 文件，解压后，放到Agent对话框后，对Agent说：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
解压里面的skill.md文件并安装</td>
</tr>
</tbody>
</table>

2.2 **Openclaw 的安装方式二（自动安装）**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
帮我从clawhub安装名为MinerU Document Extractor 的 Mineru Skills。</td>
</tr>
</tbody>
</table>

2.3 **在Claude Code中的安装方式
【由于CC对于ClawHub的网络限制，建议用以下命令】**

A. 全局安装：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
帮我安装https://github.com/opendatalab/MinerU-Ecosystem/blob/main/skills/SKILL.md
这个skill</td>
</tr>
</tbody>
</table>

B. 项目局部安装

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
<em>帮我把这个 skill
局部安装到当前项目：https://github.com/opendatalab/MinerU-Ecosystem/blob/main/skills/SKILL.md</em></td>
</tr>
</tbody>
</table>

2.4 **为MinerU skill 精准模式添加token**

在 https://mineru.net/apiManage/token 官网创建token后，对Agent说：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
为MinerU skill 精准模式添加token：{你的token}</td>
</tr>
</tbody>
</table>

**二、MinerU Skill 能力范围**

在写 Skill 之前，先搞清楚这个工具能做什么、不能做什么。

3\. **支持的输入与输出**

|              |                             |
|:------------:|:---------------------------:|
| **输入类型** |          **示例**           |
|     PDF      |  论文、报告、扫描件、表格   |
|     图片     | JPG、PNG、BMP（含扫描图片） |
| Office 文档  |     DOC/DOCX、PPT/PPTX      |
|   网页/URL   | 技术文档、新闻页面、产品页  |

|                 |                                    |
|:---------------:|:----------------------------------:|
|  **输出格式**   |              **说明**              |
|    Markdown     |        默认输出，兼容性最好        |
|      JSON       |    结构化内容列表，适合二次处理    |
| DOCX/HTML/LaTeX | 精准解析模式的额外格式（需 Token） |

4\. **两种模式：选哪个**

这是 MinerU Skill 最核心的决策点。理解两种模式的差异，是写好 Skill
的前提： 明确cli工具的能力边界。

|  |  |  |
|:---|:---|:---|
| **对比项** | **flash-extract（轻量）** | **extract（精准）** |
| 是否需要 Token | **不需要**，按 IP 限流 | **需要** MinerU Token |
| 文件大小限制 | ≤ 10 MB | ≤ 200 MB |
| 页数限制 | ≤ 20 页 | ≤ 200 页 |
| 模型选择 | 固定轻量模型 | pipeline / vlm / MinerU-HTML |
| 批量处理 | 不支持 | 支持 |
| 输出格式 | 仅 Markdown | Markdown + JSON + docx/html/latex |
| 适合场景 | 快速预览、小文件、AI Agent | 大文件、批量、高精度 |

5\. **模式选择决策树**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
用户提交文件/URL<br />
│<br />
▼<br />
文件 &gt; 10MB 或页数 &gt; 20？<br />
│<br />
是 ──→ 走 extract（精准模式）<br />
│<br />
否<br />
│<br />
▼<br />
需要批量处理？<br />
│<br />
是 ──→ 走 extract<br />
│<br />
否<br />
│<br />
▼<br />
需要多格式输出（docx/latex）？<br />
│<br />
是 ──→ 走 extract<br />
│<br />
否<br />
│<br />
▼<br />
走 flash-extract（优先，无需 Token）<br />
<br />
──────────────────────────────<br />
extract 内部还有一层模型选择：<br />
- 复杂版面/图文混排 ──→ --model vlm（多模态，推荐）<br />
- 稳定无幻觉优先 ──→ --model pipeline（纯结构化）<br />
- 网页提取 ──→ --model MinerU-HTML 或直接用 crawl</td>
</tr>
</tbody>
</table>

把这个决策树写进 Skill，OpenClaw 就能自动替用户做判断，这是 Skill
最有价值的部分之一。

**三、高质量 Skill 的结构**

一个 Skill 本质上是一个特殊格式的 Markdown
文件（SKILL.md）。写好它需要理解每个字段的作用。

**SKILL.md 的完整结构**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
---<br />
name: # Skill 名称，影响检索<br />
description: # 能力描述，最关键的排名因子<br />
tags: # 分类标签<br />
metadata: # 运行约束（依赖、安装、隐私）<br />
allowed-tools: # 允许调用的工具<br />
---<br />
<br />
# README 正文</td>
</tr>
</tbody>
</table>

每个字段都有明确分工，下面逐一展开。

**四、字段详解：怎么写才对**

1\. **name — 名称**

**作用**：显示在 ClawHub 卡片上，也参与检索排名的 LexicalBoost 加分。

**写法原则**：

包含核心关键词（如 MinerU、Document、PDF）

简洁，不超过 5 个词

不要写成动词句（"Extract PDF to Markdown" 不如 "MinerU Document
Extractor"）

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">YAML<br />
name: MinerU Document Extractor</td>
</tr>
</tbody>
</table>

2\. **description — 描述（最关键）**

**作用**：ClawHub 搜索的向量语义匹配主要来源，直接决定 Skill
能否被召回。

**写法三段式**：

**第一句**：用户问题 + 核心能力关键词

**第二句**：能力闭环（输入 → 处理 → 输出）

**第三句**：目标场景（论文、扫描件、批量、网页）

**好的写法：**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">YAML<br />
description: &gt;<br />
Zero-setup document extraction — convert PDFs, images, Word, and
PowerPoint to Markdown using MinerU open API CLI.<br />
Supports OCR, table recognition, formula extraction, and web crawling;
outputs Markdown or multi-format (docx, latex, html).<br />
Ideal for academic papers, scanned documents, batch processing, and
webpage archiving.</td>
</tr>
</tbody>
</table>

**不好的写法：**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">YAML<br />
description: &gt;<br />
This skill uses MinerU to extract documents.<br />
It supports PDF and images.</td>
</tr>
</tbody>
</table>

区别在哪？前者覆盖了用户可能搜索的所有关键词（PDF、OCR、table、formula、Markdown、batch、webpage），后者只覆盖了两个。

Tip：description不是越长越好，太长会影响clawhub搜多的similarity
consine，核心是围绕如何让模型更快了解你的skill的作用

3\. **metadata — 运行约束**

**作用**：告诉 OpenClaw 这个 Skill 依赖什么、怎么安装、有没有隐私风险。

**原则**：只写"可验证事实"，不要在这里堆关键词（关键词放 description 和
README）。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">YAML<br />
metadata: {"openclaw":{<br />
"emoji":"📄",<br />
"homepage":"https://mineru.net",<br />
"author":"OpenDataLab",<br />
"privacy":"文件内容通过 MinerU Open API 处理；免登录模式按 IP
限流，不持久化存储；精准模式需 Token，处理后文件按 MinerU
官方隐私策略保留。",<br />
"requires":{"bins":["mineru-open-api"]},<br />
"optional":{"env":["MINERU_API_TOKEN"]},<br />
"install":[<br />
{<br />
"id":"curl-install",<br />
"kind":"shell",<br />
"label":"Install via install script (Mac/Linux)",<br />
"command":"curl -fsSL
https://cdn-mineru.openxlab.org.cn/open-api-cli/install.sh | sh"<br />
}<br />
]<br />
}}</td>
</tr>
</tbody>
</table>

**字段说明：**

requires.bins：声明依赖的可执行文件，OpenClaw 会在启动时检查是否已安装

optional.env：可选的环境变量，未配置时走 flash-extract 免 Token 模式

install：安装指引，OpenClaw 会在检测到缺依赖时主动提示用户执行

4\. **allowed-tools — 工具白名单**

**作用**：声明这个 Skill 允许调用哪些工具，是安全控制机制。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">YAML<br />
allowed-tools: Bash(mineru-open-api:*), Read, Write, Edit</td>
</tr>
</tbody>
</table>

Bash(mineru-open-api:\*) 表示只允许执行 mineru-open-api 开头的 shell
命令，防止滥用。

Tip： 上述两个影响的是上传到clawhub的判断是否被suspicious

**五、Skill.md 正文：智能行为的核心**

name/description/metadata 定义了 Skill 的"身份"，README 正文定义了 Skill
的「行为」——即 OpenClaw 在运行时如何决策。

README 有四个固定章节，各有明确分工：

|  |  |
|:--:|:--:|
| **章节** | **作用** |
| \# \<Title\> 搜索排名关键检索字段 | 一级标题参与 embedding，同时起 SEO 锚点作用 |
| \## Setup | 声明安装方式，OpenClaw 首次运行时引导用户执行 |
| \## workflow：快速 skill 说明流程 | 列出用户最常提的问题，帮助 OpenClaw 理解触发意图 |
| \## When To Use | 精确触发条件，OpenClaw 判断"该不该用这个 Skill" |
| \## Decision Rules | 运行时核心逻辑，OpenClaw 判断"用哪个命令、哪个参数" |

下面展开每个章节的写法与 MinerU 实际内容。

1\. **一级标题 — 搜索排名关键检索字段**

这行看起来只是标题，实际上有两个作用：

**SEO 作用**：ClawHub 对 README 一级标题做
embedding，它会参与向量检索排名。

**语义锚点**：告诉搜索引擎这个 Skill 的核心词汇是什么。

**写法原则**：在 Skill 名称后紧跟 2~4
个核心检索词，不用写成句子，关键词自然排列即可。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Markdown<br />
# MinerU Document Extractor — PDF OCR Markdown 文档提取</td>
</tr>
</tbody>
</table>

这行包含了 MinerU、Document、PDF、OCR、Markdown、文档提取
六个高频搜索词，覆盖了用户可能输入的各种搜索表达。

2\. **Setup — 安装说明**

Setup 章节只做一件事：**让用户能运行起来**。与 metadata.install
保持一致即可，不要写多余的解释。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Markdown<br />
## Setup<br />
<br />
# 安装 MinerU CLI<br />
curl -fsSL https://cdn-mineru.openxlab.org.cn/open-api-cli/install.sh |
sh<br />
<br />
# 验证安装<br />
mineru-open-api --version<br />
<br />
# （可选）精准解析模式需要 Token，运行一次登录即可<br />
mineru-open-api auth</td>
</tr>
</tbody>
</table>

**注意**：auth 步骤是可选的。未登录时 Skill 会自动走 flash-extract 免
Token 模式，不影响基础使用。

3\. **workflow — 快速 Skill 说明流程**

这是最容易被忽略但非常有价值的章节。它的作用是：**列出用户最常提的真实问题，帮助
OpenClaw 识别何时应该调用这个 Skill。**

写法：直接写用户会说的话，不是功能描述，而是用户的原话。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Markdown<br />
## workflow：快速 skill 说明流程<br />
<br />
- 帮我把这个 PDF 转成 Markdown<br />
- 这个扫描件里的表格能提取出来吗<br />
- 帮我把这篇论文的公式识别一下<br />
- 批量处理这个文件夹里所有的 PDF<br />
- 把这个网页的内容抓取下来保存成 Markdown<br />
- 这个 Word 文件能转成 Markdown 吗<br />
- 图片里的文字怎么提取出来</td>
</tr>
</tbody>
</table>

**为什么这样写有效**？当用户说"帮我把这个 PDF 转成 Markdown"时，OpenClaw
会在所有已安装的 Skill 中做语义匹配，workflow
里写的这句话与用户输入高度相似，匹配成功率远高于写"Supports PDF
extraction"这样的功能描述。

4\. **When To Use — 精确触发条件**

When To Use 比 workflow 更结构化，是 OpenClaw 判断"该不该激活这个
Skill"的规则集。写触发条件，不写功能介绍。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Markdown<br />
## When To Use<br />
<br />
- 用户提供了 PDF、图片（JPG/PNG/BMP）、Word（DOC/DOCX）、PPT/PPTX
文件，需要提取内容<br />
- 用户提到 OCR、表格识别、公式识别、文字提取<br />
- 用户想把网页 URL 或在线文档抓取成 Markdown<br />
- 用户需要批量处理多个文档文件<br />
- 用户提到"文档转换"、"内容提取"、"PDF 转文字"、"扫描件识别"<br />
- 用户需要把文档转为 docx、latex、html 等格式</td>
</tr>
</tbody>
</table>

**和 workflow 的区别**：workflow 是用户的原话，帮助模型快速上手skill
核心功能，When To Use 是对触发场景的结构化描述。两者共同作用，让 Skill
的触发更精准。

5\. **Decision Rules — 运行时核心逻辑**

这是整个 Skill 最关键的部分。OpenClaw 在确定要使用这个 Skill 之后，会读
Decision Rules 来决定：**该用哪个命令、哪个参数、出错了怎么处理**。

规则写得越具体，OpenClaw 的行为就越稳定、越符合预期。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Markdown<br />
## Decision Rules<br />
<br />
1. 无 Token / 文件 ≤ 10MB / 页数 ≤ 20 → 优先走
`flash-extract`，无需登录<br />
2. 文件 &gt; 10MB 或页数 &gt; 20 → 走 `extract`，提示用户先运行
`mineru-open-api auth`<br />
3. 需要批量处理多个文件 → `extract *.pdf -o ./results/`<br />
4. 需要输出 docx / latex / html → `extract report.pdf -f
md,docx`（仅精准模式支持）<br />
5. 复杂版面、图文混排、扫描件 → `extract --model
vlm`（多模态，推荐）<br />
6. 稳定性优先、不需要高精度 → `extract --model pipeline`（无幻觉）<br />
7. 输入是网页 URL → `crawl &lt;url&gt; -o ./results/`<br />
8. 出现 HTTP 429（IP 限流）→ 切换到 `extract` 模式，告知用户原因<br />
9. 解析结果异常或内容缺失 → 建议加 `--model vlm` 重试，扫描件确认开启
OCR<br />
10. Token 未配置但需要精准模式 → 提示运行 `mineru-open-api auth` 或设置
`MINERU_API_TOKEN`</td>
</tr>
</tbody>
</table>

**规则写作要点**：

**条件要具体**：写"文件 \> 10MB"，不要写"文件较大"

**动作要完整**：写完整命令（含参数），不要只写"用 extract 模式"

**覆盖错误路径**：429、Token 缺失、解析失败这些边界情况必须有对应规则

**顺序有优先级**：规则从上到下匹配，常见场景放前面

**六、完整 SKILL.md 模板**

下面是一个可以直接复制修改的完整模板，注释说明了每个字段的填写要点：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Markdown<br />
---<br />
name: &lt;核心关键词 + 工具名，如 "MinerU Document Extractor"&gt;<br />
description: &gt;<br />
&lt;首句：用户问题 + 核心关键词，例如 "Zero-setup PDF/image/Word/PPT
extraction to Markdown"&gt;。<br />
&lt;次句：能力闭环，输入 → 处理 → 输出&gt;。<br />
&lt;末句：目标场景，如 "Ideal for academic papers, scanned docs, batch
processing, web archiving"&gt;。<br />
tags:<br />
- document<br />
- pdf<br />
- ocr<br />
metadata: {"openclaw":{<br />
"emoji":"📄",<br />
"homepage":"&lt;官网地址&gt;",<br />
"author":"&lt;团队名&gt;",<br />
"privacy":"&lt;数据流向说明；是否持久化；第三方服务&gt;",<br />
"requires":{"bins":["&lt;cli-binary-name&gt;"]},<br />
"optional":{"env":["&lt;OPTIONAL_TOKEN_ENV&gt;"]},<br />
"install":[<br />
{<br />
"id":"install-script",<br />
"kind":"shell",<br />
"label":"Install &lt;工具名&gt;",<br />
"command":"&lt;安装命令&gt;"<br />
}<br />
]<br />
}}<br />
allowed-tools: Bash(&lt;cli-binary&gt;:*), Read, Write, Edit<br />
---<br />
<br />
# &lt;Skill Title&gt; — &lt;关键词副标题&gt;<br />
<br />
## Setup<br />
<br />
```bash<br />
&lt;安装命令&gt;<br />
&lt;验证安装命令&gt;</td>
</tr>
</tbody>
</table>

**附：MinerU CLI 常用命令速查**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
# 安装<br />
curl -fsSL https://cdn-mineru.openxlab.org.cn/open-api-cli/install.sh |
sh<br />
<br />
# Token 登录（精准模式需要）<br />
mineru-open-api auth<br />
<br />
# 轻量提取（免 Token）<br />
mineru-open-api flash-extract report.pdf<br />
mineru-open-api flash-extract https://example.com/paper.pdf<br />
<br />
# 精准提取<br />
mineru-open-api extract report.pdf<br />
mineru-open-api extract report.pdf -f md,docx -o ./output/<br />
mineru-open-api extract report.pdf --model vlm # 复杂版面<br />
mineru-open-api extract report.pdf --model pipeline # 稳定无幻觉<br />
<br />
# 批量<br />
mineru-open-api extract *.pdf -o ./output/<br />
<br />
# 网页抓取<br />
mineru-open-api crawl https://mineru.net -o ./output/</td>
</tr>
</tbody>
</table>

|  |
|:---|
| 完整参数文档：https://github.com/opendatalab/MinerU-Ecosystem/tree/main/cli/mineru-open-api |
