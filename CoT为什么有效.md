# Towards Revealing the Mystery behind Chain of Thought: A Theoretical Perspective
从计算复杂度理论分析了为什么CoT能够激发Transformer模型的能力。提出了两个问题：
1. 一次性生成的方法是否有隐含的缺陷?
2. COT为什么能够使模型的能力有很大提升?

## 背景知识
### 布尔电路理论
 布尔电路是一种有向无环图，每个结点代表一种比特，每个节点的入度称为扇入数，深度即初始结点到最终结点的路径最大值。一个布尔电路只能模拟一个固定输入bit的计算问题，当输入bit变化时，就需要多个布尔电路。计算复杂度理论主要研究的是对于给定计算问题，布尔电路的大小是如何随着输入bit而增加的，或者说给定布尔电路，什么样复杂度的问题能够被该布尔电路计算。
### 计算复杂度理论
计算复杂度理论为问题的计算复杂度分了几种类别，每一类别都有典型问题。  
电路计算复杂度理论类别(参考书籍: Computational Complexity: A Modern Approach):
   * $`P$ 类问题，能够 $O(poly(n))$ 复杂度内由图灵机解决的问题。
   * $NC^i$类问题，能够被深度为 $O(log^in)$ 、扇入c的布尔电路解决的判定问题集合，要求c是常数，不随着问题的规模n增长。布尔电路的运算集合只有{AND,OR,NOT}
   * $AC^i$类问题，能够被深度为 $O(log^in)$ 、扇入为 $O(poly(n))$ 的布尔电路解决的判定问题集合。布尔电路的运算集合只有{AND,OR,NOT}
   * $TC^i$类问题，能够被深度为 $O(log^in)$ 、扇入为 $O(poly(n))$ 的布尔电路解决的判定问题集合。布尔电路的运算集合有{AND,OR,NOT,MAJ}，MAJ为major逻辑运算，即返回众数。例如：MAJ(10001) = 0, MAJ(1111) = 1。如果0,1的个数相同则随机返回。
### 问题的完全性   
完全性是指在某一复杂度类别中的其他问题都能在多项式时间内$O(poly(n))$化简成该完全问题，也即该问题一定程度上代表了该复杂度类别的特性，例如经典的NP完全问题：给定一个有限数量的整数集合，找出任何一个此集合的非空子集且此子集内整数和为零。，即代表了NP复杂度下问题的特性。  
### 数学问题
数值计算  
![avatar](images/数学问题1.jpg)  
所有的计算表达式都可以由一颗二叉树表示，即每个运算符作为根节点，数值作为两个子节点,(token1 a op b token2)通过观察token1和token2来决定a和b是否能够进行运算。因此树高为 $logn$。  
规则：
* op $\in$ { $+$, $-$} 且 token1 $\in$ {(, empty}, token2 $\notin$ { $\times$, $\div$}
* op $\in$ { $\times$, $\div$} 且 token1 $\notin$ { $\times$, $\div$}  
 
通过每次化简一个操作符的方式来最终得到计算结果

解线性方程组  
![avatar](images/数学问题2.jpg)  
利用高斯消元法解线性方程组，即每次将一个变量用其他变量表示，而后将该变量带入其他表达式中。
## 为什么Transformer不能解决数值计算和解线性方程的问题
有限深度、对数精度的Transformer模型的计算复杂类型是 $TC^0$（来源于The parallelism tradeoff: Limitations of log-precision
transformers），而数值计算问题的复杂度是 $NC^1$的，因此低计算复杂度的模型无法直接解决高计算复杂度的问题。
## 证明
Transformer的计算复杂类型已经在The parallelism tradeoff: Limitations of log-precision transformers证明过了，现在只需要证明数值计算问题是 $NC^1$的复杂度即可。  
布尔公式求值是典型的 $NC^1$完全问题，因此只需要将布尔公式求值问题化简为数值计算问题，即可证明数值计算的复杂度在  $NC^1$之上。那么属于 $TC^0$复杂度的计算模型有限深度、对数精度的Transformer就无法解决这个问题。
> 证明过程：布尔公式求值问题的定义，所有字符来自于字符表 $\{0,1, \vee, \wedge, \neg, ), (, \}$，布尔公式由如下规则产生:
>  * 0 和 1 本身可以作为布尔公式
>  * 如果 $\varphi$是布尔公式那么，$\neg\varphi$也是布尔公式
>  * 如果 $\varphi_1, \varphi_2$是布尔公式那么， $\varphi_1 \vee \varphi_2$ 和 $\varphi_1 \wedge \varphi_2$也是布尔公式
> 
> 将布尔公式求值问题规约为计算问题:
> * $f(0)=0$ 且， $f(1)=1$
> * 对于所有布尔表达式 $\varphi$， $f(\neg\varphi)=(1-\varphi)$
> * 对于所有布尔表达式 $\varphi_1, \varphi_2$， $f(\varphi_1 \wedge  \varphi_2)=f(\varphi_1) \times f(\varphi_2)$
> * 对于所有布尔表达式 $\varphi_1, \varphi_2$， $f(\varphi_1 \vee \varphi_2)=(1 -  (1 - f(\varphi_1)) \times f(1 - f(\varphi_2)))$ 
>
> 例子：  
> &emsp; 假设有如下表达式: $(1 \vee \neg0) \wedge (1 \wedge \neg1)$将其转换为数值表达式
> 1. $f(1 \vee \neg0) \times f(1 \wedge \neg1)$
> 2. $[1 - (1 - f(1)) \times (1 - f(\neg0))] \times [f(1) \times f(\neg1)]$
> 3. $f(1) \times f(0) \times (1 \times 0)$
> 4. $=1 \times 0 \times 1 \times 0 = 0$
 
## 为什么CoT有效
CoT的方式提高了Transformer模型的计算复杂度，将本身有限深度、对数精度的Transformer模型从 $TC^0$的复杂度变成，$NC^1$的复杂度。简要说明：  
一次生成的方式，Transformer的深度为一个常数d， 可以表达为 $O(1=log^0n)$，那么其扇入为 $O(poly(n))$，$W_Q \in \mathbb{R}^{d_{emb} \times d_{qk}}$，$W_K \in \mathbb{R}^{d_{emb} \times d_{qk}}$，$W_V \in \mathbb{R}^{d_{emb} \times d_{v}}$，$X \in \mathbb{R}^{n \times d_{emb}}$，$(XW_Q)(XW_K)^TXW_V = (QK^T)V \in \mathbb{R}^{n \times d_v}$，即当前Transformer的复杂度为 $TC^0$。CoT通过改变模型解决问题的过程，隐含的改变了有效深度，因为分为几步解决完全由模型决定，每一步都可以视为模型对数值计算问题的子问题进行了计算，因此可以看做进行了 $logn$次子计算（分解成了二叉树，树高为 $logn$。[见数学问题](#数学问题)），宏观上Transformer就变成了一个深度为 $logn$，扇入为$O(poly(n))$的计算模型，其计算复杂度变成了 $NC^1$，因此能够解决复杂度为 $NC^1$的数值计算问题。
