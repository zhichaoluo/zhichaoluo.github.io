---
layout: post
title: "使用Python进行线性回归"
author: "yphuang"
date: 2016-03-14
categories: blog
tags: [Python,线性回归,sklearn]

---



线性回归是最简单同时也是最常用的一个统计模型。线性回归具有结果易于理解，计算量小等优点。如果一个简单的线性回归就能取得非常不错的预测效果，那么就没有必要采用复杂精深的模型了。

今天，我们一起来学习使用Python实现线性回归的几种方法：

+ 通过公式编写矩阵运算程序；
+ 通过使用机器学习库sklearn;
+ 通过使用statmodels库。

这里，先由简至繁，先使用sklearn实现，再讲解矩阵推导实现。

##  1.使用scikit-learn进行线性回归

### 设置工作路径


```python
# 
import os
os.getcwd()
os.chdir('D:\my_python_workfile\Project\Writting')
```

### 加载扩展包


```python
import pandas as pd
import numpy as np
import pylab as pl
import matplotlib.pyplot as plt
```

### 载入数据并可视化分析 

这里，为了简单起见，使用sklearn中自带的数据集鸢尾花数据`iris`进行分析，探索『花瓣宽』和『花瓣长』之间的线性关系。


```python
from sklearn.datasets import load_iris
# load data
iris = load_iris()
# Define a DataFrame
df = pd.DataFrame(iris.data, columns = iris.feature_names)
# take a look
df.head()
#len(df)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sepal length (cm)</th>
      <th>sepal width (cm)</th>
      <th>petal length (cm)</th>
      <th>petal width (cm)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5.1</td>
      <td>3.5</td>
      <td>1.4</td>
      <td>0.2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4.9</td>
      <td>3.0</td>
      <td>1.4</td>
      <td>0.2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4.7</td>
      <td>3.2</td>
      <td>1.3</td>
      <td>0.2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4.6</td>
      <td>3.1</td>
      <td>1.5</td>
      <td>0.2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5.0</td>
      <td>3.6</td>
      <td>1.4</td>
      <td>0.2</td>
    </tr>
  </tbody>
</table>
</div>




```python
# correlation
df.corr()

```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sepal length (cm)</th>
      <th>sepal width (cm)</th>
      <th>petal length (cm)</th>
      <th>petal width (cm)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>sepal length (cm)</th>
      <td>1.000000</td>
      <td>-0.109369</td>
      <td>0.871754</td>
      <td>0.817954</td>
    </tr>
    <tr>
      <th>sepal width (cm)</th>
      <td>-0.109369</td>
      <td>1.000000</td>
      <td>-0.420516</td>
      <td>-0.356544</td>
    </tr>
    <tr>
      <th>petal length (cm)</th>
      <td>0.871754</td>
      <td>-0.420516</td>
      <td>1.000000</td>
      <td>0.962757</td>
    </tr>
    <tr>
      <th>petal width (cm)</th>
      <td>0.817954</td>
      <td>-0.356544</td>
      <td>0.962757</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>




<center>
    <p><img src="https://raw.githubusercontent.com/yphuang/yphuang.github.io/master/img/iris_corr.png" align="center"></p>
</center>



```python
# rename the column name 

df.columns = ['sepal_length','sepal_width','petal_length','petal_width']
df.columns
```




    Index([u'sepal_length', u'sepal_width', u'petal_length', u'petal_width'], dtype='object')




```python
plt.matshow(df.corr())
```


<center>
    <p><img src="https://raw.githubusercontent.com/yphuang/yphuang.github.io/master/img/iris_lm_fit.png" align="center"></p>
</center>




由上面分析可知，花瓣长`sepal length`和花瓣宽`septal width`有着非常显著的相关性。

下面，通过线性回归进一步进行验证。


```python
# save image
fig,ax = plt.subplots(nrows = 1, ncols = 1) 
ax.matshow(df.corr())
fig.savefig('./image/iris_corr.png')
```

### 建立线性回归模型


```python
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error

lr = LinearRegression()
X = df[['petal_length']]
y = df['petal_width']
lr.fit(X,y)
# print the result
lr.intercept_,lr.coef_


```




    (-0.3665140452167297, array([ 0.41641913]))




```python
# get y-hat
yhat = lr.predict(X = df[['petal_length']])

# MSE
mean_squared_error(df['petal_width'],yhat)

```


```python

# lm plot
plt.scatter(df['petal_length'],df['petal_width'])
plt.plot(df['petal_length'],yhat)

#save image
plt.savefig('./image/iris_lm_fit.png')
```

## 2.使用statmodels库


```python
#import statsmodels.api as sm
import statsmodels.formula.api as sm

linear_model = sm.OLS(y,X)

results = linear_model.fit()

results.summary()
```




<table class="simpletable">
<caption>OLS Regression Results</caption>
<tr>
  <th>Dep. Variable:</th>       <td>petal_width</td>   <th>  R-squared:         </th> <td>   0.967</td> 
</tr>
<tr>
  <th>Model:</th>                   <td>OLS</td>       <th>  Adj. R-squared:    </th> <td>   0.967</td> 
</tr>
<tr>
  <th>Method:</th>             <td>Least Squares</td>  <th>  F-statistic:       </th> <td>   4372.</td> 
</tr>
<tr>
  <th>Date:</th>             <td>Mon, 14 Mar 2016</td> <th>  Prob (F-statistic):</th> <td>2.55e-112</td>
</tr>
<tr>
  <th>Time:</th>                 <td>22:55:09</td>     <th>  Log-Likelihood:    </th> <td> -9.4520</td> 
</tr>
<tr>
  <th>No. Observations:</th>      <td>   150</td>      <th>  AIC:               </th> <td>   20.90</td> 
</tr>
<tr>
  <th>Df Residuals:</th>          <td>   149</td>      <th>  BIC:               </th> <td>   23.91</td> 
</tr>
<tr>
  <th>Df Model:</th>              <td>     1</td>      <th>                     </th>     <td> </td>    
</tr>
<tr>
  <th>Covariance Type:</th>      <td>nonrobust</td>    <th>                     </th>     <td> </td>    
</tr>
</table>
<table class="simpletable">
<tr>
        <td></td>          <th>coef</th>     <th>std err</th>      <th>t</th>      <th>P>|t|</th> <th>[95.0% Conf. Int.]</th> 
</tr>
<tr>
  <th>petal_length</th> <td>    0.3364</td> <td>    0.005</td> <td>   66.124</td> <td> 0.000</td> <td>    0.326     0.346</td>
</tr>
</table>
<table class="simpletable">
<tr>
  <th>Omnibus:</th>       <td>19.115</td> <th>  Durbin-Watson:     </th> <td>   0.855</td>
</tr>
<tr>
  <th>Prob(Omnibus):</th> <td> 0.000</td> <th>  Jarque-Bera (JB):  </th> <td>  22.606</td>
</tr>
<tr>
  <th>Skew:</th>          <td> 0.940</td> <th>  Prob(JB):          </th> <td>1.23e-05</td>
</tr>
<tr>
  <th>Kurtosis:</th>      <td> 3.292</td> <th>  Cond. No.          </th> <td>    1.00</td>
</tr>
</table>



## 3.使用公式推导

线性回归，即是使得如下目标函数最小化：

$$
\sum_{i = 1}^{n}(y_i - {x_{i}}^T\beta)^2
$$

使用最小二乘法，不难得到$$\beta$$的估计：

$$
\hat{\beta} = (\mathbf{X^{T}X})^{-1}\mathbf{X}^{T}y
$$

从而，我们可以根据此公式，编写求解$$\hat{\beta}$$的函数。


```python
from numpy import *

#########################
# 定义相应的函数进行矩阵运算求解。
def standRegres(xArr, yArr):
    xMat = mat(xArr)
    yMat = mat(yArr).T
    xTx = xMat.T * xMat
    if linalg.det(xTx) == 0.0:
        print "this matrix is singular, cannot do inverse!"
        return NA
    else :
        ws = xTx.I * (xMat.T * yMat)
        return ws
```


```python
# test
x0 = np.ones((150,1))
x0 = pd.DataFrame(x0)
X0 = pd.concat([x0,X],axis  = 1)
standRegres(X0,y)

```




    matrix([[-0.36651405],
            [ 0.41641913]])



结果一致。


## 参考文献

+ [python中的线性回归](http://xccds1977.blogspot.sg/2014/10/python_18.html)

+ [Data Science in Python](http://blog.yhat.com/posts/data-science-in-python-tutorial.html)

+ 『机器学习实战』
