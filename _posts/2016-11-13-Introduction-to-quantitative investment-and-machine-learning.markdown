---
layout: post
title:  "机器学习&量化投资入门"
categories: 量化投资
tags: 机器学习 量化投资 Python
excerpt: 概要：

 介绍Logistic Regression的数学模型，推导并详细解释求解最优回归系数的过程
 Python实现Logistic Regression的基本版
 介绍sklearn中的Logistic Regression算法及其关键参数
 实现一个基于Logistic Regression的简单选股策略
---
* content
{:toc}

## 1. 学习机器学习的动机  

[詹姆斯·西蒙斯](https://link.zhihu.com/?target=http%3A//baike.baidu.com/link%3Furl%3Df7gg3CfSgV0FnVShzVofMK65MC3UU6kfUbK0_RupJ0ZHr4rG2UCfYWz5PEHjxqodrhf0O22RKKXJhi3NiCVZxa) 在TED的对话中有提到：  
Q：机器学习在这里扮演了怎样一个角色？  
A：某种意义上，我们做的就是机器学习。你观察一大堆数据，模拟不同的预测方案，直到你越来越擅长于此。我们所做之事，不见得一定有自我反馈，但确实有效。  
视频地址：[与横扫华尔街数学家的珍贵对话](http://v.qq.com/boke/page/s/0/w/s0186b0yrpw.html)

<iframe frameborder="0" width="640" height="498" src="https://v.qq.com/iframe/player.html?vid=s0186b0yrpw&tiny=0&auto=0" allowfullscreen></iframe>

&emsp;&emsp;然而现在有许多已实现的机器学习开源包可供我们调用，如sklearn，更高级的技术还有Hadoop、Spark等等。
那么是不是我们只须知道如何将训练数据与测试数据输入到模型，然后调用分类或回归的结果就好了呢？  
　　理解其数学原理，自己再实现一遍算法有无必要呢？  
　　在汲取一些前辈的建议以及结合自己的思考后，我觉得自己亲手推一遍数学公式，再用Python实现算法还是有必要的。  

* 因为这样不仅能使我们加深对机器学习本质的理解，还能让我们对模型与数据之间联系更加敏感。
例如，如何选择模型、如何选择特征、如何调整参数等等。  
* 其次，目前开源的机器学习算法包提供的都是通用型的算法，并非针对量化投资这一领域来进行优化。
所以，当有必要时，我们须根据自己的需求来优化这些通用算法，甚至重写。另一方面，真正有用的算法在量化领域是不会开源的。 

&emsp;&emsp;当然机器学习这么多的算法也没有必要全都实现一遍，但常见的算法的核心部分还是要亲手实现的。 

&emsp;&emsp;**笔记详细记录了Logistic Regression的数学原理与推导过程。在核心公式推导过程中，都会引出并详细解释关键的求解技巧和对应的数学知识。因此，只要是对本科高数还有记忆的同学都能完全理解和掌握。**

## 2. 什么是Logistic Regression  

&emsp;&emsp;首先，我们知道机器学习是一系列对数据执行分类、回归和聚类等操作的统计算法的统称，这些算法根据历史数据（训练数据）的特征，来对未来数据（测试数据）进行判断。文中介绍的是一种最常用也最重要的分类算法—Logistic Regression。  
　　首先我们引入对数几率函数（logistic function）：

$$
\begin{equation}
y = \frac{1}{1+e^{-z}} 
\end{equation}\tag{1}
$$

其函数曲线如下图所示：  
![](http://7u2ho6.com1.z0.glb.clouddn.com/jekyllWorkSpace.png)

　　从上图我们可以看出，此函数可将定义域为(-∞, +∞)自变量z映射到(0,1)区间。若以 y=0.5为阈值，我们可以利用这个函数实现一个二分类器：

* 当 z>0 ( y>0.5)，将其划分为1类；
* 当 z<0 ( y<0.5)，将其划分为0类。

　　由于于输出y为0到1之间的连续值，我们也可以认为函数是对类别的概率估计。

### 2.1 若有多个输入变量呢？  
那么z由下列公式表示：

$$
\begin{equation}
z=w_0+w_1x_1+...+w_nx_n
\end{equation}\tag{2}
$$

其中$$ω_i$$表示第i个输入变量的权重或是回归系数，第一个输入变量$$x_0$$默认为1。

　　公式(2)的向量形式又如下公式(3)表示：

$$
\begin{equation}
z=w^Tx
\end{equation}\tag{3}
$$

其中，$$ω$$与$$x$$均为n行1列的列向量。

### 2.2 为什么称有如此形式的函数为对数几率函数呢？

通过简单的推导，`Logistic`函数可以写成如下形式：

$$
\begin{equation}
y=\frac{1}{1+e^-z}\Leftrightarrow ln\frac{y}{1-y}=w^Tx
\end{equation}\tag{4}
$$
　其中$$ln\frac{y}{1−y}$$便称为**对数几率**（log odds）。“**几率**”（odds）反映了输入向量x被划分为1类的相对可能性。
　　因此公式（4）的意义是用线性回归模型去描述类别的**对数几率**。

### 2.3 如何求解回归模型中的最优回归系数（权重）呢？

首先根据公式（4）我们可得：

$$
\begin{equation}
ln\frac{p(y=1|x)}{p(y=0|x)}=w^Tx
\end{equation}\tag{5}
$$
进一步我们不难得到在输入为x的情况下，输出类别y分别为1和0的概率为：

$$ 
\begin{align}
p(y=1|x)&=\frac{e^{w^Tx}}{1+e^{w^Tx}}=\frac{1}{1+e^{-w^Tx}}\tag{6.1}\\
p(y=0|x)&=\frac{1}{1+e^{w^Tx}}\tag{6.2}
\end{align}
$$
#### 2.3.1 极大似然估计 是求解最优系数的常用方法之一

极大似然估计的思想为：对于所有的抽样样本，使它们联合概率达到最大的系数便是统计模型最优的系数。  
　　因此对于第i个输入数据$$x_i$$，它被划分为$$y_i$$的概率可由公式（7）表示：

$$
\begin{equation}
p(y_i|x_i;w)=p(y_i=1|x_i;w)^{y_i}p(y_i=0|x_i;w)^{1-y_i}\tag{7}
\end{equation}
$$
公式（7）巧妙之处在于，当$$y_i=1$$时，$$p(y_i=0|x_i;w)^{1-y_i}=1$$；而当$$y_i=0$时，$p(y_i=1|x_i;w)^{y_i}=1$$。  
在极大似然法中，我们假设样本之间都是独立同分布的，因此它们的联合概率就是它们各自概率的乘积。因此关于回归系数$$w$$的极大似然函数可由公式（8）表示：

$$
\begin{equation}
L(w)=\prod_{i=1}^{m}p(y_i=1|x_i;w)^{y_i}p(y_i=0|x_i;w)^{1-y_i}\tag{8}
\end{equation}
$$
为了便于计算，我们对公式（8）两边同时取对数：

$$
\begin{equation}
lnL(w)=\left (\prod_{i=1}^{m}p(y_i=1|x_i;w)^{y_i}p(y_i=0|x_i;w)^{1-y_i}\right )\tag{9}
\end{equation}
$$
根据对数的性质，公式（9）可以逐步写成如下形式：

$$
\begin{array}
lnL(w)=\left (\prod_{i=1}^{m}p(y_i=1|x_i;w)^{y_i}p(y_i=0|x_i;w)^{1-y_i}\right )\\
=\sum_{i=1}^{m}y_ilnp(y_i=1|x_i;w)+(1-y_i)lnp(y_i=0|x_i;w)\\
=\sum_{i=1}^{m}\left(y_iw^Tx-ln(1+e^{w^Tx}) \right)\tag{10}
\end{array}\\
$$
#### 2.3.2 如何求解最优的回归系数$$w$$呢？

有两种经典的数值优化算法：a) 梯度上升法；b) 牛顿法。  
　　**a) 梯度上升法**：函数在某一点的梯度总是指向该函数增长最快的方向。因此沿着该函数的梯度方向探寻就能找到该函数的最大值。梯度上升法的迭代公式如下：

$$
w^{t+1}=w^t+\alpha\nabla_wlnL(w^t)\tag{11}
$$
公式（11）中$$\alpha$$表示每次迭代的步进。$$\nabla_w$$表示梯度算子。

根据以上定义，我们求取公式（10）的梯度：  
　　首先方程两边同时取微分：

$$
\begin{array}
d(lnL(w))=\sum_{i=1}^{m}\left(y_id\left(\boldsymbol{w}^T\boldsymbol{x}_i\right)-d\left(ln(1+e^{\boldsymbol{w}^T\boldsymbol{x}_i})\right)\right)\\
=\sum_{i=1}^{m}\left(y_id\left(\boldsymbol{w}^T\boldsymbol{x}_i\right)^T-\frac{e^{\boldsymbol{w}^T\boldsymbol{x}_i}}{1+e^{\boldsymbol{w}^T\boldsymbol{x}_i}}d\left(\boldsymbol{w}^T\boldsymbol{x}_i\right)^T\right)\\
=\sum_{i=1}^{m}y\boldsymbol{x}^T_id(\boldsymbol{w})-p(y_i=1|x_i;\boldsymbol{w})\boldsymbol{x}^T_id(\boldsymbol{w})\\
=\sum_{i=1}^{m}\left(\boldsymbol{x}^T_i(y_i-p(y_i=1|\boldsymbol{x}_i;\boldsymbol{w}))\right)d(\boldsymbol{w})
\end{array}\tag{12}
$$

因此公式（10）的梯度最终可写成公式（13）的形式：

$$
\nabla_wlinL(\boldsymbol{w})=\frac{\partial lnL(\boldsymbol{w})}{\partial\boldsymbol{w}}=\sum_{i=1}^{m}(y_i-p(y_i=1|\boldsymbol{x}_i;\boldsymbol{w}))\tag{13}
$$

从公式（12）到（13）有两处向量微分运算的小技巧（刚开始我也不会...）：  
　　第一处：公式（12）的第二行，由于$$\boldsymbol{w}^T\boldsymbol{x}_i$$为标量，我们取转置后，其性质不变。  
　　第二处：在向量的微分运算中，梯度与导数矩阵是相互装置的关系（这样说可能不是特别严谨...）。  
　　　　　　所以公式（12）的$$\boldsymbol{x}_i^T$$在公式（13）中变为$$\boldsymbol{x}_i$$。

**b) 牛顿法**：其原理是利用泰勒公式不断迭代，从而逐次逼近零点或极值点。  
　　**一阶展开求零点：**  
　　我们知道泰勒公式在x0处的一阶展开如下所示：

$$
f(x)\approx f(x_0)+(x-x_0)f'(x_0)\tag{14}
$$
我们用一阶展开式求解函数的零点：

$$
f(x_0)+(x-x_0)f'(x_0)=0\tag{15}
$$
稍作整理，公式（15）可写成如下形式：

$$
x=x_0-\frac{f(x_0)}{f'(x_0)}\tag{16}
$$
　　由于一阶展开式只是与原函数近似相等，因此公式（16）中的x并非函数的零点，而只是比x0更接近零点。因此通过对公式（16）迭代可以不断逼近函数零点。

　　**如果是求函数的极值点呢？**
　　那我们就要用到泰勒二阶展开式：

$$
f(x)\approx f(x_0)+(x-x_0)f'(x_0)+\frac{1}{2}(x-x_0)^2f''(x_0)=g(x)\tag{17}
$$
　　我们知道函数在其一阶导数等于0时取到极值，因此我们求上式(17) g′(x)=0的解：

$$
g'(x)=f'(x_0)+(x-x_0)f''(x_0)=0\tag{18}
$$
　　通过像公式（15）一样整理上式（18），可得：

$$
x=x_0-\frac{f'(x_0)}{f''(x_0)}\tag{19}
$$
　　**如果输入是多维的变量呢？**  
　　高维情况的牛顿法迭代公式与公式（19）相似，其如下所示：

$$
\boldsymbol{x}^{t+1}=\boldsymbol{x}^t-\left[Hf(\boldsymbol{x}^t)\right]^{-1}\nabla f(\boldsymbol{x}^t)\tag{20}
$$

　　其中$$\nabla f(\boldsymbol{x}^t)$$表示的是函数$$f(\boldsymbol{x}^t)$$的梯度，$$\left[Hf(\boldsymbol{x}^t)\right]^{-1}$$表示的是函数的`Hessian`矩阵的逆矩阵。
　　当输入变量为一维时，`Hessian`矩阵其实可以理解为变量的二阶导数。其`Hessian`矩阵的定义如下：

$$
H(f(\boldsymbol{x}))=\frac{\partial^2f(\boldsymbol{x})}{\partial\boldsymbol{x}\partial{\boldsymbol{x}}^T}=
\begin{bmatrix}
 \frac{\partial^2f}{\partial x^2_1} & \frac{\partial^2f}{\partial x_1\partial x_2} & \cdots & \frac{\partial^2f}{\partial x_1\partial x_n}\\ 
 \frac{\partial^2f}{\partial x_2\partial x_1} & \frac{\partial^2f}{\partial x^2_2} & \cdots & \frac{\partial^2f}{\partial x_2\partial x_n}\\
 \vdots & \vdots & \ddots & \vdots\\ 
 \frac{\partial^2f}{\partial x_n\partial x_1} & \frac{\partial^2f}{\partial x_n\partial x_2} & \cdots & \frac{\partial^2f}{\partial x^2_n}\\
 \end{bmatrix}\tag{21}
$$
　　**梯度上升法与牛顿法的比较：**  
　　梯度法是一阶收敛的，而牛顿法是二阶收敛的。因此牛顿法的迭代次数要少于梯度法，然而`Hessian`的逆矩阵的计算会增加算法的复杂度。  
　　这个问题又可以通过`Quasi-Newton`方法解决。

## 3. `Python`实现`Logistic Regression`的基本版  

　　我们选择牛顿法求解`Logistic Regression`中的最优回归系数。  
　　根据牛顿法迭代公式（20）和Hessian矩阵的定义（21），本贴`Logistic Regression`中的回归系数迭代更新公式为：

$$
\boldsymbol{w}^{t+1}=\boldsymbol{w}^t-\left[\frac{\partial^2lnL(\boldsymbol{w})}{\partial\boldsymbol{w}\partial\boldsymbol{w}^T}\right]^{-1}\frac{\partial lnL(\boldsymbol{w})}{\partial\boldsymbol{w}}\tag{22}
$$
其中$$\frac{\partial lnL(w)}{\partial w}$$在公式（13）中已求得，而对$$\frac{\partial^2lnL(w)}{\partial w\partial w^T}$$的求解可按照上文所提到的两条向量微分运算方法来计算，因此：

$$
\frac{\partial^2lnL(\boldsymbol{w})}{\partial{\boldsymbol{w}}\partial{\boldsymbol{w}^T}}
=\sum_{i=1}^{m}\boldsymbol{x}_i\boldsymbol{x}_i^Tp(y_i=1|\boldsymbol{x}_i;\boldsymbol{w})\left(p(y_i=1|\boldsymbol{x}_i;\boldsymbol{w}-1\right)\tag{23}
$$

　好了！到此迭代公式已经完全掌握了，现在该实现算法了！

### 3.1 准备数据：

　　从网上整理了一组数据: 前两列元素是`feature`；最后一列是`label`，其值是0或1。


```python
data = [[6.9, 3.1, 1.0], [5.0, 3.5, 0.0], [5.0, 2.3, 1.0], [4.6, 3.4, 0.0], [5.5, 2.4, 1.0], [5.5, 3.5, 0.0], [5.2, 4.1, 0.0], [4.7, 3.2, 0.0], [5.4, 3.9, 0.0], [5.6, 3.0, 1.0], [6.1, 2.8, 1.0], [5.8, 2.7, 1.0], [6.2, 2.2, 1.0], [5.0, 3.4, 0.0], [5.7, 3.8, 0.0], [6.3, 2.5, 1.0], [6.0, 3.4, 1.0], [4.9, 3.1, 0.0], [4.9, 3.0, 0.0], [5.6, 2.7, 1.0], [5.7, 2.8, 1.0], [5.0, 3.2, 0.0], [5.9, 3.0, 1.0], [6.1, 2.8, 1.0], [5.1, 3.5, 0.0], [4.6, 3.2, 0.0], [5.1, 2.5, 1.0], [5.1, 3.7, 0.0], [5.4, 3.7, 0.0], [5.3, 3.7, 0.0], [4.8, 3.0, 0.0], [4.3, 3.0, 0.0], [4.9, 2.4, 1.0], [5.5, 2.5, 1.0], [5.8, 4.0, 0.0], [5.1, 3.8, 0.0], [5.7, 2.8, 1.0], [5.1, 3.8, 0.0], [5.0, 3.0, 0.0], [4.4, 3.0, 0.0], [5.4, 3.4, 0.0], [5.7, 2.6, 1.0], [6.4, 3.2, 1.0], [5.2, 3.5, 0.0], [4.8, 3.4, 0.0], [6.0, 2.7, 1.0], [5.6, 3.0, 1.0], [5.1, 3.3, 0.0], [5.4, 3.9, 0.0], [6.8, 2.8, 1.0], [5.9, 3.2, 1.0], [4.5, 2.3, 0.0], [6.1, 3.0, 1.0], [4.4, 2.9, 0.0], [5.4, 3.4, 0.0], [5.5, 2.4, 1.0], [5.6, 2.9, 1.0], [5.5, 2.6, 1.0], [5.7, 4.4, 0.0], [5.0, 3.6, 0.0], [5.0, 3.4, 0.0], [4.6, 3.6, 0.0], [5.7, 2.9, 1.0], [5.0, 2.0, 1.0], [6.3, 2.3, 1.0], [7.0, 3.2, 1.0], [4.6, 3.1, 0.0], [5.0, 3.3, 0.0], [5.1, 3.8, 0.0], [4.9, 3.1, 0.0], [4.7, 3.2, 0.0], [5.5, 2.3, 1.0], [6.7, 3.1, 1.0], [6.5, 2.8, 1.0], [6.7, 3.0, 1.0], [4.4, 3.2, 0.0], [6.6, 2.9, 1.0], [5.0, 3.5, 0.0], [6.4, 2.9, 1.0], [5.1, 3.5, 0.0], [6.3, 3.3, 1.0], [5.4, 3.0, 1.0], [6.2, 2.9, 1.0], [5.1, 3.4, 0.0], [5.2, 3.4, 0.0], [4.8, 3.0, 0.0], [6.0, 2.9, 1.0], [5.5, 4.2, 0.0], [4.8, 3.4, 0.0], [5.8, 2.6, 1.0], [5.8, 2.7, 1.0], [6.7, 3.1, 1.0], [4.8, 3.1, 0.0], [5.7, 3.0, 1.0], [6.1, 2.9, 1.0], [6.0, 2.2, 1.0], [6.6, 3.0, 1.0], [4.9, 3.1, 0.0], [5.6, 2.5, 1.0], [5.2, 2.7, 1.0]] 
```

### 3.2 数据预处理，包括：
　　1.增加常数项x0: 1.0；

　　2.feature 与 label的分离；

　　3.返回数组类型的feature 和 label。

```python
def preprocess(data):
    # labels 用来标识每个数据的类别:0或1
    labels = [] 
    # train_X 输入数据列表: [[1.0, x_i1, x_i2],..., [1.0, x_n1, x_n2]]
    train_X = [] 
    for x in data:
        # 二维平面的直线方程为： ax + by + c = 0，因此我们需要再原来的输入数据中增加一常数项 1.0
        train_X.append([1.0, x[0], x[1]])
        labels.append(x[2])
    return np.array(train_X), np.array(labels) 
```

### 3.3 定义`Logistic`函数：

```python
def logit(z):
    '''
        logistic function 即：
        牛顿迭代公式中的p(yi=1 | xi; w)
    '''
    return 1.0 / (1.0 + np.exp(-z))
```

### 3.4 梯度上升法算法：

```python
def grad(train_X, labels, max_iter = 500):
    # 下面两行代码用于矩阵化 训练数据
    X = np.mat(train_X) # 100 行 3列
    Y = np.mat(labels).transpose() # 100 行 1列
    rows, columns = np.shape(X)
    # 初始回归系数 weights 为全1的向量, weights的行数与输入的训练数据维度相同
    weights = np.ones((columns, 1))
    # 步进alpha: 0.001
    alpha = 0.001
    # t 表示当前迭代的次数
    t = 0
    while t < max_iter:
        P1 = logit(np.dot(X, weights))
        weights += alpha * X.T * (Y - P1)
        t += 1

    return weights 
```
&emsp;&emsp;我们还是给出了梯度上升法的算法实现。   
　　上述算法的倒数第三行，我们用矩阵形式表示梯度上升法的迭代公式，其具体数学形式如下：

$$
\begin{bmatrix}
w_0\\w_1\\w_2
\end{bmatrix}^{(t+1)}
=\begin{bmatrix}
w_0\\w_1\\w_2
\end{bmatrix}^t+\alpha
\begin{bmatrix}
1.0 & x_{11} & x_{12}\\
1.0 & x_{21} & x_{22}\\
\vdots & \vdots & \vdots\\
1.0 & x_{n1} & x_{n2}
\end{bmatrix}^T
\begin{bmatrix}
\begin{pmatrix}
y_1\\y_2\\\vdots\\y_n
\end{pmatrix}-
\begin{pmatrix}
p_1(\boldsymbol{x}_1;\boldsymbol{w})\\
p_1(\boldsymbol{x}_2;\boldsymbol{w})\\
\vdots\\
p_1(\boldsymbol{x}_n;\boldsymbol{w})
\end{pmatrix}
\end{bmatrix}
$$

### 3.5 牛顿法算法：
```python
def newton(train_X, labels, max_iter = 10):
    X = np.mat(train_X)
    Y = np.mat(labels).transpose()
    rows, columns = np.shape(X)
    # 初始回归系数 weights 为全0的向量, weights的行数与输入的训练数据维度相同。
    weights = np.zeros((columns, 1))
    # t 表示当前迭代的次数
    t = 0
    while t < max_iter:
        P1 = logit(np.dot(X, weights))
        gradient =  np.dot(X.T, (Y - P1))
        P_mat = np.array(P1) * np.array(P1 - 1) * np.eye(rows)
        hessian = X.T * P_mat * X
        # np.linalg.inv() 是求的Hessian矩阵的逆矩阵
        weights -= np.linalg.inv(hessian) * gradient
        t += 1

    return weights 
```
&emsp;&emsp;与上小节3.4一致，我们都用矩阵形式表示迭代公式。其中梯度的具体数学形式在上小节已给出。  
　　在本节的牛顿算法中，Hessian矩阵具体数学形式如下：

$$
\boldsymbol{Hessian}=
\begin{bmatrix}
1.0 & x_{11} & x_{12}\\
1.0 & x_{21} & x_{22}\\
\vdots & \vdots & \vdots\\
1.0 & x_{n1} & x_{n2}
\end{bmatrix}^T
\begin{bmatrix}
p_1(\boldsymbol{x}_1;\boldsymbol{w})(p_1(\boldsymbol{x}_1;\boldsymbol{w})-1) & &\\
 & p_1(\boldsymbol{x}_1;\boldsymbol{w})(p_1(\boldsymbol{x}_1;\boldsymbol{w})-1) &\\
 & \ddots &\\
 & & p_1(\boldsymbol{x}_1;\boldsymbol{w})(p_1(\boldsymbol{x}_1;\boldsymbol{w})-1)\\
\end{bmatrix}
\begin{bmatrix}
1.0 & x_{11} & x_{12}\\
1.0 & x_{21} & x_{22}\\
\vdots & \vdots & \vdots\\
1.0 & x_{n1} & x_{n2}
\end{bmatrix}
$$

### 3.6 选择最优算法，根据求得的最优系数画出回归直线：

```python
# 牛顿法
weights = newton(preprocess(data)[0], preprocess(data)[1])
# 梯度法
#weights = grad(preprocess(data)[0], preprocess(data)[1])
# 根据数据的类别，得到两个数据集
x0=list()
y0=list()
x1=list()
y1=list()
for i in data:
    if i[-1] == 0:
        x0.append(i[0])
        y0.append(i[1])
    else:
        x1.append(i[0])
        y1.append(i[1])

# c=sns.color_palette()[1] ： seaborn 包中的配色方案
plt.scatter(x0, y0, marker='^', c=sns.color_palette()[1])
plt.scatter(x1, y1, marker='o', c=sns.color_palette()[2])

x = np.arange(4.0, 7.0, 0.1)
# 画出回归直线： w0 + w1*x1 + w2*x2 = 0, 我们定义x1为横轴坐标，x2为纵轴坐标
y = (-weights[0] - weights[1]*x)/weights[2]
plt.plot(x, y, color=sns.color_palette()[0])
plt.xlabel('x1')
plt.ylabel('x2')
plt.show() 
```

### 3.7 下面一个cell运行整段代码：

```python
import numpy as np #数值计算包
import matplotlib.pyplot as plt #绘图包
import seaborn as sns # 绘图美化包 用于配色

data = [[6.9, 3.1, 1.0], [5.0, 3.5, 0.0], [5.0, 2.3, 1.0], [4.6, 3.4, 0.0], [5.5, 2.4, 1.0], [5.5, 3.5, 0.0], [5.2, 4.1, 0.0], [4.7, 3.2, 0.0], [5.4, 3.9, 0.0], [5.6, 3.0, 1.0], [6.1, 2.8, 1.0], [5.8, 2.7, 1.0], [6.2, 2.2, 1.0], [5.0, 3.4, 0.0], [5.7, 3.8, 0.0], [6.3, 2.5, 1.0], [6.0, 3.4, 1.0], [4.9, 3.1, 0.0], [4.9, 3.0, 0.0], [5.6, 2.7, 1.0], [5.7, 2.8, 1.0], [5.0, 3.2, 0.0], [5.9, 3.0, 1.0], [6.1, 2.8, 1.0], [5.1, 3.5, 0.0], [4.6, 3.2, 0.0], [5.1, 2.5, 1.0], [5.1, 3.7, 0.0], [5.4, 3.7, 0.0], [5.3, 3.7, 0.0], [4.8, 3.0, 0.0], [4.3, 3.0, 0.0], [4.9, 2.4, 1.0], [5.5, 2.5, 1.0], [5.8, 4.0, 0.0], [5.1, 3.8, 0.0], [5.7, 2.8, 1.0], [5.1, 3.8, 0.0], [5.0, 3.0, 0.0], [4.4, 3.0, 0.0], [5.4, 3.4, 0.0], [5.7, 2.6, 1.0], [6.4, 3.2, 1.0], [5.2, 3.5, 0.0], [4.8, 3.4, 0.0], [6.0, 2.7, 1.0], [5.6, 3.0, 1.0], [5.1, 3.3, 0.0], [5.4, 3.9, 0.0], [6.8, 2.8, 1.0], [5.9, 3.2, 1.0], [4.5, 2.3, 0.0], [6.1, 3.0, 1.0], [4.4, 2.9, 0.0], [5.4, 3.4, 0.0], [5.5, 2.4, 1.0], [5.6, 2.9, 1.0], [5.5, 2.6, 1.0], [5.7, 4.4, 0.0], [5.0, 3.6, 0.0], [5.0, 3.4, 0.0], [4.6, 3.6, 0.0], [5.7, 2.9, 1.0], [5.0, 2.0, 1.0], [6.3, 2.3, 1.0], [7.0, 3.2, 1.0], [4.6, 3.1, 0.0], [5.0, 3.3, 0.0], [5.1, 3.8, 0.0], [4.9, 3.1, 0.0], [4.7, 3.2, 0.0], [5.5, 2.3, 1.0], [6.7, 3.1, 1.0], [6.5, 2.8, 1.0], [6.7, 3.0, 1.0], [4.4, 3.2, 0.0], [6.6, 2.9, 1.0], [5.0, 3.5, 0.0], [6.4, 2.9, 1.0], [5.1, 3.5, 0.0], [6.3, 3.3, 1.0], [5.4, 3.0, 1.0], [6.2, 2.9, 1.0], [5.1, 3.4, 0.0], [5.2, 3.4, 0.0], [4.8, 3.0, 0.0], [6.0, 2.9, 1.0], [5.5, 4.2, 0.0], [4.8, 3.4, 0.0], [5.8, 2.6, 1.0], [5.8, 2.7, 1.0], [6.7, 3.1, 1.0], [4.8, 3.1, 0.0], [5.7, 3.0, 1.0], [6.1, 2.9, 1.0], [6.0, 2.2, 1.0], [6.6, 3.0, 1.0], [4.9, 3.1, 0.0], [5.6, 2.5, 1.0], [5.2, 2.7, 1.0]]

def preprocess(data):
    # labels 用来标识每个数据的类别:0或1
    labels = [] 
    # train_X 输入数据列表: [[1.0, x_i1, x_i2],..., [1.0, x_n1, x_n2]]
    train_X = [] 
    for x in data:
        # 二维平面的直线方程为： ax + by + c = 0，因此我们需要再原来的输入数据中增加一常数项 1.0
        train_X.append([1.0, x[0], x[1]])
        labels.append(x[2])
    return np.array(train_X), np.array(labels)


def logit(z):
    '''
        logistic function 即：
        牛顿迭代公式中的p(yi=1 | xi; w)
    '''
    return 1.0 / (1.0 + np.exp(-z))
    
def grad(train_X, labels, max_iter = 500):
    X = np.mat(train_X) # 100 行 3列
    Y = np.mat(labels).transpose() # 100 行 1列
    rows, columns = np.shape(X)
    # 初始回归系数 weights 为全1的向量, weights的行数与输入的训练数据维度相同
    weights = np.ones((columns, 1))
    # 步进alpha: 0.001
    alpha = 0.001
    # t 表示当前迭代的次数
    t = 0
    while t < max_iter:
        P1 = logit(np.dot(X, weights))
        weights += alpha * X.T * (Y - P1)
        t += 1
    
    return weights

def newton(train_X, labels, max_iter = 10):
    X = np.mat(train_X)
    Y = np.mat(labels).transpose()
    rows, columns = np.shape(X)
    # 初始回归系数 weights 为全0的向量, weights的行数与输入的训练数据维度相同。
    weights = np.zeros((columns, 1))
    # t 表示当前迭代的次数
    t = 0
    while t < max_iter:
        P1 = logit(np.dot(X, weights))
        gradient =  np.dot(X.T, (Y - P1))
        P_mat = np.array(P1) * np.array(P1 - 1) * np.eye(rows)
        hessian = X.T * P_mat * X
        weights -= np.linalg.inv(hessian) * gradient
        t += 1
    
    return weights
    
# 牛顿法
weights = newton(preprocess(data)[0], preprocess(data)[1])
# 梯度法
# weights = grad(preprocess(data)[0], preprocess(data)[1])
# 根据数据的类别，得到两个数据集
x0=list()
y0=list()
x1=list()
y1=list()
for i in data:
    if i[-1] == 0:
        x0.append(i[0])
        y0.append(i[1])
    else:
        x1.append(i[0])
        y1.append(i[1])

# c=sns.color_palette()[1] ： seaborn 包中的配色方案
plt.scatter(x0, y0, marker='^', c=sns.color_palette()[1])
plt.scatter(x1, y1, marker='o', c=sns.color_palette()[2])

x = np.arange(4.0, 7.0, 0.1)
# 画出回归直线： w0 + w1*x1 + w2*x2 = 0, 我们定义x1为横轴坐标，x2为纵轴坐标
y = (-weights[0] - weights[1]*x)/weights[2]
plt.plot(x, y, color=sns.color_palette()[0])
plt.xlabel('x1')
plt.ylabel('x2')
plt.show() 
```
![](../css/pics/p20161113-02.jpg)

## 4. sklearn中的`Logistic Regression`算法  
&emsp;&emsp;`sklearn`是`Python`实现的开源机器学习算法包，优矿正好也支持它。下面简单介绍下`sklearn`中`Logistic Regression`的使用。

### 4.1 实例化Logistic Regression：

```python
from sklearn.linear_model import LogisticRegression
# 参数C为正则化强度的倒数，用于控制回归系数的复杂度。
lr = LogisticRegression(C = 1.0) 
```
&emsp;&emsp;理解什么是正则化，我们先理解正则化的作用： 正则化可防止模型过于复杂。  
　　我们知道机器学习算法本质是一个最优化问题，算法根据训练数据所求得的模型系数可以使得数据分类、拟合等准确率更高。  
　　在保证模型准确性的同时，我们同样希望模型不过于复杂，因此我们就要在两者之间权衡。  
　　模型的准确性一般用代价函数来衡量，而模型的复杂度一般用系数的平方表示（系数向量的内积）： 
$$
L'(\boldsymbol{w})=L(\boldsymbol{w})+C\boldsymbol{w}^T\boldsymbol{w}
$$ 
&emsp;&emsp;在本问题中，我们将极大似然函数的负值视为代价函数，代价函数越小，说明极大似然函数越大，说明模型分类的准确性越高；  
&emsp;&emsp;$$C\boldsymbol{w}^T\boldsymbol{w}$$用以衡量模型复杂度，C用以控制原代价函数与模型复杂度之间的折中。  
&emsp;&emsp;**很显然，C越大，算法对模型复杂度越敏感。 **

### 4.2 训练数据：

```python
lr.fit(train_X, labels) 
```
&emsp;&emsp;其中，`train_X`表示的训练数据的特征，其格式n行m列的矩阵，n为训练数据的个数，m为数据的特征个数（维度）；  
&emsp;&emsp;`labels`表示的训练数据的类别，格式为n行1列的列向量。

### 4.3 输出模型系数，输出测试数据的类别与概率：

```python
# 回归系数
print '输出模型回归系数：', lr.coef_
print '\n'

# 预测类别
print '输入数据[1.0, 5.0, 2.0]的类别：', lr.predict([1.0, 5.0, 2.0])
print '输入数据[1.0, 5.0, 4.0]的类别：', lr.predict([1.0, 5.0, 4.0])
print '\n'

# 预测概率
print  '输入数据[1.0, 5.0, 2.0]为0类1类的概率：', lr.predict_proba([1.0, 5.0, 2.0])
print  '输入数据[1.0, 5.0, 4.0]为0类1类的概率：', lr.predict_proba([1.0, 5.0, 4.0]) 
```
　　如果用上一节准备好数据，我们可分别得到如下结果：  
**回归系数**
```python
输出模型回归系数： [[-0.55312625  2.2765652  -3.63240141]] 
```
**预测类别**
```python
输入数据[1.0, 5.0, 2.0]的类别： [ 1.]
输入数据[1.0, 5.0, 4.0]的类别： [ 0.] 
```
**预测概率**
```python
输入数据[1.0, 5.0, 2.0]为0类1类的概率： [[ 0.04689694  0.95310306]]
输入数据[1.0, 5.0, 4.0]为0类1类的概率： [[ 0.98597835  0.01402165]] 
```

## 5. 基于`Logistic Regression`的简单选股策略 
&emsp;&emsp;该策略原理简单，即用`T-1`天股票的因子值，预测10个交易日后股票的涨跌。  
　　训练数据为`T-1`到`T-60`之间的滚动时间窗口。  
　　策略示意图如下所示：

$$
\underbrace{ {T-60,\cdots,}\overbrace{\overbrace{T-11}^{取特征因子},\cdots,\overbrace{T-1}^{判断类别：1涨0跌}T}^{一组训练数据}}_{所有训练数据}
$$ 

&emsp;&emsp;在回测时，为了避免幸存者偏差，选股都是使用`set_universe('HS300', begin_data)`，没有直接使用`account.universe`。  
　　下面的cell给出策略代码及回测结果：

```python
from CAL.PyCAL import *
from datetime import datetime, timedelta
from sklearn.linear_model import LogisticRegression
import numpy as np
import pandas as pd

start = '2012-08-01'                       # 回测起始时间
end = '2015-08-01'                         # 回测结束时间
benchmark = 'HS300'                        # 策略参考标准
universe = set_universe('HS300')           # 证券池，支持股票和基金
capital_base = 1000000                     # 起始资金
freq = 'd'                                 # 策略类型，'d'表示日间策略使用日线回测，'m'表示日内策略使用分钟线回测
refresh_rate = 10                           # 调仓频率，表示执行handle_data的时间间隔，若freq = 'd'时间间隔的单位为交易日，若freq = 'm'时间间隔为分钟

# 佣金万八+印花税千一；滑点默认
commission = Commission(buycost=0.0008, sellcost=0.0018) 
slippage = Slippage() 

# 选取的因子
features = ["LFLO","PB","PE","VOL10","VOL20","MA10","MA20", "RSI", "Volatility"]
# 股票数目
stocksNumber = 20
# 训练数据窗口长度：windows
windows = 60
# 预测天数： span
span = 10


def initialize(account):                   # 初始化虚拟账户状态
    account.my_universe = list()

def handle_data(account):                  # 每个交易日的买入卖出指令
    # 构造训练数据：
        # 前 windows 个交易日内的每一个交易日，我们做如下判别：
            # 1. 如果某只股票的近10日超额收益率（相对HS300）大于0， 我们判别为1类，否则为0类；
            # 2. 该交易日10天前的股票因子值作为Logistic Regression的训练数据的特征值。
            
    # 创建一个空DataFrame来记录每个交易日的训练数据：train_X
    train_data = pd.DataFrame()
    
    yesterday = Date.fromDateTime(account.previous_date)
    # 从昨天开始，构造训练数据，直到第windows天前
    for t in range(windows):
        end_date = Calendar('China.SSE').advanceDate(yesterday, Period('-%dB' % t))
        end_date = end_date.strftime('%Y%m%d')
        # span天前的日期为开始的日日期： bgn_data
        bgn_date = Calendar('China.SSE').advanceDate(end_date,  Period('-%dB' % span))
        bgn_date = bgn_date.strftime("%Y%m%d")
        
        # 取HS300的span日前后的收盘价，并计算收益率：
        bgn_price_HS300 = DataAPI.MktIdxdGet(tradeDate=bgn_date, ticker='000300',field=['closeIndex'], pandas="1")
        end_price_HS300 = DataAPI.MktIdxdGet(tradeDate=end_date, ticker='000300',field=['closeIndex'], pandas="1")
        rtn_HS300 = (end_price_HS300 - bgn_price_HS300) / bgn_price_HS300
        # 提取HS300收益的数值
        rtn_HS300 = rtn_HS300.values[0][0]
        
        # 取各股票的span日前后的收盘价，并计算收益率：
        bgn_price_stk = DataAPI.MktEqudGet(tradeDate=bgn_date, secID=set_universe('HS300', bgn_date), field=['secID','closePrice'], pandas="1")
        bgn_price_stk.set_index('secID', inplace=True)
        end_price_stk = DataAPI.MktEqudGet(tradeDate=end_date, secID=set_universe('HS300', bgn_date), field=['secID','closePrice'], pandas="1")
        end_price_stk.set_index('secID', inplace=True)
        # 计算股票 超额收益率： 减去HS300收益
        alpha_rtn_stk = (end_price_stk - bgn_price_stk) / bgn_price_stk - rtn_HS300
        # 将原来的closePrice列名改为label 
        alpha_rtn_stk.columns = ['label']
        # 其中超额收益大于0的label取值为1，否则取0
        alpha_rtn_stk = alpha_rtn_stk.applymap(lambda x: 1 if x>0 else 0)
        # 丢弃缺失值
        alpha_rtn_stk.dropna(inplace=True)
        
        # 取 span 日前（即，bgn_data当日因子）股票的所有因子值：
        stk_list = alpha_rtn_stk.index.tolist()  # 股票列表：stk_list
        factors_df = DataAPI.MktStockFactorsOneDayGet(tradeDate=bgn_date, secID=stk_list, field=['secID']+features, pandas="1").set_index('secID')
        # 取factors_df中的每一列，对其因子去极值，中性化，标准化
        for factor in features:
            raw_data = factors_df[factor].to_dict()
            #new_data = standardize(neutralize(winsorize(raw_data), bgn_date))
            new_data = standardize(winsorize(raw_data))
            alpha_rtn_stk[factor] = pd.Series(new_data.values(), index=new_data.keys())
        # 重置索引，使secID重置为alpha_rtn_stk的列
        alpha_rtn_stk.reset_index(inplace=True)
        # 删除'secID'这一列
        alpha_rtn_stk.drop(['secID'], axis=1, inplace=True)
        # 丢弃缺失值
        alpha_rtn_stk.dropna(inplace=True)
        # 与存储训练数据的DataFrame-train_data 合并
        train_data = pd.concat([train_data, alpha_rtn_stk])
    
    # 训练数据 因子值和类别 分离：
    train_X = train_data[features].values
    label = train_data['label'].values
    
    
    # 准备 测试数据test_df：上一交易日HS300股票的因子值
    # 计算 近20个交易日日期，后续用以提剔除ST股、流动性差、未上市或涨停的股票
    endDate = account.previous_date
    beginDate = Calendar('China.SSE').advanceDate(endDate, Period('-20B'))
    endDate = endDate.strftime('%Y%m%d')
    beginDate = beginDate.strftime('%Y%m%d')
    
    # 剔除ST类股票
    STlist = DataAPI.SecSTGet(secID=set_universe('HS300', endDate), beginDate=endDate, endDate=endDate, field=['secID'], pandas="1")['secID'].tolist()
    account.my_universe = [stk for stk in account.universe if stk not in STlist]
    
    # 去除流动性差的股票
    turnOver = DataAPI.MktEqudGet(secID=account.my_universe, beginDate=beginDate, endDate=endDate, field=['secID', 'turnoverValue'], pandas="1")
    # 按secID分组，计算前20个交易日的平均成交量
    turnOver = turnOver.groupby(by='secID').mean()
    turnOver = turnOver[turnOver['turnoverValue'] > 10000000.].reset_index()
    account.my_universe = [stk for stk in account.my_universe if stk in turnOver['secID'].tolist()]  
    
    # 去除新上市或复牌的股票
    # 获取昨日股票池里股票的开盘价
    openPrice = DataAPI.MktEqudGet(tradeDate=endDate, secID=account.my_universe, field=['secID', 'openPrice'], pandas="1")
    openPrice = openPrice[openPrice['openPrice'] != 0]
    openPrice = openPrice[openPrice['openPrice'] != np.nan]
    account.my_universe = [stk for stk in account.my_universe if stk in openPrice['secID'].tolist()]
    
    test_df = DataAPI.MktStockFactorsOneDayGet(tradeDate=account.previous_date.strftime("%Y%m%d"), secID=account.my_universe, field=['secID']+features, pandas="1").set_index('secID')
    # 测试数据test_df中的每一列，对其因子去极值，中性化，标准化
    for factor in features:
        raw_data = test_df[factor].to_dict()
        #new_data = standardize(neutralize(winsorize(raw_data), account.previous_date.strftime("%Y%m%d")))
        new_data = standardize(winsorize(raw_data))
        test_df[factor] = pd.Series(new_data.values(), index=new_data.keys())

    # 丢弃缺失值：
    test_df.dropna(inplace=True)
    # 测试数据：
    test_X = test_df[features]
    # 股票列表：
    buy_list = test_df.index.tolist()
    
    
    ####################################### 训练Logistic Regression模型 ##########################################
    lr  = LogisticRegression(C = 0.1) # C为正则化强度的系数，C越小 模型系数容忍值越大
    lr.fit(train_X, label)
    predicted_proba = lr.predict_proba(test_X)
    # 创建DataFrame：data为预测概率，index为对应的secID
        # 其中predicted_proba有若干个list，每一个list中有两个值，第一个划分为0类的概率，第二个为1类的概率
    proba_df = pd.DataFrame(predicted_proba[:,1], index=buy_list, columns=['probability'])
    # 对概率降序排列，取概率最大的前 stock 只 股票
    proba_df = proba_df.sort(columns='probability', ascending=False)[: stocksNumber]
    
    # 股票列表：buy_list
    buy_list = proba_df.index.tolist()
    
    # 不在buy_list中的股票先全部卖出
    for stk in account.valid_secpos:
        if stk not in buy_list:
            order_to(stk, 0)
    
    # 字典change 记录buy_list中股票的仓位变化
    change = {}
    
    perCapital =  account.referencePortfolioValue / len(buy_list)
    for stk in buy_list:
        # 停牌或是还没有上市等原因不能交易
        if np.isnan(account.referencePrice[stk]) or account.referencePrice[stk] == 0:
            continue
        else:
            change[stk] = perCapital/account.referencePrice[stk] - account.valid_secpos.get(stk, 0)
    
    # 根据仓位变化大小生序排列，即先卖后买            
    for stk in sorted(change, key=change.get):
        order(stk, change[stk])

```
