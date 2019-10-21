# A Reward Optimization Model for Decision-making under Budget Constraint

> 英文，翻译标题应该是【带预算约束并用于决策的回馈优化模型】

论文旨在设计一种能够通过受限的一系列数据学习到随机函数的新型预测模型。在建立泛化能力较强的用于进行函数预测的模型时，插值算法是一种较为常见的监督学习方法。然而，类似回归或者线性SVM这种被预定义的代数表达式限制的函数形式是不适合用于学习无限参数的模糊函数的。除此之外，虽然得到充分训练的神经网络能够计算出泛化能力好的函数，但是在一些实际的类似于在线推荐这种场景中，学习过程所需要的数据量将会大到超出想象。

提出模型能够解决以上两种问题，基于一种半参数的图模型以及贝叶斯估计来实现对小数据量进行估计参数的提取。同时存在一个在线函数来展示模型是如何在确定最佳全局函数时对决策过程造成影响的，决定一个好的全局函数也是在作出最佳决策时最重要的目标。

在模型评估上，会使用一系列采样方法来抽取训练数据，以展现使用提出方法训练出的优化推荐策略将会在什么程度上提高决策的正确率。经验评估总结得到结合`Thompson采样`改进的版本能够让提出算法训练出表现更好的策略。

关键字：增强学习，概率图模型，多臂抽奖问题，推荐系统

前置知识

- `MAB`, `CTR`
- `two-fold solution`
  - `graphical model`

> 看到p190右下角最后一段

## 1 Intro

> 前略
>
> This paper models this initilization problems（解决参数无限和需要样本量过大的问题） as reward optimization with exploration-exploitation trade-off under the paradigm of the multi-armed bandit problems, in which an agent always seeks for the candidate of the highest CTR`click through rate`

- 将`MAB`的用于寻找最高`CTR`的候选方法的代理作为参考对象（`paradigm`），提出模型能够使用`探索-采集`的权衡方法，把存在的初始化问题视为反馈优化问题进行建模。
- `MAB`不会考虑自己行为对环境的影响，但本模型的`online step`考虑了
- 预算的表现形式是一个总推荐次数的计数器
- 反馈模型表现形式是根据实际用户是否点击广告，从而体现的每次点击所收获的广告费用