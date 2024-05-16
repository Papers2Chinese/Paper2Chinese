# 题目：[CATNIPS: Collision Avoidance Through Neural Implicit Probabilistic Scenes](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=10494911)  
## CATNIPS:通过神经隐式概率场景避免碰撞
**作者：Timothy Chen； Preston Culbertson；  Mac Schwager** 

**源码链接：** https://github.com/chengine/catnips
****
# 摘要
我们引入了一种将神经辐射场（NeRF）转换为等效的泊松点过程（PPP）的转换。此PPP转换允许严格量化NeRF中的不确定性，特别是用于计算机器人在NeRF环境中导航时的碰撞概率。PPP是概率占用网格向连续体积的推广，并且是潜在辐射场模型的基础。在此PPP表示的基础上，我们提出了一种机会受限的轨迹优化方法，用于在NeRFs中安全地导航机器人。我们的方法依赖于称为概率不安全机器人区域的体素表示，该表示将机会约束与NeRF模型空间融合，以促进快速轨迹优化。然后，我们结合基于图的搜索与基于样条的轨迹优化，以产生通过NeRF的机器人轨迹，这些轨迹保证满足特定用户的碰撞概率。我们通过模拟和硬件实验验证了我们的机会受限规划方法，并显示其性能优于以前的NeRF环境轨迹规划工作。
# 关键词
- 碰撞规避
- 神经辐射场(NeRFs)
- 机器人安全
- 视觉导航

# I. 引言

从船上传感器（如RGB(-D)摄像机、激光雷达或触摸传感器）构建环境模型是任何自主系统面临的基本挑战。最近，神经辐射场（NeRFs）[1] 作为一种有前景的三维场景表示方法出现，它在多种机器人领域都有潜在的应用，包括同步定位与地图构建（SLAM）[2]、姿态估计[3]、[4]、强化学习[5]和抓取[6]。NeRFs相比于传统的场景表示方法提供了几个潜在的好处：它们可以使用单目RGB图像进行训练，提供障碍物几何形状的连续表示，并且在内存效率方面表现出色，尤其是考虑到它们渲染的真实感质量。使用当前的实现[7]、[8]，NeRFs可以在几秒钟内仅使用单目摄像机捕获的RGB图像进行训练，这使得对于机器人系统来说，船上、在线的NeRF训练成为一个可行的选项。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/4f25fa97aca049aba0aab9b280c06851.png#pic_center)


然而，NeRFs并不直接提供有关空间占用的信息，这在将NeRF模型用于安全机器人导航时带来了挑战。在其他三维场景表示中，例如（密封的）三角形网格[9]、占用网格[10]或有符号距离场（SDFs）[11]，占用是定义得很好且易于查询的。然而，NeRFs不允许简单的点占用查询，因为它们通过连续的体积密度场隐式地表示场景几何。因此，将NeRF模型集成到具有数学安全保证的机器人规划器中仍然是一个开放性问题。

为此，我们开发了一个机器人轨迹规划框架，该框架可以生成通过NeRF场景的轨迹，并具有概率安全保证。为此，我们提出了一种将NeRF数学转换为泊松点过程（PPP）的方法，这允许我们严格计算机器人在NeRF场景中移动时的碰撞概率。我们进一步引入了一种新的场表示，称为概率不安全机器人区域（PURR），它将机器人几何形状与NeRF卷积，生成一个三维地图，显示所有机器人位置的碰撞概率低于用户指定的阈值。最后，我们提出了一个快速的机会约束轨迹规划器，使用PURR确保轨迹在用户指定的概率阈值下无碰撞。我们的方法，称为通过神经隐式概率场景的碰撞避免（CATNIPS），可以计算出概率安全的轨迹，速度超过每秒3次。这比现有的基于NeRF的轨迹规划器快得多，后者不提供安全保证[12]。

我们结果背后的主要理论进展是将NeRF转换为PPP的新方法。关于辐射场的现有工作要么忽略了场的底层概率解释，要么将其视为麻烦。一种简单的方法是将NeRF表示转换为更传统的确定性网格或占用表示。我们认为这样的转换计算上是缓慢的，而且它们破坏了下游规划器的任何潜在数学安全保证。例如，生成一个代表密度场水平集的三角形网格（例如，使用marching cubes[13]）需要任意选择一个密度截止值，并将密度场所代表的不确定性简化为二进制占用度量。相比之下，我们的方法直接使用NeRF密度计算严格的碰撞概率。我们提供了模拟研究，以证明我们的规划器生成的轨迹是安全的，但不过过于保守，通过环境。我们将我们的路径与使用基于水平集的环境表示生成的路径和先前工作[12]生成的路径进行对比。我们发现，我们的方法生成的路径比这些基线更直观，更容易调整，因为碰撞是通过违反碰撞概率而不是违反密度的任意水平集来定义的（图1）。我们展示了我们的方法是实时的，在笔记本电脑上以每秒3Hz的速度在线重新规划，与[12]中提出的基于梯度下降的规划器相比，后者需要大约2秒的时间进行重新规划。

本文的其余部分组织如下。在第II节中，我们讨论相关工作。在第III节中，我们回顾了NeRF的背景概念，并在第IV节中推导出NeRF的PPP解释。在第V节中，我们计算了机器人在NeRF环境中的碰撞概率，并在第VI节中展示了我们的轨迹规划算法CATNIPS。第VII节给出了我们的模拟结果，结论在第VIII节中。

# III. 神经辐射场(NeRFs)


在本节中，我们介绍神经辐射场（NeRFs）的数学基础和符号。为了清晰起见，我们使用**粗体**来表示向量变量和输出向量的函数，而非粗体用于表示标量变量、输出标量的函数，以及在某些情况下表示旋转。

NeRF 是一个神经网络，它存储了 3D 环境中的密度和颜色场。它结合了一个可微分的图像渲染模型（通常是一个可微分版本的光线追踪技术），可以从一组具有已知相机姿态的 RGB 图像中进行训练，并能生成从与训练图像不同的相机视角渲染的逼真合成图像。

具体来说，NeRF 是一对函数 $(ρ(p), c(p, d))$。密度函数 $\rho: \mathbb{R}^3 \rightarrow \mathbb{R}_{\geq 0}$ 将一个 3D 位置 $p = (x, y, z)$ 映射到一个非负密度值 $\rho$，该值编码了光射线在该点的微分终止概率。辐射度函数 $c: \mathbb{R}^3 \times \mathbb{R}^2 \rightarrow \mathbb{R}^3$ 将一个 3D 位置 $p = (x, y, z)$ 和一个相机视图方向 $d \in \{ x \in \mathbb{R}^3 \mid \|x\| = 1 \}$（或者参数化为角度向量 $(θ, φ)$ 的2D向量）到一个以 $\mathbb{R}^3$ 中的向量表示的发射RGB颜色 $c$。

在本文中，我们特别关注密度函数 $\rho(p)$ 作为占用的代理，理想情况下在自由空间中应为零，在被占用的空间中应取较大值。我们使用这个 $\rho(p)$ 函数作为规划机器人轨迹的地图表示。我们还定义 $C(o, d) \in [0, 1]^3$ 为当沿着射线 $r(t; o, d)$ 与相机原点 $o$ 和像素方向 $d$ 取期望颜色值时，图像中的渲染像素颜色，其中 $r(t) = o + t \cdot d$。渲染的颜色由下式给出：

$$
C(o, d) = \int_{t_n}^{t_f} ρ(r(t)) e^{-\int_{t_n}^{t} ρ(r(τ))dτ} c(r(t), d) dt
$$

其中，
我们只沿射线从 $t_n$ 到 $t_f$（即近平面和远平面之间）的点进行积分。在实践中，这个积分是使用分层采样的蒙特卡洛积分数值评估的。结果的渲染是可微分的。

一个渲染图像 $I_i$ 然后是与单个相机姿态相关联的像素颜色数组，其中图像 $I_i$ 中的像素 $j$ 的颜色由 $C(o_i, d_{ij})$ 给出，与相机相关的原点 $o_i$（由相机确定）和方向 $d_{ij}$ 计算得出，方向 $d_{ij}$ 是从相机光轴的角偏移量计算得出的。我们表示图像 $I_i$ 中的像素索引集为 $I_i$。相应的真实图像 $\bar{I}_ i$ 是具有颜色 $\bar{C}_ {ij}$ 的像素数组。训练NeRF的数据集 $D$ 由这样一些真实图像组成，它们具有已知的姿态。NeRF的参数通过最小化损失函数来训练：

$$
J(θ) = \frac{1}{|D|} \sum_{i \in D} \frac{1}{|I_i|} \sum_{j in I_i} ||C(o_i, d_{ij}; θ) - \bar{C}_{ij}||^2
$$

其中 $θ$ 是表示密度和辐射度场 $ρ$ 和 $c$ 的神经网络参数，这些参数通过渲染方程（1）参与计算像素颜色 $C(o_i, d_{ij}; θ)$。这种均方误差称为光度误差（或光度损失），并使用例如Pytorch中的标准随机梯度下降工具进行优化。直观地说，目标是训练网络，使得从NeRF生成的合成图像尽可能与指定相机姿态的训练图像相匹配。

虽然需要相机姿态来为每个像素找到 $o_i$ 和 $d_{ij}$ 以训练NeRF，但已经形成了一个标准流程，它采用没有相机姿态的图像，使用经典的从运动结构算法（例如，COLMAP [36]）估计相机姿态，并使用这些姿态监督NeRF训练。最近的方法还联合优化相机姿态和NeRF权重以提高性能 [8]。

因此，在实践中，可以从仅有RGB图像（没有相机姿态）中获得NeRF模型。然而，NeRF的质量和范围受训练图像的质量和覆盖的体积的限制。少量的、分辨率低的、摄影质量差和覆盖范围差的图像将产生质量差的NeRF。大量清晰、高分辨率的图像从丰富的视角将产生高质量的NeRF。我们的目标是准确量化机器人在场景中导航时的碰撞风险，而不考虑训练的NeRF的质量。通过我们的方法，同一3D场景中同一机器人姿态可能在质量差的NeRF中具有高碰撞概率，在高质量NeRF中具有低碰撞概率。碰撞概率本身是NeRF质量在机器人附近的一种表达。

# IV. NeRF密度作为泊松点过程 (PPP)
在本节中，我们展示了如何将NeRF密度场转换为PPP的密度，并且NeRF的颜色和密度场共同产生了一个“标记”PPP [37], [38]。为此，我们证明了NeRF体积渲染方程正是计算标记PPP下预期像素颜色所需的计算。通过时刻匹配预期像素颜色来训练NeRF，可以解释为拟合PPP密度参数。

这种联系是重要的，因为从NeRF密度场派生的PPP使我们能够计算概率量，例如一个给定体积被占用的概率（例如，机器人身体与NeRF发生碰撞的概率），或者NeRF模型中的熵。这也平息了文献中关于NeRF密度是否可以概率解释的争论（它可以），并为其他领域之外的实际应用铺平了道路（例如，在主动传感和主动视图规划中）。简而言之，我们发现NeRF密度编码了场景几何的概率模型，其不确定性可以通过底层PPP严格量化。

## A. 泊松点过程

这里我们回顾了PPP的定义和性质，PPP是一种在连续空间中对随机点集合分布进行建模的随机过程。本节的讨论大部分引用自 [37]，我们推荐读者阅读该文献以获得更详细和严格的处理。

首先，我们回顾一个离散随机变量N，如果它取值在N中，并且其概率质量函数由下式给出，则称其具有参数λ ≥ 0的泊松分布：

$$
\Pr(N = m) = \frac{λ^m e^{-λ}}{m!}
$$

泊松随机变量通常用来模拟固定时间段内离散事件的分布（例如，顾客到达商店的数量），或者在固定空间区域（例如，每日在给定邻里内叫车的数量）的分布。PPP自然地将这一概念扩展到了多维欧几里得空间的任何子集中点的数量分布。


## B. “渲染”一个标记PPP

PPP的强度函数允许有一个无限小的解释： $\lambda(x) \, dx$是过程在以 $x$为中心的无限小体积 $dx$内有一点的概率。这与原始NeRF作者 [1] 对密度场的解释密切相关：“ $\rho(x)$定义了在给定点 $x \in \mathbb{R}^3$的光线终止的无限小概率。” 在本节中，我们将展示体积渲染过程实际上计算了沿给定射线的标记PPP的预期颜色。

我们的主要问题是如何在NeRF中使用的光线追踪（一个1-D过程）与PPP在3-D度量中的占用 $\lambda(x)$之间建立联系。我们考虑由光线追踪扫过的体积（光线投影到单个像素斑块的金字塔）。具体来说，在从标记PPP生成图像中特定像素的颜色时，我们沿像素的射线扩展一个金字塔（见图2），并返回沿射线遇到的第一个点的颜色作为期望值。

更具体地说，对于图像中的每个像素，关联的射线为 $r(o, d)$，我们考虑由长度 $t \in [t_n, t_f]$ 参数化的金字塔 $F(t)$（见图2，红色），其由以下参数化定义：

$$
F(t) \equiv \{o + d\tau + x \hat{n}_x + y \hat{n}_y \mid |x| \leq \frac{w(\tau)}{2}, |y| \leq \frac{h(\tau)}{2}, \tau \in [t_n, t]\}
$$

此处 $\hat{n}_x, \hat{n}_y$ 是与 $d$ 正交的单位向量，构成图像平面中的基， $h, w$ 是金字塔截面的高和宽，分别，从图像平面上的像素大小 $(h_0, w_0)$ 开始。

然而，仍然存在一个建模选择：给定一个固定粒子大小 $V_{ref}$ 与初始PPP相关联，我们如何确定在金字塔的一个给定截面中必须存在多少粒子才能使光线被遮挡？我们说当在深度 $t$ 处的粒子的组合前面积占据金字塔横截面的 $\gamma \in (0, 1]$ 的给定分数时，光线就被遮挡了。

由于金字塔的面积 $A(t)$ 随深度变化，对于固定大小的粒子，需要遮挡光线的粒子数量也会随着 $t$ 变化。然而，正如之前讨论的，重新加权PPP等同于改变粒子大小。因此，在本工作中，我们考虑“无量纲”粒子，其在金字塔上的投影面积恰好为 $\gamma A(t)$（因此只需要一个粒子就可以遮挡光线），通过沿射线相应地重新加权PPP密度。因此，我们说光线在沿射线遇到的第一个“无量纲”粒子的深度处终止。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/b30c01d1e0e94730b5d2c7e4451db724.png#pic_center)

## C. NeRF和PPP渲染的等价性

这使我们得出主要结果。在这里，我们展示了，在对训练光线的分布和NeRF密度及颜色的空间变化进行适当假设的情况下，从NeRF（1）渲染的图像中给定像素的颜色完全等于从具有（缩放的）等于NeRF密度 $\rho$ 的强度和等于NeRF辐射度 $c$ 的标记的PPP渲染的相同像素的预期颜色。然后我们提供了关于 $\lambda$ 和 $\rho$ 之间空间关系的直觉。

假设1（PPP平滑性）：考虑一个PPP $\lambda(x)$ 和颜色标记 $c(x, d)$。我们假设任何球 $B_ε$，其中心为 $x_ε$，的PPP密度的平均值等于球心处的PPP密度值，其中 $\epsilon$ 是能够包含从NeRF场景的近平面投影到远平面上的像素的最小半径球。我们还假设颜色场在任何 $B_ε$ 上近似恒定。具体来说，

$$
\int_{B_ε} \lambda(x) \, dx = \lambda(x_ε) \, V_{B_ε}
$$

并且

$$
\forall (x, y) \in B_ε : \|c(x, d) - c(y, d)\| = 0
$$

其中 $B_ε$ 是能够包含从NeRF场景的近平面投影到远平面上的像素的最小球。

这个假设是合理的，因为当将连续环境编码为像素化图像的观测空间时，会丢失信息。由于相机的分辨率，颜色在单个像素内是恒定的，因此我们没有信息来区分颜色仅在一个像素的长度尺度上变化的生成环境。由于给定像素化图像重建连续几何形状是不适定的，我们将这种平滑性要求视为密度和颜色场的一种正则化先验。

此外，我们对颜色的假设更为严格，因为在实践中，辐射场（即颜色）比密度场更平滑。主要原因是辐射场即使在空旷空间中也有定义，并且因此被允许在表面上平滑变化；另一方面，密度必须在表面上更急剧地变化，以反映底层几何的离散变化。这确实反映在文献中，其中原始NeRF工作 [1] 使用一个较小的网络来模拟颜色，后来的扩展如 [28] 用平滑的低阶球谐函数替换了网络。

假设2（光线冗余）：给定一组从训练图像、姿态和相机内参得出的训练光线数据集，没有两条光线相交。

这是一个弱假设，因为光线相交是零测事件。此外，浮点精度和姿态中的噪声进一步降低了光线相交的可能性。

鉴于这些假设，我们现在陈述我们的主要结果。

命题1（PPP渲染）：考虑一个PPP $\lambda(x)$ 和辐射 $c(x, d)$，并且辐射满足假设1中的（8）。那么，从PPP渲染的像素的预期颜色与形式与原始NeRF论文 [1] 中的渲染方程（1）相匹配。

证明：让我们考虑一个射线 $r(t) = o + t \cdot d$，其中 $t \in [t_n, t_f]$。我们再次考虑相应的金字塔 $F(t)$（6），由射线 $r$ 从 $t_n$ 到 $t$ 扫过的金字塔创建的。

正如我们的渲染模型所讨论的，我们考虑“无量纲”粒子，其投影面积是 $\gamma A(t)$（因此光线被第一个遇到的粒子遮挡）。因此，这些粒子的强度/期望粒子数 $\delta F(t)$ 对于那些遮挡尺寸（ $V_d = \gamma V_{\delta F(t)}$）定义为：

$$
\Lambda (\delta F(t)) = \int_{x \in \delta F(t)} V_{ref} \lambda(x) \frac{\gamma V_{\delta F(t)}}{dx}
$$

= $\frac{δt}{γA(t)δt} \int_{A(t)} A_{ref}δtλ(r(t) + x \hat{n}_x + y \hat{n}_y) dx dy$

如果参考粒子是某个小球体，其大小为 $V_{ref} = A_{ref}δt$。
因此，切片的空洞概率（3）（或者等价地，没有遮挡的概率）是：

$$
\Pr(N(\delta F(t)) = 0) = \exp \left( -\int_{x \in \delta F(t)} V_{ref} \lambda(x) \gamma \frac{V_{\delta F(t)}}{dx} \right)
$$

现在我们考虑事件，其中金字塔的任何切片在给定深度 $t$ 之前都被遮挡了。为此，我们沿其长度将金字塔分成更小的部分。由于PPP的定义，每个部分中粒子的数量是独立的（定义1（ii）），则金字塔直到 $t$ 未被遮挡的概率要求所有部分都没有被遮挡：

$$
\Pr(N(\delta F(t)) = 0) = \prod_{t} \exp \left( -\int_{A(t)} A_{ref} \lambda(r(τ) + x \hat{n}_x + y \hat{n}_y) \gamma \frac{A(t)}{\delta t} dτ \right)
$$

= $\exp \left( -\int_{t_n}^{t} \int_{A(τ)} A_{ref}λ(r(τ) + x \hat{n}_x + y \hat{n}_y) \gamma \frac{A(τ)}{\delta t} dτ \right)$。

在极限中，当截面宽度 $δt$ 趋近于零时，它们变成切片，求和变成积分。因此，金字塔直到 $t$ 被遮挡的概率是：

$$
\Pr(F(t) \text{ 未被遮挡}) = \exp \left( -\int_{t_n}^{t} κ(τ) dτ \right)
$$

对于简化表示，我们此后将表面积分除以切片的缩放面积表示为 $κ(τ)$。

现在让我们考虑一个随机变量 $T_ {min}$，它定义了第一个遮挡切片的距离（在图2中用绿色表示）。我们可以定义这个变量的累积分布函数，对于某个 $t > t_n$， $T_ {min} \leq t$ 作为金字塔 $F(t)$ 被遮挡的概率。因此， $T_{min}$ 的CDF可以使用上述方程定义为：

$$
\Pr(T_{min} \leq t) \equiv F_{T_{min}}(t) = 1 - P (F(t) \text{ 未被遮挡}) = 1 - \exp \left( -\int_{t_n}^{t} κ(τ) dτ \right)
$$

我们可以通过对 $F_{T_{min}}$ 关于 $t$ 求导来计算 $T_{min}$ 的概率密度函数，得到：

$$
f_{T_{min}}(t) = \frac{d}{dt}F_{T_{min}}(t) = κ(t) \exp \left( -\int_{t_n}^{t} κ(τ) dτ \right)
$$

这定义了一个关于未遮挡区域范围的概率分布。

最后，由于假设1，PPP渲染返回的切片颜色等于在 $r(T_{min})$ 处评估的标记函数。因此，我们可以通过计算 $c(r(T_{min}))$ 的期望值来计算PPP渲染的预期颜色，得到：

$$
C(r) = E [c(r(T_{min}))] = \int_{t_f}^{t_n} κ(t) c(r(t)) \exp \left( -\int_{t_n}^{t} κ(τ) dτ \right) dt
$$

PPP预期颜色完全匹配原始NeRF论文 [1] 中的表达式（1），完成证明。 ■

命题2（NeRF-PPP等价性）：考虑一个具有密度 $\rho(x)$ 的NeRF，并让假设2成立。那么NeRF是局部区域平均的PPP。

证明：根据上述证明，我们对所有训练光线做出等价：

$$
\forall r(t; o, d) : ρ(r(t)) = \frac{1}{γA(t)} \int_{A(t)} A_{ref}λ(r(t) + x \hat{n}_x + y \hat{n}_y) dx dy
$$

注意，我们需要假设2因为当两条光线相交时，上述等式的左侧对于这两条光线必然是相同的，然而右侧可能不是。具体来说，两条光线在交点的积分域可能不是相同的，因此右侧的分子和分母对于这两条光线不是相同的。此外，对于 $\gamma = 1$（完全遮挡），密度是切片 $A(t)$ 上粒子的面积加权平均数。

这与 [1], [39] 提出的NeRF密度的直觉相匹配。然而，我们的推导比 [39] 的推导更一般，后者假设了一个恒定面积的金字塔。虽然我们为了说明目的使用了金字塔形的截面，但请注意，我们的推导不假设 $A(t)$ 的形状（例如，矩形、圆形），因此金字塔的形状，只要金字塔可以分解为自身相连的切片。最后，我们的推导提出了一个更一般的渲染方程，因为我们需要假设局部颜色的均匀性来检索（1）。没有这个限制，我们可以推导出一个更具表现力和准确性的渲染方程。此外，我们的推导甚至提出了一个参数 $\gamma$，可以调整以更精确地定义遮挡，也许可以提高渲染的保真度。

推论1：由（7）从假设1, $\rho(x) = \frac{A_ {ref}}{\gamma}\lambda(x)$，其中 $0 < γ \leq 1$，因此NeRF密度通过常数缩放因子 $\frac{A_ {ref}}{\gamma}$ 与等效PPP相关联。

在实践中，我们通常会发现，通过将PPP点云与NeRF渲染的场景叠加（如图3所示），可以清楚地看到NeRF场景和PPP点云之间的几何对应关系。

请注意，尽管我们对NeRF密度进行了局部区域平均以得到PPP，从而在渲染颜色时从线积分变成了体积积分，但在使用学习框架学习ρ时，我们丢失了关于λ场的信息（即，参考粒子 $A_{ref}$ 和遮挡 $\gamma$）。实际上，我们在NeRF中观察到的混叠现象，即NeRF在不同尺度上的失败，可能是由于这种平均方案造成的。Mip-NeRF等作品的成功 [30]，它们不是将像素视为射线的投影，而是将像素视为沿着金字塔评估的体积积分（即，学习λ而不是ρ），使我们有理由相信，直接学习PPP的参数可能会产生更高质量的几何形状。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/e677463b868948b18276918c3e716e10.png#pic_center)

## D. 关于PPP解释的讨论

采用PPP解释NeRF密度的一个重要优势是，它允许我们在推理NeRF时利用PPP的深入研究的属性。具体来说，对于PPP，不确定性度量（例如似然和熵）是定义得很好的。假设我们使用机载激光雷达或深度相机测量环境的点云；即，对于一组射线 $\{r_1, ..., r_k\}$ 我们获得了无噪声的深度测量 $\{d_1, ..., d_k\}$。我们可以使用下式来计算在NeRF下获得这些深度的概率：

$$
\log P(d_1, ..., d_k) = \sum_{i=1}^{k} \left( \log \rho(r_{di}) - \int_{t_n}^{d_i} \rho(r(t)) dt \right)
$$

(10)

如果机器人的状态不确定，深度测量受到噪声的影响，这个似然可以用于贝叶斯规则计算姿态估计。由于先前关于NeRF的文献中没有似然解释，现有的状态估计方法 [3], [12] 只是将光度损失最小化作为最大似然估计的代理。

此外，熵和互信息的概念可以推广到点过程，如 [40] 中所讨论的。特别是，点过程在集合B上的熵定义为

$$
H(B) \equiv \int_{B} \lambda(x)(1 - \log \lambda(x))dx = \gamma \frac{A_{ref}}{A_{aux}} \int_{B} \rho(x)(1 + \log \frac{A_{ref}}{\gamma} - \log \rho(x))dx
$$

(12)

因此， $H$可以作为衡量NeRF在某些集合 $B$上不确定性的指标，这将有助于考虑哪些NeRF的区域是被不足监督的（即，具有高熵）对于诸如最佳视图选择和主动感知等问题。

最后，PPP对NeRF的解释允许我们为NeRF训练提供一个新的视角：最小化[1]中提出的光度损失（2）可以被解释为对沿监督光线的颜色分布的第一时刻进行矩匹配[41, Ch. 4]。具体来说，如果从NeRF渲染的颜色（即，PPP的预期颜色沿射线）与样本分布（即，数据集中的颜色标签）相匹配，则光度损失将为零。我们认为对NeRF密度的概率解释也可能激发其他损失函数的灵感，以执行其他参数估计方法，如最大似然估计、期望最大化等。

# V. 使用Nerf场景计算碰撞概率

我们首先定义 $B(p, R) \subset \mathbb{R}^n$ 为由其姿态（位置 $p \in \mathbb{R}^3$ 和方向 $R \in SO(3)$）参数化的机器人身体（机器人占据的点集），并考虑一个由密度场 $\rho(p)$ 表示的环境，我们已经展示了这个密度场与PPP场通过常数缩放因子 $\lambda(p) = \gamma\rho(p) \frac{A_{ref}}{V_{aux}}$ 相关联。

我们定义碰撞概率或从PPP与机器人身体相交的点集合的概率，作为NeRF中的某些体积 $V_{max}$ 可以存在于机器人体积内的最大概率。我们称 $V_{max}$ 为指定的或允许的相互渗透体积。给定一些辅助粒子（它可能与参考粒子不同），该粒子具有用户定义的体积 $V_{aux} < V_{max}$，我们可以解决在机器人体积内应该存在的最大辅助粒子数量 $N_{max}^{aux}$。

尽管我们没有接触到参考粒子的尺寸，但我们可以解决碰撞问题。我们只是解决与辅助粒子相关的泊松点过程的累积分布函数（CDF），即：

$$
\Pr(X \leq N_{max}^{aux}; \Lambda_B) = \exp^{-\Lambda_B} \sum_{i=0}^{N_{max}^{aux}} \frac{\Lambda_B^i}{i!}
$$

其中 $\Lambda_B$ 是辅助粒子在机器人身体上的强度。

注意 $\Lambda_B$ 是与辅助粒子相关的PPP，而不是参考粒子；因此，我们必须适当地缩放NeRF密度：

$$
\Lambda_B = \int_{B} \lambda_{aux}(x) dx = \frac{V_{ref}}{V_{aux}} \int_{B} \lambda(x) dx
$$

回想一下 $V_{ref} = A_{ref} \delta t$ 并假设辅助体积具有相同的形式 $V_{aux} = A_{aux} \delta t$。此外，我们可以替换 $\lambda$ 和 $\rho$ 之间的关系来得到：

$$
\Lambda_B = \frac{A_{ref} \delta t}{A_{aux} \delta t} \int_{B} \gamma\rho(x) dx = \gamma \frac{A_{aux}}{V_{aux}} \int_{B} \rho(x) dx
$$

这引导我们给出NeRF碰撞的第一个正式定义。

**定义2（概率安全）**：如果由上述公式给出的碰撞概率满足 $\Pr(N(B(p, R)) \leq N_{max}^{aux}; \Lambda_B) \geq \sigma$，对于期望的概率阈值 $\sigma$，与指定的相互渗透体积相关的辅助粒子，以及遮挡阈值，则机器人身体由其姿态 $B(p, R)$ 参数化是概率安全的。

更简洁地说，如果机器人与NeRF之间的相互渗透体积小于阈值 $V_{max}$ 的概率至少为 $\sigma$，则我们认为机器人是概率安全的。

一个主要的剩余问题是如何选择辅助粒子大小和遮挡阈值 $\gamma$。由于我们假设了像素长度尺度上的平滑性，使用像素大小的辅助粒子是合适的。我们使用在近平面上的像素近似大小和沿射线的采样距离来找到辅助粒子的尺寸。具体来说，对于具有50毫米焦距的相机，长度为2米的场地，90度的视野，以及1000×1000的图像，图像平面上的像素边长为 $10^{-4}$米，因此对于一个正方形像素， $A_{aux} = 10^{-8} m^2$。如果像素大小是图像的二维分辨率，我们可以将沿射线的采样分辨率 $\delta t$ 视为学习NeRF的3D分辨率。通常，沿射线在近平面和远平面之间的任何地方都有100到200个采样点，因此我们选择 $\delta t = 20$ mm，得到 $V_{aux} = 2 \times 10^{-8}$ m^3。实际上，CDF对 $A_{aux}$ 相对不敏感，因为 $N_{max}^{aux}$ 和 $\Lambda_B$ 对 $A_{aux}$ 的变化都进行了相同的缩放。 $\gamma$ 本质上是一个建模参数，因为我们无法知道环境将什么定义为遮挡事件。然而，为了安全和可解释性，我们设置 $\gamma = 1$，使其具有意义（即完全遮挡），并提供对 $\lambda$ 的最保守估计。

# VI. Nerfs中机会约束轨迹生成
我们现在考虑在表示为NeRF的环境中实时轨迹规划的问题，受到碰撞概率约束。我们提出的算法，CATNIPS，有两个部分：首先，我们生成一个轻量级、基于体素的场景表示，我们称之为概率不安全机器人区域（PURR），它编码了满足特定机器人几何形状的碰撞约束的机器人位置。然后，我们使用贝塞尔曲线在位置空间中规划安全轨迹，用于机器人穿越PURR，受到概率碰撞约束。通过假设机器人的微分平坦性，我们可以保证在规划平面输出空间时解决方案的动态可行性。

我们注意到，以下碰撞计算可能会因为将机器人近似为一个球体而保守，以至于安全性与方向无关。然而，碰撞度量（13）可以在考虑机器人的位置、方向和物理几何形状的同时计算。我们还注意到，由于碰撞概率在梯度上是可微的，因此它可以在基于梯度的轨迹优化器中使用。尽管如此，我们的设计选择是出于速度、可扩展性和与其他下游任务（例如，活动视图规划，其中机器人方向可以自由操作）的模块性。

## A. 生成PURR

PURR是一个二进制的、体素化的表示NeRF的表示，它在 $R^3$中指示碰撞。如果机器人的位置位于PURR的自由体素内，机器人就是概率安全的，即碰撞约束保证被满足。否则，安全性不能保证——机器人可能（或可能不）违反了概率约束。类似于经典配置空间规划，PURR背后的核心思想是扩展占用空间，使得通过膨胀图中的自由空间规划路径对应于在底层NeRF图中满足碰撞概率约束的机器人轨迹。我们基本上是“扩展”NeRF密度函数，以考虑机器人几何形状和概率约束。

图4显示了PURR生成过程的示意图，以及PURR如何适应我们的轨迹规划流程。我们首先体素化地图的空间，标记每个单元格的NeRF密度乘以一个缩放因子，得到每个单元格的体素化单元格强度网格Ic。然后，我们取机器人的最小边界球与一个底层体素的米克什夫斯基和，产生如果机器人的质心位于一个单元格内的任何位置，机器人可能接触的所有体素集合；我们称这个集合为机器人核K。最后，我们使用3D卷积操作将机器人核K与单元格强度网格Ic卷积，得到机器人强度网格Ir。最后，我们使用机器人强度网格（以及辅助粒子参数）计算泊松累积分布函数（CDF），并通过用户定义的碰撞概率阈值σ对机器人强度网格进行阈值处理，得到二进制PURR图。这些操作在数学上更详细地描述如下。

1. 单元格强度网格：单元格强度网格Ic计算每个网格单元内的预期辅助粒子数量。

$$
Ic(v_{ijk}) = \int_{v_{ijk}} γρ(x) dV
$$

这个积分通常无法解析计算，
由于密度 $\rho$ 通常通过神经网络表示，我们使用三线性插值方案来高质量近似积分计算。如果NeRF密度使用基于体素的表示（如文献 [7], [28] 中描述的最高速和最高质量的NeRF变体），则我们的三线性插值的积分是精确的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/b9134d8289bb42639ac88e2b3c36f207.png#pic_center)

### 机器人核的计算

机器人核 $K(v_{ijk})$ 是一个掩模，指示在计算机器人位置 $p$ 在 $v_{ijk}$ 中的任何位置时的碰撞概率时考虑的单元格周围的邻域。我们首先找到机器人的最小边界球与机器人中心所在的单元格的米克什夫斯基和。然后，我们计算包含这个米克什夫斯基和的最小集合的体素。这个体素集合是机器人核，它可以有效地使用标准的PyTorch函数与3D体素化网格卷积。

### 机器人强度网格的计算

一旦定义了机器人核，计算机器人在特定单元格中的碰撞概率就只需要在核 $K$ 和单元格强度网格 $Ic$ 之间进行卷积。具体来说，我们通过将核与每个单元格卷积，生成一个机器人强度网格 $Ir$，如下所示：

$$
Ir(v_{ijk}) = \sum_{v_{lmn} \in K(v_{ijk})} Ic(v_{lmn}) = \int_{v_{lmn}} \gamma \rho(x) \, dV \geq \int_{B(p,R)} \gamma \rho(x) \, dV \quad \forall R \in SO(3), p \in v_{ijk}
$$

这表示机器人与NeRF场景的相互渗透体积的近似计算。

### 二进制碰撞概率图 (PURR) 的生成

我们使用机器人强度网格（以及辅助粒子参数）计算泊松累积分布函数（CDF），并通过用户定义的碰撞概率阈值 $\sigma$ 对机器人强度网格进行阈值处理，得到二进制PURR图：

$$
P(v_{ijk}) = \Pr(X \leq N_{max}^{aux}; Ir(v_{ijk})) < \sigma
$$

生成的PURR在不同的指定相互渗透体积下，对于合理的碰撞概率 $\sigma = 95\%$ 进行了可视化，用于Flightroom NeRF环境。图5的底部显示了通过在所需的密度水平上阈值化原始NeRF密度获得的基线体素占用表示。PURR通过精确的碰撞概率校准，而直接阈值化NeRF密度则不提供特定的安全保证。

### 碰撞偏移因子的定义

三线性插值密度（用于计算上述积分）引入了潜在的近似误差源。为消除对方法保守性的任何疑虑，我们引入三线性近似误差的上界，称为碰撞偏移因子 $\alpha$：

**定义3（碰撞偏移因子）**：碰撞偏移因子 $\alpha$ 是在整个地图上对以下差异的上界：(i) 通过在机器人核上积分NeRF密度 $\rho$ 获得的最大碰撞概率（即“真实”的碰撞概率），以及 (ii) 使用三线性插值密度 $\hat{\rho}$ 从公式中获得的碰撞概率。

$$
\alpha \leq \min_{i,j,k} (\min_{p \in v_{ijk}, R \in SO(3)} \Pr[X \leq N_{max}^{aux}; \int_{B(p,R)} \gamma \rho(x) \, dV] - P(v_{ijk}))
$$

如果 $\alpha$ 存在，则我们的PURR将使用这个碰撞偏移因子膨胀，以确保PURR具有严格的碰撞概率保证：

$$
P(v_{ijk}) = \Pr(X \leq N_{max}^{aux}; Ir(v_{ijk})) < \sigma - \alpha
$$

**定理1**：给定一个碰撞偏移 $\alpha$ 和所需的碰撞概率阈值 $\sigma$，一个机器人的位置 $p$ 在 $P$ 的补集内，就保证了概率安全。

**证明**：按照上述定义和调整，生成的PURR是对概率碰撞约束的上界，确保了机器人的安全。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/8c796ee5f9554973bbb983ba63b163b1.png#pic_center)

### 在PURR中的路径规划

我们寻求为机器人规划一条动态可行的路径，该路径穿过环境，使得轨迹上的所有点都满足碰撞的偶然约束。我们使用A*搜索找到一条初始路径，并通过贝塞尔曲线生成平滑轨迹，确保整个轨迹在概率上是安全的。

**定义4（概率安全轨迹）**：如果轨迹上的所有点在点上是概率安全的，则轨迹是概率安全的。

我们提出了一个包含三个组件的算法，用于创建这些安全、连续的轨迹。第一步是使用A*搜索在PURR的自由空间中找到一条初始的、离散的路径，从我们的初始位置p0到最终位置pf（例如，使用A*）。虽然动态上不可行，但这条路径提供了一个连接的、无碰撞的路径，从起点到终点穿过PURR，我们将将其细化为一个平滑的、可行的轨迹。第二步是围绕这个初始路径生成一个“管道”，这个管道既大（因此我们最小化了对轨迹优化的约束），又位于PURR的自由空间内（因此在这个管道内的轨迹仍然保持无碰撞）。我们选择这个设计选择是因为本文的重点是在量化NeRF中的碰撞风险，结合了一个相对简单的规划方案。可以引入更灵活和复杂的规划方案[43], [44]来改进我们的方法。最后一步是通过解决一个受约束的凸优化问题来生成一个平滑的曲线，连接我们的初始和最终位置，要求曲线位于之前生成的自由“管道”内。我们称生成的轨迹规划算法为CATNIPS。CATNIPS在给定预计算的PURR图的情况下实时执行。


## A. A* 搜索

我们首先使用A* 搜索在定义PURR自由空间的体素网格上找到一个粗略的、离散的初始路径。具体来说，给定机器人的初始位置 $\rho_ 0$ 和最终位置 $\rho_ f$，我们找到了一个最小长度（以曼哈顿距离测量）的路径，连接相应的初始和最终体素。我们搜索一个六连通图，即机器人可以沿着PURR的x、y和z轴移动到邻近的自由体素中。使用A*搜索和通常的启发式（距离目标的距离，不考虑碰撞）产生了一个连接的、无碰撞的，但从起点到终点动态上不可行的路径。

## B. 边界框生成

我们现在寻求将A*返回的离散、无碰撞路径细化为一个连续轨迹，该轨迹对机器人来说是能量效率高的，并且是动态可行的。为此，我们首先围绕A*路径生成一个“管道”，这个管道既大（因此我们最小化了对轨迹优化的约束），又位于PURR的自由空间内（所以在这个管道内的轨迹仍然保持无碰撞）。我们通过在每个线段周围扩展边界框来生成这些边界框，通过“行进”每个面沿着其法线方向，直到它与PURR边际碰撞（即，至少一个面上的细胞与不安全细胞相邻）。为了加速对候选盒子的碰撞检查，我们在此步骤中将PURR转换为KD树表示。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/54bc0b8eebf84baeaa95d2ca606126d2.png#pic_center)


## C. 通过贝塞尔曲线生成平滑轨迹

我们规划器的最后步骤是生成一个完全位于边界框联合内的平滑轨迹。为此，我们用一系列贝塞尔曲线表示我们的轨迹。具体来说，一个 $\mathbb{R}^n$ 中的贝塞尔曲线，阶数为 $N$，由下式给出：

$$
p(t; s_k) = \sum_{k=0}^{N} \binom{N}{k} (1 - t)^{N-k} t^k s_k
$$

其中 $s_k \in \mathbb{R}^n$ 是一组“控制点”，定义了曲线的几何形状，而曲线则由一个自由参数 $t \in [0, 1]$ 跟踪。对于任何 $t \in [0, 1]$，贝塞尔曲线 $p(t)$ 只是控制点 $s_k$ 的插值，这意味着参数曲线位于控制点的凸包内。因此，为了通过PURR生成一条概率安全的轨迹，我们找到了一组贝塞尔曲线，连接我们的初始位置 $p_0$ 和最终位置 $p_f$，其控制点位于之前生成的边界框内；由于每个贝塞尔曲线必须位于其控制点的凸包内，整个曲线将位于PURR的补集内。

要找到这个轨迹，假设我们有 $L$ 个从上一步生成的边界框 $\{B_1, ..., B_L\}$。然后我们找到一组 $L$ 个贝塞尔曲线，其控制点由 $s_i^k$ 给出，约束所有控制点的第 $i$ 个曲线位于相应的边界框 $B_i$ 内。由于贝塞尔曲线是控制点的线性函数，我们可以将第 $i$ 个曲线表示为 $p_i(t) \equiv \beta(t)s_i$，其中 $s_i \in \mathbb{R}^{n(N+1)}$ 是曲线 $i$ 的所有 $N + 1$ 个控制点的连接向量，而 $\beta(t) : [0, 1] \rightarrow \mathbb{R}^{n \times n(N+1)}$ 是一个只依赖于曲线参数 $t$ 的系数矩阵。我们可以类似地表示曲线的高阶导数为 $p_i^{(d)}(t) = \beta^{(d)}(t)s_i$。

为了生成我们所需的轨迹，我们解决以下优化问题：

$$
\begin{align*}
\min_{s_1,...,s_L} \quad & J(s_1, ..., s_L) \\
\text{s.t.} \quad & s_{ij} \in B_i \forall i \leq L, j \leq N \\
& \beta^{(d)}(1)s_i = \beta^{(d)}(0)s_{i+1} \forall i \leq L, d \leq D \\
& \beta^{(0)}s_1 = p_0 \\
& \beta^{(1)}s_L = p_f
\end{align*}
$$

其中，我们约束每个段的控制点，使得 $s_i$ 必须位于相应的边界框 $B_i$ 内，这定义了 $s_i$ 的一组线性不等式。我们还强制执行每个样条的连续性，直到所需的导数 $D$，这定义了一组线性等式约束。最后，我们强制执行轨迹的边界条件，即曲线从初始位置 $p_0$ 开始，以最终位置 $p_f$ 结束。我们选择优化阶数为 $N = 8$ 的贝塞尔曲线，以平衡我们模型的表达能力（需要直到4阶的非平凡导数）与指定曲线所需的参数数量。

由于我们的目标是二次的控制点，并且我们的约束由线性不等式和等式约束定义，因此生成的优化（22）是一个二次规划（QP），可以实时解决。

**推论2**：由QP（22）的解给出的轨迹是概率安全的。

**证明**：QP（22）将每个贝塞尔曲线限制在概率安全的边界框内，使每个曲线都是安全的。由贝塞尔曲线的联合给出的结果是轨迹，因此整个轨迹是概率安全的。

**备注3**：我们强调所有轨迹在概率上都是安全的，其中任何轨迹中的所有点都满足相互渗透约束（13）的概率σ。这不等于说所有轨迹的σ分数都不包含任何违反相互渗透约束的点。

**备注4**：如果机器人系统动力学是微分平坦的，即其位置是平坦输出的子集，则由所提出的QP（22）生成的路径（将D设置为平坦输出中位置的最高导数）是动态可行的。因此，跟踪此规划器的轨迹的机器人保持概率安全。

**备注5**：我们注意到，由我们的规划器返回的轨迹的每一段都由一个简单的曲线参数t参数化，该参数不必对应于时间。然而，由于我们假设系统是微分平坦的，存在一个时间缩放，使得曲线是动态可行的。为了解决这个问题，我们使用一个简单的时间重新缩放（如[47]中所述）来生成最终的轨迹作为时间的函数。
# VII. 数值结果
在本节中，我们在模拟的Stonehenge场景和真实的Statues与Flightroom环境中研究了我们提出的机会约束轨迹优化器。真实场景是使用手持手机相机捕获的，姿态是从COLMAP提取的。使用我们提出的轨迹优化器，我们为模拟和真实的四旋翼飞行器在场景中飞行生成轨迹，并研究了在大量初始和最终条件下生成的轨迹的安全性和保守性。使用相同的路径规划算法，我们对PURR和通过在所需的密度水平上阈值化原始NeRF密度获得的基线体素占用表示进行了比较。我们还将这些方法与作者之前的工作NeRF-Nav[12]进行了比较。由于该方法需要A*初始化，我们对基线网格和NeRF-Nav使用了相同的A*初始化。具体来说，NeRFNav A*初始化是从对应于截止密度ρ=102（最保守的截止）的基线密度网格生成的。

我们展示了体素方法比NeRF-Nav在计算上更高效，并且还能生成更安全（碰撞次数更少）和更不保守（路径更短）的轨迹。此外，我们发现我们的方法，CATNIPS，允许用户为碰撞设置一个明确定义的概率阈值。相比之下，密度阈值基线并不提供这样的概率保证。换句话说，可以从密度阈值化的地图获得类似的行为，但这需要用户通过试错来调整密度阈值以达到所需的安全水平，而且适用于一个环境的阈值可能不适用于其他环境。即使调整后获得了良好的经验行为，基线也没有伴随的安全保证。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/0573df46a50a4c91ae2abf0c76522e3a.png#pic_center)

## A. 算法性能

对于所有三个环境，在圆上的100个起点和终点位置（在上下方向上随机扰动）的提出方法的定性结果如图7所示。PURR是使用每边150个体素的分辨率生成的，概率阈值σ=0.95，Vmax=10^-6指定的体积穿透。此外，由于我们的四旋翼系统在位置上具有微分平坦性，机器人可以使用标准的微分平坦控制流程[47]来跟随这些路径。

我们首先将我们的方法与两个基线进行比较：“基线网格”，它从密度阈值计算占用情况，以及NeRF-Nav[12]，一个基于梯度的NeRF轨迹规划器，如图8所示。我们根据以下三个指标分析它们的性能：轨迹过程中与地面真实网格的最小有符号距离（负值表示在网格内部），轨迹过程中达到的最大相互渗透体积，以及生成的轨迹与最短直线路径长度之间的差异。前两个指标量化了安全性，最后一个指标量化了保守性。我们在均匀分布在圆上的100个随机配置上评估了算法。

我们选择显示CATNIPS的六个参数组合，这些参数在所有场景中变化，以展示通用性和可解释性。我们选择的截止值为合理的95%和99%阈值。我们选择的相互渗透体积是机器人体积的一部分。对于Stonehenge场景，相互渗透体积的数值为 $5 \times 10^{-6}$、 $10^{-6}$、 $10^{-7}$ 分别对应13%、3%、0.3%的机器人体积，而对于真实环境，这些值分别对应4%、0.8%、0.08%的机器人体积。


为了与基线进行公平的比较，我们改变了基线网格的密度截止值和NeRF-Nav中的碰撞惩罚权重。我们再次强调，NeRF-Nav和基线网格路径都没有可解释的参数，并且需要通过试错来调整参数以达到所需的安全性能，这在事先是不可能知道的，因为在现实中无法访问到地面真实。然而，为了以善意对方法进行基准测试，我们选择了在合成环境（即Stonehenge）中性能与CATNIPS相似的密度截止值，因为我们有它的地面真实网格。然后，我们对真实环境使用了相同的截止值。根据我们的经验，这些也是通常用于使用marching cubes[13]从NeRFs提取网格的阈值。对于NeRF-Nav，碰撞惩罚权重被选择为损失函数中的主导项。我们可以看到，NeRF-Nav轨迹在所有碰撞损失权重（102、103、104）下与任何体素方法生成的路径相比都是不安全的（负值表示与网格碰撞）。当我们增加碰撞惩罚的权重时，我们确实看到算法在平均安全性能上越来越安全（更高的SDF，更低的体积交集）。然而，如此高的碰撞惩罚（104）通常会导致轨迹优化器中的数值问题。此外，这些轨迹在最坏情况下也比任何体素方法都安全。最后，这些轨迹还偏离了最短路径最远，展示了保守和非平滑的路径。

基线网格派生的轨迹可以表现出安全和不保守的行为。然而，实现这种行为所需的参数不能推广到所有场景是显而易见的。这在截止ρ=104的情况下尤为明显，Stonehenge中的安全性能是合理的，但我们在真实环境中看到了不安全的性能。

我们看到我们的方法，CATNIPS，是安全的（通过构造）并且不保守。平均而言，这些轨迹与基线网格路径在保守性上相似，同时展现出合理的安全水平，并对参数变化做出预期的反应（当减小指定的体积交集和/或增加概率阈值时，SDF更大，交集更小）。我们请读者注意CATNIPS的体积交集指标（图8，中间行），其中表达了轨迹的安全性（请参见备注3以区分）。箭头表示100条轨迹的σ分位数，而虚线表示指定的体积交集。虽然我们不对轨迹的完全安全率（即，箭头不必低于相应的虚线）提出要求，但我们看到，确实，σ比例的轨迹倾向于完全安全（整个轨迹上没有不安全的点）。

我们在图9中验证了我们提出的点安全性声明，并通过网格离散化对CATNIPS进行了消融研究。该图包含具有两种不同碰撞概率、三种不同指定体积交集、三种不同网格离散化（100、150、200）和三种不同环境的参数组合的运行。请注意，一些列可能包含多个条形图，代表在保持相同的碰撞截止和体积交集的同时，不同的网格离散化或环境。条形图的高度表示所有100条轨迹中至少有一点体积交集高于该参数组合指定值的轨迹的比例。这些条形图顶部的数字表示所有轨迹中所有点低于相同参数设置的指定体积交集的百分比。我们的理论声明涉及所有轨迹上所有点的比率（条形图顶部的数字），然而我们观察到，对于我们的方法，期望的碰撞率也倾向于在完整轨迹上保持（条形图的高度）。未可视化的组合意味着在任何轨迹中没有点违反体积交集约束。

请注意，报告的所有点安全百分比（所有百分比大于概率截止）意味着我们导出的点概率安全约束得到了验证，并且满足此约束是与参数无关的。这使得（13）、PURR以及围绕它的规划架构可以推广到任意环境和合理的网格离散化。这个结果还意味着通过神经网络基密度场的三线性插值引入的误差在安全影响上是小的（即，碰撞偏移因子α≥0）。这对于事先无法验证安全的现实场景以及需要较粗网格离散化以提高计算性能的应用来说特别有吸引力。此外，轨迹安全性（如图8所示）通常在网格离散化和场景中得到满足。

这里我们想向读者总结几个关于碰撞违规的微妙点。一些点上的指定体积穿透违规（图8和9）是由于碰撞约束的概率性质，以及NeRF并不完全捕捉到真实表面的事实。为了使NeRF密度完全表示真实表面，根据我们的PPP解释，它必须在表面外正好是0，在内部是∞。在这种情况下，碰撞将是确定性的，因为满足碰撞机会约束的唯一方式就是没有碰撞。在实践中，NeRF密度场不能是0或∞，因为表示的连续性和真实表面位置的不确定性。因此，与真实表面的碰撞可能以规定的概率发生。我们认为，感知基础规划中不确定性的准确表示必须承认一些碰撞概率，因为总是存在非零概率，即感知错误导致场景占用的估计不正确。我们通过在构建PURR时嵌入几个保守的近似值来进一步缓解这种“NeRF到现实”的差距。因此，在实践中，我们与真实表面的碰撞频率比碰撞约束所要求的要低。再次，这一事实在图9中得到了说明，其中碰撞约束满足率非常接近100%，但不够保守，以至于不实用（见图7和8）。

最后，我们在Flightroom环境中的无人机硬件实验中验证了我们的CATNIPS流程（见图10）。使用预训练的静态NeRF，我们计算了从围绕场景的圆周上分布的起点-终点对出发的十条轨迹，并驱动机载控制器跟随这些航点（开环）。然后，我们选择了这两个起点-终点位置，并在同时更新无人机以跟随航点流（闭环）的同时，在线运行了CATNIPS A*、边界框生成和凸优化程序（选择下一个预测点作为航点）。所有轨迹都运行直到它们到达目标位置。开环轨迹长约20秒，因为它们是预定义的，而闭环轨迹可以有不同的时间，因为它们不是预定义的。我们看到所有轨迹的有符号距离都大于0，因此是安全的（没有碰撞）。即使对于接近0的开环轨迹，我们通过视觉验证了无碰撞轨迹。接近零的SDF是由于在计算有符号距离时用边界框过度近似无人机机身所致。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/906e5305608141a2ae9927691da9d694.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/f48efce7aae848c18fdac09624be82d5.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/d8d3f54f756d4f769fbc7056345cb54b.png#pic_center)


## B. 计算时间
上述算法的实现在配备RTX 4060 GPU的笔记本电脑上使用Pytorch完成。我们的方法是建立在NeRFStudio[8]之上的。我们没有花太多努力优化代码以加快计算速度，所以我们预计这些执行时间可以大幅度减少。此外，为了进行公平比较，我们将NeRF-Nav移植到了NeRFStudio。一般来说，CATNIPS的规划部分（A*、边界框和贝塞尔曲线生成）的运行速度约为每秒3次。从查询NeRF密度到创建PURR的操作以每秒1次的速度运行。每个操作的细分显示在表I中。作为未来工作的有希望的发展方向，可以通过在GPU上进行并行化和代码优化来进一步减少创建PURR和优化PURR中轨迹的计算时间。请注意，所提出的方法从当前位置到目标生成平滑轨迹。另外请注意，在在线重新规划场景中，通常只有下一个航点被跟踪，然后整个轨迹才会更新。因此，我们方法的某些部分可以调整为只考虑机器人的附近区域，以计算时间为代价换取次优解。相比之下，NeRF-Nav需要更长的时间才能收敛到合理的公差，并且没有任何安全保证。我们认为这主要是由于优化其高度非凸目标的困难以及在轨迹优化器内对密度神经网络所需的多次查询。为了在安全性方面产生NeRF-Nav的最佳性能，我们运行了1000次梯度步骤的算法。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/572d5cbeb1d74f4d901b55e84261f50b.png#pic_center)

# VIII. 结论

在本文中，我们提出了一种新的方法，用于在神经辐射场（NeRF）场景中进行机会约束轨迹优化。我们提出了一种将NeRF转换为泊松点过程（PPP）的方法，该方法用于为机器人在场景中的移动生成严格碰撞概率。利用碰撞概率的表达式，我们开发了一种快速在线轨迹生成方法，该方法在离线时将NeRF密度转化为基于体素的碰撞概率表示，称为概率不安全机器人区域（PURR）。利用PURR，我们提出了一种算法，用于规划表示为贝塞尔样条的轨迹，保证机器人在样条上的移动不会超过用户定义的最大碰撞概率。在数值实验中，我们展示了我们提出的方法比现有的通过NeRF进行轨迹规划的方法[12]生成更安全、更不保守的路径，并且比使用NeRF密度阈值作为碰撞概率代理的基线规划器提供更良好和更易于用户解释的结果。我们还证明了我们的流程可以实时运行。
这项工作为未来研究开拓了多个方向。由于我们的整个流程（包括PURR生成和轨迹优化）都可以实时运行，我们的规划器可以与基于NeRF的状态估计器（例如，[19]，[21]）结合使用，以在NeRF上执行主动探索或最佳视图规划，允许机器人仅使用机载视觉自主探索新环境。在导航的基础上，另一个有趣的方向是在线期间调整碰撞度量。由于碰撞概率关于姿态是可微的，因此可以在线期间响应实时收集的数据（例如，感知到的与最近障碍物的最小距离）来调整碰撞概率。这可以通过自动微分作为基于PyTorch的规划流程的一部分来实现。这里开发的用于量化NeRF表示中不确定性的概率碰撞框架可能对诸如表示为NeRF的刚体的不同iable模拟[49]或接触丰富的操作和运动规划等问题有有趣的应用。最后，由于我们从NeRF中派生PPP的推导是严格和可泛化的，我们希望我们对NeRF的解释将有助于机器人学以外的研究，例如在计算机视觉和计算机图形学领域。