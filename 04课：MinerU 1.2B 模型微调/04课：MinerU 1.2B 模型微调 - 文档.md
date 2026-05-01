**04课：MinerU 1.2B 模型微调**

**一、教程背景**

随着多模态大模型的快速发展，文档解析（Document Parsing）已成为连接非结构化数据与大模型知识库的关键环节。MinerU 作为一款高性能的文档解析引擎，其VLM(Visual Language Model)模型凭借着出色的识别解析能力，为广大开发者提供了一种端到端的优质解决方案。

在实际应用中，MinerU VLM 模型可能在特定垂直领域（如复杂的行业表格、特殊的公式排版或特定行业的文档布局）表现稍欠。为了进一步提升模型在特定场景下的识别精度，本教程将引导你基于 MS-SWIFT 框架，通过 SFT (Supervised Fine-tuning) 流程，对 MinerU2.5模型（MinerU2.5-2509-1.2B） 进行针对性的微调。

无论你是想增强表格识别、公式提取,还是版面分析能力，通过以下步骤，你都能构建出更契合业务需求的专属MinerU 文档解析模型。

**二、准备工作**

在正式开始微调前，需要提前下载好下面几个内容：

1\. **微调代码**

MinerU SFT代码基于 \[ModelScope SWIFT\](https://github.com/modelscope/ms-swift) 框架构建，主要的文件结构包括：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
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

**\[mineru_sft_code.zip\]**

|  |
|:---|
| **MS-SWIFT大模型微调框架**：该框架全称Scalable lightWeight Infrastructure for Fine-Tuning，是由**阿里魔搭**社区（ModelScope）开源的轻量化大模型微调框架，主打低显存、易使用、高兼容，支持 LoRA/QLoRA 等高效微调算法，可在单张消费级显卡上快速微调 Qwen、Llama 等主流大模型，同时覆盖 SFT、DPO/GRPO 对齐训练与多模态任务，提供命令行与 WebUI 两种方式，一站式完成训练、推理、量化与部署。 |

2\. **训练数据**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
wget https://huggingface.co/datasets/ajimeno/PubTabNet/blob/main/pubtabnet.tar.gz</td>
</tr>
</tbody>
</table>

这里为了方便调试，我们准备了一个demo数据集，是从PubTabNet里采样了1k的训练集和200的验证集：

**\[pubtabnet_demo1k.zip\]**

3\. **MinerU2.5-1.2B模型权重**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
git clone https://huggingface.co/opendatalab/MinerU2.5-2509-1.2B</td>
</tr>
</tbody>
</table>

4\. **在[MinerU数据智能与前沿预料挑战赛平台](https://www.heywhale.com/org/sh_AI%20Lab/workspace/competition/area) 上进行模型微调**

> \*说明：在平台上，已经为大家配置好环境资源，上传了数据及模型，大家根据指引fork项目后运行即可

登陆[MinerU数据智能与前沿预料挑战赛平台](https://www.heywhale.com/org/sh_AI%20Lab/workspace/competition/area) （需要报名过训练营才能自动获得平台算力）。

点击链接获得《MinerU模型训练-沐曦》的访问权限，点击右上角【fork按钮】fork项目到自己的空间。

> https://www.heywhale.com/org/sh_AI%20Lab/coc-invitation/267d452ec2139f76794b4499#
>
> <img src="04课：MinerU 1.2B 模型微调/04课：MinerU 1.2B 模型微调 - 文档 - media/media/image1.png" style="width:5.75in;height:3.01042in" />
>
> <img src="04课：MinerU 1.2B 模型微调/04课：MinerU 1.2B 模型微调 - 文档 - media/media/image2.png" style="width:5.75in;height:4.625in" />

成功创建项目后，点击右上角【运行按钮】运行项目，点击【设置按钮】可修改算力资源及镜像（此处用默认的配置即可，不需要修改，后续需要修改学员可自行进行调整），在【其他设置】勾选【使用个人磁盘】开启个人work目录。

> <img src="04课：MinerU 1.2B 模型微调/04课：MinerU 1.2B 模型微调 - 文档 - media/media/image3.png" style="width:5.75in;height:2.94792in" />
>
> <img src="04课：MinerU 1.2B 模型微调/04课：MinerU 1.2B 模型微调 - 文档 - media/media/image4.png" style="width:5.75in;height:5.4375in" />
>
> <img src="04课：MinerU 1.2B 模型微调/04课：MinerU 1.2B 模型微调 - 文档 - media/media/image5.png" style="width:5.75in;height:2.76042in" />
>
> **work目录（用户工作区）**：写到该目录下的文件将被持久化存储，支持跨项目读写，不支持协作分享，访问路径为 /home/mw/work/。
>
> **project目录的存储空间是有限的，建议过程文件可以存放在temp目录中，需要跨项目读写的文件可存放在个人磁盘（work目录）中，若系统提示project目录空间已满，可以自行将文件写至work目录。**

待右上角显示资源已链接后可以逐条运行代码，也可点击【运行所有】一键运行全部代码。

> <img src="04课：MinerU 1.2B 模型微调/04课：MinerU 1.2B 模型微调 - 文档 - media/media/image6.png" style="width:5.75in;height:3.01042in" />

如果需要修改使用数据，可点击侧边栏修改挂载的数据和模型（可以使用自己创建的或他人共享的数据）。

> <img src="04课：MinerU 1.2B 模型微调/04课：MinerU 1.2B 模型微调 - 文档 - media/media/image7.png" style="width:5.36458in;height:3.69792in" />
>
> <img src="04课：MinerU 1.2B 模型微调/04课：MinerU 1.2B 模型微调 - 文档 - media/media/image8.png" style="width:5.75in;height:4.05208in" />

结束后点击右上角关闭实例释放资源，避免造成资源浪费。

> <img src="04课：MinerU 1.2B 模型微调/04课：MinerU 1.2B 模型微调 - 文档 - media/media/image9.png" style="width:5.75in;height:2.44792in" />

后续可在平台左侧【项目】或【我的空间】列表查看该项目，也可自己创建新的项目

> <img src="04课：MinerU 1.2B 模型微调/04课：MinerU 1.2B 模型微调 - 文档 - media/media/image10.png" style="width:5.75in;height:1.88542in" />

更多平台使用技巧可参考训练营介绍页面的平台使用教程

<img src="04课：MinerU 1.2B 模型微调/04课：MinerU 1.2B 模型微调 - 文档 - media/media/image11.png" style="width:5.75in;height:4.33333in" />

除了使用官方平台外，也可以在自己本地环境进行微调。

**三、快速开始**

1\. **环境安装**

下面各个库的安装版本在 NVIDIA A100 和 H200 以及 4090上都进行过测试，如果有其他的版本需求可以参考[ms-swift官方安装教程](https://swift.readthedocs.io/zh-cn/latest/GetStarted/SWIFT-installation.html)。

1.1 **新建conda环境**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
conda create -n mineru_sft python=3.10 -y</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
conda activate mineru_sft</td>
</tr>
</tbody>
</table>

1.2 **安装pytorch**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
pip install torch==2.6.0 torchvision==0.21.0 torchaudio==2.6.0 --index-url https://download.pytorch.org/whl/cu124</td>
</tr>
</tbody>
</table>

1.3 **安装vllm**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
pip install vllm==0.8.5 --extra-index-url https://download.pytorch.org/whl/cu124</td>
</tr>
</tbody>
</table>

一些比较老gcc版本的服务器上，可能有部分包无法编译，可以通过下面的命令来安装：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
conda install xxx -c conda-forge -y</td>
</tr>
</tbody>
</table>

1.4 **安装flash_attn**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
下载cuda安装文件<br />
wget https://developer.download.nvidia.com/compute/cuda/12.4.0/local_installers/cuda_12.4.0_550.54.14_linux.run</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
安装cuda<br />
bash cuda_12.4.0_550.54.14_linux.run</td>
</tr>
</tbody>
</table>

**注意存在驱动，不要再勾选驱动安装。（建议驱动和cuda分开安装，这里都不要勾选驱动）**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
写入环境变量<br />
echo 'export PATH=/usr/local/cuda-12.4/bin:$PATH' &gt;&gt; ~/.bashrc<br />
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-12.4/lib64:$LD_LIBRARY_PATH' &gt;&gt; ~/.bashrc<br />
source ~/.bashrc</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
pip install flash-attn==2.7.4.post1 --no-build-isolation</td>
</tr>
</tbody>
</table>

|  |
|:---|
| 这里如果flash_attn实在安装不上去的话，也可以跳过，对应的需要将训练的配置文件中的attn_impl改成sdpa，packing改成false。 |

1.5 **安装其他依赖**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
pip install -r requirements.txt</td>
</tr>
</tbody>
</table>

1.6 **以 Editable 模式安装子模块核心**

这一步非常关键，它能确保 Python 环境能正确识别 ms_swift_core

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
pip install -e ms_swift_core</td>
</tr>
</tbody>
</table>

2\. **数据准备**

我们在微调代码的dataset/demo路径下放置了4个demo数据集，可以参考这4个demo文件的格式来准备4个任务的数据格式。

特别的，我们针对表格识别任务，在微调代码中也准备了获取开源数据集PubTabNet并转换格式的代码和教程，可以参《表格识别训练数据准备》(相对路径：dataset/pubtabnet/prepare_train_data.md）

2.1 **修改python文件路径**

修改dataset/pubtabnet/convert_pubtabnet2mineru.py 里面的数据集路径。

注意**这里需要写绝对路径**。

2.2 **执行转换脚本：**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
python convert_pubtabnet2mineru.py</td>
</tr>
</tbody>
</table>

3\. **参数配置**

参数配置文件统一存储在configs目录下，我们提供了一个表格识别微调的demo配置configs/sft/sft_table_rec.yaml。**需要注意的是，配置文件中的**model datasets**两个key对应的值，在训练前需要改成绝对路径**。如果希望进行其他任务的微调，通过调整训练数据即可实现。更多的训练参数配置可参考\[ms-swift的参数设置文档\](ms_swift_core/docs/source/Instruction/Command-line-parameters.md)。

4\. **启动训练**

**不要直接使用** swift sft **命令，请使用**python run_mineru.py sft**进行替代。**该脚本会自动加载 mineru_ext 中的扩展功能。

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
CUDA_VISIBLE_DEVICES=0 python run_mineru.py sft --config configs/sft/sft_table_rec.yaml</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
NPROC_PER_NODE=2 CUDA_VISIBLE_DEVICES=0,1 python run_mineru.py sft --config configs/sft/sft_table_rec.yaml</td>
</tr>
</tbody>
</table>

4.1 **可能出现的问题**

训练**可能会遇到一些依赖包找不到的错误**，比如：RuntimeError: cannot load MPI library，可以用conda安装，会把依赖也一起装上：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Plain Text<br />
conda install -c conda-forge mpi4py</td>
</tr>
</tbody>
</table>

5\. **推理**

5.1 **端到端推理**

如果需要进行端到端任务的推理(输入图片，先进行布局检测，然后对各个检测框进行裁剪并且识别，最后输出整个页面的文档提取的结果)，需要额外安装mineru_vl_utils, 使用pip install mineru_vl_utils进行安装。安装完成后，可以使用下面的脚本进行端到端任务的推理：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
python mineru_ext/scripts/run_mineru2_5_e2e_local.py \<br />
--model "模型权重文件的路径" \<br />
--img_dir "图片文件夹的路径" \<br />
--output_json "输出的content list的json路径" \<br />
--output_dir "以markdown的形式保存的提取结果"</td>
</tr>
</tbody>
</table>

5.2 **单任务推理**

单任务是指利用单任务的prompt，直接让模型进行推理，使用方法如下：

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr>
<td style="text-align: left;">Bash<br />
python mineru_ext/scripts/run_mineru_vllm_singletask.py -i "输入的图片路径" -o "输出的路径" -m "模型权重路径" -p "选择prompt"</td>
</tr>
</tbody>
</table>

prompt需要和训练数据保持一致，目前mineru2.5使用的四种prompt为：

表格识别任务: "\nTable Recognition:"

公式识别任务: "\nFormula Recognition:"

文本识别任务: "\nText Recognition:"

布局分析任务: "\nLayout Detection:"
