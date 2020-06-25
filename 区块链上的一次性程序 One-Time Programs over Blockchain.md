我们提出了自己一种适用于任何基于PoS共识协议的区块链可用的，基于乱码电路（garbled circuits）和可提取见证加密（extractable witness encryption）的一次性编译器。

**概述。**  假设区块链协议具有$(\alpha,\beta,l_1,l_2)$的可区分分叉性（distinguishable forking property）。我们知道，可区分分叉性表明，没有任何PPT（probabilistic polynomial time，概率多项式时间）敌人能够生成长度$\geq l_1+l_2$的分叉，使得分叉的第一个$l_1$区块之后的股权证明部分大于$\alpha$。此外，它还意味着诚实一方的区块链中任意$l_2$个连续区块中的风险证明分数将至少为$\beta$，其中$\beta$不可忽略地高于$\alpha$。

>An algorithm $A$ is said to run in **polynomial time** if there exists a polynomial $p(\cdot)$ such that for every input $x \in \{0,1\}^*$, the computation of $A(x)$ terminates within at most $p(|x|)$ steps.
>
>A **probabilistic algorithm** is one that has the capability of "tossing coins", i.e. the algorithm has access to a random source of randomness that yields unbiased random bits that are independently equal to 1 with $1/2$ probability and to 0 with $1/2$ probability.

在更高的层面上，该计划的工作方式如下。为了在区块链**B**上编译电路*C*，编译算法首先将电路混淆（garble）以计算被篡改的电路（garbled circuit）和线键（wire keys）。假设我们用公钥加密的方式加密线值，并且将相应的一次性程序设置为乱码电路和加密了的线值。这表明评估者（evaluator）必须与编译部分交互才能评估程序。但是，一次性程序不是在交互设置中定义的。因此，我们需要以某种方式允许有条件地释放/有条件解密加密的线值以进行评估。此外，我们需要确保评估者只学习与*一个*输入相对应的线键，否则它将不满足一次性保密条件。为此，我们使用见证加密（witness encryption）方案来加密线值，因此，为了解密线值，评估者需要产生区块链**B‘** 作为见证，**B’** 必须满足以下条件——1）**B’** 中存在包含输入的区块(评估者要在其上评估电路)；2）在输入区块之后至少还有$l_1+l_2$个区块，使得**B‘**的最后$l_2$块中的股权证明分数大于$\beta$；3）不存在发布不同输入的任何其他区块。

要评估这样一个编译的程序，评估者需要将其输入发布到区块链上，然后等待它被添加到区块链中，并通过$l_1+l_2$个区块进行扩展。然后，它可以简单地使用其区块链作为见证来解密适当的线值，然后使用这些值评估混淆后的电路。 直观地说，这将满足一次性保密属性，因为为了在第二个输入上评估程序，对手需要在输入区块之前分叉区块链。现在，由于可辨别的分叉属性保证没有**PPT**对手能够以不可忽略的概率生成这样的分叉(长度大于$l_1+l_2$)，因此随之而来的是一次性保密（生效）。