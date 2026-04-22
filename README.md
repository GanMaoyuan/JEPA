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
<img src="https://latex.codecogs.com/svg.latex?\sqrt{\text{Var}(C)}=\sqrt{\frac{1}{N}\sum_{i=1}^{N}(c_i-\mu_C)},">
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
