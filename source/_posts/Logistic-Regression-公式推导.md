title: Logistic Regression 公式推导
date: 2018-06-11 12:06:37
mathjax: true
tags:

---


假设X是一个离散型随机变量，其取值集合为$\varkappa$

## 信息量

针对事件$x_0$，它发生的“惊讶程度”或不确定性，也就是信息量，通过下面的公式计算。

$$
I(x_0) = -log_2 \, p(x_0)
$$


## 熵（信息熵）

对于一个随机变量X而言，它的所有可能取值的信息量的期望$E[I(x)]$就称为熵

离散变量


$$
H(X)= E_p log_2 \frac {1}{p(x)}
	= -\sum_{x\in \varkappa} p(x) log_2 \, p(x)
$$

连续变量

$$
H(X) = - \int_{x \in \varkappa} \, p(x) \, log_2 p(x) \, dx
$$


## 相对熵(Relative Entropy) 

相对熵(Eelative Entropy)又称为KL散度（Kullback-Leibler divergence），KL距离，是两个随机分布间距离的度量。记作$D_KL(p||q)$

$$
\begin{aligned}
D_KL(p||q) &= E_p[log \, \frac {p(x)} {q(x)}] \\
&= \sum_{x \in \varkappa} \, p(x) \, log \, \frac {p(x)} {q(x)} \\
&= \sum_{x \in \varkappa} [p(x)\,log\,p(x) - p(x)\,log\,q(x)] \\
&= \sum_{x \in \varkappa} p(x)\,log\,p(x) - \sum_{x \in \varkappa} p(x)\,log\,q(x) \\
&= -H(p) - \sum_{x \in \varkappa} p(x)\,log\,q(x) \\
&= -H(p) + E_p [-log\,q(x)] \\
&= H_p(q) - H(p)
\end{aligned}
$$


> 上述公式中$log = log_2$，但是在代码实现中都以$e$为底数

假设$p$为真实概率分布，$q$为我们假设的概率分布

* 当$p=q$时，显然$D_KL(p||q)$=0
* $H(p)$ 表示对真实分布$p$所需要的最小编码bit数
* $H_q(p)$ 表示在$p$分布下，使用$q$进行编码所需要的bit数量
* $D_KL(p||q)$表示在真实分布$p$的前提下，使用$q$进行编码相对于$p$进行编码(最优编码)多出来的bit数


## 交叉熵(Cross Entropy)

$$
CEH(p, q) = E_p[-log\,q] = - \sum_{x \in \varkappa} p(x) log\,q(x) = H(p) + D_KL(p||q)
$$

在$p$为真实概率分布的前提下，$H(p)$可以看作常数，此时交叉熵和相对熵在行为上表现一致，都反映分布$p$和$q$之前的相似程度。所以一般在机器学习中，都直接优化交叉熵。


## 逻辑回归(Logistic Regression)

* p: 真实样本分布，服从参数为p的0-1分布
* q: 待估计的模型，服从参数为q的0-1分布

定义假设函数（hypothesis function）为

$$
h_{\vartheta} = \frac {1} {1+ e ^{ -{\vartheta}^T x }}
$$


逻辑回归本质就是2分类问题

$$
P(\hat{y}| x^{(i)};\vartheta) = \begin{cases}
	h_{\vartheta}(x^{(i)}) &\text{if } \hat{y} = 1  \\
   1- h_{\vartheta}(x^{(i)}) &\text{if } \hat{y} = 0
\end{cases}
$$

上述公式可以写为更一般的形式

$$
P(\hat{y}| x^{(i)};\vartheta) = 	\hat{y} \, h_{\vartheta}(x^{(i)}) + (1- \hat{y}) (1- h_{\vartheta}(x^{(i)}))
$$


带入至交叉熵公式中

$$
\begin{aligned}
Loss(\hat{y}, y) \\
&= CEH(p, q) \\
&= - p(\hat{y})\,log\,q(y) \\
&= - [P_p(\hat{y}=1)]\,logP_q(\hat{y}=1) + P_p(\hat{y}=0)\,log\,P_q(\hat{y}=0)] \\
&= -[\hat{y}\,log\,h_{\vartheta}(x)+(1-\hat{y})\,log\,(1-h_{\vartheta}(x))]
\end{aligned}
$$


对于m个样本取均值

$$
\frac {1}{m} \sum_{i=1}^{m}[y^{(i)} h_{\vartheta}(x)^{(i)} + (1-y^{(i)}) (1-h_{\vartheta}(x)^{(i)})]
$$

## 验证

```python
import tensorflow as tf
import numpy as np
lables = np.array([0,1,1,1,0], dtype=np.float32)
predictions = np.array([0.5,0.7,0.8,0.9,0.2], dtype=np.float32)

def sigmoid_corss_entropy(lables, predictions):
    logits = 1 / (1 + np.exp(-predictions))
    ces =  - lables * np.log(logits) - (1 - lables) * np.log(1-logits)
    return ces

np_sigmoid_corss_entropy = sigmoid_corss_entropy(lables, predictions).mean()

with tf.Session() as sess:
    tf_sigmoid_cross_entropy_tensor = tf.losses.sigmoid_cross_entropy(multi_class_labels=tf.constant(lables), logits=tf.constant(predictions))
    tf_sigmoid_cross_entropy = sess.run(tf_sigmoid_cross_entropy_tensor)

print(np_sigmoid_corss_entropy) #0.577531
print(tf_sigmoid_cross_entropy) #0.577531
```


