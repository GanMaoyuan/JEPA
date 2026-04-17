#### [《A Path Towards Autonomous Machine Intelligence Version 0.9.2, 2022-06-27》](https://openreview.net/pdf?id=BZ5a1r-kVsf)论文阅读笔记
<br><br><br><br><br><br><br>

# 模式1

## 1、世界状态 ![](https://latex.codecogs.com/svg.latex?s[0]=\text{Enc}(x))

将智能体所处的“当前时刻”定义为 ![](https://latex.codecogs.com/svg.latex?t=0) ，区别于 ![](https://latex.codecogs.com/svg.latex?t=1,t=2) 等“未来时刻”。<br>
智能体（agent）的感知模块（perception module）通过传感器（sensor，例如摄像头）接收到来自所处物理世界（physical world）的信号（signal），即 ![](https://latex.codecogs.com/svg.latex?x[0])（方括号内的 ![](https://latex.codecogs.com/svg.latex?0) 指示“当前时刻”，在后文中，![](https://latex.codecogs.com/svg.latex?x[0]) 简记为 ![](https://latex.codecogs.com/svg.latex?x) ），例如由十的若干次方数量级的大量像素通道值所表示的一张高清实时帧（可记为 ![](https://latex.codecogs.com/svg.latex?x\in\mathbb{R}^d) ），并通过编码器（Encoder）对其进行处理，从而提取出智能体自己对自己所处物理世界的状态（world state）的感知（perception，带有主观性，区别于信号 ![](https://latex.codecogs.com/svg.latex?x) 的客观性，主观性意味着不可完全克服的“认知偏差”）的表征，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s[0]=\text{Enc}(x).">
</p>

这种表征通常是任务情景化的，同一物理实体的感知表征在不同任务情景中往往是不同的。<br>
假如智能体处于“组装/修理/建造”这一类特定任务情景，那么一把锤子会被它感知表征为“可能有用的工具”，而在其他那些与“组装/修理/建造”关联性较弱的任务情景中，这把锤子会被感知表征为“无关紧要的东西”。<br>
从原论文作者Yann LeCun的立场来看，针对感知模块编码器的这种“任务情景功能化配置”（contextually configuring）行为是由智能体的配置器（configurator）完成的。

## 2、即时动作 ![](https://latex.codecogs.com/svg.latex?a[0\right]=\text{A}(s[0\right]\right))

智能体的执行模块（Actor module）的策略子模块（policy sub-module）接收 ![](https://latex.codecogs.com/svg.latex?s[0]) 作为输入，直接计算并输出动作（action），即 ![](https://latex.codecogs.com/svg.latex?a[0]=\text{A}(s[0])) 。输出的动作称为“动作提议”（action proposal），它将被随即发送到智能体的效应模块（effector module，例如机械臂、机械足等）并被实际执行。<br>
该过程就相当于一个直接的函数映射，期间没有任何其他模块的参与，因而表现出反应上的迅速性，类似于人类的“条件反射”或者“肌肉记忆”。<br>
和感知模块的编码器 ![](https://latex.codecogs.com/svg.latex?\text{Enc}(\cdot)) 一样，策略子模块 ![](https://latex.codecogs.com/svg.latex?\text{A}(\cdot)) 也会接受配置器的任务情景功能化配置。

## 3、世界状态预测 ![](https://latex.codecogs.com/svg.latex?s[1]=\text{Pred}(s[0],a[0]))

智能体的世界模型模块（world model module）通过预测器（Predictor）接收 ![](https://latex.codecogs.com/svg.latex?s[0],a[0]) 作为输入，计算并输出

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s[1]=\text{Pred}(s[0],a[0]),">
</p>

作为对未来时刻 ![](https://latex.codecogs.com/svg.latex?t=1) 的世界状态的预测（prediction），或者说“世界状态演化的模拟结果”。

## 4、来自物理世界的信号的更新 ![](https://latex.codecogs.com/svg.latex?x^\prime=\text{World}(x,a[0]))

智能体在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻的动作 ![](https://latex.codecogs.com/svg.latex?a[0]) 已经做出，对其所处的客观物理世界已经产生了不可逆的影响，客观物理世界已经不可逆地发生了变化。这就意味着，智能体在 ![](https://latex.codecogs.com/svg.latex?t=1) 时刻所接收到的来自所处物理世界的信号并不会保持为 ![](https://latex.codecogs.com/svg.latex?x) 不变，而是会更新为与之不同的 ![](https://latex.codecogs.com/svg.latex?x[1]) ，后文中简记为 ![](https://latex.codecogs.com/svg.latex?x^\prime) 。<br>
尽管 ![](https://latex.codecogs.com/svg.latex?x^\prime) 与 ![](https://latex.codecogs.com/svg.latex?x) 之间在尺度上的差异（即各分量取值上的差异）有可能较为微小，但它们在本质上有着明确的时序先后关系，在本质上是不同的。<br>
![](https://latex.codecogs.com/svg.latex?x^\prime) 与 ![](https://latex.codecogs.com/svg.latex?x) 之间的时序先后关系可以用一个函数映射来表示，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?x^\prime=\text{World}(x,a[0]).">
</p>

应该特别注意，![](https://latex.codecogs.com/svg.latex?\text{World}(\cdot)) 属于客观物理世界演化的范畴，并非发生于智能体内部，并不属于其内部的任何模块。对智能体而言，它根本不具备解析形式，它对任何变量的梯度根本不可解，即：![](https://latex.codecogs.com/svg.latex?\text{World}(\cdot)) 对智能体而言根本不可微分（non-differentiable）。

## 5、代价 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]=\text{C}(s^\prime[1])) 、梯度 ![](https://latex.codecogs.com/svg.latex?\frac{\partial{}f^\prime[1]}{\partial{}a[0]}) 以及模式1的本质缺陷

### 5.1、代价 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]=\text{C}(s^\prime[1]))

在 ![](https://latex.codecogs.com/svg.latex?t=1) 时刻（区别于 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻），智能体通过传感器接收到来自所处物理世界的信号 ![](https://latex.codecogs.com/svg.latex?x^\prime) ，通过编码器对其进行处理，从而提取出智能体自己对自己所处物理世界的状态感知的表征，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s^\prime[1]=\text{Enc}(x^\prime),">
</p>

应该特别注意将其与 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻的预测

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s[1]=\text{Pred}(s[0],a[0])">
</p>

作区分。<br>
智能体的代价模块（Cost module）的内在代价子模块（Intrinsic Cost sub-module）接收 ![](https://latex.codecogs.com/svg.latex?s^\prime[1]) 作为输入，计算并输出

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?f^\prime[1]=\text{C}(s^\prime[1]),">
</p>

它的内涵可以表述为：<br>
智能体在 ![](https://latex.codecogs.com/svg.latex?t=1) 时刻对世界状态的主观感知 ![](https://latex.codecogs.com/svg.latex?s^\prime[1]) 的“不舒适度”（discomfort），或者智能体在 ![](https://latex.codecogs.com/svg.latex?t=1) 时刻对世界状态所具备的“自由能”（free energy）的主观衡量结果。它是一个标量，不同数值之间有明确的大小关系（区别于不可直接进行大小比较的向量）。智能体被架构性地设计（例如梯度下降，Descent Gradient）为“倾向于”降低不舒适度/自由能。<br>
在后文中，不舒适度/自由能简称为代价。<br>

### 5.2、梯度 ![](https://latex.codecogs.com/svg.latex?\frac{\partial{}f^\prime[1]}{\partial{}a[0]})

现在，![](https://latex.codecogs.com/svg.latex?f^\prime[1]) 既定已知，![](https://latex.codecogs.com/svg.latex?a[0]) 既定已知，并且鉴于

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}f^\prime[1]&=\text{C}(s^\prime[1])\\&=\text{C}(\text{Enc}(x^\prime))\\&=\text{C}(\text{Enc}(\text{World}(x,a[0]))),\end{align*}">
</p>

朴素地希望通过计算代价 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]) 对动作 ![](https://latex.codecogs.com/svg.latex?a[0]) 的梯度

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\frac{\partial{}f^\prime[1]}{\partial{}a[0]}">
</p>

来实现动作 ![](https://latex.codecogs.com/svg.latex?a[0]) 的（事后）梯度优化，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?a[0]\leftarrow{}a[0]-\eta\cdot\frac{\partial{}f^\prime[1]}{\partial{}a[0]},">
</p>

其中 ![](https://latex.codecogs.com/svg.latex?\leftarrow) 表示一个迭代符号， ![](https://latex.codecogs.com/svg.latex?\eta) 表示学习率（learning rate，属于超参数），表达式整体表示用新值

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?a[0]-\eta\cdot\frac{\partial{}f^\prime[1]}{\partial{}a[0]}">
</p>

覆盖旧值 ![](https://latex.codecogs.com/svg.latex?a[0]) ，从而实现 ![](https://latex.codecogs.com/svg.latex?a[0]) 的迭代更新，也就是智能体动作的迭代优化。<br>

### 5.3、模式1的本质缺陷

然而，通过对 ![](https://latex.codecogs.com/svg.latex?\partial{}f^\prime[1]/\partial{}a[0]) 进行链式分解，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\frac{\partial{}f^\prime[1]}{\partial{}a[0]}=\frac{\partial{}f^\prime[1]}{\partial\text{C}}\cdot\frac{\partial\text{C}}{\partial\text{Enc}}\cdot\frac{\partial\text{Enc}}{\partial\text{World}}\cdot\frac{\partial\text{World}}{\partial{}a[0]},">
</p>

可以发现，代价 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]) 对动作 ![](https://latex.codecogs.com/svg.latex?a[0]) 的梯度居然包含不可微分的 ![](https://latex.codecogs.com/svg.latex?\text{World}(\cdot)) ，这就意味着

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\frac{\partial\text{World}}{\partial{}a[0]}">
</p>

不可解，也就直接导致代价 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]) 对动作 ![](https://latex.codecogs.com/svg.latex?a[0]) 的梯度整体不可解。<br>
原论文作者Yann LeCun提到，强化学习中的经典方法通过“用多个扰动动作轮询世界”的方式来估计代价 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]) 对动作 ![](https://latex.codecogs.com/svg.latex?a[0]) 的梯度值，但他同时指出“这很慢并且可能很危险”，例如碰撞导致的机械臂损坏以及对环境物体的破坏。<br>
由此可以直接判断，上述这种“数学缺陷”构成了智能体在模式1（类似于人类的“直觉模式”）下的本质缺陷，这为架构设计者提供了某种优化动机，即模式2具体机制的探索动机。
<br><br><br><br><br><br><br>

# 模式2

## 1、真实代价 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]=\text{C}(s^\prime[1])) 构成相对于预测代价 ![](https://latex.codecogs.com/svg.latex?f[1]=\text{C}(s[1])) 的监督信号

实际上，在模式1当中，代价模块的另一个子模块——可训练评价器子模块（Trainable Critic sub-module，后文简称为评价器），会接收世界状态预测

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s[1]=\text{Pred}(s[0],a[0])">
</p>

作为输入，计算并输出对应的“预测代价”，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?f[1]=\text{C}(s[1]).">
</p>

应注意，这区别于真实感知并由内在代价子模块计算的

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?f^\prime[1]=\text{C}(s^\prime[1]).">
</p>

![](https://latex.codecogs.com/svg.latex?f[1]) 是预测性的，而 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]) 是真实的，在时序关系上，前者先于后者，后者滞后于前者。<br>
在 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]) 被计算得出之后，真实的世界状态感知表征 ![](https://latex.codecogs.com/svg.latex?s^\prime[1]) 及其对应的刚刚计算得到的代价 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]) 所组成的“真实状态-真实代价”配对 ![](https://latex.codecogs.com/svg.latex?(s^\prime[1],f^\prime[1])) 就会被存储于智能体的短期记忆模块（short-term memory module），而 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]) 本身就会作为（相对于 ![](https://latex.codecogs.com/svg.latex?f[1]) 的）监督信号，指导评价器的迭代优化。<br>
也就是说，可以将 ![](https://latex.codecogs.com/svg.latex?f^\prime[1]) 与 ![](https://latex.codecogs.com/svg.latex?f[1]) 之间的某种散度（Divergence）![](https://latex.codecogs.com/svg.latex?D(f^\prime[1],f[1])) 的最小化作为评价器的训练目标。例如，散度可以具体指定为绝对值的平方，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?D(f^\prime[1],f[1])=\lVert{}f^\prime[1]-f[1]\rVert^2,">
</p>

它的最小化就可以作为评价器的训练目标。<br>
训练目标的内涵就在于减小评价器的预测误差。评价器对世界模型模块所给出的预测世界状态的对应代价的准确估计，将为构成预测世界状态影响因素之一的提议动作的优化带来可能。这构成了智能体借助模式2进行提议动作后果预演（reasoning）以及最优动作序列规划（planning）的基础。

## 2、模式2的闭环

在当前时刻 ![](https://latex.codecogs.com/svg.latex?t=0) ，模式2的从感知到动作优化所组成的一个闭环（或称“回合”，episode）可以表述为总共7个步骤：<br>
**1、感知（perception）**：<br>
感知模块的编码器给出 ![](https://latex.codecogs.com/svg.latex?s[0]=\text{Enc}(x)) 。<br>
**2、动作提议（action proposal）**：<br>
执行模块提出一个有待后续进行评估/优化的初始动作序列（可以理解为“直觉动作序列”），即 ![](https://latex.codecogs.com/svg.latex?a[0],\cdots,a[t],\cdots,a[T]) 。<br>
**3、模拟世界状态演化（simulation，带有主观性）**：<br>
根据

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s[i+1]=\text{Pred}(s[i],a[i]),">
</p>

世界模型模块针对初始动作序列的每一个提议动作，依次给出对应的世界状态预测，使其组成世界状态预测序列，即 ![](https://latex.codecogs.com/svg.latex?s[1],\cdots,s[t+1],\cdots,s[T+1]) 。<br>
**4、预测代价总和评估（evaluation，可以理解为“提议动作后果预演”，reasoning）**：<br>
根据

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?f[i]=\text{C}(s[i]),">
</p>

评价器针对世界状态预测序列当中的每一个世界状态预测，分别给出对应的代价预测，使其组成代价预测序列，即 ![](https://latex.codecogs.com/svg.latex?\text{C}(s[1]),\cdots,\text{C}(s[t+1]),\cdots,\text{C}(s[T+1])) ，并计算其总和，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?F(T)=\sum_{i=1}^{T+1}\text{C}(s[i]).">
</p>

可以料想，倘若评价器欠缺良好的训练，水平欠佳，预测误差很大，那么代价预测序列及其总和就会是不准确的。<br>
**5、对提议动作序列进行迭代优化（即“最优动作序列规划”，planning）**：<br>
预测代价总和 ![](https://latex.codecogs.com/svg.latex?F(T)) 对每一个提议动作 ![](https://latex.codecogs.com/svg.latex?a[i]) 的梯度所组成的序列为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\frac{\partial{}F(T)}{\partial{}a[0]},\cdots,\frac{\partial{}F(T)}{\partial{}a[t]},\cdots,\frac{\partial{}F(T)}{\partial{}a[T]},">
</p>

其中，对于具体某一个提议动作 ![](https://latex.codecogs.com/svg.latex?a[t]) 而言，预测代价总和对它的梯度具体计算为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}\frac{\partial{}F(T)}{\partial{}a[t]}&=\frac{\partial\sum_{i=1}^{T+1}\text{C}(s[i])}{\partial{}a[t]}\\&=\frac{\partial\left[\text{C}(s[1])+\cdots+\text{C}(s[t+1])+\cdots+\text{C}(s[T+1])\right]}{\partial{}a[t]}\\&=\frac{\partial\text{C}(s[1])}{\partial{}a[t]}+\cdots+\frac{\partial\text{C}(s[t+1])}{\partial{}a[t]}+\cdots+\frac{\partial\text{C}(s[T+1])}{\partial{}a[t]}\\&=\frac{\partial\text{C}(\text{Pred}(s[0],a[0]))}{\partial{}a[t]}+\cdots+\frac{\partial\text{C}(\text{Pred}(s[t],a[t]))}{\partial{}a[t]}+\cdots+\frac{\partial\text{C}(\text{Pred}(s[T],a[T]))}{\partial{}a[t]}\\&=\frac{\partial\text{constant}_0}{\partial{}a[t]}+\cdots+\frac{\partial\text{C}(\text{Pred}(s[t],a[t]))}{\partial{}a[t]}+\cdots+\frac{\partial\text{constant}_T}{\partial{}a[t]}\\&=0+\cdots+\frac{\partial\text{C}(\text{Pred}(s[t],a[t]))}{\partial{}a[t]}+\cdots+0\\&=\frac{\partial\text{C}(\text{Pred}(s[t],a[t]))}{\partial{}a[t]}\\&=\frac{\partial\text{C}(\text{Pred}(s[t],a[t]))}{\partial\text{Pred}(s[t],a[t])}\cdot\frac{\partial\text{Pred}(s[t],a[t])}{\partial{}a[t]},\end{align*}">
</p>

由此可知，提议动作 ![](https://latex.codecogs.com/svg.latex?a[t]) 构成一个单独的自变量，只与 ![](https://latex.codecogs.com/svg.latex?s[t])（而非其他 ![](https://latex.codecogs.com/svg.latex?a[i]) ）一起产生预测代价因变量

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\text{C}(s[t+1])=\text{C}(\text{Pred}(s[t],a[t])),">
</p>

该因变量进而构成预测代价总和因变量

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?F(T)=\sum_{i=1}^{T+1}\text{C}(s[i])">
</p>

的其中一个加性（additive）贡献项；![](https://latex.codecogs.com/svg.latex?F(T)) 对 ![](https://latex.codecogs.com/svg.latex?a[t]) 的梯度实际上仅仅指 ![](https://latex.codecogs.com/svg.latex?\text{C}(s[t+1])) 对 ![](https://latex.codecogs.com/svg.latex?a[t]) 的梯度，而与由其他那些提议动作自变量所产生的加性贡献项无关。<br>
换句话说，![](https://latex.codecogs.com/svg.latex?a[t]) 作为一个单独的自变量，与其他的提议动作自变量在梯度优化意义上是解耦而互不干扰的。<br>
如果代价模块的评价器作为函数 ![](https://latex.codecogs.com/svg.latex?\text{C}(\cdot)) 是可微的（differentiable，或者称其为"well-behaved"），那么原梯度的链式分解项

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\frac{\partial\text{C}(\text{Pred}(s[t],a[t]))}{\partial\text{Pred}(s[t],a[t])}">
</p>

就是可解的；<br>
如果世界模型模块的预测器作为函数 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(\cdot,\cdot)) 是可微的，那么原梯度的另一个链式分解项

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\frac{\partial\text{Pred}(s[t],a[t])}{\partial{}a[t]}">
</p>

就是可解的；<br>
在上述两个假设成立的前提下，原梯度

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\frac{\partial\text{C}(\text{Pred}(s[t],a[t]))}{\partial{}a[t]}">
</p>

是可解的，也就是说，提议动作 ![](https://latex.codecogs.com/svg.latex?a[t]) 的基于梯度下降方法的迭代优化方式

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?a[t]\leftarrow{}a[t]-\eta\cdot\frac{\partial{}F(T)}{\partial{}a[t]}">
</p>

是可行的。提议动作序列中的其他提议动作 ![](https://latex.codecogs.com/svg.latex?a[i]) 同理。<br>
总之，在代价模块的评价器以及世界模型的预测器均可微的前提下，使用梯度下降方法来对提议动作序列 ![](https://latex.codecogs.com/svg.latex?a[0],\cdots,a[t],\cdots,a[T]) 进行迭代优化是可行的。<br>
在经过从**步骤2**到**步骤5**的充分多轮次的迭代后，可以得到使得预测代价总和最小的“最优动作序列”，记为 ![](https://latex.codecogs.com/svg.latex?\check{a}[0],\cdots,\check{a}[t],\cdots,\check{a}[T]) 。这样的迭代过程就可以理解为智能体在模式2下对最优动作序列的规划过程。<br>
**6、实际执行（acting）**：<br>
最优动作序列中的第1个动作（或者前几个动作）被随即发送到智能体的效应模块并被实际执行。<br>
应该特别注意，并非所有动作全部被发送到效应模块，这是为了避免智能体的预测误差（尤其是世界状态预测误差以及代价预测误差）累积过大所导致的决策动作错误累积过大（进而直接导致真实内在代价累积过大）。<br>
**7、评价器与预测器的优化（optimization）**：<br>
假设实际执行动作序列为 ![](https://latex.codecogs.com/svg.latex?\check{a}[0],\cdots,\check{a}[t]) 。根据

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?x[i+1]=\text{World}(x[i],a[i]),">
</p>

感知模块的传感器所接收到的物理世界信号时序可表示为 ![](https://latex.codecogs.com/svg.latex?x[1],\cdots,x[t+1]) 。先后经由感知模块的编码器处理，得到 ![](https://latex.codecogs.com/svg.latex?s^\prime[1],\cdots,s^\prime[t+1]) 。先后输入到代价模块的内在代价子模块，得到 ![](https://latex.codecogs.com/svg.latex?\text{C}(s^\prime[1]),\cdots,\text{C}(s^\prime[t+1])) ；与此同时，“真实状态-真实代价”配对序列 ![](https://latex.codecogs.com/svg.latex?(s^\prime[1],\text{C}(s^\prime[1])),\cdots,(s^\prime[t+1],\text{C}(s^\prime[t+1]))) 被先后地存储到短期记忆模块。<br>
实际上，早在**步骤4**（预测代价总和评估）当中，世界状态预测序列 ![](https://latex.codecogs.com/svg.latex?s[1],\cdots,s[t+1],\cdots,s[T+1]) 与计算得到的代价预测序列 ![](https://latex.codecogs.com/svg.latex?\text{C}(s[1]),\cdots,\text{C}(s[t+1]),\cdots,\text{C}(s[T+1])) 所组成的“预测状态-预测代价”配对序列 ![](https://latex.codecogs.com/svg.latex?(s[1],\text{C}(s[1])),\cdots,(s[t+1],\text{C}(s[t+1])),\cdots,(s[T+1],\text{C}(s[T+1]))) 就已经被存储到短期记忆模块。<br>
先前存储的“预测状态-预测代价”配对序列与刚刚存储的“真实状态-真实代价”配对序列的用途，就在于**步骤7**的此刻所发生的评价器优化以及预测器优化：<br>
![](https://latex.codecogs.com/svg.latex?\text{C}(s^\prime[1]),\cdots,\text{C}(s^\prime[t+1])) 分别作为相对于 ![](https://latex.codecogs.com/svg.latex?\text{C}(s[1]),\cdots,\text{C}(s[t+1])) 的监督信号，指导评价器在模式2的此轮闭环中的优化，也可通过梯度分解链间接指导预测器在模式2的此轮闭环中的优化。<br>
模式2下不断重复的闭环，在迭代实现最优动作序列规划（当前主观认为的最优不等同于未来主观认为的最优）及其部分实际执行的同时，也实现了评价器以及预测器的迭代优化，后两者的迭代优化为前者的迭代实现提供支撑。<br>
根据原论文作者Yann LeCun的观点，模式2的这种闭环是“带有滚动时域的模型预测控制（Model Predictive Control，MPC）”的一个实例，并且它与经典实现方式的区别在于，产生世界状态预测序列 ![](https://latex.codecogs.com/svg.latex?s[1],\cdots,s[t+1],\cdots,s[T+1]) 的世界模型模块预测器是可训练的（与预编程相对），产生代价预测序列 ![](https://latex.codecogs.com/svg.latex?\text{C}(s[1]),\cdots,\text{C}(s[t+1]),\cdots,\text{C}(s[T+1])) 的代价模块评价器也是可训练的。
<br><br><br><br><br><br><br>

# 用模式1近似实现模式2在特定任务中的功能

## 1、最初的“最优动作序列”迭代收敛至真正的最优动作序列

首次面对某个几乎全新的特定任务时，智能体需要掌握针对该任务的新技能，这要求它动用模式2来进行提议动作后果预演以及最优动作序列规划。在规划出当前最优动作序列并部分实际执行之后，智能体可能仍然“感到不满意”（即真实内在代价总和 ![](https://latex.codecogs.com/svg.latex?\text{C}(s^\prime[1])+\cdots+\text{C}(s^\prime[t])) 仍然过高），选择重复模式2的闭环。那么，伴随着评价器以及预测器的迭代优化，该序列还会经过多轮迭代，最终收敛至真正的最优动作序列。<br>
此时，相比之下，最初规划的“最优动作序列”可能是次优的。<br>
总之，智能体在首次面对某个几乎全新的任务时，既可能仅仅动用1次模式2闭环，也可能多次重复模式2闭环，以规划出真正的最优动作序列 ![](https://latex.codecogs.com/svg.latex?\check{a}[0],\cdots,\check{a}[t],\cdots,\check{a}[T]) 。

## 2、用模式1近似实现模式2在特定任务中的功能

### 2.1、策略子模块的训练以及摊销推理

现在，针对该任务，智能体已经掌握了真正的最优动作序列。希望设计一种机制，使它以后再次面对（甚至多次重复面对）同样的任务时，无需再动用计算资源耗费较大、耗时较长的模式2，而是借助反应迅速的模式1来近似实现模式2的功能。<br>
该机制的具体实现方式如下：<br>
1、感知模块的编码器给出 ![](https://latex.codecogs.com/svg.latex?s[0]=\text{Enc}(x[0])) 。<br>
2、执行模块的策略子模块给出 ![](https://latex.codecogs.com/svg.latex?a[0]=\text{A}(s[0])) ，它将被随即发送到智能体的效应模块并被实际执行。<br>
3、![](https://latex.codecogs.com/svg.latex?\check{a}[0]) 作为相对于 ![](https://latex.codecogs.com/svg.latex?a[0]) 的监督信号，指导策略子模块的参数调整（将 ![](https://latex.codecogs.com/svg.latex?D(\check{a}[0],a[0])) 的最小化作为训练目标）。<br>
4、感知模块通过传感器接收到来自所处物理世界的信号 ![](https://latex.codecogs.com/svg.latex?x[1]=\text{World}(x[0],a[0])) ，后续过程同理，![](https://latex.codecogs.com/svg.latex?D(\check{a}[1],a[1])) 的最小化将会作为策略子模块的训练目标。<br>
5、![](https://latex.codecogs.com/svg.latex?D(\check{a}[2],a[2]),\cdots,D(\check{a}[t],a[t]),\cdots,D(\check{a}[T],a[T])) 同理。<br>
6、期间只有策略子模块的参与，没有任何其他模块的参与。<br>
一旦策略子模块完成了上述这种训练，智能体以后面对同样的任务时，便只需动用模式1的这一策略子模块来迅速输出近似最优动作序列（循环且迅速地执行上述的步骤1至步骤2），无需动用模式2。<br>
这实现了“摊销推理”（amortized inference）的效果，即：<br>
繁重的预演、规划、规划迭代、策略习得仅仅发生在“熟练掌握新技能”这一初始时期，并且耗费大量计算资源。而在这一时期结束过后，每当面对同样的任务，智能体均可以极低的计算成本（仅 ![](https://latex.codecogs.com/svg.latex?\text{A}(\cdot)) 函数映射所需的计算资源）近似复现当初所习得的最优动作序列。该过程的本质是推理，它具备摊销性（平均推理成本随次数增多而下降）。

### 2.2、短期记忆容量限制

针对某个特定任务，试图绕过策略子模块的训练而完整精确地存储最优动作序列，以期实现另一种摊销推理，这种做法实际上并不可行。<br>
这是因为，智能体在其生命周期当中需要面对海量不同的任务，每天都可能面对新的任务，新任务的数量总是不断增长，倘若对每种不同的任务都完整精确地存储最优动作序列，很有可能早早触及短期记忆容量上限，或者早早遗忘，这种做法也就变得不可行。<br>
与之相对，智能体可以为多种不同的任务（或者任务集）分别配置专门的策略子模块（而非完整精确存储对应的最优动作序列），它们甚至可以同时工作。这是可行的，因为对策略子模块的存储仅仅意味着 ![](https://latex.codecogs.com/svg.latex?\text{A}(\cdot)) 这一函数的少量参数的存储，这具备存储资源占用上的可控性。

### 2.3、质量较高的初始序列

如果智能体意图针对某个特定任务进一步规划出更优的动作序列，那么它可以首先利用编码器（给出 ![](https://latex.codecogs.com/svg.latex?s[0]) ）、策略子模块（给出 ![](https://latex.codecogs.com/svg.latex?a[i]) ）、预测器（给出 ![](https://latex.codecogs.com/svg.latex?s[i]) ），较为迅速地生成质量较高的初始动作序列（近似于原先规划出的最优动作序列），即 ![](https://latex.codecogs.com/svg.latex?\tilde{a}[0],\cdots,\tilde{a}[t],\cdots,\tilde{a}[T]) ，其中

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\tilde{a}[i+1]=\text{A}(s[i+1]),\quad{}s[i+1]=\text{Pred}(s[i],\tilde{a}[i]),">
</p>

再以之为规划的起点，规划出更优的动作序列，记为 ![](https://latex.codecogs.com/svg.latex?\check{\check{a}}[0],\cdots,\check{\check{a}}[t],\cdots,\check{\check{a}}[T]) ，从而实现加速优化的效果。<br>

### 2.4、模式1与模式2的分工

智能体只拥有唯一一个世界模型模块与唯一一个代价模块，其模式2单次只能用于应对单个任务。为了避免资源错配，有必要使模式2仅仅用于应对全新的或者复杂的任务，而将简单的或者已经熟练掌握的众多任务交给多个可以并行工作的模式1策略子模块。
<br><br><br><br><br><br><br>

# 潜变量能量模型

## 1、一切智能个体所进行的主观预测行为的本质局限以及不确定性根基

在当前时刻，智能体的感知模块通过传感器接收到来自所处物理世界的信号 ![](https://latex.codecogs.com/svg.latex?x) ，这是刚刚发生并且已经发生的事实；<br>
在未来时刻，智能体的感知模块通过传感器接收到来自所处物理世界的信号可能为 ![](https://latex.codecogs.com/svg.latex?y) ，这是一种概率性事件，当前并未发生，未来也不保证一定发生，但可以说，未来它有可能发生。
