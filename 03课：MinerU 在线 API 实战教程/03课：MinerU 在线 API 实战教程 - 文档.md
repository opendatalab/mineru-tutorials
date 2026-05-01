**03课：MinerU 在线 API 实战教程**

*本教程带你从零认识 MinerU Open API，了解在产品的生态工具，并通过两个实战项目上手：飞书知识库、批量发票提取器。*

**一、MinerU 在线 API 速览**

[MinerU](https://mineru.net) 是一款开源文档提取工具，同时提供在线 API 服务。目前，我们将其命名为「MinerU Open API」。通过提供线上服务，帮助开发者和用户免部署、开箱即用。「MinerU Open API」现阶段分为2种类型，可满足不同场景：

|  |  |  |
|:--:|:--:|:--:|
|  | **精准解析 API** | **Agent 轻量解析 API** |
| **需要 Token** | 是 | 否（按 IP 限流） |
| **文件大小** | ≤ 200 MB | ≤ 10 MB |
| **页数限制** | ≤ 600 页 | ≤ 20 页 |
| **模型选择** | pipeline / vlm / MinerU-HTML | 固定轻量模型 |
| **批量处理** | 支持（≤ 200 文件） | 不支持 |
| **输出格式** | Markdown + JSON + docx/html/latex | 仅 Markdown |
| **适用场景** | 生产环境、大文件、批量任务 | AI Agent、快速预览、小文件 |

**\**如何获取 Token：*** *访问 [mineru.net](https://mineru.net) → 注册 → 进入「API管理 → Token」→ 复制。MinerU官网提供了每日免费 2000 页高优先级额度。完整 API 参数请查阅 [官方文档](https://mineru.net/apiManage/docs)，本教程只在实战中用到时才展开。*

**二、核心参数详解及注意事项**

<table>
<colgroup>
<col style="width: 16%" />
<col style="width: 40%" />
<col style="width: 20%" />
<col style="width: 21%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: center;"><strong>参数名</strong></td>
<td style="text-align: center;"><strong>取值</strong></td>
<td style="text-align: center;"><strong>API接口默认值</strong></td>
<td style="text-align: center;"><strong>影响</strong></td>
</tr>
<tr>
<td style="text-align: center;">language</td>
<td style="text-align: center;">ch,en,korean,japan等</td>
<td style="text-align: left;">ch(Chiness+English)</td>
<td style="text-align: left;">影响ocr识别精度，建议小语种的扫描件都显式设置下对应的语言</td>
</tr>
<tr>
<td style="text-align: center;">is_ocr</td>
<td style="text-align: left;">true|false 是否强制开启ocr</td>
<td style="text-align: left;">false</td>
<td style="text-align: left;">不强制开启模型会自动识别是否需要进行ocr识别，通常情况下设为false即可。如果对解析效果不满意，可以开启试试效果</td>
</tr>
<tr>
<td style="text-align: center;"><p>model_version</p>
<p>(精度解析api)</p></td>
<td style="text-align: center;"><p>pipeline: <strong>无幻觉</strong>，可解析pdf、word、ppt、图片</p>
<p>vlm：<strong>多模态高精度，推荐，</strong>可解析pdf、word、ppt、图片</p>
<p>MinerU-HTML：html，提取正文，支持url及文件上传</p></td>
<td style="text-align: left;">pipeline</td>
<td style="text-align: left;">解析精度及文件类型</td>
</tr>
<tr>
<td style="text-align: center;"><p>extra_formats</p>
<p>(精度解析api)</p></td>
<td style="text-align: center;"><p>docx：pdf转docx场景</p>
<p>latex：学术场景</p>
<p>html</p></td>
<td style="text-align: left;">空</td>
<td style="text-align: left;">转换后的文件会加到zip包返回</td>
</tr>
</tbody>
</table>

1\. **URL解析注意事项**

提交的文件 URL 需为公网可访问地址，确保系统能够正常下载文件。

批量提交大量 URL 任务时，需特别关注文件源站是否存在访问频率限制。例如，arXiv 等网站对 PDF 文件访问可能设置了频控。建议根据源站规则合理控制请求并发量，避免因触发反爬或限流机制导致任务失败。

URL 任务提交接口仅支持传入文件链接，不支持直接上传文件

2\. **本地文件解析注意事项**

在申请文件链接时，建议提供包含文件后缀在内的完整文件名，以提高文档校验的成功率。

文件上传完成后，可直接使用第一步申请文件链接时返回的 batch_id（精准解析）或 task_id（轻量解析）轮询任务结果，无需再通过文件 URL 重复提交任务。

**三、解析结果详解**

**解析状态说明**

<table>
<colgroup>
<col style="width: 15%" />
<col style="width: 84%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><strong>状态</strong></td>
<td style="text-align: left;"><strong>说明</strong></td>
</tr>
<tr>
<td style="text-align: left;">waiting-file</td>
<td style="text-align: left;">系统尚未检测到文件上传，只对本地文件解析接口有效。注：文件上传成功后，系统检测到会有毫秒级的延迟</td>
</tr>
<tr>
<td style="text-align: left;">uploading</td>
<td style="text-align: left;">文件下载中，对于url提交的文件，后端服务需要先下载文件再给模型推理。对于精准解析中的Word、PPT文件，转换为pdf也在该阶段执行</td>
</tr>
<tr>
<td style="text-align: left;">pending</td>
<td style="text-align: left;">排队中，当在线任务过多时，任务可能会停留在pending状态一段时间</td>
</tr>
<tr>
<td style="text-align: left;">running</td>
<td style="text-align: left;"><p>解析中，对于精准解析API，会返回解析进度信息，例如：</p>
<p>"extract_progress": {</p>
<p>"extracted_pages": 1,</p>
<p>"total_pages": 2,</p>
<p>"start_time": "2025-01-20 11:43:20"</p>
<p>}</p></td>
</tr>
<tr>
<td style="text-align: left;">converting</td>
<td style="text-align: left;">格式转换中，当设置了extra_formats参数时，会存在该状态</td>
</tr>
<tr>
<td style="text-align: left;">done</td>
<td style="text-align: left;">解析成功，下载相应的结果文件即可</td>
</tr>
<tr>
<td style="text-align: left;">failed</td>
<td style="text-align: left;">解析失败，err_msg里有详细的失败原因</td>
</tr>
</tbody>
</table>

**解析结果文件说明**

<table>
<colgroup>
<col style="width: 45%" />
<col style="width: 54%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: center;"><img src="03课：MinerU 在线 API 实战教程/03课：MinerU 在线 API 实战教程 - 文档 - media/media/image1.png" style="width:2.44792in;height:1.40625in" /></td>
<td style="text-align: left;"><p>*_content_list.json：以段落为单位的json</p>
<p>*_model.json: 模型返回的原始json</p>
<p>content_list_v2.json: 段落+页为单位的json</p>
<p>full.md: markdown格式</p>
<p>layout.json: 以span为单位，span拼接为段即为content_list.json</p>
<p>详细文件结构：https://opendatalab.github.io/MinerU/reference/output_files</p></td>
</tr>
</tbody>
</table>

**四、生态工具一览**

MinerU 围绕 API 构建了一整套开箱即用的生态工具，便于嵌入现有开发框架与 Agent 系统：

|  |  |  |
|:--:|:--:|:--:|
| **工具** | **一句话介绍** | **安装方式** |
| **CLI** | 零依赖命令行工具，支持 flash-extract（免登录）和 extract（精准） | curl -fsSL https://cdn-mineru.openxlab.org.cn/open-api-cli/install.sh \| sh |
| **Python SDK** | 一行代码提取文档，支持批量 + 异步 | pip install mineru-open-sdk |
| **Go SDK** | 零依赖 Go 库，API 与 Python SDK 对齐 | go get github.com/opendatalab/MinerU-Ecosystem/sdk/go@latest |
| **TypeScript SDK** | Node.js/Bun/Deno 可用 | npm install mineru-open-sdk |
| **LangChain 集成** | 直接对接 RAG Pipeline | pip install langchain-mineru |
| **LlamaIndex Reader** | 官方推荐 Reader，PDF 直接转为 Document 格式 | pip install llama-index-readers-mineru |
| **MCP Server** | 让 Claude/Cursor 等 AI 客户端直接调用 MinerU | 配置 JSON，见下方 |

该系列工具现已全部开源，欢迎前往 GitHub 项目主页了解与使用：https://github.com/opendatalab/MinerU-Ecosystem。

如果这些工具对你有帮助，也欢迎顺手点个 **Star** 支持我们。

**CLI**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
# mac&amp;&amp;linux<br />
curl -fsSL https://cdn-mineru.openxlab.org.cn/open-api-cli/install.sh | sh<br />
# windows<br />
curl -fsSL https://cdn-mineru.openxlab.org.cn/open-api-cli/install.sh | sh<br />
<br />
<strong># 轻量解析API</strong><br />
# Print markdown to stdout<br />
mineru-open-api flash-extract report.pdf<br />
<br />
# Extract from URL<br />
mineru-open-api flash-extract https://cdn-mineru.openxlab.org.cn/demo/example.pdf<br />
<br />
<strong># 精准解析API</strong><br />
# Authenticate once<br />
mineru-open-api auth<br />
<br />
# Print markdown to stdout<br />
mineru-open-api extract report.pdf<br />
<br />
# Save markdown and docx<br />
mineru-open-api extract report.pdf -f md,docx -o ./results/<br />
<br />
<strong>#网页提取</strong><br />
# Print markdown<br />
mineru-open-api crawl https://mineru.net -o ./results/<br />
<br />
<strong>#批量解析</strong><br />
mineru-open-api extract *.pdf -o ./results/<br />
<br />
更多使用介绍请参考：https://github.com/opendatalab/MinerU-Ecosystem/tree/main/cli/mineru-open-api</td>
</tr>
</tbody>
</table>

**Python SDK**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
pip install mineru-open-sdk<br />
<br />
<strong># 轻量解析API</strong><br />
from mineru import MinerU<br />
<br />
client = MinerU()<br />
result = client.flash_extract("https://cdn-mineru.openxlab.org.cn/demo/example.pdf")<br />
<br />
print(result.markdown)<br />
<br />
<strong># 精准解析API</strong><br />
from mineru import MinerU<br />
<br />
# Get your free token from https://mineru.net/apiManage/token<br />
client = MinerU("your-api-token")<br />
result = client.extract("https://cdn-mineru.openxlab.org.cn/demo/example.pdf")<br />
<br />
print(result.markdown)<br />
print(result.images) # Access extracted images<br />
<br />
更多使用介绍请参考：<br />
https://github.com/opendatalab/MinerU-Ecosystem/tree/main/sdk</td>
</tr>
</tbody>
</table>

**MCP Server 配置示例（Token 请到 [mineru.net](https://mineru.net/apiManage/docs) 创建）：**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">JSON<br />
<strong>运行在本地(推荐)：</strong><br />
需要先安装uvx<br />
{<br />
"mcpServers": {<br />
"mineru": {<br />
"command": "uvx",<br />
"args": ["mineru-open-mcp"],<br />
"env": {<br />
"MINERU_API_TOKEN": "your token" // 可选，不配置即为快速模式<br />
}<br />
}<br />
}<br />
}<br />
<br />
<strong>运行在远端(适合处理文件url，本地文件不便捷):</strong><br />
{<br />
"mcpServers": {<br />
"mineru": {<br />
"type": "streamableHttp",<br />
"url": "https://mcp.mineru.net/mcp",<br />
"env": {<br />
"MINERU_API_TOKEN": "your token"<br />
}<br />
}<br />
}<br />
}</td>
</tr>
</tbody>
</table>

这些工具本质上都是对 API 的上层封装。掌握了 API 的核心用法后，无论使用 CLI、SDK，还是集成到其他框架中，都会更加顺手。接下来的实战部分将主要以 CLI 和 Python SDK 为例展开说明。

**Cursor中使用mineru在线API示例**

**使用MCP示例**

增加mcp配置

对话中直接使用

<img src="03课：MinerU 在线 API 实战教程/03课：MinerU 在线 API 实战教程 - 文档 - media/media/image2.png" style="width:3.25in;height:2.6875in" />

**Cursor中只用SDK示例**

<img src="03课：MinerU 在线 API 实战教程/03课：MinerU 在线 API 实战教程 - 文档 - media/media/image3.png" style="width:1.64583in;height:4.63542in" />

**五、实战 A —— 将有价值的文档或网页变成可复用的飞书知识库**

<img src="03课：MinerU 在线 API 实战教程/03课：MinerU 在线 API 实战教程 - 文档 - media/media/image4.png" style="width:5.75in;height:2.26042in" />

1\. **具体场景**

当你在日常浏览中看到一篇值得保存的产品文档，或者技术文章、行业报告的网页时，可以通过接入飞书的 OpenClaw 调用 MinerU 的 SILL 能力，将内容快速解析并沉淀到飞书知识库，方便后续统一整理、检索和阅读。

2\. **前置准备**

接入飞书OpenClaw ，具体教程详见：飞书官方文档[OpenClaw飞书官方插件上安装更新教程与常见问题](https://www.feishu.cn/content/article/7613711414611463386)

为需要接入OpenClaw的飞书机器人开通创建文档的权限

3\. **接入MineU CLI SKILL**

在与飞书机器人的对话框直接说：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
https://clawhub.ai/mineru-extract/mineru-document-extractor 装一下这个skill</td>
</tr>
</tbody>
</table>

<img src="03课：MinerU 在线 API 实战教程/03课：MinerU 在线 API 实战教程 - 文档 - media/media/image5.png" style="width:5.35417in;height:2.08333in" />

4\. **随手保存知识库，有空的时候再阅读**

<img src="03课：MinerU 在线 API 实战教程/03课：MinerU 在线 API 实战教程 - 文档 - media/media/image6.png" style="width:5.75in;height:3.20833in" />

**六、实战 B —— 三个函数把繁琐的发票快速生成汇总表**

1\. **具体场景**

当你需要整理一批报销发票、采购票据或财务凭证时，无需再逐张查看和手工录入信息。借助 MinerU API，可批量提取发票中的关键字段，并自动汇总为 Excel，显著提升整理效率。

整个流程只需三步：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plaintext<br />
PDF 发票 → MinerU SDK 批量提取 → LLM 结构化提取 → Excel</td>
</tr>
</tbody>
</table>

例如，你有一个文件夹，里面是几十张 PDF 发票。你想要一张 Excel 表格，每行一张发票，包含三列：

|          |            |           |
|:---------|:-----------|:----------|
| 发票号码 | 开票日期   | 价税合计  |
| 01234567 | 2024-03-15 | ¥1,280.00 |
| ...      | ...        | ...       |

你需要对上述发票进行解析生成 Excel。

2\. **前置准备**

安装 mineru-open-sdk

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
pip install mineru-open-sdk openai openpyxl python-dotenv</td>
</tr>
</tbody>
</table>

你需要：

从 [mineru.net](https://mineru.net/apiManage/token) 获取MinerU API Toke

准备一个 OpenAI 兼容接口的 API Key（本教程使用 Moonshot/Kimi，任何兼容接口均可）

在项目目录创建 .env 文件：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plaintext<br />
MINERU_TOKEN=你的MinerU Token<br />
LLM_API_KEY=你的Moonshot API Key<br />
LLM_BASE_URL=https://api.moonshot.cn/v1<br />
LLM_MODEL=moonshot-v1-128k</td>
</tr>
</tbody>
</table>

3\. **完整代码**

整个脚本只有三个函数。关键设计：extract_batch 是 yield 式返回——解完一张就立刻喂给 LLM，不用等全部解完。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Python<br />
"""<br />
MinerU 发票提取器<br />
PDF 发票 → MinerU SDK 批量提取 JSON → LLM 结构化提取 → Excel<br />
"""<br />
import os<br />
import sys<br />
import json<br />
from pathlib import Path<br />
from dotenv import load_dotenv<br />
<br />
from mineru import MinerU, FileParam<br />
from openai import OpenAI<br />
from openpyxl import Workbook<br />
<br />
# -------- 加载环境变量 --------<br />
load_dotenv()<br />
<br />
MINERU_TOKEN = os.getenv("MINERU_TOKEN")<br />
LLM_API_KEY = os.getenv("LLM_API_KEY")<br />
LLM_BASE_URL = os.getenv("LLM_BASE_URL", "https://api.moonshot.cn/v1")<br />
LLM_MODEL = os.getenv("LLM_MODEL", "moonshot-v1-128k")<br />
<br />
EXTRACT_PROMPT = (<br />
'你是一个发票信息提取器。以下是MinerU SDK从发票图片中提取的content_list。'<br />
'每个文本块格式：{"type": "text", "text": "发票号码：144000000000", ...} '<br />
'或 {"type": "table", "table_body": "&lt;table&gt;...&lt;td&gt;价税合计(小写)&lt;/td&gt;'<br />
'&lt;td&gt;¥ 90000.00&lt;/td&gt;...&lt;/table&gt;"}。'<br />
'请找出：1)发票号码(以"发票号码："开头的text冒号后数字) '<br />
'2)开票日期("开票日期："开头，转YYYY-MM-DD) '<br />
'3)价税合计(表格中"价税合计(小写)"对应¥后面数字)。'<br />
'只返回JSON，不要任何解释或思考过程。'<br />
'格式：{"invoice_number":"","date":"","total":""}'<br />
)<br />
<br />
<br />
# -------- 函数 1：MinerU 批量解析，解完一个 yield 一个 --------<br />
def parse_invoices(pdf_dir: str):<br />
"""将目录下所有 PDF/图片通过 MinerU SDK 批量提取，解完一个 yield 一个"""<br />
files = []<br />
for ext in ["*.pdf", "*.jpg", "*.jpeg", "*.png", "*.bmp"]:<br />
files.extend(sorted(Path(pdf_dir).glob(ext)))<br />
if not files:<br />
print("No files found (supported: pdf, jpg, png, bmp)")<br />
return<br />
<br />
client = MinerU(MINERU_TOKEN)<br />
<br />
# extract_batch 支持的全局参数：<br />
# model - 模型版本: "vlm"(推荐) / "pipeline" / "html"，默认自动推断<br />
# ocr - 是否开启 OCR，默认 False<br />
# formula - 是否开启公式识别，默认 True<br />
# table - 是否开启表格识别，默认 True<br />
# language - 文档语言，默认 "ch"<br />
# extra_formats - 额外导出格式: ["docx", "html", "latex"]<br />
# timeout - 轮询超时秒数，默认 1800<br />
# file_params - 按文件覆盖参数，key 为文件路径，value 为 FileParam<br />
<br />
# FileParam 支持的字段：<br />
# pages - 页码范围，如 "1-5"<br />
# ocr - 覆盖全局 OCR 开关<br />
# data_id - 自定义业务标识<br />
<br />
# 发票是扫描件/图片，需要开 OCR；通过 FileParam 按文件设置参数<br />
file_params = {str(f): FileParam(ocr=True) for f in files}<br />
for result in client.extract_batch(<br />
[str(f) for f in files],<br />
table=True, # 发票中的金额在表格里，确保开启<br />
file_params=file_params,<br />
):<br />
print(f" Parsed: {result.filename}")<br />
yield {"filename": result.filename, "content": result.content_list}<br />
client.close()<br />
<br />
<br />
# -------- 函数 2：LLM 提取结构化字段 --------<br />
def extract_fields(invoice: dict, llm: OpenAI) -&gt; dict:<br />
"""将单张发票的 content_list 喂给 LLM，提取发票号码/日期/金额"""<br />
content_str = json.dumps(invoice["content"], ensure_ascii=False)<br />
resp = llm.chat.completions.create(<br />
model=LLM_MODEL,<br />
messages=[{"role": "user", "content": EXTRACT_PROMPT + content_str}],<br />
temperature=0,<br />
)<br />
raw = resp.choices[0].message.content.strip()<br />
try:<br />
fields = json.loads(raw)<br />
except json.JSONDecodeError:<br />
fields = {"invoice_number": "PARSE_ERROR", "date": "", "total": ""}<br />
fields["filename"] = invoice["filename"]<br />
print(f" Extracted: {invoice['filename']}")<br />
return fields<br />
<br />
<br />
# -------- 函数 3：写入 Excel --------<br />
def write_excel(data: list[dict], output: str = "发票汇总.xlsx"):<br />
"""将提取结果写入 Excel"""<br />
wb = Workbook()<br />
ws = wb.active<br />
ws.append(["文件名", "发票号码", "开票日期", "价税合计"])<br />
for row in data:<br />
ws.append([<br />
row.get("filename"),<br />
row.get("invoice_number"),<br />
row.get("date"),<br />
row.get("total")<br />
])<br />
wb.save(output)<br />
print(f"已保存: {output}")<br />
<br />
<br />
# -------- 主流程：流水线式处理 --------<br />
def main():<br />
pdf_dir = sys.argv[1] if len(sys.argv) &gt; 1 else "./invoices"<br />
llm = OpenAI(api_key=LLM_API_KEY, base_url=LLM_BASE_URL)<br />
<br />
results = []<br />
for invoice in parse_invoices(pdf_dir): # 解完一张<br />
results.append(extract_fields(invoice, llm)) # 立刻提取<br />
<br />
write_excel(results)<br />
<br />
if __name__ == "__main__":<br />
main()</td>
</tr>
</tbody>
</table>

**运行：**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
python invoice_extractor.py ./invoices</td>
</tr>
</tbody>
</table>

已在 .env 中配置过 Token 和 API Key，直接运行即可。三个函数，一条流水线，PDF 进 Excel 出。

**总结**

本教程覆盖了 MinerU Open API 的核心路径：

**认识 API** — 两套 API 各有定位，精准解析适合生产，轻量解析零门槛

**认识生态** — CLI / SDK / LangChain / MCP，总有一款适合你的工作流

**实战A** — 用 OpenClaw + MinerU Skill，将有价值的文档或网页变成可复用的飞书知识库

**实战B** — SDK + LLM + Excel，三个函将繁琐的发票快速生成汇总表（实战 B）
