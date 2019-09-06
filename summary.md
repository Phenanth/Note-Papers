# Paper Summary Feat. Tsuruoka

## Reinforcement Learning

> 翻译：使用词预测对运用增强学习的句子生成器加速，[强化学习](https://blog.csdn.net/coffee_cream/article/details/57085729)

这里只贴总结，有空应该会把完整笔记补上，标题和论文的解读应该是这样：

1. 运用`Reinforcement Learning，下记RL`来进行`Sentence Generation`，其中包含状态转换和动作选择的迭代。
2. 发现和`Cross Entrophy`训练方法相比，`RL`的训练效率太低。
3. 加入`Vocabulary Prediction`，在句子长度$K$和训练数据集合$V$的基础上训练参数，这些参数将用于生成一个较小的$V'$，这样可以起到对`RL`加速的效果。

## Source-Side Phrase Structures

> 翻译：使用运用源侧短语结构的神经网络机器学习进行翻译，神经网络机器学习，`NMT`，[地址](https://blog.csdn.net/antkillerfarm/article/details/77633782)
>
> 还没认真看，感觉这个文章的前后叙述有点不一致。

关键字：`Syntactic information`, `NMT Models`, `Machine Translation`

本文是2016年的`tree-to-sequence NMT`在两种方法上的扩展，在`seq2seq`的基础上添加`Source-Side Phrase Structure`，更加关注输入句子的文法结构，在翻译过程中选择目标语言中与源语言定位更加接近的词来进行翻译。

支持运用文法结构进行机器翻译工作的模型原本是`Syntactical Machine Translation, SMT`，但上述的`NMT`模型并不能够借文法结构方法进行词汇选择，于是本文想要提出一种`Syntactical NMT, SNMT`模型，主要使用由底而上的递归方法，借助树形结构的`Recursive Neural Network`来生成句子向量。此外，还在`decoder`中用到了`Attention Mechanism`来有对齐地生成目标词汇。

操作中，在中转日与英转日的数据上通过与`seq2seq`模型对比的方式进行实践。结果显示文法结构的应用在数据集量小的时候效果明显，但是在使用`bi-directional encoder`的时候效率就不是那么令人满意了。而且训练数据量上升的情况下，文法树也开始失去它的优势。

### 研究的关注点

- 在训练结点中用到的`attention mechanism`能够在选择翻译词语的时候尽量选择在措词上更加接近源语言中词汇的目标语言词语
- 对`sequence-to-sequence NMT`模型在文法结构上做出了扩展，提出`tree-to-sequence NMT`模型
- 在关注`source-side phrase structures`的基础上，构建了一个基于语法树的编码器，是顺序编码器的文法结构扩展版本，所以这个编码器的叶子结点也能够与顺序模型共同工作

### 研究内容

论文中，作者提出了`tree-to-sequence NMT`，其中叙述了该新模型在中译日和英译日上的性能评估等。

- 核心的`tree-to-sequence NMT`编码器
  - 作者对`sequence-to-sequence NMT`的编码器作出改进，将输入从句子序列改成了文法树
    - 作者使用的`Tree-to-Sequence`的`Tree-LSTM`中，叶结点$h_k$的输入不是简单第$k$位单词的`word embedding`，而是用`seq2seq`的单向编码器生成的对应第$k$位单词第隐藏状态$h_k^`$，这是为了能够在`Tree LSTM`里面体现上下文信息。
  - 由于输入变化了，`attention`机制中的分数计算也要做出相应适配
- 对解码器的`Beam Search`的公式也做出更改，提高其在长句子翻译上的性能（作用存疑）

### 研究结论

- 双向编码器能够同时提高英译日和中译日的准确性和评估算法得分
- 添加语法树在英译日任务上的性能提高更明显，但是一旦训练数据集的数据量变大，`tree-to-sequence NMT`模型在`WAT 2015 English-to-Japanese`上的提升量开始下降

综上所述：可以从`tree-to-sequence`模型的分析揭示`sequence-to-sequence`模型的不同趋势，还能够明白新提出的模型可以灵活地学习哪些措辞或短语。

## Task-Specific Reinforcement Learning

> 简单贴一下总结的翻译，具体看对应的md文件。
>
> 为了让跟随训练不断变化的模型将自己对环境造成的影响也看作是训练过程中观测数据的组成部分，作者引入了`CNN`和`LSTM`网络的交叉训练，弱化模型参数对自身模型的影响，并放大其对另外一个模型的影响。
>
> 但是问题也随之而来，这样又可能造成`LSTM`网络的隐藏层参数$h$对自身模型的拟合不足，导致在`LSTM`预测下一隐藏状态的表现有所下降。为此，作者又提出了改进后的`LSTM`网络并称它为`*full model*`预测器，虽然对原始提出模型的提升并不大。
>
> 这一篇研究用到很多`I2A`的内容，但是基本上还是一个基于模型的增强学习，其中用到的学习数据是按时序排序的（推箱子的步骤），所以日后作者想要处理随即状态转换。

此次提出的基于模型的增强学习框架的重点在于：**能够让模型在适合于任务实现的低维度表示的基础上进行下一状态的预测**，这对基于得到的表示的环境状态转换模型、基于获取到状态模型的策略的获取都很有帮助。

以及很重要的一点，即使与复杂的模型结合在一起，我**们的架构也能够帮助压缩预测时间**，将计算时间控制在理想范围内。

未来工作内容：可能会通过结合`MDN-RNN`，来进行随即状态转换的计算。本文中没有提到的使用一个网络来预测奖励的机制也有可能在日后工作中大放光彩。

## Neural Fictitious Self-Play on ELF Mini-RTS

- `NFSP`可以和`policy-based reinforcement learning`结合，且可以用在`RTS`游戏上。
- 可以用`SF`对`NFSP`预训练以达到提高模型规模的目的
  - 训练高效
  - 保证收敛

> 感觉从见到训练效果不佳开始展开的一系列尝试可以作为参考：
>
> 1. 给目标网络（`NFSP`）构造一个网络作为对手（`SP`）
> 2. 给这个对手增加筹码，就是添加新的训练机制（`PPO`）
> 3. 见到目标网络效果不佳，综合训练网络二者（预训练）提出新模型
>
> 等等。