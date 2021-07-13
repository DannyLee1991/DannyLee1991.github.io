title: NumPy快速入门
date: 2016-1-5 11:43:55
tags:
  - numpy
categories:
  - 算法
  - 工具包
mathjax: true
comments: true
---

## NumPy快速入门

### 为什么需要NumPy?

NumPy是python的一个矩阵类型，提供了大量的矩阵处理的函数。

由于其内部是通过**C语言**来实现，所以效率很高。

尽管声称是一个关于矩阵的库，NumPy实际上包含了两种基本的数据类型：数组和矩阵。二者在处理上稍有不同。在使用标准的python时，处理这两类数据，均需要使用循环语句，而使用NumPy则可以省去这些语句。

## 数组

### 数组相加

```python
>>> from numpy import array
>>> mm=array((1, 1, 1))
>>> pp=array((1, 2, 3))
>>> pp+mm
array([2, 3, 4])
```

如果使用常规的python，就需要循环来处理了。

### 数组的数乘运算

```python
>>> pp*2
array([2, 4, 6])
```

### 对每个元素求平方

```python
>>> pp**2
array([1, 4, 9])
```

### 可以像列表中一样访问数组里的元素：

```python
>>> pp[1]
1
```

NumPy中也支持多维数组：

```python
>>> jj = array([[1, 2, 3],[1, 1, 1]])
```

多维数组中的元素也可以像列表中一样访问：

```python
>>> jj[0]
array([1, 2, 3])
>>> jj[0][1]
2
```

也可以用矩阵方式访问：

```python
>>> jj[0, 1]
2
```

### 把两个数组乘起来的时候，两个数组的元素将对应相乘：

```python
>>> a1=array([1, 2, 3])
>>> a2=array([0.3, 0.2, 0.3])
>>> a1*a2
array([0.3, 0.4 0.9])
```

## 矩阵

### Numpy矩阵的引入

与数组一样，需要从NumPy中导入`matrix`或者`mat`模块(`mat`其实就是`matrix`的缩写)

```python
>>> from numpy import mat, matrix
>>> ss = mat([1, 2, 3])
>>> ss
matrix([[1, 2, 3]])
>>> mm = matrix([1, 2, 3])
>>> mm
matrix([[1, 2, 3]])
```
可以访问矩阵中的单个元素：

```python
>>> mm[0,1]
2
```

### 将python列表转换为NumPy矩阵

```python
>>> pyList = [5, 11, 1605]
>>> mat(pyList)
matrix([[   5,   11, 1605]]) 
```
### 矩阵乘法的运算

如果你不熟悉矩阵乘法的运算规则，请看[这里](./2015/11/29/线性代数01-矩阵乘法/)。

```python
>>> ss = mat([1, 2, 3])
>>> mm = matrix([1, 2, 3])
>>> mm*ss.T
matrix([[14]])
```

其中`ss.T`是将ss进行转置，因为做矩阵乘法运算的两个矩阵必须满足`左矩阵的列数与右矩阵的行数相等`的条件。

### 获取矩阵的行列数

```python
>>> from numpy import shape
>>> mm = matrix([1, 2, 3])
>>> shape(mm)
(1, 3)
```

### 两个矩阵对应元素相乘(multiply)

```python
>>> from numpy import multiply
>>> mm = matrix([1, 2, 3])
>>> ss = matrix([1, 2, 3])
>>> multiply(mm, ss)
matrix([[1, 4, 9]])
```

### 矩阵的排序

```python
>>> d = matrix([4, 5, 1])
>>> d.sort()
>>> d
matrix([[1, 4, 5]])
```

注意：该方法是原地排序（即排序后到结果占用原始的存储空间），所以如果希望保存数据的原始顺序，必须事先拷贝一份。也可以使用argsort()方法得到矩阵中每个元素的排序序号：

```python
>>> d = matrix([4, 5, 1])
>>> d.argsort()
matrix([[2, 0, 1]])
```

### 计算矩阵的平均值

```python
>>> d = matrix([4, 5, 1])
>>> d.mean()
3.3333333333333335
```

### 通过`:`去除矩阵中指定片段

python中的切片用法简单，功能强大，NumPy中的矩阵同样也支持该操作，比如在想在下面一个2x3的矩阵中取出第一行元素：

```python
>>> jj = mat([[1, 2, 3],[8, 8, 8]])
>>> shape(jj)
(2, 3)
>>> jj[1,:]
matrix([[8, 8, 8]])
```

也可以使用以下方法取出第一行第0列到第1列的元素：

```python
>>> jj[1,0:2]
matrix([[8, 8]])
```

### 更多

更多NumPy的相关用法，建议浏览完整的官方文档

[http://docs.scipy.org/doc/](http://docs.scipy.org/doc/)
