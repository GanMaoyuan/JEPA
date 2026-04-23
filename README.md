#### [《A Path Towards Autonomous Machine Intelligence Version 0.9.2, 2022-06-27》](https://openreview.net/pdf?id=BZ5a1r-kVsf)论文阅读笔记
<br><br><br><br><br><br><br><br><br><br>

# 模式1

## 1、世界状态 ![](https://latex.codecogs.com/svg.latex?s[0]=\text{Enc}(x))

将智能体所处的“当前时刻”定义为 ![](https://latex.codecogs.com/svg.latex?t=0) ，区别于 ![](https://latex.codecogs.com/svg.latex?t=1,t=2) 等“未来时刻”。<br>
智能体（agent）的感知模块（perception module）通过传感器（sensor，例如摄像头）接收到来自所处物理世界（physical world）的信号（signal），即 ![](https://latex.codecogs.com/svg.latex?x[0])（方括号内的 ![](https://latex.codecogs.com/svg.latex?0) 指示“当前时刻”，在后文中，![](https://latex.codecogs.com/svg.latex?x[0]) 简记为 ![](https://latex.codecogs.com/svg.latex?x) ），例如由十的若干次方数量级的大量像素通道值所表示的一张高清实时帧（可记为 ![](https://latex.codecogs.com/svg.latex?x\in\mathbb{R}^{d_{\text{frame}}}) ），并通过编码器（Encoder）对其进行处理，从而提取出智能体自己对自己所处物理世界的状态（world state）的感知（perception，带有主观性，区别于信号 ![](https://latex.codecogs.com/svg.latex?x) 的客观性，主观性意味着不可完全克服的“认知偏差”）的表征，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s[0]=\text{Enc}(x).">
</p>

这种表征通常是任务情景化的，同一物理实体的感知表征在不同任务情景中往往是不同的。<br>
假如智能体处于“组装/修理/建造”这一类特定任务情景，那么一把锤子会被它感知表征为“可能有用的工具”，而在其他那些与“组装/修理/建造”关联性较弱的任务情景中，这把锤子会被感知表征为“无关紧要的东西”。<br>
从原论文作者Yann LeCun的立场来看，针对感知模块编码器的这种“任务情景功能化配置”（contextually configuring）行为是由智能体的配置器（configurator）完成的。

## 2、即时动作 ![](https://latex.codecogs.com/svg.latex?a[0]=\text{A}(s[0]))

智能体的执行模块（Actor module）的策略子模块（policy sub-module）接收 ![](https://latex.codecogs.com/svg.latex?s[0]) 作为输入，直接计算并输出动作（action），即 ![](https://latex.codecogs.com/svg.latex?a[0]=\text{A}(s[0])) 。输出的动作称为“动作提议”（action proposal），它将被随即发送到智能体的效应模块（effector module，例如机械臂、机械足等）并被实际执行。<br>
该过程就相当于一个直接的函数映射，期间没有（除配置器以外的）任何其他模块的参与，因而表现出反应上的迅速性，类似于人类的“条件反射”或者“肌肉记忆”。<br>
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

它的内涵可以表述为：智能体在 ![](https://latex.codecogs.com/svg.latex?t=1) 时刻对世界状态的主观感知 ![](https://latex.codecogs.com/svg.latex?s^\prime[1]) 的“不舒适度”（discomfort），或者智能体在 ![](https://latex.codecogs.com/svg.latex?t=1) 时刻对世界状态所具备的“自由能”（free energy）的主观衡量结果。它是一个标量，不同数值之间有明确的大小关系（区别于不可直接进行大小比较的向量）。智能体被架构性地设计（例如梯度下降，Descent Gradient）为“倾向于”降低不舒适度/自由能。<br>
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
<br><br><br><br><br><br><br><br><br><br>

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

在当前时刻 ![](https://latex.codecogs.com/svg.latex?t=0) ，模式2的从感知到动作优化所组成的一个闭环（在原论文中称为“回合”，episode，笔者选择使用“闭环”概念）可以表述为总共7个步骤：<br>
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
<br><br><br><br><br><br><br><br><br><br>

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
<br><br><br><br><br><br><br><br><br><br>

# 潜变量能量模型

## 1、一切智能个体所进行的主观预测行为的本质局限以及不确定性根基

在当前时刻，智能体的感知模块通过传感器接收到来自所处物理世界的信号 ![](https://latex.codecogs.com/svg.latex?x) ，这是刚刚发生并且已经发生的事实；<br>
在未来时刻，智能体的感知模块通过传感器接收到来自所处物理世界的信号可能为 ![](https://latex.codecogs.com/svg.latex?y) ，这是一种概率性事件，当前并未发生，未来也不保证一定发生，但可以说，未来它有可能发生。<br>
智能体无法直接从 ![](https://latex.codecogs.com/svg.latex?x) 获取足够信息来完全解释为什么会发生 ![](https://latex.codecogs.com/svg.latex?y) ，或者无法直接从 ![](https://latex.codecogs.com/svg.latex?x) 获取足够信息来确定性地推演出 ![](https://latex.codecogs.com/svg.latex?y) 。只要 ![](https://latex.codecogs.com/svg.latex?y) 尚未发生，智能体就不会有100%的把握来判断“ ![](https://latex.codecogs.com/svg.latex?x) 一定会演化为 ![](https://latex.codecogs.com/svg.latex?y) ”。<br>
根据原论文作者Yann LeCun的观点，这是因为<br>
1、世界在本质上就是随机的（即“偶然不确定性”，aleatoric uncertainty）。<br>
2、世界状态感知表征 ![](https://latex.codecogs.com/svg.latex?s[0]) 所包含的关于真实世界状态的信息是不完整的（即“认知不确定性”，epistemic uncertainty）。<br>
3、世界模型预测器由于重重限制（例如训练数据有限、表征维度/能力有限、计算资源有限等）而远不足以实现“上帝水平”的完美精确预测。<br>
这三大因素构成了一切智能个体所进行的主观预测行为的本质局限以及不确定性根基，即：<br>
给定一个初始的物理世界信号 ![](https://latex.codecogs.com/svg.latex?x) ，个体认为它会演化到多种可能的 ![](https://latex.codecogs.com/svg.latex?y) ，演化到每一种可能的 ![](https://latex.codecogs.com/svg.latex?y) 的过程都有（主观上感觉）合理的解释。个体没有办法完全消除这种不确定性，没有办法一口咬定 ![](https://latex.codecogs.com/svg.latex?x) 必然会演化到某个特别的 ![](https://latex.codecogs.com/svg.latex?y) 而必然不会演化到其他的 ![](https://latex.codecogs.com/svg.latex?y) 。

## 2、潜变量能量模型的潜变量推断以及能量景观

如果要让智能体的智能行为更真实、更加具备可解释性、更易于泛化，那么，针对这种根植于智能体智能行为本质的不确定性的数学建模就是绕不开的。<br>
引入一个“潜变量”（latent variable），即 ![](https://latex.codecogs.com/svg.latex?z) ，用于表征从 ![](https://latex.codecogs.com/svg.latex?x) 演化到 ![](https://latex.codecogs.com/svg.latex?y) 的过程的“合理解释”。<br>
例如，假设 ![](https://latex.codecogs.com/svg.latex?x) 是人像的正视图，某个可能的 ![](https://latex.codecogs.com/svg.latex?y_1) 是人像的左侧视图，某个可能的 ![](https://latex.codecogs.com/svg.latex?y_2) 是人像的右侧视图，那么<br>
1、![](https://latex.codecogs.com/svg.latex?z_1) 可以表征“原来相机镜头顺时针绕了90度”这一无法直接从 ![](https://latex.codecogs.com/svg.latex?x) 发掘的潜藏信息（latent information）。<br>
2、![](https://latex.codecogs.com/svg.latex?z_2) 可以表征“原来相机镜头逆时针绕了90度”这一同样无法直接从 ![](https://latex.codecogs.com/svg.latex?x) 发掘的潜藏信息。<br>
引入“潜变量能量模型”（Latent-Variable Energy-Based Model，后文简称为LVEBM），即 ![](https://latex.codecogs.com/svg.latex?E_\omega(x,y,z)) ，它本质上是一个用于度量 ![](https://latex.codecogs.com/svg.latex?x,y,z) 三者的“兼容性”的函数（下标 ![](https://latex.codecogs.com/svg.latex?\omega) 表示神经网络参数），其输出是一个标量，表示三者的“不兼容度”或“能量”。<br>
![](https://latex.codecogs.com/svg.latex?E_\omega(x,y,z)) 的值越大，三者的不兼容性就越严重，兼容性就越差。<br>

### 2.1、潜变量推断

LVEBM的潜变量推断，就是在 ![](https://latex.codecogs.com/svg.latex?x) 与 ![](https://latex.codecogs.com/svg.latex?y) 的众多可能的关系解释中，找到相对最合理的关系解释的潜变量表征，换句话说就是在固定一对 ![](https://latex.codecogs.com/svg.latex?x,y) 的前提下，找到使得 ![](https://latex.codecogs.com/svg.latex?E_\omega(x,y,z)) 取得最小值的那个 ![](https://latex.codecogs.com/svg.latex?z) ，记为 ![](https://latex.codecogs.com/svg.latex?\check{z}) ，可以将其称为“最优潜变量”。潜变量推断可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\check{z}=\underset{z\in\mathcal{Z}}{\text{argmin}}\,E_\omega(x,y,z).">
</p>

在这之后，将推断出的最优潜变量 ![](https://latex.codecogs.com/svg.latex?\check{z}) 代入 ![](https://latex.codecogs.com/svg.latex?E_\omega(x,y,z)) ，从而使后者转化为仅针对 ![](https://latex.codecogs.com/svg.latex?x,y) 两者的兼容性函数，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?F_\omega(x,y)=\min_{z\in\mathcal{Z}}E_\omega(x,y,z)=E_\omega(x,y,\check{z}),">
</p>

后文依然将其称为能量。

### 2.2、能量景观

无数个 ![](https://latex.codecogs.com/svg.latex?(x,y,F_\omega(x,y))) 坐标点所组成的“三维”景观也称为“能量景观”（energy landscape）。<br>
“山峰”就是 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,y)) 的值较大的坐标点，其所对应的 ![](https://latex.codecogs.com/svg.latex?x) 与 ![](https://latex.codecogs.com/svg.latex?y) 的能量（可以理解为“重力势能”）较高，兼容性较差；<br>
“山谷”就是 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,y)) 的值较小的坐标点，其所对应的 ![](https://latex.codecogs.com/svg.latex?x) 与 ![](https://latex.codecogs.com/svg.latex?y) 的能量较低，兼容性较好。<br>
需要澄清的是，能量景观仅仅表征着LVEBM的“世界观”（主观上认为什么是可能的，什么是不可能的），尚不构成LVEBM在实际推理时的直接依据。在后文 **“JEPA的训练：VICReg-第2节”** 这一部分内容当中，将对联合嵌入预测架构（LVEBM的一个实例）的预测性推理（区别于传统意义上的生成式推理）的实际过程及其与能量景观的关系展开进一步澄清。

## 3、LVEBM的训练目标

能量景观是通过训练来“雕刻”（sculpturing）的，能量景观的质量优劣完全取决于训练方式的优劣。<br>
在训练过程中，能量景观雕刻的基本逻辑在于：<br>
1、使训练样本数据点 ![](https://latex.codecogs.com/svg.latex?(x,y)) 的能量 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,y)) 相对较低。<br>
2、使非训练样本数据点 ![](https://latex.codecogs.com/svg.latex?(x,\hat{y})) 的能量 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,\hat{y})) 相对较高。<br>
如果仅仅关注前者而忽略后者，就无法确保每当 ![](https://latex.codecogs.com/svg.latex?\hat{y}\neq{}y) 时 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,\hat{y})>F_\omega(x,y)) 恒成立，此时能量景观可能发生“坍塌”（collapse），即：<br>
在推理时，给定一个 ![](https://latex.codecogs.com/svg.latex?x) ，它与任意 ![](https://latex.codecogs.com/svg.latex?y) 的能量都一样低，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?F_\omega(x,y_1)\approx\cdots\approx{}F_\omega(x,y_m)\approx\cdots\approx{}F_\omega(x,y_n)\approx{}0.">
</p>

这就意味着，LVEBM失去区分能力，认为所有的 ![](https://latex.codecogs.com/svg.latex?y) 都是合理的。这样的LVEBM在推理时不能表现出任何价值。<br>
总之，使训练样本数据点的能量相对较低，使非训练样本数据点的能量相对较高，两者共同构成LVEBM的训练目标。

## 4、LVEBM的训练：对比方法（contrastive methods）

### 4.1、第1种对比损失函数：铰链损失

第1种对比损失函数设计为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}L(\omega,x,y,\hat{y})&=H(F_\omega(x,y),F_\omega(x,\hat{y}),m(y,\hat{y}))\\&=\Bigl[m(y,\hat{y})-(F_\omega(x,\hat{y})-F_\omega(x,y))\Bigr]^+\\&=\begin{cases}m(y,\hat{y})-(F_\omega(x,\hat{y})-F_\omega(x,y)),&m(y,\hat{y})-(F_\omega(x,\hat{y})-F_\omega(x,y))>0\\0,&m(y,\hat{y})-(F_\omega(x,\hat{y})-F_\omega(x,y))\leq{}0\end{cases}\end{align*}">
</p>

其中 ![](https://latex.codecogs.com/svg.latex?m(y,\hat{y})) 正比于 ![](https://latex.codecogs.com/svg.latex?y) 与 ![](https://latex.codecogs.com/svg.latex?\hat{y}) 的某种距离度量。例如 ![](https://latex.codecogs.com/svg.latex?\mu\lVert{}y-\hat{y}\rVert^2) ，它正比于 ![](https://latex.codecogs.com/svg.latex?y) 与 ![](https://latex.codecogs.com/svg.latex?\hat{y}) 之差的二范数 ![](https://latex.codecogs.com/svg.latex?\lVert{}y-\hat{y}\rVert^2) 。<br>
该损失函数的内涵在于，一对相距较远的 ![](https://latex.codecogs.com/svg.latex?y,\hat{y}) ，它们的能量差 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,\hat{y})-F_\omega(x,y)) 应该较大，否则施加损失。<br>
如果 ![](https://latex.codecogs.com/svg.latex?m(y,\hat{y})-(F_\omega(x,\hat{y})-F_\omega(x,y))>0) ，即 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,\hat{y})-F_\omega(x,y)<m(y,\hat{y})) ，说明能量差 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,\hat{y})-F_\omega(x,y)) 相对 ![](https://latex.codecogs.com/svg.latex?m(y,\hat{y})) 太小，![](https://latex.codecogs.com/svg.latex?F_\omega(x,\hat{y})) 相对不够高（或者 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,y)) 相对不够低），此时就把 ![](https://latex.codecogs.com/svg.latex?m(y,\hat{y})) 与 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,\hat{y})-F_\omega(x,y)) 的差值 ![](https://latex.codecogs.com/svg.latex?m(y,\hat{y})-(F_\omega(x,\hat{y})-F_\omega(x,y))) 作为损失项，施加给训练中的LVEBM；<br>
如果 ![](https://latex.codecogs.com/svg.latex?m(y,\hat{y})-(F_\omega(x,\hat{y})-F_\omega(x,y))\leq{}0) ，即 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,\hat{y})-F_\omega(x,y)\geq{}m(y,\hat{y})) ，说明能量差 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,\hat{y})-F_\omega(x,y)) 相对 ![](https://latex.codecogs.com/svg.latex?m(y,\hat{y})) 足够大，![](https://latex.codecogs.com/svg.latex?F_\omega(x,\hat{y})) 相对足够高（或者 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,y)) 相对足够低），此时无需施加任何损失（损失项为0）。<br>
以 ![](https://latex.codecogs.com/svg.latex?m(y,\hat{y})=\mu\lVert{}y-\hat{y}\rVert^2) 为例，上述这种对比损失函数将使得能量景观呈现一种特点：非训练数据点与训练数据点的能量差随它们之间距离度量的增长而至少呈现二次增长（grow quadratically）。<br>
上述这种对比损失函数属于“铰链损失”（hinge loss）的范畴。<br>
特定数据点（实数或 ![](https://latex.codecogs.com/svg.latex?d) 维实向量）集合所构成的特定数据分布称为“数据流形”（data manifold）。在经过良好雕刻的能量景观中，山谷与训练数据流形可以产生对应关系。

### 4.2、第2种对比损失函数：InfoNCE

第2种对比损失函数设计为涉及多个对比数据点 ![](https://latex.codecogs.com/svg.latex?(x,\hat{y}[1]),\cdots,(x,\hat{y}[k])) 的

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?L(\omega,x,y,\hat{y}[1],\cdots,\hat{y}[k])=H(F_\omega(x,y),F_\omega(x,\hat{y}[1]),\cdots,F_\omega(x,\hat{y}[k])),">
</p>

例如

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?L(\omega,x,y,\hat{y}[1],\cdots,\hat{y}[k])=F_\omega(x,y)+\log\left[\exp(-F_\omega(x,y))+\sum_{i=1}^k\exp(-F_\omega(x,\hat{y}[i]))\right],">
</p>

以下展示其构造过程。<br>
对于训练数据点 ![](https://latex.codecogs.com/svg.latex?(x,y)) 以及 ![](https://latex.codecogs.com/svg.latex?k) 个对比数据点 ![](https://latex.codecogs.com/svg.latex?(x,\hat{y}[1]),\cdots,(x,\hat{y}[k])) ，定义 ![](https://latex.codecogs.com/svg.latex?(x,y)) 在其中的权重为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?W(x,y)=\frac{\exp(-F_\omega(x,y))}{\exp(-F_\omega(x,y))+\sum_{i=1}^k\exp(-F_\omega(x,\hat{y}[i]))},">
</p>

两边取对数，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\log{}W(x,y)=-F_\omega(x,y)-\log\left[\exp(-F_\omega(x,y))+\sum_{i=1}^k\exp(-F_\omega(x,\hat{y}[i]))\right],">
</p>

两边取负号，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?-\log{}W(x,y)=F_\omega(x,y)+\log\left[\exp(-F_\omega(x,y))+\sum_{i=1}^k\exp(-F_\omega(x,\hat{y}[i]))\right].">
</p>

使 ![](https://latex.codecogs.com/svg.latex?W(x,y)) 较大（甚至接近1），就意味着分式的分子 ![](https://latex.codecogs.com/svg.latex?\exp(-F_\omega(x,y))) 相对较大，而分母

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\exp(-F_\omega(x,y))+\sum_{i=1}^k\exp(-F_\omega(x,\hat{y}[i]))">
</p>

相对较小，也就进一步意味着训练数据点的能量 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,y)) 相对较低，而任意一个对比数据点的能量 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,\hat{y}[i])) 都相对较高。<br>
这正好契合LVEBM的训练目标。因此，可以将 ![](https://latex.codecogs.com/svg.latex?W(x,y)) 的负对数 ![](https://latex.codecogs.com/svg.latex?-\log{}W(x,y)) 定义为损失项，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}L(\omega,x,y,\hat{y}[1],\cdots,\hat{y}[k])&=-\log{}W(x,y)\\&=F_\omega(x,y)+\log\left[\exp(-F_\omega(x,y))+\sum_{i=1}^k\exp(-F_\omega(x,\hat{y}[i]))\right],\end{align*}">
</p>

从而在 ![](https://latex.codecogs.com/svg.latex?W(x,y)) 较大时该损失项较小，在 ![](https://latex.codecogs.com/svg.latex?W(x,y)) 较小时该损失项较大。<br>
原论文作者Yann LeCun指出，该对比损失函数实例即为InfoNCE。<br>

### 4.3、对比方法的缺陷

对比方法在图像、视频等高维数据空间中得以实施的前提在于生成足够多的对比数据点，其数量级可能难以接受。<br>
换句话说，在高维数据空间中，生成足够多的对比样本就意味着高昂的生成计算成本。<br>
根据原论文作者Yann LeCun的观点，由于“维度诅咒”（the curse of the dimensionality），在最坏情况下，所需的对比样本的数量可能随数据维度的增长而指数增长。
<br><br><br><br><br><br><br><br><br><br>

# 联合嵌入预测架构

在后文中，本质上属于LVEBM范畴的联合嵌入预测架构（Joint Embedding Predictive Architecture）将简称为JEPA。<br>
![](https://latex.codecogs.com/svg.latex?x) 与 ![](https://latex.codecogs.com/svg.latex?y) 的模态既可以相同，也可以不同。例如，![](https://latex.codecogs.com/svg.latex?x) 是视频的同时，![](https://latex.codecogs.com/svg.latex?y) 既可以是视频，也可以是音频。分别处理 ![](https://latex.codecogs.com/svg.latex?x) 与 ![](https://latex.codecogs.com/svg.latex?y) 的编码器 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_x(\cdot),\text{Enc}_y(\cdot)) 既可以具备相同的架构，也可以具备不同的架构。<br>

## 1、JEPA所特有的预测器 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(\cdot,\cdot))

### 1.1、![](https://latex.codecogs.com/svg.latex?\text{Pred}(\cdot,\cdot)) 的引入

针对某对特定的 ![](https://latex.codecogs.com/svg.latex?x,y) ，它们之间客观地存在众多（在连续高维空间中则为无数多个）可能的关系解释表征（即潜变量）![](https://latex.codecogs.com/svg.latex?z) 。<br>
JEPA首先通过编码器将像素性的 ![](https://latex.codecogs.com/svg.latex?x,y) 非线性投影到抽象表征空间，得到抽象性的 ![](https://latex.codecogs.com/svg.latex?s_x,s_y) 。<br>
针对某个可能的 ![](https://latex.codecogs.com/svg.latex?z) ，将它连同 ![](https://latex.codecogs.com/svg.latex?s_x) 一起输入到JEPA所特有的预测器 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(\cdot,\cdot)) ，输出的 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) 的内涵可表述为：给定初始的世界状态感知表征 ![](https://latex.codecogs.com/svg.latex?s_x) 以及某个可能的“世界状态演化限制框架” ![](https://latex.codecogs.com/svg.latex?z) ，JEPA主观上认为，演化后的世界状态的感知表征将会是 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) 。

### 1.2、![](https://latex.codecogs.com/svg.latex?\text{Pred}(\cdot,\cdot)) 的一对一或者多对一性质

![](https://latex.codecogs.com/svg.latex?s_x,s_y) 是固定的，而 ![](https://latex.codecogs.com/svg.latex?z) 是可切换的，不同的 ![](https://latex.codecogs.com/svg.latex?z) 既可能对应相同的 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) ，也可能各自唯一对应不同的 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) 。<br>
需要特别强调的是，“不确定性”特指作为函数输入之一的 ![](https://latex.codecogs.com/svg.latex?z) 的取值多样性，绝非 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(\cdot,\cdot)) 函数本身的“映射不确定性”——一对一或者多对一是函数映射的基本性质，不存在“一对多”这种不确定性的说法。倘若一对多，那就不叫函数了。

## 2、JEPA的潜变量推断

### 2.1、能量函数 ![](https://latex.codecogs.com/svg.latex?E_\omega(x,y,z)) 在JEPA中的定义

在JEPA当中，![](https://latex.codecogs.com/svg.latex?E_\omega(x,y,z)) 定义为 ![](https://latex.codecogs.com/svg.latex?s_y=\text{Enc}_y(y)) 与 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) 的散度，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?E_\omega(x,y,z)=D(s_y,\text{Pred}(s_x,z)),">
</p>

其中 ![](https://latex.codecogs.com/svg.latex?s_x=\text{Enc}_x(x)) 。

### 2.2、JEPA的潜变量推断

在逻辑上，JEPA的潜变量推断流程可描述为：<br>
1、将所有的 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) 分别与 ![](https://latex.codecogs.com/svg.latex?s_y) 进行散度计算，分别得到 ![](https://latex.codecogs.com/svg.latex?D(s_y,\text{Pred}(s_x,z))) 。<br>
2、在所有的 ![](https://latex.codecogs.com/svg.latex?D(s_y,\text{Pred}(s_x,z))) 当中，选择值最小的那一个，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\min_{z\in\mathcal{Z}}D(s_y,\text{Pred}(s_x,z))=D(s_y,\text{Pred}(s_x,\check{z}))=F_\omega(x,y),">
</p>

其对应的 ![](https://latex.codecogs.com/svg.latex?z) 的取值

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\check{z}=\underset{z\in\mathcal{Z}}{\text{argmin}}\,D(s_y,\text{Pred}(s_x,z))=\underset{z\in\mathcal{Z}}{\text{argmin}}\,E_\omega(x,y,z)">
</p>

即为 ![](https://latex.codecogs.com/svg.latex?x) 与 ![](https://latex.codecogs.com/svg.latex?y) 之间众多可能的关系解释表征当中，JEPA主观上认为最合理的那个关系解释表征。<br>
此外，由于不同的 ![](https://latex.codecogs.com/svg.latex?z) 对应相同的 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(s_x,z)) 这种多对一情形是可能存在的，最优潜变量 ![](https://latex.codecogs.com/svg.latex?\check{z}) 不一定只有唯一一个，值不相同的多个 ![](https://latex.codecogs.com/svg.latex?\check{z}) 可以同时成为最合理的关系解释表征。

### 2.3、举例

假设 ![](https://latex.codecogs.com/svg.latex?x) 表示“车辆行驶至分岔口”的视频片段，![](https://latex.codecogs.com/svg.latex?y) 表示“车辆行驶至右分叉道路后通过传感器所观察到的沿途风景”，那么最合理的关系解释表征 ![](https://latex.codecogs.com/svg.latex?\check{z}) 将会表示“原来车辆在分岔口选择右转”，而不会表示“原来车辆在分岔口选择左转”，因为左分叉道路的沿途风景将与实际观察到的右分叉道路的沿途风景产生矛盾，“原来车辆在分岔口选择左转”并不能成为 ![](https://latex.codecogs.com/svg.latex?x) 与 ![](https://latex.codecogs.com/svg.latex?y) 这两个视频片段之间的合理关系解释。<br>
具体而言，设 ![](https://latex.codecogs.com/svg.latex?z_1) 表征“原来车辆在分岔口选择右转”，![](https://latex.codecogs.com/svg.latex?z_2) 表征“原来车辆在分岔口选择左转”，那么

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?D(s_y,\text{Pred}(s_x,\check{z}))=D(s_y,\text{Pred}(s_x,z_1))\ll{}D(s_y,\text{Pred}(s_x,z_2)),">
</p>

即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?F_\omega(x,y)=E_\omega(x,y,\check{z})=E_\omega(x,y,z_1)\ll{}E_\omega(x,y,z_2).">
</p>

## 3、JEPA良好预测行为的特征

对于身处真实物理世界并时时刻刻执行预测任务的JEPA而言，其良好的预测行为特征应该体现为：根据某个单一的 ![](https://latex.codecogs.com/svg.latex?x) ，借助潜变量 ![](https://latex.codecogs.com/svg.latex?z) ，预测出多个（而非仅仅1个）较为合理的 ![](https://latex.codecogs.com/svg.latex?y) 。<br>
这就意味着，对于JEPA而言，其良好的能量景观应该具备的一种特征将会体现为：对于某个单一的 ![](https://latex.codecogs.com/svg.latex?x) ，存在多个（而非仅仅1个）![](https://latex.codecogs.com/svg.latex?y) ，使数据点 ![](https://latex.codecogs.com/svg.latex?(x,y)) 的能量 ![](https://latex.codecogs.com/svg.latex?F_\omega(x,y)) 相等且较低。<br>
倘若JEPA的预测行为不具备上述这种特性，根据某个单一的 ![](https://latex.codecogs.com/svg.latex?x) 只能预测出单一的 ![](https://latex.codecogs.com/svg.latex?y) ，那么JEPA将沦为只会机械复刻训练数据场景的预编程机器，其在真实而复杂的物理世界中的泛化性能就将大打折扣。<br>
“根据某个单一的 ![](https://latex.codecogs.com/svg.latex?x) 预测出多个较为合理的 ![](https://latex.codecogs.com/svg.latex?y) ”所体现的良好预测方式称为“多峰值预测”（multi-modal prediction），它捕捉到了 ![](https://latex.codecogs.com/svg.latex?x) 与多个较为合理的 ![](https://latex.codecogs.com/svg.latex?y) 的依赖关系。原论文作者Yann LeCun将这种依赖关系称为“多峰值依赖关系”（multi-modal dependencies）。<br>
在逻辑上，所有可能的 ![](https://latex.codecogs.com/svg.latex?y) 组成一个分布，多个较为合理的 ![](https://latex.codecogs.com/svg.latex?y) 拥有较大的“可能度”（可以理解为“未归一化的概率值”，例如能量的负数的指数），从而形成分布上的“多个峰值”。这就是多峰值预测中“多峰值”一词的组词逻辑。

## 4、使JEPA具备良好预测行为特征的关键架构设计

### 4.1、编码器 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_y(\cdot)) 的多对一特性

编码器 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_y(\cdot)) 可以消除 ![](https://latex.codecogs.com/svg.latex?y) 当中的无关细节，仅保留其抽象语义，并用输出的 ![](https://latex.codecogs.com/svg.latex?s_y) 来表征这种抽象语义。<br>
这就意味着，值不相同的多个 ![](https://latex.codecogs.com/svg.latex?y)（例如局部纹理细节、无关物体细节不同的视频帧）虽然有着各异的无关细节，但在编码器的处理下，有可能具备完全相同的抽象语义。<br>
也就是说，值不相同的多个 ![](https://latex.codecogs.com/svg.latex?y) 有可能对应同一个 ![](https://latex.codecogs.com/svg.latex?s_y) ，进而可以推知，它们与同一个 ![](https://latex.codecogs.com/svg.latex?x) 所组成的数据点的能量是有可能相同的。<br>
编码器 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_y(\cdot)) 的这种多对一特性直接支持了JEPA的多峰值预测。

### 4.2、![](https://latex.codecogs.com/svg.latex?z) 在分布空间集合 ![](https://latex.codecogs.com/svg.latex?\mathcal{Z}) 上变化的过程

“固定一对 ![](https://latex.codecogs.com/svg.latex?x,y) ，推断 ![](https://latex.codecogs.com/svg.latex?z) ”的最优潜变量推断视角，可以拓展为“固定 ![](https://latex.codecogs.com/svg.latex?x) ，将每1个 ![](https://latex.codecogs.com/svg.latex?z) 均视为最优潜变量，反向推断出与之适配的 ![](https://latex.codecogs.com/svg.latex?y) ”这种前向输出性的视角。后者并未与前者产生逻辑上的矛盾。<br>
当 ![](https://latex.codecogs.com/svg.latex?z) 在集合 ![](https://latex.codecogs.com/svg.latex?\mathcal{Z}) 上变化时，每1个 ![](https://latex.codecogs.com/svg.latex?z) 都会产生1个 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z))（不同的 ![](https://latex.codecogs.com/svg.latex?z) 既可能产生相同的 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) ，也可能产生不同的 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) ），与之对应的最适配的某些 ![](https://latex.codecogs.com/svg.latex?y) 所对应的同1个 ![](https://latex.codecogs.com/svg.latex?s_y) 满足

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s_y=\tilde{s}_y=\text{Pred}(s_x,z).">
</p>


由于 ![](https://latex.codecogs.com/svg.latex?s_y) 与 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 相等，它们的散度 ![](https://latex.codecogs.com/svg.latex?D(s_y,\tilde{s}_y)=0) ，进而有

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?E_\omega(x,y,z)=D(s_y,\text{Pred}(s_x,z))=D(s_y,\tilde{s}_y)=0,">
</p>

也就是说，固定的 ![](https://latex.codecogs.com/svg.latex?x) 、最优潜变量 ![](https://latex.codecogs.com/svg.latex?z) 、最适配的 ![](https://latex.codecogs.com/svg.latex?y) 这三者的能量为0，是最低的。能量不可能为负数，因为散度永远大于等于0，小于0无意义。<br>
简而言之，针对某个单一的 ![](https://latex.codecogs.com/svg.latex?x) ，![](https://latex.codecogs.com/svg.latex?z) 的数量可以很多；针对每1个 ![](https://latex.codecogs.com/svg.latex?z) ，它会对应1个 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z))（取值可能重复），进而对应1个 ![](https://latex.codecogs.com/svg.latex?s_y) ，进而对应**一群** ![](https://latex.codecogs.com/svg.latex?y) 。<br>
与集合 ![](https://latex.codecogs.com/svg.latex?\mathcal{Z}) 对应，众多 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 也组成一个集合，可表示为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\{\tilde{s}_y=\text{Pred}(s_x,z),\forall{}z\in\mathcal{Z}\},">
</p>

可简记为 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(s_x,\mathcal{Z})) 。<br>
![](https://latex.codecogs.com/svg.latex?z) 在分布空间集合 ![](https://latex.codecogs.com/svg.latex?\mathcal{Z}) 上变化的过程间接支持了JEPA的多峰值预测。
<br><br><br><br><br><br><br><br><br><br>

# JEPA的训练目标

JEPA作为LVEBM的一个实例，其训练目标依然没有脱离“使训练数据点的能量相对较低，使非训练数据点的能量相对较高”这一范畴。<br>
针对JEPA，“使训练数据点的能量相对较低”这一训练子目标可以被解读为：使JEPA的基于 ![](https://latex.codecogs.com/svg.latex?s_x) 并借助于 ![](https://latex.codecogs.com/svg.latex?z) 的预测 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) 的误差（即 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 相对于监督信号/标签 ![](https://latex.codecogs.com/svg.latex?s_y) 的散度 ![](https://latex.codecogs.com/svg.latex?D(s_y,\tilde{s}_y)) ）实现最小化。<br>
简而言之就是：使JEPA的前向输出 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) 在训练过程中逐渐趋同于 ![](https://latex.codecogs.com/svg.latex?s_y) 。<br>
那么，“使训练数据点的能量相对较低”这一训练子目标对应的损失项即为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?L_1=D(s_y,\tilde{s}_y).">
</p>

差值的二范数 ![](https://latex.codecogs.com/svg.latex?\lVert\cdot-\cdot\rVert^2) 可以作为散度 ![](https://latex.codecogs.com/svg.latex?D(\cdot,\cdot)) 的实例。<br>
在后文中，将散度具体指定为差值的二范数，第1个训练子目标所对应的损失项指定为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?L_1=\lVert{}s_y-\tilde{s}_y\rVert^2.">
</p>

<br><br><br><br><br><br><br><br>

# JEPA的训练：前三项原则

JEPA可以使用对比方法进行训练，但前文 **“潜变量能量模型-第4节-第4.3小节”** 已经指出对比方法的缺陷，现在希望探索出一种全新的训练方法。

## 1、单一损失项 ![](https://latex.codecogs.com/svg.latex?L_1=\lVert{}s_y-\tilde{s}_y\rVert^2) 的根本局限

JEPA的训练遵循4项原则，前两项分别为：<br>
1、在某种前提下，使 ![](https://latex.codecogs.com/svg.latex?s_x) 所包含的关于 ![](https://latex.codecogs.com/svg.latex?x) 的信息内容最大化（尽可能完整包含有关 ![](https://latex.codecogs.com/svg.latex?x) 的信息）。<br>
2、在同一前提下，使 ![](https://latex.codecogs.com/svg.latex?s_y) 所包含的关于 ![](https://latex.codecogs.com/svg.latex?y) 的信息内容最大化（尽可能完整包含有关 ![](https://latex.codecogs.com/svg.latex?y) 的信息）。<br>
倘若不对 ![](https://latex.codecogs.com/svg.latex?s_x,s_y) 这两个表征向量的各分量取值在全体训练样本（训练数据点 ![](https://latex.codecogs.com/svg.latex?(x,y)) ）批次的尺度上施加任何限制（不施加某种损失项），那么既然JEPA的训练子目标是使 ![](https://latex.codecogs.com/svg.latex?L_1=\lVert{}s_y-\tilde{s}_y\rVert^2) 最小化，在训练过程中JEPA完全有可能将 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_y(\cdot)) 的参数 ![](https://latex.codecogs.com/svg.latex?\omega_{\text{Enc}_y}) 以及 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_x(\cdot)) 的参数 ![](https://latex.codecogs.com/svg.latex?\omega_{\text{Enc}_x}) 最终调整至某个状态，使得 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_y(\cdot)) 对一切样本输入 ![](https://latex.codecogs.com/svg.latex?y) 都恒定输出一个完全相同的 ![](https://latex.codecogs.com/svg.latex?\hat{s}_y) ，![](https://latex.codecogs.com/svg.latex?\text{Enc}_x(\cdot)) 对一切样本输入 ![](https://latex.codecogs.com/svg.latex?x) 也同样都恒定输出一个完全相同的 ![](https://latex.codecogs.com/svg.latex?\hat{s}_x) ，并将 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(\cdot,\cdot)) 的参数 ![](https://latex.codecogs.com/svg.latex?\omega_{\text{Pred}}) 最终调整至某个状态，使得 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(\hat{s}_x,\cdot)) 对一切输入 ![](https://latex.codecogs.com/svg.latex?z) 都恒定输出 ![](https://latex.codecogs.com/svg.latex?\hat{s}_y) ，此时

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}L_1&=\lVert{}s_y-\text{Pred}(s_x,z)\rVert^2\\&=\lVert\text{Enc}_y(y)-\text{Pred}(\text{Enc}_x(x),z)\rVert^2\\&=\lVert\hat{s}_y-\text{Pred}(\hat{s}_x,z)\rVert^2\\&=\lVert\hat{s}_y-\hat{s}_y\rVert^2\\&=0.\end{align*}">
</p>

换句话说，倘若不对 ![](https://latex.codecogs.com/svg.latex?s_x,s_y) 这两个表征向量的各分量取值在全体训练样本批次的尺度上施加任何限制，那么在训练过程中，JEPA完全有可能演化到一种状态，在该状态下，它完全无视不同样本 ![](https://latex.codecogs.com/svg.latex?(x,y)) 之间的具体差异，完全没有对 ![](https://latex.codecogs.com/svg.latex?x,y) 进行有意义的特征提取（ ![](https://latex.codecogs.com/svg.latex?s_x) 所包含的关于 ![](https://latex.codecogs.com/svg.latex?x) 的信息内容等同于空白，也就是完全不包含，![](https://latex.codecogs.com/svg.latex?s_y) 同理），却同样能“完美”达成训练子目标（即达成 ![](https://latex.codecogs.com/svg.latex?L_1=0) ）。<br>
由于JEPA的参数组 ![](https://latex.codecogs.com/svg.latex?\omega_{\text{Enc}_x},\omega_{\text{Enc}_y},\omega_{\text{Pred}}) 最终收敛至上述这种病理性的状态（对一切输入都恒定输出同一个值），此时JEPA可以理解为学习到了“任何 ![](https://latex.codecogs.com/svg.latex?(x,y)) 的能量都是0，这与它是否出现在我的训练集当中无关”这种完全错误的观念。<br>
换句话说，表征着JEPA“世界观”的能量景观坍塌为“所有地点的能量值均为0的平原”，此时JEPA失去区分能力，认为所有的 ![](https://latex.codecogs.com/svg.latex?(x,y)) 都是兼容的。这样的JEPA在预测性推理时不能表现出任何价值。<br>
总之，前文提到的诸多因素共同构成了单一损失项 ![](https://latex.codecogs.com/svg.latex?L_1=\lVert{}s_y-\tilde{s}_y\rVert^2) 的根本局限，新损失项的引入势在必行。

## 2、JEPA预测性推理的实际过程及其与能量景观的关系

本小节暂时搁置对新损失项的探索，转而聚焦于JEPA预测性推理的实际过程及其与能量景观的关系。<br>
假设JEPA的训练方式恰当，训练效果良好，训练数据点的能量相对较低，在能量景观当中形成了训练数据流形；同时，非训练数据点的能量相对较高。总之，假设能量景观并未坍塌为平原。<br>
在此之后，JEPA开始在真实的物理世界中执行预测性推理任务。<br>
现在，输入 ![](https://latex.codecogs.com/svg.latex?x) ，针对相应数据模态的编码器将会输出 ![](https://latex.codecogs.com/svg.latex?s_x=\text{Enc}_x(x)) ；将 ![](https://latex.codecogs.com/svg.latex?s_x) 连同 ![](https://latex.codecogs.com/svg.latex?z) 作为预测器的输入，预测器将会输出 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) 。<br>
![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 所对应的一群 ![](https://latex.codecogs.com/svg.latex?\tilde{y}) ，各自与 ![](https://latex.codecogs.com/svg.latex?x) 所组成的数据点，在逻辑上构成对（已经过良好训练雕刻的）能量景观的采样结果，这些样本数据点位于能量景观中的训练数据流形（即谷底）。<br>
在实际的预测性推理过程中，并未发生对低能量数据点 ![](https://latex.codecogs.com/svg.latex?(x,y)) 的采样。这一步骤在物理层面上是不存在的，仅存在于逻辑层面。<br>
JEPA基于 ![](https://latex.codecogs.com/svg.latex?s_x) ，借助 ![](https://latex.codecogs.com/svg.latex?z) ，输出的对象实际上是抽象语义层面上的 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) ，并非像素细节层面上的 ![](https://latex.codecogs.com/svg.latex?\tilde{y}) 。这从根本上否定了“在能量景观中对低能量数据点 ![](https://latex.codecogs.com/svg.latex?(x,y)) 进行采样”的物理实在性。<br>
进一步地说，JEPA在推理时所执行的任务是对未来帧 ![](https://latex.codecogs.com/svg.latex?\tilde{y}) 的抽象语义表征 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 的预测（predicting），而非对未来帧 ![](https://latex.codecogs.com/svg.latex?\tilde{y}) 的直接采样（sampling）或者生成（generating）。对未来帧的直接采样或生成也可以理解为对未来帧的每一处像素细节的“预测”，这在本质上是困难的，原论文作者Yann LeCun认为并不可取。<br>
实际上，对未来帧的直接采样或生成属于以Diffusion为代表的生成式模型的任务范畴；而JEPA的任务与之相对，是对未来帧抽象语义表征的预测。<br>
原论文作者Yann LeCun明确指出，在视频预测场景中，对未来每一帧的每个像素（通道）值的预测在本质上就是不可能的，“地毯纹理的细节、树上迎风颤动的树叶或者池塘上的涟漪，是无法被准确预测的，至少无法在不消耗巨量计算资源的同时被长时间准确预测”。<br>
此外还需澄清以下两点：<br>
1、在训练时，JEPA通过损失项 ![](https://latex.codecogs.com/svg.latex?L_1=\lVert{}s_y-\text{Pred}(s_x,z)\rVert^2) ，得以接收来自训练样本“未来帧”（此处定义为在训练样本视频中相对于当前时刻帧的时序较为靠后的帧，对接受训练的JEPA而言是已知而可见的）![](https://latex.codecogs.com/svg.latex?y) 的抽象语义表征 ![](https://latex.codecogs.com/svg.latex?s_y) 的信息，此时 ![](https://latex.codecogs.com/svg.latex?s_y) 本质上是监督信号/标签。<br>
2、而在预测性推理时，真实物理世界的未来帧 ![](https://latex.codecogs.com/svg.latex?y) 对当前时刻的JEPA而言还根本不存在，也就没有“接收 ![](https://latex.codecogs.com/svg.latex?s_y) 的信息”这种说法。JEPA的预测性推理其实也根本不依赖于 ![](https://latex.codecogs.com/svg.latex?y) 或 ![](https://latex.codecogs.com/svg.latex?s_y)（过去不可能依赖未来），只需接收 ![](https://latex.codecogs.com/svg.latex?s_x,z) 的信息并进行一次前向计算

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)">
</p>

即可实现。

## 3、为了遵循前两项原则，抽象语义表征向量应该具备两个分量结构特征

后文以 ![](https://latex.codecogs.com/svg.latex?s_x) 为例，![](https://latex.codecogs.com/svg.latex?s_y) 与之完全同理。<br>
设训练样本批次（Batch）的大小为 ![](https://latex.codecogs.com/svg.latex?N) ，该批次内训练样本的数量即为 ![](https://latex.codecogs.com/svg.latex?N) 。该批次内的 ![](https://latex.codecogs.com/svg.latex?N) 个训练样本 ![](https://latex.codecogs.com/svg.latex?x) 经编码器处理后，得到 ![](https://latex.codecogs.com/svg.latex?N) 个 ![](https://latex.codecogs.com/svg.latex?s_x) 。<br>
这 ![](https://latex.codecogs.com/svg.latex?N) 个 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 所组成的矩阵可表示为 ![](https://latex.codecogs.com/svg.latex?B_1\in\mathbb{R}^{N\times{}d}) 。<br>
可以合理推测，如果要使 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 所包含的关于 ![](https://latex.codecogs.com/svg.latex?x) 的信息内容最大化，那么，在批次统计尺度上，![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 的各分量应该具备以下两个特征：<br>
1、不恒定（ ![](https://latex.codecogs.com/svg.latex?B_1\in\mathbb{R}^{N\times{}d}) 任意一列的 ![](https://latex.codecogs.com/svg.latex?N) 个元素不完全相同，它们的标准差统计量不为0）。<br>
2、独立（ ![](https://latex.codecogs.com/svg.latex?B_1\in\mathbb{R}^{N\times{}d}) 任意一行 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 的不同分量之间既保持线性不相关，也进一步保持非线性不相关，确保不同的维度不会重复携带相同信息）。<br>
在后文中，将用“特征1”、“特征2”分别进行指代。

## 4、引入新损失项，使 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 具备特征2

为了使 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 具备特征2，考虑引入一个新损失项。

### 4.1、新损失项 ![](https://latex.codecogs.com/svg.latex?L_2) 的引入

使用一个可训练的神经网络对每个 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 进行处理。输入 ![](https://latex.codecogs.com/svg.latex?B_1\in\mathbb{R}^{N\times{}d}) ，输出 ![](https://latex.codecogs.com/svg.latex?B_2\in\mathbb{R}^{N\times{}d^\prime}) ，其中 ![](https://latex.codecogs.com/svg.latex?d^\prime>d) ，![](https://latex.codecogs.com/svg.latex?B_2\in\mathbb{R}^{N\times{}d^\prime}) 的任意一行记为 ![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) 。<br>
换句话说，该神经网络将每个 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 非线性映射为更高维的 ![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) 。该神经网络也称为“扩展器”（expander）。<br>
设第 ![](https://latex.codecogs.com/svg.latex?i) 行 ![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) 的任意两个不同维度（不妨分别命名为A维度、B维度）的分量为 ![](https://latex.codecogs.com/svg.latex?a_i,b_i) 。<br>
在批次尺度上，对这两个维度之间的协方差统计量 ![](https://latex.codecogs.com/svg.latex?\text{Cov}(A,B)) 进行计算。首先分别计算这两个维度的均值统计量 ![](https://latex.codecogs.com/svg.latex?\mu_A,\mu_B) ，具体为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mu_A=\frac{1}{N}\sum_{i=1}^{N}a_i,\quad\mu_B=\frac{1}{N}\sum_{i=1}^{N}b_i,">
</p>

随后计算 ![](https://latex.codecogs.com/svg.latex?\text{Cov}(A,B)) ，具体为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\text{Cov}(A,B)=\frac{1}{N}\sum_{i=1}^{N}(a_i-\mu_A)(b_i-\mu_B),">
</p>

计算得到的 ![](https://latex.codecogs.com/svg.latex?\text{Cov}(A,B)) 的平方值 ![](https://latex.codecogs.com/svg.latex?\text{Cov}(A,B)^2) 将作为损失项施加给训练中的JEPA。可能为负值的 ![](https://latex.codecogs.com/svg.latex?\text{Cov}(A,B)) 不应直接作为损失项，这是因为，训练目标是使 ![](https://latex.codecogs.com/svg.latex?\text{Cov}(A,B)) 为0而非使 ![](https://latex.codecogs.com/svg.latex?\text{Cov}(A,B)) 不断变小（不断“负得更多”，这与训练目标相悖）。<br>
![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) 有 ![](https://latex.codecogs.com/svg.latex?d^\prime) 个维度，因此这样的损失项有

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\binom{d^\prime}{2}=C_{d^\prime}^2=\frac{d^\prime{}!}{2!(d^\prime-2)!}">
</p>

个，这些损失项的总和即为第1个新损失项，记为 ![](https://latex.codecogs.com/svg.latex?L_2) 。

### 4.2、![](https://latex.codecogs.com/svg.latex?L_2) 可用于消除线性相关性

![](https://latex.codecogs.com/svg.latex?L_2) 的本质是对 ![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) 的协方差正则化（regularization），它直接迫使 ![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) 的不同分量之间保持线性不相关，此时其协方差矩阵 ![](https://latex.codecogs.com/svg.latex?\text{Cov}(v_x)\in\mathbb{R}^{d^\prime\times{}d^\prime}) 保持为对角矩阵（对角线元素任意、非对角线元素均为0的矩阵）。<br>
![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 到 ![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) 的非线性映射尽管可能非常复杂，但它本质上是确定性的（没有任何与分布采样有关的随机性成分），这就意味着，在训练过程中，通过损失项实现的对 ![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) 的正则化会反向传导至 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) ，从而间接迫使 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 的不同分量之间同样保持线性不相关。

### 4.3、![](https://latex.codecogs.com/svg.latex?L_2) 可用于消除非线性相关性

在高维空间中强制线性不相关，会间接地在原始空间中消除非线性依赖关系，实现原始空间中的非线性不相关。<br>
具体而言，损失项 ![](https://latex.codecogs.com/svg.latex?L_2)（协方差正则化）强制 ![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) 的不同分量之间保持线性不相关，会间接实现 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 的不同分量之间的非线性不相关。

## 5、引入新损失项，使 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 具备特征1

为了使 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 具备特征1，考虑引入另一个新损失项。

### 5.1、新损失项 ![](https://latex.codecogs.com/svg.latex?L_3) 的引入

设第 ![](https://latex.codecogs.com/svg.latex?i) 行 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 的任意一个维度（不妨命名为C维度）的分量为 ![](https://latex.codecogs.com/svg.latex?c_i) 。<br>
在批次尺度上，对该维度的标准差统计量 ![](https://latex.codecogs.com/svg.latex?\text{SD}(C)=\sqrt{\text{Var}(C)}) 进行计算。首先计算该维度的均值统计量 ![](https://latex.codecogs.com/svg.latex?\mu_C) ，具体为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mu_C=\frac{1}{N}\sum_{i=1}^{N}c_i,">
</p>

随后计算 ![](https://latex.codecogs.com/svg.latex?\sqrt{\text{Var}(C)}) ，具体为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\sqrt{\text{Var}(C)}=\sqrt{\frac{1}{N}\sum_{i=1}^{N}(c_i-\mu_C)^2},">
</p>

计算得到的 ![](https://latex.codecogs.com/svg.latex?\sqrt{\text{Var}(C)}) 将作为一种铰链损失的组成部分。该损失项的具体形式为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\left[T-\sqrt{\text{Var}(C)}\right]^+=\begin{cases}T-\sqrt{\text{Var}(C)},&T-\sqrt{\text{Var}(C)}>0\\0,&T-\sqrt{\text{Var}(C)}\leq{}0\end{cases}">
</p>

其内涵在于，标准差 ![](https://latex.codecogs.com/svg.latex?\sqrt{\text{Var}(C)}) 应该较大，否则施加损失。<br>
如果 ![](https://latex.codecogs.com/svg.latex?T-\sqrt{\text{Var}(C)}>0) ，即 ![](https://latex.codecogs.com/svg.latex?\sqrt{\text{Var}(C)}<T) ，说明标准差 ![](https://latex.codecogs.com/svg.latex?\sqrt{\text{Var}(C)}) 相对阈值（Threshold，属于超参数，运行时为常数）![](https://latex.codecogs.com/svg.latex?T) 太小，此时就把 ![](https://latex.codecogs.com/svg.latex?T) 与 ![](https://latex.codecogs.com/svg.latex?\sqrt{\text{Var}(C)}) 的差值 ![](https://latex.codecogs.com/svg.latex?T-\sqrt{\text{Var}(C)}) 作为损失项施加给训练中的JEPA；<br>
如果 ![](https://latex.codecogs.com/svg.latex?T-\sqrt{\text{Var}(C)}\leq{}0) ，即 ![](https://latex.codecogs.com/svg.latex?\sqrt{\text{Var}(C)}\geq{}T) ，说明标准差 ![](https://latex.codecogs.com/svg.latex?\sqrt{\text{Var}(C)}) 相对阈值 ![](https://latex.codecogs.com/svg.latex?T) 足够大，此时无需施加任何损失（损失项为0）。<br>
![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 有 ![](https://latex.codecogs.com/svg.latex?d) 个维度，因此这样的损失项有 ![](https://latex.codecogs.com/svg.latex?d) 个，这些损失项的总和即为第2个新损失项，记为 ![](https://latex.codecogs.com/svg.latex?L_3) 。<br>
在原论文中，![](https://latex.codecogs.com/svg.latex?T) 设置为 ![](https://latex.codecogs.com/svg.latex?1) 。对于 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 的协方差矩阵 ![](https://latex.codecogs.com/svg.latex?\text{Cov}(s_x)\in\mathbb{R}^{d\times{}d}) 而言，在 ![](https://latex.codecogs.com/svg.latex?L_2) 的作用下，其非对角线元素（不同维度之间的协方差）将趋近于0；而在 ![](https://latex.codecogs.com/svg.latex?L_3) 的作用下，由于 ![](https://latex.codecogs.com/svg.latex?T=1) ，其对角线元素（各维度自己的方差）将趋近于1。因此，在 ![](https://latex.codecogs.com/svg.latex?L_2,L_3) 的共同作用下，![](https://latex.codecogs.com/svg.latex?\text{Cov}(s_x)\in\mathbb{R}^{d\times{}d}) 将趋近于单位矩阵（或称为恒等矩阵）![](https://latex.codecogs.com/svg.latex?I\in\mathbb{R}^{d\times{}d}) 。

### 5.2、![](https://latex.codecogs.com/svg.latex?L_3) 可用于维持抽象语义表征向量在批次尺度上必要的差异性

![](https://latex.codecogs.com/svg.latex?L_3) 的本质是对 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 的方差正则化（或称为标准差正则化），它直接迫使 ![](https://latex.codecogs.com/svg.latex?B_1\in\mathbb{R}^{N\times{}d}) 任意一列的 ![](https://latex.codecogs.com/svg.latex?N) 个元素不完全相同。换句话说，它直接维持了批次内不同表征向量 ![](https://latex.codecogs.com/svg.latex?s_x) 之间必要的差异性。

### 5.3、新损失项 ![](https://latex.codecogs.com/svg.latex?L_4) 的引入

![](https://latex.codecogs.com/svg.latex?L_4) 与 ![](https://latex.codecogs.com/svg.latex?L_3) 是类似的。唯一的区别仅在于，![](https://latex.codecogs.com/svg.latex?L_4) 的直接正则化对象是 ![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) ，而 ![](https://latex.codecogs.com/svg.latex?L_3) 的直接正则化对象是 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 。

### 5.4、![](https://latex.codecogs.com/svg.latex?L_4) 可用于间接增强对抽象语义表征向量的方差正则化

![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 到 ![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) 的非线性映射是确定性的，对 ![](https://latex.codecogs.com/svg.latex?v_x\in\mathbb{R}^{d^\prime}) 的正则化会反向传导至 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) ，从而间接增强对 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^d) 的方差正则化，从而有利于维持抽象语义表征向量在批次尺度上必要的差异性。

## 6、第3项原则以及损失项 ![](https://latex.codecogs.com/svg.latex?L_1=\lVert{}s_y-\tilde{s}_y\rVert^2) 的作用定位

JEPA训练遵循的第3项原则为：使 ![](https://latex.codecogs.com/svg.latex?s_y) 容易被 ![](https://latex.codecogs.com/svg.latex?s_x) 预测。<br>
它可以被等价解读为“使JEPA的前向输出 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) 在训练过程中逐渐趋同于 ![](https://latex.codecogs.com/svg.latex?s_y) ”，进而可以被等价解读为“使训练数据点的能量相对较低”，即LVEBM的第1个训练子目标。<br>
原始损失项 ![](https://latex.codecogs.com/svg.latex?L_1=\lVert{}s_y-\tilde{s}_y\rVert^2) 即对应于这一原则，降低 ![](https://latex.codecogs.com/svg.latex?L_1) 的过程就是遵循该原则的过程。<br>
前两项原则中的“某种前提”，指的就是第3项原则。<br>
此外需要指出的是，第3项原则本质上是在促使JEPA捕捉 ![](https://latex.codecogs.com/svg.latex?x,y) 这两个原始输入之间在抽象语义 ![](https://latex.codecogs.com/svg.latex?s_x,s_y) 层面上的一致性/不变性（invariance），而绝非迫使JEPA强行学习从 ![](https://latex.codecogs.com/svg.latex?x) 到 ![](https://latex.codecogs.com/svg.latex?y) 的复杂映射，强行根据 ![](https://latex.codecogs.com/svg.latex?x) 拟合出 ![](https://latex.codecogs.com/svg.latex?y) 。后一做法在本质上就是不可取的，尤其是在 ![](https://latex.codecogs.com/svg.latex?x,y) 均为高维连续空间中的数据（例如视频）这种常见的情况下。

## 7、对前三项原则的总结

前两项原则及其对应的损失项 ![](https://latex.codecogs.com/svg.latex?L_2,L_3,L_4) ，连同第3项原则及其对应的损失项 ![](https://latex.codecogs.com/svg.latex?L_1) 一道，共同组成了名为“方差-不变性-协方差正则化”（Variance-Invariance-Covariance Regularization，VICReg）的训练方式，其中“方差正则化”对应于 ![](https://latex.codecogs.com/svg.latex?L_3,L_4) ，“不变性正则化”对应于 ![](https://latex.codecogs.com/svg.latex?L_1) ，“协方差正则化”对应于 ![](https://latex.codecogs.com/svg.latex?L_2) 。<br>
抽象语义表征的完整性（completeness，对应于 ![](https://latex.codecogs.com/svg.latex?L_2,L_3,L_4) ）与可预测性（predictability，对应于 ![](https://latex.codecogs.com/svg.latex?L_1) ）这对训练目标，本质上就是相互抵触的——增强完整性就可能削弱（至少不可能同时增强）可预测性，增强可预测性就可能削弱（至少不可能同时增强）完整性。<br>
在VICReg下训练出的JEPA，能够在抽象语义表征的完整性与可预测性这对相互抵触的训练目标之间做出良好权衡；也就是说，在VICReg下训练出的JEPA，自发形成了关于“什么信息具备可预测性，需要关注”以及“什么信息不具备可预测性，只能忽略”（例如地毯纹理的细节、树上迎风颤动的树叶或者池塘上的涟漪）的信息处理偏好（在原论文中称为“归纳偏置”，inductive bias）。
<br><br><br><br><br><br><br><br><br><br>

# JEPA的训练：第4项原则

## 1、单一损失项 ![](https://latex.codecogs.com/svg.latex?L_1=\lVert{}s_y-\tilde{s}_y\rVert^2) 的另一种根本局限

JEPA训练遵循的第4项原则为：使潜变量 ![](https://latex.codecogs.com/svg.latex?z) 的信息内容最小化（最低限度必要化）。<br>
倘若不对 ![](https://latex.codecogs.com/svg.latex?z) 的分量取值施加任何限制，那么在训练过程中，JEPA完全有可能将 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(\cdot,\cdot)) 的参数 ![](https://latex.codecogs.com/svg.latex?\omega_{\text{Pred}}) 最终调整至某个状态，使得 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(\cdot,\cdot)) 完全无视输入的 ![](https://latex.codecogs.com/svg.latex?s_x) ，完全根据输入的 ![](https://latex.codecogs.com/svg.latex?z) 来输出 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)) 。<br>
与此同时，由于 ![](https://latex.codecogs.com/svg.latex?z) 的分量取值没有被施加任何限制，![](https://latex.codecogs.com/svg.latex?z) 的取值就是任意的，这意味着 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(s_x,z)) 这个神经网络函数的输出 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 就是任意的。在这种情况下，既然JEPA的训练子目标是使 ![](https://latex.codecogs.com/svg.latex?L_1=\lVert{}s_y-\text{Pred}(s_x,z)\rVert^2) 最小化，那么在训练过程中，在让 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(\cdot,\cdot)) 完全无视 ![](https://latex.codecogs.com/svg.latex?s_x) 的同时，JEPA还完全有可能直接利用“ ![](https://latex.codecogs.com/svg.latex?\text{Pred}(s_x,z)) 输出任意”这一点，将 ![](https://latex.codecogs.com/svg.latex?\omega_{\text{Pred}}) 从刚才的状态进一步调整至某个新状态，使得 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(s_x,z)) 直接精确地输出 ![](https://latex.codecogs.com/svg.latex?s_y) ，此时

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}L_1&=\lVert{}s_y-\text{Pred}(s_x,z)\rVert^2\\&=\lVert{}s_y-s_y\rVert^2\\&=0,\end{align*}">
</p>

“完美”达成训练子目标，却在这个过程中完全没有对 ![](https://latex.codecogs.com/svg.latex?s_x) 与 ![](https://latex.codecogs.com/svg.latex?s_y) 之间的映射关系（依赖关系）在神经网络参数层面上进行有意义的建模（因为 ![](https://latex.codecogs.com/svg.latex?s_x) 都被忽略了，![](https://latex.codecogs.com/svg.latex?s_x) 与 ![](https://latex.codecogs.com/svg.latex?s_y) 之间的映射关系建模也就无从谈起）。<br>
换句话说，JEPA通过刻意忽略 ![](https://latex.codecogs.com/svg.latex?s_x) ，再通过“ ![](https://latex.codecogs.com/svg.latex?z) 取值任意”这一点，得以利用“ ![](https://latex.codecogs.com/svg.latex?\text{Pred}(s_x,z)) 输出任意”这一漏洞，“作弊式”地达成训练子目标，但没有从训练数据点 ![](https://latex.codecogs.com/svg.latex?(x,y)) 当中学到任何东西。<br>
由于JEPA的参数 ![](https://latex.codecogs.com/svg.latex?\omega_{\text{Pred}}) 最终收敛至上述这种病理性的状态（完全无视 ![](https://latex.codecogs.com/svg.latex?s_x) ，仅凭 ![](https://latex.codecogs.com/svg.latex?z) 实现 ![](https://latex.codecogs.com/svg.latex?s_y) 的“完美”重构），此时JEPA仍然可以理解为学习到了“任何 ![](https://latex.codecogs.com/svg.latex?(x,y)) 的能量都是0，这与它是否出现在我的训练集当中无关”这种完全错误的观念。<br>
此时能量景观坍塌为平原，其后果不再赘述。<br>
此外需要指出的是，![](https://latex.codecogs.com/svg.latex?L_2,L_3,L_4) 所要解决的问题与上述问题根本不在同一个维度，是毫无关联的，![](https://latex.codecogs.com/svg.latex?L_2,L_3,L_4) 完全无助于解决上述问题。<br>
总之，前文提到的诸多因素共同构成了单一损失项 ![](https://latex.codecogs.com/svg.latex?L_1=\lVert{}s_y-\tilde{s}_y\rVert^2) 的另一种根本局限，针对 ![](https://latex.codecogs.com/svg.latex?z) 的正则化新损失项的引入势在必行。<br>
损失项众多所导致的训练稳定性欠佳，可能是早期JEPA难以落地的重要因素之一，这也构成了JEPA体系演进优化的动力（例如重新定义问题本质，设计新的训练范式，减少损失项的数量，增强训练稳定性）。

## 2、为了遵循第4项原则，引入新损失项 ![](https://latex.codecogs.com/svg.latex?L_5)

引入一个针对 ![](https://latex.codecogs.com/svg.latex?z) 的正则化（Regularization）损失项，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?L_5=R(z)=\alpha\lVert{}z\rVert_1,">
</p>

其中 ![](https://latex.codecogs.com/svg.latex?\lVert{}z\rVert_1) 表示 ![](https://latex.codecogs.com/svg.latex?z) 的一范数（即各分量绝对值之和），![](https://latex.codecogs.com/svg.latex?\alpha>0) 为超参数，用于调节该损失项的“强度”。<br>
设 ![](https://latex.codecogs.com/svg.latex?s_x,s_y\in\mathbb{R}^m,z\in\mathbb{R}^n) ，注意与前文中设定的 ![](https://latex.codecogs.com/svg.latex?s_x,s_y\in\mathbb{R}^{d^\prime}) 作区分。为了让视觉表示效果更加具备区分性，选择将记号 ![](https://latex.codecogs.com/svg.latex?d^\prime) 变更为 ![](https://latex.codecogs.com/svg.latex?m) ，但保持实际值不变，即 ![](https://latex.codecogs.com/svg.latex?m=d^\prime) 。

### 2.1、引入 ![](https://latex.codecogs.com/svg.latex?L_5=R(z)=\alpha\lVert{}z\rVert_1) 之前

起初，没有引入 ![](https://latex.codecogs.com/svg.latex?L_5=R(z)=\alpha\lVert{}z\rVert_1) 。<br>
在损失项

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?L_1=E_\omega(x,y,z)=D(s_y,\text{Pred}(s_x,z))">
</p>

的作用下，训练中的JEPA会无视 ![](https://latex.codecogs.com/svg.latex?s_x) ，并推断出能够使

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?L_1=E_\omega(x,y,z)=D(s_y,\text{Pred}(s_x,z))">
</p>

取得最小值的那个 ![](https://latex.codecogs.com/svg.latex?z) ，也就是使得

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?L_1=E_\omega(x,y,z)=D(s_y,\text{Pred}(s_x,z))=0">
</p>

的那个 ![](https://latex.codecogs.com/svg.latex?z) ，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\check{z}=\underset{z\in\mathbb{R}^n}{\text{argmin}}\,E_\omega(x,y,z).">
</p>

在能量景观当中，训练数据点 ![](https://latex.codecogs.com/svg.latex?(x,y)) 的能量即为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?F_\omega(x,y)=\min_{z\in\mathbb{R}^n}E_\omega(x,y,z)=E_\omega(x,y,\check{z})=0.">
</p>

由于 ![](https://latex.codecogs.com/svg.latex?\omega_{\text{Pred}}) 的病态性，远不止训练数据点，所有数据点的能量都将为0，能量景观坍塌为平原。

### 2.2、引入 ![](https://latex.codecogs.com/svg.latex?L_5=R(z)=\alpha\lVert{}z\rVert_1) 之后

首先澄清一点，![](https://latex.codecogs.com/svg.latex?L_2,L_3,L_4) 完全不涉及潜变量 ![](https://latex.codecogs.com/svg.latex?z) ，这3个损失项与当前内容无关。<br>
现在，引入了 ![](https://latex.codecogs.com/svg.latex?L_5=R(z)=\alpha\lVert{}z\rVert_1) 。<br>
在 ![](https://latex.codecogs.com/svg.latex?L_1) 与 ![](https://latex.codecogs.com/svg.latex?L_5) 之和

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?L_1+L_5=E_\omega(x,y,z)+R(z)=D(s_y,\text{Pred}(s_x,z))+\alpha\lVert{}z\rVert_1">
</p>

的作用下，训练中的JEPA会推断出能够使

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?L_1+L_5=E_\omega(x,y,z)+R(z)=D(s_y,\text{Pred}(s_x,z))+\alpha\lVert{}z\rVert_1">
</p>

取得最小值的那个 ![](https://latex.codecogs.com/svg.latex?z) ，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\check{z}=\underset{z\in\mathbb{R}^n}{\text{argmin}}\,\Bigl[E_\omega(x,y,z)+R(z)\Bigr].">
</p>

在能量景观当中，训练数据点的能量即为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?F(s_x,s_y)=\min_{z\in\mathbb{R}^n}\Bigl[E_\omega(x,y,z)+R(z)\Bigr],">
</p>

它将会足够小，至少小于某个正数。<br>
为了防止能量景观坍塌，一个基本的构想表述为如下4点：<br>
1、定义一个阈值，能量低于该阈值的区域定义为“低能量区域”，反之则为“高能量区域”。<br>
2、如果低能量区域的测度（measure，二维空间测度的别名就是“面积”，三维空间测度的别名就是“体积”）是有限的，能量景观就不会坍塌。能量景观坍塌后所变成的平原，其低能量区域的测度恰恰就是无限的。<br>
3、对于非平原的能量景观，定义的阈值越小，低能量区域的测度就越小。<br>
4、训练中的JEPA会在损失项的驱动下，使训练数据点占据测度有限的低能量区域，并且默认非训练数据点位于高能量区域，不会使其参与对低能量区域的占据。<br>
由此，一套用于论证 ![](https://latex.codecogs.com/svg.latex?L_5=R(z)=\alpha\lVert{}z\rVert_1) 如何防止能量景观坍塌的数学框架呼之欲出。

## （选读）3、论证框架的初步建立

对于能量景观中的任意数据点 ![](https://latex.codecogs.com/svg.latex?(x,y)) ，定义其能量为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}F(s_x,s_y)&=\min_{z\in\mathbb{R}^n}\Bigl[E_\omega(x,y,z)+R(z)\Bigr]\\&=\min_{z\in\mathbb{R}^n}\Bigl[D(s_y,\text{Pred}(s_x,z))+\alpha\lVert{}z\rVert_1\Bigr].\end{align*}">
</p>

假设 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)\in\mathbb{R}^m) 关于 ![](https://latex.codecogs.com/svg.latex?z\in\mathbb{R}^n) 是线性的，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z)=Az+b\in\mathbb{R}^m,">
</p>

其中 ![](https://latex.codecogs.com/svg.latex?A\in\mathbb{R}^{m\times{}n},b\in\mathbb{R}^m) 均为关于 ![](https://latex.codecogs.com/svg.latex?s_x\in\mathbb{R}^m) 的函数，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?A=A(s_x)\in\mathbb{R}^{m\times{}n},\quad{}b=b(s_x)\in\mathbb{R}^m.">
</p>

于是，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}F(s_x,s_y)&=\min_{z\in\mathbb{R}^n}\Bigl[D(s_y,\text{Pred}(s_x,z))+\alpha\lVert{}z\rVert_1\Bigr]\\&=\min_{z\in\mathbb{R}^n}\Bigl[D(s_y,Az+b)+\alpha\lVert{}z\rVert_1\Bigr].\end{align*}">
</p>

定义阈值 ![](https://latex.codecogs.com/svg.latex?h>0) ，并将低能量区域定义为满足 ![](https://latex.codecogs.com/svg.latex?F(s_x,s_y)<h) 的 ![](https://latex.codecogs.com/svg.latex?s_y) 的集合，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{Y}_h=\{s_y:F(s_x,s_y)<h\},">
</p>

只需论证 ![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 的测度有限或者 ![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 作为子集，被包含于某个测度有限的集合即可。<br>
在后文中，不仅限于对后一命题进行论证，而是会对 ![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 的集合包含关系的上下界以及集合包含关系在 ![](https://latex.codecogs.com/svg.latex?h\to{}0^+) 时的极限行为进行全面论证。

## （选读）4、![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 的集合包含关系的下界

### 4.1、下界推导的步骤1

定义

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{S}\subseteq\{1,\cdots,n\},\quad\mathcal{M}_{\mathcal{S}}=\{Az+b:z_i=0\,\;\text{for}\,\;\forall{}i\notin\mathcal{S}\},\quad\mathcal{M}_{\mathcal{S}}^\delta=\{Az+b:z_i=0\,\;\text{for}\,\;\forall{}i\notin\mathcal{S},\,\;\lVert{}z\rVert_1<\delta\}.">
</p>

根据 ![](https://latex.codecogs.com/svg.latex?\quad\mathcal{M}_{\mathcal{S}}^\delta) 的定义可知，对于任意的 ![](https://latex.codecogs.com/svg.latex?p\in\mathcal{M}_{\mathcal{S}}^{h/\alpha}) ，都存在 ![](https://latex.codecogs.com/svg.latex?z^\prime) ，满足 ![](https://latex.codecogs.com/svg.latex?z_i^\prime=0,\forall{}i\notin\mathcal{S}) 以及 ![](https://latex.codecogs.com/svg.latex?\lVert{}z^\prime\rVert_1<h/\alpha) ，并使得 ![](https://latex.codecogs.com/svg.latex?p=Az^\prime+b) 。<br>
于是，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}F(s_x,p)&=\min_{z\in\mathbb{R}^n}\Bigl[D(p,Az+b)+\alpha\lVert{}z\rVert_1\Bigr]\\&\leq{}D(p,Az^\prime+b)+\alpha\lVert{}z^\prime\rVert_1\\&=D(p,p)+\alpha\lVert{}z^\prime\rVert_1\\&=0+\alpha\lVert{}z^\prime\rVert_1\\&=\alpha\lVert{}z^\prime\rVert_1\\&<\alpha\cdot\frac{h}{\alpha}\\&=h.\end{align*}">
</p>

根据 ![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 的定义，可以直接判断，![](https://latex.codecogs.com/svg.latex?p\in\mathcal{Y}_h) 。

### 4.2、下界推导的步骤2

总之，对于任意的 ![](https://latex.codecogs.com/svg.latex?p\in\mathcal{M}_{\mathcal{S}}^{h/\alpha}) ，都有 ![](https://latex.codecogs.com/svg.latex?p\in\mathcal{Y}_h) 。<br>
对A集合而言，如果它的任意一个元素都属于B集合，那么A集合被包含于B集合。这两个命题是等价的。因此，可以直接判断，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}}^{h/\alpha}\subseteq\mathcal{Y}_h,">
</p>

![](https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}}^{h/\alpha}) 即为 ![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 的下界。

## （选读）5、![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 的集合包含关系的上界

### 5.1、上界推导的步骤1

根据 ![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 的定义可知，对于任意的 ![](https://latex.codecogs.com/svg.latex?p\in\mathcal{Y}_h) ，都存在 ![](https://latex.codecogs.com/svg.latex?z^*) ，使得

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}F(s_x,p)&=\min_{z\in\mathbb{R}^n}\Bigl[D(p,Az+b)+\alpha\lVert{}z\rVert_1\Bigr]\\&=D(p,Az^*+b)+\alpha\lVert{}z^*\rVert_1\\&<h,\end{align*}">
</p>

因此，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}\alpha\lVert{}z^*\rVert_1&\leq{}D(p,Az^*+b)+\alpha\lVert{}z^*\rVert_1\\&<h,\end{align*}">
</p>

因此，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\lVert{}z^*\rVert_1<\frac{h}{\alpha}.">
</p>

### 5.2、上界推导的步骤2

设 ![](https://latex.codecogs.com/svg.latex?z^*) 的所有非零分量当中，绝对值最小的分量的绝对值为 ![](https://latex.codecogs.com/svg.latex?\varepsilon) ，则存在

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?k_{\max}\in\mathcal{N},\quad{}k_{\max}\cdot\varepsilon\leq\lVert{}z^*\rVert_1<\frac{h}{\alpha},\quad{}k_{\max}=\lfloor\frac{h}{\varepsilon\alpha}\rfloor,">
</p>

使得 ![](https://latex.codecogs.com/svg.latex?z^*) 的零范数（即非零分量的个数）![](https://latex.codecogs.com/svg.latex?\lVert{}z^*\rVert_0) 满足

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\lVert{}z^*\rVert_0\leq{}k_{\max}.">
</p>

### 5.3、上界推导的步骤3

总之，对于任意的 ![](https://latex.codecogs.com/svg.latex?p\in\mathcal{Y}_h) ，都存在 ![](https://latex.codecogs.com/svg.latex?z^*) ，满足

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{S}^*=\{i:i\in\mathcal{S},z_i^*\neq{}0\},\quad{}\lvert\mathcal{S}^*\rvert=\lVert{}z^*\rVert_0\leq{}k_{\max},">
</p>

其中 ![](https://latex.codecogs.com/svg.latex?\lvert\mathcal{S}^*\rvert) 表示集合 ![](https://latex.codecogs.com/svg.latex?\mathcal{S}^*) 的基数（即非零元素的个数），并使得

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?D(p,Az^*+b)+\alpha\lVert{}z^*\rVert_1<h.">
</p>

### 5.4、上界推导的步骤4

由于

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}^*}=\{Az+b:z_i=0\,\;\text{for}\,\;\forall{}i\notin\mathcal{S}^*\},">
</p>

所以 ![](https://latex.codecogs.com/svg.latex?Az^*+b\in\mathcal{M}_{\mathcal{S}^*}) 。

定义

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\text{Proj}_{\mathcal{M}_{\mathcal{S}}}(s_y)=\underset{p\in\mathcal{M}_{\mathcal{S}}}{\text{argmin}}\,D(s_y,p),\quad\mathcal{N}_{h,\mathcal{S}}=\{s_y:D(s_y,\text{Proj}_{\mathcal{M}_{\mathcal{S}}}(s_y))<h\},">
</p>

其中，<br>
1、![](https://latex.codecogs.com/svg.latex?\text{Proj}_{\mathcal{M}_{\mathcal{S}}}(s_y)) 意即：给定一个点 ![](https://latex.codecogs.com/svg.latex?s_y) ，要求在集合 ![](https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}}) 中找到一个点；在集合 ![](https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}}) 中的所有点当中，找到的那个点与给定的点 ![](https://latex.codecogs.com/svg.latex?s_y) 的“距离”最小；该“距离”就定义为点 ![](https://latex.codecogs.com/svg.latex?s_y) 与集合 ![](https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}}) 的“距离”。在三维几何当中，该过程其实就是指一个点对一个面发起（垂直）投影（Projection），得到一个投影点；在面上的所有点当中，这个投影点与发起投影的那个点的距离是最小的，这个距离就定义为点与面的距离。<br>
2、![](https://latex.codecogs.com/svg.latex?\mathcal{N}_{h,\mathcal{S}}) 意即：与集合 ![](https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}}) 的“距离”小于 ![](https://latex.codecogs.com/svg.latex?h) 的无数个点所组成的集合，也就是 ![](https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}}) 的h-邻域。<br>
考虑到 ![](https://latex.codecogs.com/svg.latex?Az^*+b\in\mathcal{M}_{\mathcal{S}^*}) ，![](https://latex.codecogs.com/svg.latex?Az^*+b) 本质上是集合 ![](https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}^*}) 中的“任意点”，于是，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}D(s_y,\text{Proj}_{\mathcal{M}_{\mathcal{S}^*}}(s_y))&\leq{}D(s_y,Az^*+b)\\&\leq{}D(s_y,Az^*+b)+\alpha\lVert{}z^*\rVert_1\\&<h,\end{align*}">
</p>

因此，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?p\in\mathcal{N}_{h,\mathcal{S}^*}.">
</p>

### 5.5、上界推导的步骤5

再根据

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\lvert\mathcal{S}^*\rvert\leq{}k_{\max},">
</p>

可以直接判断，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{N}_{h,\mathcal{S}^*}\subseteq\mathcal{N}_{h,\mathcal{S}^{(0)}}\cup\mathcal{N}_{h,\mathcal{S}^{(1)}}\cup\mathcal{N}_{h,\mathcal{S}^{(2)}}\cup\cdots\cup\mathcal{N}_{h,\mathcal{S}^*}\cup\cdots\cup\mathcal{N}_{h,\mathcal{S}^{(k_{\max})}}=\bigcup_{\mathcal{S}:\lvert\mathcal{S}\rvert\leq{}k_{\max}}\mathcal{N}_{h,\mathcal{S}},">
</p>

其中 ![](https://latex.codecogs.com/svg.latex?\mathcal{S}^{(0)},\mathcal{S}^{(1)},\mathcal{S}^{(2)},\cdots,\mathcal{S}^{(k_{\max})}) 分别表示基数为 ![](https://latex.codecogs.com/svg.latex?0,1,2,\cdots,k_{\max}) 的集合 ![](https://latex.codecogs.com/svg.latex?\mathcal{S}) 。

### 5.6、上界推导的步骤6

于是，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?p\in\mathcal{N}_{h,\mathcal{S}^*}\subseteq\bigcup_{\mathcal{S}:\lvert\mathcal{S}\rvert\leq{}k_{\max}}\mathcal{N}_{h,\mathcal{S}},">
</p>

从而有

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?p\in\bigcup_{\mathcal{S}:\lvert\mathcal{S}\rvert\leq{}k_{\max}}\mathcal{N}_{h,\mathcal{S}}.">
</p>

### 5.7、上界推导的步骤7

总之，对于任意的 ![](https://latex.codecogs.com/svg.latex?p\in\mathcal{Y}_h) ，都有

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?p\in\bigcup_{\mathcal{S}:\lvert\mathcal{S}\rvert\leq{}k_{\max}}\mathcal{N}_{h,\mathcal{S}},">
</p>

所以

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{Y}_h\subseteq\bigcup_{\mathcal{S}:\lvert\mathcal{S}\rvert\leq{}k_{\max}}\mathcal{N}_{h,\mathcal{S}},">
</p>

后者即为 ![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 的上界。上界的测度是有限的，那么 ![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 的测度就必然有限。由此完成了“ ![](https://latex.codecogs.com/svg.latex?L_5=R(z)=\alpha\lVert{}z\rVert_1) 防止能量景观坍塌”的论证。<br>
后文将对集合包含关系在 ![](https://latex.codecogs.com/svg.latex?h\to{}0^+) 时的极限行为进行延伸探索。

## （选读）6、集合包含关系在 ![](https://latex.codecogs.com/svg.latex?h\to{}0^+) 时的极限行为

### 6.1、![](https://latex.codecogs.com/svg.latex?h\to{}0^+) 时的下界 ![](https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}}^{h/\alpha})

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}}^{h/\alpha}=\{Az+b:z_i=0\,\;\text{for}\,\;\forall{}i\notin\mathcal{S},\,\;\lVert{}z\rVert_1<\frac{h}{\alpha}\},">
</p>

当 ![](https://latex.codecogs.com/svg.latex?h\to{}0^+) 时，![](https://latex.codecogs.com/svg.latex?\lVert{}z\rVert_1\to{}0^+) ，从而 ![](https://latex.codecogs.com/svg.latex?z\to\mathbf{0})（零向量），从而 ![](https://latex.codecogs.com/svg.latex?Az+b\to{}b) ，从而

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{M}_{\mathcal{S}}^{h/\alpha}\to\{b\}.">
</p>

### 6.2、![](https://latex.codecogs.com/svg.latex?h\to{}0^+) 时的 ![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h)

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}\mathcal{Y}_h&=\{s_y:F(s_x,s_y)<h\}\\&=\{s_y:\min_{z\in\mathbb{R}^n}\Bigl[D(s_y,Az+b)+\alpha\lVert{}z\rVert_1\Bigr]<h\},\end{align*}">
</p>

存在 ![](https://latex.codecogs.com/svg.latex?z^*) 使得

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}\mathcal{Y}_h&=\{s_y:\min_{z\in\mathbb{R}^n}\Bigl[D(s_y,Az+b)+\alpha\lVert{}z\rVert_1\Bigr]<h\}\\&=\{s_y:D(s_y,Az^*+b)+\alpha\lVert{}z^*\rVert_1<h\},\end{align*}">
</p>

当 ![](https://latex.codecogs.com/svg.latex?h\to{}0^+) 时，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?D(s_y,Az^*+b)\to{}0^+,\quad{}\alpha\lVert{}z^*\rVert_1\to{}0^+,">
</p>

从而 ![](https://latex.codecogs.com/svg.latex?\lVert{}z^*\rVert_1\to{}0^+) ，从而 ![](https://latex.codecogs.com/svg.latex?z^*\to\mathbf{0}) ，从而 ![](https://latex.codecogs.com/svg.latex?Az^*+b\to{}b) ，从而

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?D(s_y,Az^*+b)\to{}D(s_y,b),">
</p>

再根据 ![](https://latex.codecogs.com/svg.latex?D(s_y,Az^*+b)\to{}0^+) ，可知

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?D(s_y,b)\to{}0^+,">
</p>

从而 ![](https://latex.codecogs.com/svg.latex?s_y\to{}b) ，从而

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{Y}_h\to\{b\}.">
</p>

### 6.3、![](https://latex.codecogs.com/svg.latex?h\to{}0^+) 时的上界 ![](https://latex.codecogs.com/svg.latex?\bigcup_{\mathcal{S}:\lvert\mathcal{S}\rvert\leq{}k_{\max}}\mathcal{N}_{h,\mathcal{S}})

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{N}_{h,\mathcal{S}}=\{s_y:D(s_y,\text{Proj}_{\mathcal{M}_{\mathcal{S}}}(s_y))<h\},">
</p>

当 ![](https://latex.codecogs.com/svg.latex?h\to{}0^+) 时，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}\mathcal{N}_{h,\mathcal{S}}&\to\{s_y:D(s_y,\text{Proj}_{\mathcal{M}_{\mathcal{S}}}(s_y))=0\}\\&=\mathcal{M}_{\mathcal{S}},\end{align*}">
</p>

因此

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\bigcup_{\mathcal{S}:\lvert\mathcal{S}\rvert\leq{}k_{\max}}\mathcal{N}_{h,\mathcal{S}}\to\bigcup_{\mathcal{S}:\lvert\mathcal{S}\rvert\leq{}k_{\max}}\mathcal{M}_{\mathcal{S}}.">
</p>

<br><br><br><br><br><br><br><br>

# JEPA的训练：关于训练阶段潜变量 ![](https://latex.codecogs.com/svg.latex?z) 的事实澄清

## 1、![](https://latex.codecogs.com/svg.latex?z) 的梯度优化过程

在针对样本 ![](https://latex.codecogs.com/svg.latex?(x,y)) 的训练（各样本训练过程并行）刚开始的时候，![](https://latex.codecogs.com/svg.latex?z) 处于初始值状态，它在接下来的训练过程中也是需要作为梯度下降优化对象来接受优化的；换句话说，最优潜变量 ![](https://latex.codecogs.com/svg.latex?\check{z}) 并非凭空产生，它是通过多轮迭代优化得到的。<br>
具体而言，设损失项总和 ![](https://latex.codecogs.com/svg.latex?L=L_1+L_2+L_3+L_4+L_5) ，那么 ![](https://latex.codecogs.com/svg.latex?L) 对 ![](https://latex.codecogs.com/svg.latex?z) 的梯度为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}\frac{\partial{}L}{\partial{}z}&=\frac{\partial{}(L_1+L_2+L_3+L_4+L_5)}{\partial{}z}\\&=\frac{\partial{}L_1}{\partial{}z}+\frac{\partial{}L_2}{\partial{}z}+\frac{\partial{}L_3}{\partial{}z}+\frac{\partial{}L_4}{\partial{}z}+\frac{\partial{}L_5}{\partial{}z},\end{align*}">
</p>

由于 ![](https://latex.codecogs.com/svg.latex?L_2,L_3,L_4) 作为仅针对抽象语义表征向量的分量取值的损失项，并不包含变量 ![](https://latex.codecogs.com/svg.latex?z) ，因此它们对 ![](https://latex.codecogs.com/svg.latex?z) 的梯度均为0，从而

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}\frac{\partial{}L}{\partial{}z}&=\frac{\partial{}L_1}{\partial{}z}+\frac{\partial{}L_2}{\partial{}z}+\frac{\partial{}L_3}{\partial{}z}+\frac{\partial{}L_4}{\partial{}z}+\frac{\partial{}L_5}{\partial{}z}\\&=\frac{\partial{}L_1}{\partial{}z}+\frac{\partial{}L_5}{\partial{}z}\\&=\frac{\partial\lVert{}s_y-\text{Pred}(s_x,z)\rVert^2}{\partial{}z}+\frac{\partial\alpha\lVert{}z\rVert_1}{\partial{}z}.\end{align*}">
</p>

![](https://latex.codecogs.com/svg.latex?z) 的梯度优化过程可以表示为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?z\leftarrow{}z-\eta\cdot\frac{\partial{}L}{\partial{}z},">
</p>

这一过程会被多次重复，从而实现 ![](https://latex.codecogs.com/svg.latex?z) 的迭代优化。<br>
![](https://latex.codecogs.com/svg.latex?L) 对 ![](https://latex.codecogs.com/svg.latex?z) 的优化与 ![](https://latex.codecogs.com/svg.latex?L) 对各个神经网络组件的参数（即 ![](https://latex.codecogs.com/svg.latex?\omega_{\text{Enc}_y},\omega_{\text{Enc}_x},\omega_{\text{Pred}}) ）的优化不存在直接联系，仅通过共同的损失项总和函数产生间接联系。

## 2、优化成果的存储与不存储

### 2.1、![](https://latex.codecogs.com/svg.latex?L) 对 ![](https://latex.codecogs.com/svg.latex?z) 的优化成果 ![](https://latex.codecogs.com/svg.latex?\check{z}) 以及能量景观不会被存储

训练结束后，![](https://latex.codecogs.com/svg.latex?L) 对 ![](https://latex.codecogs.com/svg.latex?z) 的优化成果包括一个仅存在于逻辑层面的能量景观。针对样本 ![](https://latex.codecogs.com/svg.latex?(x,y)) ，其作为训练数据点在该能量景观中的能量即为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}F_\omega(x,y)&=\min_{z\in\mathcal{Z}}(L_1+L_5)\\&=\min_{z\in\mathcal{Z}}\Bigl[\lVert{}s_y-\text{Pred}(s_x,z)\rVert^2+\alpha\lVert{}z\rVert_1\Bigr]\\&=\min_{z\in\mathcal{Z}}\Bigl[E_\omega(x,y,z)+R(z)\Bigr],\end{align*}">
</p>

它带有正则化项 ![](https://latex.codecogs.com/svg.latex?R(z)) ，区别于在前文 **“潜变量能量模型-第2节-第2.1小节”** 这一部分内容当中所提出的原始的

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?F_\omega(x,y)=\min_{z\in\mathcal{Z}}E_\omega(x,y,z).">
</p>

在前文 **“JEPA的训练：第4项原则-第2节-第2.2小节”** 这一部分内容当中，训练数据点的能量表示为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?F(s_x,s_y)=\min_{z\in\mathbb{R}^n}\Bigl[E_\omega(x,y,z)+R(z)\Bigr],">
</p>

它与刚刚提出的

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?F_\omega(x,y)=\min_{z\in\mathcal{Z}}\Bigl[E_\omega(x,y,z)+R(z)\Bigr]">
</p>

没有实质性的区别，仅在符号记法上存在差异。<br>
需要特别强调以下3点：<br>
1、潜变量 ![](https://latex.codecogs.com/svg.latex?z) 仅在训练阶段会被存储于物理硬件中，其值会被不断更新；<br>
2、训练结束后，针对样本 ![](https://latex.codecogs.com/svg.latex?(x,y)) 而言，使 ![](https://latex.codecogs.com/svg.latex?L_1+L_5)（或者 ![](https://latex.codecogs.com/svg.latex?L) ）取得最小值的最优潜变量 ![](https://latex.codecogs.com/svg.latex?\check{z}) 并不会被存储于物理硬件中。<br>
在前文 **“JEPA的训练：前三项原则-第2节”** 这一部分内容当中已经指出，JEPA在推理阶段接收 ![](https://latex.codecogs.com/svg.latex?s_x,z) 作为输入，输出 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) ，现在可以进一步澄清：此时 ![](https://latex.codecogs.com/svg.latex?z) 是对一个关于 ![](https://latex.codecogs.com/svg.latex?z) 的分布（用 ![](https://latex.codecogs.com/svg.latex?R) 表示）的采样结果，绝非事先单独存储于物理硬件的数据（这是因为，鉴于高维空间的连续性，这意味着对几乎无限的计算机存储资源的需求，根本不现实）。<br>
针对 ![](https://latex.codecogs.com/svg.latex?R) 分布的专门说明将在后文 **“潜变量分布（潜变量采样器）![](https://latex.codecogs.com/svg.latex?R) ”** 这一部分内容当中展开。<br>
3、训练结束后，针对全体样本而言，能量景观并不会被存储于物理硬件中，具体而言就是：能量景观的数据点 ![](https://latex.codecogs.com/svg.latex?(x,y,F_\omega(x,y))) 并不会被存储于物理硬件中（这同样不现实）。<br>
在前文 **“JEPA的训练：前三项原则-第2节”** 这一部分内容当中已经对此进行过说明。

### 2.2、![](https://latex.codecogs.com/svg.latex?L) 对各个神经网络组件的参数的优化成果会被存储

训练结束后，![](https://latex.codecogs.com/svg.latex?\omega_{\text{Enc}_y},\omega_{\text{Enc}_x},\omega_{\text{Pred}}) 作为JEPA的参数，会被存储于物理硬件中，这是当然的。
<br><br><br><br><br><br><br><br><br><br>

# 分层式JEPA

在后文中，将分层式JEPA（Hierarchical JEPA）简称为H-JEPA。<br>
设H-JEPA在 ![](https://latex.codecogs.com/svg.latex?t=0,t=1,t=2) 时刻先后接收到 ![](https://latex.codecogs.com/svg.latex?x[0],x[1],x[2]) 。

## 1、第1个JEPA层

设第1个JEPA层（后文简称为JEPA-1）的编码器为 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_1) ，预测器为 ![](https://latex.codecogs.com/svg.latex?\text{Pred}_1) ，潜变量采样器为 ![](https://latex.codecogs.com/svg.latex?R_1) 。

### 1.1、JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻的推理行为

JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻的推理行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}s_1[0]&=\text{Enc}_1(x[0])\\z_1&\sim{}R_1\\\tilde{s}_1[1]&=\text{Pred}_1(s_1[0],z_1)\end{align*}">
</p>

它具备代表性，![](https://latex.codecogs.com/svg.latex?t=1,t=2) 等后续时刻的JEPA-1完整推理行为（包括 ![](https://latex.codecogs.com/svg.latex?s_1[2]=\text{Enc}_1(x[2])) 等）与之同构，后文不再分别赘述。

### 1.2、JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=1) 时刻的训练行为

JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=1) 时刻的训练行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}s_1[1]&=\text{Enc}_1(x[1])\\\text{Loss}_1&=D_1(s_1[1],\tilde{s}_1[1])\end{align*}">
</p>

训练目标即为 ![](https://latex.codecogs.com/svg.latex?\text{Loss}_1) 的最小化，从而实现 ![](https://latex.codecogs.com/svg.latex?\omega_{\text{Enc}_1},\omega_{\text{Pred}_1})（如果 ![](https://latex.codecogs.com/svg.latex?R_1) 是可训练参数化的，而非固定参数化的，则还包括 ![](https://latex.codecogs.com/svg.latex?\omega_{R_1}) ）的迭代优化。

## 2、第2个JEPA层

设第2个JEPA层（后文简称为JEPA-2）的编码器为 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_2) ，预测器为 ![](https://latex.codecogs.com/svg.latex?\text{Pred}_2) ，潜变量采样器为 ![](https://latex.codecogs.com/svg.latex?R_2) 。

### 2.1、JEPA-2在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻的推理行为

JEPA-2在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻的推理行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}s_2[0]&=\text{Enc}_2(s_1[0])\\z_2&\sim{}R_2\\\tilde{s}_2[2]&=\text{Pred}_2(s_2[0],z_2)\end{align*}">
</p>

它具备代表性，![](https://latex.codecogs.com/svg.latex?t=1,t=2) 等后续时刻的JEPA-2完整推理行为与之同构，后文不再分别赘述。

### 2.2、JEPA-2在 ![](https://latex.codecogs.com/svg.latex?t=2) 时刻的训练行为

JEPA-2在 ![](https://latex.codecogs.com/svg.latex?t=2) 时刻的训练行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}s_2[2]&=\text{Enc}_2(s_1[2])\\\text{Loss}_2&=D_2(s_2[2],\tilde{s}_2[2])\end{align*}">
</p>

训练目标即为 ![](https://latex.codecogs.com/svg.latex?\text{Loss}_2) 的最小化，从而实现 ![](https://latex.codecogs.com/svg.latex?\omega_{\text{Enc}_2},\omega_{\text{Pred}_2})（如果 ![](https://latex.codecogs.com/svg.latex?R_2) 是可训练参数化的，而非固定参数化的，则还包括 ![](https://latex.codecogs.com/svg.latex?\omega_{R_2}) ）的迭代优化。

## 3、H-JEPA的本质特征

对于H-JEPA而言，推理与训练每时每刻都在同步进行，不存在纯粹的“推理时段”，也不存在纯粹的“训练时段”。推理与训练是一体化的，彼此深度耦合：推理的结果构成训练的基础，训练的结果支撑更准确的推理。<br>
这构成了H-JEPA的本质特征。

## 4、H-JEPA的内涵

### 4.1、时间尺度差异

JEPA-1与JEPA-2所进行的预测性推理的时间尺度是明显不同的。<br>
JEPA-1根据 ![](https://latex.codecogs.com/svg.latex?s_1[0]) 预测出 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_1[1]) ，而JEPA-2根据 ![](https://latex.codecogs.com/svg.latex?s_2[0]) 预测出时序更靠后的 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_2[2]) 。<br>
特别值得注意的是，同为“抽象语义表征向量”，![](https://latex.codecogs.com/svg.latex?s_2[0]) 是通过 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_2) 对 ![](https://latex.codecogs.com/svg.latex?s_1[0]) 的进一步处理而得到的，这就意味着，![](https://latex.codecogs.com/svg.latex?s_2[0]) 的抽象层次明确高于 ![](https://latex.codecogs.com/svg.latex?s_1[0]) 的抽象层次。<br>
基于 ![](https://latex.codecogs.com/svg.latex?s_2[0]) 的预测在更高层次的抽象语义上进行，并且时间尺度更大，时序跨度更长，其接受反馈修正的时机也就更晚。具体而言就是：![](https://latex.codecogs.com/svg.latex?\tilde{s}_2[2]) 与 ![](https://latex.codecogs.com/svg.latex?\text{Loss}_2) 所对应的 ![](https://latex.codecogs.com/svg.latex?t=2) 时刻与初始时刻 ![](https://latex.codecogs.com/svg.latex?t=0) 的时间差更大。<br>
基于 ![](https://latex.codecogs.com/svg.latex?s_1[0]) 的预测在更低层次的抽象语义上进行，并且时间尺度更小，时序跨度更短，其接受反馈修正的时机也就更早。具体而言就是：![](https://latex.codecogs.com/svg.latex?\tilde{s}_1[1]) 与 ![](https://latex.codecogs.com/svg.latex?\text{Loss}_1) 所对应的 ![](https://latex.codecogs.com/svg.latex?t=1) 时刻与初始时刻 ![](https://latex.codecogs.com/svg.latex?t=0) 的时间差更小。<br>

### 4.2、分层设计的合理性

这种分层设计具备合理性。<br>
对于智能体而言，低层次抽象语义的变化是剧烈的，其反馈修正是频繁的，这导致其只适合于短期预测，并不适合于长期预测。<br>
例如，智能体可以根据方向盘与踏板上的提议动作序列，对“行驶轨迹细节”这种低层次抽象语义进行准确的短期预测，但其长期预测对智能体而言是困难的，因为这依赖于“其他车辆、交通信号灯、行人以及其他那些在某种程度上不可被预测的外部事件”。<br>
对于智能体而言，高层次抽象语义的变化是平缓的，其反馈修正并不频繁，在某些情况下高层次抽象语义甚至是保持不变的，这使其适合于长期预测。<br>
例如，“到达目的地的大致时间范围”这种省略了大量具体细节的高层次抽象语义（更“粗糙”的世界状态感知）的长期预测，对智能体而言是容易的。<br>
原论文作者Yann LeCun明确主张：“在多个抽象层次上表征世界状态序列的能力，构成了智能行为的实质（essential to intelligent behavior）。”
<br><br><br><br><br><br><br><br><br><br>

# 分层式最优动作序列规划

分层式最优动作序列规划（Hierarchical Planning）意即H-JEPA在最优动作序列规划中的应用形态。后文将其简称为H-Planning。<br>
设H-Planning在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻接收到 ![](https://latex.codecogs.com/svg.latex?x[0]) 。

## 1、JEPA-2与Actor-2的协作

### 1.1、形式化描述

设JEPA-2的编码器为 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_2) ，预测器为 ![](https://latex.codecogs.com/svg.latex?\text{Pred}_2) ，提议“动作”序列为 ![](https://latex.codecogs.com/svg.latex?a_2[2],a_2[4]) ，评价器为 ![](https://latex.codecogs.com/svg.latex?\text{C}_2) 。<br>
其中：<br>
1、编码器属于感知模块。<br>
2、预测器属于世界模型模块。<br>
3、评价器属于代价模块。<br>
4、提议“动作”序列是由执行模块（在本节实例中具体指第2个Actor层，简称Actor-2，注意与JEPA-2区分）提出的。<br>
JEPA-2在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?s_1[0]=\text{Enc}_1(x[0])) 进一步处理，其感知行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s_2[0]=\text{Enc}_2(s_1[0]).">
</p>

JEPA-2在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?t=2) 时刻世界状态的预测行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s_2[2]=\text{Pred}_2(s_2[0],a_2[2]).">
</p>

JEPA-2在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?t=4) 时刻世界状态、代价的预测行为分别可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}s_2[4]&=\text{Pred}_2(s_2[2],a_2[4])\\&\text{C}_2(s_2[4])\end{align*}">
</p>

### 1.2、H-JEPA与H-Planning的关系澄清

1、H-JEPA在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻的目标在于：使当前时刻 ![](https://latex.codecogs.com/svg.latex?t=0) 下所预测的 ![](https://latex.codecogs.com/svg.latex?t=i) 未来时刻世界状态 ![](https://latex.codecogs.com/svg.latex?\tilde{s}[i]) 趋同于真实世界状态 ![](https://latex.codecogs.com/svg.latex?s[i]) ，使它们的散度 ![](https://latex.codecogs.com/svg.latex?D(s[i],\tilde{s}[i])) 实现最小化。H-JEPA只有唯一一个用途：为当前时刻提供准确的未来世界状态预测。<br>
2、H-planning在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻的目标在于：使未来时刻 ![](https://latex.codecogs.com/svg.latex?t=i) 的预测代价 ![](https://latex.codecogs.com/svg.latex?\text{C}(s[i])) 实现最小化（在本节实例中具体指 ![](https://latex.codecogs.com/svg.latex?\text{C}_2(s_2[4])) 的最小化）。这意味着对 ![](https://latex.codecogs.com/svg.latex?t=0) 到 ![](https://latex.codecogs.com/svg.latex?t=i) 这一时间段内所要执行的动作序列的审慎规划。<br>
尤其需要指出的是，H-Planning的质量优劣恰恰完全取决于未来世界状态预测的准确与否。只有在未来世界状态预测足够准确的时候，动作序列的优化方向与收敛状态才是正确的。<br>
换句话说，H-JEPA构成了H-Planning的根本前提，为H-Planning提供了不可或缺的预测能力底座。低水平的H-JEPA必然导致错误频出的H-Planning，使其失去真实物理场景中的使用价值。

### 1.3、JEPA-2与Actor-2的协作成果

![](https://latex.codecogs.com/svg.latex?\text{C}_2(s_2[4])) 的最小化过程，就是最优“动作”序列规划的过程，其细节不再赘述。设规划出的最优“动作”序列为 ![](https://latex.codecogs.com/svg.latex?\check{a}_2[2],\check{a}_2[4]) 。<br>
“动作”一词带有双引号，其内涵将在**下一节的第2.2小节**当中进行说明。

## 2、JEPA-1与Actor-1的协作

### 2.1、形式化描述

设JEPA-1的编码器为 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_1) ，预测器为 ![](https://latex.codecogs.com/svg.latex?\text{Pred}_1) ，提议动作序列为 ![](https://latex.codecogs.com/svg.latex?a_1[0],a_1[1],a_1[2],a_1[3]) ，评价器为 ![](https://latex.codecogs.com/svg.latex?\text{C}_1) 。<br>
其中：<br>
1、编码器属于感知模块。<br>
2、预测器属于世界模型模块。<br>
3、评价器属于代价模块。<br>
4、提议动作序列是由执行模块（在本节实例中具体指第1个Actor层，简称Actor-1，注意与JEPA-1区分）提出的。<br>
JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻的感知行为、在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻对 ![](https://latex.codecogs.com/svg.latex?t=1) 时刻世界状态的预测行为分别可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}s_1[0]&=\text{Enc}_1(x[0])\\s_1[1]&=\text{Pred}_1(s_1[0],a_1[0])\end{align*}">
</p>

JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?t=2) 时刻世界状态的预测行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s_1[2]=\text{Pred}_1(s_1[1],a_1[1]).">
</p>

JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，专门针对预测出的 ![](https://latex.codecogs.com/svg.latex?s_1[2]) ，还存在一个特殊行为，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[2])=D(\check{a}_2[2],s_1[2]).">
</p>

JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?t=3) 时刻世界状态的预测行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s_1[3]=\text{Pred}_1(s_1[2],a_1[2]).">
</p>

JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?t=4) 时刻世界状态的预测行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s_1[4]=\text{Pred}_1(s_1[3],a_1[3]).">
</p>

JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，专门针对预测出的 ![](https://latex.codecogs.com/svg.latex?s_1[4]) ，同样存在一个特殊行为，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[4])=D(\check{a}_2[4],s_1[4]).">
</p>

### 2.2、针对 ![](https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[2])) 与 ![](https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[4])) 的解释

现在明确指出：JEPA-2与Actor-2所协作规划出的最优“动作” ![](https://latex.codecogs.com/svg.latex?\check{a}_2[2],\check{a}_2[4]) 并非真实的物理动作。

#### 2.2.1、![](https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[2]))

![](https://latex.codecogs.com/svg.latex?\check{a}_2[2]) 本质上构成了 ![](https://latex.codecogs.com/svg.latex?s_1[2]) 所要满足的某种“条件”（a condition to be satisfied），也可以理解为专门给 ![](https://latex.codecogs.com/svg.latex?s_1[2]) 设定的需要让它达成的某种“标准”。<br>
![](https://latex.codecogs.com/svg.latex?s_1[2]) 违背这种条件/标准的程度，是用

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[2])=D(\check{a}_2[2],s_1[2])">
</p>

来衡量的，其中 ![](https://latex.codecogs.com/svg.latex?D(\cdot,\cdot)) 表示某种“违背程度衡量函数”。

#### 2.2.2、![](https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[4]))

![](https://latex.codecogs.com/svg.latex?\check{a}_2[4]) 同理，它本质上构成了 ![](https://latex.codecogs.com/svg.latex?s_1[4]) 所要满足的某种条件，也可以理解为专门给 ![](https://latex.codecogs.com/svg.latex?s_1[4]) 设定的需要让它达成的某种标准。<br>
![](https://latex.codecogs.com/svg.latex?s_1[4]) 违背这种条件/标准的程度，是用

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[4])=D(\check{a}_2[4],s_1[4])">
</p>

来衡量的。

#### 2.2.3、“动作”仅仅是下层要满足的条件

![](https://latex.codecogs.com/svg.latex?\check{a}_2[2],\check{a}_2[4]) 也称为“动作的中间层词汇”（the intermediate vocabulary of actions）。<br>
原论文作者Yann LeCun提到：“动作仅仅是下层要满足的条件这一想法，在控制理论当中实际上并不新鲜。”

### 2.3、JEPA-1与Actor-1的协作成果

![](https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[2])) 与 ![](https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[4])) 的同时最小化过程，就是最优动作序列规划的过程，其细节不再赘述。规划出的最优动作序列 ![](https://latex.codecogs.com/svg.latex?\check{a}_1[0],\check{a}_1[1],\check{a}_1[2],\check{a}_1[3]) 均为真实的物理动作。

## 3、总结

总而言之，<br>
1、位于高层次的JEPA-2与Actor-2协作进行长期预测，定义长远目标，并根据长远目标对高层次“动作”序列进行规划，规划出的高层次最优“动作”序列将为低层次动作序列的规划提供指导；<br>
2、位于低层次的JEPA-1与Actor-1协作进行短期预测，仅在高层次最优“动作”序列的指导下，对低层次动作序列进行规划，不涉及对长远目标的访问。<br>
这是一种自上而下式的规划。<br>
不过，原论文作者Yann LeCun认为，“联合优化高层次与低层次动作序列可能更有利”。在联合式规划的情形下，低层次动作序列的规划也会在某种程度上接受长远目标的指导，而不再仅仅是接受高层次最优“动作”序列的指导。<br>
以上内容聚焦于仅具备两层结构、仅涉及五个时间步（从 ![](https://latex.codecogs.com/svg.latex?t=0) 到 ![](https://latex.codecogs.com/svg.latex?t=4) ）的H-Planning，但它所体现的概念可以直接迁移到具备多层结构、涉及更大时间尺度的H-Planning。
<br><br><br><br><br><br><br><br><br><br>

# 不确定性环境中的H-Planning

在后文中，不确定性环境中的H-Planning将被简称为H-zPlanning，其中小写字母 ![](https://latex.codecogs.com/svg.latex?z) 表示潜变量 ![](https://latex.codecogs.com/svg.latex?z)（用于表征不确定性）。<br>
相比H-Planning，H-zPlanning引入了潜变量采样器，用于对环境中的不确定性因素进行建模。<br>
设H-zPlanning在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻接收到 ![](https://latex.codecogs.com/svg.latex?x[0]) 。

## 1、JEPA-2与Actor-2的协作

### 1.1、形式化描述

设JEPA-2的编码器为 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_2) ，预测器为 ![](https://latex.codecogs.com/svg.latex?\text{Pred}_2) ，潜变量采样器为 ![](https://latex.codecogs.com/svg.latex?R_2) ，提议“动作”序列为 ![](https://latex.codecogs.com/svg.latex?a_2[2],a_2[4]) ，评价器为 ![](https://latex.codecogs.com/svg.latex?\text{C}_2) 。<br>
其中：<br>
1、编码器属于感知模块。<br>
2、预测器属于世界模型模块。<br>
3、潜变量采样器可以被视为世界模型模块的可选增强组件，因此认为潜变量采样器同样属于世界模型模块。<br>
4、评价器属于代价模块。<br>
5、提议“动作”序列是由执行模块（即Actor-2）提出的。<br>
JEPA-2在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?s_1[0]=\text{Enc}_1(x[0])) 进一步处理，其感知行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s_2[0]=\text{Enc}_2(s_1[0]).">
</p>

JEPA-2在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?t=2) 时刻世界状态的预测行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?z_2[2]\sim{}R_2,\quad{}s_2[2]=\text{Pred}_2(s_2[0],a_2[2],z_2[2])">
</p>

JEPA-2在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?t=4) 时刻世界状态的预测行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?z_2[4]\sim{}R_2,\quad{}s_2[4]=\text{Pred}_2(s_2[2],a_2[4],z_2[4])">
</p>

长远目标即为 ![](https://latex.codecogs.com/svg.latex?\text{C}_2(s_2[4])) 的最小化。

### 1.2、JEPA-2与Actor-2的协作成果

为了简化叙述，默认采用自上而下式规划。<br>
![](https://latex.codecogs.com/svg.latex?\text{C}_2(s_2[4])) 的最小化过程，就是最优“动作”序列规划的过程，其细节不再赘述。设规划出的最优“动作”序列为 ![](https://latex.codecogs.com/svg.latex?\check{a}_2[2],\check{a}_2[4]) 。

## 2、JEPA-1与Actor-1的协作

### 2.1、形式化描述

设JEPA-1的编码器为 ![](https://latex.codecogs.com/svg.latex?\text{Enc}_1) ，预测器为 ![](https://latex.codecogs.com/svg.latex?\text{Pred}_1) ，潜变量采样器为 ![](https://latex.codecogs.com/svg.latex?R_1) ，提议动作序列为 ![](https://latex.codecogs.com/svg.latex?a_1[0],a_1[1],a_1[2],a_1[3]) ，评价器为 ![](https://latex.codecogs.com/svg.latex?\text{C}_1) 。<br>
其中：<br>
1、编码器属于感知模块。<br>
2、预测器、潜变量采样器同属于世界模型模块。<br>
3、评价器属于代价模块。<br>
4、提议动作序列是由执行模块（即Actor-1）提出的。<br>
JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻的感知行为、在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻对 ![](https://latex.codecogs.com/svg.latex?t=1) 时刻世界状态的预测行为分别可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}s_1[0]&=\text{Enc}_1(x[0])\\z_1[0]\sim{}R_1,\quad{}s_1[1]&=\text{Pred}_1(s_1[0],a_1[0],z_1[0])\end{align*}">
</p>

JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?t=2) 时刻世界状态的预测行为、条件满足程度评估行为分别可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}z_1[1]\sim{}R_1,\quad{}s_1[2]&=\text{Pred}_1(s_1[1],a_1[1],z_1[1])\\\text{C}_1(s_1[2])&=D(\check{a}_2[2],s_1[2])\end{align*}">
</p>

JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?t=3) 时刻世界状态的预测行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?z_1[2]\sim{}R_1,\quad{}s_1[3]=\text{Pred}_1(s_1[2],a_1[2],z_1[2]).">
</p>

JEPA-1在 ![](https://latex.codecogs.com/svg.latex?t=0) 时刻，对 ![](https://latex.codecogs.com/svg.latex?t=4) 时刻世界状态的预测行为、条件满足程度评估行为分别可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}z_1[3]\sim{}R_1,\quad{}s_1[4]&=\text{Pred}_1(s_1[3],a_1[3],z_1[3])\\\text{C}_1(s_1[4])&=D(\check{a}_2[4],s_1[4])\end{align*}">
</p>

### 2.3、JEPA-1与Actor-1的协作成果

![](https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[2])) 与 ![](https://latex.codecogs.com/svg.latex?\text{C}_1(s_1[4])) 的同时最小化过程，就是最优动作序列规划的过程，其细节不再赘述。规划出的最优动作序列 ![](https://latex.codecogs.com/svg.latex?\check{a}_1[0],\check{a}_1[1],\check{a}_1[2],\check{a}_1[3]) 均为真实的物理动作。
<br><br><br><br><br><br><br><br><br><br>

# 键值联想记忆（key-value associative memory）

设查询（query）向量为 ![](https://latex.codecogs.com/svg.latex?q\in\mathbb{R}^{d_{qk}}) 。<br>
针对联想记忆神经网络而言，![](https://latex.codecogs.com/svg.latex?q\in\mathbb{R}^{d_{qk}}) 可以理解为“内存地址”，但实质上是一种外部访问量，其身份可以理解为智能体对自身的联想记忆神经网络所发起的各类记忆行为的载体数据。<br>
设键（key）矩阵为 ![](https://latex.codecogs.com/svg.latex?K\in\mathbb{R}^{d_{qk}\times{}n}) ，值（value）矩阵为 ![](https://latex.codecogs.com/svg.latex?V\in\mathbb{R}^{d_v\times{}n}) 。<br>
键矩阵的第 ![](https://latex.codecogs.com/svg.latex?i) 列向量 ![](https://latex.codecogs.com/svg.latex?k_i\in\mathbb{R}^{d_{qk}}) 与值矩阵的第 ![](https://latex.codecogs.com/svg.latex?i) 列向量 ![](https://latex.codecogs.com/svg.latex?v_i\in\mathbb{R}^{d_v}) 是一一配对的，它们组成 ![](https://latex.codecogs.com/svg.latex?(k_i,v_i)) 。<br>
1、键矩阵与值矩阵其实就构成了联想记忆神经网络的实质。<br>
2、![](https://latex.codecogs.com/svg.latex?(k_i,v_i)) 配对其实就表征着智能体针对世界中某个实体（记为实体 ![](https://latex.codecogs.com/svg.latex?i) ）的主观记忆内容。<br>
3、键/值矩阵的列向量个数 ![](https://latex.codecogs.com/svg.latex?n) 其实就表示当前的记忆容量大小（可以增长）。

## 1、记忆检索行为

记忆检索行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\tilde{w}_i=\text{match}(q,k_i)\in\mathbb{R},\quad\tilde{w}=\text{Match}(q,K)\in\mathbb{R}^n">
</p>

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?w=\text{Normalize}(\tilde{w})\in\mathbb{R}^n,\quad{}w_i\in\mathbb{R}">
</p>

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\text{Memory}(q)=\sum_{i=1}^{n}w_iv_i\in\mathbb{R}^{d_v}">
</p>

其中：<br>
1、![](https://latex.codecogs.com/svg.latex?\tilde{w}_i\in\mathbb{R}) 表示 ![](https://latex.codecogs.com/svg.latex?q\in\mathbb{R}^{d_{qk}}) 与 ![](https://latex.codecogs.com/svg.latex?k_i\in\mathbb{R}^{d_{qk}}) 的匹配程度，进而作为 ![](https://latex.codecogs.com/svg.latex?v_i\in\mathbb{R}^{d_v}) 的未归一化权重。<br>
2、![](https://latex.codecogs.com/svg.latex?w\in\mathbb{R}^n) 是 ![](https://latex.codecogs.com/svg.latex?\tilde{w}\in\mathbb{R}^n) 的归一化形式，![](https://latex.codecogs.com/svg.latex?w_i\in\mathbb{R}) 就是 ![](https://latex.codecogs.com/svg.latex?v_i\in\mathbb{R}^{d_v}) 的权重。<br>
3、![](https://latex.codecogs.com/svg.latex?\text{Normalize}(\cdot)) 函数针对分量 ![](https://latex.codecogs.com/svg.latex?\tilde{w}_i\in\mathbb{R}) 的变换为 ![](https://latex.codecogs.com/svg.latex?\text{normalize}(\cdot)) ，![](https://latex.codecogs.com/svg.latex?w_i=\text{normalize}(\tilde{w}_i)) 的具体实例可以是

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?w_i=\frac{\exp(\tilde{w}_i)}{\gamma+\sum_{j=1}^n\exp(\tilde{w}_j)},">
</p>

其中 ![](https://latex.codecogs.com/svg.latex?\gamma>0) 为超参数。<br>
4、![](https://latex.codecogs.com/svg.latex?\text{Memory}(q)\in\mathbb{R}^{d_v}) 就是记忆检索结果。

## 2、记忆更新行为

设 ![](https://latex.codecogs.com/svg.latex?r\in\mathbb{R}^{d_v}) 是一个用于指导联想记忆神经网络更新（具体而言是 ![](https://latex.codecogs.com/svg.latex?V\in\mathbb{R}^{d_v\times{}n}) 的更新，此阶段 ![](https://latex.codecogs.com/svg.latex?K\in\mathbb{R}^{d_{qk}\times{}n}) 不会参与更新）的外部访问量，它与 ![](https://latex.codecogs.com/svg.latex?q\in\mathbb{R}^{d_{qk}}) 是配套的，组成 ![](https://latex.codecogs.com/svg.latex?(q,r)) 。<br>
记忆更新行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\tilde{w}_i=\text{match}(q,k_i)\in\mathbb{R},\quad\tilde{w}=\text{Match}(q,K)\in\mathbb{R}^n">
</p>

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?w=\text{Normalize}(\tilde{w})\in\mathbb{R}^n,\quad{}w_i\in\mathbb{R}">
</p>

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\check{v}_i=\text{update}(v_i,r,w_i)\in\mathbb{R}^{d_v},\quad\check{V}=\text{Update}(V,r,w)\in\mathbb{R}^{d_v\times{}n}">
</p>

其中：<br>
1、![](https://latex.codecogs.com/svg.latex?v_i\in\mathbb{R}^{d_v}) 表征旧记忆内容，![](https://latex.codecogs.com/svg.latex?\check{v}_i\in\mathbb{R}^{d_v}) 表征新记忆内容。<br>
2、![](https://latex.codecogs.com/svg.latex?\check{v}_i=\text{update}(v_i,r,w_i)) 的具体实例可以是

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\check{v}_i=w_i\cdot{}r+(1-w_i)\cdot{}v_i,">
</p>

当 ![](https://latex.codecogs.com/svg.latex?w_i\to{}0) 时，![](https://latex.codecogs.com/svg.latex?\check{v}_i\to{}v_i) ，这表示 ![](https://latex.codecogs.com/svg.latex?r\in\mathbb{R}^{d_v}) 与 ![](https://latex.codecogs.com/svg.latex?v_i\in\mathbb{R}^{d_v}) 几乎无关（相关程度由分别对应的 ![](https://latex.codecogs.com/svg.latex?q\in\mathbb{R}^{d_{qk}}) 与 ![](https://latex.codecogs.com/svg.latex?k_i\in\mathbb{R}^{d_{qk}}) 的匹配程度衡量），记忆内容几乎保持不变；<br>
当 ![](https://latex.codecogs.com/svg.latex?w_i\to{}1) 时，![](https://latex.codecogs.com/svg.latex?\check{v}_i\to{}r) ，这表示 ![](https://latex.codecogs.com/svg.latex?r\in\mathbb{R}^{d_v}) 与 ![](https://latex.codecogs.com/svg.latex?v_i\in\mathbb{R}^{d_v}) 高度相关，记忆内容几乎被完全覆写。

## 3、记忆新增行为

记忆新增行为可以形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\forall\tilde{w}_i=\text{match}(q,k_i)<T">
</p>

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?K\in\mathbb{R}^{d_{qk}\times{}n}\rightarrow\check{K}\in\mathbb{R}^{d_{qk}\times(n+1)},\quad\Bigl[k_1,\cdots,k_n\Bigr]\rightarrow\Bigl[k_1,\cdots,k_n,q\Bigr]">
</p>

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?V\in\mathbb{R}^{d_v\times{}n}\rightarrow\check{V}\in\mathbb{R}^{d_v\times(n+1)},\quad\Bigl[v_1,\cdots,v_n\Bigr]\rightarrow\Bigl[v_1,\cdots,v_n,r\Bigr]">
</p>

也就是说，如果 ![](https://latex.codecogs.com/svg.latex?r\in\mathbb{R}^{d_v}) 与值矩阵的任意一个列向量 ![](https://latex.codecogs.com/svg.latex?v_i\in\mathbb{R}^{d_v}) 的相关程度均小于阈值 ![](https://latex.codecogs.com/svg.latex?T) ，那么：<br>
1、![](https://latex.codecogs.com/svg.latex?q\in\mathbb{R}^{d_{qk}}) 将作为新增的键向量，添加到键矩阵 ![](https://latex.codecogs.com/svg.latex?K\in\mathbb{R}^{d_{qk}\times{}n}) 沿列序号方向的末尾，使之更新为 ![](https://latex.codecogs.com/svg.latex?\check{K}\in\mathbb{R}^{d_{qk}\times(n+1)}) 。<br>
2、同时，![](https://latex.codecogs.com/svg.latex?r\in\mathbb{R}^{d_v}) 将作为配套新增的值向量，添加到值矩阵 ![](https://latex.codecogs.com/svg.latex?V\in\mathbb{R}^{d_v\times{}n}) 沿列序号方向的末尾，使之更新为 ![](https://latex.codecogs.com/svg.latex?\check{V}\in\mathbb{R}^{d_v\times(n+1)}) 。<br>
换句话说，如果 ![](https://latex.codecogs.com/svg.latex?r\in\mathbb{R}^{d_v}) 与值矩阵的任意一个列向量 ![](https://latex.codecogs.com/svg.latex?v_i\in\mathbb{R}^{d_v}) 的相关程度均小于阈值 ![](https://latex.codecogs.com/svg.latex?T) ，那么 ![](https://latex.codecogs.com/svg.latex?(q,r)) 配对将添加到联想记忆神经网络当中，成为其中的新增部分；联想记忆神经网络的容量大小也就从原先的 ![](https://latex.codecogs.com/svg.latex?n) 增长到更大的 ![](https://latex.codecogs.com/svg.latex?n+1) 。<br>
![](https://latex.codecogs.com/svg.latex?\text{normalize}(\cdot)) 函数中的超参数 ![](https://latex.codecogs.com/svg.latex?\gamma) 可以作为阈值 ![](https://latex.codecogs.com/svg.latex?T) 的取值。

## 4、记忆更新行为的实例

在智能体的联想记忆神经网络当中，世界中的瓶子、厨房、餐厅这3个实体分别被表征为3个键值配对，即 ![](https://latex.codecogs.com/svg.latex?(k_{\text{bottle}},v_{\text{bottle}}),(k_{\text{kitchen}},v_{\text{kitchen}}),(k_{\text{dining}-\text{room}},v_{\text{dining}-\text{room}})) 。<br>
智能体将一个瓶子从厨房拿到餐厅，那么智能体针对这3个实体的记忆将发生更新，具体而言：<br>
1、在 ![](https://latex.codecogs.com/svg.latex?v_{\text{bottle}}\in\mathbb{R}^{d_v}) 的 ![](https://latex.codecogs.com/svg.latex?d_v) 个分量当中，对“位置”进行编码的那些分量将发生更新。它们在更新前表征“厨房”，在更新后表征“餐厅”。<br>
2、在 ![](https://latex.codecogs.com/svg.latex?v_{\text{kitchen}}\in\mathbb{R}^{d_v}) 的 ![](https://latex.codecogs.com/svg.latex?d_v) 个分量当中，对“内容”进行编码的那些分量将发生更新。它们在更新前表征“包含瓶子”，在更新后不再表征“包含瓶子”。<br>
3、在 ![](https://latex.codecogs.com/svg.latex?v_{\text{dining}-\text{room}}\in\mathbb{R}^{d_v}) 的 ![](https://latex.codecogs.com/svg.latex?d_v) 个分量当中，对“内容”进行编码的那些分量将发生更新。它们在更新前不表征“包含瓶子”，在更新后转而表征“包含瓶子”。<br>
智能体针对那些与“将一个瓶子从厨房拿到餐厅”这一事件无关的其他实体的记忆，不会也不需要发生更新。这体现了某种高效性。

## 5、键-值联想记忆的优势

原论文作者Yann LeCun指出：“传统上，深度学习架构中的模块通过向量或多维数组来进行状态通信。但当被建模对象的状态从一个时间步到下一个时间步期间仅仅发生微小变化时，这往往是一种非常低效的方法。”<br>
他进一步指出：“智能体的典型动作只会修改世界状态的一小部分。如果一个瓶子正从厨房移到餐厅，瓶子、厨房、餐厅的状态将被修改。但世界的其余部分将不受影响。”<br>
因此他主张：“这表明世界状态应该以某种可写内存的形式进行维护。每当事件发生时，只有受事件影响的世界状态内存部分需要更新，而其余部分保持不变。”<br>
键-值联想记忆神经网络恰好可以用于达成这一目的。
<br><br><br><br><br><br><br><br><br><br>

# 潜变量分布（潜变量采样器）![](https://latex.codecogs.com/svg.latex?R)

## 1、形式化描述框架

### 1.1、基本概念

分布有2种类型：能量分布，概率分布。<br>
能量分布不需要强制进行积分归一化，而概率分布需要强制进行积分归一化。<br>
对于能量分布而言，样本的能量越低，它被抽到的可能性就越强；<br>
而对于概率分布而言，样本的概率越高，它被抽到的可能性才会越强。<br>
潜变量分布 ![](https://latex.codecogs.com/svg.latex?R) 是一种能量分布。对于 ![](https://latex.codecogs.com/svg.latex?R) 中某个可能被抽到的样本 ![](https://latex.codecogs.com/svg.latex?z) ，它的能量可以记为 ![](https://latex.codecogs.com/svg.latex?R(z)) 。<br>
![](https://latex.codecogs.com/svg.latex?R(z)) 越小，![](https://latex.codecogs.com/svg.latex?z) 被抽到的可能性越强；<br>
![](https://latex.codecogs.com/svg.latex?R(z)) 越大，![](https://latex.codecogs.com/svg.latex?z) 被抽到的可能性越弱。

### 1.2、水平集

定义潜变量 ![](https://latex.codecogs.com/svg.latex?z) 的水平集（level set）![](https://latex.codecogs.com/svg.latex?\mathcal{Z}_h) ，也就是使得 ![](https://latex.codecogs.com/svg.latex?R(z)) 小于阈值 ![](https://latex.codecogs.com/svg.latex?h) 的那些 ![](https://latex.codecogs.com/svg.latex?z) 所组成的集合。<br>
![](https://latex.codecogs.com/svg.latex?\mathcal{Z}_h) 形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{Z}_h=\{z:R(z)<h\}.">
</p>

当 ![](https://latex.codecogs.com/svg.latex?z) 在 ![](https://latex.codecogs.com/svg.latex?\mathcal{Z}_h) 上变化时，输出的 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 会在 ![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 集合上同步变化。![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 可以理解为 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 的水平集。<br>
![](https://latex.codecogs.com/svg.latex?\mathcal{Y}_h) 形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\mathcal{Y}_h=\{\tilde{s}_y:\tilde{s}_y=\text{Pred}(s_x,z),\forall{}z\in\mathcal{Z}_h\}.">
</p>

## 2、能量分布与概率分布的等价关系

能量分布与概率分布可以通过吉布斯-玻尔兹曼公式（Gibbs-Boltzmann formula）建立等价关系。<br>
在能量分布 ![](https://latex.codecogs.com/svg.latex?R) 实例当中，这具体指

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?P(z)=\frac{\exp\Bigl[-R(z)\Bigr]}{\int{}dz^\prime\cdot\exp\Bigl[-R(z^\prime)\Bigr]}=\frac{\exp\Bigl[-R(z)\Bigr]}{\int_{\mathcal{Z}}\exp\Bigl[-R(z^\prime)\Bigr]},">
</p>

其中：<br>
1、![](https://latex.codecogs.com/svg.latex?\mathcal{Z}) 表示所有可能被抽到的潜变量所组成的潜变量分布空间。<br>
2、分子中的 ![](https://latex.codecogs.com/svg.latex?z) 是特指某个潜变量；而分母中的 ![](https://latex.codecogs.com/svg.latex?z^\prime) 泛指潜变量分布空间中任意一个可能被抽到的潜变量。<br>
3、分母整体意即 ![](https://latex.codecogs.com/svg.latex?\exp\Bigl[-R(z^\prime)\Bigr]) 函数在整个潜变量分布空间中的积分。<br>
以上公式实例构成能量分布 ![](https://latex.codecogs.com/svg.latex?R) 与概率分布 ![](https://latex.codecogs.com/svg.latex?P) 的等价关系。

## 3、![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 所服从的条件分布对应的条件概率密度函数

![](https://latex.codecogs.com/svg.latex?\tilde{s}_y,s_x,z) 三者的关系被形式化描述为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\tilde{s}_y=\text{Pred}(s_x,z).">
</p>

![](https://latex.codecogs.com/svg.latex?z) 是从分布当中随机抽取得到的，因此，![](https://latex.codecogs.com/svg.latex?z) 内在地具备随机性。<br>
尽管 ![](https://latex.codecogs.com/svg.latex?\text{Pred}(\cdot,\cdot)) 函数的映射过程是确定的（不具备随机性），但由于作为函数输入之一的 ![](https://latex.codecogs.com/svg.latex?z) 具备随机性，函数输出的 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 也就内在地具备随机性。<br>
这意味着，![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 也会服从某个分布，它在本质上也是从某个分布当中随机抽取得到的。

### 3.1、![](https://latex.codecogs.com/svg.latex?P(\tilde{s}_y\mid{}s_x,z))

实际上，如果将 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_x,z) 视为条件变量，![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 视为事件变量，那么 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 所服从的条件概率分布对应的条件概率密度函数可以朴素而直接地表示为

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?P(\tilde{s}_y\mid{}s_x,z)=\delta(\tilde{s}_y-\text{Pred}(s_x,z)),">
</p>

其中，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\delta(v-a)=\begin{cases}+\infty,&v=a\\0,\quad{}&v\neq{}a\end{cases}\quad\quad\quad\quad\int{}dv\cdot\delta(v-a)=1.">
</p>

现在希望将其转化为 ![](https://latex.codecogs.com/svg.latex?P(\tilde{s}_y\mid{}s_x)) ，去掉随机成分 ![](https://latex.codecogs.com/svg.latex?z) 。

### 3.2、更一般的 ![](https://latex.codecogs.com/svg.latex?P(y\mid{}x,z),P(y\mid{}x))

以下给出更一般的问题形式化框架。<br>
设 ![](https://latex.codecogs.com/svg.latex?z) 服从某个分布，其概率密度函数为 ![](https://latex.codecogs.com/svg.latex?P(z)) ，并且与 ![](https://latex.codecogs.com/svg.latex?x) 相互独立。<br>
目标是将 ![](https://latex.codecogs.com/svg.latex?P(y\mid{}x,z)) 转化为 ![](https://latex.codecogs.com/svg.latex?P(y\mid{}x)) 。<br>
首先，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?P(y\mid{}x)=\int{}dz\cdot{}P(y,z\mid{}x)">
</p>

属于边缘化（marginalization）操作，是成立的。<br>
根据

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?P(y,z\mid{}x)=\frac{P(x,y,z)}{P(x)}=\frac{P(x,y,z)}{P(x,z)}\cdot\frac{P(x,z)}{P(x)}=P(y\mid{}x,z)\cdot{}P(z\mid{}x),">
</p>

可以进一步推知

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}P(y\mid{}x)&=\int{}dz\cdot{}P(y,z\mid{}x)\\&=\int{}dz\cdot{}P(y\mid{}x,z)\cdot{}P(z\mid{}x).\end{align*}">
</p>

由于 ![](https://latex.codecogs.com/svg.latex?z) 与 ![](https://latex.codecogs.com/svg.latex?x) 相互独立，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?P(x,z)=P(x)\cdot{}P(z),">
</p>

因此，

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?P(z\mid{}x)=\frac{P(x,z)}{P(x)}=\frac{P(x)\cdot{}P(z)}{P(x)}=P(z),">
</p>

从而

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}P(y\mid{}x)&=\int{}dz\cdot{}P(y\mid{}x,z)\cdot{}P(z\mid{}x)\\&=\int{}dz\cdot{}P(y\mid{}x,z)\cdot{}P(z).\end{align*}">
</p>

由此，完成了 ![](https://latex.codecogs.com/svg.latex?P(y\mid{}x,z)) 到 ![](https://latex.codecogs.com/svg.latex?P(y\mid{}x)) 的转化：让 ![](https://latex.codecogs.com/svg.latex?P(y\mid{}x,z)) 对 ![](https://latex.codecogs.com/svg.latex?z) 取期望，就可以得到 ![](https://latex.codecogs.com/svg.latex?P(y\mid{}x)) 。

### 3.3、![](https://latex.codecogs.com/svg.latex?P(\tilde{s}_y\mid{}s_x))

现在，![](https://latex.codecogs.com/svg.latex?\tilde{s}_y,s_x,z) 可以分别作为上述一般形式中 ![](https://latex.codecogs.com/svg.latex?y,x,z) 的具体实例，此时有

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\begin{align*}P(\tilde{s}_y\mid{}s_x)&=\int{}dz\cdot{}P(\tilde{s}_y\mid{}s_x,z)\cdot{}P(z)\\&=\int{}dz\cdot\delta(\tilde{s}_y-\text{Pred}(s_x,z))\cdot{}P(z)\\&=\int_{\mathcal{Z}}\delta(\tilde{s}_y-\text{Pred}(s_x,z))\cdot{}P(z),\end{align*}">
</p>

此即 ![](https://latex.codecogs.com/svg.latex?\tilde{s}_y) 所服从的条件分布对应的条件概率密度函数。
