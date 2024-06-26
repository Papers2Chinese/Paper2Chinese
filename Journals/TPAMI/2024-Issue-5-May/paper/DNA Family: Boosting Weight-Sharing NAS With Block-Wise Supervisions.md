# [DNA Family: Boosting Weight-Sharing NAS With Block-Wise Supervisions](https://ieeexplore.ieee.org/document/10324326/)
## 题目：DNA家族：利用块级监督策略增强权重共享的神经网络架构搜索
**作者：Guangrun Wang , Member, IEEE, Changlin Li , Liuchun Yuan, Jiefeng Peng, Xiaoyu Xian , Xiaodan Liang ,
Xiaojun Chang , Senior Member, IEEE, and Liang Lin , Senior Member, IEEE**

****
# 摘要
神经架构搜索（NAS）旨在通过机器自动设计神经架构，被认为是自动机器学习的关键步骤。其中一类值得注意的NAS方法是权重共享NAS，它显著提高了搜索效率，并允许NAS算法在普通计算机上运行。尽管备受期待，但这类方法在搜索有效性上存在不足。通过使用泛化界限工具，我们展示了导致这一缺点的原因是潜在的不可靠架构评分，这是由于可能架构的搜索空间过大造成的。为了解决这个问题，我们将一个大型搜索空间模块化为具有较小搜索空间的块，并开发了一组使用蒸馏神经架构（DNA）技术的模型。这些提出的模型，即DNA家族，能够解决权重共享NAS的多个困境，如可扩展性、效率和多模态兼容性。我们提出的DNA模型能够评估所有架构候选项，与以往工作不同，后者只能使用启发式算法访问子搜索空间。此外，在一定的计算复杂性约束下，我们的方法可以寻找具有不同深度和宽度的架构。广泛的实验评估表明，我们的模型在ImageNet上分别为移动卷积网络和小型视觉变换器实现了78.9%和83.6%的最新顶级1准确率。此外，我们还提供了深入的实证分析和对神经架构评分的洞察。

# 关键词
- 块级学习
- 泛化界限
- 神经架构搜索
- 视觉变换器

# I. 引言
神经架构搜索（NAS）[1]，旨在用机器取代人类专家设计神经架构，被广泛期待。典型的工作包括强化学习方法[2]、[3]，进化算法[4]、[5]和贝叶斯方法[6]、[7]。这些方法需要多次尝试（即单独训练许多架构以评估其质量），这对许多研究人员来说是计算上不可行的。最近的权重共享NAS解决方案将搜索空间编码为权重共享的超网络，并在超网络中同时训练所有架构，显著提高了搜索效率。

尽管效率很高，但权重共享NAS的有效性令人担忧[8]、[9]、[10]。罪魁祸首可能是由于搜索空间过大导致的不可靠的架构评分。具体来说，权重共享NAS方法共同训练超网络权重以评估架构候选项。随着搜索空间的增加，泛化界限工具表明超网络权重的泛化能力降低，导致不可靠的架构评分。这种预测和真实架构排名之间的不相关性使权重共享NAS变得无效。

为了提升权重共享NAS，减小搜索空间是有帮助的。因此，我们将大型搜索空间分解为具有较小搜索空间的块。具体来说，我们将神经架构分成块（见图1(a)）并分别训练每个块。这样，与整个搜索空间中的架构候选项相比，块中的架构候选项数量呈指数级减少，从而允许更可靠的架构评分。有了这些块级表示，我们使用蒸馏神经架构（DNA）技术开发了三种新型NAS模型。我们将这些模型归纳为一个称为“DNA家族”的家族，因为它们在构建模型的机制上具有相似性，即通过从教师网络中提取知识来训练学生超网络[11]。以下，我们将总结三个DNA模型的特点，并讨论它们如何解决权重共享NAS的多重困境。
- DNA：基本模型（见图1(b)）采用传统的监督学习，即把固定教师网络的潜在代码作为监督。由于可以同时训练几个学生超网络，因此非常高效。
- DNA+：这个模型（见图1(c)）允许在架构搜索过程中逐步更新教师网络。因此，DNA+的性能对教师网络的初始容量的依赖性较小，从而实现了更好的可扩展性。
- DNA++：这个模型（见图1(d)）使得教师网络和学生超网络以自监督学习的方式共同优化，即在架构搜索期间不需要标记数据。DNA++的一个优势是能够容忍教师网络和学生超网络的不同基本架构（例如，教师使用CNN，学生超网络使用视觉变换器（ViT））。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/b48e975fe1634d17bd7c743c716f0bfe.png" width="70%" /> </div>


有了块级表示，我们可以评估所有候选架构，与以往只能使用启发式算法访问子搜索空间的方法不同。因此，我们的方法在与其他权重共享NAS方法相比时显示出有希望的有效性水平。总体而言，我们的贡献是五方面的：
- 我们展示了过度的搜索空间显著阻碍了大多数权重共享NAS方法的有效性。随着搜索空间的扩展，超网络权重表现出降低的泛化能力，导致不精确的架构排名和无效的搜索。为了解决这个问题，我们通过使用块级表示对广泛的搜索空间进行模块化，系统地减少了搜索范围。
- 我们在通用知识蒸馏框架下探索了三种实现，以平衡可扩展性、学习效率和与各种神经网络结构的兼容性。所提出的DNA家族可以灵活地采用，以解决NAS应用的不同困境。
- 块级监督使我们的模型能够评估所有候选人，这与以前的方法不同，后者只能探索搜索空间的一个子集。此外，这种方法还允许我们在计算资源的约束下搜索具有不同深度和宽度的架构。
- 我们通过全面的实证研究突出了架构评分的重要性，包括模型排名、使用Kendall Tau度量的评估以及对训练稳定性的评估。这些研究为传统的权重共享NAS方法的（无效）有效性提供了见解。
- 在ImageNet、CIFAR10和PASCAL VOC分割上提供了有希望的高效实验结果。通常，我们的DNA家族在移动设置中获得了78.9%的顶级1准确率，在ImageNet上的ViT上获得了83.6%的顶级1准确率。

# III. 方法论

我们首先分析了权重共享神经架构搜索（NAS）的困境（第三节 A）。为了解决这个问题，我们将搜索空间划分为区块（第三节 B）。利用这些区块化的表示，我们开发了三种利用神经架构蒸馏技术的 NAS 模型，以应对各种权重共享 NAS 面临的挑战。第三节 C 介绍了使用监督学习的 DNA，提供了可搜索的宽度/深度和资源适应性。第三节 D 概述了使用渐进学习的 DNA+。第三节 E 提出了使用自监督学习的 DNA++。

## A. 权重共享 NAS 困境的基本分析

寻找最优架构需要多次/贪婪的试验，即分别训练许多/所有架构以评估它们的质量，这在计算上对许多研究人员来说是不可承受的。最近的工作放弃了单独训练每个候选架构的做法；相反，它们将搜索空间 A 编码成一个权重共享的超网络，并同时在超网络中训练所有架构。这些方法被称为权重共享 NAS，显著提高了搜索效率。尽管效率很高，但权重共享 NAS 的有效性并不令人满意。我们在这里分析了造成这一问题的原因。

正式来说，让 $\alpha_j$ 和 $\psi_j$ 分别表示一个架构及其网络权重。NAS 是一个问题，需要找到最优的一对 $(\alpha^ *_ j, \psi^ *_ j)$ ，使得模型性能最大化。为了执行搜索，人们需要评估许多架构的质量作为调整搜索策略的奖励，即他们需要单独训练每个架构以获得 $\psi^ *_ j = \arg\min L(\psi_j|\alpha_j)$ ，并使用 $\psi^ *_ j$ 来评价架构。然而，训练每个架构是非常低效的。权重共享 NAS 方法将搜索空间 A 制定成一个过度参数化的超网络，使得每个候选架构 $\alpha_j$ 是超网络的一个子网络。它们在超网络中同时训练所有架构以获得最优的超网络权重 $\Psi_{sup} = \arg\min L(\Psi|A)$。然后，它们从超网络中提取每个子网络的权重进行验证，并使用这个验证准确率来评价子网络。权重共享 NAS 的困境在于预测的和实际的架构质量之间的相关性很低。为了分析这个猜想，我们首先有一个关于泛化有界的定理。

定理 1.（泛化有界）：对于任何子网络 $\alpha_j$ ，我们使用 $\psi_{sup,j}$ 表示从训练好的超网络中提取的次优权重，并使用 $\psi^*_ j$ 表示其单独训练时的理想权重。那么， $\psi_{sup,j}$ 的 Frobenius 范数上界为：
 
$$
\| \psi_{sup,j} \|_ F \leq \sqrt{C_1 + C_2 T \sum_{t=1}^{T} \| C_3 - L(\psi^*_t|\alpha_t) \|^2}
$$ 
 
其中 $C_1 \geq 0$ ， $C_2 \geq 0$ ，和 $C_3$ 是常数。

备注：定理 1 的证明在附录中，可在线获取。定理 1 表明，使用训练好的超网络权重，一个子网络的模型复杂度（通常衡量泛化能力）与 T 有关，有一个上界。增加搜索空间会导致泛化能力变差，进一步意味着超网络权重估计的架构质量不能预测实际质量。假设一个架构具有很好的质量，但基于超网络权重的泛化能力很差。那么，它的验证准确率可能很低，其质量将被低估。总之，过大的搜索空间是罪魁祸首，因为大搜索空间会导致泛化能力差，进一步导致架构评级不准确，最终导致搜索无效。相反，减少搜索空间可以提高权重共享 NAS 的有效性。

定理 1 强调了区块化监督提高架构排名的潜力，这与“模块化学习”的概念相呼应。模块化学习有效地证明了将神经网络训练分解为模块化组件并独立训练每个模型的优势，展示了多种好处。在我们的情况下，定理 1 支持这样的观点：模块化学习确保了每个子网络的公平和全面训练，这是提高排名的关键因素。

## B. 将搜索空间模块化为区块

如上所述，为了提高权重共享 NAS 的有效性，应该减少搜索空间。不幸的是，直接用一个小空间替换一个大空间是不可取的，因为这会导致准确性范围变小，使搜索失去意义。保持整个搜索空间不变，我们将大空间模块化为区块。每个区块中的候选数量因此显著小于整个搜索空间中的候选数量。也就是说，我们将超网络 U 划分为 N 个区块，关于超网络的深度：

$$
U = U_N \circ \cdots \circ U_{k+1} \circ U_k \circ \cdots \circ U_1 \quad \Psi = [\Psi_N; \ldots; \Psi_{k+1}; \Psi_k; \ldots; \Psi_1] \quad Z = [Z_N, ; \ldots; Z_{k+1}; Z_k; \ldots; Z_1]
$$

其中 $U_{k+1} \circ U_k$ 表示在超网络中第 (k+1) 个区块连接到第 k 个区块。 $\Psi_k$ 是第 k 个区块的网络权重。 $Z_k = \{X_k, Y_k\}$ 是第 k 个区块的输入数据和监督。我们分别优化超网络的每个区块：

$$
\Psi_{sup,k} = \arg\min_{\Psi_k} L_{train}(\Psi_k|A_k, X_k, Y_k), \quad i = 1, 2, \ldots, N
$$

其中 $A_k$ 表示第 k 个区块的搜索空间。

区块与整个搜索空间的比较：设 $d_k$ 和 $c$ 分别表示第 k 个区块的深度和每层的候选操作数量。那么，第 k 个区块的搜索空间大小为 $c^{d_k}$；整个搜索空间的大小为 $\prod_{k=1}^{N} c^{d_k}$。这表明搜索空间的指数级减少，即 $(\prod_{k=1}^{N} c^{d_k}) / c^{d_k}$。在我们的实验中，区块化搜索空间显著小于整个搜索空间（例如，减小比率约为 $1e^{15 N}$），确保了有效的权重共享 NAS。最后，架构在整个搜索空间 A 中的不同区块中进行搜索：

$$
\alpha^* = \arg\min_{\forall \alpha \in A} \sum_{k=1}^{N} \lambda_k L_{val} (\Psi_{sup,k} | A_k, X_k, Y_k)
$$

其中 $\lambda_k$ 表示损失平衡。

备注：重要的是要强调，尽管我们使用区块化监督来帮助训练超网络以提供区块化的局部评分，但我们的架构搜索仍然是在整体架构中执行的，通过考虑所有局部评分。以前的工作 [23] 假设深度网络中每一层的权重是独立的。相反，我们认为深度网络中不同层的权重高度依赖于彼此。我们在第三节 C 中的算法允许我们在 10^17 个架构的完整搜索空间中精确地识别出最高奖励的架构。此外，我们的方法还可以在特定的计算约束下搜索具有不同深度和宽度的架构。

## C. DNA: 通过监督学习进行蒸馏

尽管我们在第III-B节中有很好的动机，但技术障碍在于缺乏内部真实标注的(3)。受知识蒸馏（KD）[11]的启发，我们使用教师的隐藏特征作为监督信号。设 $Y_k$ 为第k块教师的输出张量， $\hat{Y}_k(X_k)$ 为超网络第k块的输出张量。我们采用L2范数作为代价函数。(3)中的损失函数可以写成：

$$
L_{train}(\Psi_k|A_k, X_k, Y_k) = \frac{1}{K} \sum_{i=1}^{K} \| Y_k - \hat{Y}_k(X_k) \|^2_2,
$$

其中K表示 $Y$ 中神经元的数量。此外，受到在变换器[46]、[47]、[48]中丢弃RNN的低效顺序训练的显著成功的启发，我们类比地并行化了我们的超网络训练。具体来说，我们使用教师的第(k-1)块的输出作为超网络第(k-1)块的输入，即我们用 $Y_{k-1}$ 替换(5)中的 $X_k$ 。因此，搜索可以加速并行化。(5)可以写成：

$$
L_{train} (\Psi_k|A_k, Y_{k-1}, Y_k) = \frac{1}{K} \sum_{i=1}^{K} \| Y_k - \hat{Y}_ k(Y_{k-1}) \|^2_2.
$$

图2展示了DNA的流程。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/8d918b43355e4437a028f2e70a7f9e23.png" width="70%" /> </div>


*可搜索的深度和宽度*：在NAS中，自动分配每个块的模型复杂度在特定约束下是至关重要的。为了更好地模仿教师，每个块的模型复杂度应根据相应教师块的学习难度自适应地分配。输入图像大小和每个块的步长固定，通常，计算分配只与每个块的宽度和深度有关，这在权重共享NAS中搜索是繁琐的，因为宽度和深度在设计超网络时通常是预定义的。大多数以前的工作包括将恒等操作作为深度搜索的候选操作([15], [16], [19], [22], [23])。然而，正如[49]所指出的，将恒等操作作为候选操作可能会导致超网络收敛困难和子模型之间的不公平比较。此外，将恒等操作作为候选操作可能会导致冗余搜索空间。例如，序列{conv, identity, conv}等价于{conv, conv, identity}。根据定理1，这种冗余搜索空间可能导致搜索无效。此外，[50]首先搜索具有固定操作的层数，然后在固定层数下搜索三种操作。然而，操作选择取决于每个块的深度，导致累积搜索偏差，尤其是当搜索空间增加时。由于块级监督，我们可以并行地在每个块中训练具有不同通道数/层数的几个单元，以确保深度/宽度的可变性。具体来说，在每个训练步骤中，教师的前一个特征图被馈送到并行的三个具有不同深度/宽度的单元中（例如，图2中数据流的实线）。对于每个单元的每一层，从候选操作中随机采样一个操作（例如，图2中数据流的虚线）。

*约束搜索算法*：我们的典型超网络包含大约 $10^{17}$ 个子模型，阻止我们对所有子模型进行评分。以前的权重共享NAS使用随机采样、进化算法和强化学习从训练好的超网络中对子模型进行采样以进行进一步评分。在最新的工作中([23], [50])，使用贪婪搜索算法通过逐层选择性能最好的部分模型逐步缩小搜索空间。相反，尽管使用块级监督计算块级局部得分，我们的架构搜索仍然在全局搜索空间中执行，通过考虑所有局部得分进行。我们可以巧妙地遍历所有子网，以在某些约束下选择性能最好的模型。
- 评分步骤：块级监督使我们能够在搜索空间中对所有候选者进行评分。我们首先使用块级监督计算局部得分，这是可承受的，因为每个单元中只有 $10^4$ 个子模型。为了进一步加速，我们以类似于深度优先算法的方式逐个节点处理批处理数据，通过保存每个节点的中间输出并由后续节点重用来避免从头节点重新计算它。特征共享评估算法在附录中的算法1中概述，可在线获取。通过评估超网络中的所有单元，我们可以得到一个块中所有可能路径的评估损失。我们可以在几秒钟内使用单个CPU快速对这个列表进行排序，该列表大约有 $10^4$ 个元素。每个块都有这样一个局部得分列表。

- 搜索步骤：给定一个计算成本约束，我们应该使用一个公平的评分指标自动为不同块分配成本。由于MSE损失受到教师特征图的方差影响，我们使用一个更公平的指标，即相对L1损失：

$$
L_{val}(\Psi_k|A_k, Y_{k-1}, Y_k) = \frac{\|Y_k - \hat{Y}_ k(Y_{k-1})\|_1}{K \cdot D(Y_k)},
$$

其中D(·)测量方差。块级局部得分Lval(Ψk|Ak, Yk−1, Yk)被求和以进行全局搜索。高效地，我们不需要为所有 $10^{17}$ 个候选项计算成本（例如，FLOPs）和损失。随着局部得分列表中的分数排名，我们提出了一个高效的搜索算法，用于访问所有可能的模型（附录中在线提供的算法2）。首先，在搜索循环中，如果成本已经超过约束，我们使用语句“continue”跳到下一个循环迭代。其次，当找到满足约束的模型时，我们返回到前一个块，因为当前块中的这个模型是最优的。第三，我们通过预先计算的查找表获得每个候选操作的成本，以节省时间。该方法可以类比为从起点A到目的地B的徒步旅行，有许多可能的路线可以探索。每条路径产生不同的奖励。如果我们确定从A到B通过C的关键里程碑C点，并且进一步发现从A到B通过C的最佳路径，记为AC，那么很明显，从A到B通过C的最佳路径必须包括AC。任何不包含AC的路径都不是最优的。利用这个概念，我们能够高效准确地从大量的 $10^{17}$ 个样本中确定最高奖励的架构。

## D. DNA+: 通过渐进学习进行蒸馏

在搜索之后，我们搜索到的架构表现出色。如果我们将搜索到的架构扩展到与教师相同的大小，我们的扩展架构可以显著优于原始教师。这一令人印象深刻的结果鼓励我们进行级联NAS。我们迭代地将搜索到的架构扩展为新的教师，并一代又一代地在新教师的指导下搜索新架构，模仿了具有多代知识传递的重生神经网络[30]的优点。这形成了另一个DNA版本，即DNA+。

在第一代中，我们使用一个现有的模型（例如EfﬁcientNet-B1）作为教师模型，即：

$$
Y^1_k = f(X^1_k | \beta^1_k, \Psi^1_k),
$$

其中 $\beta^1$ 和 $\Psi^1$ 是第一代教师的架构和网络权重。 $X^1$ 和 $Y^1$ 是输入和输出。这里，我们用k表示第k个块。将 $Y^1_k$ 代入(4)中，我们可以训练第一代超网络并搜索第一代的最优架构 $\alpha^*_1$ 。

在连续的一代中，通过使用[3]中的扩展策略（在(9)中表示为Scaling）获得新的教师，然后从头开始重新训练新的教师模型，该模型用作这一代的块级监督：

$$
\beta^m = \text{Scaling}(\alpha^{(m-1)*}),
Y^m_k = f(X^m_k | \beta^m_k, \Psi^m_k).
$$

每一代的搜索过程与第一代相似。我们对几代进行迭代搜索以进行饱和。在过程结束时，最后搜索到的架构是最后一代的最优学生 $\alpha^*_M$ ，它将在没有扩展的情况下重新训练。图3展示了DNA+的流程。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/ea40ece4a654404eaeeb20708f3d4be8.png" width="70%" /> </div>


*友情提醒*：我们想强调的是，架构排名的改进主要归功于块级监督学习。知识蒸馏只是实现这种块级监督学习的手段。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/4146c79e3c444de196b76a339b6fb687.png" width="70%" /> </div>
## E. DNA++: 通过自监督学习进行蒸馏

尽管DNA和DNA+的搜索性能很高，但他们的搜索结果可能会受到教师架构的影响。具体来说，根据(7)，与教师操作更相似的候选者在每个块中往往会有更好的局部得分，导致架构排名偏差。当搜索空间接近教师网络时，这种偏差不明显。但正如[37]所建议的，如果搜索空间更加多样化，这种偏差将会被放大。幸运的是，没有访问现有教师，自监督NAS方法已被证明能够实现与监督NAS方法相当的性能。因此，我们使用自监督作为现有教师监督的替代品，以减少使用现有教师模型引起的排名偏差。

在计算机视觉中使用自监督的最有效策略包括对比学习[48]、[51]、[52]、度量学习[53]和生成[54]。在这些策略中，人们为每个图像进行两次独立的数据增强，并将它们输入到可学习的和不可学习的网络中。通过充分利用两个输出的相似性来学习表示。注意，不可学习网络通常是可学习网络的移动平均值（即，均值教师）[48]、[51]、[52]、[53]，或者是其梯度被停止的可学习网络的副本[55]。换句话说，可学习网络和不可学习网络需要是相同的；否则，学习将会崩溃。这使得将对比学习和度量学习引入NAS非常困难。为了解决这个问题，[37]和[10]将教师和学习网络都设置为超网络，这使得它们非常庞大且不优雅。幸运的是，受到[56]的启发，我们能够通过引入DNA++来优雅地解决这个问题。

*概述和符号*：DNA++的概述如图2所示。对于每个块，我们首先使用两个MLP来提取其特征。这些特征表示为 $Z_{j_k}$ 和 $\hat{Z}_{j_k}$ ，其中j表示一批中的第j个样本，k表示第k个块。然后我们对特征施加两种损失，即自监督损失和特定的损失以去除冗余的不可学习超网络。

*自监督损失*：自监督学习中的基本损失是距离度量损失[51]、[52]、[53]，其目的是学习特征之间的相似性和不相似性。为了稳定优化并简化自监督学习，通常还需要进行规范。因此，我们的自监督损失定义为：

$$
L_{SSL} = \lambda_1 \sum_{k=1}^{K} \sum_{j=1}^{M} \|Z_{j_k} - \hat{Z}_ {j_k}\|^2_2 + \lambda_2 \sum_{k=1}^{K-1} \sum_{c=1}^{C} (\| \text{Off-diag}(\text{Cov}(Z_k)) \|^2_2 + \| \text{Off-diag}(\text{Cov}(\hat{Z}_k)) \|^2_2),
$$

其中第一项是距离度量损失，第二项是正则化损失。正则化项的作用可以在[57]中看到。这里K是块的数量，M是批量大小，C是通道数。 $\lambda_1$ 和 $\lambda_2$ 用于损失平衡。Off-diag表示取矩阵的非对角元素。Cov表示计算向量集的协方差矩阵，即 $\text{Cov}(Z_k) = \frac{1}{M-1} \sum_{j=1}^{M} (Z_{j_k} - \bar{Z}_ k)(Z_{j_k} - \bar{Z}_ k)^T$ ，其中 $\bar{Z}_ k = \frac{1}{M} \sum_{j=1}^{M} Z_{j_k}$ 。

方程(10)是标准的自监督损失，并且继承了它的一个不便之处，即它需要一个冗余的不可学习网络进行优化。在权重共享NAS的背景下，如果只使用(10)，不可避免地需要一个冗余的不可学习超网络，这使得优化变得困难和不优雅。接下来，我们的目标是去除这个冗余的不可学习超网络。

*避免冗余超网络*：现有的自监督算法依赖于冗余的不可学习网络的原因是，如果只有可学习网络，优化将会崩溃成一条捷径，例如不同样本的输出都是相同的常数向量。因此，避免冗余不可学习超网络的一种可能方法是鼓励不同样本的输出发散。受[56]的启发，我们对不同样本的输出施加损失函数，最大化它们的方差，即：

$$
L_{SDR} = \lambda_3 \sum_{k=1}^{K} \sum_{c=1}^{C} \max(0, \gamma - \text{std}(Z_{k_c})),
$$

其中max(0, )是一个类似铰链的损失，广泛应用于SVMs。 $\gamma$ 是一个阈值。std表示计算标准差，即std $(Z_{k_c}) = \sqrt{\text{Var}(Z_{k_c}) + \epsilon}$ 。 $\epsilon$ 是一个小常数， $Z_{k_c}$ 表示第k个块的第c个通道神经元。添加了上述超网络去冗余损失LSDR后，我们不再需要额外的不可学习超网络来训练我们的超网络。我们的最终损失是LSSL + LSDR。通过这种方式，自监督巧妙地与权重共享NAS结合，形成了我们的DNA++。

在DNA++中，我们的自监督可以是CNN，学生超网络是ViT。因此，我们可以搜索ViT。图4展示了DNA++的流程。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/7be6788f7bbf42ccbb503a1c2e2b35e1.png" width="70%" /> </div>


# IV. 实验
## A. 数据集
我们的方法在ImageNet [58]上进行了评估，这是实用NAS方法的基准。为了进行架构搜索，我们通过从原始训练集的每个类别中随机选择50张图像创建了一个包含50k图像的验证集。其余的图像用于超网络训练。搜索完成后，我们在原始训练集上从头开始重新训练找到的架构和缩放的架构，然后在原始验证集上进行测试。此外，我们在CIFAR-10和CIFAR-100 [59]上评估了迁移能力。为了更全面地评估泛化能力，我们在PASCAL VOC2012 [60]和ADE20K [61]上进行了语义分割任务，并在MS-COCO数据集 [62]上进行了目标检测任务。

*ImageNet NAS Bench*: 为了评估NAS方法，之前的方法通常从头开始重新训练搜索到的架构，这很难辨别改进是由于NAS的有效性还是重新训练技术。NAS基准存在，但通常基于小型数据集（例如CIFAR-10或ImageNet-Tiny）和基于单元的搜索空间。为了解决这个问题，我们在完整的ImageNet上创建了一个基准，使用移动设置搜索空间，包括23个随机采样的架构，其top-1准确率从73.58%到75.52%不等，与真实准确率分布一致。更多细节在附录中的图12在线可查。


## B. 搜索空间和架构细节
我们的搜索空间通过从操作候选中选择操作和选择具有不同通道和层数的单元来定义。它们包括ViT和MBConv搜索空间。

*MBConv搜索空间*：在这些空间中，操作候选是MBConvs [63]。我们使用了两个MBConv搜索空间。第一个在第IV-C节中使用，与大多数最近的NAS作品[2]、[3]、[49]、[64]类似，以确保公平比较，总共有6种操作类型，即卷积核大小的组合{3, 5, 7}和扩展率{3, 6}。附录中的表15在线提供了与现有NAS方法的详细搜索空间比较。为了在第IV-E节和IV-F节中快速评估，使用了更小的搜索空间，具有4种操作类型（卷积核大小为{3, 5}和扩展率{3, 6}）。此外，我们还搜索了具有不同通道数和层数的单元，如第III-C节中介绍的。前五个块中每个块有3个单元，最后一个块中有1个单元。每个单元的层数和通道数如表I所示。整个搜索空间包含2 × 10^17个架构。

我们使用EfﬁcientNet B7 [3]作为我们的教师模型进行超网络训练，因为它的性能处于前沿，与ResNeXt-101 [65]和其他手动设计的模型相比，计算成本相对较低。我们根据它们的过滤器数量将教师模型分成6个连续的块。这些块的细节如表I所示。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/ef046f4273f5456a97473d98fc3e6c82.png" width="70%" /> </div>


*ViT搜索空间*：在这个空间中，我们基于著名的DeiT [66]构建了我们的超网络。搜索空间设计受到可扩展网络[67]、[68]、[69]的启发，特别是DS-ViT [70]。我们通过嵌入维度（224, 448, 32）、头部数量（7, 14, 1）、MLP比率（2.5, 4, 0.5）和网络深度（12, 16, 1）定义候选操作。这里，(a, b, l)表示从a到b的范围，间隔为l的搜索空间。整个搜索空间被分成4个块，总共包含大约1.7 × 10^7个架构。

尽管我们的搜索空间/超网络是基于ViT的，但我们并没有使用ViT作为教师来指导我们的超网络训练。相反，我们采用了EfﬁcientNet-B1作为我们的教师，原因有两个。首先，尽管ViT性能高，但其计算复杂性（例如，参数数量）高于EfﬁcientNet-B1。其次，教师和学生超网络之间的架构差异有助于测试我们提出的块状自监督技术在解决架构偏移方面的有效性。

## C. 在MBConv搜索空间上进行搜索
为什么我们选择DNA和DNA+？本节仅测试DNA和DNA+，因为MBConv是高效的，适用于像DNA+这样的高训练复杂度算法。我们将DNA++留待在ViT搜索空间中进行测试，因为DNA+可以解决该情况下的架构偏移问题。

我们单独训练超网络中的每个单元20个周期，在相应块中教师的特征图的指导下进行。对于DNA和DNA+，我们使用0.002作为第一个块的起始学习率，其余块为0.005。我们使用Adam作为我们的优化器，并且每个周期将学习率减少0.9。

对于DNA/DNA+，使用8个NVIDIA GTX 2080Ti GPU训练一个典型的超网络需要三天。然而，简化超网络，每个块只包含一个单元，只需要一天。使用算法1，我们的架构评分成本约为0.6 GPU天。在特定约束下搜索最佳模型，我们在CPU上执行算法2，成本不到一个小时。

为了在ImageNet上重新训练我们搜索到的（和缩放的）架构，我们使用了与[3]相似的设置，即批量大小为4,096，RMSprop优化器，动量为0.9，初始学习率为0.256，每2.4个周期衰减0.97。我们的模型首先在没有额外技巧的情况下进行训练，以进行公平比较，然后使用RandAugment [71]来激发它们的潜力。对于迁移学习实验，我们遵循[79]和[80]的相同微调设置。

1) DNA。Top-1准确率：如表II所示，我们的DNA模型与最近的NAS模型相比取得了最先进的结果。首先，在350 M FLOPs的约束下，DNA-a超过了SCARLET-A，参数少了1.8 M。其次，为了与EfﬁcientNet-B0进行公平比较，我们在350 M FLOPs和5.3 M参数的约束下进一步执行我们的DNA，分别获得了DNA-b和DNA-c。DNA-b和DNA-c都以较大的优势（1.1%和1.5%）超过了EfﬁcientNet-B0。即使与最近表现良好的MixNet-M相比，我们的DNA-b在参数更少的情况下也超过了MixNet-M 0.4%。第三，无约束搜索时，我们的DNA-d在6.4 M参数和611 M FLOPs下达到了78.4%的top-1准确率。当以与EfﬁcientNet-B1相同的输入大小（240 × 240）进行测试时，DNA-d达到了78.8%的top-1准确率，准确度相当，但比EfﬁcientNet-B1小1.4 M。第四，使用RandAugment [71]时，DNA-c和DNA-d进一步提高到了78.1%和78.9%的top-1准确率。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/91f015f65f2546bdb63fef4e1f5df09c.png" width="70%" /> </div>


*架构可视化*：我们在附录中的图13在线可查，展示了我们搜索到的架构，从中我们有几点观察。i) 无约束搜索时，DNA-d倾向于选择相对昂贵的操作，具有高扩展率和大核大小，从而实现了最佳性能。这验证了我们的DNA能够在整个搜索空间中找到最优架构。ii) 在最大参数数量约束下，DNA-c倾向于丢弃具有冗余通道的操作以节省参数。它也倾向于在最后几个块中选择较低的扩展率，因为最后几个块的通道比前几个块多。iii) 在最大计算成本约束下，DNA-b和DNA-a倾向于在每个块中均匀选择具有较少通道和较低扩展率的操作。图13中高性能架构的显著风格差异证明了DNA的架构搜索能力。

*模型复杂度与模型准确率*：图5比较了大多数最近的NAS模型的模型大小与准确率以及FLOPs与准确率的曲线。我们的DNA模型比其他NAS模型在更小的模型大小和更低的计算复杂度下实现了更好的准确率。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/489b30d366774852a1b4994f176879dd.png" width="70%" /> </div>


*迁移能力*：为了测试我们搜索到的架构的迁移能力，我们在两个广泛使用的迁移学习数据集上评估了我们搜索到的架构，即CIFAR-10和CIFAR-100。结果是在表III中报告的。如所示，我们的模型在迁移后保持了优越性。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/195ecc34d3e84ba9ad271d22e87a353d.png" width="70%" /> </div>


*模型可扩展性*：由于我们搜索到的架构以显著较少的参数实现了非常高的准确率，为了充分探索我们搜索到的架构的性能，我们通过同时扩大DNA-c的深度、宽度和输入大小，将DNA-c扩展到不同的模型大小。尽管像EfﬁcientNet [3]那样使用网格搜索为我们的DNA寻找最优的规模策略会更好，但为了简单起见，我们只是借用了[3]的规模策略，而没有进行策略搜索，这使我们的方法处于不利地位。结果如表IV所示。可以看出，我们缩放的架构的准确率在所有级别上都超过了竞争的EfﬁcientNet，即使使用的规模策略可能由于缺乏策略搜索而不适合我们的DNA。左图5展示了散点图。总之，这个比较验证了我们搜索到的架构已经学到的通用特征。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/e905266515b343e6803e42c4540593a1.png" width="70%" /> </div>


*更公平的比较*：请注意在将我们的方法与最先进的方法进行比较时确保公平的重要性。在这方面有两点需要深思熟虑：首先，在MBConv搜索空间的验证过程中，我们搜索到的架构从头开始重新训练，没有使用知识蒸馏或微调。相反，最近的做法通常涉及使用超网络的预训练权重初始化搜索到的架构，或者使用更大的网络进行知识蒸馏。这些差异可能使我们的DNA方法处于潜在的不利地位。为了解决这种差异，我们在修订版中主动将知识蒸馏纳入了我们的重新训练过程。其次，鉴于我们对参数效率的强调，DNA家族在搜索阶段以参数数量为目标进行搜索。因此，有必要将其与表现出公平参数效率的最新先进技术进行比较。我们仔细比较了我们的方法与最新的先进技术，并彻底讨论了结果。如表II所示，这种更公平的比较揭示了DNA比以前的最先进的方法（如FBNetV3 [74]（79.7%对78.4%），AttentiveNAS [75]（79.7%对77.3%），AlphaNet [76]（79.7%对77.8%），EfﬁcientNetV2 [77]（79.7%对78.7%）和OnceForAll [78]（79.7%对76.4%））有显著的优势。

2) DNA+：DNA+是DNA的多代级联。如表V所示，第0代和第1代的搜索模型是Efﬁcient-b1和DNA-c。第0代和第1代的缩放搜索模型是Efﬁcient-b7和DNA-c1，它们进一步分别作为第1代和第2代的老师。第2代和第3代的搜索模型是DNA+2和DNA+3。对于每一代，我们使用我们的ImageNet NAS Bench评估超网络的排名能力。排名Kendall tau和搜索模型准确率被报告。在第1代准确率跃升了1.8%。在第2代，准确率进一步提高了0.2%，排名tau相对提高了约3%。排名相关性在第3代达到平稳，因为教师模型可能已经达到了搜索空间的上限。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/f2ce92b770d341d8ba70ed7d895e49b5.png" width="70%" /> </div>

## D. 搜索视觉变换器
为什么我们选择DNA和DNA++？本节重点测试DNA和DNA++，因为：首先，我们使用的教师是一个CNN，而要搜索的架构是ViTs，存在架构偏移。DNA++采用的自监督学习技术可以解决这个架构偏移问题。其次，DNA+耗时且已在高效的MBConv空间中进行了测试。

*超网络训练*：我们单独训练超网络中的每个单元20个周期，在教师模型的块状指导下进行。我们使用0.33和0.0025（对于256个总批量大小）作为DNA和DNA++的起始学习率。对于DNA，我们使用AdamW作为我们的优化器。DNA的学习率每3个周期减少到其0.9倍。对于DNA++，我们使用余弦调度器来减少学习率，并使用LARS作为优化器。使用8个NVIDIA GTX 2080Ti GPU训练一个典型的超网络，DNA和DNA++分别需要1天和2天。使用算法1，我们的架构评分成本约为0.6和0.1 GPU天。在特定约束下搜索最佳模型，我们在CPU上执行算法2，成本不到一个小时。

*重新训练搜索到的模型*：为了在ImageNet上重新训练我们搜索到的架构，我们使用了与[66]相似的设置，即批量大小为1,024，AdamW优化器，初始学习率为1e-3，余弦学习率调度器。权重衰减设置为5e-2。架构训练了300个周期，其中有5个周期用于学习率预热。我们使用了与DeiT [66]相似的正则化和增强，包括标签平滑[85]、随机深度[86]、Cutmix[87]、RandAugment[88]、随机擦除[89]和Mixup[90]。遵循[66]，我们还探索了使用KD进行训练，KD是用于训练ViTs的广泛使用的标准技术。

1）*DNA*：如表VI所示，我们的DNA模型与现有模型相比取得了最先进的结果。更具体地说，我们有三个关键发现。首先，我们DNA搜索的框架的性能与手工设计的框架的性能相匹配甚至超过了它。例如，在大约4.5 G FLOPs的计算下，我们的DNA-T超过了ViT-S/16 [81]和DeiT-S [66]，接近Swin-T [83]，略低于T2T-ViT-14 [82]。请注意，T2T-ViT-14的计算成本比我们的DNA-T更高（即5.2 G对4.5 G）。其次，我们的方法在知识蒸馏方面保持了对手工设计模型的优势（例如，对于DNA-T和DeiT-S，分别为82.8%和81.2%）。最后但同样重要的是，我们慷慨地承认我们的DNA-T略逊于最好的NAS方法。 

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/4437e72770964422956aa6807877abfa.png" width="70%" /> </div>


*分析*：有两个原因可以解释为什么我们的方法没有明显优势于现有的NAS方法。首先，现有的方法比我们的DNA-T计算成本更高（例如，BossNAS-T对DNA-T的5.7 G FLOPs对4.5 G FLOPs）。其次，我们的DNA-T在教师和学生之间存在架构偏移。具体来说，我们的教师是一个CNN，我们的学生是ViTs。在知识蒸馏过程中，与教师类似的操作往往具有较低的蒸馏损失。这使得搜索到的网络趋向于接近教师。这种偏见阻止了发现最佳架构。幸运的是，我们的DNA++可以解决NAS中教师和学生之间的架构偏移问题，这将在下一节中介绍。

2）*DNA++*：我们已经在表VI中广泛地定量比较了我们的DNA方法和这些应用于ViTs的最先进NAS方法。这些比较在表VI中广泛地展示。令人信服的结果明确表明，我们的方法在各种方法中一致地实现了更高的准确率水平，即使在类似的FLOPs下（例如，对于DNA对GLIT [38]、VITAS [39]、AutoFormer [40]、NASViT [42]、Twins [84]，分别为83.6%对80.5%、82.0%对81.7%、82.9%对82.7%）。

一方面，我们的DNA-T++在所有对比方法中实现了最佳性能，包括手工架构（例如，Twins [84]，提出了一种智能的注意力机制，用于视觉变换器）和自动搜索的架构。在与类似FLOPs的Twins [84]方法比较时，我们获得了以下结果：DNA-T++对Twins-PCPVT-B的83.6%对82.7%。此外，DNA-T++以明显的优势超过了BossNAS，这是当前视觉变换器搜索的最佳NAS方法（即82.0%对81.6%）。DNA++甚至也显著超过了当前表现最好的动态ViTs，如DS-ViT-L++ [70]（即83.6%对83.0%）。另一方面，DNA-T++远远超过了DNA-T（即，没有知识蒸馏的82.0%对81.2%，有知识蒸馏的83.6%对82.9%），这对我们非常鼓舞人心。DNA-T++对DNA-T的显著优势证实了自监督技术可以解决NAS中教师和学生之间的架构偏移问题。这个激动人心的结果不仅证明了DNA++的有效性，还证明了我们的DNA家族在面对网络结构搜索中的各种角落案例时的鲁棒性。 

## E. 有效性 
我们通过将DNA与最近的NAS方法在模型排名、定量有效性/效率分析、训练稳定性和特征可视化方面进行比较，来证明DNA的有效性，并提供更多关于DNA的见解。选择了三种方法，即DARTS [16]、SPOS [20]和MnasNet [2]，作为当前NAS三种主流方法的代表：联合优化的权重共享方法、一次性权重共享方法和独立地下训练方法。本节（即第IV-E节）的实验是在我们的ImageNet NAS Bench上进行的。 

*模型排名*：一个主要优势是我们DNA的可靠架构排名，这意味着未训练模型（即“预测模型排名”）的排名可以作为这些模型（即“真实模型排名”）的实际排名的指标。为了评估架构排名，在图6中，我们比较了DARTS、SPOS、MnasNet和DNA的“预测模型排名”（P）和“真实模型排名”（T）。所有这些方法都在相同的MBConv搜索空间中进行了测试（即第IV-B节中描述的较小的MBConv空间）。这些模型的真实准确率和“真实模型排名”是从我们的ImageNet NAS Bench获得的。我们使用这四种方法的未训练网络权重获得了它们的“预测模型排名”。我们的DNA超网络的每个块训练了20个周期，总共增加了120个周期。SPOS超网络也按照官方协议 [20] 训练了120个周期。我们通过使用超网络权重评估验证损失来获得DNA和SPOS的“预测模型排名”。DARTS超网络训练了50个周期，因为过度训练DARTS可能会导致性能下降，正如[91]所建议的。DARTS的“预测模型排名”是通过排序选择不同操作候选的预测联合概率获得的。至于MnasNet，每个架构都按照官方协议 [2] 独立训练了五个周期。MnasNet的“预测模型排名”是通过排名这些未训练模型的性能获得的。由于MnasNet的未训练设置没有详细提供，我们尝试了两种不同的学习率，即0.2和0.001，得到了两个非常不同的结果。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/3ff09a5f8b174738b7431179a71b07f8.png" width="70%" /> </div>


图6展示了结果。首先，SPOS和DARTS未能正确排名架构。这可能归因于大搜索空间引起的不良泛化能力，正如定理1所建议的。其次，由于独立训练，MnasNet的模型排名优于DARTS和SPOS。然而，MnasNet的成本相当高。一方面，MnasNet在获得Pareto前沿的过程中训练了10^13个架构中的8,000个，这耗费了288 TPU天 [2]。另一方面，对于MnasNet，评估一个架构需要五次ImageNet训练，而像SPOS和DNA这样的权重共享方法平均只需要10^10次ImageNet训练就可以评估一个架构。此外，MnasNet的排名结果高度依赖于超参数（见图6的最后两个子图）。上述两个观察结果验证了DNA在正确和高效评估架构方面的优越性，从而实现了有效和高效的架构搜索。

我们还绘制了不同NAS方法下不同架构在ImageNet上的真实准确率与其未训练模型的损失的关系，如图7所示。同样，我们可以发现，由我们的DNA获得的损失是架构真实性能的指标。相比之下，DARTS和SPOS预测的损失并不指示架构的真实准确率。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/526494abc3654bd5808e0433193ff399.png" width="70%" /> </div>


*定量分析*：我们进一步提供了对四种NAS方法的有效性和效率的定量分析。按照[8]、[9]，我们通过排名相关性度量来衡量有效性，例如Kendall Tau (τ) [92]、Spearman Rho (ρ)和Pearson R (R)。所有这三个度量的范围从-1到1，其中“-1”表示完全相反的排名，“1”表示完全正确的排名，而“0”表示排名之间没有相关性。我们通过搜索成本来衡量效率。

表VII显示了结果。首先，尽管DARTS以高效率著称，但由于大的内存占用，它不如SPOS和DNA高效。其次，DARTS、SPOS和学习率为0.2的MnasNet可能效果不佳——它们并不比或努力胜过随机架构选择（τ ≈ 0）。第三，学习率为0.001的MnasNet在有效性上取得了显著的好成绩（τ = 0.61），但效率极低（288 TPU天）。我们的DNA甚至获得了更好的有效性（τ = 0.64对τ = 0.61），效率也显著更高（8.5 GPU天对288 TPU天）。总之，DNA是唯一一个实现了高效性和有效性的方法。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/3f3d1657d76c4bd78c74a3b1ab7b6df8.png" width="70%" /> </div>


*训练稳定性*：为了进一步验证搜索有效性，我们检查DNA是否能够随着超网络训练的进行一致地找到更好的架构。我们比较了DNA与两种基于超网络的方法，即SPOS和DARTS。对于每种方法，我们在超网络训练过程中选择了8个中间检查点，并在每个检查点搜索最佳架构。然后，我们随机重新初始化架构的网络权重，并从头开始重新训练它们以获得它们的真实性能。图8显示了结果。首先，对于SPOS和DARTS，随着超网络训练的进行，搜索到的架构的性能随机波动，尽管超网络的训练损失不断减少。这使得好的架构几乎无法获得，因为我们无法决定超网络检查点的哪个周期是最优的。其次，我们搜索到的架构的准确率随着超网络训练的进行而逐步提高（即从75.7%提高到77.5%），直到16至20个周期之间的收敛。这意味着我们可以使用收敛的超网络（例如，最后一个周期）来搜索架构。请注意，准确率在早期阶段迅速提高，与训练损失减少的趋势相同，这证明了搜索到的架构的准确率与超网络的损失之间存在相关性。搜索到的架构性能的稳定提高证明了DNA的稳定性。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/9efbb46843554bc582cb9aa7d50ad3a0.png" width="70%" /> </div>


*可重复性/鲁棒性*：NAS因其难以复制而令人担忧[9]。例如，种子在复制DARTS结果中至关重要，可能是因为它们对初始权重的敏感性。为了评估DNA的可重复性，我们进行了重复实验，有两种消融类型：i) 不同的随机种子和ii) 不同的数据量。对于种子，我们选择了{0, 41, 42}，这影响了超网络权重的初始化和搜索过程中的随机路径采样。表VIII显示，对于不同的种子，模型排名是稳定的，证明了DNA的高可重复性和鲁棒性。对于数据量，我们随机抽取了ImageNet训练集的一个子集作为新的训练集，并保持总训练步骤不变。表IX显示，当训练数据减少到80%和进一步减少到40%时，DNA的模型排名保持稳定。不同数据量的详细模型排名比较如图9所示。为了消除规模方差的效应并给出更直观的比较，我们对总损失进行了标准化。每3个点与相同的准确度相关联。如图所示，每个模型的点紧密相连，三个数据量的回归线彼此重合。这证明了DNA对数据量的鲁棒性，意味着DNA可以是一个数据高效的少镜头NAS方法。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/249a8f2291234ce6bae5bfb5ca375069.png" width="70%" /> </div>
<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/f997ddf96e8547feb8371257e7b0ad91.png" width="70%" /> </div>
<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/4b08dd2273fb456da647619de16c48bb.png" width="70%" /> </div>


*特征图可视化*：图10展示了教师和学生的几个特征图（块2和4，周期16）。如图所示，尽管我们的学生超网络只有教师模型1/3的层数，但它可以很好地模仿教师。每个通道的纹理都非常接近，即使是在高度抽象的块4的14 × 14特征图上，证明了DNA在从教师中提取知识方面的有效性。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/c38f8ec55fca4a02aaddf713fc97bc49.png" width="70%" /> </div>


## F. 消融研究
*蒸馏策略*：我们在DNA中使用了并行蒸馏策略，即我们并行地蒸馏不同块的学生超网络，因为每个学生块的输入和监督都是教师的可以预计算的特征图。我们将我们的并行策略与两种渐进式蒸馏策略进行了比较。渐进式策略意味着逐个块地训练和搜索学生超网络，其中：一种训练超网络的第i个块，并在第1到i块中执行架构搜索以获得m个最优的块状架构；然后，用这m个块状架构，训练超网络的第i+1个块，并在第1到i+1块中执行架构搜索以获得m个最优的块状架构；循环重复，直到搜索完成。两种渐进式策略的区别在于：(A) 当训练超网络的第i个块时，所有的i个块都从头开始训练；(B) 当训练超网络的第i个块时，只有第i个块从头开始训练，超网络的前(i-1)个块的权重被冻结。表X中的结果证明了我们策略的优越性。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/bda4f32b60ff478995bde57642e0e855.png" width="70%" /> </div>


*单细胞与多细胞*：为了检验多细胞搜索的影响，我们执行了每个块中只有一个单元的DNA进行比较。如表X所示，多细胞搜索在5.3 M参数约束下提高了top-1准确率0.2%，在无约束下提高了0.3%。注意到单细胞情况在5.3 M参数约束下获得了参数更少的架构，这可能归因于通道和层数的可变性相对较低。

*教师依赖性*：由于我们在搜索阶段（但不是在重新训练阶段）采用了知识蒸馏，我们必须澄清我们搜索到的架构的性能是否受到教师容量的限制。首先，我们用一个较低级别的教师（即，EfﬁcientNet-B0）替换了我们的高级教师（即，EfﬁcientNet-B7）来指导在相同约束下的架构搜索。结果显示在表XI中。令人惊讶的是，使用EfﬁcientNet-B0作为教师指导的搜索架构的性能几乎与使用EfﬁcientNet-B7指导的相同，这表明我们DNA的性能并不一定依赖于高性能的教师。更重要的是，我们可以发现，使用EfﬁcientNet-B0作为教师搜索的DNAfrom-B0在相同模型大小（5.27 M对5.28 M）的情况下显著优于其教师1.5%，这证明了我们的架构蒸馏的性能不受教师性能的限制。如上所示，我们可以通过自我蒸馏架构来提高任何架构的性能。其次，由于我们搜索的架构DNA-c在参数数量显著减少（5.28 M对66.4 M）的情况下达到了与其教师相同的准确率，我们将DNA-c扩展到与教师相似的模型大小，按照[3]的方法。如输入和监督是教师模型的内部特征图，表XI中的EfﬁcientNet-B7和DNA-c7都在224 × 224的输入大小下进行了测试。此外，表XI中的DNA-c7也是在224 × 224的输入大小下训练的。引人注目的是，缩放的架构（即，DNA-c7）比教师（即，EfﬁcientNet-B7）高出2.1%，展示了我们DNA的实用性和可扩展性。第三（最后但同样重要），尽管我们的架构蒸馏的性能不受教师性能的限制，但反复将我们的学生规模扩大为新教师可以搜索越来越好的架构，正如我们的DNA+所建议的。这表明我们可以随机选择一个现有的模型作为我们的教师，而不需要投入太多的努力。然后，我们可以执行循环NAS来获得最优的架构。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/46d034c765d5421ab6105441d970a0c9.png" width="70%" /> </div>

## G. 在VOC12和ADE20 K上的语义分割
**PASCAL VOC12**。由于手工制作的架构通常在ImageNet/CIFAR分类之外表现出良好的泛化能力，我们以具有挑战性的PASCAL VOC12基准 [60]为例，检验我们搜索到的架构的通用性。该数据集包括21个类别，有10,582张训练图像([60], [93])和1,449张像素级标记的验证集图像。我们使用标准的像素交叉联合（mIOU）度量在21个类别上平均的准确率。我们将最先进的分割模型DeepLabv3+ [94]的骨干替换为我们的DNA-c3，形成DNA-Seg。其他协议与[94]一致。

*复杂度与准确率*：在VOC12分割的测试集上检查性能（即，争取排行榜冠军）通常包括许多未知的技巧（例如，在评估期间使用多尺度输入，添加左右翻转输入，预训练在MSCOCO数据集[62]上，预训练在JFT-300 M数据集[95]上，以及集成多模型），这使得不同方法的模型复杂度不明确。为了避免这种模糊性，我们在VOC12分割的验证集上比较了不同模型的复杂度和准确率，以进行公平比较。结果如图11所示。在相似的准确率约束下，我们的DNA-Seg比最近的DeepLabv3+(ResNet101)少用21倍的FLOPs和4.5倍的参数，比最近的DeepLabv3+(Xception)少用6倍的FLOPs和4.2倍的参数。特别是，使用单一模型和单一测试时间尺度，我们的DNA-Seg在12.75 M参数和14.01 FLOPs下达到了最先进的79.76%的验证IoU。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/33869dfaa1ec49bd9a5e5ee2e3d7412d.png" width="70%" /> </div>


*ADE20 K*：除了PASCAL VOC12，我们还在ADE20K [61]上进行了进一步的实验，扩大了我们的评估范围。对于分割任务，我们采用了Atrous Spatial Pyramid Pooling框架 [97]作为我们的分割器。结果，如表XII所示，清楚地证明了我们DNA方法比现有的高效骨干方法有显著的优势。例如，性能比较表明，我们DNA方法在40.7%的准确率上优于EfﬁcientNet-B0 [3]（38.9%）、EfﬁcientNet-B1 [3]（39.2%）和EfﬁcientNet-B2 [3]（40.4%）。此外，值得注意的是，即使FBNetV5 [96]在ADE20 K上报告的准确率（40.4%）也不如我们DNA方法的性能。这些发现证实了我们方法的有效性和优越性。

我们进一步将我们的方法与两种杰出的基于ViT的方法进行了基准测试，具体是NASViT [42]和S3 [43]。我们采用了DNA-T++作为我们选择的骨干架构。在UperNet配置下（与S3使用的设置相同），我们的模型在ADE20 k数据集上展示了卓越的准确率，达到了47.3%，超过了NASViT（41.4%）和S3（46.27%）。至关重要的是要强调，我们的模型在规模上比NASViT大（5.6 G对1.9 G），见表XIII。然而，值得一提的是，这个模型已经是NASViT论文和GitHub页面上可用的最大模型。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/d8b65be70f6a42d59dd7cab3063a0f71.png" width="70%" /> </div>

## H. 在MSCOCO上的目标检测
为了进一步评估搜索到的模型在各种任务中的可转移性，我们将DNA-d用作Faster R-CNN中骨干特征提取器的替代品，具体是使用conv4（C4）骨干，遵循FBNetV3 [74]论文。然后，我们继续比较这种配置与其他模型在COCO检测数据集上的性能。我们在MSCOCO数据集 [62]上与最先进的方法如FBNetV3 [74]和EfﬁcientNet-B0（根据FBNetV3论文[74]的推荐选择）进行了全面比较。结果，如图XIV所示，明确揭示了我们方法对这些尊敬的竞争对手的显著优势，超过了它们（32.3%对30.5%）。

<div align=center>   <img src="https://img-blog.csdnimg.cn/direct/85c9ba3b01b14052ba075ae715a43907.png" width="70%" /> </div>


# V. 结论

在本文中，我们使用一种泛化界限的工具将权重共享NAS的低效与由于巨大搜索空间导致的不可靠的架构评分联系起来。为了解决这个问题，我们将搜索空间模块化成块，并应用了蒸馏神经架构技术。我们探索了三种块级学习方法：监督学习（DNA）、渐进学习（DNA+）和自监督学习（DNA++）。我们的DNA家族评估了所有候选架构，这是对以前方法的一个显著进步，以前的方法仅限于通过启发式算法访问较小的子搜索空间。此外，我们的方法还支持在指定的计算约束下搜索具有可变深度和宽度的架构。认识到架构评分在NAS中的核心作用，我们提供了广泛的实验结果来仔细审查这一点。最后，我们的方法在各种任务上都取得了最先进的结果。未来的工作将扩展我们的DNA方法的应用到NLP、3D架构[98]、[99]、[100]和生成模型[101]、[102]、[103]。

# VI. 亮点与展望

NAS一直是深度学习和人工智能中一个重要的研究和开发领域。然而，可以想象，在某些时刻或特定背景下，NAS所受到的关注程度可能已经经历了演变或变化。有几个可以想象的原因可能解释了为什么NAS在某些时刻或特定背景下的关注度有所减少：(a) 架构排名：我们认为，次优的架构排名是导致NAS关注度降低的关键因素。具体来说，对于搜索过程的有效性存在显著的担忧，这使得研究人员和实践者感到担忧。更准确地说，这种担忧集中在搜索阶段程序性能与最终任务性能之间的相关性上。在没有这两个任务之间的强联系的情况下，研究人员可能对NAS采取谨慎的态度。(b) 架构的成熟：一些深度学习架构，如CNN和ViT，已经成熟并在广泛的应用中表现出强大的性能。如果没有解决架构排名问题，可能就没有必要寻找新的架构。(c) 计算成本：NAS在计算上是昂贵且耗时的，需要大量的计算资源来搜索最优的神经网络架构。研究人员和组织可能会优先考虑更有效的模型设计和优化方法，除非解决了我们论文中提出的架构排名问题。(d) 基础模型：基础模型[104]已成为许多AI任务的流行技术。

这些方法允许实践者利用现有的架构并将其适应于特定任务，而无需进行广泛的架构搜索，除非解决了我们论文中提出的架构排名问题。

正如显而易见的，解决架构排名问题有能力缓解上述所有挑战。因此，本文的核心重点是通过解决架构排名挑战，努力提高NAS方法的有效性。

# 声明

本文内容为论文学习收获分享，受限于知识能力，本文队员问的理解可能存在偏差，最终内容以原论文为准。本文信息旨在传播和学术交流，其内容由作者负责，不代表本号观点。文中作品文字、图片等如涉及内容、版权和其他问题，请及时与我们联系，我们将在第一时间回复并处理。
