---
layout: post
title:  机器学习（八）——在线学习
category: theory 
---

## 交叉验证（续）

由于$$S_{train}$$和$$S_{CV}$$是随机选取的，因此我们可以认为这里的经验误差$$\hat\varepsilon_{S_{CV}}(h_i)$$是$$h_i$$的泛化误差的一个很好的估计值。测试集一般占所有样本数的1/4~1/3，这里的30%是一个典型值。

还可以对模型作改进，当选出最佳的模型$$M_i$$后，再在全部数据S上做一次训练，显然训练数据越多，模型参数越准确。

简单交叉验证方法的缺点在于得到的最佳模型是在70%的训练数据上选出来的，不代表在全部训练数据上是最佳的。还有当训练数据本来就很少时，再分出测试集后，训练数据就太少了。

我们对简单交叉验证方法再做一次改进，如下：

>1.将全部训练集S分成k个不相交的子集，假设S中的训练样例个数为m，那么每一个子集有m/k个训练样例，相应的子集称作$$\{S_1,\dots,S_k\}$$。   
>2.每次从模型集合$$\mathcal{M}$$中拿出来一个$$M_i$$，然后在S中选择出k-1个子集$$S_1\cup\dots\cup S_{j-1}\cup S_{j+1}\cup\dots\cup S_k$$，在这个集合上训练$$M_i$$得到预测函数$$h_{ij}$$。在$$S_j$$上测试$$h_{ij}$$，得到相应的经验误差$$\hat\varepsilon_{S_j}(h_{ij})$$。   
>3.使用$$\frac{1}{k}\sum_{j=1}^k\hat\varepsilon_{S_j}(h_{ij})$$作为$$M_i$$泛化误差的估计值。   
>4.选出泛化误差估计值最小的$$M_i$$，在S上重新训练，得到最终的预测函数$$h_i$$。

这个方法被称为k-折叠（k-fold）交叉验证。一般来说k取值为10，这样训练数据稀疏时，基本上也能进行训练，缺点是训练和测试次数过多。

更极端的，如果$$k=m$$，则该方法又被称为leave-one-out交叉验证。

## 特征选择

特征选择（Feature Selection）严格来说也是模型选择中的一种。

假设我们想对维度为n的样本点进行回归，如果，n远远大于训练样例数m，且你认为其中只有很少的特征起关键作用的话，就可以对整个特征集进行特征选择，以减少特征的数量。

对于n个特征的$$\mathcal{M}$$来说，根据特征是否包含在最终结果中，可以写出$$2^n$$个不同的$$M_i$$。直接使用上面的交叉验证方法，计算量过大。这时可以采用如下启发式算法：

>1.初始化特征集$$\mathcal{F}=\emptyset$$。   
>2.Repeat {   
><span style="white-space: pre">	</span>(a)for 特征i=1 to n, {   
><span style="white-space: pre">	     </span>如果$$i\notin\mathcal{F}$$，则$$\mathcal{F}_i=\mathcal{F}\cup\{i\}$$。   
><span style="white-space: pre">	     </span>在$$\mathcal{F}_i$$上使用交叉验证方法评估它的泛化误差。   
><span style="white-space: pre">	</span>}   
><span style="white-space: pre">	</span>(b)将第(a)步中最优的$$\mathcal{F}_i$$设为新的$$\mathcal{F}$$。   
>}   
>3.选择并输出搜索过程中得到的最优子集。

这个算法被称为前向搜索（forward search）。其外部循环的终止条件为$$\lvert\mathcal{F}\rvert$$达到n或者事先设定的门限值。

前向搜索属于wrapper model特征选择方法的一种。 Wrapper这里指不断地使用不同的特征子集来测试学习的算法。

除了前向搜索之外，还有后向搜索（backward search）算法。它和前者的区别在于，它的初始集合为全集，然后每次删除一个特征，并评价，直到$$\lvert\mathcal{F}\rvert$$达到阈值或者为空，然后选择最佳的$$\mathcal{F}$$即可。

可以看出无论前向搜索，还是后向搜索，其算法复杂度都是$$O(n^2)$$。

## KL散度

KL散度（Kullback–Leibler divergence）是两个随机分布间距离的度量。其定义如下：

$$D_{KL}(P\|Q)=\sum_iP(i)\log\frac{P(i)}{Q(i)}$$

其中，P和Q是离散概率分布，$$P(i)$$和$$Q(i)$$是相应分布的概率密度函数。如果P和Q是连续随机变量的话，将上式中的累加符号换成积分符号即可。

但KL散度并不是真正的度量（metric）。它既不满足三角不等式(两边之和$$\ge$$第三边)，也不满足对称性（即$$D_{KL}(P\|Q)\neq D_{KL}(Q\|P)$$）。

>注：Solomon Kullback，1907～1994，美国数学家和密码学家。乔治·华盛顿大学博士。NSA首任首席科学家。二战期间，参与破解德国的Enigma机器。

>Richard Leibler，1914～2003，美国数学家和密码学家。伊利诺伊大学博士。NSA高级主管，入选NSA名人堂。

## 过滤特征选择方法

过滤特征选择（Filter feature selection）方法，是另一种启发式的特征选择算法，计算量比较小。它的思路是计算特征$$x_i$$和类别标签y之间的相关度的评分$$S(i)$$。

可以使用$$x_i$$和y之间的互信息量（mutual information），作为评分依据。

$$MI(x_i,y)=\sum_{x_i\in X_i}\sum_{y\in Y}p(x_i,y)\log\frac{p(x_i,y)}{p(x_i)p(y)}$$

其中，$$p(x_i,y)$$是$$x_i$$和y的联合概率密度，$$p(x_i)$$和$$p(y)$$是相应的边缘概率密度。

和KL散度类似，如果x和y是连续随机变量的话，将上式中的累加符号换成积分符号即可。

MI也可以用KL散度来表示：

$$MI(x_i,y)=KL(p(x_i,y)\|p(x_i)p(y))$$

过滤特征选择方法的算法复杂度为$$O(n)$$。

最后一个问题，选择多少个特征合适呢？按照$$S(i)$$从高到低的顺序，依次选择1到n个特征进行交叉验证，直到效果达到预期为止。

## 贝叶斯统计和规则化

前面提到最大似然（maximum likelihood）估计方法的公式如下：

$$\theta_{ML}=\arg\max_\theta\prod_{i=1}^mp(y^{(i)}\vert x^{(i)};\theta)$$

从频率统计（frequentist statistic）学派的观点来看，这里的$$\theta$$是一个未知的常数，我们的任务就是求出这个常数。然而从贝叶斯学派的观点来看，$$\theta$$是一个未知的随机变量。

也就是说似然函数，对于前者来说，是这样的：$$\prod_{i=1}^mp(y^{(i)}\vert x^{(i)};\theta)$$；但对于后者来说，却是这样的：$$\prod_{i=1}^mp(y^{(i)}\vert x^{(i)},\theta)$$

我们首先假定$$\theta$$的分布为$$p(\theta)$$，这种假定由于没有事实根据，通常被称作先验分布（prior distribution）。

我们针对训练集$$S=\{(x^{(i)},y^{(i)})\}_{i=1}^m$$，训练得到预测函数。并按照如下公式计算后验分布（posterior distribution）：

$$p(\theta\vert S)=\frac{p(S\vert\theta)p(\theta)}{p(S)}\tag{1}$$

由全概率公式可得：

$$p(S)=p(S\vert\theta_1)p(\theta_1)+\dots+p(S\vert\theta_n)p(\theta_n)$$

上式的$$\theta_i$$表示$$\theta$$的各个取值区间，然而由于$$\theta$$是连续随机变量，根据微积分原理可得：

$$p(S)=\int_\theta p(S\vert\theta)p(\theta)\mathrm{d}\theta\tag{2}$$

将公式2代入公式1可得：

$$p(\theta\vert S)=\frac{p(S\vert\theta)p(\theta)}{\int_\theta p(S\vert\theta)p(\theta)\mathrm{d}\theta}=\frac{\left(\prod_{i=1}^mp(y^{(i)}\vert x^{(i)},\theta)\right)p(\theta)}{\int_\theta\left(\prod_{i=1}^mp(y^{(i)}\vert x^{(i)},\theta)\right)p(\theta)\mathrm{d}\theta}$$

当我们针对新的样本x进行预测时，和上面的推导类似，可得：

$$p(y\vert x,S)=\int_\theta p(y\vert x,\theta,S)p(\theta\vert S)\mathrm{d}\theta$$

因为预测样本集和训练样本集的分布是独立的，因此上式又可写为：

$$p(y\vert x,S)=\int_\theta p(y\vert x,\theta)p(\theta\vert S)\mathrm{d}\theta$$

这个公式又被称作后验预测分布（Posterior predictive distribution）。

$$p(\theta\vert S)$$可由前面的公式得到。

假若我们要求期望值的话，那么套用求期望的公式即可：

$$E[y\vert x,S]=\int_y yp(y\vert x,S)\mathrm{d}y$$

由上可见，贝叶斯估计将$$\theta$$视为随机变量，$$\theta$$的值满足一定的分布，不是固定值，我们无法通过计算获得其值，只能在预测时计算积分。

上述贝叶斯估计方法，虽然公式合理优美，但后验概率$$p(\theta\vert S)$$通常是很难计算的，因为它是$$\theta$$上的高维积分函数。

观察$$p(\theta\vert S)$$的公式，在分母$$P(S)$$一定的情况下，分子越大则值越大，也就是$$p(\theta\vert S)$$的概率越大。

因此，可得如下算法：

$$\theta_{MAP}=\arg\max_\theta\left(\prod_{i=1}^mp(y^{(i)}\vert x^{(i)},\theta)\right)p(\theta)$$

这个算法叫做最大后验概率估计（maximum a posteriori）。

和ML相比，MAP算法只是在最后多了一项$$p(\theta)$$。通常使用中，我们认为$$p(\theta)$$符合$$\theta\sim N(0,\tau^2I)$$。

由于$$p(\theta)$$实际上就是先验分布，它会对最后结果进行一定的修正。因此，实际上最大后验概率估计相对于最大似然估计来说，较容易克服过度拟合的问题。

![](/images/article/MAP.png)

参见：

http://www.cs.cornell.edu/courses/cs5540/2010sp/lectures/Lec9.Estimation-continued.pdf

# 在线学习

我们之前讨论的算法，都是给定一个训练集S，经训练之后，得到预测函数h，然后再在新的样本集上进行预测。这种方法被称为批量学习（batch learning）。

与之相对的，还有一种边学习边预测的在线学习（online learning）算法。其步骤如下：

>1.$$i:=0$$。   
>2.输入$$x^{(i)}$$，算法预测$$y^{(i)}$$。   
>3.根据$$y^{(i)}$$的真实值，修正算法模型。这一步也被称作更新过程（update procedure）   
>4.令$$i:=i+1$$，以处理下一个数据样本。

在线学习的优点：
1.算法在学习过程中，即可预测。
2.随着数据样本的增多，预测会更加准确，即具有自我完善的的能力。

下面以感知器（perceptron）算法为例，讨论一下在线学习的误差问题。

首先回忆一下感知器算法：

$$h_\theta(x)=g(\theta^Tx),y\in\{1,-1\}$$

$$g(z)=\begin{cases}
1, & z\ge 0 \\
-1, & z<0 \\
\end{cases}$$

对于训练样本$$(x,y)$$，其更新过程为：

$$\theta:=\begin{cases}
\theta, & h_\theta(x)=y \\
\theta+yx, & otherwise \\
\end{cases}\tag{3}$$

这里给出一个和感知器算法有关的定理：

对于给定的m个样本**序列**（注意“序列”二字，在线算法对于样本的顺序是敏感的）$$x^{(i)}$$，如果所有的$$\|x^{(i)}\|\le D$$，并且存在单位向量u，使得所有样本的$$y^{(i)}(u^Tx^{(i)})\ge \gamma$$，则感知器算法在该序列上的预测错误个数最多为$$(D/\gamma)^2$$。

