**08课：OCR模型对标——基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>课程简介</strong></p>
<p>本次课程将教会大家通过使用 OmniDocBench文档解析评测集和评测工具Dingo，科学评估文档解析模型解析文档的质量，精准定位问题并指导优化方向。</p>
<p><strong>🎯 你将学到什么</strong></p>
<p>理解评测体系 — 掌握 OmniDocBench 评测集的设计思路和核心指标（文本、表格、公式、阅读顺序）</p>
<p>诊断模型问题 — 学会使用 Dingo 工具输出具体的解析问题（漏检、误识别、结构错误等）</p>
<p>指导模型优化 — 把"低分"转化为"具体 Bug"，明确模型迭代方向</p>
<p>工程落地实践 — 掌握 SDK、CLI、SaaS 三种使用方式，直接应用到实际项目中</p>
<p><strong>💡 为什么学本节课</strong></p>
<p>文档解析是 RAG、知识库、数据提取等场景的基础环节。但很多团队面临这些问题：</p>
<p>模型输出质量不稳定，不知道哪里出了问题</p>
<p>评测只看总分，无法定位具体缺陷</p>
<p>缺乏系统化的评测方法论和工具链</p>
<p>这节课帮你建立可诊断、可回归、可对标的评测体系，让模型优化有据可依。</p>
<p><strong>相关资源：</strong></p>
<p>OmniDocBench地址: https://github.com/opendatalab/OmniDocBench/tree/main</p>
<p>OmniDocBench官网地址：https://opendatalab.com/omnidocbench</p>
<p>OmniDocBench 论文：https://arxiv.org/abs/2412.07626</p>
<p>Dingo地址: https://github.com/MigoXLab/dingo</p>
<p>Dingo SAAS地址：https://dingo.openxlab.org.cn/</p>
<p>Dingo SDK仓库地址：https://github.com/MigoXLab/dingo</p></td>
</tr>
</tbody>
</table>

1\. **模型质量评测分层**

模型质量评测分两个层面：训练数据的质量，以及模型本身的评测分析。这两个层面我们都会仔细评估。训练数据生产有自己的 pipeline，我们会评估它的迭代效果，持续提升精度。模型质量也一样，评测之后不断迭代优化。

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image1.jpeg" style="width:5.75in;height:3.44792in" />

2\. **OmniDocBench介绍（输出评测指标）**

2.1 **整体介绍**

OmniDocBench是一个针对真实场景下多样性文档解析评测集，具有以下特点：

**文档类型多样：**该评测集涉及1651个PDF页面，涵盖10种文档类型、4种layout类型，3种语种，15种block级别的标注，5种页面级别的属性，3种文本属性和6种表格属性。覆盖了包含学术文献、财报、报纸、教材、手写笔记等场景

**标注信息丰富：**包含28个block级别（文本段落、标题、表格等）和4个Span级别（文本行、行内公式、角标等，总量超过80k）的文档元素的定位信息，以及每个元素区域的识别结果（文本Text标注，公式LaTeX标注，表格包含LaTeX和HTML两种类型的标注）。OmniDocBench还提供了各个文档组件的阅读顺序的标注。除此之外，在页面和block级别还包含多种属性标签，标注了5种页面属性标签、3种文本属性标签和6种表格属性标签。

**标注质量高：**经过人工筛选，智能标注，人工标注及全量专家质检和大模型质检，数据质量较高。

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image2.png" style="width:5.75in;height:3.35417in" />

多样性

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image3.png" style="width:5.75in;height:3.79167in" />

丰富密集的标注

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image4.png" style="width:5.75in;height:4.5in" />

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image5.png" style="width:5.75in;height:4.66667in" />

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image6.png" style="width:5.75in;height:4.625in" />

多样性的标注

2.2 **评测方法介绍**

> <img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image7.png" style="width:5.75in;height:1.98958in" />
>
> 我们的评测会做得比较细粒度。具体做法是：先把模型预测出来的 Markdown 结果，从中分别提取出表格（table）、公式（formula）和纯文本（text），然后把这些提取的内容与 Ground Truth（GT）进行匹配，逐项计算得分。接下来，我介绍一下对应的评测指标。

|  |  |  |  |
|:--:|:---|:---|:--:|
| **指标** | 方向 | 解释 | **计算方式** |
| Overall | ↑ 越高越好 | 整体质量综合得分 | Overall=((1−Text Edit Distance)×100+Table TEDS+Formula CDM )/ 3 |
| Text(Edit) | ↓ 越低越好 | 文本相似度（基于编辑距离） | 文本编辑距离 |
| Formula(CDM) | ↑ 越高越好 | 公式相似度 | CDM（[项目链接](https://github.com/opendatalab/UniMERNet/tree/main/cdm)） |
| Table（TEDS） | ↑ 越高越好 | 表格相似度（内容+结构） | TEDS |
| Table（TEDS-S） | ↑ 越高越好 | 表格结构相似度 | TEDS-S |
| ReadOrder(Edit) | ↓ 越低越好 | 阅读顺序相似度 | 阅读顺序编辑距离 |

2.3 **OmniDocBench使用介绍**

接下来我们将介绍 OmniDocBench的使用方法。该工具支持对文档解析模型进行细粒度评估，涵盖文本、表格、公式及阅读顺序等多个维度的评测指标。我们提供了预构建的 Docker 镜像，以简化环境配置与运行流程。

评估流程需要 Python 3.10 环境，以及几个系统级依赖（TeX Live、ImageMagick、Ghostscript），这些是用来计算 CDM 公式指标的。我们提供了两种部署方式(详见README [OmniDocBench](https://github.com/opendatalab/OmniDocBench))，推荐使用 Docker 方案。

**一、拉取镜像**

执行以下命令，从 GitHub Container Registry 拉取预置运行环境的镜像：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
docker pull ghcr.io/zeng-weijun/omnidocbench-eval:repro-ubuntu2204</td>
</tr>
</tbody>
</table>

该镜像已包含全部依赖及评测脚本。

**二、执行评测代码**

请提前准备以下三部分内容：

GT文件（JSON 格式，例如 your_gt.json，OmniDocBench的json文件可从[huggingface.co](https://huggingface.co/datasets/opendatalab/OmniDocBench)、[OmniDocBench-opendatalab](https://opendatalab.com/OpenDataLab/OmniDocBench)下载）

模型预测结果文件夹（例如 predictions）

用于存放评测结果的输出目录（例如 output）

运行以下命令（请根据实际路径替换 /path/to/ 对应的部分）：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
docker run --rm \ --entrypoint bash \ -v /path/to/your_gt.json:/workspace/gt/your_gt.json:ro \ -v /path/to/your_predictions:/workspace/data_md/predictions:ro \ -v /path/to/output:/workspace/result \ ghcr.io/zeng-weijun/omnidocbench-eval:repro-ubuntu2204 \ -c 'cat &gt; configs/custom.yaml &lt;&lt; "EOF"end2end_eval: metrics: text_block: metric: [Edit_dist] display_formula: metric: [Edit_dist, CDM] table: metric: [TEDS, Edit_dist] reading_order: metric: [Edit_dist] dataset: dataset_name: end2end_dataset ground_truth: data_path: ./gt/your_gt.json prediction: data_path: ./data_md/predictions match_method: quick_match match_workers: 4 quick_match_truncated_timeout_sec: 300 timeout_fallback_max_chunk_span: 10 timeout_fallback_order_penalty: 0.10EOFpython pdf_validation.py --config configs/custom.yaml'</td>
</tr>
</tbody>
</table>

参数说明：

-v 选项用于将宿主机的目录或文件挂载到容器内部（只读或可写）。

配置文件 custom.yaml 定义了需要评估的指标类型：文本块使用编辑距离，显示公式使用编辑距离与 CDM，表格使用 TEDS 与编辑距离，阅读顺序使用编辑距离。

最后执行 python pdf_validation.py 并指定配置文件，启动评测流程。

**三、获取评测结果**

完成代码执行后，将会获取评测结果，目前OmniDocBenchv1.6的榜单如下：

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image8.png" style="width:5.75in;height:4.16667in" />

OmniDocBench v1.6榜单

榜单地址：https://opendatalab.com/omnidocbench

2.4 **不同模型在OmniDocBench榜单上解析结果举例**

<table>
<colgroup>
<col style="width: 5%" />
<col style="width: 22%" />
<col style="width: 22%" />
<col style="width: 21%" />
<col style="width: 27%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: center;"><strong>模块</strong></td>
<td style="text-align: center;"><strong>原图</strong></td>
<td style="text-align: center;"><strong>MinerU2.5 Pro</strong></td>
<td style="text-align: center;"><strong>PaddleOCR_VL 1.5</strong></td>
<td style="text-align: center;"><strong>GLM OCR</strong></td>
</tr>
<tr>
<td style="text-align: center;"><strong>整页表格</strong></td>
<td style="text-align: center;"><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image9.png" style="width:1.11458in;height:1.57292in" /></td>
<td style="text-align: center;"><p><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image10.png" style="width:1.11458in;height:1.5in" /></p>
<p>解析正确</p></td>
<td style="text-align: center;"><p><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image11.png" style="width:1.08333in;height:1.44792in" /></p>
<p>最右边单元格结构错误</p></td>
<td style="text-align: center;"><p><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image12.png" style="width:1.41667in;height:1.54167in" /></p>
<p>右侧单元格结构解析错误</p></td>
</tr>
<tr>
<td style="text-align: center;"><strong>旋转表格</strong></td>
<td style="text-align: center;"><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image13.png" style="width:1.11458in;height:1.57292in" /></td>
<td style="text-align: center;"><p><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image14.png" style="width:1.11458in;height:0.47917in" /></p>
<p>解析正确</p></td>
<td style="text-align: center;"><p><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image15.png" style="width:1.08333in;height:0.44792in" /></p>
<p>表格结构严重错误</p></td>
<td style="text-align: center;"><p><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image16.png" style="width:1.41667in;height:0.32292in" /></p>
<p>表格结构错误</p></td>
</tr>
<tr>
<td style="text-align: center;"><strong>公式</strong></td>
<td style="text-align: center;"><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image17.png" style="width:1.11458in;height:1.4375in" /></td>
<td style="text-align: center;"><p><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image18.png" style="width:1.11458in;height:1.84375in" /></p>
<p>解析正确</p></td>
<td style="text-align: center;"><p><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image19.png" style="width:1.08333in;height:1.30208in" /></p>
<p>公式渲染错误</p></td>
<td style="text-align: center;"><p><img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image20.png" style="width:1.41667in;height:1.66667in" /></p>
<p>公式识别错误</p></td>
</tr>
</tbody>
</table>

3\. **Dingo介绍（输出解析问题）**

|  |
|:---|
| 使用参考：[Dingo 仓库 README](https://github.com/MigoXLab/dingo) 与中文 README（若仓库提供 README_zh-CN.md）。 |

前面我们讲了如何通过 OmniDocBench 获取文档解析模型的各项指标。但拿到指标之后，怎么用它来指导模型优化呢？另外，对于不在 OmniDocBench 评测范围内的文档解析结果，如果发现了问题，除了人工一张张去看，还有没有更自动化的方式？答案是肯定的——Dingo 就是这样一款能自动化筛查数据问题的工具。

接下来，我们来介绍这款 AI 数据质量评测工具——Dingo。它可以输出文档解析数据中存在的具体问题，广泛适用于训练数据生产检查、模型质检等环节。Dingo 的输入非常简单：任意一张待解析的图片，以及对应的解析结果（可以是 Markdown 格式，也可以是布局框）。通过多模态大模型，配合精心设计的 Prompt，Dingo 会对照原图和识别结果，自动输出解析过程中存在的问题。

在 MinerU 的迭代过程中，我们沉淀出了几大类问题类型，形成了一套完整的问题指标体系。下面请大家跟着这张表，来全面了解一下。

|  |  |  |
|:--:|:--:|:--:|
| **问题类型** | **具体含义** | **对应Dingo的检测规则** |
| layout检测遗漏错误 | 页面上的元素完全漏检 | VLMLayoutQuality |
| layout检测不准错误 | 框冗余、框不完整、框重叠 | VLMLayoutQuality |
| layout检测粒度错误 | 表格识别成文本 | VLMLayoutQuality |
|  | 表格识别成图片 | VLMLayoutQuality |
|  | 表格识别成公式 | VLMLayoutQuality |
|  | 公式识别成表格 | VLMLayoutQuality |
|  | 公式识别成图片 | VLMLayoutQuality |
|  | 公式识别成文本 | VLMLayoutQuality |
|  | 正文识别成页眉、页脚、边栏 | VLMLayoutQuality |
|  | 正文识别成图片 | VLMLayoutQuality |
|  | 图片识别成文本 | VLMLayoutQuality |
| 阅读顺序错误 | 阅读顺序错误 | VLMLayoutQuality |
| 公式识别问题 | 公式字符识别错误 | LLMMinerURecognizeQuality、VLMDocumentParsingOCRTrain |
|  | 公式级别错误-渲染错误 | LLMMinerURecognizeQuality、VLMDocumentParsingOCRTrain |
|  | 检测不准 | LLMMinerURecognizeQuality、VLMDocumentParsingOCRTrain |
|  | 公式不合理分段错误 | LLMMinerURecognizeQuality、VLMDocumentParsingOCRTrain |
| 表格识别 | 表格输出otsl格式有误导致转换失败 | LLMMinerURecognizeQuality、VLMDocumentParsingOCRTrain |
|  | 表格结构错误（结构造成的内容丢失也算在里面） | LLMMinerURecognizeQuality、VLMDocumentParsingOCRTrain |
|  | 表格内容错误（结构是对的，仅文本错） | LLMMinerURecognizeQuality、VLMDocumentParsingOCRTrain |
|  | 表格内容模型输出重复 | LLMMinerURecognizeQuality、VLMDocumentParsingOCRTrain |

3.1 **Dingo使用举例**

3.1.1 **SDK模式：评估数据集**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Python<br />
from pathlib import Path<br />
<br />
from dingo.config import InputArgs<br />
from dingo.exec import Executor<br />
<br />
# 获取项目根目录<br />
PROJECT_ROOT = Path(__file__).parent.parent.parent<br />
<br />
if __name__ == '__main__':<br />
input_data = {<br />
"input_path": str(PROJECT_ROOT / "test/data/test_layout_quality.jsonl"),<br />
"dataset": {<br />
"source": "local",<br />
"format": "image",<br />
},<br />
"executor": {<br />
"result_save": {<br />
"bad": True,<br />
"good": True<br />
}<br />
},<br />
"evaluator": [<br />
{<br />
"fields": {"id": "id", "content": "pred", "image": "image_path"},<br />
"evals": [<br />
{"name": "VLMLayoutQuality", "config": {"model": "", "key": "", "api_url": ""}},<br />
]<br />
}<br />
]<br />
}<br />
input_args = InputArgs(**input_data)<br />
executor = Executor.exec_map["local"](input_args)<br />
result = executor.execute()<br />
print(result)</td>
</tr>
</tbody>
</table>

3.1.2 **CLI模式**

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
"input_path": "test/data/test_layout_quality.jsonl",<br />
"dataset": {<br />
"source": "local",<br />
"format": "image"<br />
},<br />
"evaluator": [<br />
{<br />
"evals": [<br />
{"name": "VLMLayoutQuality"}<br />
]<br />
}<br />
]<br />
}</td>
</tr>
</tbody>
</table>

3.2 **Dingo SAAS 使用举例**

3.2.1 **Quick-Start模式-Playground**

3.2.1.1 **以VLMLayoutQuality为例**

数据示例：https://github.com/MigoXLab/dingo/blob/1e33a80975e040360bc6a94abfbee8ff737fe2e6/test/data/test_layout_quality.jsonl

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image21.png" style="width:5.75in;height:2.38542in" />

这里需要关注下模型输出会存在不稳定的情况，但是整体上对人工总结提效还是很有帮助的。

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image22.png" style="width:5.75in;height:1.71875in" />

**可视化样例**

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image23.png" style="width:5.75in;height:8.21875in" />

3.2.1.2 **以LLMMinerURecognizeQuality为例**

https://github.com/MigoXLab/dingo/blob/1e33a80975e040360bc6a94abfbee8ff737fe2e6/test/data/test_document_OCR_recognize.jsonl

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image24.png" style="width:5.75in;height:2.77083in" />

3.2.1.3 **以VLMDocumentParsingOCRTrain为例：**

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image25.png" style="width:5.75in;height:2.13542in" />

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image26.png" style="width:5.75in;height:2.22917in" />

**3.2.2 批量跑测模式**

**3.2.2.1 以LLMMinerURecognizeQuality为例：**

https://github.com/MigoXLab/dingo/blob/1e33a80975e040360bc6a94abfbee8ff737fe2e6/test/data/test_document_OCR_recognize.jsonl

**3.2.2.2 创建数据集**

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image27.png" style="width:5.75in;height:1.55208in" />

**3.2.2.3 创建实验**

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image28.png" style="width:5.75in;height:4.41667in" />

**3.2.2.4 实验结果：**

<img src="08课：OCR模型对标：基于 OmniDocBench 与 Dingo 的 MinerU 模型深度评测 - 文档 - media/media/image29.png" style="width:5.75in;height:1.875in" />

4\. **课后思考**

请简述为什么在文档解析模型评测中，不能只看总分（如 Overall 得分），还需要使用 Dingo 这类工具输出具体问题类型（如漏检、结构错误等）？
