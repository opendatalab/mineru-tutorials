**07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>Dingo SAAS地址：https://dingo.openxlab.org.cn/</p>
<p>Dingo SDK仓库地址：https://github.com/MigoXLab/dingo</p>
<p>MinerU-HTML论文地址：https://arxiv.org/abs/2511.23119</p>
<p>WebMainBench评测仓库地址：https://github.com/opendatalab/WebMainBench/</p>
<p>WebMainBench数据集地址：https://huggingface.co/datasets/opendatalab/WebMainBench</p></td>
</tr>
</tbody>
</table>

1\. **HTML解析质量评测方法介绍**

HTML解析质量评测旨在系统化评估网页正文抽取的准确性、完整性，核心解决"如何量化判断一个解析工具好不好"的问题。

<table>
<colgroup>
<col style="width: 18%" />
<col style="width: 81%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: center;"><strong>维度</strong></td>
<td style="text-align: center;"><strong>关键内容</strong></td>
</tr>
<tr>
<td style="text-align: center;">评测目标</td>
<td style="text-align: center;">正文提取准确率、噪声过滤率、富内容（代码/公式/表格等）保真度</td>
</tr>
<tr>
<td style="text-align: center;">主流解析方法</td>
<td style="text-align: center;">规则启发式（Readability/Trafilatura）→ 机器学习 → 大模型生成（ReaderLM）→ 混合架构（MinerU-HTML）</td>
</tr>
<tr>
<td style="text-align: center;">主流的评测方法</td>
<td style="text-align: center;">离线评测（固定的人工标注评测集，如WebMainBench，WCEB等） ；<br />
在线评测（无固定groundtruth，一般依赖LLM-as-judge，适用于大规模、黑盒系统对比，如dingo，deepeval等）；</td>
</tr>
<tr>
<td style="text-align: center;">核心指标</td>
<td style="text-align: center;">ROUGE-5 F1（文本相似度）、细粒度编辑距离（text/code/formula/table）、结构保真度（TEDS）、吞吐量（pages/sec）</td>
</tr>
<tr>
<td style="text-align: center;">数据要求</td>
<td style="text-align: center;">人工标注ground truth、多语言/多域名覆盖、难度分级、富内容精细标注</td>
</tr>
</tbody>
</table>

👉 第2章将介绍 WebMainBench——首个支持富内容、多语言、难度分级的高精度HTML解析评测基准；

👉 第3章将展示如何用 Dingo 实现规模化、可视化的解析质量量化在线评估。

2\. **WebMainBench评测介绍**

仓库地址：https://github.com/opendatalab/WebMainBench/

数据集地址：https://huggingface.co/datasets/opendatalab/WebMainBench

2.1 **评测集介绍**

WebMainBench 是一个用于评测网页正文抽取质量的高精度基准，提供：

7,809 页、100% 人工标注的评测数据集，覆盖 5,434 个独立域名、150 个顶级域名和 46 种语言。

545 条样本子集，附带人工校准的 ground-truth markdown（groundtruth_content），支持文本、代码、公式、表格维度的细粒度指标评测。

2.2 **评测指标**

2.2.1 **ROUGE-N F1**

所有抽取内容通过 html2text 转换为标准 Markdown，再用 ROUGE-N（N=5，jieba 分词）评分。

2.2.2 **细粒度编辑距离指标**

基于 545 条人工校准 groundtruth_content 的子集计算：

![](07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image1.png)

**点击图片可查看完整电子表格**

所有分数范围为 \[0, 1\]，越高越好。

2.2.3 **评测架构**

<img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image2.png" style="width:5.75in;height:2.67708in" />

核心模块：

![](07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image3.png)

**点击图片可查看完整电子表格**

2.2.4 **评测榜单**

**ROUGE-N F1 — 全量数据集（7,809 条）**

<img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image4.png" style="width:5.27083in;height:3.42708in" />

结论：在WebMainBench的全量数据集评测中，MinerU-HTML (Dripper)表现卓越。其在Simple、Mid、Hard三个难度级别上均名列前茅，不仅大幅超越了Readability、Trafilatura等传统工具，甚至在部分指标上接近DeepSeek-V3.2等大型语言模型在特定模式下的表现。

**细粒度指标（545 条子集）**

<img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image5.png" style="width:5.75in;height:1.625in" />

结论：MinerU-HTML 提取完整性，准确性行业领先；

3\. **基于Dingo量化评测介绍**

仓库地址：https://github.com/MigoXLab/dingo

saas平台地址：https://dingo.openxlab.org.cn/

3.1 **Dingo数据质量介绍**

Dingo 是一款全面的 AI 数据、模型和应用质量评估工具，专为机器学习工程师、数据工程师和 AI 研究人员设计。它帮助你系统化地评估和改进训练数据、微调数据集和生产AI系统的质量。

3.2 **为什么选择 Dingo?**

**生产级质量检查：**从预训练数据集到 RAG 系统，确保你的 AI 获得高质量数据

**多数据源集成：**无缝连接本地文件、SQL 数据库（PostgreSQL/MySQL/SQLite）、HuggingFace 数据集和 S3 存储

**多字段评估：**对不同字段并行应用不同的质量规则（例如：对 isbn 字段进行 ISBN 验证，对 title 字段进行文本质量检查）

**RAG 系统评估：**使用 5 个学术支持的指标全面评估检索和生成质量

**LLM、规则和智能体混合：**结合快速启发式规则（30+ 内置规则）和基于 LLM 的深度评估

**灵活执行：**本地运行快速迭代，或使用 Spark 扩展到数十亿级数据集

**丰富报告：**详细的质量报告，带有 GUI 可视化和字段级洞察

3.3 **架构图**

<img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image6.png" style="width:5.75in;height:3.20833in" />

3.4 **快速启动**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
# 核心包（包含规则评估、LLM 评估、MCP 服务、数据源支持）<br />
pip install dingo-python<br />
<br />
# 安装 HHEM 幻觉检测模型（需要 transformers + torch）<br />
pip install "dingo-python[hhem]"<br />
<br />
# 安装全部功能（HHEM + Agent）<br />
pip install "dingo-python[all]"</td>
</tr>
</tbody>
</table>

3.5 **Dingo 使用示例**

3.5.1 **SDK模式：评估LLM对话数据**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Python<br />
from dingo.config.input_args import EvaluatorLLMArgs<br />
from dingo.io.input import Data<br />
from dingo.model.llm.text_quality.llm_text_quality_v4 import LLMTextQualityV4<br />
from dingo.model.rule.rule_common import RuleSpecialCharacter<br />
<br />
data = Data(<br />
data_id='123',<br />
prompt="hello, introduce the world",<br />
content="�I am 8 years old. ^I love apple because:"<br />
)<br />
<br />
<br />
def llm():<br />
LLMTextQualityV4.dynamic_config = EvaluatorLLMArgs(<br />
key='YOUR_API_KEY',<br />
api_url='https://api.openai.com/v1/chat/completions',<br />
model='gpt-4o',<br />
)<br />
res = LLMTextQualityV4.eval(data)<br />
print(res)<br />
<br />
def rule():<br />
res = RuleSpecialCharacter().eval(data)<br />
print(res)<br />
<br />
rule()</td>
</tr>
</tbody>
</table>

3.5.2 **SDK模式：评估数据集**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Python<br />
from dingo.config import InputArgs<br />
from dingo.exec import Executor<br />
<br />
# Evaluate a dataset from Hugging Face<br />
if __name__ == '__main__':<br />
input_data = {<br />
"input_path": "tatsu-lab/alpaca", # Dataset from Hugging Face<br />
"dataset": {<br />
"source": "hugging_face",<br />
"format": "plaintext" # Format: plaintext<br />
},<br />
"executor": {<br />
"result_save": {<br />
"bad": True # Save evaluation results<br />
}<br />
},<br />
"evaluator": [<br />
{<br />
"evals": [<br />
{"name": "RuleColonEnd"},<br />
{"name": "RuleSpecialCharacter"}<br />
]<br />
}<br />
]<br />
}<br />
<br />
input_args = InputArgs(**input_data)<br />
executor = Executor.exec_map["local"](input_args)<br />
result = executor.execute()<br />
print(result)</td>
</tr>
</tbody>
</table>

3.5.3 **CLI模式**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
$ dingo eval --input .github/env/local_plaintext.json<br />
<br />
$ cat .github/env/local_plaintext.json<br />
{<br />
"input_path": "test/data/test_local_plaintext.txt",<br />
"dataset": {<br />
"source": "local",<br />
"format": "plaintext"<br />
},<br />
"evaluator": [<br />
{<br />
"evals": [<br />
{"name": "RuleColonEnd"}<br />
]<br />
}<br />
]<br />
}</td>
</tr>
</tbody>
</table>

3.5.4 **MCP Server模式**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
# Start MCP server (SSE transport, default port 8000)<br />
$ dingo serve<br />
<br />
# stdio transport (for Claude Desktop)<br />
$ dingo serve --transport stdio</td>
</tr>
</tbody>
</table>

3.6 **Dingo SAAS平台介绍**

Dingo SaaS 是基于开源项目 [Dingo](https://github.com/MigoXLab/dingo) 构建的企业级数据质量评估与管理平台。它将 Dingo 强大的数据质量评估能力产品化，提供 Web 界面、API 服务和完整的数据管理工作流。

3.6.1 **核心特性**

✅ **数据集管理** - 支持多种数据源（本地文件、HuggingFace、S3等）

✅ **指标配置** - 灵活的评估规则和 LLM 评估器管理

✅ **实验执行** - 支持实验创建、执行、停止和日志查看

✅ **可视化报告** - 交互式仪表盘和详细的质量分析

✅ **API & SDK** - RESTful API 和 Python SDK

3.6.2 **功能演示**

Dingo SaaS 平台提供（“Quick Try”和“Full Pipeline”）两种评测模式，满足不同场景需求：

<img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image7.png" style="width:5.75in;height:2.67708in" />

3.6.2.1 **Quick Try 模式 - 快速验证**

**适用场景：**单条数据质量快速检查、规则调试、Prompt 工程验证

**操作流程：**

在平台首页选择"Quick Try"

选择评估指标（如 LLMTextQualityV5、RuleSpecialCharacter 等）

输入待评测文本/图片等（支持纯文本/JSON 格式）

即时获取评分和详细诊断报告

**优势：**秒级响应，无需创建实验，适合开发阶段快速迭代

<img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image8.png" style="width:5.75in;height:2.58333in" />

3.6.2.2 **Full Pipeline 模式 - 批量评测**

**适用场景：**大规模数据集质量评估、模型对比实验、生产环境监控

**操作流程：**

上传数据集（支持本地文件、HuggingFace、S3 等多源）

配置评估规则组合（可混合 LLM 评估器和规则评估器）

创建实验并执行（支持本地/Spark 分布式执行）

查看可视化报告（字段级质量分布、问题样本定位、趋势分析）

**优势：**提供完整的实验管理和报告导出功能

<img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image9.png" style="width:5.75in;height:2.67708in" />

3.6.3 **实战：MinerU-Html vs. Trafilatura抽取结果对比评测**

对比测试数据地址：[link](https://github.com/MigoXLab/dingo/blob/dev/docs/mineru_xunlianying/demo.jsonl)

3.6.3.1 **数学公式抽取对比**

**测试样本：**选取包含 LaTeX 数学公式的网页（[原始网页link](https://terrytao.wordpress.com/2008/05/16/285g-lecture-12-high-curvature-regions-of-ricci-flow-and-%ce%ba-solutions/)）

**评估Metric：**LLMHtmlExtractCompareV3

**对比维度：**

公式完整性：是否正确保留 \$\$...\$\$ 或 \\...\\ 定界符

结构保真度：分式、上下标、矩阵等复杂结构是否完整

特殊符号：希腊字母、运算符、箭头等是否正确转译

**评测结果：**

MinerU-HTML 显著优于 Trafilatura，后者系统性剥离数学符号和公式，将 \$\kappa\$-solutions 错误转换为 -solutions（丢失希腊字母）；

|  |  |
|:--:|:--:|
| <img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image10.png" style="width:2.71875in;height:1.46875in" /> | <img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image11.png" style="width:2.6875in;height:1.46875in" /> |

<img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image12.png" style="width:5.75in;height:2.67708in" />

3.6.3.2 **代码数据抽取对比**

**测试样本：**包含多语言代码块（Python/JavaScript/C++）的技术博客（[原始网页link](https://www.w3school.com.cn/python/numpy_creating_arrays.asp)）

**评估Metric：**LLMHtmlExtractCompareV3

**对比维度：**

语法完整性：缩进、括号匹配、关键字是否完整

语言标识：是否正确保留代码语言标记（如 \`\`\`python）

内联代码：反引号包裹的短代码是否保留

**评测结果：**

MinerU-HTML 显著优于 Trafilatura，后者代码块严重损坏，缩进结构完全破坏等；

|  |  |
|:--:|:--:|
| <img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image13.png" style="width:2.65625in;height:1.625in" /> | <img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image14.png" style="width:2.75in;height:1.63542in" /> |

<img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image15.png" style="width:5.75in;height:2.60417in" />

3.6.3.3 **表格数据抽取对比**

**测试样本：**包含复杂表格（合并单元格、嵌套表头、数值对齐）的网页（[原始网页link](https://nauticalstudies.org/category/psychico/)）

**评估Metric：**LLMHtmlExtractCompareV3

**对比维度：**

结构保真度：行列关系、表头层次是否保留

内容完整性：单元格文本是否完整提取

语义对齐：数值与对应标签是否正确关联

**评测结果：**

MinerU-HTML 显著优于 Trafilatura，表格结构完全损坏，列对齐混乱，无法区分单元格边界等

|  |  |
|:--:|:--:|
| <img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image16.png" style="width:2.72917in;height:1.45833in" /> | <img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image17.png" style="width:2.67708in;height:1.45833in" /> |

<img src="07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法/07课：MinerU-HTML 解析效能验证：基于 Dingo 的量化评测方法 - 文档 - media/media/image8.png" style="width:5.75in;height:2.58333in" />

4\. **快速练习**

**第一步：**登陆平台（https://dingo.openxlab.org.cn/）；

**第二步：**点击“Quick Try”模式，选择用LLMTextQualityV5 metric评估这段话“今天星期一今天星期一今天星期一今天星期一今天星期一今天星期一今天星期一今天星期一今天星期一今天星期一今天星期一今天星期一今天星期一”；

**第三步：**点击“Experiments”，执行“Demo Experiment (LLM)”和“Demo Experiment (Rule)”两个任务，并查看评测结果；
