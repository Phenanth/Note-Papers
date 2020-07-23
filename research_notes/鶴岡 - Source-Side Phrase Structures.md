# Source-Side Phrase Structures

> 省略关于模型评估的部分，`3`是提出的模型的重点部分。
>
> 

关键字：`Syntactic information`, `NMT Models`, `Machine Translation`

前置知识：

- `SMT`
- `Attention Mechanism`
- `Tree-structured recursive neural network, Pollack 1990`
- `Seq2Seq`
  - `Input-Feeding`（？）
  - 随机梯度下降
- `tree-to-sequence`
  - `RvNN`
  - `negative sampling: BlackOut sampling`
  - `Beam Search`（？）

总结：

提出了`tree-to-sequence NMT`，其中叙述了该新模型在中译日和英译日上的性能评估等。

- 核心的`tree-to-sequence NMT`编码器
	- 作者对`sequence-to-sequence NMT`的编码器作出改进，将输入从句子序列改成了文法树
	- 由于输入变化了，`attention`机制中的分数计算也要做出相应适配
- 对解码器的`Beam Search`的公式也做出更改，提高其在长句子翻译上的性能（作用存疑）

创新：

- 作者使用的`Tree-to-Sequence`的`Tree-LSTM`中，叶结点$h_k$的输入不是简单第$k$位单词的`word embedding`，而是用`seq2seq`的单向编码器生成的对应第$k$位单词第隐藏状态$h_k^`$，这是为了能够在`Tree LSTM`里面体现上下文信息。

## 0 Summary

`Neural Machine Translation, NMT`早期使用`seq2seq`，缺少对句子内部结构的分析，仅仅是将源词语向量序列转换为另外一个向量作为结果。

本篇文章中将会在`seq2seq`的基础上添加`Source-Side Phrase Structure`，更加关注输入句子的文法结构，并且提出一种`tree-to-sequence`的`NMT`模型。原理是在翻译过程中选择目标语言中与源语言定位更加接近的词来进行翻译（直译：使解码器在对齐，`align`，源语言用词的情况下生成翻译的词语）

操作中，在中转日与英转日的数据上通过与`seq2seq`模型对比的方式进行实践。结果显示文法结构的应用在数据集量小的时候效果明显，但是在使用`bi-directional encoder`[^1]的时候效率就不是那么令人满意了。而且训练数据量上升的情况下，文法树也开始失去它的优势。

[^1]: 关于双向编码器的解释，这里有个[Bi-Directional RNN](https://blog.csdn.net/antkillerfarm/article/details/77633782)

## 1 Intro

机器翻译的历史：在`Encoder-Decoder`模型中，编码和解码器都是RNN网络，编码器产出是一个定长的向量，解码器通过向量生成一系列目标词语。此后，这个模型被`attention`机制改进，使得其能够从源语言和目标语言中学习意义与定位相似的词语。近期，`NMT`模型被用于多种语言的翻译工作中。

本次实践中将会在`NMT`模型的基础上加入`Syntactic Information`，将源语言的词汇根据文法结构的构成生成一颗词汇树，根据词汇树的结点代表的词翻译为目标语言中相应的词语。

![image-20190817094157959](/Users/charlotte_chen/Library/Application Support/typora-user-images/image-20190817094157959.png)

支持运用文法结构进行机器翻译工作的模型原本是`Syntactical Machine Translation, SMT`，但上述的`NMT`模型并不能够借文法结构方法进行词汇选择，于是本文想要提出一种`Syntactical NMT, SNMT`模型，主要使用由底而上的递归方法，借助树形结构的`Recursive Neural Network`来生成句子向量。此外，还在`decoder`中用到了`Attention Mechanism`来有对齐地生成目标词汇。

本文是2016年的`tree-to-sequence NMT`在两种方法上的扩展，新增了中译日的任务，还发现了双向编码器运用在树形结构基础的编码器上，无论是英译日还是中译日上表现都有所提高。还分析了我们的模型，以及`NMT`模型在文法基础上或是序列基础上的不同。

文章的结构：

- 2 基础`seq2seq NMT`模型
- 3 作者定义的`tree-to-sequence NMT`模型
- 4 试验性设计
- 5 在小规模数据上执行英/中译日翻译，并分析所提出方法的关键组成内容
- 6 大规模数据上完成英译日工作的实践
- 7 总结`文法基础NMT`模型相关的近期研究
- 8 总结本次研究的贡献

## 2 Seq2seq NMT 模型

### 2.1 模型描述

模型使用`encoder`来通过输出句子的词向量生成向量空间，再用`decoder`通过向量空间生成输出句子，其中用到条件概率$P_\theta(y_j|y_{<j}, x)$。编码器通过${h^s_i}^\to={RNN}^\to_{enc}({h^s_{i-1}}^\to,Emb(x_i))$来计算第$i$个隐藏状态，其中${h^s_i}^\to \in R^{d \times 1}$，${h^s_{i-1}}^\to$是前一个隐藏状态，$Emb(x_i)$是第$i$个输入词$x_i$的`Word Embedding`概念下映射的向量。

相似地，类似上述的前向计算方法，还要计算后向的`RNN`单元，计算方法：${h^s_i}^\gets={RNN}^\gets_{enc}({h^s_{h+1}}^\gets,Emb(x_i))$。计算前向和后向的状态向量则是为了`bi-directional encoder`在运用时用到的源隐藏状态，由前后向的计算结果拼接成的$H_i=[{h^s_i}^\to,{h^s_i}^\gets]$。当编码器被用作实现基于前向`RNN`的编码器时，第$i$个前向隐藏状态${h^s_i}^\gets$就会被考虑为是第$i$个源隐藏状态$h^s_i$。

![image-20190818093450577](/Users/charlotte_chen/Library/Application Support/typora-user-images/image-20190818093450577.png)

这里的隐藏状态计算方法中的$RNN_{enc}$可以有多种选择，它也叫做`单元`。例如`GRUs, Gated Recurrent Units`、`LSTM, Long Short-Term Memory`

> 中间略去一部分

简单总结模型算法，假设为第$t$步时：

1. 训练隐藏状态（`bi-directional encoder`）
   1. 编码器的前向`RNN`按训练数据训练学习得到其隐藏状态$h^s_t$和记忆单元$c_t$
   2. 用`RNN`网络来计算解码器的隐藏状态$h^t_j=RNN_{dec}(h^t_{j-1},Emb(y_{j-1}))$：编码器的后向`RNN`模拟信息回传的过程，预测（计算）生成句子中$y-1$位置的隐藏状态$h^t_j$（后向的初始化根据前向最终结果来）
2. 从隐藏状态$h^s_t$和$h^t_j$中生成第$j$位的条件概率
   1. 解码器（用到`input-feeding method`[^2]）获取编码器的隐藏状态，根据源隐藏状态和注意力机制中的权重计算一个**上下文向量**$d_j \in R^{d \times 1}$，公式：$d_j = \sum^n_{i=1} \alpha_j(i)h^s_i$。其中，注意力机制的权重是根据后向预测目标状态$h^t_j$和前向的源状态向量$h^s_t$来计算的一个相似度得分，公式：$\alpha_j(i)=\frac {exp(h^s_i \cdot h^t_j )} {\sum^n_{k=1} exp(h^s_k \cdot h^t_j)}$。
   2. 准备一个新的隐藏状态${h^t_j}^`=tanh(W_2[h^t_j;d_j] + b_2)$用于计算$j$位输出的条件概率
   3. 计算$j$位输出的**条件概率**$P(y_j|y_{<j},x)$，其中用到`softmax`函数，如下：$P(y_j|y_{<j},x)=softmax(W_1{h^t_j}^` + b_1)$。（$W_1$、$b_1$等都是权重矩阵以及偏重向量之类的）
3. 根据目标函数（见下）对参数进行迭代调整

[^2]: 为了提高效率，作者将自己的解码器修正为$h^t_j=RNN_{dec}([h^t_{j-1};{h^t_{j-1}}^`],Emb(y_{j-1}))$，为了能够将新隐藏状态运用到`RNN`网络的隐藏状态中。

### 2.2 训练模型参数

参数$\theta$的训练数据：平行的句子级的集合$D_{train}={(x_1,y_1),…,(x_N,y_N)}$

模型的目标函数$J(\theta)=\frac 1 {|D|} \sum_{(x,y) \in D} logP(y|x) = \frac 1 {|D|} \sum_{(x,y) \in D} \sum^m_{j=1}logP(y_j|y_{<j},x)$，是翻译对的对数似然值的和。

另外，学习过程中，参数$\theta$的学习用到了随即梯度下降法。

## 3 Tree-to-Seq NMT模型

树结构基础的编码器是`sequence-based`编码器（使用`RNN`）和`binary tree-based`编码器（使用`RvNN`）的结合体。在使用`RNN-based`编码器通过时间序列计算得到单向信息时，它也会舍弃掉一些文法上的信息，这时通过`tree-based`编码器能够直接地让句子中的文法结构发挥作用，从而优化计算出的词向量。

![image-20190818093339317](/Users/charlotte_chen/Library/Application Support/typora-user-images/image-20190818093339317.png)

### 3.1  tree-based 编码器

步骤：

1. 使用一个额外的`parser`生成一个`binary parse tree`，其叶子结点为输入句子经过`sequence-based encoder`生成的单元，也就是$h_{leaf_i}=h^s_i$，代表第$i$个输入单词经过`seq-based encoder`生成的隐藏状态。
2. 从树的叶子结点开始，向上计算`k-th`隐藏状态$h^{(phr)}_k \in R^{d \times 1}$，公式：$h^{(phr)}_k=RvNN(h^{left}_k, h^{right}_k)$，$h^{left}_k$和$h^{right}_k$代表计算的结点的左子结点和右子结点。
3. 一直计算到$h^{(phr)}_{root}$根结点计算完成。

完成之后，会得到两种句子向量来表达输入句子：$h^{(phr)}_{root}$以及$h^s_n$。

> 实际情况中，作者使用`tree-LSTM`单元作为$RvNN$的选择。
>
> 关于这里使用`tree-LSTM`的计算过程，见`pdf p6`，概括就是计算：
>
> - 输出翻译器$o_j=\sigma(u_l^{(o)}h^{left}_k + u_r^{(o)}h^{right}_k + b^{(o)})$
> - 记忆单元的更新状态${c_j}^` \in R^{d \times 1} = tanh(u_l^{(c^`)}h^{left}_k + u^{(c^`)}_rh^{right}_k)+b^{(c^`)}$
> - $k$位的记忆单元状态$c^{(phr)}_k=i_k \odot c^`_k+f^{left}_k \odot c^{left}_k + f^{right}_k \odot c^{right}_k$
> - 以及$i_k$、$f^l_k$、$f^r_k$、等输入翻译器、左右子树的遗忘器，形式类似于$i_k=\sigma(u^{(i)}_lh^{left}_k + u_r^{(i)}h^{right}_k+b^{(i)})$
> - 进一步计算**第$k$位的词隐藏状态**$h^{(phr)}_k \in R^{d \times 1} = o_k \odot tanh(c^{(phr)}_k)$。
>
> 其中的$u^{(`)}_{(`)} \in R^{d \times d}$和$b^(`)\in R^{d \times 1}$分别是权重矩阵和偏重向量。

由于使用了`tree-based encoder`，它能够直接地根据词结点构建一个语法树，因此我们的模型应该能够比普通的`seq2seq`模型更加关注输入句子的文法结构，以此能够解决翻译工作中遇到的句意模糊的问题。

这里提议的`tree-based encoder`实际上是`sequential encoder`的一个自然的扩展，因为`Tree-LSTM`实际上是`链状结构的LSTM`的概括版本。但是和原始的`Tree-LSTM`不同，这里的叶结点（`LSTM单元`）首先通过单向编码器进行编码，然后使用该编码结果作为`Tree-based encoder`所使用的输入词。

这样做的理由是想要将词结点构造为对上下文敏感的形式，希望模型能够在即使是同一个词的情况下，只要该词在句子中的表示不同[^3]，模型就能够处理这之间的差异。而对比传统的`Tree-LSTM`单元，叶结点只是单纯的词嵌入，不包含任何上下文信息。

[^3]: 由于是顺序模型，出现位置不同就代表上下文不同

### 3.2 使用 tree-based 编码器的解码

这里会用到上面`3.1小节`计算的$h^s_n$以及$h^{(phr)}_{root}$两种不同的隐藏状态。

解码器的隐藏状态初始化为$h^t_1$，其公式使用了与`3.1小节`中不同的`Tree-LSTM`函数，公式：$h^t_1=TreeLSTM(h^s_n,h^{(phr)}_{root})$

> 附加说明，如果在第一步的预处理阶段，句子不能够被`parse`为一个树形结构，算法只会将句子转换由顺序隐藏状态$h^s_n$组成的结点集合，而树形隐藏状态会被设置为`0向量`。树形编码器只会在句子能够被`parse`的情况下工作，如果失败了，`3.1小节`研究的编码器的工作方式与`sequence-based encoder`相同。

对$TreeLSTM$的解释：为了在模型变动后依然利用注释2[^2]提到的解码器原理获取第$j$位的隐藏状态，计算方法需要参考`2.1小节`，作出如下改动：

1. 引入新的注意力得分$a^{(tree)}_j(i)=\frac {exp(h_i \cdot h^t_j)} {\sum^{2n-1}_{l=1}exp(h_l \cdot h^t_j)}$，初始化值$h^t_1$如上。
2. 计算新的上下文向量$d^{(tree)}_j=\sum^n_{i=1}\alpha^{(tree)}_j(i)h^{(leaf)}_i+\sum^{2n-1}_{i=n+1}\alpha^{(tree)}_j(i)h^{(phr)}_i$
3. 利用这些新的计算结果，使用相同的方法计算第$j$位的条件概率$P(y_j|y_{<j},x)=softmax(W_1{h^t_j}^` + b_1)$，${h^t_j}^`=tanh(W_2[h^t_j;d_j] + b_2)$，$d_j$将会被替换成$d^{(tree)}_j$。

新的注意力得分能够同时接触到混合模型中的顺序模型和树形模型的隐藏状态单元，使得`tree-to-sequence`模型的单元学习结果很大可能性上取决于训练过程。

### 3.3 基于负采样的NMT模型评估（`Blackout sampling`原理部分看不懂）

为了解决计算条件概率中用到的`softmax`效率低[^4]的问题，提出了`negative sampling`解决方法，其中包括：

- `BlackOut sampling`【更加适合`RNN`的语言模型】
- `noise-contrastive estimation, NCE`
- `binary code prediction`

`2.2小节`提出的目标函数是经典的基于`RNN`的语言模型——`NMT`的代表，于是在加入`BlackOut sampling`后，我们定义目标函数为：$J(\theta)=\frac 1 {|D|} \sum_{(x,y)\in D} \sum_{j=1}^{|y|} (\log{p^`(y_j|y_{<j},x) + \sum_{k \in S^K}(1-\log{p^`(y^k|y_{<j},x)})})$，其中$p^`(y_j|y_{<j},x)=\frac {q_j \exp(s^j_j)} {q_j \exp{s^j_j} + \sum_{k\in S^K} (q^k \exp{s^k_j})}$，$s^k_j=w_k{h^t_j}^`+b_k$（取自上方`softmax`方法中的$W_1$以及$b_1$，是第$k$行或者第$k$个元素）。

> 其他关于此采样方法的参数介绍见`pdf p8`，有关于负采样获得的采样数据子集$S^K$、在使用$K$个负采样样本时，大数据集中在第$j$步时第$k$位的单词（向量${h^t_j}^`$）使用带权`softmax`的得分$s^k_j$等。

[^4]: 根据训练单词表大小线性变化

## 4 试验性设计（未开始）

### 4.1 试验性设定

中译日：

数据集用`ASPEC`，使用不同的分词器对中日两种语言进行分词，舍弃两方中词汇量大于50的句子对，对中文句子用`Stanford Parser - Chinese-Factored`额外进行一次`parse`（但是并不使用它提供句子标签的功能）来形成语法树，且如果失败的话就用简单的概率语法模型`PCFG`（不包括上下文）进行`parse`。此后将上文语法树转换为二叉树（因为`NMT`模型输入是二叉语法树）。

统计处理后的句子中出现单词的次数，中文大于三，日文大于二就保留，结果的中文词集合大小为29011，日语为32640。另外不存在的词语映射为`unk`，句子结尾插入特殊符号`EOS`。

英译日：

省略，其中谈到了不一样的参数设置：关于词嵌入的维数，`512-dimensional word embeddings`以及`d-dimensional hidden units`，而且当利用波束搜索在测试时生成源句子的目标句子时，作者选择了在开发数据集上找到的最佳波束宽度。（？）

评估方法用的是两种自动评估：`RIBES`和`BLEU`。`RIBES`基于词语之间的关联因数的等级给词语准确度打分，而且公认在英语和日语的评估上比`BLEU`更好；而`BLEU`基于`n-gram`词准确性和给比参照短的输出进行简易惩罚。（？）

###  4.2 参数设定

> 参考了很多论文，这一部分看`pdf p10`

### 4.3 使用Beam Search的解码以及过长惩罚得分

> 关于[Beam Search](https://blog.csdn.net/batuwuhanpei/article/details/64162331)

略去`Beam Score`的计算。总之是用在解码器生成词序列的时候用的。

目标句子越长，普通的`Beam Search`就越没什么优势，使得解码`NMT`模型的难度也越高。

于是首先用处理好的英语和中文数据中的句子计算一个长度相关的传统概率$Length_{x,y}=\log{p(length(y)|length(x))}$，优化`Beam Score`公式为 ：$score(x,y)=Length_{x,y}+\sum ^m_{j=1}\log{p(y_j|y_{<j},x)}$，只要模型预测这里有一个`EOS`符号，这个长度概率就会被加上。

于是，模型就能够在统计学的角度，在句子较长的情况下综合考虑输入句子的长度来生成目标句子。

> 略去参数设置

## 8 总结

此次研究的关注点：

- 在训练结点中用到的`attention mechanism`能够在选择翻译词语的时候尽量选择在措词上更加接近源语言中词汇的目标语言词语
- 对`sequence-to-sequence NMT`模型在文法结构上做出了扩展，提出`tree-to-sequence NMT`模型
- 在关注`source-side phrase structures`的基础上，构建了一个基于语法树的编码器，是顺序编码器的文法结构扩展版本，所以这个编码器的叶子结点也能够与顺序模型共同工作

研究的结论：

- 双向编码器能够同时提高英译日和中译日的准确性和评估算法得分
- 添加语法树在英译日任务上的性能提高更明显，但是一旦训练数据集的数据量变大，`tree-to-sequence NMT`模型在`WAT 2015 English-to-Japanese`上的提升量开始下降

可以从`tree-to-sequence`模型的分析揭示`sequence-to-sequence`模型的不同趋势，还能够明白新提出的模型可以灵活地学习哪些措辞或短语。