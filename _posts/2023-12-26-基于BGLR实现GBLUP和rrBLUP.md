---
layout:     post
title:      "「GS」基于BGLR实现GBLUP和rrBLUP"
subtitle:   "使用R包BGLR构建GBLUP和rrBLUP模型"
date:       2023-12-26 12:00:00
author:     "云伴风行"
header-img: "img/blog-life4.jpg"
header-mask: 50%
catalog: true
mathjax: true
tags:
    - BGLR
    - rrBLUP
    - GBLUP
---

> 本文使用 **[BGLR](https://github.com/gdlc/BGLR-R)** 实现GBLUP与rrBLUP模型的应用  
> 本文使用R版本：R version 4.3.1

[BGLR](https://github.com/gdlc/BGLR-R)是一款功能丰富的全基因组选择工具包，实现大量贝叶斯回归模型。

主要参数：

- y：表型向量（可以是"gaussian"和"ordinal", 用response type指定），可以有缺失值
- ETA: 线性预测器，默认包括截距项，其他需要用户指定
- nIter：控制采用迭代次数
- burnln：控制丢弃的样本数量
- thin： 控制用于计算后验均值的抽样间隔
- saveAt：输出结果保存路径（包括前缀）

在这里主要讨论一下如何使用BGLR实现连续变量的GBLUP和RRBLUP模型。对于连续型变量的线性预测模型：

$$
y=\eta+\varepsilon
$$

其中y为响应变量，$\varepsilon$为残差项，$\eta$为线性预测器：

$$
\eta=1\mu+\sum_{j}^{J}X_j\beta_j+\sum_{l}^L\mu_l
$$

其中，$μ$是截距，$X_j$是预测变量的设计矩阵，$β_j$是与$X_j$列相关联的效应向量，$μ_l$是随机效应向量。BGLR支持不同的先验分布类型，包括固定效应(Flat)、高斯分布(BRR)、缩放t分布(BayesA)、双指数分布(BL)、高斯混合分布(BayesC)和缩放t混合分布(BayesB)。

![BGLR](/img/data/BGLR.png)

下面使用BGLR分别构建GBLR和rrBLUP模型
```R
# 加载数据小麦599
rm(list = ls())
if (!require("BGLR"))install.packages('BGLR')
library(BGLR)
data(wheat)
X=wheat.X
Y=wheat.Y
```
构建rrBLUP模型，这里用到小麦599的SNP数据X，模型选择BRR。返回值中`fmRR$ETA[[1]]$b`是标记效应值，对于rrBLUP模型我们可以根据基因型矩阵和标记效应值计算个体估计育种：

$$
GEBV_j=\sum_j^p{X_{ij}}\hat\beta_j
$$

```R
RRETA=list(list(X=X, model="BRR"))
set.seed(666)
fmRR <- BGLR(y=Y[,1], ETA = RRETA, nIter = 12000, burnIn = 2000, thin = 10, saveAt = "./fmRR_")
# 估计育种值计算
GEBV.RR = X%*%fmRR$ETA[[1]]$b
# 拟合相关性
> cor(fmRR$yHat, Y[,1])
[1] 0.8178176
```
在BGLR中构建GBLUP模型选择RKHS模型，其中`fmG$ETA[[1]]$u`为估计育种值。
```R
Z = scale(X, center = TRUE, scale = TRUE)
G = tcrossprod(Z)/ncol(Z)

GETA = list(list(K=G, model="RKHS"))
fmG <- BGLR(y=Y[,1], ETA = GETA, nIter = 12000, burnIn = 2000, thin = 10, saveAt = "./fmG_")
# 估计育种值计算
GEBV.G = fmG$ETA[[1]]$u
# 拟合相关性
> cor(fmG$yHat, Y[,1])
[1] 0.8167722
```
同时拟合基因组关系矩阵和标记，因为这里包含两个模型，返回值中`fmRRG$ETA`包含BRR和RKHS的结果。
```R
RRGETA = list(
  list(X=X, model="BRR"),
  list(K=G, model="RKHS")
)
fmRRG <- BGLR(y=Y[,1], ETA=RRGETA, nIter = 12000, burnIn = 2000, thin = 10, saveAt = "./fmRRG_")
# 估计育种值
GEBV.GRR = X%*%fmRRG$ETA[[1]]$b + fmRRG$ETA[[2]]$u
# 输出拟合相关性结果
> cor(fmRRG$yHat, Y[,1])
[1] 0.8222919
```
就该性状而言，rrBLUP和GBLUP方法拟合结果区别不大。另外可以选择交叉验证，对表型赋予缺失值查看预测的准确性。

> 参考：  
> Paulino Pérez, Gustavo de los Campos, Genome-Wide Regression and Prediction with the BGLR Statistical Package, *Genetics*, Volume 198, Issue 2, 1 October 2014, Pages 483–495, https://doi.org/10.1534/genetics.114.164442