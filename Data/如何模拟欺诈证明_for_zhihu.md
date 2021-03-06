<div align='center' ><font size='70'>如何模拟欺诈证明</font></div>

本文的目标是通过模拟系统来验证它们有关数据可用性部分的结果。[论文](https://eprint.iacr.org/2018/046.pdf)使用了一些知名的组合公式来证明网络在各种设置中的安全性。由于模型是概率的，我们可以编写一个程序来模拟它。在运行仿真程序足够多次并得到平均结果后，我们希望得到相同的结果。

在讨论仿真之前，让我们先对模拟论文的相关部分进行最少的介绍。

***

简而言之，欺诈证明是一种工具，它让轻节点收到某些区块无效的证据。诚实的完整节点打包了挑战轻节点以检查并确信区块出现问题所需的最少信息量。以可扩展的方式执行此操作涉及在模块中的证明简洁度和实际信息密度之间进行权衡。

但是，要进行欺诈证明，获得区块数据是一项基本要求。如果我们没有数据来证明状态转换（state trasition）是错误的，我们怎么能拥有所有必要的信息来说服另一个节点？

论文用两个基本思路解决了数据可用性问题：

- 二维Reed-Solomon作为擦除代码（erasure codes）
- 轻节点对区块共享（block share）进行随机采样

论文将这两种观点与闭合公式相结合，以计算网络某些参数集的数据可用性概率：

- _k_ ， 区块内 _共享_ 的数量。
- _s_ ， 能够使每个轻节点被满足地试图从完整节点中提取的共享数。（the number of shares each light-nodes try to pull from a full-node to be satisfied.）
- _p_ ， 轻节点通过共享采样能够重建区块的可能性。（the probability of block reconstruction made possible by light-nodes share sampling.）
- _c_ ， 满足为k和s的网络中p所需的轻节点节点数。（the number of light-nodes necessary to satisfy p in a network that is configured for k and s.）

请记住这些参数，因为它们用于本文的其余部分。

# 随机采样方案 Random-sampling scheme

下面是主要的想法，在此可以理解我们正在努力做什么。

给定一个新块，网络中诚实的节点最感兴趣的是保证块数据可用。这并不一定意味着强制每个诚实的节点拥有所有共享，但只知道作为一个团队他们可以重建它。

首先，轻节点从新区块中拉出随机的一组s共享。轻节点在要求什么共享方面没有协调，所以很多轻节点拉相同共享的可能性是合理的。作为一个类比，你可以想到得到一个拼图。__轻__ 节点不是试图拉整个拼图，只是一些随机的碎片。__完整__ 节点才对完整的拼图感兴趣。

最后，轻节点通过 _gossiping_ 与它们连接到的其他完整节点进行交互。首先，轻节点会通知它连接到的完整节点有关提取的共享。保持轻节点和完整节点之间以及完整节点和其他完整节点之间运行，允许一组诚实的完整节点来重建整个拼图（区块数据）。

执行随机采样有许多优点：

- 轻节点只需要较小的带宽使用，因为它们提取了一小部分共享。
- 轻节点只需要较小的存储空间。
- 完整节点利用轻节点实现具有所有区块数据的共同目标。(Full-nodes leverage light-nodes towards the mutual goal of having all the block data.)
- 区块重建是分布式的.
- 它利用到了可用的轻型客户端的数量。轻节点客户端越多，性能越好。

在区块数据中使用擦除代码使得每个被扩展的共享在用于区块重建时的单独的重要性降低,这是对随机抽样和gossiping目标的重要贡献.

在全图中，轻节点需要额外检查。(In the full picture the light-nodes an extra check. )每个共享都附带Merkle证明,以证明当前共享来自区块数据. 由于论文讨论的是2维编码, 因此此证明可以从编码的行或列维度进行。如果我们使用 N 维擦除代码，则它可以证明共享所存在(lives)的 N 种可能Merkle树。

***

# 模拟作为验证 Simulation as a verification

通常，[蒙特卡洛法](https://baike.baidu.com/item/%E8%92%99%E7%89%B9%C2%B7%E5%8D%A1%E7%BD%97%E6%96%B9%E6%B3%95/8664362?fromtitle=%E8%92%99%E7%89%B9%E5%8D%A1%E6%B4%9B%E6%B3%95&fromid=2056487&fr=aladdin)是在复杂系统中进行计算的方法，在复杂系统中，正式的证明很难，或者完整计算难以实现。其应用是如此广泛，我想这个词经常被滥用。

最基本的思想是使用随机抽样法来计算模型所需输出的 _经验均值_，目标是希望接近实际平均值。

通过模拟概率系统，我们可以应用同样的想法来发现结果。我们运行许多次模拟，并计算 _经验均值_，应匹配期望的实际平均值。作为验证工具，我们可以将仿真结果与以另一种方式获得的结果进行比较，例如，正式的证明。

然而，模拟的办法，事实上，更强大。我们可以开始使用这个系统，调整参数或引入新的想法，并快速看到它的反应。使用仿真的易用程度取决于资源密集型如何运行仿真实例，以及计算 _可靠的_ 经验均值所需的最小迭代量。

# 欺诈证明网络模拟 Fraud-proof network simulation

论文的作者使用闭合数学公式分析解决方案的多种属性. 由于轻节点对区块的共享进行随机采样，大多数各种数学都围绕着计算其概率。

为了检查这些闭合公式，我们可以制作一个程序来模拟轻节点和完整节点行为，让他们进行交互，看看会发生什么。如果我们多次运行此模拟，我们可以对有趣的指标进行统计分析，并查看它们与论文中的封闭公式是否匹配。

在每个模拟实例中启动的完整节点提供了由 <img src="https://www.zhihu.com/equation?tex=4k^2" alt="4k^2" class="ee_img tr_noresize" eeimg="1"> 个共享组成的新区块。数量为 <img src="https://www.zhihu.com/equation?tex=c" alt="c" class="ee_img tr_noresize" eeimg="1"> 的 _轻节点_ 试图从全节点中提取不同的随机共享。完整节点接受向轻节点发送共享直到它发送的不同共享达到 <img src="https://www.zhihu.com/equation?tex=4k^2-(k+1)^2" alt="4k^2-(k+1)^2" class="ee_img tr_noresize" eeimg="1"> 个;这是对轻节点最坏的情景.

__当模拟迭代次数达到完整节点决定拒绝响应发送共享请求的次数时，则迭代被视为 *成功（这意味着已公开完整节点）* 。通过运行多个迭代，我们可以通过计算根据这一设定成功迭代的比例来估计 <img src="https://www.zhihu.com/equation?tex=p" alt="p" class="ee_img tr_noresize" eeimg="1"> 。__

模拟器是用Go写的程序并且[开源](https://github.com/jsign/fraudproofsim)，任何人都可以看到细节或改进它。它有一个CLI，有三个命令：

- *verifypaper*：验证论文表1的结果。
- *solve*： 求解c用于k、s和p的特定设置。
- *compare*：比较论文中提出的标准与增强模型。

在没有命令的情况下运行程序提供了一些有关如何使用 CLI 界面的有用信息：

```shell
$ git clone https://github.com/jsign/fraudproofsim.git
$ cd fraudproofsim && go get ./...
$ go run main.go
It permits to compare, solve and verify fraud-proof networks.
Usage:
  fraudproofsim [command]
Available Commands:
  compare     Compares the Standard and Enhanced models
  help        Help about any command
  solve       Solves c for k, s and p
  verifypaper Verifies setups calculated in the paper
Flags:
      --enhanced   run an Enhanced Model
  -h, --help       help for fraudproofsim
      --n int      number of iterations to run per instance (default 500)
Use "fraudproofsim [command] --help" for more information about a command.
```

您可以看到上述三个命令，也可以看到两个通用标志：

- *enhanced*, 允许您选择在增强型模型上运行网络。默认值为标准模型.
- *n*, 是用于计算所需结果的设置中的模拟迭代次数。默认值为 500。


