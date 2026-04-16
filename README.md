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
训练目标的内涵就在于减小评价器的预测误差。评价器对世界模型模块所给出的预测世界状态的对应代价的准确估计，将为构成预测世界状态影响因素之一的提议动作的优化带来可能。这构成了智能体借助模式2进行提议动作后果预演以及最优动作序列规划（planning）的基础。

## 2、模式2的闭环

在当前时刻 ![](https://latex.codecogs.com/svg.latex?t=0) ，模式2的从感知到动作优化所组成的一个闭环（或称“回合”，episode）可以表述为总共7个步骤：<br>
**1、感知（perception）**：<br>
感知模块给出 ![](https://latex.codecogs.com/svg.latex?s[0]=\text{Enc}(x)) 。<br>
**2、动作提议（action proposal）**：<br>
执行模块提出一个有待后续进行评估/优化的初始动作序列（可以理解为“直觉动作序列”），即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?a[0],\cdots,a[t],\cdots,a[T].">
</p>

**3、模拟世界状态演化（simulation，带有主观性）**：<br>
根据

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s[i+1]=\text{Pred}(s[i],a[i]),">
</p>

世界模型模块针对初始动作序列的每一个提议动作，依次给出对应的世界状态预测，使其组成世界状态预测序列，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?s[1],\cdots,s[t+1],\cdots,s[T+1].">
</p>

**4、预测代价总和评估（evaluation，可以理解为“提议动作后果预演”）**：<br>
根据

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?f[i]=\text{C}(s[i]),">
</p>

评价器针对世界状态预测序列当中的每一个世界状态预测，分别给出对应的代价预测，使其组成代价预测序列，即

<p align="center">
<img src="https://latex.codecogs.com/svg.latex?\text{C}(s[1]),\cdots,\text{C}(s[t+1]),\cdots,\text{C}(s[T+1]),">
</p>

并计算其总和，即

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

的其中一个加性（additive）贡献项；![](https://latex.codecogs.com/svg.latex?F(T)) 对 ![](https://latex.codecogs.com/svg.latex?a[t]) 的梯度实际上仅仅指 ![](https://latex.codecogs.com/svg.latex?\text{C}(s[t+1])) 对 ![](https://latex.codecogs.com/svg.latex?a[t]) 的梯度，而与由其他那些提议动作自变量所产生的加性贡献项无关。
