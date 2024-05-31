# 1. Yuan2.0-gguf

## 1.1 概述
源2.0 是浪潮信息发布的新一代基础语言大模型。我们开源了全部的3个模型源2.0-102B，源2.0-51B和源2.0-2B。并且我们提供了预训练，微调，推理服务的相关脚本，以供研发人员做进一步的开发。源2.0是在源1.0的基础上，利用更多样的高质量预训练数据和指令微调数据集，令模型在语义、数学、推理、代码、知识等不同方面具备更强的理解能力。

详情请参考Yuan2.0模型
<a href="https://arxiv.org/ftp/arxiv/papers/2311/2311.15786.pdf" target="_blank">**技术报告**</a>和
<a href="https://github.com/IEIT-Yuan/Yuan-2.0" target="_blank">**github**</a>


本项目基于llama.cpp（version：b1742）在Windows系统（CPU Only）上对源2.0-2B模型的适配。

由于源2.0模型结构与llama结构存在差异，针对源2.0模型（Yuan2.0-2B）模型LFA结构，进行如下修改：

- llama计算图中添加LFA结构，修改ggml_conv_1d逻辑，以适配源2.0，保证前后序列长度不变；
- llama计算图中添加q、k混合相关的逻辑；
- 修改IM2COL算子，修改数组的读取方式；
- 修改ADD算子，将卷积模块的输出进行转置，以适配后面的计算；
- 修改concat算子，以适配q、k混合的逻辑；
- 支持多线程推理，加速生成速率；

目前支持fp16精度模型的gguf文件转换，后续会持续进行其他精度的工作。

## 1.2. gguf文件的生成

```shell
python convert.py --model yuan2b-hf\yuan2-2B --outfile zh-models/Yuan2-2B-Februa-hf-GGUF.gguf
```

## 1.3. 编译
  - Using `make`:
  - On Linux or MacOS:

      ```bash
      make
      ```

  - On Windows:

    1. Download the latest fortran version of [w64devkit](https://github.com/skeeto/w64devkit/releases).
    2. Extract `w64devkit` on your pc.
    3. Run `w64devkit.exe`.
    4. Use the `cd` command to reach the `llama.cpp` folder.
    5. From here you can run:
        ```bash
        make
        ```

- Using `CMake`:

    ```bash
    mkdir build
    cd build
    cmake ..
    cmake --build . --config Release


## 1.4. 测试Demo

### <font color=#FFC125 >1.4.1 测试环境 </font>

- python3.9
- 11th Gen Intel(R) Core(TM) i5-1145G7 @2.60GHz 2.61GHz
- 8.00GB RAM
- Windows 10专业版（21H1）

### <font color=#FFC125 >1.4.2 测试方法 </font>


```shell
main.exe -m D:\\llama-cpp\\llama.cpp\\zh-models\\Yuan2-2B-Februa-hf-GGUF.gguf -p "北京简介" -n 400  --top-k  5 --threads 4
```


### <font color=#FFC125 >1.4.3 测试效果 </font>

```shell
llama_new_context_with_model: n_ctx      = 512
llama_new_context_with_model: freq_base  = 10000.0
llama_new_context_with_model: freq_scale = 1
llama_new_context_with_model: KV self size  =   96.00 MiB, K (f16):   48.00 MiB, V (f16):   48.00 MiB
llama_build_graph: non-view tensors processed: 628/820
llama_build_graph: ****************************************************************
llama_build_graph: not all non-view tensors have been processed with a callback
llama_build_graph: this can indicate an inefficiency in the graph implementation
llama_build_graph: build with LLAMA_OFFLOAD_DEBUG for more info
llama_build_graph: ref: https://github.com/ggerganov/llama.cpp/pull/3837
llama_build_graph: ****************************************************************
llama_new_context_with_model: compute buffer total size = 270.94 MiB
Model metadata: {'tokenizer.ggml.add_eos_token': 'true', 'tokenizer.ggml.padding_token_id': '77185', 'tokenizer.ggml.seperator_token_id': '77185', 'tokenizer.ggml.eos_token_id': '77185', 'general.architecture': 'llama', 'llama.context_length': '8192', 'general.name': 'E:\\ckpts\\yuan2b-hf', 'tokenizer.ggml.add_bos_token': 'true', 'llama.embedding_length': '2048', 'llama.feed_forward_length': '8192', 'llama.attention.layer_norm_rms_epsilon': '0.000001', 'llama.rope.dimension_count': '64', 'llama.attention.head_count': '32', 'tokenizer.ggml.bos_token_id': '77185', 'llama.block_count': '24', 'llama.attention.head_count_kv': '32', 'tokenizer.ggml.model': 'llama', 'general.file_type': '1'}
北京是中国的首都，是华北地区的中心城市。它是中国政治、文化和经济的中心，也是中国最大的城市之一。北京具有悠久的历史和丰富的文化遗产，包括长城、故宫、天坛等世界闻名的古迹，以及现代化的摩天大楼和购物中心。
北京也是中国的教育和科研中心，拥有众多高等院校和科研机构，为中国培养了许多杰出人才。此外，北京还是中国重要的商业和交通中心，拥有众多的商业区和购物中心。
北京还是一个旅游胜地，吸引着大量的国内外游客前来参观和旅游。著名的景点包括故宫博物院、颐和园、天坛公园、八达岭长城等。这些景点都有着独特的历史文化背景和壮观的自然风光，给人们带来了不同的体验。
总的来说，北京是一个历史悠久、文化底蕴丰富、旅游胜地众多的城市，值得一游。无论是对于艺术爱好者、历史文化爱好者还是自然探索者，北京都有着不可多得的体验。


llama_print_timings:        load time =     579.04 ms
llama_print_timings:      sample time =     126.74 ms /   193 runs   (    0.66 ms per token,  1522.81 tokens per second)
llama_print_timings: prompt eval time =     578.98 ms /     4 tokens (  144.74 ms per token,     6.91 tokens per second)
llama_print_timings:        eval time =   20562.87 ms /   192 runs   (  107.10 ms per token,     9.34 tokens per second)
llama_print_timings:       total time =   22961.55 ms

```
### <font color=#FFC125 >1.4.4 测试性能 </font>


<table>
    <tr>
        <th rowspan="2">推理性能</th><th>GGUF格式（C++）</th><th>HF格式（Python）</th><th>加速比</th>
    </tr>
    <tr>
        <td align="center">9.16 tokens/s</td><td align="center">1.21 tokens/s</td><td align="center">7.57</td>
    </tr>
    <tr>
        <th rowspan="2">内存占用</th><th>GGUF格式（C++）</th><th>HF格式（Python）</th><th>内存占比（GGUF/HF）</th>
    </tr>
    <tr>
        <td align="center">~0.4 GB</td><td align="center">~8.6 GB</td><td align="center">4.65%</td>
    </tr>
</table>









# 2. Yuan2.0-M32-gguf

20240531更新

- 支持yuan2b、yuan2b-moe；
- 修改lfa输入缓存为2个token，内存占用减少，推理速度略微提升；
- 修改-i交互模式下的若干bug；

## 2.1 概述
浪潮信息 **“源2.0 M32”大模型（简称，Yuan2.0-M32）** 采用稀疏混合专家架构（MoE），以Yuan2.0-2B模型作为基底模型，通过创新的门控网络（Attention Router）实现32个专家间（Experts*32）的协同工作与任务调度，在显著降低模型推理算力需求的情况下，带来了更强的模型精度表现与推理性能；源2.0-M32在多个业界主流的评测进行了代码生成、数学问题求解、科学问答与综合知识能力等方面的能力测评。结果显示，源2.0-M32在多项任务评测中，展示出了较为先进的能力表现，MATH（数学求解）、ARC-C（科学问答）测试精度超过LLaMA3-700亿模型。**Yuan2.0-M32大模型** 基本信息如下：

+ **模型参数量：** 40B <br>
+ **专家数量：** 32 <br>
+ **激活专家数：** 2 <br>
+ **激活参数量：** 3.7B <br>
+ **训练数据量：** 2000B tokens <br>
+ **支持序列长度：** 16K <br>

详情请参考Yuan2.0-M32模型
<a href="https://arxiv.org/abs/2405.17976" target="_blank">**技术报告**</a>和
<a href="https://github.com/IEIT-Yuan/Yuan2.0-M32" target="_blank">**github**</a>


## 2.1 如何转换gguf
    # 由于hf格式ckpt的transformers版本与转换gguf的transformers不一致，需要使用hf加载，然后再保存一下；
    # 可运行的环境如下：
    python3.9 cpu win10/linux
    torch                     2.0.1                    pypi_0    pypi
    transformers              4.39.0                   pypi_0    pypi

    # transformers加载
    model = AutoModelForCausalLM.from_pretrained(path, device_map="cpu", trust_remote_code=True,torch_dtype=torch.bfloat16).eval()
    # 保存模型
    torch.save(model.state_dict(),r"E:\ckpts\yuan2b-moe-2\pytorch_model_hf2-bk-fp16.bin")

    # 使用新生成的bin，再使用convert.py，不会报错

    convert.py的传入参数model_type，需要根据模型的结构进行修改；

## 2.2 编译
    跟yuan2-gguf的编译方法一样；

## 2.3 推理脚本：
    windows:
    ./main.exe -m yuan2b-moe-40b-q4_0.gguf -t 6 -n 20 -p '北京简介'

    linux：
    ./main -m /mnt/md0/yuan2b-moe-40b-24-f16-hf.gguf --file /mnt/md0/shenchong/llama.cpp-gitee/prompt_text.txt -c 1024 -b 1024 -t 64 -n 10


## 2.4 量化脚本：
    windows：
    ./quantize.exe yuan2b-moe-40b-24-f16-hf.gguf yuan2b-moe-40b-q4_0.gguf q4_0

    linux：
    ./quantize yuan2b-moe-40b-24-f16-hf.gguf yuan2b-moe-40b-q4_0.gguf q4_0


# 3 faq

- 1、是否支持gpu推理？是否支持其他gguf模型？
  - 由于修改了gguf原有的一些算子实现，该项目目前并不能使用gpu进行推理，而且也不能跑llama等其他gguf模型；

- 2、支持模型列表
  - yuan2.0-2b
  - yuan2.0-m32



