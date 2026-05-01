**01课：MinerU 全场景产品矩阵：快速上手与选型**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p><strong>课程定位</strong>：本课是 MinerU 实战训练营的入门第一课，帮助你建立对 MinerU 全产品线的完整认知，并找到最适合自己场景的工具入口。</p>
<p><strong>适合人群</strong>：所有参营学员，无论技术背景。</p>
<p><strong>学完你将能</strong>：清楚说出 MinerU 有哪些产品、各自适合什么场景，并能根据自身需求做出选型决策。</p></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>MinerU相关资源</strong></p>
<p><strong>MinerU官网：</strong>https://mineru.net/</p>
<p><strong>MinerU开源项目：</strong>https://github.com/opendatalab/mineru</p>
<p><strong>MinerU skills：</strong><u>https://clawhub.ai/MinerU-Extract/mineru-ai</u></p>
<p><strong>MinerU API：</strong>https://mineru.net/apiManage/docs</p>
<p><strong>MCP Server、CLI/SDK、RAG框架插件：</strong><u>https://mineru.net/ecosystem</u></p>
<p><strong>MinerU在线产品（客户端，web端，在线api）相关问题，请进对应群交流：</strong>https://mineru.net/community-portal/?aliasId=73ea3ef3</p>
<p><strong>MinerU开源项目相关问题，请进对应群交流：</strong>https://mineru.net/community-portal/?aliasId=3c430f94</p></td>
</tr>
</tbody>
</table>
<p>同步自文档: <a href="https://aicarrier.feishu.cn/docx/Y6PUdFwiBohGEix2LILcFed3nUh#LR38dKhxdsORoVbXEf0cXIK7nAb">https://aicarrier.feishu.cn/docx/Y6PUdFwiBohGEix2LILcFed3nUh#LR38dKhxdsORoVbXEf0cXIK7nAb</a></p></td>
</tr>
</tbody>
</table>

**一、为什么需要 MinerU？**

在正式讲解MinerU之前，我们先聊聊大家都遇到过的真实PDF解析场景：

市面上能处理 PDF 的商业工具，大多要么按订阅收费，要么按页计费。顶级大模型倒是能读得懂，但用量一上去成本就很难看。

开源项目倒是不少，但真正拿去跑生产环境，遇到复杂版面就容易翻车——双栏论文、图文混排、扫描合同，稍微难一点的格式，解析出来的东西要么各种乱码错误，要么直接不能用。其中一些传统OCR工具，遇到多栏排版就会串行，遇到表格就丢结构，遇到数学公式就输出问号。如果你是一个开发者，你拿这样的文本去做 RAG，检索结果可想而知。

还有工程层面的问题。很多团队的解决方案是把 OCR、版面分析、格式转换几个工具串起来用。但这几个工具的输出格式各自为政，每换一个环节就要写一段胶水代码，系统越拼越脆。这类项目在部署过程就很复杂，需要安装左一个库，右一个依赖，同时后期运维成本也越来越高。

这三件事合在一起——**成本、精度、集成**——是整个文档解析领域长期没解决好的问题。

**二、MinerU 产品全景图**

针对前述行业痛点，MinerU 凭借其卓越的解析性能，构建了“一站式、全场景”的文档处理工具链。MinerU 并非单一的格式转换工具，而是由**本地部署、在线服务、生态工具**三大核心产品线构成的完整矩阵，旨在为广大用户提供多维度的技术支撑：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
MinerU 全场景产品矩阵<br />
│<br />
├── 产品线一：本地部署<br />
│ ├── ① MinerU 开源版（多种后端，适配10余家主流国产算力）<br />
│ ├── ② MinerU-HTML（网页语料专用）<br />
│<br />
├── 产品线二：在线服务<br />
│ ├── ④ 桌面客户端（零代码拖拽）<br />
│ ├── ⑤ 在线网页版（浏览器直用，全平台，无需安装）<br />
│ ├── ⑥ 在线 API（提供快速模式与精准模式2种模式；前者免登录，后者更精准）<br />
│<br />
└── 产品线三：生态工具<br />
├── ⑧ Skills / Agent（自然语言调用）<br />
├── ⑨ MCP Server（IDE 直连）<br />
├── ⑩ CLI / SDK（代码集成）<br />
└── ⑪ RAG 框架插件（Dify / LangChain 等）</td>
</tr>
</tbody>
</table>

接下来，我将逐一介绍 MinerU 的各条产品线，帮大家快速上手，选出最适合自己的PDF解析工具。

**三、产品线一：本地部署**

本地部署意味着数据完全在你自己的机器或服务器上处理，不经过任何第三方。这对金融、政务、医疗等有数据合规要求的场景尤为重要。

1\. **MinerU 开源版（⭐ 56k+）**

这是 MinerU 的"原点"产品，也是目前 GitHub 上最受欢迎的文档解析开源项目之一。其具备多种解析模式**，各有所长：**

|  |  |  |  |
|:--:|:--:|:--:|:--:|
| **解析模式** | **后端特性** | **通俗解释** | **适用场景** |
| **pipeline** | 兼容性好，主要依赖多种机器学习小模型实现对PDF的解析 | 老实巴交的传统模式，什么电脑都能跑，但精度一般，速度也一般 | 电脑配置一般，或者没有独立显卡的用户 |
| **hybrid-auto-engine** | 下一代高精度解决方案，通过本地计算能力实现。能够自动切换为pipeline或者纯视觉大模型模式 | 新一代智能模式，用你自己的显卡算，精度高，是默认选择 | 有不错显卡（8GB显存以上），想要最好效果的用户 |
| **vlm-auto-engine** | 通过本地计算能力实现高精度 | 纯视觉大模型模式，完全靠显卡算，精度最高但最吃配置 | 有高端显卡（10GB显存以上），追求极致精度的用户 |
| **hybrid-http-client** | 高精度但需要少量本地计算能力，适用于OpenAI兼容服务器 | 半远端半本地模式，连个远程服务器，本地稍微辅助一下 | 想要高精度但显卡不够好，有网络和远程服务器的用户 |
| **vlm-http-client** | 通过远程计算能力实现高精度，适用于OpenAI兼容服务器 | 纯远端模式，所有计算都交给远程服务器 | 显卡很差或者没有显卡，但有网络和远程服务器的用户 |

目前 MinerU 开源版本先后完成了**昇腾、平头哥、沐曦、海光、燧原、摩尔线程、天数智芯、寒武纪、昆仑芯、太初元碁、壁仞等** 10 余家主流国产算力的适配。主要采用 **Docker 一键部署**，无需手动配置底层驱动，开箱即用。

2\. **MinerU-HTML 开源版**

专为**网页语料**设计的解析器。与 PDF 解析不同，MinerU-HTML 聚焦于从网页中提取干净、结构完整的文本内容，进而为大模型提供更加优质的 AI-Ready 语料：

保留标题层级、表格、列表等 HTML 语义结构

自动去除广告、导航栏等干扰元素

**典型用途：**从 Common Crawl 等网页语料库中抽取高质量训练数据

**四、产品线二：在线服务**

如果你不想折腾本地环境，MinerU 的在线服务是最快的起点。

1\. **桌面客户端**

图形界面，拖拽即解析，**全程零代码**。

支持 Windows、macOS等系统

**适合人群**：产品经理、运营、高校老师、非技术背景的参营学员

下载地址：[mineru.net/client](https://mineru.net/client)

2\. **在线网页版**

浏览器打开即用，无需安装任何东西。解析结果会以颜色块直观标注不同版面区域（正文、表格、图片、公式……），帮助你快速建立对 MinerU 解析能力的感知。

体验地址：[mineru.net/OpenSourceTools/Extractor](https://mineru.net/OpenSourceTools/Extractor)

**建议**：第一次使用 MinerU，先去 Web 体验页上传一份你手边的 PDF，用 5 分钟建立直观认知，再决定后续选哪种产品。

3\. **在线API**

针对开发者，我们提供了在线API，方便在服务中进行调用。在线API在持续迭代中， 目前主要分为快速模式（Flash）与精准模式（Precision ）。这是 MinerU 在线 API 的两种核心模式，针对不同需求设计：

|                 |                          |                                 |
|:---------------:|:------------------------:|:-------------------------------:|
|   **对比项**    | **⚡ 快速模式（Flash）** |  **🎯 精准模式（Precision）**   |
| 是否需要 Token  |        ❌ 免登录         |       ✅ 官网注册免费申请       |
|      速度       |       极快（秒级）       |              标准               |
| 表格 / 公式识别 |            ❌            |               ✅                |
|    批量处理     |            ❌            |               ✅                |
|    输出格式     |         Markdown         | MD / HTML / LaTeX / DOCX / JSON |
|    文件限制     |      ≤ 10MB / 20 页      |        ≤ 200MB / 600 页         |
|  **推荐场景**   | Agent 实时调用、快速验证 | 大赛语料批量生产、学术论文处理  |

**如何选择？**

你在写 Agent，需要实时解析文档 → 用 **Flash**，免登录，秒出结果

你要批量处理 100+ 篇论文或报告 → 用 **Precision**，注册后免费申请 Token

|                                                              |
|:-------------------------------------------------------------|
| Flash 模式完全免费且免登录，是你今天就能上手体验的最快路径。 |

**五、产品线三：生态工具**

生态工具层让 MinerU 的文档解析能力可以无缝嵌入到你的开发工作流中。

1\. **Skills / Agent（自然语言调用）**

通过 OpenClaw 等 Agent 平台，用**自然语言**指挥 MinerU 完成文档解析任务，无需写任何代码：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
"帮我提取这个 PDF 第 10-15 页的表格"<br />
"把这 500 页论文集的所有公式转成 LaTeX"<br />
"批量解析这个文件夹里所有扫描件"</td>
</tr>
</tbody>
</table>

支持平台：OpenClaw、ZeroClaw、NanoClaw、Nanobot、PicoClaw

安装地址：[clawhub.ai/MinerU-Extract/mineru-document-extractor](https://clawhub.ai/MinerU-Extract/mineru-document-extractor)

|                                                                |
|:---------------------------------------------------------------|
| 本训练营第 5 课《OpenClaw Skill 开发实践》将带你深入这个方向。 |

2\. **MCP Server（IDE 直连）**

遵循 MCP 标准协议，让 **Cursor、Claude Desktop、Windsurf** 等 AI 编程工具直接调用 MinerU 解析能力。

支持本地部署，文档不经第三方中转（数据安全有保障）

实时流式响应，解析结果即时返回到 IDE

3\. **CLI / SDK（代码集成）**

支持 Python、Go、JavaScript，适用于 macOS / Linux / Windows。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
# 快速解析（免 Token）<br />
mineru-open-api flash-extract paper.pdf<br />
<br />
# 精准批量解析，输出 Markdown 和 HTML<br />
mineru-open-api extract *.pdf -f md,html -o ./output/</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p>本训练营第 3 课《MinerU 在线 API 深度调用》将手把手带你写出批量解析脚本。</p>
<p>MinerU 目前支持通过 <strong>HTTP 请求</strong> 方式进行调用（API 模式），但暂时没有提供原生 Java 版的 SDK</p></td>
</tr>
</tbody>
</table>

4\. **RAG 框架插件**

与主流 RAG 框架深度集成，支持 **Dify、LangChain、Flowise、RagFlow、FastGPT**。

核心优势：

**解析即分块**：每个 chunk 自动携带页码、版面类型、坐标元数据

**Flash 模式免 Token 即可跑通**，快速验证 RAG 流程

输出标准化文档对象，直接对接向量库，**零二次清洗**

**六、解析效果：输入与输出对照**

不同文档类型推荐使用不同产品，下表帮你快速匹配：

|  |  |  |
|:--:|:--:|:--:|
| **输入文档类型** | **推荐产品** | **输出效果** |
| 双栏学术论文 PDF | 桌面客户端/在线网页版/精准模式 API / 开源项目 | 完整 Markdown，公式为 LaTeX，表格结构完整 |
| 重要扫描合同 | 国产算力版 + OCR | 文字识别率 \>98%，结构化段落 |
| 网页文章 | 桌面客户端 / MinerU-HTML 开源版 | 清洁 HTML，去广告，保留正文结构 |
| 带图报告 / PPT | 桌面客户端 / SDK | 图文分离，图片单独保存，文字完整提取 |
| 批量 PDF（100+） | 精准模式 API 异步批处理 | 并发处理，状态监控，自动重试 |
| 实时问答场景 | Skills / 快速模式 API | 秒级响应，即问即答 |

**七、选型决策树**

面对这么多产品，怎么快速做决定？跟着这棵决策树走：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
你的数据能上传到云端吗？<br />
│<br />
├── 不能（合规要求 / 敏感数据）<br />
│ └── 本地部署<br />
│ └── 普通 GPU/CPU → MinerU 开源版<br />
│ ├── 有国产算力 → Docker一键部署<br />
│ ├── 根据算力情况 → 选择不同后端<br />
│<br />
└── 可以上传<br />
├── 不想写代码<br />
│ ├── 想在浏览器里试试 → 在线网页版<br />
│ └── 想长期使用图形界面 → 桌面客户端<br />
│<br />
└── 需要代码集成<br />
├── Agent / 自然语言调用 → Skills<br />
├── IDE 里直接调用 → MCP Server<br />
├── 快速实时解析（秒级）→ 快速模式（Flash） API<br />
└── 批量 / 需要表格公式 → 精准模式（Precision ）API</td>
</tr>
</tbody>
</table>

**八、今天就动手：三件必做任务**

理论读完，立刻动手才是真正的学习。完成以下三件事，你就算正式开始了：

**任务 1：体验在线网页解析**

上传一份你手边的 PDF，感受解析效果，用时约 5 分钟。

👉 [mineru.net/OpenSourceTools/Extractor](https://mineru.net/OpenSourceTools/Extractor)

**任务 2：安装 MinerU Skills**

在 Agent 平台中一键接入文档解析能力，为后续实战做准备。

👉 https://clawhub.ai/mineru-extract/mineru-ai

**任务 3：报名 MDIC 大赛**

用 MinerU 解析语料，把本课学到的知识应用到实战比赛中。

👉 https://mineru.net/MDIC2026/competition

**九、关键链接速查**

|  |  |
|:--:|:--:|
| **资源** | **链接** |
| GitHub 开源仓库 | [github.com/opendatalab/MinerU](https://github.com/opendatalab/MinerU) |
| Web 在线体验 | [mineru.net/OpenSourceTools/Extractor](https://mineru.net/OpenSourceTools/Extractor) |
| API 文档 | [mineru.net/apiManage/docs](https://mineru.net/apiManage/docs) |
| 桌面客户端下载 | [mineru.net/client](https://mineru.net/client) |
| Skills 安装 | [clawhub.ai/MinerU-Extract/mineru-document-extractor](https://clawhub.ai/MinerU-Extract/mineru-document-extractor) |
| 大赛官网 | [mineru.net/MDIC2026](https://mineru.net/MDIC2026) |
| 生态资讯 | [minerunews.manus.space](https://minerunews.manus.space) |
| 常见问题 Q&A | [deepwiki.com/opendatalab/MinerU](https://deepwiki.com/opendatalab/MinerU) |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>下节预告</strong>：《MinerU 多环境部署实践：从开源容器化到信创生态适配》</p>
<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">我们将手把手带你完成 CPU/GPU 版本 Docker 安装，以及国产算力（沐曦）的实战部署。</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>
