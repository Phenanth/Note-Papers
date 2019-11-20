# 其他教授总结

> 暂时跳过跑题论文：7

## 发想

- 1
  - 需要看看结合的`Bayesian regularization`是什么，以及是怎么和神经网络结合的，我有多大可能可以借鉴这个结合的手法
  - 需要看看举例里面对`Bayesian regularization`（？可以包括神经网络）的描述，至少对于概率图的描述以及股票预测的重要性可以用于我描述**研究背景**的部分
- 2
  - 因为个人的方向是贝叶斯网络的应用，所以主要参考这篇`Bayesian Belief Network`的**研究手法**（第三章是介绍这个技术和第四章是这个技术的应用方法）【详细看第三章和第四章希望】，可以在背景研究里面提一下贝叶斯网络可以做的事情。
  - 这是讲述如何用数学语言描述商业收益（maeasuring bussiness performance）的一篇文章，如果想参考`BN`在分类器上的应用的话可以考虑看看这篇，但是不是很重要
  - 由于`scenario analysis`，情景分析这个词听起来挺好的，也经常用在金融方面，可以在**先行研究**里面提到一下，指的是通过考虑可能的替代结果（有时称为“替代性世界”）来分析未来可能发生的事件的过程（来自[Wiki](https://en.wikipedia.org/wiki/Scenario_analysis)）
- 3
- 6
  - 将BBN应用在渔业决策问题上，**先行研究**部分，又误打误撞和教授方向一样了
  - Intro的后面讲了BN的介绍，可以用在**研究背景**
  -  

## 1 BRANN用于SP 2013

> 先行研究SP，研究手法参考
>
> 在概要里选择了感觉比较有参考性的语句作为重点。
>
> 这篇虽然有`Bayesian regularization`作为添头，但是主体还是`aritificial neural network`，等会需要查查这是一个什么组合（`hybrid technique`）

概要：每日股市价格以及财经指数将经过优化，作为用于预测未来收市股票价格的输入数据。（原本是作为时序列数据进行分析，是投资者提高回报率的重要参考）预测这些趋势的复杂性在于每日股票价格走势中固有的噪声和波动性。**贝叶斯正则化网络为网络权重分配了概率性质，允许网络以最佳方式自动地约束网络的复杂性（对过于复杂的网络进行惩罚）**。在检验上，用了微软和Goldman Sachs Group两支股票来展示模型的准确性，**在拥有较好的准确性的同时还不需要对数据的预处理，季节性测试或者是周期性分析**。

> 同样是概率图模型，对比`Bayesian Network`存在自动处理网络概率权重的优势

介绍：...股票价格变动的复杂性实际上是由包括政治事件、市场资讯、季度收益报告以及交易行为冲突所构成的一系列因素决定的。但单单靠每日的股票数据想要进行日常或每周的市场动向预测还是很困难的。

实际结果表明，概率神经网络（分类模型）在预测股价趋势方面优于标准前馈神经网络（水平估计模型）。（主要讲述概率神经网络对于前馈神经网络的有点在于不会由于训练数据集而过拟合导致新数据的准确率下降问题）于是有学者提出了总和`Hidden Markov Model, Artificial Neural Networks以及Genetic Algorithm`来预测三支主流股票，获得了显著准确率提升。

> 本文也将主要讨论作为混合方案的一员，结合了`Bayesian Regularization`之后的`Artificial neural network`在股价预测上的表现如何。
>
> 第二节讨论了`Bayesian regularization`在处理经济上的时序列数据的应用潜力，可以在做这项技术的背景导读时了解一下

总结：模型使用了结合`Bayesian regularization`的`Levenberg-Marquardt`算法来预测股票价格。结果上减少了常规神经网络技术经常遇到的过拟合以及只能找到局部最小解的问题。由于引入了概率性质，网络避免了由于过量引入参数而提升的过拟合的风险。比起另一个混合神经网络模型，以及`fusion model & AR-IMA model`也有潜在优势。（可以了解一下这里指的是什么模型）

重要的是要注意，增加或替代其他技术指标可以在将来的应用中提高此模型的质量。（<u>什么技术指标？</u>）

> 这篇论文的研究由另外几个工作室支持，不知道有什么意思。

## 2 使用贝叶斯信念网络评估精益生产对业务绩效的影响 2015

> 先行研究BN（BBN），研究手法参考
>
> 首先要知道精益生产（lean manufacturing）是指：当用于食品时，术语“瘦肉”是指不存在脂肪含量。可以用几乎相同的方式来看待“精益”一词。精益企业从他们的组织中学习减少脂肪（浪费和最昂贵的活动），使他们只能以节省成本的方式执行纯净的生产过程。小型企业所有者可以应用各种各样的精益业务技术来增强其业务，并且精益业务方法学上仍有很大的创新空间。
>
> 主要在`scenario analysis`中使用`Naive Bayesian Belief Network`，用来评估在汽车供应链中使用不同精益工具带来的经济效益。

概要：针对汽车行业的供应商进行了案例研究，并研究了精益因子不同组合的场景。这项研究为将贝叶斯网络应用于业务绩效指标提供了新的愿景，同时考虑了变化下企业经营状况的有形和无形结果。

介绍：虽然精益生产被证明可以提高竞争性以及团队组织性，但是却没有人证明其经济效益。不仅如此，目前的情景分析主要关注的是对当前状态建模的准备，但不会根据做出选择而自动改变（猜测是这个意思）。而使用朴素贝叶斯的情景分析来进行成功、失败以及风险分析可能成为未来的研究方式中的一种，它同时也被视为是管理决策选择中用于链接两种定性方法的重要工具。p2左边诠述`BN`能够在位置环境使用受限的知识进行分析的能力。

论文希望能够衡量使用提高质与量的商业效果的技术后生产商获得的效果，论文目标有三个，使用贝叶斯信仰网络做到：

1. 评估联合在一起的对量与对质的精益因子
2. 解决语言评估中存在的依存关系问题（？）
3. 确认结合了精益技术后的经济与非经济收益以及变化情况下的持续能力

> 可能只有`Formation of the Bayesian Map`和`Preparing the dependency matrices`部分有用，其他的前面部分讲的是数据预处理，后面讲的是建立`scenario analysis`的基础，和贝叶斯网络在预测的应用上已经没什么关系了

总结：`Scenarios can be driven by events or problems that may arise
from a company’s portfolio of business lines and risks.`论文使用贝叶斯信仰网络来就经济效益角度比较了几种精益管理工具的实际效果。（不清楚网络里用到的精益因子是怎么分类的，据说它们和效益组成一起都是由专家用Delphi analysis决定的）问题是一方面缺少除了汽车供应商的其他厂家，另一方面无法获得精益收益相关的真实数据。

## 3 使用Dynamic 贝叶斯网络进行供应链诊断 2002

> 这篇应该是和2很像，晚点再看

## 4 环境建模中使用BN的优劣势

> 先行研究BN，不急

## 5 环境建模中使用贝叶斯网络

> 先行研究BN，不急

## 6 贝叶斯方法在股票评估模型不确定性中的应用

> 先行研究BN（BBN），研究手法参考
>
> 这个是一个BBN在决策问题上的应用（practical decision making），可以稍微优先看看
>
> 评价：感觉只有对BN的描述能用上，其他的部分研究手法差距太大了

概论：贝叶斯网络允许进行正式的决策分析，并且有助于模型不确定性的合并。在选择策略的时候，贝叶斯网络方法可以根据数据，提供一个平均表现最好的决策方法（省略了很多废话，应该是这个意思）这里主要讨论的是和渔业有关的内容

介绍：在许多情况下，与将不同结构模型纳入统计推断的影响相比，参数估计方法之间的差异微不足道。本文的一个目的是想要表明在假设的渔业系统中确认模型不确定性的重要性，另一个目的是描述使用平均模型作为决策过程中的辅助工具的重要性，即使不同的国家，不同的组织会推举不同的模型。

总结：尽管模型的参数也很重要，但是真正影响最大的应该是模型结构的不确定性。总之还是一个决策问题。



## 8 用Feature selection等另一种方法进行估价预测

> 叫板论文，可以晚点看看，先行研究SP（就别管研究手法了，技术都不同）



## 9 用BN进行交通流预测

> 先行研究BN

介绍：将相接的一些路口用贝叶斯网络进行建模。构建成的贝叶斯网络中的原因结点（优化后用作预测的数据）以及结果结点（需要预测的数据）之间的联合概率分布是由一个高斯混合模型（`Gaussian mixture model, GMM`）所描述的，这个高斯混合模型的参数经过`Competitive Expectation Maximization`算法估计。交通流的预测则是在经过`mmse, minimum mean square error`来演绎。该方法与许多现有的交通流量预测模型不同，因为它明确包含来自相邻道路链接的信息，且以统计方式分析当前路口的交通状态趋势。它也是对交通状态在数据不完整情况下做出的一种改进方法，这是一个创新点。

## 10 使用贝叶斯网络预测股票每日趋势

> 先行研究BN+SP，研究手法参考



## 11 使用动态贝叶斯因子图的股票市场动向预测

> 先行研究SP，研究手法参考



## 12 使用包括BN在内的多种方法评估生物产业收入管理方法

> 先行研究BN，研究手法参考
>
> 和2/3有点像，都是用来做管理的，用了很多方法所以可以把这些方法全都写在做决策的参考方法里



## 13 用BN预测破产

> 先行研究BN，研究手法参考



## 14 BN模型对portfolio risk and return建模

> 先行研究BN，值得一提portfolio risk是在第二篇论文的总结部分被提到过的，需要查一下这是个什么概念