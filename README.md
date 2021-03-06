# 激活函数 目标函数 损失函数 优化方法总结
## 1.1 激活函数
关于激活函数，首先要搞清楚的问题是，激活函数是什么，有什么用？不用激活函数可不可以？答案是不可以。激活函数的主要作用是提供网络的非线性建模能力。如果没有


激活函数，那么该网络仅能够表达线性映射，此时即便有再多的隐藏层，其整个网络跟单层神经网络也是等价的。因此也可以认为，只有加入了激活函数之后，深度神经网


络才具备了分层的非线性映射学习能力。 那么激活函数应该具有什么样的性质呢？


　　可微性： 当优化方法是基于梯度的时候，这个性质是必须的。 
  
  
　　单调性： 当激活函数是单调的时候，单层网络能够保证是凸函数。 
  
  
　　输出值的范围： 当激活函数输出值是 有限 的时候，基于梯度的优化方法会更加 稳定，因为特征的表示受有限权值的影响更显著;当激活函数的输出是 无限 的时候，
  
  
  模型的训练会更加高效，不过在这种情况小，一般需要更小的learning rate
  
  
　　从目前来看，常见的激活函数多是分段线性和具有指数形状的非线性函数
  
  ### 1.1.1 sigmoid
  
　　sigmoid 是使用范围最广的一类激活函数，具有指数函数形状，它在物理意义上最为接近生物神经元。此外，(0, 1) 的输出还可以被表示作概率，或用于输入的归一
  
  化，代表性的如Sigmoid交叉熵损失函数。
  
  
　　然而，sigmoid也有其自身的缺陷，最明显的就是饱和性。从上图可以看到，其两侧导数逐渐趋近于0 
  
  具有这种性质的称为软饱和激活函数。具体的，饱和又可分为左饱和与右饱和。与软饱和对应的是硬饱和, 即 
  
　　f′(x)=0，当|x|>c，其中c为常数。
  
　　sigmoid 的软饱和性，使得深度神经网络在二三十年里一直难以有效的训练，是阻碍神经网络发展的重要原因。具体来说，由于在后向传递过程中，sigmoid向下传导
  
  的梯度包含了一个 f′(x) 因子（sigmoid关于输入的导数），因此一旦输入落入饱和区，f′(x) 就会变得接近于0，导致了向底层传递的梯度也变得非常小。此时，网
  
  络参数很难得到有效训练。这种现象被称为梯度消失。一般来说， sigmoid 网络在 5 层之内就会产生梯度消失现象
  
  
　　此外，sigmoid函数的输出均大于0，使得输出不是0均值，这称为偏移现象，这会导致后一层的神经元将得到上一层输出的非0均值的信号作为输入。
  
  
### 1.1.2 tanh

　　tanh也是一种非常常见的激活函数。与sigmoid相比，它的输出均值是0，使得其收敛速度要比sigmoid快，减少迭代次数。然而，从途中可以看出，tanh一样具有软
  
  饱和性，从而造成梯度消失。
  
  
### 1.1.3 ReLU，P-ReLU, Leaky-ReLU
  
 
　　ReLU的全称是Rectified Linear Units，是一种后来才出现的激活函数。 可以看到，当x<0时，ReLU硬饱和，而当x>0时，则不存在饱和问题。所以，ReLU 能够在
  
  x>0时保持梯度不衰减，从而缓解梯度消失问题。这让我们能够直接以监督的方式训练深度神经网络，而无需依赖无监督的逐层预训练。
  
  
　　然而，随着训练的推进，部分输入会落入硬饱和区，导致对应权重无法更新。这种现象被称为“神经元死亡”。与sigmoid类似，ReLU的输出均值也大于0，偏移现象和
  
  神经元死亡会共同影响网络的收敛性。
  
  
　　针对在x<0的硬饱和问题，我们对ReLU做出相应的改进，使得 

　　这就是Leaky-ReLU, 而P-ReLU认为，α也可以作为一个参数来学习，原文献建议初始化a为0.25，不采用正则。
  
  
### 1.1.4 ELU
  

　　融合了sigmoid和ReLU，左侧具有软饱和性，右侧无饱和性。右侧线性部分使得ELU能够缓解梯度消失，而左侧软饱能够让ELU对输入变化或噪声更鲁棒。ELU的输出均
  
  值接近于零，所以收敛速度更快。在 ImageNet上，不加 Batch Normalization 30 层以上的 ReLU 网络会无法收敛，PReLU网络在MSRA的Fan-in （caffe ）初始
  
  
  化下会发散，而 ELU 网络在Fan-in/Fan-out下都能收敛
 
### 1.1.5 Maxout
  
　　在我看来，这个激活函数有点大一统的感觉，因为maxout网络能够近似任意连续函数，且当w2,b2,…,wn,bn为0时，退化为ReLU。Maxout能够缓解梯度消失，同时又
  
  规避了ReLU神经元死亡的缺点，但增加了参数和计算量。
 
________________________________________
 
## 2.1 损失函数
　　损失函数（loss function）是用来估量模型的预测值f(x)与真实值Y的不一致程度，它是一个非负实值函数,通常使用L(Y, f(x))来表示，损失函数越小，模型的鲁
  
  棒性就越好。损失函数是经验风险函数的核心部分，也是结构风险函数重要组成部分。模型的结构风险函数包括了经验风险项和正则项，通常可以表示成如下式子：
  
 
　　其中，前面的均值函数表示的是经验风险函数，L代表的是损失函数，后面的Φ是正则化项（regularizer）或者叫惩罚项（penalty term），它可以是L1，也可以是
  
  L2，或者其他的正则函数。整个式子表示的意思是找到使目标函数最小时的θ值。下面主要列出几种常见的损失函数。
  
  
理解：损失函数旨在表示出logit和label的差异程度，不同的损失函数有不同的表示意义，也就是在最小化损失函数过程中，logit逼近label的方式不同，得到的结果可

能也不同。


一般情况下，softmax和sigmoid使用交叉熵损失（logloss），hingeloss是SVM推导出的，hingeloss的输入使用原始logit即可。


### 2.1.1 LogLoss对数损失函数（逻辑回归，交叉熵损失）


　　有些人可能觉得逻辑回归的损失函数就是平方损失，其实并不是。平方损失函数可以通过线性回归在假设样本是高斯分布的条件下推导得到，而逻辑回归得到的并不是
  
  平方损失。在逻辑回归的推导中，它假设样本服从伯努利分布（0-1分布），然后求得满足该分布的似然函数，接着取对数求极值等等。而逻辑回归并没有求似然函数的
  
  极值，而是把极大化当做是一种思想，进而推导出它的经验风险函数为：最小化负的似然函数（即max F(y, f(x)) —> min -F(y, f(x)))。从损失函数的视角来看，
  
  它就成了log损失函数了。
  
  
log损失函数的标准形式：
 
　　刚刚说到，取对数是为了方便计算极大似然估计，因为在MLE（最大似然估计）中，直接求导比较困难，所以通常都是先取对数再求导找极值点。
  
  损失函数L(Y, P(Y|X))表达的是样本X在分类Y的情况下，使概率P(Y|X)达到最大值（换言之，就是利用已知的样本分布，找到最有可能（即最大概率）导致这种分布的
  
  参数值；或者说什么样的参数才能使我们观测到目前这组数据的概率最大）。因为log函数是单调递增的，所以logP(Y|X)也会达到最大值，因此在前面加上负号之后，
  
  最大化P(Y|X)就等价于最小化L了。
  
  
　　逻辑回归的P(Y=y|x)表达式如下（为了将类别标签y统一为1和0，下面将表达式分开表示）：
  
 
　　将它带入到上式，通过推导可以得到logistic的损失函数表达式，如下：
  
 
　　逻辑回归最后得到的目标式子如下：
  
 
　　上面是针对二分类而言的。这里需要解释一下：之所以有人认为逻辑回归是平方损失，是因为在使用梯度下降来求最优解的时候，它的迭代式子与平方损失求导后的式
  
  子非常相似，从而给人一种直观上的错觉。
  
  
这里有个PDF可以参考一下：Lecture 6: logistic regression.pdf.


注意：softmax使用的即为交叉熵损失函数，binary_cossentropy为二分类交叉熵损失，categorical_crossentropy为多分类交叉熵损失，当使用多分类交叉熵损失函

数时，标签应该为多分类模式，即使用one-hot编码的向量。


### 2.1.2平方损失函数（最小二乘法, Ordinary Least Squares ）


　　最小二乘法是线性回归的一种，最小二乘法（OLS）将问题转化成了一个凸优化问题。在线性回归中，它假设样本和噪声都服从高斯分布（为什么假设成高斯分布呢？
  
  其实这里隐藏了一个小知识点，就是中心极限定理，可以参考【central limit theorem】），最后通过极大似然估计（MLE）可以推导出最小二乘式子。最小二乘的
  
  基本原则是：最优拟合直线应该是使各点到回归直线的距离和最小的直线，即平方和最小。换言之，OLS是基于距离的，而这个距离就是我们用的最多的欧几里得距离。
  
  为什么它会选择使用欧式距离作为误差度量呢（即Mean squared error， MSE），主要有以下几个原因：
  
  
•	简单，计算方便；


•	欧氏距离是一种很好的相似性度量标准；


•	在不同的表示域变换后特征性质不变。


平方损失（Square loss）的标准形式如下：

 
当样本个数为n时，此时的损失函数变为：
 
Y-f(X)表示的是残差，整个式子表示的是残差的平方和，而我们的目的就是最小化这个目标函数值（注：该式子未加入正则项），也就是最小化残差的平方和（residual


sum of squares，RSS）。


而在实际应用中，通常会使用均方差（MSE）作为一项衡量指标，公式如下：

 
上面提到了线性回归，这里额外补充一句，我们通常说的线性有两种情况，一种是因变量y是自变量x的线性函数，一种是因变量y是参数α的线性函数。在机器学习中，通常

指的都是后一种情况。


### 2.1.3 指数损失函数（Adaboost）


学过Adaboost算法的人都知道，它是前向分步加法算法的特例，是一个加和模型，损失函数就是指数函数。在Adaboost中，经过m此迭代之后，可以得到fm(x):
 
Adaboost每次迭代时的目的是为了找到最小化下列式子时的参数α 和G：
 
而指数损失函数（exp-loss）的标准形式如下
 
可以看出，Adaboost的目标式子就是指数损失，在给定n个样本的情况下，Adaboost的损失函数为：
 
关于Adaboost的推导，可以参考Wikipedia：AdaBoost或者《统计学习方法》P145.


### 2.1.4 Hinge损失函数（SVM）


在机器学习算法中，hinge损失函数和SVM是息息相关的。在线性支持向量机中，最优化问题可以等价于下列式子：

 
下面来对式子做个变形，令：
 
于是，原式就变成了：
 
如若取λ=1/(2C)，式子就可以表示成： 
 
可以看出，该式子与下式非常相似：
 
前半部分中的 l 就是hinge损失函数，而后面相当于L2正则项。


Hinge 损失函数的标准形式

 
可以看出，当|y|>=1时，L(y)=0。


更多内容，参考Hinge-loss。


补充一下：在libsvm中一共有4中核函数可以选择，对应的是-t参数分别是：


•	0-线性核；


•	1-多项式核；


•	2-RBF核；


•	3-sigmoid核。


### 2.1.5 其它损失函数


除了以上这几种损失函数，常用的还有：

0-1损失函数

 
绝对值损失函数

 
下面来看看几种损失函数的可视化图像，对着图看看横坐标，看看纵坐标，再看看每条线都表示什么损失函数，多看几次好好消化消化。

 
### 2.1.6 Keras / TensorFlow 中常用 Cost Function 总结


•	mean_squared_error或mse 


•	mean_absolute_error或mae 


•	mean_absolute_percentage_error或mape 


•	mean_squared_logarithmic_error或msle


•	squared_hinge 


•	hinge 


•	categorical_hinge 


•	binary_crossentropy（亦称作对数损失，logloss） 


•	logcosh 


•	categorical_crossentropy：亦称作多类的对数损失，注意使用该目标函数时，需要将标签转化为形如(nb_samples, nb_classes)的二值序列


•	sparse_categorical_crossentrop：如上，但接受稀疏标签。注意，使用该函数时仍然需要你的标签与输出值的维度相同，你可能需要在标签数据上增加一个维度：

np.expand_dims(y,-1) 


•	kullback_leibler_divergence:从预测值概率分布Q到真值概率分布P的信息增益,用以度量两个分布的差异. 


•	poisson：即(predictions - targets * log(predictions))的均值 


•	cosine_proximity：即预测值与真实标签的余弦距离平均值的相反数 


　　需要记住的是：参数越多，模型越复杂，而越复杂的模型越容易过拟合。过拟合就是说模型在训练数据上的效果远远好于在测试集上的性能。此时可以考虑正则化，通
  
  过设置正则项前面的hyper parameter，来权衡损失函数和正则项，减小参数规模，达到模型简化的目的，从而使模型具有更好的泛化能力。
  


## 3.1优化方法 
　　主要是一阶的梯度法，包括SGD, Momentum, Nesterov Momentum, AdaGrad, RMSProp, Adam。 其中SGD,Momentum,Nesterov Momentum是手动指定学习速率
  
  的,而后面的AdaGrad, RMSProp, Adam,就能够自动调节学习速率.
  
  
### 3.1.1BGD
　　即batch gradient descent. 在训练中,每一步迭代都使用训练集的所有内容. 也就是说,利用现有参数对训练集中的每一个输入生成一个估计输出yi^,然后跟实际
  
  输出yi比较,统计所有误差,求平均以后得到平均误差,以此来作为更新参数的依据.
  
  
　　具体实现: 
  
  
　　需要:学习速率 ϵ, 初始参数 θ 
  
  
　　每步迭代过程: 
  
  
　　　　1. 提取训练集中的所有内容{x1,…,xn},以及相关的输出yi 
    
    
　　　　2. 计算梯度和误差并更新参数: 
    

 
　　优点: 
  
  
　　由于每一步都利用了训练集中的所有数据,因此当损失函数达到最小值以后,能够保证此时计算出的梯度为0,换句话说,就是能够收敛.因此,使用BGD时不需要逐渐减小
  
  学习速率ϵk
  
  
　　缺点: 
  
  
　　由于每一步都要使用所有数据,因此随着数据集的增大,运行速度会越来越慢.
  
  
### 3.1.2 SGD
　　SGD全名 stochastic gradient descent， 即随机梯度下降。不过这里的SGD其实跟MBGD(minibatch gradient descent)是一个意思,即随机抽取一批样本,以此
  
  为根据来更新参数.
  
  
　　具体实现: 
  
  
　　需要:学习速率 ϵ, 初始参数 θ 
  
  
　　每步迭代过程: 
  
  
　　　　1. 从训练集中的随机抽取一批容量为m的样本{x1,…,xm},以及相关的输出yi 
    
    
　　　　2. 计算梯度和误差并更新参数: 
    
    

 
　　优点: 
  
  
　　　　训练速度快,对于很大的数据集,也能够以较快的速度收敛.
    
    
　　缺点: 
  
　　　　由于是抽取,因此不可避免的,得到的梯度肯定有误差.因此学习速率需要逐渐减小.否则模型无法收敛 
    
    
　　　　因为误差,所以每一次迭代的梯度受抽样的影响比较大,也就是说梯度含有比较大的噪声,不能很好的反映真实梯度.
    
    
　　学习速率该如何调整: 
  
  
　　　　那么这样一来,ϵ如何衰减就成了问题.如果要保证SGD收敛,应该满足如下两个要求: 
    
    

 
　　而在实际操作中,一般是进行线性衰减: 
  
  
 
　　其中ϵ0是初始学习率, ϵτ是最后一次迭代的学习率. τ自然代表迭代次数.一般来说,ϵτ 设为ϵ0的1%比较合适.而τ一般设为让训练集中的每个数据都输入模型上百次比
  
  较合适.那么初始学习率ϵ0怎么设置呢?书上说,你先用固定的学习速率迭代100次,找出效果最好的学习速率,然后ϵ0设为比它大一点就可以了.
  
  
 
________________________________________
### 3.1.3 Momentum
　　上面的SGD有个问题,就是每次迭代计算的梯度含有比较大的噪音. 而Momentum方法可以比较好的缓解这个问题,尤其是在面对小而连续的梯度但是含有很多噪声的时
  
  候,可以很好的加速学习.Momentum借用了物理中的动量概念,即前几次的梯度也会参与运算.为了表示动量,引入了一个新的变量v(velocity).v是之前的梯度的累加,但
  
  是每回合都有一定的衰减.
  
  
　　具体实现: 
  
  
　　需要:学习速率 ϵ, 初始参数 θ, 初始速率v, 动量衰减参数α 
  
  
　　每步迭代过程: 
  
  
　　　　1. 从训练集中的随机抽取一批容量为m的样本{x1,…,xm},以及相关的输出yi 
    
    
　　　　2. 计算梯度和误差,并更新速度v和参数θ: 
    
    

 
　　其中参数α表示每回合速率v的衰减程度.同时也可以推断得到,如果每次迭代得到的梯度都是g,那么最后得到的v的稳定值为 
  
  
 
　　也就是说,Momentum最好情况下能够将学习速率加速11−α倍.一般α的取值有0.5,0.9,0.99这几种.当然,也可以让α的值随着时间而变化,一开始小点,后来再加大.不过
  
  这样一来,又会引进新的参数.
  
  
　　特点: 
  
 
　　前后梯度方向一致时,能够加速学习 
  
  
　　前后梯度方向不一致时,能够抑制震荡
  
  
________________________________________
### 3.1.4 Nesterov Momentum
　　这是对之前的Momentum的一种改进,大概思路就是,先对参数进行估计,然后使用估计后的参数来计算误差
  
  
　　具体实现: 
  
  
　　需要:学习速率 ϵ, 初始参数 θ, 初始速率v, 动量衰减参数α 
  
  
　　每步迭代过程: 
  
  
　　　　1. 从训练集中的随机抽取一批容量为m的样本{x1,…,xm},以及相关的输出yi 
    
    
　　　　2. 计算梯度和误差,并更新速度v和参数θ: 
    

 
 
注意在估算g^的时候,参数变成了θ+αv而不是之前的θ

 
________________________________________


### 3.1.5 AdaGrad


AdaGrad可以自动变更学习速率,只是需要设定一个全局的学习速率ϵ,但是这并非是实际学习速率,实际的速率是与以往参数的模之和的开方成反比的. Adagrad 亦称为自

适应梯度（adaptive gradient），允许学习率基于参数进行调整，而不需要在学习过程中人为调整学习率。Adagrad 根据不常用的参数进行较大幅度的学习率更新，根

据常用的参数进行较小幅度的学习率更新。因此，Adagrad 成了稀疏数据如图像识别和 NLP 的天然选择。然而 Adagrad 的最大问题在于，在某些案例中，学习率变得太

小，学习率单调下降使得网络停止学习过程。在经典的动量算法和 Nesterov 中，加速梯度参数更新是对所有参数进行的，并且学习过程中的学习率保持不变。在 

Adagrad 中，每次迭代中每个参数使用的都是不同的学习率。


也许说起来有点绕口,不过用公式来表示就直白的多: 

 
　　其中δ是一个很小的常亮,大概在10−7,防止出现除以0的情况.
  
  
　　具体实现: 
  
  
　　需要:全局学习速率 ϵ, 初始参数 θ, 数值稳定量δ 
  
  
　　中间变量: 梯度累计量r(初始化为0) 
  
  
　　每步迭代过程: 
  
  
　　　　1. 从训练集中的随机抽取一批容量为m的样本{x1,…,xm},以及相关的输出yi 
    
    
　　　　2. 计算梯度和误差,更新r,再根据r和梯度计算参数更新量 
    

 
　　优点: 
  
  
　　　　能够实现学习率的自动更改。如果这次梯度大,那么学习速率衰减的就快一些;如果这次梯度小,那么学习速率衰减的就满一些。
    
　　缺点: 
  
　　　　任然要设置一个变量ϵ 
    
　　　　经验表明，在普通算法中也许效果不错，但在深度学习中，深度过深时会造成训练提前结束。
    
________________________________________


### 3.1.6 RMSProp


　　RMSProp通过引入一个衰减系数，让r每回合都衰减一定比例，类似于Momentum中的做法。
  
  
　　具体实现: 
  
  
　　需要:全局学习速率 ϵ, 初始参数 θ, 数值稳定量δ，衰减速率ρ 
  
  
　　中间变量: 梯度累计量r(初始化为0) 
  
  
　　每步迭代过程: 
  
  
　　　　1. 从训练集中的随机抽取一批容量为m的样本{x1,…,xm},以及相关的输出yi 
    
    
　　　　2. 计算梯度和误差,更新r,再根据r和梯度计算参数更新量 

 
　　优点： 
  
　　　　相比于AdaGrad,这种方法很好的解决了深度学习中过早结束的问题 
    
    
　　　　适合处理非平稳目标，对于RNN效果很好
    
　　缺点： 
  
　　　　又引入了新的超参，衰减系数ρ 
    
　　　　依然依赖于全局学习速率
________________________________________


### 3.1.7 RMSProp with Nesterov Momentum

　　当然，也有将RMSProp和Nesterov Momentum结合起来的
  
  
　　具体实现: 
  
  
　　　　需要:全局学习速率 ϵ, 初始参数 θ, 初始速率v，动量衰减系数α, 梯度累计量衰减速率ρ 
    
    
　　　　中间变量: 梯度累计量r(初始化为0) 
    
    
　　每步迭代过程: 
  
  
　　　　1. 从训练集中的随机抽取一批容量为m的样本{x1,…,xm},以及相关的输出yi 
    
    
　　　　2. 计算梯度和误差,更新r,再根据r和梯度计算参数更新量 

 
________________________________________


### 3.1.8 Adam


Adam(Adaptive Moment Estimation)本质上是带有动量项的RMSprop，它利用梯度的一阶矩估计和二阶矩估计动态调整每个参数的学习率。Adam的优点主要在于经过偏

置校正后，每一次迭代学习率都有个确定范围，使得参数比较平稳。


#### Adam 优化算法的基本机制


Adam 算法和传统的随机梯度下降不同。随机梯度下降保持单一的学习率（即 alpha）更新所有的权重，学习率在训练过程中并不会改变。而 Adam 通过计算梯度的一阶

矩估计和二阶矩估计而为不同的参数设计独立的自适应性学习率。


Adam 算法的提出者描述其为两种随机梯度下降扩展式的优点集合，即：


•	适应性梯度算法（AdaGrad）为每一个参数保留一个学习率以提升在稀疏梯度（即自然语言和计算机视觉问题）上的性能。


•	均方根传播（RMSProp）基于权重梯度最近量级的均值为每一个参数适应性地保留学习率。这意味着算法在非稳态和在线问题上有很有优秀的性能。


Adam 算法同时获得了 AdaGrad 和 RMSProp 算法的优点。Adam 不仅如 RMSProp 算法那样基于一阶矩均值计算适应性参数学习率，它同时还充分利用了梯度的二阶矩


均值（即有偏方差/uncentered variance）。具体来说，算法计算了梯度的指数移动均值（exponential moving average），超参数 beta1 和 beta2 控制了这些移

动均值的衰减率。


移动均值的初始值和 beta1、beta2 值接近于 1（推荐值），因此矩估计的偏差接近于 0。该偏差通过首先计算带偏差的估计而后计算偏差修正后的估计而得到提升。


　　具体实现: 
  
  
　　　　需要:步进值 ϵ, 初始参数 θ, 数值稳定量δ，一阶动量衰减系数ρ1, 二阶动量衰减系数ρ2 
    
    
　　　　其中几个取值一般为：δ=10−8,ρ1=0.9,ρ2=0.999
    
    
　　　　中间变量：一阶动量s，二阶动量r,都初始化为0 
    
    
　　每步迭代过程: 
  
  
　　　　1. 从训练集中的随机抽取一批容量为m的样本{x1,…,xm},以及相关的输出yi
    
　　　　2. 计算梯度和误差,更新r和s,再根据r和s以及梯度计算参数更新量 

 



### 4.正则化


深度学习中的正则化与优化策略一直是非常重要的部分，它们很大程度上决定了模型的泛化与收敛等性能。本文主要以深度卷积网络为例，探讨了深度学习中的五项正则化

与七项优化策略，并重点解释了当前最为流行的 Adam 优化算法。本文主体介绍和简要分析基于南洋理工的概述论文，而 Adam 方法的具体介绍基于 14 年的 Adam 论

文。


近来在深度学习中，卷积神经网络和循环神经网络等深度模型在各种复杂的任务中表现十分优秀。例如卷积神经网络（CNN）这种由生物启发而诞生的网络，它基于数学的

卷积运算而能检测大量的图像特征，因此可用于解决多种图像视觉应用、目标分类和语音识别等问题。


但是，深层网络架构的学习要求大量数据，对计算能力的要求很高。神经元和参数之间的大量连接需要通过梯度下降及其变体以迭代的方式不断调整。此外，有些架构可能

因为强大的表征力而产生测试数据过拟合等现象。这时我们可以使用正则化和优化技术来解决这两个问题。


梯度下降是一种优化技术，它通过最小化代价函数的误差而决定参数的最优值，进而提升网络的性能。尽管梯度下降是参数优化的自然选择，但它在处理高度非凸函数和搜

索全局最小值时也存在很多局限性。


正则化技术令参数数量多于输入数据量的网络避免产生过拟合现象。正则化通过避免训练完美拟合数据样本的系数而有助于算法的泛化。为了防止过拟合，增加训练样本是

一个好的解决方案。此外，还可使用数据增强、L1 正则化、L2 正则化、Dropout、DropConnect 和早停（Early stopping）法等。


增加输入数据、数据增强、早停、dropout 及其变体是深度神经网络中常用的调整方法。本论文作为之前文章《徒手实现 CNN：综述论文详解卷积网络的数学本质 》的补

充，旨在介绍开发典型卷积神经网络框架时最常用的正则化和优化策略。


主体论文：Regularization and Optimization strategies in Deep Convolutional Neural Network


论文地址：https://arxiv.org/pdf/1712.04711.pdf


摘要：卷积神经网络（ConvNet）在一些复杂的机器学习任务中性能表现非常好。ConvNet 架构需要大量数据和参数，因此其学习过程需要消耗大量算力，向全局最小值的

收敛过程较慢，容易掉入局部极小值的陷阱导致预测结果不好。在一些案例中，ConvNet 架构与数据产生过拟合，致使架构难以泛化至新样本。为了解决这些问题，近年

来研究者开发了多种正则化和优化策略。此外，研究显示这些技术能够大幅提升网络性能，同时减少算力消耗。使用这些技术的前提是全面了解该技术提升网络表达能力的

理论原理，本论文旨在介绍开发 ConvNet 架构最常用策略的理论概念和数学公式。


#### 正则化技术


正则化技术是保证算法泛化能力的有效工具，因此算法正则化的研究成为机器学习中主要的研究主题 [9] [10]。此外，正则化还是训练参数数量大于训练数据集的深度学

习模型的关键步骤。正则化可以避免算法过拟合，过拟合通常发生在算法学习的输入数据无法反应真实的分布且存在一些噪声的情况。过去数年，研究者提出和开发了多种


适合机器学习算法的正则化方法，如数据增强、L2 正则化（权重衰减）、L1 正则化、Dropout、Drop Connect、随机池化和早停等。


除了泛化原因，奥卡姆剃刀原理和贝叶斯估计也都支持着正则化。根据奥卡姆剃刀原理，在所有可能选择的模型中，能很好解释已知数据，并且十分简单的模型才是最好的



模型。而从贝叶斯估计的角度来看，正则化项对应于模型的先验概率。


#### 4.1.2 数据增强


数据增强是提升算法性能、满足深度学习模型对大量数据的需求的重要工具。数据增强通过向训练数据添加转换或扰动来人工增加训练数据集。数据增强技术如水平或垂直


翻转图像、裁剪、色彩变换、扩展和旋转通常应用在视觉表象和图像分类中。


#### 4.2 L1 和 L2 正则化
L1 和 L2 正则化是最常用的正则化方法。L1 正则化向目标函数添加正则化项，以减少参数的绝对值总和；而 L2 正则化中，添加正则化项的目的在于减少参数平方的总

和。根据之前的研究，L1 正则化中的很多参数向量是稀疏向量，因为很多模型导致参数趋近于 0，因此它常用于特征选择设置中。机器学习中最常用的正则化方法是对权

重施加 L2 范数约束。


L1正则化可以产生稀疏模型（L1是怎么让系数等于零的），以及为什么L2正则化可以防止过拟合。

标准正则化代价函数如下：


 
其中正则化项 R(w) 是：


 
另一种惩罚权重的绝对值总和的方法是 L1 正则化：

 
L1 正则化在零点不可微，因此权重以趋近于零的常数因子增长。很多神经网络在权重衰减公式中使用一阶步骤来解决非凸 L1 正则化问题 [19]。L1 范数的近似变体是：

 
另一个正则化方法是混合 L1 和 L2 正则化，即弹性网络罚项 [20]。


在《深度学习》一书中，参数范数惩罚 L2 正则化能让深度学习算法「感知」到具有较高方差的输入 x，因此与输出目标的协方差较小（相对增加方差）的特征权重将会

收缩。而 L1 正则化会因为在方向 i 上 J(w; X, y) 对 J(w; X, y) hat 的贡献被抵消而使 w_i 的值变为 0（J(w; X, y) hat 为 J(w; X, y) 加上 L1 正则

项）。此外，参数的范数正则化也可以作为约束条件。对于 L2 范数来说，权重会被约束在一个 L2 范数的球体中，而对于 L1 范数，权重将被限制在 L1 所确定的范围

内。

#### 4.3 Dropout


Bagging 是通过结合多个模型降低泛化误差的技术，主要的做法是分别训练几个不同的模型，然后让所有模型表决测试样例的输出。而 Dropout 可以被认为是集成了大

量深层神经网络的 Bagging 方法，因此它提供了一种廉价的 Bagging 集成近似方法，能够训练和评估值数据数量的神经网络。


Dropout 指暂时丢弃一部分神经元及其连接。随机丢弃神经元可以防止过拟合，同时指数级、高效地连接不同网络架构。神经元被丢弃的概率为 1 − p，减少神经元之间

的共适应。隐藏层通常以 0.5 的概率丢弃神经元。使用完整网络（每个节点的输出权重为 p）对所有 2^n 个 dropout 神经元的样本平均值进行近似计算。Dropout 显

著降低了过拟合，同时通过避免在训练数据上的训练节点提高了算法的学习速度。

### 4.4 Drop Connect
Drop Connect 是另一种减少算法过拟合的正则化策略，是 Dropout 的一般化。在 Drop Connect 的过程中需要将网络架构权重的一个随机选择子集设置为零，取代了

在 Dropout 中对每个层随机选择激活函数的子集设置为零的做法。由于每个单元接收来自过去层单元的随机子集的输入，Drop Connect 和 Dropout 都可以获得有限的

泛化性能 [22]。Drop Connect 和 Dropout 相似的地方在于它涉及在模型中引入稀疏性，不同之处在于它引入的是权重的稀疏性而不是层的输出向量的稀疏性。


#### 4.5 早停法
早停法可以限制模型最小化代价函数所需的训练迭代次数。早停法通常用于防止训练中过度表达的模型泛化性能差。如果迭代次数太少，算法容易欠拟合（方差较小，偏差

较大），而迭代次数太多，算法容易过拟合（方差较大，偏差较小）。早停法通过确定迭代次数解决这个问题，不需要对特定值进行手动设置。





参考链接：
https://cloud.tencent.com/developer/article/1165263
https://www.cnblogs.com/sxron/articles/6591149.html
https://www.jiqizhixin.com/articles/2017-12-20
https://blog.csdn.net/jinping_shi/article/details/52433975

  



