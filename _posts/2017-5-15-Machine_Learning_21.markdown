---
layout: post
title:  机器学习（二十一）——EMD, LSA, HMM
category: theory 
---

# P-R、ROC和AUC

很多学习器是为测试样本产生一个实值或概率预测，然后将这个预测值与一个分类阈值（threshold）进行比较，若大于阈值则分为正类，否则为反类。这个实值或概率预测结果的好坏，直接决定了学习器的泛化能力。实际上，根据这个实值或概率预测结果，我们可将测试样本进行排序，“最可能”是正例的排在最前面，“最不可能”是正例的排在最后面。这样，分类过程就相当于在这个排序中以某个“截断点”（cut point）将样本分为两部分，前一部分判作正例，后一部分则判作反例。

在不同的应用任务中，我们可根据任务需求来采用不同的截断点，例如若我们更重视 “查准率”（precision），则可选择排序中靠前的位置进行截断，若更重视“查全率”（recall，也称召回率），则可选择靠后的位置进行截断。

对于二分类问题，可将样例根据其真实类别与学习器预测类别的组合划分为真正例（true positive）、假正例（false positive）、真反例（true negative）和假反例（true negative）。

查准率P和查全率R的定义如下：

$$P=\frac{TP}{TP+FP},R=\frac{TP}{TP+FN}$$

以P和R为坐标轴，所形成的曲线就是P-R曲线。

ROC（Receiver operating characteristic）曲线的纵轴是真正例率（True Positive Rate，TPR），横轴是假正例率（False Positive Rate，FPR）。其定义如下：

$$TPR=\frac{TP}{TP+FN},FPR=\frac{FP}{TN+FP}$$

ROC曲线下方的面积被称为AUC（Area Under ROC Curve）。

更多内容参见下图：

![](/images/article/sensitivity_and_specificity.png)

原图地址：

https://en.wikipedia.org/wiki/Sensitivity_and_specificity

# Earth mover's distance

推土机距离（EMD）是两个概率分布之间的距离度量的一种方式。如果将区间D的概率分布比作沙堆P，那么$$P_r$$和$$P_\theta$$之间的EMD距离，就是推土机将$$P_r$$改造为$$P_\theta$$所需要的工作量。

![](/images/article/earth_move.png)

EMD的计算公式为：

$$EMD(P_r,P_\theta) = \frac{\sum_{i=1}^m \sum_{j=1}^n f_{i,j}d_{i,j}}{\sum_{i=1}^m \sum_{j=1}^n f_{i,j}}$$

其中，f表示土方量，d表示运输距离。

EMD可以是多维分布之间的距离。一维的EMD也被称为Match distance。

EMD有时也称作Wasserstein距离。

在文本处理中，有一个和EMD类似的编辑距离（Edit distance），也叫做Levenshtein distance。它是指两个字串之间，由一个转成另一个所需的最少编辑操作次数。许可的编辑操作包括将一个字符替换成另一个字符，插入一个字符，删除一个字符。一般来说，编辑距离越小，两个串的相似度越大。

>注：严格来说，Edit distance是一系列字符串相似距离的统称。除了Levenshtein distance之外，还包括Hamming distance等。

>Vladimir Levenshtein，1935年生，俄罗斯数学家，毕业于莫斯科州立大学。2006年获得IEEE Richard W. Hamming Medal。

参考：

https://vincentherrmann.github.io/blog/wasserstein/

http://chaofan.io/archives/earth-movers-distance-%e6%8e%a8%e5%9c%9f%e6%9c%ba%e8%b7%9d%e7%a6%bb

# LSA

## 基本原理

Latent Semantic Analysis（隐式语义分析），也叫Latent Semantic Indexing。它是PCA算法在NLP领域的一个应用。

在TF-IDF模型中，所有词构成一个高维的语义空间，每个文档在这个空间中被映射为一个点，这种方法维数一般比较高而且每个词作为一维割裂了词与词之间的关系。

为了解决这个问题，我们要把词和文档同等对待，构造一个维数不高的语义空间，每个词和每个文档都是被映射到这个空间中的一个点。

LSA的思想就是说，我们考察的概率既包括文档的概率，也包括词的概率，以及他们的联合概率。

为了加入语义方面的信息，我们设计一个假想的隐含类包括在文档和词之间，具体思路是这样的：

1.选择一个文档的概率是$$p(d)$$

2.找到一个隐含类的概率是$$p(z\vert d)$$

3.生成一个词w的概率为$$p(w\vert z)$$

## 实现方法

![](/images/article/Topic_model_scheme.jpg)

上图中，行表示单词，列表示文档，单元格的值表示单词在文档中的权重，一般可由TF-IDF生成。

聪明的读者看到这里应该已经反应过来了，这不就是《机器学习（十四）》中提到的协同过滤的商品打分矩阵吗？

没错！LSA的实现方法的确与之类似。多数的blog讲解LSA算法原理时，由于单词-文档矩阵较小，直接采用了矩阵的SVD分解，少数给出了EM算法实现，实际上就是ALS或其变种。

参考：

http://www.cnblogs.com/kemaswill/archive/2013/04/17/3022100.html

Latent Semantic Analysis(LSA/LSI)算法简介

http://blog.csdn.net/u013802188/article/details/40903471

隐含语义索引（Latent Semantic Indexing）

http://www.shareditor.com/blogshow/?blogId=90

比TF-IDF更好的隐含语义索引模型是个什么鬼

# HMM

![](/images/article/HMM.png)

![](/images/article/HMM_2.png)

![](/images/article/HMM_3.png)

和HMM（Hidden Markov Model，隐马尔可夫模型）模型相关的算法主要分为三类，分别解决三种问题：

1）**知道骰子有几种（隐含状态数量），每种骰子是什么（转换概率），根据掷骰子掷出的结果（可见状态链），我想知道每次掷出来的都是哪种骰子（隐含状态链）。**

这个问题呢，在语音识别领域呢，叫做解码问题。这个问题其实有两种解法，会给出两个不同的答案。每个答案都对，只不过这些答案的意义不一样。第一种解法求最大似然状态路径，说通俗点呢，就是我求一串骰子序列，这串骰子序列产生观测结果的概率最大。第二种解法呢，就不是求一组骰子序列了，而是求每次掷出的骰子分别是某种骰子的概率。比如说我看到结果后，我可以求得第一次掷骰子是D4的概率是0.5，D6的概率是0.3，D8的概率是0.2。

2）**还是知道骰子有几种（隐含状态数量），每种骰子是什么（转换概率），根据掷骰子掷出的结果（可见状态链），我想知道掷出这个结果的概率。**

看似这个问题意义不大，因为你掷出来的结果很多时候都对应了一个比较大的概率。问这个问题的目的呢，其实是检测观察到的结果和已知的模型是否吻合。如果很多次结果都对应了比较小的概率，那么就说明我们已知的模型很有可能是错的，有人偷偷把我们的骰子給换了。问题2的更一般的用法是：从若干种模型中选择一个概率最大的模型。

3）**知道骰子有几种（隐含状态数量），不知道每种骰子是什么（转换概率），观测到很多次掷骰子的结果（可见状态链），我想反推出每种骰子是什么（转换概率）。**

这个问题很重要，因为这是最常见的情况。很多时候我们只有可见结果，不知道HMM模型里的参数，我们需要从可见结果估计出这些参数，这是建模的一个必要步骤。

参考：

https://www.zhihu.com/question/20962240

如何用简单易懂的例子解释隐马尔可夫模型？

http://www.cnblogs.com/kaituorensheng/archive/2012/11/29/2795499.html

隐马尔可夫模型

https://mp.weixin.qq.com/s/9MmHDVDal57pdotwxAn_uQ

HMM模型详解

## Viterbi算法

Viterbi算法是求解最大似然状态路径的常用算法，被广泛应用于通信（CDMA技术的理论基础之一）和NLP领域。

>注：Andrew James Viterbi，1935年生，意大利裔美国工程师、企业家，高通公司联合创始人。MIT本硕+南加州大学博士。viterbi算法和CDMA标准的主要发明人。

![](/images/article/HMM_4.png)

上图是一个HMM模型的概率图表示，其中{'Healthy','Fever'}是隐含状态，而{'normal','cold','dizzy'}是可见状态，边是各状态的转移概率。

![](/images/article/Viterbi_animated_demo.gif)

上图是Viterbi算法的动画图。简单来说就是，从开始状态之后的每一步，都选择最大似然状态的路径。由于每一步都是最优方案，因此整个路径也是最优路径。

参考：

https://mp.weixin.qq.com/s/FQ520ojMmbFhNMoNCVTKug

通俗理解维特比算法

## 前向算法

forward算法是求解问题2的常用算法。

仍以上面的掷骰子为例，要算用正常的三个骰子掷出这个结果的概率，其实就是将所有可能情况的概率进行加和计算。同样，简单而暴力的方法就是把穷举所有的骰子序列，还是计算每个骰子序列对应的概率，但是这回，我们不挑最大值了，而是把所有算出来的概率相加，得到的总概率就是我们要求的结果。

穷举法的计算量太大，不适用于计算较长的马尔可夫链。但是我们可以观察一下穷举法的计算步骤。

![](/images/article/forward_algorithm.png)

上图是某骰子序列的穷举计算过程，可以看出第3步计算的概率和公式的某些项，实际上在之前的步骤中已经计算出来了，前向递推的计算量并没有想象中的大。

## Baum–Welch算法

Baum–Welch算法是求解问题3的常用算法。

## HMM在NLP领域的应用

具体到分词系统，可以将“标签”当作隐含状态，“字或词”当作可见状态。那么，几个NLP的问题就可以转化为：

词性标注：给定一个词的序列（也就是句子），找出最可能的词性序列（标签是词性）。如ansj分词和ICTCLAS分词等。

分词：给定一个字的序列，找出最可能的标签序列（断句符号：[词尾]或[非词尾]构成的序列）。结巴分词目前就是利用BMES标签来分词的，B（开头）,M（中间),E(结尾),S(独立成词）

命名实体识别：给定一个词的序列，找出最可能的标签序列（内外符号：[内]表示词属于命名实体，[外]表示不属于）。如ICTCLAS实现的人名识别、翻译人名识别、地名识别都是用同一个Tagger实现的。

