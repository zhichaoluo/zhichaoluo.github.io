---
layout: post
title: "Python中的线性代数运算"
author: "yphuang"
date: "2016-03-10"
categories: blog
tags: [Python,线性代数]

---


# Python中的线性代数运算

这里，为了熟悉Python语言的特性，我们采用一种最原始的方式去定义线性代数运算的相关函数。

如果是真实应用场景，则直接使用**NumPy**的函数即可。

***

## 1.向量

### 创建一个向量

我们可以把Python中的向量理解为有限维空间中的点。


```python

height_weight_age = [70,170,40]
grades = [95,80,75,62]

```

### 向量运算


```python
#### 加法定义——两个向量
def vector_add(v,w):
    """add coresponding elements"""
    return [v_i + w_i 
                for v_i,w_i in zip(v,w)]

```


```python
#### 减法定义
def vector_substract(v,w):
    """substracts coresponding elements"""
    return [v_i - w_i
                   for v_i,w_i in zip(v,w)]
```


```python
#### 向量加法——多个向量(list of vectors)
####### method 1:
def vector_sum(vectors):
    """sums of all coresponding elements"""
    result = vectors[0]
    for vector in vectors[1:]:
        result = vector_add(result,vector)
    return result

######## mothod 2:
def vector_sum(vecotrs):
    return reduce(vector_add,vectors)

######## mothod 3:
from functools import partial

vector_sum = partial(reduce,vector_add)

```


```python
### 向量的数乘运算
def scalar_multiply(c,v):
    """c is a number,v is a vector"""
    return [c * v_i for v_i in v]


```


```python
### 向量的均值运算
def vector_mean(vectors):
    """compute the vector whose i-th element is the mean of 
    the i-th elements of the input vectors"""
    n = len(vecotrs)
    return scalar_multiply(1/n,vector_sum())

```


```python
### 向量的点乘
def dot(v,w):
    return sum(v_i * w_i 
                      for v_i,w_i in zip(v,w))


### 向量的平房和
def sum_of_squares(v):
    """v_1*v_1+v_2*v_2+...+v_n*v_n"""
    return dot(v,v)

### 向量的模
import math

def magnitude(v):
    return math.sqrt(sum_of_squares(v))


### 向量的距离
##### method 1:
def squared_distance(v,w):
    """"""
    return sum_of_squares(vector_substract(v,w))


##### method 2:
def distance(v,w):
    return magnitude(vector_substract(v,w))

##### method 3:
def distance(v,w):
    return math.sqrt(squared_distance(v,w))
```

## 2.矩阵

矩阵是一个二维的数字集合。我们可以通过**列表的列表**来表达一个矩阵，这样，内层列表是等长的，并且每个内层列表表达矩阵的一行。


```python
### 定义一个向量
A = [[1,2,3],
     [4,5,6]]

B = [[1,2],
     [3,4],
    [7,8]]
```


```python
### 获得矩阵的行数和列数
def shape(A):
    num_rows = len(A)
    num_cols = len(A[0]) if A else 0
    return num_rows,num_cols

### 提取某一行
def get_row(A,i):
    return A[i]


###提取某一列
def get_column(A,j):
    return [A_i[j]  # j-th element of row A_i
                   for A_i in A]  # for each row in A
```


```python
### 定制特殊矩阵生成函数：如单位矩阵
def make_matrix(num_rows,num_cols,entry_fn):
    """return a matrix whose (i,j)-th entry is entry_fn(i,j)"""
    return [[entry_fn(i,j) 
                    for j in range(num_cols)]
                           for i in range(num_rows)]


### 
def is_diagonal(i,j):
    return 1 if i==j else 0


make_matrix(5,5,is_diagonal)
```




    [[1, 0, 0, 0, 0],
     [0, 1, 0, 0, 0],
     [0, 0, 1, 0, 0],
     [0, 0, 0, 1, 0],
     [0, 0, 0, 0, 1]]



## 参考文献

- 『Data Science from Scratch』
