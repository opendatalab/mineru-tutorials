**04课：MinerU 1.2B 模型微调入门教程-沐曦版**

<img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image1.png" style="width:5.75in;height:4.3125in" />

**一、教程背景**

随着多模态大模型的快速发展，文档解析（Document
Parsing）正逐渐成为连接非结构化文档与大模型知识处理流程的重要环节。MinerU
是一款面向复杂文档理解的解析工具，支持将 PDF、图片、DOCX、PPTX、XLSX
等输入转换为 Markdown、JSON
等机器可读格式，用于后续检索、抽取和处理。它不仅面向常见的 PDF
解析场景，也覆盖图片级页面理解、Office 文档解析等多种输入形式。

在实际应用中，MinerU
所依赖的视觉语言模型（VLM）虽然已经具备较强的通用文档理解能力，但在特定垂直场景下仍可能存在进一步优化空间，例如复杂行业表格、特殊公式排版、领域化版式结构等。为了提升模型在特定任务中的识别精度，本教程将基于
MS-SWIFT 框架，通过监督微调（SFT）流程，对 MinerU2.5
模型（MinerU2.5-2509-1.2B）进行针对性微调。

|                                                                                                                                                                                                                                                                            |
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 本教程的目标是帮助读者在沐曦 GPU 环境中完成 MinerU2.5 最小化微调链路验证，包括环境安装、数据准备、训练启动、推理验证。教程中的最小样本训练主要用于流程打通，不以获得高质量最终效果为目标。若希望进一步提升模型表现，请在此基础上切换到正式数据集并调整训练步数与参数配置。 |

**二、项目目标**

在沐曦 GPU 环境安装 ms-swift 及相关依赖

构造最小样本并执行一次 LoRA 冒烟训练

完成 MinerU2.5 最小化微调链路验证，包括训练、保存与推理加载

在流程跑通后，切换到完整训练集启动正式训练

<img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image2.png" style="width:5.75in;height:4.3125in" />

**选读内容：**

|                                                                                                                                                                                                                                                                                                                                                                                          |
|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **MS-SWIFT大模型微调框架**：该框架全称Scalable lightWeight Infrastructure for Fine-Tuning，是由**阿里魔搭**社区（ModelScope）开源的轻量化大模型微调框架，主打低显存、易使用、高兼容，支持 LoRA/QLoRA 等高效微调算法，可在单张消费级显卡上快速微调 Qwen、Llama 等主流大模型，同时覆盖 SFT、DPO/GRPO 对齐训练与多模态任务，提供命令行与 WebUI 两种方式，一站式完成训练、推理、量化与部署。 |

**MinerU SFT微调代码**

MinerU SFT代码基于 \[ModelScope
SWIFT\](https://github.com/modelscope/ms-swift)
框架构建，主要的文件结构包括：

<table>
<tbody>
<tr class="odd">
<td>Plain Text<br />
文件树<br />
mineru_sft_code/<br />
├── ms_swift_core/ # [Submodule] 官方 ms-swift 源码<br />
├── mineru_ext/ # MinerU 额外的训练逻辑<br />
│ ├── cli # 重写的训练脚本，包含sft<br />
│ └── templates/ # 自定义 Prompt 模板等等<br />
├── config/ # 配置文件 (数据集路径等)<br />
├── run_mineru.py # [入口] 自动注入 submodule 路径的启动器<br />
└── requirements.txt</td>
</tr>
</tbody>
</table>

**\[mineru\_sft\_code.zip\]**

**三、快速上手**

1\. **安装和准备**

1.1 **配置镜像**

在常见的沐曦 GPU 云端环境中，建议优先选择沐曦-vllm
镜像。和鲸平台已经为大家配置好环境资源，登陆[MinerU数据智能与前沿语料挑战赛平台](https://www.heywhale.com/org/sh_AI%20Lab/workspace/competition/area)
获取算力（需要报名过训练营才能自动获得平台算力）。

1）新建项目

选择新建Notebook或上传Notebook。

<img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image3.png" style="width:5.75in;height:4.88542in" />

参考：

**\[MinerU2.5-1.2B微调入门-沐曦版.ipynb\]**

2）启动服务器

点击右上角运行启动服务器，勾选“挂载个人磁盘”，计算资源选择Miner大赛沐曦资源，镜像选择沐曦资源vllm-metax-0.15.0-maca3.5.3.203-torch2.8

参考图：

|                                                                                                                                                                                                   |                                                                                                                                                                                                             |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image4.png" style="width:3in;height:2.5in" /> | <img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image5.png" style="width:2.40625in;height:2.46875in" /> |

1.2 **环境自查**

安装前，建议先确认系统、Python、沐曦 GPU 与网络环境均正常。

1）调用 mx-smi 查看沐曦 MetaX 设备状态（类似 NVIDIA 的
nvidia-smi），确认驱动与运行时正常。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
!mx-smi | head -n 80</td>
</tr>
</tbody>
</table>

2）检查 GPU 是否对当前内核可见（torch.cuda.is\_available() / 设备数量 /
设备名）。如果这里是 False，后面的推理会退回 CPU 或失败。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import torch<br />
<br />
print('torch:', torch.__version__)<br />
print('cuda:', torch.cuda.is_available())<br />
print('device_count:', torch.cuda.device_count())<br />
if torch.cuda.is_available():<br />
print('device0:', torch.cuda.get_device_name(0))</td>
</tr>
</tbody>
</table>

1.3 **安装ms-swift和相关依赖库**

建议在沐曦环境安装并固定如下库：

numpy&lt;2：避免部分 PyTorch 组合出现 Numpy is not available 等问题

opencv-python==4.11.0.86：避免新版本强制依赖 numpy&gt;=2

ms-swift：提供 swift 命令行（训练 / 推理）

modelscope：下载 OpenDataLab/MinerU2.5-2509-1.2B

qwen\_vl\_utils：Qwen2-VL 多模态数据处理

接下来使用阿里云 PyPI 镜像升级 pip
并安装上述依赖；安装过程输出可能较长，属正常现象。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
!pip install -U pip -i https://mirrors.aliyun.com/pypi/simple<br />
!pip install -U "numpy&lt;2" "opencv-python==4.11.0.86" -i https://mirrors.aliyun.com/pypi/simple<br />
!pip install -U modelscope ms-swift pillow decord "qwen_vl_utils&gt;=0.0.6" -i https://mirrors.aliyun.com/pypi/simple</td>
</tr>
</tbody>
</table>

1.4 **准备项目目录**

建议把 **ModelScope 缓存、训练数据、训练输出**
放在同一根目录下，方便备份与排查。

**和鲸平台目录习惯**：

**work**（**/home/mw/work/**）：个人工作区，文件可持久化、可跨项目读写。

**project**：空间通常更小；大文件或需长期保留的建议放 **work**。

**参考代码**（创建本教程用到的子目录）：

<table>
<tbody>
<tr class="odd">
<td>Bash<br />
!mkdir -p /home/mw/work/mineru_muxi/sft/modelscope_cache<br />
!mkdir -p /home/mw/work/mineru_muxi/sft/data<br />
!mkdir -p /home/mw/work/mineru_muxi/sft/outputs<br />
!mkdir -p /home/mw/work/mineru_muxi/sft/mineru_sft_code/configs/sft</td>
</tr>
</tbody>
</table>

执行完成后，在 **/home/mw/work/mineru\_muxi/sft** 下应能看到
**modelscope\_cache**、**data**、**outputs**、**mineru\_sft\_code**
等文件夹。

如果没有看到work目录，检查右上角运行启动服务器处，“挂载个人磁盘”是否有勾选。

1.5 **配置目录和模型源**

把根目录 /home/mw/work/mineru\_muxi/sft 记到变量里，并设置
MODELSCOPE\_CACHE，后面下载模型会用到。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import os<br />
from pathlib import Path<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
MODELSCOPE_CACHE = BASE / "modelscope_cache"<br />
os.environ["MODELSCOPE_CACHE"] = str(MODELSCOPE_CACHE)<br />
<br />
print("BASE =", BASE)<br />
print("MODELSCOPE_CACHE =", MODELSCOPE_CACHE)</td>
</tr>
</tbody>
</table>

1.6 **下载基座模型（使用ModelScope）**

1）从 ModelScope 拉取 **OpenDataLab/MinerU2.5-2509-1.2B** 到本地缓存目录

2）把得到的 model\_dir **绝对路径** 写入 model\_dir.txt，后面 YAML
里直接引用该文件，避免手写长路径出错

<table>
<tbody>
<tr class="odd">
<td>Python<br />
from pathlib import Path<br />
from modelscope import snapshot_download<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
MODELSCOPE_CACHE = BASE / "modelscope_cache"<br />
<br />
model_dir = snapshot_download("OpenDataLab/MinerU2.5-2509-1.2B", cache_dir=str(MODELSCOPE_CACHE))<br />
(BASE / "model_dir.txt").write_text(model_dir, encoding="utf-8")<br />
print("model_dir =", model_dir)</td>
</tr>
</tbody>
</table>

1.7 **准备最小训练数据（JSONL）**

MinerU2.5这类多模态 VLM
微调，本质上是在学习：给定一张（或多张）图片和文字说明，模型应输出什么结果。因此，训练数据通常需要满足以下几点：

有图：例如文档页截图、表格图片、扫描件等，且路径应能被程序正确读取，通常建议使用绝对路径。

有任务描述：常见做法是使用对话格式，在用户侧说明任务目标（例如表格识别、版面理解），在助手侧给出期望输出（例如
HTML 表格、结构化文本等）。

格式符合训练框架要求：在 ms-swift 中，常用的数据组织方式是 messages +
images，即每行一条 JSONL 样本。

下面我们先构造一个最小可运行样例：自动生成一张简单表格图片
demo\_table.png，再在 train.jsonl 和 val.jsonl
中各写入一条对应样本。这样做的目的，是先用最少数据验证整条训练链路是否打通——包括图片能否读取、数据格式能否识别、训练能否启动以及结果能否正常保存。  
需要说明的是，这里生成的图片仅用于流程验证，并不代表真实训练数据；正式训练时，应替换为真实的文档页截图、表格图片或标准数据集样本。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import json<br />
from pathlib import Path<br />
from PIL import Image, ImageDraw<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
img = BASE / "data" / "demo_table.png"<br />
<br />
# 生成一张简单的 2 列 3 行表格图片<br />
W, H = 640, 320<br />
image = Image.new("RGB", (W, H), "white")<br />
draw = ImageDraw.Draw(image)<br />
<br />
# 表格区域<br />
left, top = 60, 60<br />
right, bottom = 580, 260<br />
cols, rows = 2, 3<br />
<br />
# 画外框<br />
draw.rectangle([left, top, right, bottom], outline="black", width=2)<br />
<br />
# 画列线<br />
col_width = (right - left) // cols<br />
for i in range(1, cols):<br />
x = left + i * col_width<br />
draw.line([x, top, x, bottom], fill="black", width=2)<br />
<br />
# 画行线<br />
row_height = (bottom - top) // rows<br />
for i in range(1, rows):<br />
y = top + i * row_height<br />
draw.line([left, y, right, y], fill="black", width=2)<br />
<br />
# 填入简单文本<br />
draw.text((95, 85), "Col1", fill="black")<br />
draw.text((355, 85), "Col2", fill="black")<br />
draw.text((95, 145), "A", fill="black")<br />
draw.text((355, 145), "B", fill="black")<br />
draw.text((95, 205), "C", fill="black")<br />
draw.text((355, 205), "D", fill="black")<br />
<br />
image.save(img)<br />
<br />
row = {<br />
"messages": [<br />
{"role": "user", "content": "&lt;image&gt;\nTable Recognition:"},<br />
{<br />
"role": "assistant",<br />
"content": "&lt;table&gt;&lt;tr&gt;&lt;td&gt;Col1&lt;/td&gt;&lt;td&gt;Col2&lt;/td&gt;&lt;/tr&gt;&lt;tr&gt;&lt;td&gt;A&lt;/td&gt;&lt;td&gt;B&lt;/td&gt;&lt;/tr&gt;&lt;tr&gt;&lt;td&gt;C&lt;/td&gt;&lt;td&gt;D&lt;/td&gt;&lt;/tr&gt;&lt;/table&gt;"<br />
},<br />
],<br />
"images": [str(img.resolve())],<br />
}<br />
<br />
train = BASE / "data" / "train.jsonl"<br />
val = BASE / "data" / "val.jsonl"<br />
train.write_text(json.dumps(row, ensure_ascii=False) + "\n", encoding="utf-8")<br />
val.write_text(train.read_text(encoding="utf-8"), encoding="utf-8")<br />
<br />
print("demo image =", img)<br />
print("train =", train)<br />
print("val =", val)</td>
</tr>
</tbody>
</table>

构造的图片：

<img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image6.png" style="width:5.75in;height:2.875in" />

**拓展-标准数据集：PubTabNet**

也可以使用标准数据集：**PubTabNet**数据集，相关代码如下：

<table>
<tbody>
<tr class="odd">
<td>Bash<br />
# 下载PubTabNet数据集<br />
wget https://huggingface.co/datasets/ajimeno/PubTabNet/resolve/main/pubtabnet.tar.gz<br />
<br />
# 解压并转换格式<br />
tar -xzf pubtabnet.tar.gz<br />
python dataset/pubtabnet/convert_pubtabnet2mineru.py</td>
</tr>
</tbody>
</table>

详细说明请参考仓库内的dataset/pubtabnet/prepare\_train\_data.md文件。

<img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image7.png" style="width:5.75in;height:3.07292in" />

1.8 **基座推理冒烟（可选但推荐）**

在正式开始微调前，建议先让基座模型单独跑一次推理。  
这一步的目的不是看效果有多好，而是提前确认下面几件事都没有问题：

基座模型已经成功下载，路径可读

当前环境可以正常执行 swift infer

输入数据格式没有问题

图片路径能够被程序正确读取

推理结果可以正常写出到文件

如果这一步能顺利跑通，后面即使训练阶段报错，也能更容易判断问题是出在“训练配置”还是“模型/数据/环境”本身。

**1）准备一条最小推理样本**

这条样本和前面的训练样本格式类似，仍然包含两部分：

messages：用户输入的文本

images：图片路径

由于前面已经生成了一张简单表格图，这里可以直接让模型识别其中的表格内容。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import json<br />
from pathlib import Path<br />
<br />
BASE = Path("/home/mw/work/mineru_muxi/sft").resolve()<br />
img = BASE / "data" / "demo_table.png"<br />
infer_file = BASE / "data" / "infer_smoke.jsonl"<br />
<br />
sample = {<br />
"messages": [<br />
{"role": "user", "content": "&lt;image&gt;\n请识别这张图片中的表格内容。"}<br />
],<br />
"images": [str(img.resolve())],<br />
}<br />
<br />
infer_file.write_text(<br />
json.dumps(sample, ensure_ascii=False) + "\n",<br />
encoding="utf-8"<br />
)<br />
<br />
print("已写入推理样本：", infer_file)</td>
</tr>
</tbody>
</table>

**2）调用基座模型执行推理**

下面这段代码会完成几件事：

读取前面保存的基座模型路径

指定输入文件为 infer\_smoke.jsonl

指定输出文件为 outputs/infer\_smoke.jsonl

使用 swift infer 执行一次推理

打印结果文件的前一部分内容，方便快速检查

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import os<br />
import subprocess<br />
from pathlib import Path<br />
<br />
BASE = Path("/home/mw/work/mineru_muxi/sft").resolve()<br />
model_dir = (BASE / "model_dir.txt").read_text(encoding="utf-8").strip()<br />
infer_file = BASE / "data" / "infer_smoke.jsonl"<br />
result_file = BASE / "outputs" / "infer_smoke.jsonl"<br />
<br />
result_file.parent.mkdir(parents=True, exist_ok=True)<br />
<br />
env = os.environ.copy()<br />
env["CUDA_VISIBLE_DEVICES"] = "0"<br />
env.setdefault("MAX_PIXELS", "50176")<br />
<br />
cmd = [<br />
"swift",<br />
"infer",<br />
"--model", model_dir,<br />
"--model_type", "qwen2_vl",<br />
"--template", "qwen2_vl",<br />
"--val_dataset", str(infer_file),<br />
"--infer_backend", "transformers",<br />
"--max_new_tokens", "64",<br />
"--torch_dtype", "bfloat16",<br />
"--attn_impl", "sdpa",<br />
"--result_path", str(result_file),<br />
]<br />
<br />
print("开始执行基座推理冒烟测试...")<br />
subprocess.check_call(cmd, env=env)<br />
<br />
print("推理完成，结果文件：", result_file)<br />
print(result_file.read_text(encoding="utf-8")[:800])</td>
</tr>
</tbody>
</table>

<img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image8.png" style="width:5.75in;height:3.44792in" />

运行正常，你会看到：

命令执行结束，没有报错

outputs/infer\_smoke.jsonl 被成功生成

文件中能看到模型输出内容

这就说明：基座模型、推理命令、数据格式、图片读取、结果写出这条链路已经打通。后面进入微调阶段时，排查范围就会小很多。

2\. **启动训练**

2.1 **参数配置：写入YAML 与 run\_[mineru.py](http://mineru.py)**

接下来做的是
「训练参数说明书」：把「用哪套模型、数据在哪、结果存哪、算多久、怎么省显存」写进一个
YAML 文件，训练框架（这里是
ms-swift）会按这个文件启动一次监督微调（SFT）。

同时放一个 run\_mineru.py，是为了和常见资料里的命令形式一致：python
run\_mineru.py sft --config ...。

官方完整仓库里，run\_mineru.py 可能还会包一层 mineru\_ext
里的逻辑；本教程用最小脚本，不依赖那套扩展，只保证「命令长得像资料」且底层仍是
swift sft。

关键参数：

model：基座模型所在目录，也就是前面下载好的 MinerU2.5
权重路径。训练时程序会从这里加载原始模型。

dataset / val\_dataset：分别表示训练集和验证集文件路径。

output\_dir：训练输出目录。训练过程中产生的日志、参数快照、checkpoint
等文件，都会保存在这里。建议使用单独目录，方便后续查看和排查。

model\_type /
template：当前加载的是什么模型，以及它在训练时应当使用哪套输入模板。本教程使用的是
MinerU2.5 对应的 Qwen2-VL 架构，所以这里使用 qwen2\_vl。

tuner\_type / lora\_rank：微调方式。本教程使用的是
LoRA，也就是只训练少量附加参数，而不是更新整个模型。这样做的好处是更省显存、训练更轻量，也更适合入门测试。其中
lora\_rank 可以理解为 LoRA
的容量大小。数值越大，模型可学习的附加能力通常越强，但显存占用也会更高。

attn\_impl: sdpa：指定用 PyTorch 自带的注意力实现。很多环境装
FlashAttention 不方便，用 sdpa 更稳、兼容性更好。

torch\_dtype: bfloat16：用 BF16
数字格式做混合精度计算，一般能在不太牺牲效果的情况下减轻显存和计算量（若硬件不支持，再考虑改
float16 等，需另做兼容性评估）。

**max\_steps: 1：这里把训练步数设成
1，并不是为了得到效果好的模型，而只是为了验证整条训练链路是否能正常运行，包括：配置文件能否被正确读取、数据是否能正常加载、训练命令能否顺利启动、checkpoint
是否能成功保存。如果后续你希望真正训练出可用模型，就需要把 max\_steps
调大，并同时合理设置 save\_steps、logging\_steps、save\_total\_limit
等参数。**

<table>
<tbody>
<tr class="odd">
<td>Python<br />
from pathlib import Path<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
CODE = BASE / "mineru_sft_code"<br />
model_dir = (BASE / "model_dir.txt").read_text(encoding="utf-8").strip()<br />
train = (BASE / "data" / "train.jsonl").resolve()<br />
val = (BASE / "data" / "val.jsonl").resolve()<br />
out = (BASE / "outputs" / "sft_table_demo").resolve()<br />
<br />
lines = [<br />
"ENV:",<br />
' MAX_PIXELS: "50176"',<br />
"model: " + model_dir,<br />
"dataset:",<br />
" - " + str(train),<br />
"val_dataset:",<br />
" - " + str(val),<br />
"model_type: qwen2_vl",<br />
"template: qwen2_vl",<br />
"tuner_type: lora",<br />
"lora_rank: 8",<br />
"output_dir: " + str(out),<br />
"overwrite_output_dir: true",<br />
"max_steps: 1", # 可以调大<br />
"per_device_train_batch_size: 1",<br />
"gradient_accumulation_steps: 1",<br />
"logging_steps: 1",<br />
"save_steps: 9999",<br />
"attn_impl: sdpa",<br />
"torch_dtype: bfloat16",<br />
]<br />
cfg = CODE / "configs" / "sft" / "sft_table_rec.yaml"<br />
cfg.write_text("\n".join(lines) + "\n", encoding="utf-8")<br />
<br />
(CODE / "run_mineru.py").write_text(<br />
"#!/usr/bin/env python3\n"<br />
"import subprocess\n"<br />
"import sys\n"<br />
"\n"<br />
"def main() -&gt; int:\n"<br />
" a = sys.argv[1:]\n"<br />
' if len(a) &gt;= 3 and a[0] == "sft" and a[1] == "--config":\n'<br />
' return subprocess.call(["swift", "sft", a[2]])\n'<br />
' return subprocess.call(["swift", *a])\n'<br />
"\n"<br />
'if __name__ == "__main__":\n'<br />
" raise SystemExit(main())\n",<br />
encoding="utf-8",<br />
)<br />
print("ok", cfg)</td>
</tr>
</tbody>
</table>

2.2 **启动训练（单卡）**

若日志里出现与 **MPI** 相关的报错，可尝试：**pip install
mpi4py**（部分环境还需系统 MPI，以平台文档为准）。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
多卡代码默认不执行<br />
import os<br />
import subprocess<br />
from pathlib import Path<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
CODE = BASE / "mineru_sft_code"<br />
cfg = CODE / "configs" / "sft" / "sft_table_rec.yaml"<br />
env = os.environ.copy()<br />
env["CUDA_VISIBLE_DEVICES"] = "0"<br />
subprocess.check_call(<br />
["python3", str(CODE / "run_mineru.py"), "sft", "--config", str(cfg)],<br />
cwd=str(CODE),<br />
env=env,<br />
)<br />
print("训练命令已结束（冒烟 max_steps=1）。")<br />
print('多卡示例（在终端执行或取消下行注释）：')<br />
print(f'NPROC_PER_NODE=2 CUDA_VISIBLE_DEVICES=0,1 torchrun --nproc_per_node=2 -m swift.cli.sft {cfg}')<br />
# import os, subprocess<br />
# env = os.environ.copy()<br />
# env['CUDA_VISIBLE_DEVICES'] = '0,1'<br />
# subprocess.check_call(<br />
# ['torchrun', '--nproc_per_node=2', '-m', 'swift.cli.sft', str(cfg)], cwd=str(CODE), env=env</td>
</tr>
</tbody>
</table>

<img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image9.png" style="width:5.75in;height:1.44792in" />

2.3 **查看训练输出目录**

训练结束后，YAML 中设置的 output\_dir 下会生成本次训练的输出结果。  
通常会先看到一层 v\* 目录，每个 v\* 目录代表一次独立的训练运行；进入某个
v\* 目录后，还会看到 checkpoint-\* 目录，这里面保存的就是训练得到的 LoRA
权重。

后面做推理验证时，通常需要把 --adapters 指向某一个 checkpoint-\* 目录。

可以先用下面的代码查看当前有哪些训练输出：

<table>
<tbody>
<tr class="odd">
<td>Python<br />
from pathlib import Path<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
out = BASE / "outputs" / "sft_table_demo"<br />
<br />
print("训练输出目录：", out)<br />
print("当前已有的运行记录：")<br />
for p in sorted(out.glob("v*")):<br />
print(" ", p)</td>
</tr>
</tbody>
</table>

如果你想继续查看某一次训练里面有哪些 checkpoint，也可以再执行：

<table>
<tbody>
<tr class="odd">
<td>Python<br />
from pathlib import Path<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
<br />
# 把这里改成你想查看的那次训练目录<br />
run_dir = BASE / "outputs" / "sft_table_demo" / "v0-2026xxxx-xxxxxx"<br />
<br />
print("当前运行目录：", run_dir)<br />
print("其中包含的 checkpoint：")<br />
for p in sorted(run_dir.glob("checkpoint-*")):<br />
print(" ", p)</td>
</tr>
</tbody>
</table>

如：

<img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image10.png" style="width:5.75in;height:2.08333in" />

3\. **推理验证**

3.1 **加载指定的 LoRA 权重进行推理验证**

训练完成后，可以指定某一个已经生成好的 LoRA
权重目录，将其加载到基座模型上执行推理验证。

这一步的主要目的，是确认以下链路已经打通：

训练得到的 LoRA 权重可以被正常加载

swift infer 能够正确执行推理

输入数据格式没有问题

推理结果可以正常写出到文件

请先把下面代码里的 adapter\_dir 改成你自己的实际 checkpoint-\* 路径。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import os<br />
import subprocess<br />
from pathlib import Path<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
<br />
# 读取基座模型路径<br />
model_dir = (BASE / "model_dir.txt").read_text(encoding="utf-8").strip()<br />
<br />
# 指定要测试的 LoRA 权重目录<br />
# 请改成你自己的实际 checkpoint 路径<br />
adapter_dir = BASE / "outputs" / "sft_table_demo" / "v0-20260418-xxxxxx" / "checkpoint-1"<br />
<br />
# 验证集与结果输出路径<br />
val_file = BASE / "data" / "val.jsonl"<br />
result_file = BASE / "outputs" / "infer_result.jsonl"<br />
result_file.parent.mkdir(parents=True, exist_ok=True)<br />
<br />
env = os.environ.copy()<br />
env["CUDA_VISIBLE_DEVICES"] = "0"<br />
env.setdefault("MAX_PIXELS", "50176")<br />
<br />
cmd = [<br />
"swift",<br />
"infer",<br />
"--model", model_dir,<br />
"--model_type", "qwen2_vl",<br />
"--template", "qwen2_vl",<br />
"--adapters", str(adapter_dir),<br />
"--val_dataset", str(val_file),<br />
"--infer_backend", "transformers",<br />
"--max_new_tokens", "64",<br />
"--torch_dtype", "bfloat16",<br />
"--attn_impl", "sdpa",<br />
"--result_path", str(result_file),<br />
]<br />
<br />
print("开始推理...")<br />
print("基座模型路径：", model_dir)<br />
print("LoRA 权重路径：", adapter_dir)<br />
print("输入数据：", val_file)<br />
print("结果文件：", result_file)<br />
<br />
subprocess.check_call(cmd, env=env)<br />
<br />
print("推理完成。")<br />
print(result_file.read_text(encoding="utf-8")[:1200])</td>
</tr>
</tbody>
</table>

3.2 **拓展-使用一张新图片做推理验证（可选）**

前一步主要是在验证训练产物能否正常加载和执行。  
如果希望进一步检查微调后的模型在**新输入**上的表现，可以再准备一张新的测试图片，单独做一次推理，并将结果额外导出为
Markdown 文件，方便直接查看。

如果你手头已经有一张新的测试图片，可以把它上传到 data/
目录下，例如：data/test.jpg（可以下载下图）

<img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image11.png" style="width:5.75in;height:0.9375in" />

上传方式：

<img src="04-扩展：MinerU 1.2B 模型微调入门教程-沐曦版 - 文档 - media/media/image12.png" style="width:5.75in;height:1.875in" />

说明：由于当前仅进行了最小样本的冒烟训练，模型对复杂版式、低清晰度图片或分布差异较大的新图片泛化能力有限。为便于验证
LoRA
权重是否能够正常加载并输出结果，建议优先使用背景干净、文字清晰、结构简单的表格图片进行测试。

**1）写入单条推理样本**

将测试图片与推理指令组织成一条 JSONL 格式的输入样本，供 swift infer
调用。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import json<br />
from pathlib import Path<br />
<br />
BASE = Path("/home/mw/work/mineru_muxi/sft").resolve()<br />
<br />
# 指定测试图片<br />
test_img = BASE / "data" / "test.jpg"<br />
<br />
# 写入单条推理样本<br />
infer_file = BASE / "data" / "infer_test.jsonl"<br />
sample = {<br />
"messages": [<br />
{<br />
"role": "user",<br />
"content": "&lt;image&gt;\n请识别这张图片中的表格内容。"<br />
}<br />
],<br />
"images": [str(test_img.resolve())],<br />
}<br />
<br />
infer_file.write_text(<br />
json.dumps(sample, ensure_ascii=False) + "\n",<br />
encoding="utf-8"<br />
)<br />
<br />
print("已写入推理样本：", infer_file)</td>
</tr>
</tbody>
</table>

**2）执行推理**

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import os<br />
import subprocess<br />
from pathlib import Path<br />
<br />
BASE = Path("/home/mw/work/mineru_muxi/sft").resolve()<br />
<br />
# 读取基座模型路径<br />
model_dir = (BASE / "model_dir.txt").read_text(encoding="utf-8").strip()<br />
<br />
# 改成你自己的真实 checkpoint 路径<br />
adapter_dir = BASE / "outputs" / "sft_table_demo" / "v0-20260xxx-xxxxxx" / "checkpoint-1"<br />
if not adapter_dir.exists():<br />
raise FileNotFoundError(f"LoRA 权重目录不存在: {adapter_dir}")<br />
<br />
# 推理输入与结果输出<br />
infer_file = BASE / "data" / "infer_test.jsonl"<br />
result_file = BASE / "outputs" / "infer_test_result.jsonl"<br />
result_file.parent.mkdir(parents=True, exist_ok=True)<br />
<br />
env = os.environ.copy()<br />
env["CUDA_VISIBLE_DEVICES"] = "0"<br />
<br />
# 若推理结果过短、缺字或接近空输出，可以调整为更大的值<br />
env["MAX_PIXELS"] = "262144"<br />
<br />
cmd = [<br />
"swift",<br />
"infer",<br />
"--model", model_dir,<br />
"--model_type", "qwen2_vl",<br />
"--template", "qwen2_vl",<br />
"--adapters", str(adapter_dir),<br />
"--val_dataset", str(infer_file),<br />
"--infer_backend", "transformers",<br />
"--max_new_tokens", "256",<br />
"--torch_dtype", "bfloat16",<br />
"--attn_impl", "sdpa",<br />
"--result_path", str(result_file),<br />
]<br />
<br />
print("开始推理...")<br />
print("基座模型路径：", model_dir)<br />
print("LoRA 权重路径：", adapter_dir)<br />
print("输入文件：", infer_file)<br />
print("结果文件：", result_file)<br />
print("MAX_PIXELS：", env["MAX_PIXELS"])<br />
<br />
subprocess.check_call(cmd, env=env)<br />
<br />
print("推理完成。")<br />
print(result_file.read_text(encoding="utf-8")[:2000])</td>
</tr>
</tbody>
</table>

**3）将推理结果导出为 Markdown 文件**

swift infer 默认输出的是 JSONL
文件。如果希望像普通文档一样直接查看结果，可以再从 JSONL
中读取模型输出，并额外保存为一个 .md 文件。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import json<br />
from pathlib import Path<br />
<br />
BASE = Path("/home/mw/work/mineru_muxi/sft").resolve()<br />
result_file = BASE / "outputs" / "infer_test_result.jsonl"<br />
md_file = BASE / "outputs" / "infer_test_result.md"<br />
<br />
lines = result_file.read_text(encoding="utf-8").strip().splitlines()<br />
if not lines:<br />
raise ValueError(f"结果文件为空: {result_file}")<br />
<br />
item = json.loads(lines[0])<br />
<br />
# 根据实际输出结果，尝试寻找模型回答字段<br />
markdown_text = (<br />
item.get("response")<br />
or item.get("predict")<br />
or item.get("prediction")<br />
or item.get("output")<br />
or item.get("generated_text")<br />
)<br />
<br />
if not markdown_text:<br />
print("未直接找到常见输出字段，当前结果包含的字段有：")<br />
print(list(item.keys()))<br />
raise KeyError("请根据实际输出字段名，手动修改 markdown_text 的读取逻辑。")<br />
<br />
md_file.write_text(markdown_text, encoding="utf-8")<br />
<br />
print("Markdown 文件已生成：", md_file)<br />
print(md_file.read_text(encoding="utf-8")[:1200])</td>
</tr>
</tbody>
</table>

运行正常，你会看到：

outputs/infer\_test\_result.jsonl 被成功生成

outputs/infer\_test\_result.md 被成功生成

Markdown 文件中能够看到模型对新图片内容的输出结果

**四、进阶1-训练增强版数据（选做）**

前面的步骤主要是用最小样本完成一次冒烟验证，确认环境、模型、数据格式、训练与推理链路都能够正常工作。

下面我们可以尝试基于同一张简单表格图，构造 4
条不同任务的样本，分别对应：

表格 HTML 输出

表格 Markdown 输出

纯文本提取

简单版面描述

这样做的好处是：虽然数据量仍然很小，但模型能接触到不止一种输出形式，后面推理验证时会更容易看到变化。

需要说明的是，这里生成的图片和样本仅用于教学演示与流程验证，并不代表真实训练数据；正式训练时，仍建议替换为真实文档页截图、表格图片或标准数据集样本。

1\. **构造增强版训练数据**

为了和前面的最小样本版本区分开，这里单独写入一组新的训练文件：

train\_v2.jsonl

val\_v2.jsonl

对应图片也单独命名为：

demo\_table\_v2.png

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import json<br />
from pathlib import Path<br />
from PIL import Image, ImageDraw<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
img = BASE / "data" / "demo_table_v2.png"<br />
<br />
# 生成一张简单的 2 列 3 行表格图片<br />
W, H = 640, 320<br />
image = Image.new("RGB", (W, H), "white")<br />
draw = ImageDraw.Draw(image)<br />
<br />
left, top = 60, 60<br />
right, bottom = 580, 260<br />
cols, rows = 2, 3<br />
<br />
# 画外框<br />
draw.rectangle([left, top, right, bottom], outline="black", width=2)<br />
<br />
# 画列线<br />
col_width = (right - left) // cols<br />
for i in range(1, cols):<br />
x = left + i * col_width<br />
draw.line([x, top, x, bottom], fill="black", width=2)<br />
<br />
# 画行线<br />
row_height = (bottom - top) // rows<br />
for i in range(1, rows):<br />
y = top + i * row_height<br />
draw.line([left, y, right, y], fill="black", width=2)<br />
<br />
# 填入简单文本<br />
draw.text((95, 85), "Col1", fill="black")<br />
draw.text((355, 85), "Col2", fill="black")<br />
draw.text((95, 145), "A", fill="black")<br />
draw.text((355, 145), "B", fill="black")<br />
draw.text((95, 205), "C", fill="black")<br />
draw.text((355, 205), "D", fill="black")<br />
<br />
image.save(img)<br />
<br />
rows = [<br />
{<br />
"messages": [<br />
{"role": "user", "content": "&lt;image&gt;\n请识别这张图片中的表格内容，并输出为 HTML。"},<br />
{<br />
"role": "assistant",<br />
"content": "&lt;table&gt;&lt;tr&gt;&lt;td&gt;Col1&lt;/td&gt;&lt;td&gt;Col2&lt;/td&gt;&lt;/tr&gt;&lt;tr&gt;&lt;td&gt;A&lt;/td&gt;&lt;td&gt;B&lt;/td&gt;&lt;/tr&gt;&lt;tr&gt;&lt;td&gt;C&lt;/td&gt;&lt;td&gt;D&lt;/td&gt;&lt;/tr&gt;&lt;/table&gt;"<br />
},<br />
],<br />
"images": [str(img.resolve())],<br />
},<br />
{<br />
"messages": [<br />
{"role": "user", "content": "&lt;image&gt;\n请将图片中的表格识别后输出为 Markdown 表格。"},<br />
{<br />
"role": "assistant",<br />
"content": "| Col1 | Col2 |\n|---|---|\n| A | B |\n| C | D |"<br />
},<br />
],<br />
"images": [str(img.resolve())],<br />
},<br />
{<br />
"messages": [<br />
{"role": "user", "content": "&lt;image&gt;\n请提取图片中的全部文字内容。"},<br />
{<br />
"role": "assistant",<br />
"content": "Col1 Col2\nA B\nC D"<br />
},<br />
],<br />
"images": [str(img.resolve())],<br />
},<br />
{<br />
"messages": [<br />
{"role": "user", "content": "&lt;image&gt;\n请描述这张图片的版面结构。"},<br />
{<br />
"role": "assistant",<br />
"content": "这是一张 2 列 3 行的简单表格。第一行为表头 Col1、Col2；第二行为 A、B；第三行为 C、D。"<br />
},<br />
],<br />
"images": [str(img.resolve())],<br />
},<br />
]<br />
<br />
train_v2 = BASE / "data" / "train_v2.jsonl"<br />
val_v2 = BASE / "data" / "val_v2.jsonl"<br />
<br />
train_v2.write_text(<br />
"\n".join(json.dumps(x, ensure_ascii=False) for x in rows) + "\n",<br />
encoding="utf-8"<br />
)<br />
<br />
# 验证集先保留 1 条，便于快速检查<br />
val_v2.write_text(<br />
json.dumps(rows[0], ensure_ascii=False) + "\n",<br />
encoding="utf-8"<br />
)<br />
<br />
print("demo image =", img)<br />
print("train_v2 =", train_v2)<br />
print("val_v2 =", val_v2)<br />
print("训练样本数 =", len(rows))</td>
</tr>
</tbody>
</table>

2\. **写入增强版训练配置**

这一节也建议与前面的冒烟配置分开，单独写入新的配置文件和新的输出目录，例如：

配置文件：sft\_table\_rec\_v2.yaml

输出目录：outputs/sft\_table\_demo\_v2

这样前后的结果不会混在一起。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
from pathlib import Path<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
CODE = BASE / "mineru_sft_code"<br />
model_dir = (BASE / "model_dir.txt").read_text(encoding="utf-8").strip()<br />
<br />
train_v2 = (BASE / "data" / "train_v2.jsonl").resolve()<br />
val_v2 = (BASE / "data" / "val_v2.jsonl").resolve()<br />
out_v2 = (BASE / "outputs" / "sft_table_demo_v2").resolve()<br />
<br />
lines = [<br />
"ENV:",<br />
' MAX_PIXELS: "100352"',<br />
"model: " + model_dir,<br />
"dataset:",<br />
" - " + str(train_v2),<br />
"val_dataset:",<br />
" - " + str(val_v2),<br />
"model_type: qwen2_vl",<br />
"template: qwen2_vl",<br />
"tuner_type: lora",<br />
"lora_rank: 8",<br />
"output_dir: " + str(out_v2),<br />
"overwrite_output_dir: true",<br />
"max_steps: 20",<br />
"per_device_train_batch_size: 1",<br />
"gradient_accumulation_steps: 1",<br />
"logging_steps: 1",<br />
"save_steps: 10",<br />
"save_total_limit: 2",<br />
"attn_impl: sdpa",<br />
"torch_dtype: bfloat16",<br />
]<br />
<br />
cfg_v2 = CODE / "configs" / "sft" / "sft_table_rec_v2.yaml"<br />
cfg_v2.write_text("\n".join(lines) + "\n", encoding="utf-8")<br />
<br />
print("增强版配置文件：", cfg_v2)</td>
</tr>
</tbody>
</table>

3\. **启动增强版训练**

这一步和前面的训练方式一致，只是把配置文件改成增强版的
sft\_table\_rec\_v2.yaml。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import os<br />
import subprocess<br />
from pathlib import Path<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
CODE = BASE / "mineru_sft_code"<br />
cfg_v2 = CODE / "configs" / "sft" / "sft_table_rec_v2.yaml"<br />
<br />
env = os.environ.copy()<br />
env["CUDA_VISIBLE_DEVICES"] = "0"<br />
<br />
subprocess.check_call(<br />
["python3", str(CODE / "run_mineru.py"), "sft", "--config", str(cfg_v2)],<br />
cwd=str(CODE),<br />
env=env,<br />
)<br />
<br />
print("增强版训练已结束。")</td>
</tr>
</tbody>
</table>

查看增强版输出目录

<table>
<tbody>
<tr class="odd">
<td>Python<br />
from pathlib import Path<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
out_v2 = BASE / "outputs" / "sft_table_demo_v2"<br />
<br />
print("增强版训练输出目录：", out_v2)<br />
print("当前已有的运行记录：")<br />
for p in sorted(out_v2.glob("v*")):<br />
print(" ", p)</td>
</tr>
</tbody>
</table>

增强版训练结束后，输出目录中通常会比前面的最小样本冒烟训练包含更多内容。这是因为本节将训练步数从
max\_steps=1 提高到了更大的值，并设置了更频繁的保存策略，例如
save\_steps=10、save\_total\_limit=2。因此训练过程中可能会生成多个
checkpoint，而不再像冒烟训练那样通常只产生一次最小保存结果。

一般来说，你会先看到一层 v\* 目录，每个 v\*
目录代表一次独立的增强版训练运行；进入某个运行目录后，还可能看到：

checkpoint-\*：训练过程中保存的中间权重

args.json：本次训练使用的参数快照

logging.jsonl：训练日志

其他与训练状态相关的输出文件

如果你发现增强版输出目录里的文件数量比前面的最小样本训练更多，这通常是正常现象，主要是因为训练步数增加、保存频率更高。

4\. **使用增强版权重做推理验证**

最后，可以加载增强版训练得到的 LoRA
权重，对前面我们准备的测试图片进行推理验证。

1）写入单条推理样本

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import json<br />
from pathlib import Path<br />
<br />
BASE = Path("/home/mw/work/mineru_muxi/sft").resolve()<br />
<br />
# 指定测试图片<br />
test_img = BASE / "data" / "test.jpg"<br />
<br />
# 写入单条推理样本<br />
infer_file_v2 = BASE / "data" / "infer_v2.jsonl"<br />
sample = {<br />
"messages": [<br />
{<br />
"role": "user",<br />
"content": "&lt;image&gt;\n请识别这张图片中的表格内容。"<br />
}<br />
],<br />
"images": [str(test_img.resolve())],<br />
}<br />
<br />
infer_file_v2.write_text(<br />
json.dumps(sample, ensure_ascii=False) + "\n",<br />
encoding="utf-8"<br />
)<br />
<br />
print("已写入推理样本：", infer_file_v2)<br />
print(infer_file_v2.read_text(encoding="utf-8"))</td>
</tr>
</tbody>
</table>

2）执行推理

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import os<br />
import subprocess<br />
from pathlib import Path<br />
<br />
BASE = Path("/home/mw/work/mineru_muxi/sft").resolve()<br />
<br />
# 读取基座模型路径<br />
model_dir = (BASE / "model_dir.txt").read_text(encoding="utf-8").strip()<br />
<br />
# 改成你自己的真实 checkpoint 路径<br />
adapter_dir = BASE / "outputs" / "sft_table_demo_v2" / "v0-2026xxxx-xxxxxx" / "checkpoint-10/20"<br />
if not adapter_dir.exists():<br />
raise FileNotFoundError(f"LoRA 权重目录不存在: {adapter_dir}")<br />
<br />
# 推理输入与结果输出<br />
infer_file_v2 = BASE / "data" / "infer_v2.jsonl"<br />
result_file_v2 = BASE / "outputs" / "infer_result_v2.jsonl"<br />
result_file_v2.parent.mkdir(parents=True, exist_ok=True)<br />
<br />
env = os.environ.copy()<br />
env["CUDA_VISIBLE_DEVICES"] = "0"<br />
<br />
# 若推理结果过短、缺字或接近空输出，可以尝试调大<br />
env["MAX_PIXELS"] = "262144"<br />
<br />
cmd = [<br />
"swift",<br />
"infer",<br />
"--model", model_dir,<br />
"--model_type", "qwen2_vl",<br />
"--template", "qwen2_vl",<br />
"--adapters", str(adapter_dir),<br />
"--val_dataset", str(infer_file_v2),<br />
"--infer_backend", "transformers",<br />
"--max_new_tokens", "256",<br />
"--torch_dtype", "bfloat16",<br />
"--attn_impl", "sdpa",<br />
"--result_path", str(result_file_v2),<br />
]<br />
<br />
print("开始推理...")<br />
print("基座模型路径：", model_dir)<br />
print("LoRA 权重路径：", adapter_dir)<br />
print("输入文件：", infer_file_v2)<br />
print("结果文件：", result_file_v2)<br />
print("MAX_PIXELS：", env["MAX_PIXELS"])<br />
<br />
subprocess.check_call(cmd, env=env)<br />
<br />
print("推理完成。")<br />
print(result_file_v2.read_text(encoding="utf-8")[:2000])</td>
</tr>
</tbody>
</table>

**五、进阶2-启动正式训练（选做）**

当前面部分已经跑通后，可以选择切换到完整数据集，启动正式训练。

下面以标准表格识别数据集 PubTabNet
为例，说明如何准备正式训练数据并开始训练。PubTabNet
是表格识别任务中常用的公开数据集，适合用来验证 MinerU2.5
在表格结构解析任务上的微调流程。

正式训练通常需要完成以下几步：

准备完整训练集与验证集

修改 YAML 中的数据路径与训练参数

启动正式训练

根据输出目录选择合适的 checkpoint 做后续验证或部署

1\. **下载并转换 PubTabNet 数据集**

直接下载 PubTabNet，并将其转换为 MinerU / ms-swift 可用的数据格式。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
# 下载 PubTabNet 数据集<br />
wget https://huggingface.co/datasets/ajimeno/PubTabNet/resolve/main/pubtabnet.tar.gz<br />
<br />
# 解压数据集<br />
tar -xzf pubtabnet.tar.gz<br />
<br />
# 转换为 MinerU 可用格式<br />
python dataset/pubtabnet/convert_pubtabnet2mineru.py</td>
</tr>
</tbody>
</table>

执行完成后，会生成适用于训练的标准数据文件。

详细的数据准备说明，请参考仓库中的：

<table>
<tbody>
<tr class="odd">
<td>Python<br />
dataset/pubtabnet/prepare_train_data.md</td>
</tr>
</tbody>
</table>

上述环节将原始 PubTabNet
数据转换成与前文最小样本相同风格的训练格式，例如包含 messages 和 images
字段的 JSONL 文件，便于后续直接接入 ms-swift 训练流程。

2\. **修改 YAML 配置**

完成数据转换后，需要将 YAML
中原本指向最小样本的训练集、验证集路径，替换为 PubTabNet
转换后得到的正式数据路径；同时将冒烟训练参数改为正式训练参数。

和前面的冒烟训练相比，正式训练至少需要调整以下几项：

dataset：改为 PubTabNet 转换后的训练集路径

val\_dataset：改为 PubTabNet 转换后的验证集路径

output\_dir：建议单独设置正式训练输出目录，避免与冒烟结果混在一起

max\_steps：改为更大的训练步数，而不是 1

save\_steps：设置合理的 checkpoint 保存间隔

logging\_steps：设置日志打印频率

下面给出一个示例。请根据 convert\_pubtabnet2mineru.py
实际生成的数据文件路径，修改其中的 train\_full 和 val\_full。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
from pathlib import Path<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
CODE = BASE / "mineru_sft_code"<br />
model_dir = (BASE / "model_dir.txt").read_text(encoding="utf-8").strip()<br />
<br />
# 请根据转换脚本实际生成的数据文件路径进行修改<br />
train_full = (BASE / "data" / "pubtabnet_train.jsonl").resolve()<br />
val_full = (BASE / "data" / "pubtabnet_val.jsonl").resolve()<br />
out_full = (BASE / "outputs" / "sft_pubtabnet_full").resolve()<br />
<br />
lines = [<br />
"ENV:",<br />
' MAX_PIXELS: "50176"',<br />
"model: " + model_dir,<br />
"dataset:",<br />
" - " + str(train_full),<br />
"val_dataset:",<br />
" - " + str(val_full),<br />
"model_type: qwen2_vl",<br />
"template: qwen2_vl",<br />
"tuner_type: lora",<br />
"lora_rank: 8",<br />
"output_dir: " + str(out_full),<br />
"overwrite_output_dir: true",<br />
"max_steps: 1000",<br />
"per_device_train_batch_size: 1",<br />
"gradient_accumulation_steps: 1",<br />
"logging_steps: 10",<br />
"save_steps: 100",<br />
"attn_impl: sdpa",<br />
"torch_dtype: bfloat16",<br />
]<br />
cfg_full = CODE / "configs" / "sft" / "sft_pubtabnet_full.yaml"<br />
cfg_full.write_text("\n".join(lines) + "\n", encoding="utf-8")<br />
<br />
print("正式训练配置文件：", cfg_full)</td>
</tr>
</tbody>
</table>

3\. **启动正式训练**

确认数据路径与配置参数无误后，即可使用新的 YAML 文件启动正式训练。

<table>
<tbody>
<tr class="odd">
<td>Python<br />
import os<br />
import subprocess<br />
from pathlib import Path<br />
<br />
BASE = Path(r"/home/mw/work/mineru_muxi/sft").resolve()<br />
CODE = BASE / "mineru_sft_code"<br />
cfg_full = CODE / "configs" / "sft" / "sft_pubtabnet_full.yaml"<br />
<br />
env = os.environ.copy()<br />
env["CUDA_VISIBLE_DEVICES"] = "0"<br />
<br />
subprocess.check_call(<br />
["python3", str(CODE / "run_mineru.py"), "sft", "--config", str(cfg_full)],<br />
cwd=str(CODE),<br />
env=env,<br />
)<br />
<br />
print("正式训练已结束。")</td>
</tr>
</tbody>
</table>

**查看正式训练输出**

正式训练结束后，可以到新的输出目录中查看运行结果，例如：outputs/sft\_table\_full/

其中通常包括：

v\* 运行目录

checkpoint-\* 权重目录

args.json 参数快照

logging.jsonl 训练日志

建议优先检查以下内容：

是否成功生成多个 checkpoint

日志中 loss 是否正常下降

输出目录是否与冒烟训练分开，避免覆盖或混淆

4\. **使用正式训练得到的权重继续推理验证**

正式训练完成后，可以沿用第三部分的方法，将 adapter\_dir
改为正式训练输出目录中的某个 checkpoint-\*，继续做推理验证或新图片测试。

如果只是想快速检查训练结果，通常可先选择最后一个保存的
checkpoint；如果后续需要更细致地比较效果，再结合日志和验证集输出，选择更合适的权重版本。

**六、常见问题**

找不到 work 目录？

先检查启动服务器时是否勾选“挂载个人磁盘”。

找不到 checkpoint？

先执行 2.3 查看输出目录，确认 v\* 和 checkpoint-\* 的实际路径，再修改
adapter\_dir。

Markdown 文件没有生成？

先确认 infer\_test\_result.jsonl 是否成功生成；如果 JSONL 有内容但 .md
未生成，通常是输出字段名与示例代码不同，可先打印 list(item.keys())
查看实际字段。

正式训练时找不到 pubtabnet\_train.jsonl？

不同版本的转换脚本输出文件名可能不同，请以 convert\_pubtabnet2mineru.py
实际生成结果为准，修改 YAML 中路径。

推理结果里出现 &lt;fcel&gt; 和 &lt;nl&gt; 是什么？

这通常表示模型输出的是带结构标记的表格中间表示，其中 &lt;fcel&gt;
可理解为单元格分隔，&lt;nl&gt;
表示换行。说明模型已经识别出表格内容和行列结构，只是输出格式还不是最终
Markdown 或 HTML，可通过简单后处理转换为更直观的表格格式。

增强版推理时报 LoRA 权重目录不存在？

请先检查增强版训练输出目录 outputs/sft\_table\_demo\_v2
下的实际运行目录与 checkpoint 名称，再将 adapter\_dir
修改为真实路径。注意 checkpoint 路径应写成 checkpoint-10 或
checkpoint-20 这种单独目录，而不是 checkpoint-10/20。
