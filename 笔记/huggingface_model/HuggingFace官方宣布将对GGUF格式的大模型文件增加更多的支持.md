当前的大模型的参数规模较大，数以千亿的参数导致了它们的预训练结果文件都在几十GB甚至是几百GB，这不仅导致其使用成本很高，在不同平台进行交换也非常困难。因此，**大模型预训练结果文件的保存格式对于模型的使用和生态的发展来说极其重要**。昨天HuggingFace官方宣布将推动GGUF格式的大模型文件在HuggingFace上的使用。

#### 大模型预训练结果文件格式GGUF简介

大语言模型的开发通常使用PyTorch等框架，其预训练结果通常也会保存为相应的二进制格式，如pt后缀的文件通常就是PyTorch框架保存的二进制预训练结果。

但是，大模型的存储一个很重要的问题是它的模型文件巨大，而模型的结构、参数等也会影响模型的推理效果和性能。为了让大模型更加高效的存储和交换，就有了不同格式的大模型文件。其中，GGUF就是非常重要的一种大模型文件格式。

**GGUF文件全称是GPT-Generated Unified Format，是由Georgi Gerganov定义发布的一种大模型文件格式。Georgi Gerganov是著名开源项目llama.cpp的创始人。**

**GGUF就是一种二进制格式文件的规范，原始的大模型预训练结果经过转换后变成GGUF格式可以更快地被载入使用，也会消耗更低的资源**。原因在于GGUF采用了多种技术来保存大模型预训练结果，包括采用紧凑的二进制编码格式、优化的数据结构、内存映射等。

下图展示了GGUF格式的具体情况：

![](https://www.datalearner.com/resources/blog_images/9e9739b1-9a4f-4ee8-83e1-7b7e82fa0ba5.jpg)

  

GGUF格式的前身是GGML，也是同一个作者开发的，但是GGML格式因为种种限制目前被放弃。关于GGUF格式的大模型文件大家可以参考此前DataLearnerAI做的详细说明： [GGUF格式的大模型文件是什么意思？如何使用？为什么有GGUF格式的大模型文件？GGUF大模型文件与GGML的差异是啥？](https://www.datalearner.com/blog/1051705718835586 "GGUF格式的大模型文件是什么意思？如何使用？为什么有GGUF格式的大模型文件？GGUF大模型文件与GGML的差异是啥？")

#### HuggingFace宣布加大对GGUF格式文件的支持

HuggingFace是大模型开源领域当之无愧的第一平台。因此，它们对某种技术的支持也会极大得影响该技术的发展。

此前，HuggingFace推出过safetensors格式的大模型文件。专为安全高效地存储张量而设计。与PyTorch中使用的pickle格式不同，后者加载文件时可能会执行任意代码，从而带来安全风险，safetensors提供了一个更安全的选择。

**可能是此前的设计过于关注pt文件的不安全性，safetensors没有关注性能和跨平台交换等方面。**虽然safetensors也被很多厂商采用，但是在大模型高效序列化、数据压缩、量化等方面不足。而且它只保存了张量数据，没有任何关于模型的元数据信息。而GGUF格式因为良好的设计，通过各种优化手段实现了快速的模型加载，这对于需要频繁载入不同模型的场景尤为重要，同时将模型的元数据信息压缩到二进制文件中，对于跨平台操作和传输都非常有价值。因此，被很多开发者认可。

HuggingFace的工作人员称**HF平台上GGUF格式的大模型文件爆发式增长且似乎没有慢下来**。为此，他们决定对GGUF格式文件做更多的支持。主要包括：

##### HuggingFace模型检索支持按GGUF格式过滤

HuggingFace的模型检索支持按照GGUF格式过滤，大家可以通过[https://huggingface.co/models?library=gguf](https://huggingface.co/models?library=gguf) 链接来筛选访问。

![](https://www.datalearner.com/resources/blog_images/c59bbdf4-87ea-4a93-b6db-77017b9246c5.png)

  

可以看到，Google家开源的Gemma系列已经支持了GGUF格式文件了。

##### 可以在HuggingFace平台上预览GGUF的元数据

可以直接在**HF平台上对GGUF的元数据进行预览**，包括模型的架构、具体参数等。下图是谷歌Gemma模型的一个实例：

![](https://www.datalearner.com/resources/blog_images/c2f5f951-c47d-427a-acb8-a6fc16d76fec.png)

  

注意，该特性可能目前还在测试中，DataLearnerAI并未在平台上找到。

##### HuggingFace提供工具支持网站显示平台GGUF格式模型信息

HuggingFace开发了一个JavaScript脚本，可以用来解析远端的HuggingFace Hub上模型的信息，这意味着未来很多网站在宣传自己的模型的时候使用这个技术可以实时展示HuggingFace平台上模型的信息！

这些行动表明，HuggingFace认可GGUF的技术并积极推广支持这项技术。而GGUF的创建者、llama.cpp项目的开发者Georgi Gerganov则转发了HF这个信息并说明：

> GGUF文件格式是一个很好的例子，展示了开源社区可以实现的酷炫事物。

#### HuggingFace支持GGUF格式文件的总结和思考

从HuggingFace积极支持GGUF格式文件的事情我们至少可以看到几个结果。一个是**开源社区对大模型的贡献是令所有人都受益的且不可缺少的**。即便是有着强大用户基础和开源经验的HuggingFace，在考虑到要在大模型文件上做点事情的时候也没有那么周全。safetensors的设计考虑了安全但却忽略了大模型的使用性能和跨平台交换等需求。

而另一个角度看，国外**开源生态的互相融合和促进也是令人羡慕**。且不说国产开源生态的贫瘠。换个角度说，如果国内大厂有了类似HF平台，还有可能引入支持开源的GGUF格式这种事情吗？大概率应该是继续演进safetensors，把开源特性“抄袭”过来而已。

作者：阿拉姆  
链接：https://www.zhihu.com/question/622339524/answer/3458752406  
来源：知乎  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。  
  

### **llama.cpp: gguf文件解析**

想学习一下llama.cpp项目中的模型格式gguf。llama.cpp之前支持的是ggml文件格式，新版本只支持gguf。本文讲解一下gguf的文件结构和完整解析的python代码。

## **ggml vs gguf**

gguf(GPT-Generated Unified Format)是ggml(GPT-Generated Model Language)升级版，从2023年8月开始支持。ggml有以下缺点：

- 无版本信息，导致无法管理，和向后兼容
- 增加或者修改信息非常不灵活
- 手动修改模型信息很困难

gguf特性：

- 方便添加新信息到模型中
- mmap兼容，能够直接将文件映射到内存中，而不需要要先load到内存，然后再从内存中各个地方搬运，问题这是怎么实现的？
- 简化模型加载和保存过程，消除额外的第三方依赖
- 模型文件包含所有的关键信息，而不需要用户的额外输入
- 量化兼容
- key-value structure，也被称为metadata，这种转变允许增加新信息，而不会破坏原来的模型结构，之前的模型完全是按照顺序保存数据，模型文件是写死的，没法动。修改模型文件会导致模型没法使用。

例子：

1. 添加key-value。给gguf添加一个chat_template，[https://blog.csdn.net/bruceunx/article/details/136543532](https://link.zhihu.com/?target=https%3A//blog.csdn.net/bruceunx/article/details/136543532)
2. 修改value。修改gguf的模型头部信息（metadata）里的value：[https://github.com/ggerganov/llama.cpp/blob/master/gguf-py/scripts/gguf-set-metadata.py](https://link.zhihu.com/?target=https%3A//github.com/ggerganov/llama.cpp/blob/master/gguf-py/scripts/gguf-set-metadata.py)

知道模型结构的优劣和gguf的特性之后，还是觉得有点虚，特别是上面这一系列的gguf特性到底意味着什么？遂决定照着huggingface做的gguf的可视化脚本改写了一个python版本的`gguf_reader.py`。从0到1的解析完一遍模型，。再进一步了解一下gguf的每个单元是做什么的，下面这张图很好的展示了gguf的结构：

![](https://pic1.zhimg.com/80/v2-40bb06e675a21dc3945ce44d4752704b_720w.webp?source=1def8aca)

图片来源：https://huggingface.co/docs/hub/gguf

也没什么可说的，该有的图上都已经展示好了，要注意的是，gguf v1中的tensor_count 和metadata_kv_count是uint32而不是上图的uint64。其次是tensor_info中的offset值是从0开始，随着一个个tensor的累加，一个个增加。offset 0位置是在上图中reset of file位置开始，而不是整个模型的开始，所以在metadata部分插入或者删除，或者修改value之后，再将后面的rest of file文件拼接起来，并不会损坏模型结构。

```python
# reference:https://github.com/huggingface/huggingface.js/blob/main/packages/gguf/src/gguf.ts
# 目前脚本只支持小端格式的模型
import sys
from typing import Any
from enum import IntEnum
​
import numpy as np
import numpy.typing as npt
​
class GGUFValueType(IntEnum):
    UINT8   = 0
    INT8    = 1
    UINT16  = 2
    INT16   = 3
    UINT32  = 4
    INT32   = 5
    FLOAT32 = 6
    BOOL    = 7
    STRING  = 8
    ARRAY   = 9
    UINT64  = 10
    INT64   = 11
    FLOAT64 = 12
​
def check_version(version):
    if version == 1 or version == 2 or version == 3:
        return True
    else:
        return False
​
def data_get(
    data, offset: int, dtype: npt.DTypeLike, count: int = 1) -> npt.NDArray[Any]:
    count = int(count)
    itemsize = int(np.empty([], dtype = dtype).itemsize)
    end_offs = offset + itemsize * count
    return (
        data[offset:end_offs]
        .view(dtype = dtype)[:count]
        # .newbyteorder(override_order)
    )
​
def data_read_version_size(data, offset: int, version: int):
    if version == 1:
        return data_get(data, offset, np.uint32)[0], 4
    elif version == 2 or version == 3:
        return data_get(data, offset, np.uint64)[0], 8
    else:
        raise ValueError(f'Sorry, file appears to be version {version} which we cannot handle')
​
def data_read_string(data, offset: int, version: int):
    str_length, str_length_len = data_read_version_size(data, offset, version)
    # 在内存上切出来string部分的数据
    byte = data[offset+int(str_length_len):offset+int(str_length_len)+int(str_length)]
    value = byte.tobytes().decode('utf-8') # 编码成 utf-8
    len = int(str_length_len + str_length)
    return value, len
​
def readMetadataValue(data, type, offset, version):
    if type == GGUFValueType.UINT8:
        return data_get(data, np.uint8)[0], 1
    elif type == GGUFValueType.INT8:
        return data_get(data, np.int8)[0], 1
    elif type == GGUFValueType.UINT16:
        return data_get(data, offset, np.uint16)[0], 2
    elif type == GGUFValueType.INT16:
        return data_get(data, offset, np.int16)[0], 2
    elif type == GGUFValueType.UINT32:
        return data_get(data, offset, np.uint32)[0], 4
    elif type == GGUFValueType.INT32:
        return data_get(data, offset, np.int32)[0], 4
    elif type == GGUFValueType.FLOAT32:
        return data_get(data, offset, np.float32)[0], 4
    elif type == GGUFValueType.BOOL:
        return data_get(data, offset, np.uint8)[0], 1
    elif type == GGUFValueType.STRING:
        return data_read_string(data, offset, version=version)
    elif type == GGUFValueType.ARRAY:
        typeArray = data_get(data, offset, np.uint32)
        typeLength = 4
        lengthArray, lengthLength = data_read_version_size(data, offset + typeLength, version=version)
        length = typeLength + lengthLength
​
        arrayValues = []
        for i in range(lengthArray):
            value, len = readMetadataValue(data, typeArray, offset= offset + length, version=version)
            arrayValues.append(value)
            length += len
​
        return arrayValues, length
    elif type == GGUFValueType.UINT64:
        return data_get(data, offset, np.uint64)[0], 8
    elif type == GGUFValueType.INT64:
        return data_get(data, offset, np.int64)[0], 8
    elif type == GGUFValueType.FLOAT64:
        return data_get(data, offset, np.float64)[0], 8
    else:
        raise ValueError(f'Sorry, un-supported GGUFValueType {type}!')
​
def parse_gguf(model_path):
    data = np.memmap(model_path, mode = 'r')
​
    offs = 0
    magic = data_get(data, offs, np.uint32).tobytes()
    print("magic", magic.decode('utf-8'))
    if (magic != b'GGUF'):
        print("is not gguf file")
        sys.exit(1)
​
    offs += 4
    version = data_get(data, offs, np.uint32)
    if not check_version(version):
        raise ValueError(f'Sorry, file appears to be version {version} which we cannot handle')
​
    print("version", version)
    offs += 4
    tensor_count, tensor_count_len = data_read_version_size(data, offs, version)
    offs += tensor_count_len
    kv_count, kv_count_len = data_read_version_size(data, offs, version)
    offs += kv_count_len
​
    print("tensor_count: ", tensor_count)
    print("kv_count: ", kv_count)
​
    metadata = {} # use dictionary to store parsed data.
​
    # 解析gguf 头部信息
    for i in range(kv_count):
        key, k_len = data_read_string(data, offs, version)
        offs += k_len
        
        type = data_get(data, offs, np.uint32)[0]
        offs += 4
​
        value, len = readMetadataValue(data, type, offs, version)
        if len > 100:
            print("i = ", i, ", k-v = ", key, ":", value[:100])
        else:
            print("i = ", i, ", k-v = ", key, ":", value)
        offs += len
        metadata[key] = value
​
    # 解析tensor info的信息
    for i in range(tensor_count):
        key, k_len = data_read_string(data, offs, version)
        offs += k_len
        
        nDims = data_get(data, offs, np.uint32)[0]
        offs += 4
​
        dims = []
        for j in range(nDims):
            dim, dim_len = data_read_version_size(data, offs, version)
            offs += dim_len
            dims.append(dim)
        
        type = data_get(data, offs, np.uint32)[0]
        offs += 4
​
        tensorOffset = data_get(data, offs, np.uint64)[0]
        offs += 8
​
        print("tensor i = ", i, ", k = ", key, ", type = ", type, ", dims = ", dims, ", tensorOffset = ", tensorOffset)
​
if __name__ == '__main__':
    model_path = "./TinyLlama-1.1B-Chat-v1.0/tinyllama-1.1b-chat-v1.0.Q2_K.gguf"
    parse_gguf(model_path)
```

模型在网站上的渲染截图：

[https://huggingface.co/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF/tree/main?show_tensors=tinyllama-1.1b-chat-v1.0.Q2_K.gguf](https://link.zhihu.com/?target=https%3A//huggingface.co/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF/tree/main%3Fshow_tensors%3Dtinyllama-1.1b-chat-v1.0.Q2_K.gguf)

![](https://pic1.zhimg.com/80/v2-5a2714ef05da24be548d4728890ed98d_720w.webp?source=1def8aca)

图片来源：https://huggingface.co/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF/tree/main?show_tensors=tinyllama-1.1b-chat-v1.0.Q2_K.gguf

以下是脚本输出解析的输出，tensor的顺序和上图顺序有点不一样，看来hf网站是排序过的，我这里是按照tensor的offset来排序：

```text
magic GGUF
version [3]
tensor_count:  201
kv_count:  23
i =  0 , k-v =  general.architecture : llama
i =  1 , k-v =  general.name : tinyllama_tinyllama-1.1b-chat-v1.0
i =  2 , k-v =  llama.context_length : 2048
i =  3 , k-v =  llama.embedding_length : 2048
i =  4 , k-v =  llama.block_count : 22
i =  5 , k-v =  llama.feed_forward_length : 5632
i =  6 , k-v =  llama.rope.dimension_count : 64
i =  7 , k-v =  llama.attention.head_count : 32
i =  8 , k-v =  llama.attention.head_count_kv : 4
i =  9 , k-v =  llama.attention.layer_norm_rms_epsilon : 1e-05
i =  10 , k-v =  llama.rope.freq_base : 10000.0
i =  11 , k-v =  general.file_type : 10
i =  12 , k-v =  tokenizer.ggml.model : llama
i =  13 , k-v =  tokenizer.ggml.tokens : ['<unk>', '<s>', '</s>', ...]
i =  14 , k-v =  tokenizer.ggml.scores : [0.0, 0.0, 0.0, 0.0, 0.0, ...]
i =  15 , k-v =  tokenizer.ggml.token_type : [2, 3, 3, 6, 6, 6, 6, ...]
i =  16 , k-v =  tokenizer.ggml.merges : ['▁ t', 'e r', 'i n', '▁ a', 'e n', ...]
i =  17 , k-v =  tokenizer.ggml.bos_token_id : 1
i =  18 , k-v =  tokenizer.ggml.eos_token_id : 2
i =  19 , k-v =  tokenizer.ggml.unknown_token_id : 0
i =  20 , k-v =  tokenizer.ggml.padding_token_id : 2
i =  21 , k-v =  tokenizer.chat_template : {% for message in messages %}
{% if message['role'] == 'user' %}
{{ '<|user|>
' + message['content']
i =  22 , k-v =  general.quantization_version : 2
tensor i =  0 , k =  output.weight , type =  14 , dims =  [2048, 32000] , tensorOffset =  0
tensor i =  1 , k =  token_embd.weight , type =  10 , dims =  [2048, 32000] , tensorOffset =  53760000
tensor i =  2 , k =  blk.0.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  75264000
tensor i =  3 , k =  blk.0.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  75272192
tensor i =  4 , k =  blk.0.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  80228352
tensor i =  5 , k =  blk.0.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  85184512
tensor i =  6 , k =  blk.0.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  90140672
tensor i =  7 , k =  blk.0.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  90148864
tensor i =  8 , k =  blk.0.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  90320896
tensor i =  9 , k =  blk.0.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  92123136
tensor i =  10 , k =  blk.0.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  93499392
tensor i =  11 , k =  blk.1.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  93724672
tensor i =  12 , k =  blk.1.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  93732864
tensor i =  13 , k =  blk.1.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  98689024
tensor i =  14 , k =  blk.1.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  103645184
tensor i =  15 , k =  blk.1.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  108601344
tensor i =  16 , k =  blk.1.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  108609536
tensor i =  17 , k =  blk.1.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  108781568
tensor i =  18 , k =  blk.1.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  110583808
tensor i =  19 , k =  blk.1.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  111960064
tensor i =  20 , k =  blk.10.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  112185344
tensor i =  21 , k =  blk.10.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  112193536
tensor i =  22 , k =  blk.10.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  117149696
tensor i =  23 , k =  blk.10.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  122105856
tensor i =  24 , k =  blk.10.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  127062016
tensor i =  25 , k =  blk.10.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  127070208
tensor i =  26 , k =  blk.10.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  127242240
tensor i =  27 , k =  blk.10.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  129044480
tensor i =  28 , k =  blk.10.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  130420736
tensor i =  29 , k =  blk.11.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  130646016
tensor i =  30 , k =  blk.11.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  130654208
tensor i =  31 , k =  blk.11.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  135610368
tensor i =  32 , k =  blk.11.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  140566528
tensor i =  33 , k =  blk.11.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  145522688
tensor i =  34 , k =  blk.11.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  145530880
tensor i =  35 , k =  blk.11.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  145702912
tensor i =  36 , k =  blk.11.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  147505152
tensor i =  37 , k =  blk.11.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  148881408
tensor i =  38 , k =  blk.12.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  149106688
tensor i =  39 , k =  blk.12.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  149114880
tensor i =  40 , k =  blk.12.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  154071040
tensor i =  41 , k =  blk.12.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  159027200
tensor i =  42 , k =  blk.12.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  163983360
tensor i =  43 , k =  blk.12.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  163991552
tensor i =  44 , k =  blk.12.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  164163584
tensor i =  45 , k =  blk.12.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  165965824
tensor i =  46 , k =  blk.12.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  167342080
tensor i =  47 , k =  blk.13.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  167567360
tensor i =  48 , k =  blk.13.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  167575552
tensor i =  49 , k =  blk.13.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  172531712
tensor i =  50 , k =  blk.13.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  177487872
tensor i =  51 , k =  blk.13.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  182444032
tensor i =  52 , k =  blk.13.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  182452224
tensor i =  53 , k =  blk.13.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  182624256
tensor i =  54 , k =  blk.13.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  184426496
tensor i =  55 , k =  blk.13.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  185802752
tensor i =  56 , k =  blk.14.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  186028032
tensor i =  57 , k =  blk.14.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  186036224
tensor i =  58 , k =  blk.14.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  190992384
tensor i =  59 , k =  blk.14.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  195948544
tensor i =  60 , k =  blk.14.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  200904704
tensor i =  61 , k =  blk.14.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  200912896
tensor i =  62 , k =  blk.14.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  201084928
tensor i =  63 , k =  blk.14.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  202887168
tensor i =  64 , k =  blk.14.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  204263424
tensor i =  65 , k =  blk.15.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  204488704
tensor i =  66 , k =  blk.15.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  204496896
tensor i =  67 , k =  blk.15.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  209453056
tensor i =  68 , k =  blk.15.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  214409216
tensor i =  69 , k =  blk.15.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  219365376
tensor i =  70 , k =  blk.15.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  219373568
tensor i =  71 , k =  blk.15.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  219545600
tensor i =  72 , k =  blk.15.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  221347840
tensor i =  73 , k =  blk.15.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  222724096
tensor i =  74 , k =  blk.16.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  222949376
tensor i =  75 , k =  blk.16.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  222957568
tensor i =  76 , k =  blk.16.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  227913728
tensor i =  77 , k =  blk.16.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  232869888
tensor i =  78 , k =  blk.16.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  237826048
tensor i =  79 , k =  blk.16.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  237834240
tensor i =  80 , k =  blk.16.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  238006272
tensor i =  81 , k =  blk.16.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  239808512
tensor i =  82 , k =  blk.16.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  241184768
tensor i =  83 , k =  blk.17.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  241410048
tensor i =  84 , k =  blk.17.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  241418240
tensor i =  85 , k =  blk.17.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  246374400
tensor i =  86 , k =  blk.17.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  251330560
tensor i =  87 , k =  blk.17.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  256286720
tensor i =  88 , k =  blk.17.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  256294912
tensor i =  89 , k =  blk.17.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  256466944
tensor i =  90 , k =  blk.17.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  258269184
tensor i =  91 , k =  blk.17.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  259645440
tensor i =  92 , k =  blk.18.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  259870720
tensor i =  93 , k =  blk.18.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  259878912
tensor i =  94 , k =  blk.18.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  264835072
tensor i =  95 , k =  blk.18.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  269791232
tensor i =  96 , k =  blk.18.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  274747392
tensor i =  97 , k =  blk.18.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  274755584
tensor i =  98 , k =  blk.18.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  274927616
tensor i =  99 , k =  blk.18.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  276729856
tensor i =  100 , k =  blk.18.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  278106112
tensor i =  101 , k =  blk.19.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  278331392
tensor i =  102 , k =  blk.19.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  278339584
tensor i =  103 , k =  blk.19.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  283295744
tensor i =  104 , k =  blk.19.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  288251904
tensor i =  105 , k =  blk.19.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  293208064
tensor i =  106 , k =  blk.19.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  293216256
tensor i =  107 , k =  blk.19.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  293388288
tensor i =  108 , k =  blk.19.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  295190528
tensor i =  109 , k =  blk.19.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  296566784
tensor i =  110 , k =  blk.2.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  296792064
tensor i =  111 , k =  blk.2.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  296800256
tensor i =  112 , k =  blk.2.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  301756416
tensor i =  113 , k =  blk.2.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  306712576
tensor i =  114 , k =  blk.2.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  311668736
tensor i =  115 , k =  blk.2.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  311676928
tensor i =  116 , k =  blk.2.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  311848960
tensor i =  117 , k =  blk.2.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  313651200
tensor i =  118 , k =  blk.2.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  315027456
tensor i =  119 , k =  blk.20.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  315252736
tensor i =  120 , k =  blk.20.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  315260928
tensor i =  121 , k =  blk.20.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  320217088
tensor i =  122 , k =  blk.20.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  325173248
tensor i =  123 , k =  blk.20.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  330129408
tensor i =  124 , k =  blk.20.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  330137600
tensor i =  125 , k =  blk.20.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  330309632
tensor i =  126 , k =  blk.20.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  332111872
tensor i =  127 , k =  blk.20.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  333488128
tensor i =  128 , k =  blk.21.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  333713408
tensor i =  129 , k =  blk.21.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  333721600
tensor i =  130 , k =  blk.21.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  338677760
tensor i =  131 , k =  blk.21.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  343633920
tensor i =  132 , k =  blk.21.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  348590080
tensor i =  133 , k =  blk.21.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  348598272
tensor i =  134 , k =  blk.21.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  348770304
tensor i =  135 , k =  blk.21.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  350572544
tensor i =  136 , k =  blk.21.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  351948800
tensor i =  137 , k =  blk.3.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  352174080
tensor i =  138 , k =  blk.3.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  352182272
tensor i =  139 , k =  blk.3.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  357138432
tensor i =  140 , k =  blk.3.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  362094592
tensor i =  141 , k =  blk.3.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  367050752
tensor i =  142 , k =  blk.3.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  367058944
tensor i =  143 , k =  blk.3.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  367230976
tensor i =  144 , k =  blk.3.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  369033216
tensor i =  145 , k =  blk.3.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  370409472
tensor i =  146 , k =  blk.4.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  370634752
tensor i =  147 , k =  blk.4.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  370642944
tensor i =  148 , k =  blk.4.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  375599104
tensor i =  149 , k =  blk.4.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  380555264
tensor i =  150 , k =  blk.4.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  385511424
tensor i =  151 , k =  blk.4.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  385519616
tensor i =  152 , k =  blk.4.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  385691648
tensor i =  153 , k =  blk.4.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  387493888
tensor i =  154 , k =  blk.4.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  388870144
tensor i =  155 , k =  blk.5.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  389095424
tensor i =  156 , k =  blk.5.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  389103616
tensor i =  157 , k =  blk.5.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  394059776
tensor i =  158 , k =  blk.5.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  399015936
tensor i =  159 , k =  blk.5.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  403972096
tensor i =  160 , k =  blk.5.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  403980288
tensor i =  161 , k =  blk.5.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  404152320
tensor i =  162 , k =  blk.5.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  405954560
tensor i =  163 , k =  blk.5.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  407330816
tensor i =  164 , k =  blk.6.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  407556096
tensor i =  165 , k =  blk.6.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  407564288
tensor i =  166 , k =  blk.6.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  412520448
tensor i =  167 , k =  blk.6.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  417476608
tensor i =  168 , k =  blk.6.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  422432768
tensor i =  169 , k =  blk.6.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  422440960
tensor i =  170 , k =  blk.6.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  422612992
tensor i =  171 , k =  blk.6.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  424415232
tensor i =  172 , k =  blk.6.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  425791488
tensor i =  173 , k =  blk.7.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  426016768
tensor i =  174 , k =  blk.7.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  426024960
tensor i =  175 , k =  blk.7.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  430981120
tensor i =  176 , k =  blk.7.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  435937280
tensor i =  177 , k =  blk.7.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  440893440
tensor i =  178 , k =  blk.7.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  440901632
tensor i =  179 , k =  blk.7.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  441073664
tensor i =  180 , k =  blk.7.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  442875904
tensor i =  181 , k =  blk.7.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  444252160
tensor i =  182 , k =  blk.8.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  444477440
tensor i =  183 , k =  blk.8.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  444485632
tensor i =  184 , k =  blk.8.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  449441792
tensor i =  185 , k =  blk.8.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  454397952
tensor i =  186 , k =  blk.8.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  459354112
tensor i =  187 , k =  blk.8.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  459362304
tensor i =  188 , k =  blk.8.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  459534336
tensor i =  189 , k =  blk.8.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  461336576
tensor i =  190 , k =  blk.8.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  462712832
tensor i =  191 , k =  blk.9.attn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  462938112
tensor i =  192 , k =  blk.9.ffn_down.weight , type =  11 , dims =  [5632, 2048] , tensorOffset =  462946304
tensor i =  193 , k =  blk.9.ffn_gate.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  467902464
tensor i =  194 , k =  blk.9.ffn_up.weight , type =  11 , dims =  [2048, 5632] , tensorOffset =  472858624
tensor i =  195 , k =  blk.9.ffn_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  477814784
tensor i =  196 , k =  blk.9.attn_k.weight , type =  10 , dims =  [2048, 256] , tensorOffset =  477822976
tensor i =  197 , k =  blk.9.attn_output.weight , type =  11 , dims =  [2048, 2048] , tensorOffset =  477995008
tensor i =  198 , k =  blk.9.attn_q.weight , type =  10 , dims =  [2048, 2048] , tensorOffset =  479797248
tensor i =  199 , k =  blk.9.attn_v.weight , type =  11 , dims =  [2048, 256] , tensorOffset =  481173504
tensor i =  200 , k =  output_norm.weight , type =  0 , dims =  [2048] , tensorOffset =  481398784
```

总结，逼逼这么多，还得是代码来的直接。