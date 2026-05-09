**02课：MinerU 多环境部署实践：从开源容器化到信创生态适配**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>重要提示：</strong></p>
<p>在开始部署前，请各位同学确保具备基本的 Linux 使用能力（如命令行操作、环境变量配置等），并已安装好基础依赖工具，包括 Python（建议 3.10 及以上）、conda（或 miniconda）以及 pip 包管理工具。同时，请确保当前部署环境具备访问外部资源的能力（如 PyPI、GitHub、Hugging Face 等），以便顺利下载依赖包和模型文件；若处于受限网络环境，建议提前准备离线安装包或通过代理方式进行访问。</p>
<p>此外，建议各位同学具备一定的基础知识，包括：Python 基础编程、虚拟环境管理（conda/pip）、基础的深度学习/大模型使用经验（如模型加载与推理）、以及简单的 GPU/CUDA 环境配置认知（如需使用 vLLM 后端）。这些知识将有助于更顺利地完成部署与问题排查。</p></td>
</tr>
</tbody>
</table>

**第一部分：MinerU 项目**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p><strong>本文面向具备基础命令行与 Docker 使用经验的同学，目标是帮助你用一份教程走通 MinerU 的几条常见路径：</strong></p>
<p>用 uv 创建虚拟环境并通过 PyPI 安装 MinerU</p>
<p>基于源码做可编辑安装，便于二开与调试</p>
<p>用 Docker / Docker Compose 快速完成单机部署与服务化</p>
<p>在国产算力平台上一键部署并使用 MinerU</p></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p>推荐大家问题优先使用问答神器<strong>deepwiki</strong>，为大家提供问题排查思路：https://deepwiki.com/opendatalab/MinerU</p>
<p>MinerU 开源项目：https://github.com/opendatalab/MinerU</p>
<p>MinerU 开源模型：https://huggingface.co/opendatalab/MinerU2.5-2509-1.2B</p></td>
</tr>
</tbody>
</table>

**一、部署路线图**

<img src="02课：MinerU 多环境部署实践：从开源容器化到信创生态适配 - 文档 - media/media/image1.jpeg" style="width:5.75in;height:5.75in" />

在开始安装之前，先确定自己应该走哪条路径。

|  |  |  |
|:--:|:--:|:--:|
| 场景 | 推荐路径 | 适用原因 |
| 本机没有 GPU，体验本地部署流程 | uv + mineru\[core\] + -b pipeline | 兼容性最好，可在纯 CPU 环境运行 |
| 本机有 NVIDIA GPU / Apple Silicon，希望快速体验 | uv + mineru\[all\] | 默认会尝试 cuda / mps 加速 |
| 希望避免本地依赖冲突，快速交付可复用环境 | Docker | 镜像已封装核心依赖，适合交付与复现 |
| 希望暴露 API / WebUI / OpenAI-compatible 服务 | Docker Compose | 可以直接启动 mineru-api、mineru-gradio、mineru-router、mineru-openai-server |
| 希望做二次开发、调试源码、提 PR | 源码安装 | 支持 uv pip install -e .\[all\] 可编辑模式 |
| 使用国产算力平台 | https://opendatalab.github.io/MinerU/zh/usage/ | 支持10余种国产算力平台部署 |

**二、本地使用虚拟环境安装**

1\. **安装 uv 并创建虚拟环境**

1.1 **安装 uv**

建议先升级 pip，再安装 uv：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
pip install --upgrade pip -i https://mirrors.aliyun.com/pypi/simple<br />
pip install uv -i https://mirrors.aliyun.com/pypi/simple<br />
uv --version</td>
</tr>
</tbody>
</table>

1.2 **创建虚拟环境**

MinerU 当前要求 Python 3.10-3.13（Linux / macOS）或3.10-3.12（Windows）。  
教程中使用3.12版本保证兼容性，建议在项目目录或单独工作目录中创建 .venv：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
uv venv .venv312 --python 3.12</td>
</tr>
</tbody>
</table>

Linux / macOS 激活方式：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
source .venv312/bin/activate<br />
python --version</td>
</tr>
</tbody>
</table>

Windows PowerShell 激活方式：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">PowerShell<br />
.venv312\Scripts\Activate.ps1<br />
python --version</td>
</tr>
</tbody>
</table>

2\. **在虚拟环境中安装MinerU**

您可以根据部署路线图中自身需求，选择直接安装pypi包或源码安装

2.1 **通过 PyPI 安装 MinerU**

对绝大多数用户，最直接的方式是安装 mineru\[all\]：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
uv pip install -U "mineru[all]" -i https://mirrors.aliyun.com/pypi/simple</td>
</tr>
</tbody>
</table>

这组依赖包含：

core：命令行、fastapi、gradio、router 等核心能力

vllm：Linux 下的 VLM 推理加速能力

lmdeploy：Windows 下的 VLM 推理加速能力

mlx：macOS 下的 VLM 推理加速能力

安装完成后建议做一次自检：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
mineru --version<br />
mineru --help<br />
mineru-api --help<br />
mineru-gradio --help<br />
mineru-router --help</td>
</tr>
</tbody>
</table>

2.2 **通过源码安装 MinerU**

当你需要调试代码、验证 PR、修改本地逻辑时，使用源码安装更合适。

2.2.1 **克隆仓库并进入目录**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
git clone https://github.com/opendatalab/MinerU.git<br />
cd MinerU</td>
</tr>
</tbody>
</table>

2.2.2 **使用可编辑模式安装**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
uv pip install -e .[all] -i https://mirrors.aliyun.com/pypi/simple</td>
</tr>
</tbody>
</table>

源码安装适合下面这些场景：

你需要在 mineru/ 目录下直接修改代码并立刻生效

你需要阅读 pyproject.toml 确认依赖分组与脚本入口

你需要基于仓库中的 demo/、docs/、docker/ 目录做本地演示或培训

2.3 **模型源与模型下载**

MinerU 默认使用 HuggingFace 作为模型源；在中国大陆网络环境下，通常建议切换为 ModelScope。

2.3.1 **临时切换模型源**

程序会在首次运行时自动通过预设的模型源缓存模型到本地，您可以配置模型源后直接开始使用

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
export MINERU_MODEL_SOURCE=modelscope</td>
</tr>
</tbody>
</table>

2.3.2 **下载模型到本地**

如果你希望模型文件在本地缓存并在后续过程中从本地加载模型，可以执行：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
mineru-models-download</td>
</tr>
</tbody>
</table>

下载完成后，可以切换到本地模型模式：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
export MINERU_MODEL_SOURCE=local</td>
</tr>
</tbody>
</table>

2.4 **本地简单使用**

下面直接使用仓库中的 demo 文件完成几次最常见的本地验证。

2.4.1 **解析文档（图片、pdf、docx）**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
# input_path可以是文件夹也可以是文件<br />
# output_path需要是文件夹<br />
mineru -p &lt;input_path&gt; -o &lt;output_path&gt;</td>
</tr>
</tbody>
</table>

如果当前机器没有可用 GPU，需要明确指定 pipeline 后端：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
mineru -p &lt;input_path&gt; -o &lt;output_path&gt; -b pipeline</td>
</tr>
</tbody>
</table>

2.4.2 **常用输出怎么看**

一次解析结束后，通常至少会看到以下几类输出：

.md、content_list.json：便于做检索、RAG、内容消费

content_list_v2.json：content_list.json的迭代版本，保留信息更多，目前处于开发中，结构可能会有调整

layout.pdf：查看布局分析与阅读顺序是否正确

span.pdf：仅 pipeline 后端提供，用于定位文本丢失、公式和 OCR 问题

model.json：内置模型一级输出，结构不统一，用于模型开发调试

middle.json：不同类型model.json二次处理得到，结构一致，用于二次开发

建议第一次使用时，重点关注以下几类产出：

.md 是否能够在阅读器中渲染（推荐使用vscode查看）

layout.pdf 中阅读顺序编号是否合理

middle.json 是否满足后续系统集成需求

**三、使用 Docker 部署 MinerU**

如果你希望尽量减少本地环境差异，或者需要把环境交付给其他同学，优先使用 Docker。

使用 Docker 部署前需要注意：

设备包含Volta及以后架构的显卡，且可用显存大于等于8G。

物理机的显卡驱动应支持CUDA 12.9.1或更高版本，可通过nvidia-smi命令检查驱动版本。

docker中能够访问物理机的显卡设备。

1\. **使用 Dockerfile 构建镜像**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
wget https://gcore.jsdelivr.net/gh/opendatalab/MinerU@master/docker/china/Dockerfile<br />
docker build -t mineru:latest -f Dockerfile .</td>
</tr>
</tbody>
</table>

这个镜像有几个关键特征：

基于 vllm/vllm-openai:v0.11.2 基础镜像已替换成国内加速源

已安装 mineru\[core\]\>=3.0.0 pypi已替换成国内加速源

已在镜像构建阶段执行 mineru-models-download -s modelscope -m all使用国内模型源下载所有模型

入口会默认设置 MINERU_MODEL_SOURCE=local指定使用本地模型文件

2\. **进入容器并做交互式验证**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
docker run --gpus all \<br />
--shm-size 32g \<br />
-p 30000:30000 -p 7860:7860 -p 8000:8000 -p 8002:8002 \<br />
-it mineru:latest \<br />
/bin/bash</td>
</tr>
</tbody>
</table>

进入容器后，可以直接执行：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
mineru -v<br />
mineru -p &lt;input_path&gt; -o &lt;output_path&gt;</td>
</tr>
</tbody>
</table>

如果你的目标只是单机命令行体验，这一步就足够了。

3\. **使用 Docker Compose 启动四类服务**

仓库已经提供 docker/compose.yaml，可以直接启动服务：

3.1 **下载 compose.yaml 文件**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
wget https://gcore.jsdelivr.net/gh/opendatalab/MinerU@master/docker/compose.yaml</td>
</tr>
</tbody>
</table>

3.2 **mineru-openai-server**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
docker compose -f compose.yaml --profile openai-server up -d</td>
</tr>
</tbody>
</table>

适用场景：

暴露 OpenAI-compatible 接口

在另一台机器上通过 vlm-http-client 或 hybrid-http-client 复用服务

OpenAI-compatible 接口仅供mineru的http-client连接使用，不可以通过常规chat

连接方式示例：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
mineru -p ./demo/pdfs/demo1.pdf -o ./output -b vlm-http-client -u http://127.0.0.1:30000</td>
</tr>
</tbody>
</table>

3.3 **mineru-api**

mineru-api即 MinerU 开源项目的 FastAPI 服务命令。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
docker compose -f compose.yaml --profile api up -d</td>
</tr>
</tbody>
</table>

在浏览器中访问 http://127.0.0.1:8000/docs 查看API文档。

健康检查接口：GET /health 返回 protocol_version、processing_window_size、max_concurrent_requests 等服务信息

异步任务提交接口：POST /tasks

同步解析接口：POST /file_parse

任务查询接口：GET /tasks/{task_id}、GET /tasks/{task_id}/result

3.3.1 **注意事项**

上传文件当前支持 PDF、图片与 DOCX

POST /tasks 会立即返回 task_id；POST /file_parse 会在内部提交到同一个任务管理器，等待任务完成后同步返回最终结果。

当任务处于排队状态时，任务提交结果和状态查询结果中可能会返回 queued_ahead 字段，用于表示前方排队任务数。

任务为单进程、进程内状态实现，服务重启、--reload 热重载或多进程部署后不保证仍可查询历史任务状态。

默认任务完成或失败后保留 24 小时，随后自动清理任务状态和输出目录；清理后访问任务状态或结果会返回 404。

可通过环境变量 MINERU_API_TASK_RETENTION_SECONDS 和 MINERU_API_TASK_CLEANUP_INTERVAL_SECONDS 调整保留时长与清理轮询间隔。

3.3.2 **异步任务提交示例**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
curl -X POST http://127.0.0.1:8000/tasks \<br />
-F "files=@demo/pdfs/demo1.pdf" \<br />
-F "return_md=true"</td>
</tr>
</tbody>
</table>

3.3.3 **同步解析示例**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
curl -X POST http://127.0.0.1:8000/file_parse \<br />
-F "files=@demo/pdfs/demo1.pdf" \<br />
-F "return_md=true" \</td>
</tr>
</tbody>
</table>

3.3.4 **轮询任务状态与结果**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
curl http://127.0.0.1:8000/tasks/&lt;task_id&gt;<br />
curl http://127.0.0.1:8000/tasks/&lt;task_id&gt;/result<br />
curl http://127.0.0.1:8000/health</td>
</tr>
</tbody>
</table>

3.3.5 **http异步调用代码示例**

> [Python版本](https://github.com/opendatalab/MinerU/blob/master/demo/demo.py)

3.4 **mineru-router**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
docker compose -f compose.yaml --profile router up -d</td>
</tr>
</tbody>
</table>

适用场景：

mineru-router 对外暴露与 mineru-api 一致的 /health、/tasks、/file_parse、/tasks/{task_id}、/tasks/{task_id}/result 接口。

可重复使用 --upstream-url 聚合多个已有 mineru-api 服务，也可通过 --local-gpus 自动拉起本地 worker。

适用于多服务、多 GPU 和统一入口部署场景。

默认文档地址：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
http://127.0.0.1:8002/docs</td>
</tr>
</tbody>
</table>

3.5 **mineru-gradio**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
docker compose -f compose.yaml --profile gradio up -d</td>
</tr>
</tbody>
</table>

在浏览器中访问 http://127.0.0.1:7860 使用 Gradio WebUI。

未传 --api-url 时，Gradio 会自动拉起可复用的本地 mineru-api；传入 --api-url 时则会复用已有本地或远端服务。

|  |
|:---|
| vllm 会预分配显存，因此同一台机器上不建议同时跑多个争抢显存的 vllm 服务。如果你遇到到显存不足的报错问题，优先检查是否重复开启了多个使用vllm的mineru服务或存在其他服务占用显卡。 |

**四、其他国产算力平台部署（🚀官方支持/❤️社区贡献）**

[昇腾 Ascend](https://opendatalab.github.io/MinerU/zh/usage/acceleration_cards/Ascend/) 🚀

[平头哥 T-Head](https://opendatalab.github.io/MinerU/zh/usage/acceleration_cards/THead/) 🚀

[沐曦 METAX](https://opendatalab.github.io/MinerU/zh/usage/acceleration_cards/METAX/) 🚀

[海光 Hygon](https://opendatalab.github.io/MinerU/zh/usage/acceleration_cards/Hygon/) 🚀

[燧原 Enflame](https://opendatalab.github.io/MinerU/zh/usage/acceleration_cards/Enflame/) 🚀

[摩尔线程 MooreThreads](https://opendatalab.github.io/MinerU/zh/usage/acceleration_cards/MooreThreads/) 🚀

[天数智芯 IluvatarCorex](https://opendatalab.github.io/MinerU/zh/usage/acceleration_cards/IluvatarCorex/) 🚀

[寒武纪 Cambricon](https://opendatalab.github.io/MinerU/zh/usage/acceleration_cards/Cambricon/) 🚀

[昆仑芯 Kunlunxin](https://opendatalab.github.io/MinerU/zh/usage/acceleration_cards/Kunlunxin/) 🚀

[太初元碁 Tecorigin](https://opendatalab.github.io/MinerU/zh/usage/acceleration_cards/Tecorigin/) ❤️

[壁仞 Biren](https://opendatalab.github.io/MinerU/zh/usage/acceleration_cards/Biren/) ❤️

[瀚博 VastAI](https://github.com/opendatalab/MinerU/discussions/4237) ❤️

**五、常见问题**

1\. **首次运行为什么会慢？**

首次运行通常会下载模型，或者初始化本地缓存。只要模型缓存成功，后续启动会明显更快。

2\. **什么时候必须手工指定 -b pipeline？**

当机器没有满足条件的 GPU、希望明确走纯 CPU 路线、或在排障时希望先回退到兼容性最好的路径时，直接指定 -b pipeline。

3\. **Docker 镜像为什么能直接用 local 模型？**

因为 Dockerfile 在构建阶段下载了模型，并在入口中默认设置了 MINERU_MODEL_SOURCE=local。

4\. **mineru-api、mineru-router、mineru-gradio 该怎么选？**

只要标准 HTTP 接口：mineru-api

需要多服务 / 多 GPU 编排：mineru-router

需要浏览器可视化演示：mineru-gradio

5\. **Docker 和本地 uv 安装怎么选？**

学习、调试、读源码：优先 uv

环境交付、标准化部署、减少依赖冲突：优先 Docker

6\. **mineru-api运行的机器没有显卡，希望远程有GPU显卡的服务器运行vlm提供openai接口，如何使用mineru-api服务器连接 vlm提供的openai服务？**

请详细阅读本文档，并参考：https://deepwiki.com/search/mineruapimineru-vlm_51693b8e-71ce-4879-918c-d585a3a84929?mode=fast

**六、建议的练习顺序**

如果你是第一次接触 MinerU，建议按下面的顺序完成练习：

用 uv 创建 .venv，安装 mineru\[all\]

用 demo/pdfs/demo1.pdf 跑通一次 PDF 解析

用 Dockerfile 构建镜像并进入容器验证

用 compose.yaml 起一个 mineru-api 或 mineru-gradio

如果手头有国产算力机器，参考[MinerU 多环境部署实践：从 uv 本地安装到 Docker一键部署](https://aicarrier.feishu.cn/wiki/IOlewRsRMiPi03krjz2cY3Jxnmc#share-WdVOdPeYkoQjGqxcxdQc3lgXnZc)自己尝试在国产算力平台上部署并使用一次MinerU

完成这 5 步之后，你就已经具备了在本地、容器和国产算力场景中部署并使用 MinerU 的基础能力。

**第二部分：MinerU-HTML项目**

在处理大规模预训练语料（如 Common Crawl）时，网页数据的清洗一直是个难题。长期以来，行业重心都在下游的过滤策略上，但对 **HTML 提取** 本身却重视不足。现有的网页语料库多依赖于像 **Trafilatura** 这样基于启发式规则的提取器，面对万亿规模网页数据难以优化和迭代。这类工具难以保留文档结构，且经常损坏代码块、公式和表格等结构化元素。

**MinerU-HTML** 正是为此而生。它由上海人工智能实验室 OpenDataLab 团队研发，通过“将内容提取任务重构为序列标注问题”的创新思路，能够进行语义感知的网页提取，并成功构建了 7.3 万亿 tokens 的高质量语料 [AICC](https://huggingface.co/datasets/opendatalab/AICC)。MinerU-HTML 是一款基于小语言模型（SLM）的高级 HTML 正文提取工具。它能够从复杂的网页 HTML 中准确识别并提取主要内容，自动过滤导航栏、广告和元数据等辅助元素。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td><p><strong>项目链接：</strong>https://github.com/opendatalab/MinerU-HTML</p>
<p><strong>模型链接：</strong>https://huggingface.co/opendatalab/MinerU-HTML</p>
<p><strong>技术报告链接：</strong>https://arxiv.org/pdf/2511.16397v1</p>
<p><strong>在线demo 链接：</strong>https://opendatalab.com/ai-ready/AICC</p></td>
</tr>
</tbody>
</table>

该项目围绕“LLM 驱动的网页正文提取”构建，提供从 HTML 解析到结构化输出的完整处理流水线。通过紧凑格式与正则结构化输出优化推理效率，并兼容 VLLM、Transformers、OpenAI API 等多种推理后端。在能力上，既支持多格式输出（Markdown / JSON / TXT），又具备完善的容错与回退机制，保证在复杂网页环境下的稳定提取。同时采用模块化架构设计，便于扩展与自定义，并配套完整测试体系，确保系统可靠性。

具体来看：

🎯 LLM 驱动提取：利用先进的语言模型智能识别正文内容。

📝 可扩展输出：集成 [<u>MinerU-Webkit</u>](https://github.com/opendatalab/MinerU-Webkit)，支持将提取的 HTML 高效转换为 Markdown、JSON 和 Txt 格式。

⚡ 紧凑格式 (Compact Format)：模型现采用紧凑格式而非原始 JSON 格式，推理速度更快。

⚡ 正则结构化输出：模型现采用正则结构化输出替代自定义 logits 处理器，从而支持使用 vLLM v1 后端进行推理。

🔄 完整的处理流水线：包括 HTML 简化、提示词构建、LLM 推理、结果解析、正文提取等完整流程。

🚀 多推理后端支持：支持 VLLM、Transformers 和 OpenAI API 三种推理后端。

🛡️ 容错机制：支持回退机制（trafilatura、bypass 或 empty），确保处理失败时仍能提取内容。

🔌 模块化设计：架构清晰，易于扩展和自定义。

🧪 完善的测试：包含全面的单元测试和集成测试。

**一、 深度理解：MinerU-HTML 究竟是怎么工作的**

1\. **MinerU-HTML 架构图**

<img src="02课：MinerU 多环境部署实践：从开源容器化到信创生态适配 - 文档 - media/media/image2.png" style="width:5.75in;height:4.77083in" />

MinerU-HTML 是一个面向 HTML 内容的抽取与理解流水线。它从输入的 HTML 页面出发，先进行“HTML 简化”，去除与语义无关的噪声与冗余结构；随后根据目标任务自动“提示词构建”，把页面要点组织成适合大模型推理的上下文与指令。接着进入“LLM 推理”阶段，调用后端的大语言模型产出结构化或半结构化结果。推理完成后，系统首先做“结果解析”，将模型输出还原为可操作的数据结构；并行地，它还会进行“正文提取”，将页面中的主体文本与关键信息抽离出来。得到的内容再经过“格式转换”，统一为所需的下游格式；若中途出现异常或页面不适配，系统通过“回退处理”选择合适的兜底策略，保证流程稳健。

想深入学习相关代码并上手修改，可以理解以下核心概念：

2\. **整体构建**

MinerU-HTML 的处理流水线包含以下步骤：

HTML 简化 (simplify_html)：将原始 HTML 简化为结构化格式，并为每个元素分配唯一的 \_item_id 属性。

提示词构建 (build_prompt)：基于简化后的 HTML 构建 LLM 提示词，引导模型进行内容分类。

LLM 推理 (inference)：利用 LLM 对每个元素进行分类，标记为main（正文）或other（辅助内容）。

结果解析 (parse_result)：解析 LLM 返回的分类结果。

正文提取 (extract_main_html)：根据分类结果从原始 HTML 中提取正文内容。

格式转换 (convert2content)：将提取的 HTML 转换为 Markdown、JSON 或 Txt 格式。

Fallback处理：若上述流程失败，则使用回退机制（trafilatura、bypass 或 empty）进行提取。

trafilatura：使用trafilatura方式提取

bypass：HTML全部返回

empty：返回空

3\. **核心配置**

**配置选项**

MinerUHTMLConfig 支持以下配置：

use_fall_back: 回退类型，可选 'trafilatura'、'bypass' 或 'empty'。

early_load: 是否提前加载模型（默认 True）。

prompt_version: 提示词版本，可选 'v0'、'v1'、'v2'、'compact'、'short_compact'。MinerUHTML 和 MinerUHTML_Transformers 接口默认使用 'short_compact'；MinerUHTML_OpenAI 接口默认使用 'v2'。

response_format: 模型的输出格式。仅允许 'json' 或 'compact'。VLLM/Transformers 默认使用 'compact'；OpenAI 默认使用 'json'。

'json'格式：{"1": "main", "2": "other", "3": "main"}

'compact'格式：1main2other3main

output_format: 目标转换格式，支持 'mm_md'（标准 Markdown）、'md'（带图片的 Markdown）、'json' 和 'txt'。

**不同 prompt_version 的使用说明**

'compact': 用于本地模型推理，返回更简洁的结果（仅保留 JSON 字典中的键和值）。推荐使用 'compact' 模型以获得更快的推理速度。

'v2': 用于 OpenAI API 推理，是经过提示词优化后的结果。

**二、快速上手：本地部署环境**

1\. **环境准备**

为了避免不同代码的环境冲突，建议创建独立的conda环境：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
<em># 创建Python 3.10环境</em><br />
conda create -n html python=3.10<br />
conda activate html</td>
</tr>
</tbody>
</table>

2\. **环境安装**

2.1 **安装**

**使用pip安装MinerU-HTML**，我们推荐使用 vLLM 后端进行安装，以获得最佳性能并适配主要使用场景。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
pip install mineru_html[vllm]</td>
</tr>
</tbody>
</table>

**或通过源码安装MinerU-HTML。**如有开发需求，可选择通过源码安装

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
git clone https://github.com/opendatalab/MinerU-HTML<br />
cd MinerU-HTML<br />
pip install -e .[vllm]</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p>Tips：</p>
<p>如果 VLLM 不适用于您的环境（例如您希望使用不同的推理引擎、缺少 GPU 资源，或需要使用 OpenAI API），可以选择以下后端之一：</p>
<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
# 安装 Transformers 后端依赖<em><br />
</em>pip install mineru_html<em><br />
<br />
</em># 安装 OpenAI 后端依赖<em><br />
</em>pip install mineru_html[openai]</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

2.2 **验证安装**

可以这样判断是否安装成功：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
pip list | grep mineru-html</td>
</tr>
</tbody>
</table>

如果输出类似下面内容，说明安装成功：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
mineru-html 1.1.2</td>
</tr>
</tbody>
</table>

**三、测试推理**

1\. **基础推理**

1.1 **使用 vLLM 后端 （默认）**

默认使用托管在huggingface的模型进行解析，首次使用时会自动下载所需模型文件，后续使用将直接加载本地缓存的模型。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;"><p>Tips:</p>
<p>如果您无法访问huggingface，可以手动下载modelscope模型:</p>
<p>pip install modelscope</p>
<p>modelscope download --model OpenDataLab/MinerU-HTML-v1.1-hunyuan0.5B-compact</p>
<p>如果您无法联网，可参考【常见问题】离线下载并加载模型。</p></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Python<br />
from mineru_html import MinerUHTML, MinerUHTMLConfig<br />
config = MinerUHTMLConfig(<br />
use_fall_back='trafilatura', # 可选 'trafilatura','bypass' 或 'empty'<br />
prompt_version='short_compact', # v1.1 仅支持 'short_compact'<br />
response_format='compact', # v1.1 仅支持 'compact'<br />
early_load=True<br />
)<br />
<br />
# 初始化 MinerUHTML<br />
extractor = MinerUHTML(<br />
model_path='opendatalab/MinerU-HTML-v1.1-hunyuan0.5B-compact', # 首次会联网下载模型<br />
config=config<br />
)<br />
<br />
# 处理单个 HTML<br />
html_content = """<br />
&lt;html&gt;<br />
&lt;body&gt;<br />
&lt;div&gt;<br />
&lt;h1&gt;This is a title&lt;/h1&gt;<br />
&lt;p&gt;This is a paragraph&lt;/p&gt;<br />
&lt;p&gt;This is another paragraph&lt;/p&gt;<br />
&lt;/div&gt;<br />
&lt;div&gt;<br />
&lt;p&gt;Related content&lt;/p&gt;<br />
&lt;p&gt;Advertising content&lt;/p&gt;<br />
&lt;/div&gt;<br />
&lt;/body&gt;<br />
&lt;/html&gt;<br />
"""<br />
result = extractor.process(html_content)<br />
print(result[0].output_data.main_content)<br />
<br />
# 批量处理 HTML<br />
html_list = [html_content, '&lt;html&gt;...&lt;/html&gt;']<br />
results = extractor.process(html_list)<br />
for result in results:<br />
print(result.output_data.main_content)<br />
print(result.case_id)<br />
extractor.llm.cleanup()</td>
</tr>
</tbody>
</table>

1.2 **使用 Transformers 后端**

除了VLLM，我们也提供 Transformers 和 openai 后端推理

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Python<br />
from mineru_html import MinerUHTML_Transformers, MinerUHTMLConfig<br />
<br />
config = MinerUHTMLConfig(<br />
use_fall_back='trafilatura',<br />
prompt_version='short_compact', # v1.1 仅支持 'short_compact'<br />
response_format='compact', # v1.1 仅支持 'compact'<br />
early_load=True<br />
)<br />
<br />
# 初始化 MinerUHTML_Transformers<br />
extractor = MinerUHTML_Transformers(<br />
model_path='opendatalab/MinerU-HTML-v1.1-hunyuan0.5B-compact', # 首次会联网下载模型<br />
config=config,<br />
model_init_kwargs={<br />
'device_map': 'auto',<br />
'dtype': 'auto',<br />
},<br />
model_gen_kwargs={<br />
'max_new_tokens': 16 * 1024,<br />
}<br />
)<br />
<br />
# 处理 HTML<br />
html_content = """<br />
&lt;html&gt;<br />
&lt;body&gt;<br />
&lt;div&gt;<br />
&lt;h1&gt;This is a title&lt;/h1&gt;<br />
&lt;p&gt;This is a paragraph&lt;/p&gt;<br />
&lt;p&gt;This is another paragraph&lt;/p&gt;<br />
&lt;/div&gt;<br />
&lt;div&gt;<br />
&lt;p&gt;Related content&lt;/p&gt;<br />
&lt;p&gt;Advertising content&lt;/p&gt;<br />
&lt;/div&gt;<br />
&lt;/body&gt;<br />
&lt;/html&gt;<br />
"""<br />
result = extractor.process(html_content)<br />
print(result[0].output_data.main_content)</td>
</tr>
</tbody>
</table>

1.3 **使用 OpenAI API 后端**

除了VLLM，我们也提供Transformers和openai后端推理

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Python<br />
from mineru_html import MinerUHTML_OpenAI, MinerUHTMLConfig<br />
<br />
# 创建配置<br />
config = MinerUHTMLConfig(<br />
use_fall_back='trafilatura',<br />
prompt_version='v2', # 可选 v1 或 v2，推荐 v2<br />
response_format='json', # OpenAI 推荐 json<br />
early_load=True<br />
)<br />
<br />
# 初始化 MinerUHTML_OpenAI<br />
extractor = MinerUHTML_OpenAI(<br />
base_url='https://api.openai.com/v1',<br />
sk='your-api-key',<br />
model='gpt-4',<br />
config=config,<br />
retry_times=3<br />
)<br />
<br />
# 处理 HTML<br />
html_content = """<br />
&lt;html&gt;<br />
&lt;body&gt;<br />
&lt;div&gt;<br />
&lt;h1&gt;This is a title&lt;/h1&gt;<br />
&lt;p&gt;This is a paragraph&lt;/p&gt;<br />
&lt;p&gt;This is another paragraph&lt;/p&gt;<br />
&lt;/div&gt;<br />
&lt;div&gt;<br />
&lt;p&gt;Related content&lt;/p&gt;<br />
&lt;p&gt;Advertising content&lt;/p&gt;<br />
&lt;/div&gt;<br />
&lt;/body&gt;<br />
&lt;/html&gt;<br />
"""<br />
result = extractor.process(html_content)<br />
print(result[0].output_data.main_content)</td>
</tr>
</tbody>
</table>

2\. **批量推理**

git库中提供了 100 条 html 数据供测试学习，我们来测试一下。

2.1 **安装对应环境**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
pip install -r baselines.txt</td>
</tr>
</tbody>
</table>

2.2 **MinerU-HTML运行推理**

我们不仅提供 MinerU-HTML 的批量推理，也提供如 magic-html、readability、trafilature 等不同的网页提取方法进行批量推理。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
<br />
BENCHMARK_DATA=benchmark/WebMainBench_100.jsonl<br />
RESULT_DIR=benchmark_results<br />
mkdir $RESULT_DIR<br />
<br />
# For MinerU-HTML<br />
EXTRACTORS=(<br />
"mineru_html_fallback-html-md"<br />
)<br />
MODEL_PATH=YOUR_MINERUHTML_MODEL_PATH<br />
<br />
for extractor in ${EXTRACTORS[@]}; do<br />
python eval_baselines.py --bench $BENCHMARK_DATA --task_dir $RESULT_DIR/$extractor --extractor_name $extractor --model_path $MODEL_PATH --default_config gpu<br />
done</td>
</tr>
</tbody>
</table>

其它网页提取方法：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
<br />
# For CPU Extractors<br />
EXTRACTORS=(<br />
"magichtml-html-md"<br />
"readability-html-md"<br />
"trafilatura-html-md"<br />
"resiliparse-text"<br />
"trafilatura-md"<br />
"trafilatura-text"<br />
"fullpage-html-md"<br />
"boilerpy3-text"<br />
"gne-html-md"<br />
"newsplease-text"<br />
"justtext-text"<br />
"boilerpy3-html-md"<br />
"goose3-text"<br />
)<br />
<br />
for extractor in ${EXTRACTORS[@]}; do<br />
python eval_baselines.py --bench $BENCHMARK_DATA --task_dir $RESULT_DIR/$extractor --extractor_name $extractor<br />
done<br />
<br />
# For ReaderLM<br />
extractor=readerlm-text<br />
MODEL_PATH=YOUR_READERLM_MODEL_PATH<br />
<br />
python eval_baselines.py --bench $BENCHMARK_DATA --task_dir $RESULT_DIR/$extractor --extractor_name $extractor --model_path $MODEL_PATH --default_config gpu</td>
</tr>
</tbody>
</table>

**四、常见问题**

如何下载huggingface模型并离线使用？

下载模型：可访问 Hugging Face 页面 [<u>MinerU-HTML-v1.1-compact</u>](https://huggingface.co/opendatalab/MinerU-HTML-v1.1-hunyuan0.5B-compact) 下载模型并传输到指定地址，或者有网环境下使用hf下载

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
pip install -U huggingface_hub<br />
hf download opendatalab/MinerU-HTML-v1.1-hunyuan0.5B-compact</td>
</tr>
</tbody>
</table>

在离线环境中加载：模型地址改为文件夹的绝对路径。为了防止代码在运行时尝试联网检查更新导致卡顿或报错，建议在执行脚本前设置环境变量：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
export TRANSFORMERS_OFFLINE=1<br />
export HF_DATASETS_OFFLINE=1</td>
</tr>
</tbody>
</table>
