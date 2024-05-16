# 题目：[Revisiting Person Re-Identification by Camera Selection](https://ieeexplore.ieee.org/document/10308725)  
## 通过摄像机选择重新识别个人
**作者：Yi-Xing Peng; Yuanxun Li; Wei-Shi Zheng** 

****
# 摘要

行人重识别（Person Re-ID）是视觉监控中的一个基础任务。给定目标人物的查询图像，传统的Re-ID专注于候选图像与查询图像之间的成对相似性。然而，传统Re-ID并未评估检索结果的一致性，即排名最前的图像是否包含同一个人，这在某些应用中是有风险的，例如遗漏了患者经过的地方可能会妨碍流行病学调查。在本项工作中，我们研究了一个更具挑战性的任务：在所有摄像头视图中一致且成功地检索到目标人物。我们将该任务定义为连续行人Re-ID，并提出了相应的评估指标，称为整体Rank-K准确性。与传统Re-ID不同，单个摄像头视图下的任何错误检索都会引发不一致，从而导致连续Re-ID失败。因此，摄像头部署对于跨摄像头视图的连续跟踪至关重要，我们从摄像头部署的角度重新思考行人Re-ID，并通过对连续Re-ID的评估来评估摄像头网络的质量。此外，我们提出了自动检测严重妨碍连续Re-ID的故障摄像头的方法。由于当摄像头网络变得复杂时，暴力搜索的成本很高，我们明确地模拟了摄像头之间的视觉关系以及空间关系，并开发了一个关系深度Q网络来选择适当部署的摄像头，而未选中的摄像头被视为故障摄像头。由于大多数现有的数据集并未提供有关摄像头网络拓扑信息，它们不适合研究空间关系在摄像头选择中的重要性。因此，我们收集了一个新的数据集，包括20个摄像头和拓扑信息。与随机移除摄像头相比，实验结果表明，我们的方法可以有效检测故障摄像头，以便在实践中对这些摄像头进行进一步操作。

# 关键词
- 摄像头选择（Camera selection）
- 行人重识别（Person re-identification）
- 视觉监控（Visual surveillance）

# I. INTRODUCTION

随着视觉监控的发展，无人值守的视觉监控系统越来越多地被部署在社会的各个方面。在视觉监控中，一个紧迫且具有挑战性的问题是行人重识别（Person Re-ID），其目标是匹配同一人在不同摄像头下的图像，尽管存在显著的跨视图失真。随着深度卷积网络的使用，现有的行人Re-ID研究工作已经从手工制作的特征转移到了网络架构以及目标函数上，并在基准数据集上取得了巨大的进步。

传统上，Re-ID评估成对图像之间的配对相似性，并通过根据它们与查询图像的相似性对候选图像进行排名来操作。然而，传统的Re-ID忽略了量化检索结果的一致性，即排名列表中最具相似性的图像是否包含同一个人。例如，如图1所示，尽管在其他摄像头下的常规Re-ID检索结果是正确的，但在蓝色摄像头下检索到的第一个图像包含了与在其他摄像头视图下检索到的图像不同的人，导致了不一致性。

一致的检索结果是一些现实世界应用至关重要的。例如，期望Re-ID系统能够为流行病学调查产生一致的检索结果，以便人们可以找出感染者所经过的所有地方，并防止病毒的传播。因此，提出了一个更具挑战性但较少探索的问题：连续行人Re-ID（见图1）。与成对相似性不同，连续Re-ID关注所有摄像头视图中排名最高的图像是否包含同一个人。本质上，连续Re-ID是在摄像头网络中跟踪人物，而传统的Re-ID仅检索查询人物的图像。以图1中的示例为例，我们可以根据连续Re-ID和视觉监控系统中的时间信息发现查询人物的移动路径。因此，我们提出了一个新的评估指标，称为“整体”Rank-K准确性，它表示通过从每个摄像头视图中检索K个图像来成功完成连续Re-ID的概率（见图2）。

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/d6a61a00d7be44cfa3c91f671bde0288.png#pic_center" width="70%" />
</div>

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/720206eb13a1457bae865632e086d5fa.png#pic_center" width="70%" />
</div>

连续Re-ID与摄像头网络中的摄像头部署密切相关。通常，为了在多个摄像头视图之间进行连续跟踪，如果Re-ID系统频繁地无法将一个摄像头视图下的图像与来自其他视图的图像关联起来，那么该摄像头可能成为摄像头网络的缺陷，因为它阻碍了连续Re-ID。因此，我们从摄像头部署的角度重新思考行人Re-ID。为了便于人类操作员有效且高效地改进摄像头部署，我们提出基于连续Re-ID评估摄像头网络的质量，并开发了一个摄像头代理来自动检测大幅降低跨摄像头视图的连续跟踪性能的摄像头。

在摄像头网络中检测故障摄像头是具有挑战性的。它不仅由环境因素决定，还受到摄像头网络中摄像头之间的全局一致性的影响，因为每个摄像头都希望能够为智能视觉监控（如连续Re-ID）提供一致且连续的信息。虽然直接方法是对故障摄像头应用暴力搜索，但随着摄像头数量的增加，时间成本呈指数级增长，使其变得不切实际。此外，暴力搜索缺乏灵活性，因为它很难回答诸如哪个摄像头是摄像头网络中最大的缺陷之类的问题。

因此，我们应用深度强化学习来训练一个摄像头选择的代理。具体来说，代理应该评估每个摄像头对摄像头网络的影响（表示为动作-价值函数），并相应地选择摄像头网络中某些适当部署的摄像头。通过这种方式，未选中的摄像头被视为故障摄像头。为了评估检测到的故障摄像头对连续Re-ID的负面影响，我们在移除代理检测到的故障摄像头之前和之后执行连续Re-ID，并计算性能提升来证明摄像头网络一致性的改进。

我们推测摄像头之间的关系对于检测故障摄像头至关重要，因为故障摄像头下的图像很难与其他图像关联。因此，我们开发了一个关系深度Q网络，它明确地模拟了摄像头之间的空间和视觉关系，以评估单个摄像头对整个摄像头网络的影响。除了关系建模，我们还引入了一种视觉质量评估技术，用于每个摄像头，以改进摄像头选择，因为每个摄像头捕获的图像质量是检测故障摄像头的重要线索。最后，我们开发的摄像头代理可以全面评估单个摄像头的影响，并找出故障摄像头，这有助于人类操作员采取进一步操作，改进摄像头部署，以跟踪跨越多个摄像头视图的人员。

由于大多数现有的数据集（例如MSMT17 [16]）不提供摄像头网络的拓扑信息，因此很难有效地模拟摄像头之间的空间关系。因此，我们收集了一个新的基准数据集，包括20个摄像头和隐私协议，以促进对连续Re-ID和摄像头选择的调查。

本工作的主要贡献有三个方面。
- 据我们所知，我们是第一个调查连续Re-ID问题并相应提出一个新指标，即整体Rank-K准确性。我们提出利用连续Re-ID评估摄像头部署的质量。
- 此外，为了自动检测摄像头网络中的故障摄像头，我们提出了一个关系深度Q网络，可以利用摄像头之间的关系来评估每个摄像头在摄像头网络中的影响。我们的实验结果表明了故障摄像头对跨多个摄像头视图的连续跟踪的负面影响以及我们方法的有效性。
- 我们引入了一个新的基准数据集，称为多摄像头连续行人Re-ID数据集（MC-CPD），以更好地调查连续行人Re-ID和摄像头选择。它比现有的公共标记数据集拥有更多的摄像头，并包含丰富的跨视图变化。它还包含详细的摄像头部署拓扑信息。

本文的组织如下。第二节回顾了行人Re-ID的相关研究工作。第三节形式化了连续Re-ID以及所提出的评估指标。第四节形式化了摄像头选择问题，并详细介绍了所提出的关系深度Q网络，用于自动检测故障摄像头。第五节介绍了我们数据集的详细信息。第六节报告了实验设置，并展示了实验结果以及分析。第七节扩展了我们的方法，用于内部摄像头监督摄像头选择，该方法无需昂贵的跨视图注释。我们最后在第八节总结了我们的工作。

# II. RELATED WORKS

## A. Evaluation Metrics for Re-ID

行人重识别旨在识别不同摄像头下的行人。因此，在评估行人Re-ID模型时，几乎所有研究都采用了CMC（累积匹配特征）曲线。CMC曲线反映了成功检索目标人物图像的前K次的准确性。不考虑画廊中有多少真实匹配，在CMC计算中只计算第一个匹配。在存在多个真实情况的情况下，Zheng等人提出了使用mAP（平均精度均值）进行评估，并且mAP有利于那些真实情况排在前面的结果。最近，Ye等人引入了一种新的指标，称为mINP（平均逆负惩罚），它关注最难正确检索的排名结果。

然而，广泛使用的CMC曲线、mAP和mINP并不适合连续Re-ID，因为它们不评估检索结果的一致性。如图2所示，一个在传统指标上表现良好的Re-ID模型并不一定比其它模型更适合连续Re-ID。因此，我们提出了一个新的指标，称为整体Rank-K准确性。与现有指标不同，整体Rank-K准确性评估检索结果的一致性，并计算在摄像头网络中一致且成功捕获目标人物的概率。

## B. Person Re-ID With Multiple Cameras

对于跨多个摄像头的Re-ID，传统方法集中在设计手工制作的特征和从标记数据中学习距离度量。此外，摄像头网络的拓扑结构也被用来提高成对匹配的效率和效果。然而，手工制作的特征很难克服人体姿态、背景、照明等的大幅度变化，使得它们的性能不令人满意。随着深度学习的飞速发展，深度卷积神经网络已经主导了这一领域，因为它能够自动从大量标记数据中学习到区分性的特征嵌入。最近，研究工作集中在网络架构和目标函数上，这些方法在基准数据集上取得了令人印象深刻的性能。

一些现有的工作关注来自多个摄像头的检索，旨在提高Re-ID模型在CMC和mAP方面的性能。他们提到，给定查询从不同摄像头的检索结果应该一致地包含同一个人，否则会出现冲突。他们将一致性表述为目标函数并对其进行优化，以获得更好的Re-ID模型。然而，由于更强大的网络架构的存在，深度模型可以很容易地在训练集中获得不同摄像头之间一致的匹配关系。换句话说，在训练后，给定训练集中的一个行人，Re-ID模型可以在每个摄像头视图下一致地识别目标人物。此外，施加一致性约束的方法在评估时非常耗时，当测试集变大时很难应用。总之，现有方法不考虑摄像头部署，并且不能检测摄像头网络中的故障摄像头。它们的工作重点是图像之间的关系，我们实验表明，它们受到故障摄像头的困扰。

与现有工作不同，我们研究了连续Re-ID，它衡量了一个方法在多个摄像头上一致地跟踪查询人物的性能；然而，传统的Re-ID只提供了从画廊集中检索查询人物的图像，而没有指示它是否成功地在每个摄像头视图中匹配了查询人物。此外，现有工作不考虑检测故障摄像头。相比之下，我们利用连续Re-ID评估摄像头部署的质量，并提出自动检测摄像头网络中的故障摄像头。我们引入了一个关系深度Q网络，以提供有关单个摄像头对摄像头网络影响的信息。实验结果表明，故障摄像头有效地被我们的方法检测到，这些摄像头妨碍了连续监控。我们还扩展了我们的方法到内部摄像头监督摄像头选择，它利用轨迹检测故障摄像头，并且无需身份标签。我们认为摄像头部署对于视觉监控至关重要，我们的工作可以帮助改进部署。

## C. Camera Selection

一些现有的摄像头选择工作集中在为体育赛事的广播选择摄像头视图。他们的目标与我们的不同。具体来说，我们试图发现摄像头网络中的故障摄像头，而他们寻找的是每个时刻的最佳质量摄像头：他们一次只选择一个摄像头，用于体育赛事的最佳广播，并且不考虑摄像头之间的关系。因此，他们的方法不适合我们的问题。

还有一些研究关于摄像头选择用于跟踪，其中候选摄像头视场是重叠的。他们需要目标人物的位置来选择连续的摄像头进行跟踪，这在我们的工作中是不可用的，他们选择的摄像头与特定行人的移动有关。不同地，我们的工作旨在检测摄像头网络上的故障摄像头，而不是为跟踪特定人物选择连续的摄像头。此外，我们工作中的摄像头视场不重叠。

总之，先前工作的动机与我们的完全不同，这使得现有的方法不能应用于我们的问题。

# III. CONTINUOUS PERSON RE-IDENTIFICATION

## Problem Formulation
假设在摄像头网络中有 $M$ 个摄像头，以及一组画廊图像，记为 $G = \bigcup_{i=1}^{M} G_i$ ，其中 $G_i = \( x_{i,g}, y_{i,g} \) ^{N_{Gi}}_ {g=1}$ ， $x_{i,g}$ 表示第 $i$ 个视图中的第 $g$ 个画廊图像，带有身份标签 $y_{i,g}$ ， $N_{Gi}$ 是 $Gi$ 中图像的数量。假设Re-ID系统由一个特征提取器 $f(\cdot)$（通常是手工制作的特征描述符或深度模型）组成，该提取器将图像 $x_{i,g}$ 映射到特征嵌入 $f(x_{i,g})$ ，以及一个相似性度量 $sim(\cdot)$ ，用于测量两个图像特征之间的相似性。给定一个查询图像 $x_q$ ，连续Re-ID的目标是在摄像头网络的每个可能的摄像头视图中一致且成功地检索包含与 $x_q$ 相同人物的图像。

正式地说，在第 $i$ 个视图下，我们根据查询图像和画廊图像之间的相似性，将第 $i$ 个视图的画廊图像集 $G_i$ 按降序排序，以获得 $G_{q_i} = \( x_{i,gq_h} \)^{N_{Gi}}_ {h=1}$ ，其中 $x_{i,gq_h}$ 表示在第 $i$ 个视图中与查询图像 $x_q$ 最相似的第 $h$ 个画廊图像，即满足：

$$
sim(f(x_{i,gq_h}), f(x_q)) \geq sim(f(x_{i,gq_{h'}}), f(x_q)) \quad \text{for} \ h < h'.
$$

类似地，我们可以获得 $M$ 个排序的画廊集合，每个集合都是一个摄像头视图中的排名列表。最后，我们通过从每个排序的画廊集合 $G_{q_i}$ 中检索前 $K$ 个图像来获得检索结果 $G_{q,K}$ ，记为 $G_{q,K} = \bigcup_{i=1}^{M} G_{q,K_i}$ ，其中 $G_{q,K_i} = \( x_{i,gq_1}, \ldots, x_{i,gq_K} \)$ 。如果存在至少一个图像在 $G_{q,K_i}$ 中包含与 $x_q$ 相同的人物，我们认为 Re-ID 系统能够在第 $i$ 个摄像头下识别 $x_q$ ，并记为 $Acc_{K_i}^q = 1$ ；否则为零。如果对于所有 $i = 1, \ldots, M$ ， $Acc_{K_i}^q = 1$ 都成立，我们认为 Re-ID 系统能够一致地识别所有摄像头视图中的 $x_q$ ，并记为 $Acc_K^q = 1$ ；否则为零，即 $Acc_K^q = \prod_{i=1}^{M} Acc_{K_i}^q$ 。

注意， $x_q$ 中的人物可能没有出现在所有 $M$ 个摄像头视图中。在计算连续Re-ID模型的整体Rank-K准确性时，对于查询人物没有出现的摄像头（例如，第 $i$ 个摄像头视图），我们忽略这些摄像头并设置 $Acc_{K_i}^q = 1$ 。

## Overall Rank-K Accuracy. A New Evaluation Metric
给定查询集 $Q$ 和画廊集 $G$ ，Re-ID 系统的整体 Rank-K 准确性，由特征提取器 $f(\cdot)$ 和相似性度量 $sim(\cdot)$ 组成，计算为：

$$
ORank-K(Q, G, f(\cdot), sim(\cdot)) = \frac{1}{N_Q} \sum_{q} Acc_K^q,
$$

其中 $N_Q$ 是查询图像的数量。

## Comparison to Conventional Rank-K Accuracy
整体 Rank-K 准确性与传统的 Rank-K 准确性不同。作为比较，我们重新审视传统的 Rank-K 准确性。给定一个查询图像 $x_q$ ，传统的 Re-ID 会将整个画廊集合 $G$ 进行排序。如果在排序后的画廊集合的前 $K$ 个图像中包含了目标人物，则记为 $\hat{Acc}_K^q = 1$ ；否则为零。然后，传统的 Rank-K 准确性计算为：

$$
Rank-K(Q, G, f(\cdot), sim(\cdot)) = \frac{1}{N_Q} \sum_{q} \hat{Acc}_K^q.
$$

通过比较整体 Rank-K 准确性和传统 Rank-K 准确性的计算，我们可以看到，传统的 Re-ID 旨在检索查询人物的图像，但它不测量 Re-ID 模型是否能够在每个摄像头视图中成功匹配查询人物。不同地，连续 Re-ID 旨在跨多个摄像头跟踪查询人物。

# IV. 通过连续重识别进行摄像机选择

发现对连续重识别性能有显著影响的有缺陷摄像机非常重要。在本节中，我们的目标是基于连续重识别自动检测有缺陷的摄像机。假设存在一个训练集  $X = \bigcup_{i} X_i$ ，其中  $X_i = \{ x_{i,j}, y_{i,j} \}^{N_{X_i}}_ {j=1}$ ， $N_{X_i}$  是  $X_i$  中图像的数量。我们开发了一个摄像机代理来自动选择摄像机网络中相对适当部署的摄像机。给定摄像机网络中的训练集  $X$  和一个定制的摄像机数量  $\hat{M}$ ，摄像机代理应该输出一个  $\hat{M}$  摄像机的序列。如果所选摄像机部署得当，那么在所选摄像机下的连续重识别系统应该表现良好。这样，未选中的其他摄像机被视为有缺陷的摄像机。我们通过实验发现，有效发现有缺陷的摄像机，并且当适当数量的有缺陷摄像机被移除时，重识别性能变得稳定。

主要挑战是 1) 如何处理巨大的搜索空间（例如，在 20 个摄像机中选择 10 个摄像机有 184,756 种可能的结果）和 2) 如何评估每个摄像机对摄像机网络的影响，以便人类操作员可以获得足够的信息进行摄像机调整。为了克服这些挑战并获得所需的摄像机代理，我们首先介绍了一个基于深度强化学习的基础方法。然后，我们提出了一个关系深度 Q-网络来利用摄像机之间的关系进行改进。

## A. 摄像机选择的基本框架

摄像机选择的策略基本上遵循马尔可夫决策过程（MDP），在每个时间  $t$ ，摄像机代理处于状态  $s_t$  并且可以在状态  $s_t$  中采取一个动作  $a_t$ 。采取动作  $a_t$  后，状态从  $s_t$  转移到新的状态  $s_{t+1} = T(s_t, a_t)$ ，并且代理获得相应的奖励  $r_t = R(s_t, a_t)$ 。在我们的MDP公式中，摄像机代理的目标是检测并移除有缺陷的摄像机。

**状态（State）**：状态  $s_t$  表示时间  $t$  的摄像机部署方案。注意，摄像机网络中有  $M$  个摄像机，因此  $s_t$  被定义为一个  $M$  维的二进制向量。向量  $s_t$  中的第  $i$  个元素  $s_{i_t}$  表示第  $i$  个摄像机的状态。如果  $s_{i_t} = 1$ ，则表示第  $i$  个摄像机处于激活状态，否则它是被停用的。被停用的摄像机被视为检测到的有缺陷的摄像机。

**动作（Action）**：由于我们的目标是检测有缺陷的摄像机，在每个状态  $s_t$  中，可用的动作是移除一个可能干扰连续视觉监控的激活摄像机。动作  $a_t$  被定义为一个  $M$  维的单热编码向量，其中第  $i$  个元素  $a_{i_t} = 1$  表示停用第  $i$  个摄像机。当前动作影响的摄像机称为动作摄像机。对于  $a_t$  有两个约束条件：

$$
\sum_{i} a_{i_t} = 1
$$

$$
\sum_{i} s_{i_t} \cdot a_{i_t} = 1
$$

第一个方程约束动作只能停用一个摄像机，第二个方程进一步约束动作只能对一个激活的摄像机起作用。状态转移函数如下：

$$
s_{t+1} = T(s_t, a_t) = s_t \oplus a_t
$$

其中  $\oplus$  表示逻辑异或操作， $T(\cdot, \cdot)$  是状态转移函数。由于  $a_t$  的约束存在，连续两个状态  $(s_t, s_{t+1})$  之间的差异是一个激活的摄像机变为停用状态。

**状态值（Value of a State）**：直观上，状态  $s_t$  的值是状态  $s_t$  下摄像机部署的质量，它可以通过激活摄像机中的连续重识别性能来反映。

我们假设有一个预训练的特征提取器  $f_e(\cdot)$  是可用的，因为有公共数据集如 Market-1501 用于预训练。使用标记的训练集  $X$ ，我们在状态  $s_t$  下评估预训练特征提取器  $f_e(\cdot)$  的连续重识别性能。具体来说，在状态  $s_t$  中，我们从  $X_q$  和  $X_g$  中选择激活摄像机下的图像，分别表示为  $X_{q_{s_t}}$  和  $X_{g_{s_t}}$ 。

$$
X_{q_{s_t}} = \bigcup_{x_{i,q} \in X_q, s_{i_t} = 1} x_{i,q}
$$

$$
X_{g_{s_t}} = \bigcup_{x_{i,g} \in X_g, s_{i_t} = 1} x_{i,g}
$$

因此，我们使用  $f_e(\cdot)$  在  $X_{q_{s_t}}$  和  $X_{g_{s_t}}$  上的整体 Rank-1 准确性来表示状态值  $v(s_t)$ ：

$$
v(s_t) = ORank-1(X_{q_{s_t}}, X_{g_{s_t}}, f_e(\cdot), cos(\cdot))
$$

其中余弦相似度  $cos(\cdot)$  被用作相似度度量，遵循之前工作的做法。

**奖励（Reward）**：奖励  $r_t$  是一个标量，表示在排除动作摄像机后整个摄像机网络的一致性的即时增益。奖励  $r_t$  允许代理学习在采取动作  $a_t$  之前和之后状态值的差异。基于上述价值函数  $(9)$ ，奖励函数  $R(\cdot, \cdot)$  被制定为：

$$
r_t = R(s_t, a_t) = \tanh (v(s_{t+1}) - v(s_t))
$$

其中  $s_{t+1}$  在  $(6)$  中定义。

**训练（Training）**：我们的目标是学习理想的累积奖励  $Q^\*(s_t, a_t)$ ，这也被称为动作-价值函数。直观上， $Q^\*(s_t, a_t)$  表示在  $s_t$  状态下采取动作  $a_t$  的累积增益，移除有缺陷的摄像机应该带来较大的累积增益。累积奖励  $Q^\*(s_t, a_t)$  满足：

$$
Q^\*(s_t, a_t) = r_t + \gamma \max_{a_{t+1}} Q^\*(s_{t+1}, a_{t+1})
$$

其中  $\gamma$  是一个参数，用于控制未来奖励的重要性。当  $\gamma > 0$  时， $Q^\*(s_t, a_t)$  表示在采取动作  $a_t$  后的预期累积奖励，这是当前奖励  $r_t$  与下一个状态  $s_{t+1}$  中预期累积奖励的总和。当  $\gamma = 0$  时， $Q^\*(s_t, a_t)$  等于  $r_t$ 。我们使用常用的深度 Q-网络（DQN） $f_\theta(\cdot, \cdot)$ ，它由全连接层和激活层组成，根据给定的状态向量和动作向量来近似  $Q^\*(\cdot, \cdot)$ 。显然， $f_\theta(\cdot, \cdot)$  的输出应该满足  $(11)$ 。因此，学习  $f_\theta(\cdot, \cdot)$  的目标函数是：

$$
\hat{L}_ {RL} = \| r_t + \gamma \max_{a_{t+1}} f_\theta(s_{t+1}, a_{t+1}) - f_\theta(s_t, a_t) \|^2_2
$$

在这里，为了稳定学习过程，通常在强化学习中会开发一个辅助网络  $f_{\theta'}(\cdot, \cdot)$ ：

$$
\bar{L}_ {RL} = \| r_t + \gamma \max_{a_{t+1}} f_\theta(s_{t+1}, a_{t+1}) - f_\theta(s_t, a_t) \|^2_2
$$

其中  $f_{\theta'}(\cdot, \cdot)$  是通过  $f_\theta(\cdot, \cdot)$  动量更新的，或者每几个训练迭代更新一次的  $f_\theta(\cdot, \cdot)$  的副本。

每次迭代包含几个集数。在每个集数中，状态从充满 1 的二进制向量开始，表示所有摄像机在初始状态  $s_0$  中都被选中。摄像机代理开始采取行动，逐渐移除有缺陷的摄像机，当摄像机数量减少到 1 时，集数结束。在训练过程中，代理采用 ϵ-贪婪策略，以概率 ϵ 采取动作  $a_t = \arg\max_{a_t} f_\theta(s_t, a_t)$  或以概率  $1-\epsilon$  进行随机行动。

**评估（Evaluation）**：从状态  $s_0 = [1, ..., 1]$  开始，这是一个用 1 填充的  $M$  维向量，摄像机代理根据  $f_\theta(s_0, a_0)$  采取行动以最大化奖励，然后根据  $(6)$  获得下一个状态  $s_1$ 。每次摄像机代理采取行动  $a_t = \arg\max_{a_t} f_\theta(s_t, a_t)$ ，就移除一个摄像机。这个过程一直持续，直到只剩下  $\hat{M}$  个激活的摄像机。剩下的  $\hat{M}$  个激活的摄像机被视为部署良好的摄像机，移除的摄像机被视为有缺陷的摄像机。

**讨论**：上述价值函数的一个关注点是特征提取器  $f_e(\cdot)$  是在另一个领域（即源领域）中训练的，如果直接将其应用于当前的摄像机网络，可能会存在领域差距。然而，并不要求  $f_e(\cdot)$  在状态  $s_t$  上表现良好，关键在于不同状态之间的相对性能比较，这对于我们的工作是关键。具体来说，如果  $f_e(\cdot)$  在  $s_t$  上的表现优于  $s_t'$ ，那么状态值之间的差异，即  $(v(s_t) - v(s_t'))$ ，将表明  $s_t$  比  $s_t'$  好多少。状态值之间的差异对于引导摄像机代理检测有缺陷的摄像机是重要的，因为无论应用哪个特征提取器，有缺陷的摄像机之间的重新识别都是相对更困难的。另一个关注点是标记数据  $X$  的成本很高。为了解决这个问题，我们在本文的倒数第二节中采取了一步，将我们的方法扩展到不需要身份标签的摄像机内监督摄像机选择，该方法利用轨迹检测有缺陷的摄像机。

除了上述关注点，基本框架中存在一个关键弱点，即基本框架仅将状态和动作作为输入。因此，它缺乏对摄像机之间关系的建模能力，这限制了其性能。

## B. 关系深度Q网络用于摄像机选择

从多个摄像机视图的连续跟踪角度来看，我们推测摄像机之间的关系对于检测有缺陷的摄像机至关重要，因为连续跟踪要求一个摄像机视图下的图像能够与其它视图下的图像关联。因此，我们开发了一个关系深度Q网络（relational deep Q-network），以明确地模拟摄像机之间的关系，更好地评估每个摄像机的影响。我们首先考虑了哪些类型的摄像机关系对摄像机选择有益。一方面，存在视觉关系，因为具有相似拍摄角度和亮度的摄像机有助于个人重识别的成功。另一方面，相邻摄像机捕获的信息可能相似。因此，空间关系也应该被考虑，因为模型可能会过分关注局部摄像机组，从而无法捕捉摄像机网络中的全局关系。

基于上述考虑，我们根据摄像机的视觉和空间特征探索了摄像机之间的关系。视觉模型从每个摄像机视图中捕获的原始帧中提取视觉信息，我们基于摄像机之间的物理距离计算空间关系。关系聚合模型将从视觉信息中挖掘视觉关系，并将视觉关系与空间关系聚合。然后，预测模型  $fr(\cdot)$  利用聚合信息来近似动作-价值函数。关系深度Q网络的架构如图3所示。

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/37ce378a718646c5bd571deec0abf88d.png#pic_center" width="70%" />
</div>

**视觉模型**：视觉模型采用卷积神经网络  $fv(\cdot)$  来提取特征。 $fv(\cdot)$  接受被选中摄像机捕获的一批原始帧作为输入，并提取视觉信息。在我们的实验中，我们采用了在ImageNet上预训练的ResNet50作为  $fv(\cdot)$ ，因为它可以提取从ImageNet学习到的多样化视觉特征。基于状态  $s_t$  和动作  $a_t$ ，视觉特征进一步被分为两组，一组代表将被  $a_t$  移除的摄像机的视觉特征，另一组代表状态  $s_t$  中剩余摄像机的视觉特征。

假设第  $i$  个摄像机为  $c_i$ 。假设在状态  $s_t$  和动作  $a_t$  下，剩余的摄像机为  $c_1, ..., c_{i-1}, c_{i+1}, ..., c_{i+k}$ ，并且摄像机  $c_i$  是将被动作  $a_t$  移除的动作摄像机。 $\hat{X}_ t^i = \{ \hat{x}_ {j}^i \}^{|\hat{X}_ t^i|}_{j=1}$  表示在时间戳  $t$  从摄像机  $c_i$  采样的一批原始帧。剩余摄像机的数量为  $M_r = (i + k - 1)$ ，视觉模型的输出可以被公式化如下：

$$
Fv( \hat{X}_ t^i ) = \frac{1}{|\hat{X}_ t^i|} \sum_{j} fv(\hat{x}_{j}^i),
$$

$$
\bar{V}_t^r = [Fv( \hat{X}_t^1), ..., Fv( \hat{X}_t^{i-1}), Fv( \hat{X}_t^{i+1}), ..., Fv( \hat{X}_t^{i+k})]^T,
$$

其中  $Fv( \hat{X}_t^i ) \in \mathbb{R}^{d_v \times 1}$  是动作摄像机  $c_i$  的视觉信息， $\bar{V}_t^r \in \mathbb{R}^{M_r \times d_v}$  代表剩余摄像机的视觉信息。 $d_v$  是特征维度。

**空间拓扑关系**：空间关系是基于摄像机之间的物理距离建模的。假设摄像机之间的空间距离是可用的，并且第  $i$  个摄像机和第  $k$  个摄像机之间的距离被表示为  $d_{i,k}$ 。

基于状态  $s_t$  和动作  $a_t$ ，剩余摄像机和动作摄像机之间的距离被认为是空间拓扑关系  $\omega_G$ ，其维度与剩余摄像机的数量一致。正式地说，如果有  $i + k$  个激活的摄像机在状态  $s_t$  中，并且动作  $a_t$  是停用第  $i$  个摄像机，我们有：

$$
\omega_G = [d_{1,i}, ..., d_{i-1,i}, d_{i+1,i}, ..., d_{i+k,i}] .
$$

**关系聚合模型**：为了评估在状态  $s_t$  下采取动作  $a_t$  的影响，关系聚合模型首先捕获动作摄像机和剩余摄像机之间的视觉关系，然后将视觉关系和空间关系聚合。综合关系被用来学习动作-价值函数。

视觉关系是通过注意力机制来挖掘的。给定动作摄像机的视觉特征  $Fv( \hat{X}_t^i )$  和剩余摄像机的  $\bar{V}_t^r$ ，关系聚合模型通过不同的线性投影将它们映射到查询  $z_t^a$ 、键  $K_t^r$  和值  $V_t^r$ ，以发现它们之间的潜在关系：

$$
z_t^a = (Fv( \hat{X}_t^i))^T W^Q,
$$

$$
K_t^r = \bar{V}_t^r W^K,
$$

$$
V_t^r = \bar{V}_t^r W^V,
$$

其中  $W^Q, W^K, W^V \in \mathbb{R}^{d_v \times d_q}$  是三个可学习的线性投影。从潜在视觉特征中派生出的  $K_t^r$  和  $z_t^a$  的内积代表了不同摄像机视觉特征之间的相似性。因此，我们计算动作摄像机和剩余摄像机之间的视觉关系  $\omega_V$  如下：

$$
\omega_V = z_t^a (K_t^r)^T.
$$

由于相邻的摄像机容易具有相似的视觉特征，仅考虑视觉关系会使模型倾向于关注局部区域的一组摄像机，这不是我们期望的。为此，我们利用空间关系鼓励模型捕捉摄像机网络中的全局关系。我们通过逐元素乘法将视觉关系  $\omega_V$  和空间拓扑关系  $\omega_G$  结合起来，以获得动作摄像机和剩余摄像机之间更全面的关系。应用softmax函数进行归一化，我们获得摄像机之间的全面关系：

$$
\omega = \text{softmax} \left( \frac{\omega_G \odot \omega_V}{\hat{d}_q} \right),
$$

其中  $\hat{d}_q$  是  $z_t^a$  的特征维度， $\odot$  是逐元素乘法。值  $V_t^r$  代表剩余摄像机的潜在表示。通过结合  $V_t^r$  和  $\omega$ ，我们可以表示在评估动作摄像机影响时重要的剩余摄像机。

$$
u_t = \omega V_t^r .
$$

动作摄像机的表示  $z_t^a$  和相关剩余摄像机的表示  $u_t$  被连接起来。生成的表示将通过几个全连接层和激活层  $fr(\cdot)$  传递，以评估在状态  $s_t$  下采取动作  $a_t$  的累积奖励。通过  $fr([z_t^a, u_t])$ ，整个关系深度Q网络通过最小化目标函数进行训练：

$$
L_{RL} = \| r_t + \gamma \max_{a_{t+1}} fr([z_{t+1}^a, u_{t+1}]) - fr([z_t^a, u_t]) \|^2_2,
$$

其中  $fr'(\cdot)$  是通过  $fr(\cdot)$  动量更新的，或者是每几个训练迭代更新一次的  $fr(\cdot)$  的副本，不同的  $a_{t+1}$  将产生不同的  $z_{t+1}^a, u_{t+1}$ 。

总之，关系深度Q网络在每个时间  $t$  接受状态  $s_t$ 、动作  $a_t$ 、每个摄像机下的原始帧  $\{ \hat{X}_ t^i \}$  和摄像机之间的空间距离  $\{ d_{i,k} \}$  作为输入，并挖掘动作摄像机和剩余摄像机之间的关系，以评估累积奖励  $fr([z_t^a, u_t])$ 。关系深度Q网络的评估与上述基本框架相同。

## C. 单摄像机质量评估

除了摄像机之间的关系外，每个单独摄像机的质量也是检测摄像机网络中有缺陷摄像机的关键线索。我们希望摄像机能够以适当的视角拍摄行人，并具有良好的照明条件。例如，如果一个摄像机大多数情况下拍摄行人的正面，它应该被赋予一个高的角度分数；反之，则应该被赋予一个低的角度分数。此外，如果一个摄像机视图下的图像亮度太亮或太暗，该摄像机应该被赋予一个低的照明分数。

然而，正确量化摄像机角度和其他因素的评分是成本高昂的，因为它需要手动定义的准则和繁琐的评估工作。此外，一个摄像机的质量取决于许多超出角度和照明条件的因素。

为此，我们提出利用预训练的特征提取器  $f_e(\cdot)$ ，它用于计算状态值  $(9)$ ，以及标记数据  $X$  来自动评估每个摄像机的视觉效果。直观上，如果一个摄像机下的图像质量高，并且清晰地揭示了身份区分信息，那么该摄像机下同类之间的变化将会很小，不同类之间的变化将会很大。我们量化第  $i$  个视图下同类变化  $S_{intra}^i$  如下：

$$
S_{intra}^i = \frac{1}{|PX_i|} \sum_{(x_{j}^i,x_{p}^i) \in PX_i} \| f_e(x_{j}^i) - f_e(x_{p}^i) \|_2^2
$$

其中  $\| \cdot \|_2^2$  表示  $L2$  范数的平方， $PX_i$  表示  $X_i$  中包含同一个人的图像对的集合：

$$
PX_i = \{ (x_{j}^i, x_{p}^i) | x_{j}^i \in X_i, x_{p}^i \in X_i, y_{j}^i = y_{p}^i \}
$$

类似地，第  $i$  个视图下不同类之间的变化  $S_{inter}^i$  被量化为：

$$
S_{inter}^i = \frac{1}{|NX_i|} \sum_{(x_{j}^i,x_{n}^i) \in NX_i} \| f_e(x_{j}^i) - f_e(x_{n}^i) \|_2^2
$$

其中  $NX_i$  表示  $X_i$  中包含不同人图像对的集合：

$$
NX_i = \{ (x_{j}^i, x_{n}^i) | x_{j}^i \in X_i, x_{n}^i \in X_i, y_{j}^i \neq y_{n}^i \}
$$

最后，我们通过综合考虑第  $i$  个视图下的同类和不同类变化来评估该摄像机下的图像质量  $S_i$ ：

$$
S_i = \frac{S_{inter}^i}{S_{intra}^i}
$$

一个更大的  $S_i$  意味着第  $i$  个摄像机视图下的图像质量更高。

我们进一步通过引入单摄像机质量评估来改进价值函数  $(9)$ ：

$$
v(s_t) = ORank-1(X_{q_{s_t}}, X_{g_{s_t}}, f_e(\cdot), cos(\cdot)) + \lambda Z(s_t)
$$

其中  $\lambda$  是一个权衡参数， $Z(s_t)$  表示状态  $s_t$  中激活摄像机的平均质量：

$$
Z(s_t) = \frac{1}{\sum_{i} s_{i_t} s_{i_t}} \sum_{i} s_{i_t} \cdot S_i
$$

记住  $s_{i_t}$  是  $s_t$  的第  $i$  个元素， $s_{i_t} = 1$  表示第  $i$  个摄像机在  $s_t$  中是激活的。通过改进后的价值函数  $(29)$ ，我们可以更有效地测量奖励，并引导关系深度Q网络学习更好的动作-价值函数  $(23)$ 。

# V. 多摄像机连续人员重识别（MC-CPD）数据集

由于人员重识别在近年来受到越来越多的关注，越来越多的数据集被引入，如VIPeR [12]、CUHK03 [13]、Market-1501 [14]、DukeMTMC4ReID [15]和MSMT17 [16]。大多数现有的数据集包含少于10个摄像机，因此这些数据集的摄像机选择问题相对容易。Shinpuhkan2014 [64]和MSMT17 [16]是目前拥有最大数量摄像机的公共数据集。然而，Shinpuhkan2014中的行人数量太少（仅有24个身份）。而且，包括MSMT17在内的许多数据集缺乏摄像机部署的拓扑信息，这使得研究摄像机之间的空间拓扑关系变得不可能。

为了促进摄像机选择的调查，我们提出了一个名为MC-CPD的新数据集。我们在校园内外的室内外部署了一个由20个摄像机组成的监控网络，包括十一个1920×1080 FHD摄像机和九个1280×720 HD摄像机。我们数据集中可能的摄像机方案数为 \(2^{20}\)。MC-CPD数据集包含461个身份，训练集包含209个穿着夏装的人。值得注意的是，测试集包含252个人，其中42个人穿着冬装。数据集是在设计有中庭的建筑中收集的，因此由于一些摄像机受到阳光的严重影响，一些摄像机部署在昏暗的角落，数据集包含剧烈的照明变化。摄像机5（简称为C5）监控中庭的地面。C4监控中庭的入口，部分受到中庭阳光的影响。此外，如图10所示，部署在走廊中的摄像机，如C2、C7和C8，也受到阳光的影响。值得注意的是，由于摄像机位于建筑物中，计算摄像机之间的拓扑关系时考虑了相邻楼层和相邻位置的拓扑信息。更多细节，包括计算拓扑关系的方法，显示在附录中，可在线获取。

在形成数据集时，采用了最先进的人员检测器[68]进行行人检测。身份的标签是手工标注的。我们数据集与其他数据集的比较显示在表I中，图4显示了我们数据集的统计信息。摄像机的拓扑部署如图5所示。

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/a1efc1cb64494c9fb7c6feb337928dd4.png#pic_center" width="70%" />
</div>

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/c8d1fb488e094b988177c854a98b2d83.png#pic_center" width="70%" />
</div>

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/9dcdc926563342adb380b987b20aaaea.png#pic_center" width="70%" />
</div>

总之，我们的数据集与以前的公共数据集相比具有以下优势：

1. 据我们所知，我们数据集中的摄像机数量是现有公共标记人员重识别数据集中最大的。此外，不同的摄像机在角度、图像质量和光照强度上有所不同，这使得数据集适合调查摄像机对连续人员重识别的影响。摄像机网络的拓扑信息也是可用的。
2. 我们的数据显示对隐私风险是免费的，因为所有的参与者都签署了隐私协议。我们将在不久的将来发布这个数据集。请注意，所有志愿者都同意捕获的数据只能用于科学研究。

# VI. 实验

## A. 实验设置

**数据集和模型**：我们在以下四个数据集上应用了先进的重识别模型进行连续重识别任务，以调查传统重识别和连续重识别之间的差异：Market1501 [14]、MSMT17 [16]、SAIVT-Softbio（简称SAIVT）[67]和MC-CPD。这些模型包括广泛使用的骨干网络ResNet50 [36]、经典的基于部分的模型PCB [37]、强大的基于注意力的模型ABD-Net [11]、群体一致性学习模型CRF [50]和先进的基于变换器的模型TransReID [39]。

为了验证所提出方法的有效性以及有缺陷摄像机对连续重识别的负面影响，我们在MC-CPD和SAIVT数据集上应用了关系深度Q网络来检测有缺陷的摄像机，因为这些数据集拥有相当数量的摄像机并提供了足够的拓扑信息（用于模拟空间关系）和原始帧（用于模拟视觉关系）。在移除所检测到的有缺陷摄像机后，我们得到了修改后的数据集，分别称为MC-CPD(S)和SAIVT(S)。然后，我们再次评估了上述先进的重识别模型在这些修改后的数据集上的性能。如果所提方法能有效地检测到有缺陷的摄像机，那么这些先进的重识别模型预期将在修改后的数据集上实现令人印象深刻的连续重识别性能。由于随机减少数据集中的摄像机数量也可能提高重识别性能，我们在第六节中将我们的方法与随机选择方法和其他相关工作进行了比较。

值得注意的是，修改后数据集的训练集与原始数据集相同，只有测试集不同。我们保持训练集不变的原因是：1）为了显示有缺陷摄像机的负面影响，我们应该在摄像机选择前后保持用于评估的重识别模型不变；2）在所选摄像机下重新训练重识别模型并不会改变我们的主要结论，即有缺陷的摄像机严重妨碍了连续重识别，同时它还带来了额外的计算成本。我们还训练了在所选摄像机下的数据上的重识别模型进行评估，并发现大多数重识别模型的性能可以略微提高。有关详细信息，请参阅在线附录。

**评估指标**：对于传统重识别，我们采用Rank-1、Rank-5和Rank-10准确性作为评估指标。对于连续重识别，我们采用所提出的整体Rank-1、整体Rank-5以及整体Rank-10准确性作为评估指标。

## B. 实施细节

我们在 SAIVT 和 MC-CPD 数据集上执行了有缺陷摄像机的检测实验。对于摄像机代理的训练，我们在一个 Market-1501 数据集上训练了一个 PCB [37] 模型，使用 ResNet-50 [36] 作为骨干网络，并将其用作特征提取器  $f_e(\cdot)$  来计算状态值  $(29)$ 。默认情况下，在训练阶段，我们使用标准的 SGD 优化器，并将学习率、动量和权重衰减设置为 0.00025、0.9 和 0.0005，分别。小批量的大小设置为 32（每个训练迭代包含 32 个集数），并且我们的关系深度 Q-网络的总体训练过程包括 128 个周期。累积奖励折扣  $\gamma$  设置为 0.5。视觉特征向量在  $(14)$  中的特征维度  $\hat{d}_ v$  是 2048，而在  $(21)$  中的  $\hat{d}_ q$  是 512。在图 3 中的  $fr(\cdot)$  由三个全连接层组成，每个全连接层后面都跟着一个 ReLU 激活层，除了最后一个。单摄像机质量评估的权重  $\lambda$  设置为 2.0。 $\epsilon$ -贪婪策略中的参数  $\epsilon$  最初设置为 0.8，并且每 32 个周期将其当前值减少到  $0.9$  倍。摄像机之间的空间距离  $d_ {i,k}$  的计算细节在附录中提供，代码将公开提供。

## C. 研究连续重识别的紧迫性

表 II 显示了先进重识别模型在传统重识别和连续重识别上的性能。实验结果展示了连续重识别的难度。由于先进重识别模型在连续重识别上无法达到令人满意的性能，可以得出结论，针对优化重识别的成对相似性的研究工作并不等同于有效提高检索结果的一致性。例如，尽管 PCB 在 MC-CPD 数据集上达到了 81.3% 的 Rank-1 准确率，但它只在整体 Rank-1 准确率上达到了 36.3%，这对于连续重识别来说是不满意的。因此，值得将更多的研究工作投入到连续重识别中。

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/6bebf5e868e04ce6a99a1c0290ef794d.png#pic_center" width="70%" />
</div>

## D. 有缺陷摄像机对连续重识别的负面影响

为了展示摄像机部署的影响，我们在通过我们的方法检测并排除了原始数据集中的有缺陷摄像机后，对 MC-CPD(S) 数据集和 SAIVTSoftbio(S) 数据集进行了进一步评估。每个修改后数据集的测试集包含原始数据集中一半数量的摄像机。从表 II 中可以看出，在 MC-CPD(S) 数据集上，PCB 模型的整体 Rank-1 准确率显著提高到了 82.5%，比原始数据集上的原始结果高出 46.2%。在移除检测到的有缺陷摄像机后，连续重识别性能的变化表明这些摄像机严重妨碍了连续重识别，而我们的方法能够有效地检测到这些摄像机。

此外，一个有趣的现象是，在 SAIVT 数据集上排除有缺陷摄像机后，Re-ID 模型的 Rank-1 准确率下降了，甚至低于整体 Rank-1 准确率。注意到我们的方法是为了提高摄像机网络的一致性，它并不保证提高成对匹配的准确性。这种现象的主要原因如下：i) 存在一对摄像机，它们之间的匹配准确度很高，而它们与其他摄像机的匹配准确度很低。因此，这样的一对摄像机对于 Rank-1 准确率是有益的，但对整体 Rank-1 准确率是有害的。当这样的一对摄像机被移除时，整体 Rank-1 准确率增加，而 Rank-1 准确率下降。ii) Rank-1 准确率有时低于整体 Rank-1 准确率的现象，是由于两种评估指标中不同的检索方法造成的。在计算传统的 Rank-1 准确率时，所有摄像机的图像同时排名，与查询摄像机相同的摄像机下的图像（C1中的图像）将影响排名结果。然而，在计算整体 Rank-1 准确率时，不同摄像机下的图像是单独排名的，因此排名结果不会受到跨视角的硬负样本的影响，这使得整体 Rank-1 准确率可能高于 Rank-1 准确率（见第 III 节的公式）。图 6 展示了一个示例。

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/1c83ca71b01d4a57a4d84cf88bb5b3f8.png#pic_center" width="70%" />
</div>

我们强调，在实验中移除有缺陷的摄像机是为了展示这些摄像机的负面影响。然而，我们的目标并不是建议人们简单地移除这些摄像机。我们的目的是利用连续重识别来检测有缺陷的摄像机，而不是简单地报告改进。我们的方法为人们提供了关于当前摄像机网络中潜在风险的提示：除非取得重大进展，否则在这些有缺陷的摄像机中进行重新识别对于当前的重识别模型来说是很难的，这些摄像机的智能监控是薄弱的（需要改进）。因此，直到更强大重识别方法出现之前，从摄像机部署的角度改进重识别系统是实际可行的。人们可以通过改善照明或调整被检测为有缺陷的摄像机的角度或采取其他措施来增强视觉监控。我们在下面报告了更多的实验结果。

## E. 与相关方法的比较

为了验证我们的改进并非微不足道，我们将我们的方法与其他相关方法进行了比较，包括 i) 关注学习图像之间一致性关系的方法，如 CADL [42] 和 CRF [50]，这些方法旨在通过指导模型学习不同摄像机视图图像之间的一致性关系来改进监控系统；ii) 摄像机选择方法，如随机选择方法和他启发式搜索方法，后者将在后文详细说明。

为了评估每种方法在数据集上的有效性，我们使用数据集的训练集训练了一个重识别模型。然后，我们应用这些方法，例如通过添加一致性损失 [42] 对 Re-ID 模型进行微调，或者在排除了这些方法检测到的有缺陷摄像机后评估 Re-ID 模型。为了不失一般性，我们在以下实验中使用经典的 Re-ID 模型 PCB [37]。请注意，使用其他 Re-ID 模型也可以得到相同的结论，因为检测有缺陷的摄像机对各种 Re-ID 模型都是有益的，如表 II 所示。

总之，表 III、IV 和图 7 中的实验结果展示了关注图像之间一致性关系的方法的局限性。这是因为它们忽略了有缺陷摄像机带来的负面影响。此外，实验结果表明，我们的方法在很大程度上优于其他摄像机选择方法。

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/ee7b9e41f94e4ed78b61d5a804048c07.png#pic_center" width="70%" />
</div>

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/d5d49a25335a4398ae2ea33453dd7f46.png#pic_center" width="70%" />
</div>

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/551d659fdbc4454ea504ebfd5369ce43.png#pic_center" width="70%" />
</div>

与关注图像之间一致性关系的方法的比较：CADL 关注不同视图下的检索结果之间的一致性，优化模型以识别尽可能多的摄像机下的人；CRF 在学习过程中对图像施加群体一致性约束。从表 II、III 和 IV 中，我们可以观察到这些方法在连续 Re-ID 上表现不佳，并且不如摄像机选择方法。例如，CRF 在 MC-CPD 数据集上仅达到了 45.7% 的整体 Rank-1 准确率，比我们的方法低 36.8%。一个关键原因是，它们的一个动机是传统的成对验证损失在学习数据时是弱的，因为它只学习一对样本之间的关系，而考虑一组样本之间关系的客观函数可以提高模型性能。然而，现有的目标函数，如交叉熵损失，优于验证损失，它不仅仅从成对图像中学习，而是通过强制图像与其他人区分开来学习区分特征。此外，得益于强大的骨干网络，如 ResNet-50 [36]，Re-ID 模型可以很容易地拟合数据集，并在每个视图下完美地识别训练集中的人。因此，关注图像之间一致性关系的方法即使对于通过交叉熵损失优化的简单 ResNet-50 来说也无法带来显著改进，它们受到探测器摄像机的影响。相比之下，我们提出的方法不关注图像之间的关系，而是模拟摄像机之间的关系来检测有缺陷的摄像机。

总体而言，实验结果表明，对骨干网络和目标函数的研究努力很难缓解有缺陷摄像机带来的不一致性，自动检测这些摄像机以促进手动调整是有价值的。

与随机选择方法的比较：为了展示随机选择的性能，我们从数据集中随机移除一半的摄像机，并计算整体 Rank-1 准确率。重复随机移除过程 1,000 次，我们报告平均整体 Rank-1 准确率作为表 III 和 IV 中随机选择的性能。实验结果表明，我们提出的关系深度 Q 网络在整体 Rank-1 准确率上显著优于随机选择方法 30.7%/16.2% 在 MC-CPD/SAIVT 数据集上。比较表明，我们的方法在检测摄像机网络中有缺陷的摄像机方面是有效的。

与启发式搜索方法的比较：为了检测阻碍连续 Re-ID 的有缺陷摄像机，一种启发式解决方案是对每个摄像机估计整体 Rank-1，并将相对整体 Rank-1 准确率较低的摄像机视为有缺陷的摄像机。具体来说，对于每个摄像机，我们使用该视图下的图像作为查询，并使用其他摄像机视图下的图像作为库。然后，我们使用预训练的 Re-ID 模型  $f_e(\cdot)$ （ $f_e(\cdot)$  也用于在 (29) 中计算学习摄像机代理的状态值）进行连续 Re-ID，并计算整体 Rank-1 准确率作为该视图的分数。在 MC-CPD 上得分最低的 10 个摄像机和在 SAIVT 数据集上得分最低的 4 个摄像机被视为有缺陷的摄像机。我们评估了排除这些摄像机后的连续 Re-ID，以展示启发式搜索方法的有效性。

表 III 和 IV 中的实验结果表明，我们的方法在整体 Rank-1 准确率上比启发式搜索方法高出 26.5%/19.5% 在 MCCPD/SAIVT 数据集上。性能差距的主要原因是多个摄像机的全局关系对于检测有缺陷的摄像机至关重要，因此仅考虑单个摄像机视图的特征会导致次优解决方案。

为了更好地说明所提出方法的优越性，我们进一步报告了使用不同方法在 MC-CPD 数据集上检测不同数量的有缺陷摄像机的性能。图 7 中的实验结果表明，我们的方法在不同条件下一致优于其他方法。此外，我们观察到使用我们的方法在 MC-CPD 数据集上剩余摄像机数量为 9 时，Re-ID 性能变得稳定。这主要是因为我们的方法已经检测并排除了适当数量的有缺陷摄像机。在实践中，我们可以通过评估摄像机代理来获得性能曲线，如图 7 所示，用于检测有缺陷的摄像机。可以合理推测，一旦智能监控摆脱了有缺陷的摄像机，连续 Re-ID 性能应该变得稳定，并且不易受到摄像机网络变化的影响。因此，根据曲线，我们可以根据连续 Re-ID 性能变得稳定的点推断出有缺陷摄像机的数量。

## F. 消融研究

在本节中，我们在 MC-CPD 数据集和 SAIVT-Softbio 数据集上进行了广泛的实验，以展示我们方法中每个组件的有效性。

**单摄像机质量评估的不可或缺性**：从表 V 和 VI 中的实验结果可以看出，如果没有单摄像机质量评估，整体 Rank-1 准确率在 MC-CPD/SAIVT 数据集上分别下降了 5.1%/8.6%，表明我们方法在检测有缺陷摄像机方面的有效性在移除单摄像机质量评估后有所降低。我们还评估了不同权衡参数下的我们的方法，并在图 8 中展示了实验结果。我们可以观察到，设置 0 < λ < 4 对两个数据集上的连续 Re-ID 性能都有益，表明我们的方法可以在适当的 λ 下更好地检测有缺陷的摄像机。然而，λ 不应该设置为一个很大的值，否则它将主导奖励计算，并阻碍关系深度 Q 网络学习摄像机之间的全局关系。

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/c92696639a5646578459e0d641bcee15.png#pic_center" width="70%" />
</div>

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/4fcdef14439946e0893e50563691a987.png#pic_center" width="70%" />
</div>

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/a604ce53c97149bf93c565dd8df15ce8.png#pic_center" width="70%" />
</div>


**关系建模的有效性**：没有视觉关系和空间关系，MC-CPD 数据集上的整体 Rank-1 准确率分别下降了 14.6% 和 24.6%，如表 V 所示。性能下降验证了摄像机之间的关系对检测有缺陷的摄像机是有益的。此外，我们可以观察到空间关系比视觉关系更重要，因为移除空间关系会导致更大的性能下降。这可能是因为单摄像机质量评估在一定程度上可以弥补视觉信息的缺失，而空间关系可以鼓励模型捕捉摄像机网络中的全局关系。

从表 VI 中的实验结果来看，我们发现关系建模在我们方法的 SAIVT 数据集上没有带来改进。主要原因是 SAIVT 的摄像机数量少于 MC-CPD，并且从 8 个摄像机中检测 4 个有缺陷的摄像机只有少数几种可能的选择。在这种情况下，多个摄像机的全局关系并不重要，每个摄像机的质量评估在 SAIVT 数据集上更有价值。

**每个组件在检测不同数量的有缺陷摄像机上的有效性**：由于上述比较是通过从数据集中选择一半的摄像机来进行的，这等同于在 SAVIT/MC-CPD 数据集上检测了最多的 4/10 个有缺陷的摄像机，我们在图 9 中进一步展示了在 MC-CPD 数据集上检测不同数量的有缺陷摄像机的实验结果，以进一步证明我们模型中每个组件的有效性。

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/c59bddc4b42648ceb2f5dd23dabdd9b4.png#pic_center" width="70%" />
</div>

从图 9 中，我们可以观察到每个组件对检测有缺陷的摄像机都是至关重要的。首先，当移除最 10 个有缺陷的摄像机时，连续 Re-ID 性能的提高变得稳定，这表明适当数量的有缺陷摄像机已经被检测和移除。我们可以观察到，完整模型的性能提高比其它模型更快地变得稳定。此外，在大多数情况下，移除完整模型检测到的有缺陷的摄像机将导致最佳的连续 Re-ID 性能。这些实验结果表明，完整模型在检测有缺陷的摄像机方面更有效。

请注意，某些情况下，完整模型并不总是实现最佳性能。例如，在 MC-CPD 数据集中检测最多的 5 个有缺陷的摄像机时（在这种情况下剩余摄像机的数量是 15），移除标记为“w/o Visual R.”的模型检测到的有缺陷的摄像机可以导致连续 Re-ID 的最大改进。然而，这并不意味着完整模型是劣等的。完整模型性能下降的原因是数据集中存在超过 5 个有缺陷的摄像机。当摄像机网络中仍存在相当数量的有缺陷的摄像机时，连续 Re-ID 的性能无法完全提高是合理的。

## G. 可视化

图 10 展示了我们的关系深度 Q 网络检测到的有缺陷摄像机。第一行显示了来自部署良好的摄像机的图像，第二行显示了来自有缺陷摄像机的图像。从图 10 中，我们可以观察到明显的差异，第一行中的图像在照明、角度等方面具有一致的视觉特征，而第二行中的摄像机的视觉特征变化很大（例如，C4 太亮，C9 太暗），这表明我们的摄像机选择不仅取决于环境因素，还取决于摄像机之间的一致性。实验结果表明，我们应该调整有缺陷摄像机的部署。

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/faa079f2a4b74c14835fb4e63ceaf6cc.png#pic_center" width="70%" />
</div>

# VII. 扩展到摄像机内监督摄像机选择

给定一个状态  $s_t$ ，我们通过在标记数据集上执行连续重识别来计算状态的价值  $(29)$  并评估单摄像机评估  $(28)$ 。然而，为多摄像机图像注释身份标签的成本非常高，尤其是当摄像机数量很大时。在本节中，我们描述了一种无需跨视图注释的方法，利用每个摄像机下的轨迹来学习有效的摄像机选择代理。

直观上，按照之前的工作 [69]、[70]，可以很容易地知道轨迹的摄像机标签，因此我们制定了一个摄像机内监督摄像机选择。在这个公式中，每个轨迹由来自同一个人的图像组成，否则轨迹可以通过预处理进一步分离。具体来说，我们将同一轨迹中的图像分配相同的内视图标签，并引导模型区分轨迹以获得区分性的特征表示。注意，内视图标签并不代表真实的标识身份。为了补偿缺少跨视图身份注释，我们将图像投影到不同摄像机视图的特征空间中，并基于最近的跨视图特征生成技术 [71]、[72] 为每个图像生成跨视图特征。由于单摄像机质量评估不需要跨视图身份标签，因此摄像机内监督摄像机代理学习的关键是在没有身份标签的情况下获得状态的价值，而不是使用需要身份标签的  $(9)$  中的函数。

具体来说，表示训练集  $X = \bigcup_{i} X_i$ ， $X_i = \{ x_{i,j}, y_{i,j} \}^{N_{X_i}}_{j=1}$ 。为了生成跨视图特征，我们为每个摄像机学习一个特定于视图的批量归一化层  $\psi_i(\cdot)$  和一个分类器  $\phi_i(\cdot)$ ，以及与视图共享的骨干  $f(\cdot)$ ，通过引导它们区分同一视图下的轨迹，定义目标函数为：

$$
L_u = \sum_{i} \sum_{j} \text{CE}(\phi_i(\psi_i(f(x_{i,j}))), y_{i,j})
$$

其中 CE 表示 softmax 交叉熵损失， $\psi_i(\cdot)$  和  $\phi_i(\cdot)$  是为第  $i$  个摄像机视图学习的。优化  $L_u$  到收敛后，特定于视图的批量归一化层  $\psi_i(\cdot)$  可以捕获该摄像机视图下的特征分布 [40]、[72]。不失一般性，我们选择第一个视图下的图像  $X_1$ ，并利用  $\psi_i(\cdot)$  为它们生成不同视图下的特征。形式上， $X_1$  在第  $i$  个摄像机视图下的特征是：

$$
F_{1i} = \{ \psi_i(f(x_{1,j})) | x_{1,j} \in X_1 \}
$$

通过利用  $F_{1i}$  和轨迹标签，我们可以定义状态价值函数：

$$
\bar{v}(s_t) = \min_{i,k} \frac{1}{N_{X_1}} \sum_{j} \min_{n} \max_{p} \frac{\| \psi_i(f(x_{1,j})) - \psi_k(f(x_{1,n})) \|_ 2^2}{\| \psi_i(f(x_{1,j})) - \psi_k(f(x_{1,p})) \|_2^2} + \lambda Z(s_t)
$$

其中  $i$  号摄像机和  $k$  号摄像机在状态  $s_t$  中被选择， $\| \cdot \|_2^2$  是  $L2$  范数的平方， $Z(s_t)$  是状态  $s_t$  中激活摄像机的平均质量分数，如  $(30)$  中计算的。

我们没有直接在集合  $\{ F_{1i} \}^{M}_{i=1}$  上执行连续 Re-ID 来计算状态价值，因为特征是生成的，这使得它很容易从其他视图中检索到正样本。相反，受到连续 Re-ID 的启发，我们使用最困难特征之间的距离来定义状态价值，因为最困难的正样本（负样本）对之间的距离越小（大），就越有可能获得一致的结果。

在 MC-CPD 数据集上应用摄像机内监督摄像机选择的实验结果如表 VII 所示。我们观察到我们的方法仍然显著优于随机选择方法，并且不如监督对应物。由于表 V 中提到的启发式方法需要跨视图注释，因此在摄像机内监督设置下我们无法将其与我们的方法进行比较。

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/09a00d9d27334551a7e65f35f9947f01.png#pic_center" width="70%" />
</div>

预训练有利于摄像机内监督摄像机选择：我们还在 SAIVT-Softbio 数据集上进行了实验，实验结果如表 VIII 所示。我们可以观察到，我们的方法在摄像机内监督下达到了 72.6% 的整体 Rank-1 准确率，比随机选择高出 7.6%。实验结果证明了摄像机内监督摄像机选择的有效性。此外，我们可以观察到在 MC-CPD 数据集上的预训练提高了 SAIVTSoftbio 数据集上的性能 9.8% 的整体 Rank-1 准确率（82.4% 对比 72.6%），表明我们的模型可以从源领域学习的知识在应用于新领域时发挥作用。通过在 MC-CPD 数据集上进行预训练和在 SAIVT-Softbio 数据集上进行摄像机内监督微调，我们的模型甚至在整体 Rank-1 准确率方面略微超过了监督对应物（82.4% 对比 81.2%）。

<div align=center>
  <img src="https://img-blog.csdnimg.cn/direct/b34ed9893ff5424082e7c2dd43484a8b.png#pic_center" width="70%" />
</div>

# VIII. 结论

在这项工作中，我们首先调查了具有挑战性的连续重识别问题，并提出了相应的度量标准。我们发现连续重识别的性能将受到摄像机部署的强烈影响，因为连续重识别要求跨摄像机视图的检索结果一致。因此，我们从摄像机部署的角度重新思考人物重识别问题，这对于跟踪多个摄像机中的行人至关重要。基于连续重识别，我们开发了一种关系深度Q网络，用于自动和有效地检测有缺陷的摄像机，以便人们可以相应地改进摄像机网络。此外，除了 (4) 和 (5) 中提到的，用户在应用我们的方法时可以引入特定于用户的约束。为了减少对身份标签的依赖，我们还扩展了我们的方法到摄像机内监督条件，使其在身份标签不可用时成为可能。为了促进摄像机选择的调查，我们收集了一个新的基准数据集 MC-CPD，该数据集包含隐私协议，包括更多的摄像机，并提供了全面的拓扑信息。实验结果表明，我们的方法在监督设置和摄像机内监督设置中都有效，从源领域学习到的摄像机代理可以有效地适应到新领域，通过摄像机内监督摄像机选择。然而，如何使用目标摄像机网络中的未标记数据改进预训练摄像机代理仍然是一个未探索的问题，这可能在未来的工作中得到解决。

此外，当前的 MC-CPD 数据集包含的摄像机数量有限，可以通过在更多场景中部署更多摄像机来改进。拥有更多摄像机的数据集将促进连续重识别的调查，并有利于开发用于终身重识别系统的增量摄像机中连续学习重识别模型。