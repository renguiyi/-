最近在复习逻辑斯蒂回顾的基础，发现之前的学习笔记存在诸多问题，最大的问题来自于对很多概念的理解不深，只知其然，不知其所以然，于是断断续续两个月时间，解答了不少关于逻辑斯蒂回归的常见问题，这是一篇总集，将从逻辑斯蒂回归的介绍开始，逐步深入到各种细节问题，相信读完这一篇，你对逻辑斯蒂回归模型的理解会到达一个比较深入的层次。
如若存在错误，或是还有其他关于逻辑斯蒂回归的问题，我会尽早修正更新，欢迎大家指正！

# 1. 逻辑斯蒂回归的简介
##  1.1 从感知机到逻辑斯蒂回归

在感知机中，我们知道一个分离超平面 ![](http://latex.codecogs.com/gif.latex?\\wx 将特征空间分成两个部分，样本在不同的子空间中则被分为相对应的类。但是感知机的一个问题在于，我们仅能知道这个样本被分类的结果，但不知道它属于一个类的概率是多少。
换句话说，样本在特征空间中的位置可能与分离超平面距离非常近，也有可能非常远，如果距离较远，那么它更有可能被分成它所在一侧对应的类，但是如果与超平面的距离非常近，说明它被分成另一类的可能性也很大，比如被分成A的可能性为51%，而分成B类的可能性为49%，此时线性回归会将其分为A类，而忽略了49%分成B类的可能性：
​

编辑
添加图片注释，不超过 140 字（可选）
于是，为了得到这一概率，我们引入了 Sigmoid 函数：
\\sigmoid (x)=\frac{1}{1+e^{-x}}  
​

编辑

切换为全宽
添加图片注释，不超过 140 字（可选）
Sigmoid 函数能够将感知机的预测值(-∞，+∞)转换到概率的(0,1)区间，这样，就可以显示一个样本被分为一个类的概率是多少了（引入问题：为什么经过 Sigmoid 函数转换后得到的值，就可以认为是概率？请看1.3）：
P=sigmoid(wx)=\frac{1}{1+e^{-wx}} \\ 
比如我们认为A类为正类，B类为负类，那么当某个样品分为A类的概率>50%，我们可认为其为A类，如果<50%，我们可认为其为B类（引入问题：为什么要用0.5作为分类阈值？请看1.4）：
\hat y = \left\{ \begin{array}{} 1, &P>0.5 \\ 0, &P < 0.5  \end{array} \right\}\\ 
知道概率的好处是什么呢？现实中我们很可能遇到这样一种情况，就是要得到最有可能被分到A类的前10个样本，以对用户进行推荐，这样我们只需要对模型的结果进行排序，就可以解决这个问题。
1.2 模型的训练：
在逻辑斯蒂回归中，使用的损失函数是对数似然函数，也就是交叉熵（引入问题：为什么逻辑斯蒂回归使用交叉熵，而不是MSE？请看1.5），现在先写出似然函数：
L(w)=\prod_{i=1}^{N}[P(x_{i})]^{y_{i}}[1-P(x_{i})]^{1-y_{i}}\\ 
为了计算方便，我们对似然函数取对数，得到对数似然函数：
logL(w)= \sum_{i=1}^{N}[y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]\\  = \sum_{i=1}^{N}[y_ilogP(x_i)-y_ilog(1-P(x_i))+log(1-P(x_i))]\\ =\sum_{i=1}^{N}[y_ilog\frac{P(x_i)}{1-P(x_i)} + log(1-P(x_i))]\\ =\sum_{i=1}^{N}[y_ilog\frac{\frac{1}{1+e^{-wx}}}{1-\frac{1}{1+e^{-wx}}}+log(1-\frac{1}{1+e^{-wx}})]\\ =\sum_{i=1}^{N}[wy_ix_i-log(1+e^{wx_i})]\\ 
接着我们对 w 求导，得到梯度：
(wy_ix_i)' = y_ix_i\\ log(1+e^{wx_i})'=\frac{e^{wx_i}}{1+e^{wx_i}}x_i=x_iP(x_i)\\ 
\frac{\partial L(w)}{\partial w}=\sum_{i=1}^{n}(y_{i} x_{i}-\frac{x_i}{1+e^{-w \cdot x_{i}}})\\ 
接下来只要采用梯度下降法或拟牛顿法等即可，当然，优化方法就是另一个坑了。
代码实现：machine-learning-notesandcode/逻辑斯蒂回归Logistic Regression at master · Zhouxiaonnan/machine-learning-notesandcode (github.com)
1.3 为什么经过 Sigmoid 函数转换后得到的值，就可以认为是概率？
如果要用一句话解答这个问题，我会这么说：因为逻辑斯蒂回归的 Sigmoid 函数是符合广义线性模型（General Linear Model）的伯努利分布（Bernoulli Distribution）的规范联系函数（Canonical Link Function）的反函数，Sigmoid 函数将线性函数映射到伯努利分布的期望。
第一次接触这个问题，可能会对里面的各种新概念搞晕，没有关系，我们一个一个来解释，首先，了解一下什么是广义线性模型。
1.3.1 广义线性模型
线性模型的定义为
​

编辑

切换为居中
《机器学习》中对线性模型的定义
那么在广义线性模型中所谓的“广义”二字的含义，即可以将一些符合特定特征的模型通过一定的转换，将其转变为线性模型的形式，这些模型就被囊括进了广义线性模型之中。
举一个简单的例子：如果标记值（label）随着输入的变化呈指数型增长，那么我们就可以用标记值的对数作为线性模型逼近的目标，即
\ln y = w^{T}x + b\\
这里我们将一个非线性的函数转换为了线性函数，其中 \ln(·) 就是在该特例下的联系函数，联系函数用 g(·) 代表， y = e^{\textbf{wx} + b} 为联系函数的反函数 g^{-}(·) 。
联系函数的定义是，如果该函数的反函数能够将线性方程映射到某个广义线性模型的期望，则该函数为联系函数，记住这个概念，很重要。
现在，我们已经有一丝丝熟悉的感觉了，你会发现其实在逻辑斯蒂模型中，Sigmoid 函数就是将线性函数映射到了区间(0,1)，那么 Sigmoid 函数会不会就是某个广义线性模型的联系函数的反函数呢？由于逻辑斯蒂回归属于二分类模型，而两个事件的发生概率，可假设符合伯努利分布，那么这个广义线性模型会不会就是伯努利分布呢？
于是我们做出如下猜测：
伯努利分布的联系函数的反函数 g^{-}(·) 是 Sigmoid 函数 y =  \frac{1}{1+e^{-x}} ，该函数将线性函数映射到了伯努利分布的期望上，这使其输出可以作为概率。
注意，目前这只是一个推测，没有完整的数学证明，接下来我们需要证明两点：
证明伯努利分布属于指数分布族。
如果伯努利分布符合指数分布族，那么其联系函数是什么？
1.3.2 伯努利分布是否属于指数分布族
伯努利分布为离散分布，该分布来自于伯努利试验的结果，伯努利试验是在同样的条件下重复地、相互独立地进行的一种随机试验，这种随机试验只有两种可能结果：发生或者不发生，即0或者1，而 P 就代表发生或者不发生的概率，习惯上，我们认为 P 代表发生的概率，即1的概率。
f(x;p)=\left\{ \begin{array}{} p, &x=1\\ 1-p, &x=0 \end{array} \right\}\\ 
​

编辑

切换为全宽
伯努利分布
我们可以将上式的约束条件合并，得到下面的形式：
f(x;p)=p^x(1-p)^{1-x},x\in\{0,1\}\\ 
我们可以得到伯努利分布的期望：
E=1\times P + 0 \times (1-P)=P\\ 
接下来，我们验证伯努利分布属于指数分布族。
首先，从广义线性模型的分布来看，当一个分布能够写成如下形式的时候，即说明该分布属于指数分布族。
P(x;\eta)=b(x)exp(\eta T(x)-a(\eta))\\ 
其中 b(y)，T(y) 和 a(η) 确定了一个分布，而 η 是该分布的参数。事实上，这个 η 参数，叫做自然参数（natural parameter），可以用线性函数的 wx 替代。
那么让我们看看伯努利分布是否可以转换为这种形式：
f(x;p)=p^x(1-p)^{1-x}\\ =exp(xlogp + (1-x)log(1-p))\\ =exp(ylogp-ylog(1-p)+log(1-p))\\ =exp(ylog\frac{p}{1-p}+log(1-p)) 
哦豁？我们发现伯努利分布确实符合指数分布族的形式，其中：
\eta = log\frac{p}{1-p}\\ 
1.3.3 推导伯努利分布的联系函数
之前我们说到，实际上自然参数 η 就是线性函数 wx，我直接代入上式：
wx=log\frac{p}{1-p}\\ \Rightarrow -wx=log\frac{1-p}{p}\\ \Rightarrow e^{-wx}=\frac{1}{p} - 1\\ \Rightarrow1+e^{-wx}=\frac{1}{p}\\ \Rightarrow p=\frac{1}{1+e^{-wx}} 
然后我们发现这不就是 Sigmoid 函数吗？
好，我们之前提到，广义线性模型的联系函数的反函数将线性函数映射到指数分布族的期望，而上式正符合这一描述，事实上，上式也就是我们所要找的伯努利分布的联系函数的反函数，因此其联系函数就是：
wx=log\frac{p}{1-p}\\ 
至此，证明完毕。回到最初的问题，为什么使用逻辑斯蒂回归将实数域映射到(0,1)，就可以代表概率？因为 sigmoid 函数是伯努利分布的联系函数的反函数，它将线性函数映射到了伯努利分布的期望上，而伯努利分布的期望本身就是概率，因此，我们最终从逻辑斯蒂回归得到的输出，可以代表概率，也正是因为它代表概率，才落在(0,1)之间。
最后，关于这一推导的逻辑，我做了一个流程图放在这里，供大家参考。
​

编辑

切换为居中
添加图片注释，不超过 140 字（可选）
1.4 为什么用0.5作为分类的阈值
0.5这个阈值似乎是最直接也是最符合逻辑的，但事实上并非如此，这个阈值基于总体中正例和反例符合1:1的假设，也就是常说的正例和反例相平衡。
而我们用来训练的样本数据，通常是从总体中进行抽样得到，因此其正反例的分布也大致符合总体的分布，如果样本数据平衡，那么我们可以假设总体数据平衡，那么设置0.5为阈值便是合理的。
1.4.1 样本不平衡造成的结果
但如果我们的样本数据中的样本不平衡会发生什么呢？比如正例和反例的比值为10:1。
在这种情况下，模型会更倾向于去拟合正例，这是由于我们的损失函数所引起的。
我们知道逻辑斯蒂回归的损失函数为每个误分类样本的交叉熵之和：
logL= \sum_{i=1}^{N}[y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]\\ 
当正例远多于反例时，其因被误分类而造成的损失会更大，因而在优化的时候，模型就会更加倾向于拟合正例，最后导致正例的判断比较准确，而负例判断不准。
一个极端的情况是，正例和负例的比值为999:1，那么模型只需要将所有样本判断为正，其准确率就高达：
999/(999+1)=99.9\%\\ 
1.4.2 解决方案1：样本增强
既然样本不平衡，那就在根本上解决问题，在训练的时候减少输入数量较多的类的样本，或者收集更多数量较少的类的样本，让模型训练时输入的样本平衡即可。
对于不同的样本类型，可以存在不同的增强方式，比如对于图像而言，只需要将图像进行旋转，裁切，翻转，遮掩等都可以得到大量的新样本。
对于文本数据，则可以通过转义，同义词替换等等方式。
其他增强方式还有 SMOTE 等方法，样本增强本身是一个很大的课题，这里不展开讲解。
1.4.3 解决方案2：自定义损失函数
既然正例判断错误会贡献更多的损失，那么我们只需要对损失函数略作修改，让判断错误正例提供的损失减小，或者增大判断错误反例的损失，这样模型就不会偏向双方中的某一方了。
比如将逻辑斯蒂回归的损失函数改为：
logL= \sum_{i=1}^{N}[\frac{1}{10}y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]\\ 
当然，我们可以直接定义一个固定值的 factor，但是根据样本中正反例的比值确定 factor 也是可以的，显然这样更加方便，也可以根据实际情况的改变而适时调整，首先计算正例和反例的比例，其中 s+，s-，s 分别为为正例的个数，反例的个数，以及样本的个数：
\alpha^+=\frac{s^+}{s}\\ \alpha^-=\frac{s^-}{s}\\ 
一般而言我们会减少数量较大的类别的损失，因为我们不想每次优化时的步长太大，比如正例的数量较多，可以将损失函数改为：
logL= \sum_{i=1}^{N}[\alpha^+y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]\\ 
1.4.4 解决方案3：改变判断类别的阈值
如果我们不对损失函数进行改变，那么可以改变判断正反例的阈值，逻辑上也很直接，既然模型更加偏向将样本判断为正例，那么在最后分类的时候，让判断为正例的阈值更加严格一点，比如之前 > 0.5 即可判断为正例，现在 > 0.8 才可以判断为正例。
与损失函数相同的一点是，我们可以根据样本中正反例的数量的比值来确定分类的阈值。  class = \begin{cases} +1,\quad y > \alpha^+\\ -1,\quad y \le \alpha^-\end{cases}\\
1.5 为什么逻辑斯蒂回归使用交叉熵，而不是MSE
1.5.1 最本质的原因：(KL散度与交叉熵的关系）
首先我们要明白 MSE 和交叉熵衡量的到底是什么。
对于 MSE 而言，其公式为：
\frac{1}{m}\sum_{i=1}^m(y_i - \hat{y}_i)^2\\
所以 MSE 衡量的是真实值 - 预测值的平方和的平均值，放在坐标系中，也就是我们所熟知的欧氏距离，显然欧式距离越大，说明预测的越不准确，因此我们要最小化这一损失函数。
那么对于交叉熵而言，我们实际上是在衡量真实概率分布与预测概率分布之间的差异，因此我们真正在计算的实际上是KL散度，那么为什么使用交叉熵的公式呢，可以先看下KL散度公式：
D_{KL}(p||q)=\sum_{i=1}^np(x_i)log\frac{p(x_i)}{q(x_i)}\\
其中 p 表示的是真实分布，q 表示的是预测分布，当真实分布 p 和预测分布 q 非常不同时，则 KL 散度很大，当两个分布较为接近时，则 KL 散度较小。
现在我们将 KL 散度的公式拆开看一下：
D_{KL}(p||q)=\sum_{i=1}^np(x_i)log\frac{p(x_i)}{q(x_i)}\\ =\sum_{i=1}^np(x_i)log(p(x_i))-\sum_{i=1}^np(x_i)log(q(x_i))\\
在其中我们看到了两项，第一项是信息熵，只与我们目前所知道的信息有关，也就是只与我们用来训练的数据有关，因此这一项是已知的常数，但要注意的是，p 代表的真实概率分布只是近似的真实概率分布，因为这个概率分布是用样本得到的，而非使用总体得到，并且我们无法得到总体的数据。
第二项就是我们都很熟悉的交叉熵，对上面的式子进行一个简单的转换可得：
D_{KL}(p||q)=\sum_{i=1}^np(x_i)log(p(x_i))-\sum_{i=1}^np(x_i)log(q(x_i))\\ \Rightarrow D_{KL}(p||q) +(- \sum_{i=1}^np(x_i)log(p(x_i))) = -\sum_{i=1}^np(x_i)log(q(x_i))\\
也就是说交叉熵就是 KL 散度加上信息熵，而信息熵是一个常数，并且在计算的时候，交叉熵相较于 KL 散度更容易，所以我们直接使用了交叉熵作为损失函数。
因此我们在最小化交叉熵的时候，实际上就是在最小化 KL 散度，也就是在让预测概率分布尽可能地与真实概率分布相似。
现在回到我们最初的问题，为什么逻辑斯蒂回归使用交叉熵而不是MSE，从逻辑的角度出发，我们知道逻辑斯蒂回归的预测值是一个概率，而交叉熵又表示真是概率分布与预测概率分布的相似程度，因此选择使用交叉熵。从MSE的角度来说，预测的概率与欧氏距离没有任何关系，并且在分类问题中，样本的值不存在大小关系，与欧氏距离更无关系，因此不适用MSE。
1.5.2 MSE 的损失小于交叉熵的损失，导致对分类错误的点的惩罚不够：
关于这一点，其实我们直接作图对比一下就知道了，假设 label 为 1，预测为 y_predict，那么预测错误的点的 Loss 如下图所示，显然 log-loss 比 MSE 要大，并且对于预测错误的样本，错得越离谱，惩罚越严重。
​

编辑
添加图片注释，不超过 140 字（可选）
1.5.3 MSE 的梯度消失效应较大：
如果我们使用 MSE 作为损失函数，那么来计算一下其梯度：
Loss = (\hat{y}_i - y_i )^2\\ \frac{\partial Loss}{\partial w} =2(\hat{y}_i - y_i)\frac{\partial (\hat{y}_i - y_i)}{\partial w}\\ = 2(\hat{y}_i - y_i)\frac{\partial (\frac{1}{1+e^{-wx_i}})}{\partial w}\\ = 2(\hat{y}_i - y_i)(\frac{1}{1+e^{-wx_i}})^2\frac{\partial e^{-wx_i}}{\partial w}\\ = 2(\hat{y}_i - y_i)(\frac{1}{1+e^{-wx_i}})^2e^{-wx_i}\frac{\partial (-wx_i)}{\partial w}\\ = 2(y_i - \hat{y}_i)(\frac{1}{1+e^{-wx_i}})^2e^{-wx_i}x_i
因为：
(\frac{1}{1+e^{-wx_i}})^2e^{-wx_i}x_i = \hat{y_i}(1-\hat{y_i})\\
所以：
\frac{\partial Loss}{\partial w} =2(y_i - \hat{y}_i)\hat{y_i}(1-\hat{y_i})x_i\\
我们可以看到在 MSE 的梯度中存在一项 \hat{y_i}(1-\hat{y_i}) ，也就是说，当我们的预测值接近于1或者接近于0时，会导致其梯度接近0，无法进行有效学习。
1.5.4 损失函数的凸性（使用 MSE 可能会陷入局部最优）
对于一个非凸函数，模型有可能陷入局部最优而无法再继续优化：
​

编辑

切换为居中
添加图片注释，不超过 140 字（可选）
以 MSE 为损失函数的逻辑斯蒂回归就是一个非凸函数，如何证明这一点呢，要证明一个函数的凸性，只要证明其二阶导恒大于等于0即可，如果不是恒大于等于0，则为非凸函数。
让我们对上文中求得的 MSE 一阶导数继续求二阶导：
\frac{\partial(y_i - \hat{y}_i)\hat{y_i}(1-\hat{y_i})x_i}{\partial w}\\ = (-y_ix_i\hat{y_i}+(1+y_i)x_i\hat{y_i}^2-x_i\hat{y_i}^3)'\\ =-y_ix_i^2\hat{y_i}(1-\hat{y_i})+2(1+y_i)x_i^2\hat{y_i}^2(1-\hat{y_i})-3x_i^2\hat{y_i}^3(1-\hat{y_i})\\ = \hat{y_i}(1-\hat{y_i})x_i^2(-y_i+2(1+y_i)\hat{y_i}-3\hat{y_i}^2)\\
其中  \hat{y_i}(1-\hat{y_i})x_i^2 恒为正，只需要看 -y_i+2(1+y_i)\hat{y_i}-3\hat{y_i}^2 即可。
我们知道真实 label yi 只能取1和0，当yi=1时，上式取值范围为(-4,3)，当yi=0时，上式取值范围为(-3,2)，因此二阶导不恒大于等于0，因此 MSE 损失函数为非凸函数。
2. 逻辑斯蒂回归的正则化
我们知道很多模型都需要做正则化，不做正则化产生的问题是，模型很有可能过拟合，在逻辑斯蒂回归中的具体表现，就是模型会尝试使用所有的特征做拟合，并且特征权重的数值特别大，那么为什么这样会导致过拟合呢？
这是因为我们的样本数据往往包含着一些无用特征或者噪音，模型使用了所有特征，说明对无用特征也做了拟合，那么在预测时无用特征的作用会导致预测结果的偏离。
而特征权重如果特别大，那么当一个特征的数值有一个小波动（噪音），就会造成预测结果的大幅度变化，也会造成结果的偏离。
​

编辑

切换为居中
左边是欠拟合，右边是过拟合
这就是为什么有时候我们的模型在样本数据中表现良好，而在测试集中表现糟糕，其实就是发生了过拟合现象。
2.1 正则化的思想
正则化就是解决过拟合的方法之一，它的主要思想就是，在我们的损失函数后面，加上一个对权重的惩罚项（先验知识），也就是常说的正则项，两者即组成目标函数，在训练时，最小化这个目标函数，就会限制模型使用的特征数量或者特征权重的大小。
我们知道逻辑斯蒂回归的似然函数为：
L = \prod_{i=1}^{N}[P(x_{i})]^{y_{i}}[1-P(x_{i})]^{1-y_{i}}\\
加入正则项后，可以写成：
L = \prod_{i=1}^{N}[P(x_{i})]^{y_{i}}[1-P(x_{i})]^{1-y_{i}}\prod_{j=1}^{M}H(w_j)\\
其中 \prod_{j=1}^{M}H(w_j) 就是正则项，M 是特征的数量，当我们对整个目标函数取对数后，即变成：
logL= \sum_{i=1}^{N}[y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]+\sum_{j=1}^MH(w_j)\\
接下来我们看看具体到底有哪些正则化项，以及这些正则化项的具体作用是什么。
2.2 L0 正则
针对模型尝试使用所有特征拟合的问题，提出了两种解决方式，即 L0 正则和 L1 正则。
L0 正则化的想法十分直接，既然我们希望模型不要使用所有特征，那么只要让正则化项代表权重为非0的个数就好了，即：
H(w_J)= \left\{\begin{array}{} 1, & w_j\ne0\\ 0, & w_j = 0\\ \end{array} \right\} \\
虽然 L0 正则化特别容易理解，但在迭代时，并不容易求解，因为这是一个 NP 难问题。
因此实际上我们常用的是 L1 正则。
2.3 L1 正则
同样是为了解决模型尝试使用所有特征拟合的问题，L1 正则加入的先验知识是，模型的权重符合拉普拉斯分布，且平均值为0：
f(x)=\frac{1}{2\lambda}e^{-\frac{|x-\mu|}{\lambda}}\\
​

编辑

切换为全宽
拉普拉斯分布
此时我们的目标函数为：
L = \prod_{i=1}^{N}[P(x_{i})]^{y_{i}}[1-P(x_{i})]^{1-y_{i}}\prod_{j=1}^{M}\frac{1}{2\lambda}e^{-\frac{|w_j-\mu|}{\lambda}}\\
其中 \mu 和 \lambda 都是拉普拉斯分布的权重， \mu  取0，然后对目标函数取对数后变成：
logL= \sum_{i=1}^{N}[y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]+\sum_{j=1}^M-\frac{|w_j|}{\lambda}log\frac{1}{2\lambda}\\
由于 \lambda 为常数，我们可以忽略，于是整个目标函数为：
logL= \sum_{i=1}^{N}[y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]-\sum_{j=1}^M|w_j|\\
最后，习惯上我们再加一个负号，这样就可以使用梯度下降法优化了：
-logL= -\sum_{i=1}^{N}[y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]+\sum_{j=1}^M|w_j|\\
当然，我们也可以在正则项前加一个超参数，控制优化效果。实际上这个超参数相当于我们刚刚忽略的 \frac{1}{2\lambda^2} ，因此这个超参数的作用就是改变特征权重服从的拉普拉斯分布的形态。
那么为什么 L1 正则可以让模型中一部分特征的权重为0呢，我们可以看一下对绝对值求导时会发生什么，首先我们知道对于绝对值，在0点不可导，因此需要分段求导：
\left\{ \begin{array}{} (|w|)'=1, &w>0\\ (|w|)'=-1, &w<0  \end{array} \right\}\\
这里其实就可以看出来了，在 L1 正则化中，每次优化，都会下降一个固定的值，直到其值为0，更直观的图是这样的：
​

编辑
L1正则的每轮优化
那为什么 L1 正则可以选择出对预测不重要的特征，而不是把重要的特征的权重也一并下降到0呢？
这是因为不要忘了在目标函数中，对权重的优化不仅是正则项，还有原来的交叉熵，目标函数不仅要考虑的是权重是否为0，还要考虑分类的准确性，对分类不重要的特征，正则项的优化贡献大，最终就被优化到0，而对分类重要的特征，交叉熵的优化贡献大，于是就不会被优化到0，最后这个特征就被保留了下来。
因为这个特性，L1 正则也经常被用于特征的选择，当然，这就不是这篇文章讨论的范畴了。
现在解决了模型使用不重要的特征的问题，那有什么方法可以解决特征的权重大的问题呢，这就要谈到 L2 正则了。
2.4 L2 正则
L2 正则加入的先验知识是，模型的权重符合正态分布，且平均值为0。
正态分布我们十分熟悉：
f(w)=\frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(w-\mu)^2}{2\sigma^2}}\\
​

编辑

切换为全宽
正态分布
与 L1 的推导过程一样，L2 的目标函数为：
L = \prod_{i=1}^{N}[P(x_{i})]^{y_{i}}[1-P(x_{i})]^{1-y_{i}}\prod_{j=1}^{M}\frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(w_j-\mu)^2}{2\sigma^2}}\\ logL=\sum_{i=1}^{N}[y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]+\sum_{j=1}^M-\frac{w_j^2}{2\sigma^2}log\frac{1}{\sqrt{2\pi}\sigma}\\ =\sum_{i=1}^{N}[y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]-\sum_{j=1}^M\frac{w_j^2}{2}\\ -logL=-\sum_{i=1}^{N}[y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]+\sum_{j=1}^M\frac{w_j^2}{2}\\
这里对正则项保留一个 1/2 是为了求导时计算方便，对优化效果没有影响。
然后我们提出正则项，仅对正则项求导，发现梯度为：
(\frac{w^2}{2})'=w\\
也就是说梯度随着权重 w 的下降而下降，因此一开始下降的比较快，往后越来越慢，它会慢慢的逼近0，而最终不会等于0，其优化图像是这样的：
​

编辑

切换为全宽
添加图片注释，不超过 140 字（可选）
如此一来，L2 便解决了特征权重过大造成过拟合的问题。
2.5 对经常看到的L1和L2可视化图像的解释
在学习 L1 和 L2 正则的时候，我们经常可以看到这样一张图：
​

编辑

切换为全宽
左边是L1正则化，右边是L2正则化
对于这张图我在刚开始学习的时候根本不明白为什么这就解释了 L1 和 L2 的特点，相信很多人和我一样有着相同的困扰，这里对此进行一些解释。
首先假设我们只有两个特征，对应两个特征权重 w1 和 w2，即图中坐标轴的横轴和纵轴。图中的彩色等高线，实际上就是第三个维度，表示的是整个目标函数在不同的 w1 和 w2 下的取值，也就是 -logL 的值，其中相同颜色的线，表示 -logL 的值相同。
如果将其再简化到1个特征，就像是这样：
​

编辑
添加图片注释，不超过 140 字（可选）
所以相同颜色的等高线，就是指不同的权重组合，但 -logL 的值相同。
图中由红到紫，-logL 的值由大到小，我们最终的目的，是让一个权重组合计算得到的 -logL 到达紫色圈的中心，即全局最优点。
现在，对于L1正则项来说，它是这样的（因为我们假定只有两个特征）：
|w_1| + |w_2|\\
在图中画出来，就是左边的菱形。在红色圈的每一个点上，我们都可以画出这样的一个菱形，菱形越大，代表两者绝对值之和越大，在 -logL 不改变的情况下，我们希望两者的绝对值之和最小，那么最优的菱形，就是图中所示的菱形，其中一个权重为0，这就形象的说明了为什么 L1 会让部分权重为0。
对于 L2 正则，也是一样的道理，L2 正则是这样的：
\frac{w_1^2}{2} + \frac{w_2^2}{2}\\
画出来就是右边的黑色圆形，那么在 -logL 不改变的情况下，最优的圆形，就是与红色等高线相切的圆形了。这也就解释了为什么加了 L2 正则化的权重不会为0。
2.6 Elastic Net
弹性网络则是对 L1 正则和 L2 正则的一个结合，通过调节超参数，可以调整 L1 正则项和 L2 正则项对优化的贡献程度，弹性网络的公式是这样的：
-logL=-\sum_{i=1}^{N}[y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]+\lambda\sum_{j=1}^M[\rho\frac{w_j^2}{2} + (1-\rho)|w_j|]\\
调整超参 λ 和 ρ 就可以调整两种正则的贡献程度。
3. 逻辑斯蒂回归实现多分类的方法
逻辑斯蒂回归本身只能用于二分类问题，如果实际情况是多分类的，那么就需要对模型进行一些改动，以下是三种比较常用的将逻辑斯蒂回归用于多分类的方法：
3.1 One vs One
OvO 的方法就是将多个类别中抽出来两个类别，然后将对应的样本输入到一个逻辑斯蒂回归的模型中，学到一个对这两个类别的分类器，然后重复以上的步骤，直到所有类别两两之间都存在一个分类器。
假设存在四个类别，那么分类器的数量为6个，表格如下：
0
1
2
3
0
0 vs 1
0 vs 2
0 vs 3
1
1 vs 2
1 vs 3
2
2 vs 3
3
分类器的数量等于 C^k_2 就可以了，k 代表类别的数量。
在预测时，需要运行每一个模型，然后记录每个分类器的预测结果，也就是每个分类器都进行一次投票，取获得票数最多的那个类别就是最终的多分类的结果。
比如在以上的例子中，6个分类器有3个投票给了类别3，1个投票给了类别2，1个投票给类别1，最后一个投票给类别0，那么久取类别3为最终预测结果。
OvO 的方法中，当需要预测的类别变得很多的时候，那么我们需要进行训练的分类器也变得很多了，这一方面提高了训练开销，但在另一方面，每一个训练器中，因为只需要输入两个类别对应的训练样本即可，这样就又减少了开销。
从预测的角度考虑，这种方式需要运行的分类器非常多，而无法降低每个分类器的预测时间复杂度，因此预测的开销较大。
代码实现：machine-learning-notesandcode/One_VS_One.py at master · Zhouxiaonnan/machine-learning-notesandcode (github.com)
3.2 One vs All
OvA 的方法就是从所有类别中依次选择一个类别作为1，其他所有类别作为0，来训练分类器，因此分类器的数量要比 OvO 的数量少得多。
作为1的类别
作为0的类别
0
1；2；3
1
0；2；3
2
0；1；3
3
0；1；2
通过以上例子可以看到，分类器的数量实际上就是类别的数量，也就是k。
虽然分类器的数量下降了，但是对于每一个分类器来说，训练时需要将所有的训练数据全部输入进去进行训练，因此每一个分类器的训练时间复杂度是高于 OvO 的。
从预测的方面来说，因为分类器的数量较少，而每个分类器的预测时间复杂度不变，因此总体的预测时间复杂度小于 OvA。
预测结果的确定，是根据每个分类器对其对应的类别1的概率进行排序，选择概率最高的那个类别作为最终的预测类别。
代码实现：machine-learning-notesandcode/One_VS_All.py at master · Zhouxiaonnan/machine-learning-notesandcode (github.com)
3.3 从 sigmoid 到 softmax 的推导
第三种方式，我们可以直接从数学上使用 softmax 函数来得到最终的结果，而 softmax 函数与 sigmoid 函数有着密不可分的关系，它是 sigmoid 函数的更一般化的表示，而 sigmoid 函数是 softmax 函数的一个特殊情况。
我们知道 logit 函数代表的是某件事发生的几率，其形式为：
logit(P) = log\frac{P}{1-P} = wx\\
分子代表的是一件事发生的概率，分母代表这件事以外的事发生的概率，两者的和为1。
当我们面对的情况是多个分类时，可以让 k-1 个类别分别对剩下的那个类别做回归，即得到 k-1 个 logit 公式：
log\frac{P(Y=c_i|x)}{P(Y=c_k|x)} = w_ix (i=1,2,...,k-1)\\
然后对这些公式稍微变个型，可得：
P(Y=c_i|x) = P(Y=c_k|x)exp(w_ix) (i=1,2,...,k-1)\\
由于我们知道所有类别的可能性相加为1，因此可以得到：
P(Y=c_k|x) = 1-\sum_{i=1}^{k-1}P(Y=c_i|x)\\  = 1-\sum_{i=1}^{k-1}P(Y=c_k|x)exp(w_ix)\\
通过解上面的方程，可以得到关于某个样本被分类到类别 c_k 的概率：
P(Y=c_k|x) = 1-\sum_{i=1}^{k-1}P(Y=c_k|x)exp(w_ix)\\ P(Y=c_k|x)+\sum_{i=1}^{k-1}P(Y=c_k|x)exp(w_ix)=1\\ P(Y=c_k|x)+P(Y=c_k|x)\sum_{i=1}^{k-1}exp(w_ix)=1\\ P(Y=c_k|x)=\frac{1}{1+\sum_{i=1}^{k-1}exp(w_ix)}
这就是我们所了解的 softmax 函数了。
4. 逻辑斯蒂回归的并行化计算方法
逻辑斯蒂回归作为一个简单但经典的模型，在深度学习大行其道的现今，仍然活跃在各大应用场景之中，而推荐系统首当其冲，其中一个重要原因就是，推荐系统的特征数量动不动就达到几百万，几千万，甚至上亿，上十亿的级别（因为会对用户 id 和物品 id 进行 one-hot 编码，有时候还需要做特征交叉），同时样本数量相比于特征来说更大，因为每个用户的每个行为都会产生样品，而在业务中，又恰恰要求推荐系统能够做到在线学习，可以非常快速地更新迭代，于是计算量少而快的逻辑斯蒂回归就是最好的选择之一。
在工程上，我们还可以对逻辑斯蒂回归进行优化，让其运算速度变得更快，这个方法也常常运用到大量的模型算法之中，也就是并行运算。
逻辑斯蒂回归的损失函数为交叉熵：
logL(w)= \sum_{i=1}^{N}[y_{i} \log P(x_{i})+(1-y_{i}) \log (1-P(x_{i}))]\\
优化方法为梯度下降法：
\frac{\partial L(w)}{\partial w}=\frac{1}{n}\sum_{i=1}^{n}(y_{i} x_{i}-\frac{x_i}{1+e^{-w \cdot x_{i}}})\\
首先需要说明的是，为什么不用随机梯度下降法，而要用 batch，这是因为随机梯度下降法虽然收敛速度较快，但是收敛方向比较随机，另外会在最低点来回跳动，造成无法继续收敛的情况，也就是收敛性能不好，关于这一问题可以挖一个坑之后再填。
回到主题，我们在考虑并行化的时候，首先需要先研究一下梯度的公式，可以看到梯度公式中， y_i 和 x_i 是固定值， w 和 x_i 之间是点乘的关系，也就是对应位置相乘再相加，最后是将不同样本的梯度相加再求一个平均。
于是，在工程上，我们可以用三种方式进行并行化：
仅按照样本划分
仅按照特征划分
按照样本和特征划分
4.1 仅按照样本划分
显然，首先第一个想到的肯定是可以在样本的层次上进行拆分，对每一个分类错误的样本的计算进行并行化，然后将最终的结果相加再平均即可。
从公式上看，可以这样表示（其中计算 \rho 的部分为并行部分）：
\rho_1 = y_{1} x_{1}-\frac{x_1}{1+e^{-w \cdot x_{1}}}\\ \rho_2 = y_{2} x_{2}-\frac{x_2}{1+e^{-w \cdot x_{2}}}\\ \rho_3 = y_{3} x_{3}-\frac{x_3}{1+e^{-w \cdot x_{3}}}\\ ...\\ \rho_n = y_{n} x_{n}-\frac{x_n}{1+e^{-w \cdot x_{n}}}\\ \frac{\partial L(w)}{\partial w}=\frac{1}{n}\sum_{i=1}^n \rho_i
当然在实际情况中，还需要考虑样本数量以及特征数量来确定最佳的并行数量，比如如果样本量较少而特征较多，则每一个并行任务中仅计算一个样本就能达到较高的效率，而如果样本较多但特征数量少，则需要每一个并行任务中多分配几个样本，这是因为启动线程或进程也需要时间，如果单个并行任务中的计算时间比启动任务的时间还少，那么这种情况下使用并行方法反而是增加了时间消耗的。
代码实现：machine-learning-notesandcode/按样本划分的并行化.py at master · Zhouxiaonnan/machine-learning-notesandcode (github.com)
4.2 仅按照特征划分
如果仅按照特征进行划分，则拆分的层次就在于计算 w·x 这一步了，这样不像按样本划分那么直观，可能会有些不太好理解，且听我解释。
现在将 w·x 这一步单独提取出来，我们发现实际上就是每个对应位置的元素相乘，最后相加： \sum^{n}_{i=1}w_ix
假设我们有三个样本和三个特征，那么可以组成一个3*3的矩阵：
w1
w2
w3
x1
w1x1
w2x1
w3x1
x2
w1x2
w2x2
w3x2
x3
w1x3
w2x3
w3x3
按照特征划分，就是按照列划分，在上面的例子中，就是一个并行任务中，计算 w1x1, w1x2, w1x3，第二个并行任务中计算 w2x1, w2x2, w2x3，第三个并行任务中计算 w3x1, w3x2, w3x3。
但不同于按照样本划分，此时我们每个并行任务中，计算的是不同样本与单个特征的乘积，而在梯度公式中，需要将同一个样本与不同特征的乘积相加：
​

编辑

切换为居中
添加图片注释，不超过 140 字（可选）
那么我们就不能在并行任务中直接相加，而是要将计算结果保存起来，待所有并行任务计算完毕后，将所有结果集合成一个大表，再根据样本的维度进行加和，最终计算得到梯度。
所以这种并行方法中，并行的部分实际上是下图中红框的部分，其他部分都需要在并行化之外计算。
​

编辑
添加图片注释，不超过 140 字（可选）
在实际情况中，这个方法与按样本划分相反，如果样本量较少而特征较多，则每一个并行任务中需要传入多个特征进行计算才能达到节省时间的目的，而如果样本较多但特征数量少，那么每一个任务分配一个特征进行计算即可。
从另一个角度来看，由于需要保存整个大表的计算结果，且加和运算需要放在并行化之外，因此内存和时间的开销相较而言更大一些。另外，需要计算梯度的样本仅为分类错误的样本，因此我们需要先得到分类错误的样本，这也需要放在并行计算之外。
代码实现：machine-learning-notesandcode/按特征划分的并行化.py at master · Zhouxiaonnan/machine-learning-notesandcode (github.com)
4.3 按照样本和特征划分
按照样本和特征划分实际上与按照特征划分是一样的思路，只不过是现在在两个维度上对样本特征矩阵进行拆分，同样在计算完成后需要保存计算结果，并拼接成一个完整的矩阵后，再计算梯度。
四个样本和四个特征，那么可以组成一个4*4的矩阵，并对其进行划分
​

编辑

切换为居中
添加图片注释，不超过 140 字（可选）
5. 其他问题
5.1 为什么逻辑斯蒂回归适合稀疏矩阵
逻辑斯蒂回归或者其变体常用于推荐系统中，而推荐系统中的特征矩阵往往是稀疏的，这不是什么巧合，而是其中存在着内在的逻辑。
比如对特征进行编码，分桶和交叉后，特征矩阵变得稀疏了，但这样的特征矩阵更符合现实情况，或者增加了非线性，同时也增加了鲁棒性。
而稀疏矩阵用在逻辑斯蒂回归上，可以大大减少时间复杂度，比如对元素为0的部分，可以直接忽略其乘法运算，并且通过一些方式，也可以仅仅存储不等于0的元素，大大减少空间复杂度。
因此并非是说逻辑斯蒂回归适合稀疏矩阵，而是考虑到现实情境，为了增加非线性，导致了矩阵为稀疏的，反过来，因为逻辑斯蒂回归的特性，特征矩阵即使是很大且稀疏的，也可以快速运算。
5.1.1 One-hot 编码
对于推荐系统，往往是基于单个用户来进行推荐的，而在特征中，如何表示单个用户呢？很简单，使用用户的ID，这往往就是一串数字。但一个很明显的问题是，数字是存在大小关系的，但用户ID之间并不存在大小关系，因此与推荐结果并不存在线性关系，于是我们对其采用 one-hot 编码，这样，当用户数量增多，特征矩阵中的用户部分，就立刻变得异常稀疏了。
​

编辑
对用户 IＤ 进行 One-hot 编码
5.1.2 分桶
与将用户 ID 进行 One-hot 编码类似的理由，一些特征虽然本身存在大小关系，比如用户年龄，观看视频数量，或者注册至今时长等等，但这种特征与推荐结果往往也不存在线性关系，因此我们需要对其进行分桶，来引入非线性，而分桶之后，又会增加特征矩阵的稀疏性。
​

编辑

切换为居中
对注册年数进行分桶
5.1.3 特征交叉
同样考虑现实情况，有一些特征可能它们本身与推荐结果没有关联，但它们的组合却对推荐结果存在关联，比如仅仅通过“18-25岁“，或者“观看视频活跃时间在下午2-5点“这两个特征没法判断这个人是一个学生，这个年龄也可能是上班族，观看视频活跃时间在下午2-5点也可能是老年人，但是“18-25岁且观看视频活跃时间在下午2-5点”的人则很有可能是一个大学生。
当用来交叉的特征本身是稀疏的时候，那么交叉出来的特征也会是稀疏的，而稀疏性与特征的交叉方式有关，如果是相乘（和的关系），则稀疏性可能会增加，如果是相加（或的关系），则稀疏性可能会下降（但一般而言相乘用的比较多，依据现实情况而定）。
​

编辑

切换为居中
特征交叉：特征1 * 特征2
5.1.4 鲁棒性
鲁棒性可以分为三个层面：
要求模型的精度较高
要求噪声对模型的影响较小
要求离群点不能对模型产生较大影响
那么当我们对特征数据进行分桶后，会发现离群点（异常值）也被分到了最左或者最右边的桶中，在上述第三点上增加了模型的鲁棒性。
​

编辑

切换为居中
150岁的离群点被分配到&amp;amp;gt;=60岁中，因此离群点对模型产生的影响较小，增加了模型的鲁棒性
5.2 稀疏矩阵如何实现排零存储和排零计算
所谓稀疏矩阵，简单来说，就是矩阵中为0的元素比例较高的矩阵，由于0不携带信息，因此耗费空间存储0元素是很浪费资源的行为，特别是在矩阵极其巨大的情况下。
为了解决这个问题，在工程中可以对稀疏矩阵的存储和计算都做一些优化，减少空间复杂度和时间复杂度。
5.2.1 COO（Coordinate Format）
顾名思义，以坐标的形式存储非0元素，更常见的说法是三元组，即将非0元素转换为 (x_i, y_j, a_{ij}) 的形式，其中 x_i 即横坐标， y_j 即纵坐标， a_{ij} 即元素的值。
COO 仅存储稀疏矩阵中非0元素的坐标和元素值，虽然每一个非0元素的存储增加了，但由于减少了0元素的存储，因此整体的空间复杂度还是降低的。
​

编辑

切换为居中
添加图片注释，不超过 140 字（可选）
5.2.2 CSR（Compressed Sparse Row Format）
当我们对 COO 矩阵的行坐标进行排序后，会发现有一些元素的行坐标之间是重复的，这时候我们可不可以对 COO 矩阵进行更进一步的简化呢？
答案是可以的，我们保持 COO 矩阵中的行坐标的顺序排列，然后仅记录每一组相同行坐标的第一个值在 COO 矩阵中的索引即可，如果遇到不存在的，就重复前一个的索引。
乍看之下很难明白，画个图就明白了：
​

编辑

切换为居中
添加图片注释，不超过 140 字（可选）
其中 x_index 就是行坐标的第一个值在 COO 矩阵中的索引，当我们需要拿取第一行的元素时，在 x_index 中取出前两个数字，即(0,2)，这告诉了我们行坐标为0，以及最右边矩阵中要找的元素的存储位置，即其中的第0行到第1行，这样就取出了原矩阵第一行所有元素的列和元素值，有了行坐标，列坐标和元素值，也就取出了我们要找的元素了。
如果是要找稀疏矩阵中的第二行的元素，即x1的元素，那么只需要在 x_index 中找到第二个和第三个数字，即(2,4)，这告诉了我们列坐标和元素值的存储位置为第2行到第3行，如此取出列坐标和元素值，即得到了稀疏矩阵中第二行非零元素的所有信息。
当然，我们也可以对COO的y列进行排序，然后根据y的值进一步压缩，方法相同，就不再赘述。
以上两种方法是比较常用的稀疏矩阵存储方式，在机器学习计算中，因为存储的是特征矩阵，绝大多数的特征数值是固定的，很少需要对某个数值做修改，因此计算中顺序读取并计算即可。
对于一些特殊的矩阵形式，比如非0元素以对角线的形式排列，就会存在一些特殊的压缩存储方式（比如DIA），但由于特征矩阵基本很少以这种形式存在，这里也就不再展开讨论了。
欢迎关注专栏：目前在这个专栏中已经原创了60+篇文章啦，主要是机器学习学习笔记和代码复现，面试题
欢迎关注专栏：目前在这个专栏中已经原创了超过60篇文章啦，主要《统计学习方法》学习笔记和代码复现，面试题解答，MySQL技巧等，欢迎关注！
学习笔记：数据分析，机器学习，深度学习​zhuanlan.zhihu.com/c_1274454587772915712​zhuanlan.zhihu.com/c_1274454587772915712
​
zhuanlan.zhihu.com
图标
其他面试题解答：
面试题解答1：为什么线性回归要求假设因变量符合正态分布 - 知乎 (zhihu.com)
面试题解答2：各种回归模型与广义线性模型的关系 - 知乎 (zhihu.com)
面试题解答3：如何用方差膨胀因子判断多重共线性 - 知乎 (zhihu.com)
面试题解答4：逻辑斯蒂回归是否可以使用其他的函数替代 sigmoid 函数 - 知乎 (zhihu.com)
面试题解答5：特征存在多重共线性，有哪些解决方法？ - 知乎 (zhihu.com)
转行相关：
舟晓南：如何转行和学习数据分析 | 工科生三个月成功转行数据分析心得浅谈
舟晓南：求职数据分析师岗位，简历应该如何写？｜工科生三个月成功转行数据分析心得浅谈
数据分析，机器学习社群正式启动~
需要学习资料，想要加入社群均可私信~
在这里我会不定期分享各种数据分析相关资源，技能学习技巧和经验等等~
详情私信，一起进步吧！

写于上海 2022-01-31
