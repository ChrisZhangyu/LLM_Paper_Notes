# 目标
用小参数量加大量数据来逼近大参数量模型的性能，从而使得推理的成本降低，但是训练成本会增加。启发于  
Jordan Hoffmann 2022. Training compute-optimal
large language models.(这篇文章的观点是在成本一定的情况下，应该去使用更多的数据训练小参数的模型，而不是减少数据量去训练大参数的模型)
# 模型结构的更改
与经典transformer的不同点
* 对input进行归一化(RMSNorm)，传统是对output进行归一化
* SwiGLU激活代替了ReLU激活
* 旋转位置编码(苏剑林提出)，代替绝对位置编码。
# LlaMA Prompt格式
单轮对话
```python 
<s>[INST] <<SYS>>
{{ system_prompt }}
<</SYS>>
{{ user_message }} [/INST] 
```
* {{ system_prompt }}部分是整个对话中的通用前缀，一般用来给模型指定其角色，作为对话的大背景。
* {{ user_message }}部分是用户所提供的信息，可以理解为多轮对话中其中一轮对话的内容。

多伦对话 
```python
<s>[INST] <<SYS>>

You are are a helpful... bla bla.. assistant

<</SYS>>

Hi there! [/INST] Hello! How can I help you today? </s><s>[INST] What is a neutron star? [/INST] A neutron star is a ... </s><s> [INST] Okay cool, thank you! [/INST]
```  
* 每一组<s>和</s>之间是一个相对完整的单元，可以理解为一个对话轮次（如果直接给一个文本作为输入，也可以看到模型的输入结果分别是以这两个BOS和EOS token作为结尾的）
* [INST]和[/INST]用于区分在当前这一轮的对话（历史）中，用户输入的部分与模型返回的部分。位于[INST]之后，/[INST]之前的文本，是用户在这一轮次对话中所输入的query，而/[INST]之后的文本，是模型针对这一query所作出的回答。