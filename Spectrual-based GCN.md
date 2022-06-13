- [GCN 隐藏层通用框架](#gcn-隐藏层通用框架)
  - [输入](#输入)
  - [输出](#输出)
  - [前向传播](#前向传播)
  - [反向传播](#反向传播)
- [Weisfeiler-Lehman算法](#weisfeiler-lehman算法)
- [GCN数学原理](#gcn数学原理)
  - [GSP](#gsp)
    - [概述](#概述)
    - [傅里叶变换](#傅里叶变换)
  - [GCN](#gcn)
# GCN 隐藏层通用框架

## 输入
**特征矩阵**
$X_{N \times F^{0}}$
- $N$节点数  
- $F^{0}$节点特征数

**邻接矩阵**
$A_{N\times N}$

## 输出
$H^{l} = f(H^{l-1}, A)$
  
- $H^{l}$是一个$N \times F^{l}$的矩阵  
- $H^{0}=X$
- 不同模型，函数$f$的实现不同

## 前向传播

## 反向传播

# Weisfeiler-Lehman算法
>很多论文中会讲，从另一个角度来讲，GCN模型可以看作图上非常有名的Weisfeiler-Lehman算法的一种变形

# GCN数学原理

## GSP
>The Emerging Field of Signal Processing on Graphs (IEEE Signal
Processing Magazine, Volume: 30 , Issue: 3 , May 2013)
  
### 概述
- GSP使能够在图上进行卷积
- **在顶点域的卷积等价于在谱域的乘法**  

### 傅里叶变换
两个时域信号的叠加是将其值相加产生更强的气压(pressure)

## GCN
>Convolutional Neural Networks on Graphs with Fast Localized Spectral
Filtering (NIPS 2016)
  

