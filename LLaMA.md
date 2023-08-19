# 目标
用小参数量加大量数据来逼近大参数量模型的性能，从而使得推理的成本降低，但是训练成本会增加。启发于  
Jordan Hoffmann 2022. Training compute-optimal
large language models.(这篇文章的观点是在成本一定的情况下，应该去使用更多的数据训练小参数的模型，而不是减少数据量去训练大参数的模型)
# 模型结构的更改
与经典transformer的不同点
* 对input进行归一化(RMSNorm)，传统是对output进行归一化
* SwiGLU激活代替了ReLU激活
* 旋转位置编码(苏剑林提出)，代替绝对位置编码。