# Neural Fictitious Self-Play on ELF Mini-RTS

前置知识：

- `Self-Play`：[Competitive Self Play](https://openai.com/blog/competitive-self-play/)，[Monte Carlo Neural Fictitious Self-Play](https://medium.com/syncedreview/new-method-applies-monte-carlo-neural-fictitious-self-play-to-texas-holdem-fdb700d52eab) 
  - `Q-Learning Network`：`NFSP`的基础
  - `Nash equilibrium`：[维基百科](https://en.wikipedia.org/wiki/Nash_equilibrium)，指互相合作的代理形成的总体局面下，任何一个代理独自行动都不会对总体带来增益之时的平衡。
- `policy gradient reinforcement learning`：（？，还没看）
  - `policy-based reinforcement learning`，[参考PDF](http://home.deib.polimi.it/restelli/MyWebSite/pdf/rl7.pdf)

## 0 概要

虽然AI在主机游戏上获取了较大成功，但是仍然不能打败`real-time trategy`游戏的人类冠军。其中一个原因是`RTS`游戏需要用多代理进行分析，它的环境模型不是一个静态的马尔可夫决策过程（`Markov Decision Process`），也就不能直接套用增强学习的单代理。在这篇论文中，我们将会尝试运用`Neural Fictitious Self-Play, NFSP`（一种寻找游戏理论的方法）在`Mini-RTS`（一个在`ELF`平台上的`RTS`游戏，小但足够复杂）这一游戏中寻找到纳什平衡（`Nash equilibria`），从而找到解决`RTS`这一类游戏的方案（`game-theoretic solution`）。

此外，研究还说明`NFSP`能够与方法梯度增强学习进行结合，在`Mini-RTS`上的表现良好。实验证明，使用方法梯度简单地自我运行对模型进行预训练，能够大幅度提高`NFSP`的性能可伸展性，尽管缺少理论上的可收敛的保证，但是它本身仍然提供了强有力的策略。（`which by itself gives a strong strategy despite its lack of theoretical guarantee of convergence.`）

> 查阅资料的时候发现`NFSP`基于的`Q-Learning Network`自身就有收敛速度慢的特点，所以一下子好像就理解概要最后说的缺少可收敛的保证是什么意思了...（就是真实计算的时候会很慢）
>
> 感觉做做这种游戏AI方面的研究也挺有趣的，如果这位教授也能多做点这样的内容就好了。
>
> 结合后面内容理解：用`PPO`（一种`policy-based RL`算法）训练结果体现对手策略对环境模型造成的影响，并给予这个变化的模型进行对`RL`网络训练数据的筛选。

## 1. Intro

> 研究领域：`RTS`、`Mini-RTS`
>
> 创新内容：
>
> - 第一次进入`RTS`游戏领域，研究如何找到纳什平衡的通用性解法
> - 添加了`policy gradient reinforcement learning`对`FP`模型进行预训练以帮助提高`NFSP`模型的性能（虽然理论上无法证明做出改变后的收敛性）

随着深度神经网络的崛起，增强学习在很多复杂环境中的表现越来越引人注目。在`Atari 2600`主机的大部分游戏上，使用深度增强学习方法的代理成功地完成了一些人类才能，或者是人类都做不到的任务。然而，在实时策略游戏（`RTS`）的领域中，AI仍然无法打败人类最顶尖的玩家，所以实时策略游戏也被认为是继围棋和国际象棋领域之后，又一大人工智能的挑战性命题。

为了解决这一问题，出现了不少专门研究`RTS`游戏的平台。其中的`ELF`平台具有可扩展性，轻量级，以及移植性好的特点，它也是为了增强学习研究而设计的。它提供了一款名为`Mini-RTS`的小型但一应俱全的`RTS`游戏，该游戏的运行速度比现有的`RTS`环境快一个数量级，同时捕获了`RTS`的所有基本机制，包括战争迷雾（`fog-of-war`）、收集资源、建造军营、使用军队发动攻击等。

此次工作中，我们希望能够从`Mini-RTS`中学到关于`RTS`游戏的理论性的解决方法，以此为后续解决更加真实且复杂的`RTS`游戏打下基础。**遇到的困难**包含：决定策略和战术，实时规划以及领域知识的利用（`domain knowledge exploitation`）。本文主要讨论的是**多代理**这一特性——因为`RTS`游戏是多代理的游戏，天然地失去了能够让代理学习的静态环境（这就导致**没有办法应用前人开发的，用于单代理增强学习的`Markov Decision Process, MDP`环境模型**）。

过去，`Heinrich, Lanctot, Silver`提出了一个游戏理论性的`Self-Play`方法，叫做`Fictitious Self-Play, FSP`，其代理能够根据其对手使用增强学习来计算最佳的相应策略，而且能够以基于样本的方式对这些策略取平均以得到最佳策略。这促成了`Fictitious Play, FP`在更多游戏上的运用，这些游戏往往都是一些环境信息并不明确的大规模游戏。

由于`FP`具有收敛到纳什均衡且具有最小限制的理论保证，因此`FSP`就有了可靠的收敛理论背景，并且比`FP`方法更可能收敛。 `NFSP`，`FSP`的一种变体，使用深度强化学习作为其最佳响应对象，在没有任何先前领域知识的小型扑克游戏中学习到了近似的纳什均衡。

在本文中，我们表明`NFSP`可以有效地与**政策梯度强化学习**（`policy gradient reinforcement learning`）相结合，并可用于`Mini-RTS`领域。 我们的实验结果还表明，通过使用政策梯度方法（`policy gradient strategy`）简单地预训练`FP`模型可以大大提高`NFSP`的可扩展性（尽管**缺乏理论上的收敛保证**，但它本身提供了强有力的策略）。 据我们所知，这是第一次尝试在复杂的`RTS`游戏中找到综合性的策略理论（`convergent strategy profile`），为寻找`RTS`游戏的纳什均衡策略研究领域指出了一个方向。

## 2 目标

### 2.1 `ELF`和`Mini-RTS`

我们的目标是计算一个在`ELF`平台中无法学习到的`Mini-RTS`游戏的均衡策略。 在`Mini-RTS`中，代理的目标是用其部队摧毁对手的基地。 每个代理都有对应的基地，单位和资源。 通过基地和一些资源，代理可以构建一个工作者。 一名工人可以建造一个营房和住在营房里的攻击者。
`ELF`游戏引擎是以`tick`时间单位驱动的：在每个`tick`过去后，每个代理做决定的形式是根据观察的内容给每个单元发送命令。 游戏状态根据命令而改变，并且新的观察结果会被给予代理。由于`Mini-RTS`中的战争迷雾与其他`RTS`游戏一样，因此代理人无法在战争中观察其对手的单位，因此信息是不完整的。 该游戏的屏幕截图如下图所示：

![1567339761622](C:\Users\SKHR\AppData\Roaming\Typora\typora-user-images\1567339761622.png)

除了每个单元的“向左移动两个像素”等低级的命令之外，`ELF`引擎还有更多的高级的战略命令，例如给所有单位下达“让某人去一些可用的地方并建造一个营房”或“从敌人的攻击中包围基地“之类的命令， 我们将会使用这些命令而不是原始的命令。 具体来说，代理商有九个独立的战略行动，其中四个是关于招募可使用单元的：工人，营房，近战攻击者和远程攻击者。 接下来的四个是关于战术命令：攻击，范围内战斗，攻击并撤退和完全防御。最后一个命令是空闲，这意味着什么都不做。 这些动作是全局的，即它们影响一个命令中的所有单元。 

代理每个`tick`会接受大小为22 * 20 * 20的低级的观察矩阵，其中20 * 20表示游戏地图观察的分辨率，22个频道包含每种单位的数量，如工人或基地等。

## 3 背景

### 3.1 马尔可夫决策过程以及增强学习

`MDP`是传统增强学习的环境模型。在通常带有`MDP`的增强学习中，代理与一个`MDP`环境$\epsron$交互。在每个步骤$t$时，代理将会接受一个状态（$s_t \in S$）且从一系列可能的动作集合$A$（带有一个概率集合$\pi ： S \times A \to [0, 1]$）中选择一个动作$a_t$，这个被选出来的动作就叫做方法（`policy`）。接下来这个动作会在环境中被执行，然后传回一个下一状态$s_{t+1}$以及所执行动作的反馈$r_{t+1}$。代理的任务则是让期待回馈值$E[R_t] = E[\sum ^ \infinite _{k=0} \gamma^k r_{t+k}]$达到最大，其中$\gamma$是折扣因子。

假如代理无法分辨环境中的一些变量，那么这个环境就叫作为部分可观测的`MDP`（`Partically Observable MDP, POMDP`）。在这样的环境$\epsron_p$中，代理将会观测到$o_t=O(s_t)$，其中$s_t$是一个环境的真实状态，而$O$是一个能够将状态映射为代理的观测结果的函数。就像`MDP`中的一样，代理将从$A$中选择一个动作$a_t$，但是如何做出此决策将取决于观测结果$o_t$而不是状态$s_t$，因为代理无法观测到真实的状态。

计算中，我们会用到经典理论中提到的`状态-动作`值函数$Q_{\pi}(s_t,a_t)=E[\sum^\infinite_{k=0}\gamma^kr_{t+k}]$、值函数$V_\pi(s_t)=\sum_{a\inA}\pi(a|s_t)Q_\pi(s_t,a)$，以及优势函数$A_\pi(s_t,a_t)=Q_\pi(s_t,a_t)-V_\pi(s_t))$。

### 3.2 展开形博弈

> [简单的科普](http://www.zwbk.org/MyLemmaShow.aspx?lid=274348#4)：展开形式的博弈又可译为扩展形式的博弈、扩展式赛局或扩展型赛局。正则形式的定义为数学家们提供了“均衡”（equilibria）问题的研究一个容易使用的表达式。因为它避免了怎么计算“策略”的问题，也就是说游戏是怎么进行的问题。若要考虑游戏是如何进行的，展开形式的博弈是一个比较方便的表达式。这个形式与组合博弈论关系密切。这个定义通过一个树的形式给定。在树的每一个节点（vertex），不同的参与者选择一个边（edge）。

此次研究中，我们认为`RTS`游戏就是一个满足线性多代理的展开形博弈论游戏。表示形式则是基于有限的单节点为根的博弈树。

在展开形博弈中，每个代理$i \n N$都有其无法辨别的状态。每个数据集$u_i \in U_i$都可能包含它们无法辨别的状态。由此，无法辨别同一个信息集$u_i$的状态$s_1$以及$s_2$的代理$i$会被强迫在遇到这两种不同的状态时采用同样的行动方式。如果代理在这场博弈中从不忘记它们获得的信息，那么这场博弈被叫做是完美回想的。在完美回想的游戏中，信息组成的图会形成树的形态。如果一个游戏只有两个代理，且它们学习到的状态$R_1+R_2=0$，那么游戏也叫做是双玩家零和博弈[^1]。

每个代理都有自己的策略集$\pi_i$，这个策略集是在给定状态$s$时对应的所有可能动作集合$A(s)$中特别选取的集合。策略档案（`strategy profile`）$\pi = {\pi_1, ..., \pi_N}$是所有代理的策略集形成的元组。从一个给定的策略档案，我们能推算出期待的积累回馈值$R(\pi)$。一个代理$i$的策略集如果是由对它的所有对手而言最佳的策略组成的，那么可以叫做是最佳响应策略集$\pi_i={\pi_j|j \nequals i}$，它能够将期待的回馈函数值（$R(·|\pi_{-i})$）最大化。展开形博弈中的纳什平衡则指的是存在一个策略档案，其中的所有策略集都由最佳响应策略集组成。

由此，我们决定将游戏理论模型与`MDP`结合。在展开形博弈中，如果我们选定一个代理，并修正除了它以外其他所有代理的策略集，那么环境就会被认为是对于这个被选定代理的单代理`POMDP`。此外，如果游戏是完美回想的，这个`POMDP`可以被转化为是`MDP`环境，因为获取到不可辨别的状态的**概率分布集**是静止的，所以这多个状态可以被认为是一个状态。

[^1]: 零和博弈：零和[博弈](https://baike.baidu.com/item/博弈)（zero-sum game），又称[零和游戏](https://baike.baidu.com/item/零和游戏)，与[非零和博弈](https://baike.baidu.com/item/非零和博弈/3956943)相对，是[博弈论](https://baike.baidu.com/item/博弈论/81545)的一个概念，属[非合作博弈](https://baike.baidu.com/item/非合作博弈/4277540)。指参与博弈的各方，在严格竞争下，一方的收益必然意味着另一方的损失，博弈各方的收益和损失相加总和永远为“零”，双方不存在合作的可能。[参考](https://baike.baidu.com/item/%E9%9B%B6%E5%92%8C%E5%8D%9A%E5%BC%88/3562463)。

### 3.3 神经虚拟自演绎（`Neural Fictitious Self-Play`）

前置：

- Anticipatory dynamics：让`NFSP`代理快速跟踪对手行为的变化
- Reservoir Replay Buffer：防止采样时间过久

`NFSP`是在拟合函数（`approximation functions`）上使用神经网络和`Deep-Q`网络的`FSP`变种。`FSP`则是使用`FP`，并在展开形博弈问题上的一种演化体（主要提高了可处理问题的规模）。

`FP`是一个通过学习得到的游戏理论模型，学习过程是代理不断有玩一个游戏，在每次迭代中选择能够最好对应它的对手们的平均策略的策略。在双玩家零和博弈或者`potential games`的游戏情况下，平均策略能够收敛至纳什平衡。

`FP`的形式则是一个比较传统的表现形式，每个代理在一局游戏中也只能行动一次，所以它并不适合用在大规模的应用上。为了突破这个限制，新提出了两种方法：`full-width extensive-form FP`以及`FSP`。两种方法都对表现形式做了一定扩展，前者是一个`full-width`方法，而后者则是较好地经过估算执行的方法（所以在规模变大的时候也能够相应跟随变化）。由于使用了`FP`，`FSP`代理将会不断地游玩一款游戏，将它们的经验存储下来。它们并不计算全长度的相应对策，而是利用增强学习来得到一个估计的最佳响应。而且它们也不再直接对全长度的策略取平均值，而是利用增强学习来得到一个估计的平均策略。

谈到`NFSP`，它不是简单地在`FSP`的基础上应用了`NN`，而是从一个回放池里面取出所有记住的经验（减少选取采样的等待时间）。`NFSP`还使用了动态预期让每个代理能够有效地跟踪它们的对手行为的变化。

> 翻译完了之后不小心被自己rewrite掉了，我自闭了
>
> 开局有一定概率（$\eta$和$1-\eta$）从`RL`和`SL`两种网络中选择一种策略，这样选择的策略叫做混合策略，公式也是两种网络的混合。
>
> 总之就是说代理有的两个网络和一个储存池（$M^{SL}_i$），监督学习网络（$\pi_i$）根据储存池里的以前游戏的观察结果和行为方式（下如果开局用的策略是从增强学习网络中选定的，那么这次游戏的行为和观察结果会作为元组存入储存池，作为之后监督学习网络的数据来源）计算概率分布，得到过去策略的估计平均策略，增强学习网络（$\beta_i$）根据上一次的平均策略（基于过去行动训练得到）选定行动内容。
>
> 所以每个代理都是从过去的对敌策略中得到一个平均策略作为最佳对策使用，所以这样的训练方式也叫`approximated FP`。

### 3.4 策略选定的优化

> 感觉这个就是创新点，让训练`RL`网络的数据更有对敌的针对性

用基于策略的训练方法`PPO`，比起基于值的`RL`（比如`DQN`），更适合学习行动方式。

`PPO`在`Trust Region Policy Optimization, TRPO`上有优化，本来的目标函数本来有一个`Kullback-Leibler 收敛`作为限制策略变化过大的限制器，但是这个方法不采用限制器而是新提出了一个函数$L(\theta)$（详细省略），是一个`clipping term`，需要在无限制条件下让这个新提出函数值取最大。

> 省略`PPO`公式介绍。见`pdf p3`。

## 4 算法

根据前面总结游戏特点：

- 游戏双方的单元都由各自代理控制，所以是一个双人零和博弈。
- 战时迷雾让游戏变为信息缺失博弈。
- 因为实际中的`RTS`几乎都使用时间步骤实现的，所以认为它是`extensive-form game`

本次工作中并没有实现对过去经验的记忆（否则就是完美回想的），在日后的工作中会重新讨论。

由于对手策略并不固定，没办法用基于值的`DQN`算法（一般这种算法里面，训练代理的数据采样方式并不依赖某种行为模式（`policy`），导致训练出的结果也没有针对性），就需要引入能够反映对手策略变化的环境变化规则。*在`FP`中运用循环重复缓存的前提假设是对手的训练速度要比提出的`RL`网络慢*

> 省略引入了`PPO`的`NFSP`算法的介绍

结果是因为样本会被算法筛选一次，所以样本效率会下降。（`pdf p4`右边第一段最后一句话没看懂。）

另外一件事情就是原本算法需要计算`SL`过去策略平均值和`RL`对敌最佳策略，并从两者之中做出最终的选择，意味着需要将数据分开送进训练网络，再计算查看哪一种性能更好，很浪费计算性能。提出算法不仅可以克服这一点，实现起来也很方便。（不计算使用哪种方法，而是分别地并行构建$N$个`ELF`（游戏引擎）进程）

> 感觉上面这段还是不够直观。
>
> 下面省略关于加入`PPO`的`RL`目标函数计算公式：由`PPO`、奖励探索行为的回馈值以及损失函数（来自`OpenAI Baselines`）构成。
>
> 继续省略`SL`目标函数公式：来自`SL`和`RL`的概率分布的交叉熵。

存储经验的过程则是能够保证：存储在存储池里的元组数据永远平均享有$\frac {N_{RRB}} {M_{RRB}}$的概率被选为采样数据。

同样也尝试了使用`PPO`作为`SL`网络算法的传统`NFSP`算法，只不过在其训练函数中完全忽略`SL_Actor`和`SL`训练算法，以及所有的代理行为选择时都将参考`RL_Actor`训练函数。（这两个函数见`pdf p4`）

## 5 实际运行

### 5.1 参数设置

> 见`pdf p5`，跳过，讲了同时运行的代理用的网络是共用的，同时有512游戏进程在运行，增强网络和监督学习网络用的`CNN`的设置，	激活函数参数设置，概率分布在经过激活后在全连接网络中经过处理，再经过`softmax`之后的输出，模型优化用的`clipping`方法（前面提到过）等等设置。

### 5.2 `SP`、`NFSP`以及`Mini-RTS`介绍

游戏包含两种模式：简单和突袭，人玩的胜率分别是90%和50%。

由于游戏特性，一般都能够计算纳什平衡。但是因为`Mini-RTS`的复杂性，我们只能用`Local Best Response Teqnique, LBR`来计算纳什平衡的近似下限（无法计算上限）。

此次工作中只基于和电脑对手的胜负情况进行纳什平衡的评估，每次评估最少经过1000次游戏。

实验总结：

- `PPO`的`SP`：简单模式下的最高胜率，突袭模式下远无法达到纳什平衡。用突袭模式训练的代理无法应对简单模式。
- `NFSP`：随着经历游戏把数，胜率稳步提升。虽然胜率会比相应的`NFSP`低，但是会得到不容易过拟合的更少开发的策略。
- 纯`SP`：在简单模式比`NFSP`好，在突袭模式则一样。训练过程比`NFSP`快。有无法收敛的风险但是在此次实验中没有体现出来。收敛了就一定是纳什平衡。

另外，不总结同时使用两种模式训练的情况。因为我们想要将另一种模式作为测试的时候的【全新对手】来对待。

最终训练结果，最好的代理也只有65%胜率。因为游戏一开始的资源都是随机的，而信息又不完整，不知道对手是否会进行攻击，那么如果代理一开始没有建造攻击者，它就无从进行防御。就连是使用了一百万局游戏进行训练的`PPO`代理，在面对与他是老对手的同一个电脑AI时，仍然会输掉29%的局数。

### 5.3 具体分析训练结果代理

> 描述了两个代理比较智能的场景，前两段没怎么看懂，贴在下面了
>
> We further analyze the acquired agents. To evaluate how exploitable
> the agent is, we train another `PPO` agent against the
> target agent. If the `PPO` algorithm converges to its optimal
> strategy, the win rate of the agent is equal to the exploitability
> of the target agent in imperfect recall settings.
> The results are shown in Table 1. Compared with other
> agents, the self-play agents are less exploitable, and do not
> lose over 50 percent against the `PPO` algorithm. This result
> suggests that the obtained agents are not exploitable by
> strategies that do not use the past histories of observations.

重点是此次的代理都是在无先验知识或者是预先注入了规则的`AI`帮助下完成的训练，所以能总结出来这些规则已经很厉害了。

### 5.4 使用纯`SF`对`NFSP`预训练

> `pdf p7`里面左边靠上有一部分应该是用来解释为什么可以预训练的。

结合`SF`训练快和`NFSP`能够收敛的特性。

一开始对`NFSP`的`RL`预训练，但是因为`RL`要给没被预训练过的`SL`提供预训练策略，实际的计算完全没有被加速。后来用`SP`同时对`SL`和`RL`进行训练，得到的结果如图`pdf p7`的描述和图。

结论是可以用效率高但是不稳定的`SP`进行预训练。

## 6 总结以及日后工作

贡献：

- `NFSP`可以和`policy-based reinforcement learning`结合，且可以用在`RTS`游戏上。
- 可以用`SF`对`NFSP`预训练以达到提高模型规模的目的
  - 训练高效
  - 保证收敛

日后工作：

- 为什么`PPO`的`SP`比`NFSP`胜率高

- 训练过程非完美回想，不符合`FSP`的要求（

可能会加入`recurrent neural networks`作为`RL`组件的控制器，同时为了配合，也给`SL`的储存池加入这样的机制）

> 感觉从见到训练效果不佳开始展开的一系列尝试可以作为参考：
>
> 1. 给目标网络（`NFSP`）构造一个网络作为对手（`SP`）
> 2. 给这个对手增加筹码，就是添加新的训练机制（`PPO`）
> 3. 见到目标网络效果不佳，综合训练网络二者（预训练）提出新模型
>
> 等等。