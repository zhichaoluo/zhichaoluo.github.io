---
layout: post
title: "广义线性模型中的Gauss Seidel 迭代算法实现"
author: "yphuang"
date: 2015-11-25
categories: blog
tags: [glm,Gauss Seidel算法,R]

---




## 数值模拟的算法迭代公式推导


$$
\begin{aligned}
  \begin{array}{rcl}
\because Y|X_i &\sim& Bernoulli(p(x_i;\beta)) \\
\therefore f(y_i;p_i) &=& {p_i}^{y_i}(1-p_i)^{1-y_i}  \\
f(y_1,y_2,...,y_n;p_i) &=& \prod_{i = 1}^{n}{p_i}^{y_i}(1-p_i)^{1 - y_i} \\
\therefore l(p_i;y_1,y_2,...,y_n) &=& \sum\left[ y_{i}log(p_i) + (1-y_i)log(1-p_i) \right] \\
\text{其中，} p_i &=& \frac{exp(\alpha + \beta' x_i)}{1+ exp(\alpha + \beta' x_i)} \\
1 - p_i &=& \frac{1}{1+ exp(\alpha + \beta' x_i)} \\
\text{Let } g(\mu_i) &=& log(\frac{p_i}{1-p_i}) \\
E(Y_i) = \mu_i &=& p_i \\
var(Y_i) &=&  p_i(1-p_i) \\
g(\mu_i) &=& {x_i}'\beta = \eta_i \\
l &=& \sum y_{i}log(\frac{p_i}{1-p_i}) + \sum log[1-p_i) = \sum y_{i}(\alpha + \beta' x_i) + \sum log(\frac{1}{1+ exp(\alpha + \beta' x_i)}] \\
\frac{\partial l}{\partial \beta_j} &=& \sum_{i = i}^{n}y_{i}x_{ij} - \sum_{i = 1}^{n} x_{ij} \frac{exp[\alpha + \beta' x_i]}{1 + exp[\alpha + \beta' x_i]} \\
\frac{\partial^2 l }{\partial {\beta_j}^2} &=& - \sum_{i = 1}^{n}\left[ \frac{(x_{ij})^2 exp[ \alpha + \beta' x_i ]}{[1 + exp(\alpha + \beta' x_i)]^2} \right] \\
\therefore {\beta_j}^{(m)} &=& {\beta_j}^{(m-1)} - \left[\frac{\partial^2 l(\beta)}{\partial {\beta_j}^2}  \right]^{-1} \frac{\partial l(\beta)}{\partial \beta_j}\\ 
&=& {\beta_j}^{(m-1)} + \left\{ \sum_{i = 1}^{n}(x_{ij})^2 \left[  \frac{exp[ \alpha + \beta' x_i ]}{[1 + exp(\alpha + \beta' x_i)]^2} \right] \right\}^{-1} \times \left[\sum_{i = i}^{n}y_{i}x_{ij} - \sum_{i = 1}^{n} x_{ij} \frac{exp[\alpha + \beta' x_i]}{1 + exp[\alpha + \beta' x_i]}\right]  
  \end{array}
\end{aligned}  
$$

## R代码实现


根据以上公式，代入迭代步骤，即可实现算法。


```r
##------数据模拟--------

library(MASS)
##mvrnorm()

##定义一个产生多元正态分布的随机向量协方差矩阵
Simu_Multi_Norm<-function(x_len, sd = 1, pho = 0.5){
  #初始化协方差矩阵
  V <- matrix(data = NA, nrow = x_len, ncol = x_len)
  
  #mean及sd分别为随机向量x的均值和方差
  
  #对协方差矩阵进行赋值pho(i,j) = pho^|i-j|
  for(i in 1:x_len){ ##遍历每一行
    for(j in 1:x_len){ ##遍历每一列
      V[i,j] <- pho^abs(i-j)
    }
  }
  
  V<-(sd^2) * V
  return(V)
}


##产生模拟数值自变量X
set.seed(123)
X<-mvrnorm(n = 200, mu = rep(0,10), Simu_Multi_Norm(x_len = 10,sd  = 1, pho = 0.5))

##产生模拟数值：响应变量y
beta<-c(1,2,0,0,3,0,0,0,-2,0)

#alpha<-0

#prob<-exp(alpha + X %*% beta)/(1+exp(alpha + X %*% beta))

prob<-exp( X %*% beta)/(1+exp( X %*% beta))

y<-rbinom(n = 200, size = 1,p = prob)

##产生model matrix
mydata<-data.frame(X = X, y = y)

#X<-model.matrix(y~., data = mydata)



##包含截矩项的系数
#b_real<-c(alpha,beta)
b_real<-beta

#define the log-likelihood function
loglikelihood<-function(X, y, b){
  linear_comb<-as.vector(X %*% b)
  ll<-sum(y*linear_comb) + sum(log(1/(1+exp(linear_comb))))
  return (ll)
}


##初始化系数
b0<-rep(0,length(b_real))

#b0<- b_real+rnorm(length(b_real), mean = 0, sd = 0.1)


##b1用于记录更新系数
b1<-b0

##b.best用于存放历史最大似然值对应系数
b.best<-b0

# the initial value of loglikelihood

ll.old<-loglikelihood(X = X,y = y, b = b0)


# initialize the difference between the two steps of theta
diff<-1  
#record the number of iterations
iter<-0
#set the threshold to stop iterations
epsi<-1e-10
#the maximum iterations  
max_iter<-10000
#初始化一个列表用于存放每一次迭代的系数结果
b_history<-list(data.frame(b0))

#初始化列表用于存放似然值
ll_list<-list(ll.old)



#-------Gauss-Seidel 迭代-------
while(diff > epsi & iter < max_iter){
  for(j in 1:length(b_real)){
    #对j循环，对每个系数最优化
    
    #线性部分
    linear_comb<-as.vector(X %*% b0)
    
    #分子
    nominator<-sum(y*X[,j] - X[,j] * exp(linear_comb)/(1+exp(linear_comb)))
    #分母,即二阶导部分
    denominator<-  -sum(X[,j]^2 * exp(linear_comb)/(1+exp(linear_comb))^2)
    #
    b0[j]<-b0[j] - nominator/denominator
    #更新似然值
    ll.new<- loglikelihood(X = X, y = y, b = b0)
    
    #     #若似然值有所增加，则将当前系数保存
    if(ll.new > ll.old){
      #更新系数
      b.best[j]<-b0[j]
    }
    
    #求差异
    diff<- abs((ll.new - ll.old)/ll.old)
    ll.old <- ll.new
    iter<- iter+1 
    b_history[[iter]]<-data.frame(b0)
    ll_list[[iter]]<-ll.old
    ##当达到停止条件时，跳出循环
    if(diff < epsi){
      break
    }
    
  }
  
  
}

b_hist<-do.call(rbind,b_history)
#b_hist

ll_hist<-do.call(rbind,ll_list)
#ll_hist

#
iter

##
ll.best<-max(ll_hist)
ll.best

##
b.best




##---------glm()验证-------

my_glm<-glm(y~0 + X.1 + X.2 + X.3+ X.4+ X.5+ X.6+ X.7+ X.8+ X.9+ X.10,
            data = mydata,family = binomial(link = "logit"))
summary(my_glm)

coeff_glm<-my_glm$coefficients

cbind(coeff_glm,b.best,b_real)
```

迭代结果如下：

$$

\begin{table}[ht]
\centering
\begin{tabular}{rrrr}
  \hline
 & coeff\_glm & b.best & b\_real \\ 
  \hline
X.1 & 1.18 & 1.18 & 1.00 \\ 
  X.2 & 1.91 & 1.91 & 2.00 \\ 
  X.3 & 0.20 & 0.20 & 0.00 \\ 
  X.4 & 0.04 & 0.04 & 0.00 \\ 
  X.5 & 2.68 & 2.68 & 3.00 \\ 
  X.6 & 0.18 & 0.18 & 0.00 \\ 
  X.7 & -0.07 & -0.07 & 0.00 \\ 
  X.8 & 0.22 & 0.22 & 0.00 \\ 
  X.9 & -2.63 & -2.63 & -2.00 \\ 
  X.10 & 0.42 & 0.42 & 0.00 \\ 
   \hline
\end{tabular}
\end{table}
$$


迭代206步收敛，系数结果非常接近R内部函数glm()运行的结果，甚至稍好于这一结果。


