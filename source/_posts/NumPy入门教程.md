title: NumPy入门教程
date: 2018-5-11 11:43:55
tags: python
categories: python
comments: true
mathjax: true
---


## 快速入门教程

### 前提条件

在开始本教程之前，你需要有一定的Python基础。如果你想要回顾一下Python相关的知识点，你可以看一下[这份教程](http://docs.python.org/tut/)。

如果你希望运行本教程中的示例，那么需要在您机器上安装一些软件。有关说明，请参阅[http://scipy.org/install.html](http://scipy.org/install.html)。

### 基础

NumPy的主要对象是齐次多维数组。它是一个元素的表（元素通常是数字），所有的元素拥有相同的类型，可以被一个正整数元组来索引。在NumPy中维度称之为***axis（轴）***。

例如，在3D空间中的一个坐标点`[1, 2, 1]`拥有一个axis。这个axis拥有3个元素，所以我们说它的长度是3。在下面的例子中，有2个axis。第一个axis的长度是2，第二个axis的长度是3。



```python
[[ 1., 0., 0.],
[ 0., 1., 2.]]
```




    [[1.0, 0.0, 0.0], [0.0, 1.0, 2.0]]



NumPy的数组class称之为`ndarray`。它还有另外一个别名：`array`。注意`numpy.array`与标准Python库中的`array.array`不一样，标准库中的`array`只可以操作以为数组，并且只能提供少量的方法。`ndarray`更重要的一些属性如下：

**ndarray.ndim**

    数组的axis(维度)数量
    
**ndarray.shape**

    数组的维度。这是一个整数类型的元组，指示了数组在每个维度下的尺寸信息。对于一个n行m列的矩阵来说，它的`shape`是`(n,m)`。因此`shape`元组的长度，也是axis的数量，即`ndim`。
    
**ndarray.size**

    数组的元素总数。值等于`shape`中的元素的乘积。
    
**ndarray.dtype**

    一个描述数组中元素类型的对象。可以使用标准的Python类型创建或指定dtype。另外，也可以使用NumPy自己提供的一些类型。例如`numpy.int32`,`numpy.int16`和`numpy.float64`。
    
**ndarray.itemsize**

    数组中每个元素占用的bytes大小。例如，一个数组的元素类型为`float64`，它的`itemsize`就是8(=64/8)，另一个数组的元素类型为`complex32`的`itemsize`值为4(=32/8)。这个值相当于`ndarray.dtype.itemsize`。
    
**ndarray.data**

    该缓冲区包含数组的实际元素。通常，我们不需要使用此属性，因为我们将使用索引来访问数组中的元素。


#### 一个例子


```python
import numpy as np
a = np.arange(15).reshape(3, 5)
a
```




    array([[ 0,  1,  2,  3,  4],
           [ 5,  6,  7,  8,  9],
           [10, 11, 12, 13, 14]])




```python
a.shape
```




    (3, 5)




```python
a.ndim
```




    2




```python
a.dtype.name
```




    'int64'




```python
a.itemsize
```




    8




```python
a.size
```




    15




```python
type(a)
```




    numpy.ndarray




```python
b = np.array([6, 7, 8])
b
```




    array([6, 7, 8])




```python
type(b)
```




    numpy.ndarray



#### 数组的创建

有几种可以创建数组的方式。

例如，你可以通过使用`array`方法，从一个标准的Python列表或元组来创建一个numpy数组。数组的类型由序列中元素的类型自动推导得出。


```python
import numpy as np
a = np.array([2,3,4])
a
```




    array([2, 3, 4])




```python
a.dtype
```




    dtype('int64')




```python
b = np.array([1.2, 3.5, 5.1])
b.dtype
```




    dtype('float64')



在调用`array`方法来创建数组时，有一种常见的错误，就是在方法中传入了多个数字，而不是通过传入一个包含一组数字的list作为参数。

```
a = np.array(1,2,3,4)   # WRONG
a = np.array([1,2,3,4]) # RIGHT
```

`array`函数将序列的序列转换为二维数组，将序列的序列的序列转换成3维数组，等等。


```python
b = np.array([(1.5,2,3),(4,5,6)])
b
```




    array([[ 1.5,  2. ,  3. ],
           [ 4. ,  5. ,  6. ]])



数组的类型也可以在创建的时候，显式的指定：


```python
c = np.array([[1,2],[3,4]],dtype=complex)
c
```




    array([[ 1.+0.j,  2.+0.j],
           [ 3.+0.j,  4.+0.j]])



通常，数组的元素在初始状态下是未知的，但尺寸已知。因此，NumPy提供了一些方法来创建以初始化占位符填充的数组。这最大限度地减少了增加数组的开销，这是一项昂贵的操作。

方法`zeros`创建一个全部由0填充的数组，方法`ones`创建一个全部由1填充的数组，方法`empty`创建了一个全部由随机的数字填充的数组，随机数的值取决于内存当前的状态。默认情况下，创建出来的数组类型为`folat64`。


```python
np.zeros((3,4))
```




    array([[ 0.,  0.,  0.,  0.],
           [ 0.,  0.,  0.,  0.],
           [ 0.,  0.,  0.,  0.]])




```python
np.ones((2,3,4), dtype=np.int16) # dtype可以被指定
```




    array([[[1, 1, 1, 1],
            [1, 1, 1, 1],
            [1, 1, 1, 1]],
    
           [[1, 1, 1, 1],
            [1, 1, 1, 1],
            [1, 1, 1, 1]]], dtype=int16)




```python
np.empty((2,3)) # 未初始化，输出可能不同
```




    array([[ 0.,  0.,  0.],
           [ 0.,  0.,  0.]])



为了创建数字序列，NumPy提供了一个类似于`range`的返回数组而不是列表的函数。


```python
np.arange(10, 30, 5)
```




    array([10, 15, 20, 25])




```python
np.arange(0, 2, 0.3) # 可以接受float类型的参数
```




    array([ 0. ,  0.3,  0.6,  0.9,  1.2,  1.5,  1.8])



当`arange`与浮点参数一起使用时，由于有限的浮点精度，通常不可能预测获得的元素数量。出于这个原因，通常最好使用函数`linspace`来接收我们想要的元素数量作为参数，而不是步长：


```python
from numpy import pi
np.linspace(0, 2, 9)  # 创建9个数字，均匀分布在0到2之间
```




    array([ 0.  ,  0.25,  0.5 ,  0.75,  1.  ,  1.25,  1.5 ,  1.75,  2.  ])




```python
x = np.linspace( 0, 2*pi, 100)
f = np.sin(x)
```

#### 打印数组

当你打印一个数组时，NumPy以一种类似嵌套列表的形式来展示，同时具有以下布局：

- 最后一个axis从左向右打印
- 倒数第二个axis从上到下打印
- 其余的也是从上到下打印的，每个切片与下一个由空行分开。

然后将一维数组打印为行，将二维数组作为矩阵，将三维数组作为矩阵列表。


```python
a = np.arange(6)  # 一维数组
print(a)
```

    [0 1 2 3 4 5]



```python
b = np.arange(12).reshape(4,3)  # 二维数组
print(b)
```

    [[ 0  1  2]
     [ 3  4  5]
     [ 6  7  8]
     [ 9 10 11]]



```python
c = np.arange(24).reshape(2,3,4)  # 三维数组
print(c)
```

    [[[ 0  1  2  3]
      [ 4  5  6  7]
      [ 8  9 10 11]]
    
     [[12 13 14 15]
      [16 17 18 19]
      [20 21 22 23]]]


如果数组太大而无法打印，NumPy将自动跳过数组的中心部分并仅打印角点：


```python
print(np.arange(10000))
```

    [   0    1    2 ..., 9997 9998 9999]



```python
print(np.arange(10000).reshape(100,100))
```

    [[   0    1    2 ...,   97   98   99]
     [ 100  101  102 ...,  197  198  199]
     [ 200  201  202 ...,  297  298  299]
     ..., 
     [9700 9701 9702 ..., 9797 9798 9799]
     [9800 9801 9802 ..., 9897 9898 9899]
     [9900 9901 9902 ..., 9997 9998 9999]]


要禁用此行为并强制NumPy打印整个数组，可以使用`set_printoptions`更改打印选项。


```python
np.set_printoptions(threshold=np.nan)
```

#### 基本操作

数组上的算术运算符应用于元素。一个新的数组被创建并填充结果。


```python
a = np.array([20, 30, 40, 50])
```


```python
b = np.arange(4)
b
```




    array([0, 1, 2, 3])




```python
c = a-b
c
```




    array([20, 29, 38, 47])




```python
b**2
```




    array([0, 1, 4, 9])




```python
10*np.sin(a)
```




    array([ 9.12945251, -9.88031624,  7.4511316 , -2.62374854])




```python
a<35
```




    array([ True,  True, False, False], dtype=bool)



不像其他的矩阵语言那样，`*`操作符在NumPy中是元素间的乘法。矩阵乘法可以使用`dot`方法来实现:


```python
A = np.array([[1,1],[0,1]])
B = np.array([[2,0],[3,4]])
A*B    # 元素间的乘积
```




    array([[2, 0],
           [0, 4]])




```python
A.dot(B)   # 矩阵乘法
```




    array([[5, 4],
           [3, 4]])




```python
np.dot(A, B) # 矩阵乘法的另一种实现
```




    array([[5, 4],
           [3, 4]])



一些例如`+=`和`-=`的操作符，实现的方式是通过修改现有的矩阵而不是创建新的矩阵。


```python
a = np.ones((2,3), dtype=int)
b = np.random.random((2,3))
a *= 3
a
```




    array([[3, 3, 3],
           [3, 3, 3]])




```python
b += a
b
```




    array([[ 3.05432455,  3.59941571,  3.65058751],
           [ 3.85091779,  3.45890823,  3.55943444]])




```python
a += b # b 不会自动的转型成为 integer 类型
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-87-3054fce39e6f> in <module>()
    ----> 1 a += b
    

    TypeError: Cannot cast ufunc add output from dtype('float64') to dtype('int64') with casting rule 'same_kind'


在使用不同类型的数组时，结果数组的类型对应于更一般或精确的数组（称为向上转型）。


```python
a = np.ones(3, dtype=np.int32)
b = np.linspace(0, pi, 3)
b.dtype.name
```




    'float64'




```python
c = a+b
c
```




    array([ 1.        ,  2.57079633,  4.14159265])




```python
c.dtype.name
```




    'float64'




```python
d = np.exp(c*1j)
d
```




    array([ 0.54030231+0.84147098j, -0.84147098+0.54030231j,
           -0.54030231-0.84147098j])




```python
d.dtype.name
```




    'complex128'



许多一元运算，例如计算数组中所有元素的总和，都是作为`ndarray`类的方法来实现的。


```python
a = np.random.random((2,3))
a
```




    array([[ 0.48681264,  0.52685408,  0.53980305],
           [ 0.27958753,  0.55125855,  0.70834892]])




```python
a.sum()
```




    3.0926647737313067




```python
a.min()
```




    0.27958753466020847




```python
a.max()
```




    0.70834891569018965



默认情况下，这些操作适用于数组，就好像它是数字列表一样，无论其形状如何。但是，通过指定`axis`参数，可以沿着数组的指定轴(axis)应用操作：


```python
b = np.arange(12).reshape(3,4)
b
```




    array([[ 0,  1,  2,  3],
           [ 4,  5,  6,  7],
           [ 8,  9, 10, 11]])




```python
b.sum(axis=0)    # 每一列的和
```




    array([12, 15, 18, 21])




```python
b.min(axis=1)    # 每一行的最小值
```




    array([0, 4, 8])




```python
b.cumsum(axis=1)   # 每行的累加值
```




    array([[ 0,  1,  3,  6],
           [ 4,  9, 15, 22],
           [ 8, 17, 27, 38]])



#### 通用方法

NumPy提供了一些常见的数学运算方法，例如sin，cos和exp。在NumPy中，这些方法被称作"通用方法"(`ufunc`)。在NumPy中，这些方法操作在数组中的每个元素上，产生一个数组作为输出。


```python
B = np.arange(3)
B
```




    array([0, 1, 2])




```python
np.exp(B)
```




    array([ 1.        ,  2.71828183,  7.3890561 ])




```python
np.sqrt(B)
```




    array([ 0.        ,  1.        ,  1.41421356])




```python
C = np.array([2., -1., 4.])
np.add(B, C)
```




    array([ 2.,  0.,  6.])



#### 索引，切片和迭代

**一维**数组可以像Python中的list或其他序列一样进行索引、切片和迭代操作。


```python
a = np.arange(10)**3
a
```




    array([  0,   1,   8,  27,  64, 125, 216, 343, 512, 729])




```python
a[2]
```




    8




```python
a[2:5]
```




    array([ 8, 27, 64])




```python
a[:6:2] = -1000
```


```python
a
```




    array([-1000,     1, -1000,    27, -1000,   125,   216,   343,   512,   729])




```python
a[::-1]
```




    array([  729,   512,   343,   216,   125, -1000,    27, -1000,     1, -1000])




```python
for i in a:
    print(i**(1/3.))
```

    nan
    1.0
    nan
    3.0
    nan
    5.0
    6.0
    7.0
    8.0
    9.0


    /usr/local/Homebrew/Cellar/python3/3.6.0/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages/ipykernel_launcher.py:2: RuntimeWarning: invalid value encountered in power
      


**多维**数组每个轴（axis）都有一个索引。这些索引以逗号分隔的元组给出：


```python
def f(x,y):
    return 10*x + y

b = np.fromfunction(f,(5,4),dtype=int)
b
```




    array([[ 0,  1,  2,  3],
           [10, 11, 12, 13],
           [20, 21, 22, 23],
           [30, 31, 32, 33],
           [40, 41, 42, 43]])




```python
b[2,3]
```




    23




```python
b[0:5, 1] # 输出第2列的每一行的元素
```




    array([ 1, 11, 21, 31, 41])




```python
b[:, 1] # 与上一步操作等价
```




    array([ 1, 11, 21, 31, 41])




```python
b[1:3, :] # 输出第2和第3行的每个列元素
```




    array([[10, 11, 12, 13],
           [20, 21, 22, 23]])



当提供的索引数量少于axis的数量时，缺失的索引被视为完整的切片：


```python
b[-1] # 输出最后一行。相当于 b[-1,:]
```




    array([40, 41, 42, 43])



表达式`b[i]`的这种表示形式，意味着在`i`后面还有多个`:`，`:`的数量取决于剩余的axis数量。NumPy也允许你使用`...`来表示这一形式：`b[i,...]`。

**点**(`...`)表示产生完整索引元组所需要的冒号。例如，如果`x`是一个5轴数组，那么：

- `x[1,2,...]`等价于`x[1,2,:,:,:]`，
- `x[...,3]`等价于`x[:,:,:,:,3]`，
- `x[4,...,5,:]`等价于`x[4,:,:,5,:]`。


```python
# 一个3D数组（由两个2D数组粘贴而成）
c = np.array([[[0, 1, 2],
              [10, 12, 13]],
              [[100,101,102],
              [110,112,113]]
             ])
c.shape
```




    (2, 2, 3)




```python
c[1,...]  # 相当于 c[1,:,:] 或 c[1]
```




    array([[100, 101, 102],
           [110, 112, 113]])




```python
c[...,2]  # 相当于 c[:,:,2]
```




    array([[  2,  13],
           [102, 113]])



**迭代**多维数组是相对于第一个axis完成的：


```python
for row in b:
    print(row)
```

    [0 1 2 3]
    [10 11 12 13]
    [20 21 22 23]
    [30 31 32 33]
    [40 41 42 43]


但是，如果想要对数组中的每个元素执行操作，可以使用`flat`属性，该属性是数组中所有元素的迭代器：


```python
for element in b.flat:
    print(element)
```

    0
    1
    2
    3
    10
    11
    12
    13
    20
    21
    22
    23
    30
    31
    32
    33
    40
    41
    42
    43


### Shape操作

#### 改变一个array的shape

一个数组的形状由这个数组每个轴上的元素数量给出：


```python
a = np.floor(10*np.random.random((3,4)))
a
```




    array([[ 8.,  3.,  6.,  4.],
           [ 7.,  5.,  7.,  7.],
           [ 3.,  1.,  8.,  8.]])




```python
a.shape
```




    (3, 4)



数组的形状可以通过各种命令进行更改。请注意，以下三个命令都返回一个修改后的数组，但都没有改变原数组：


```python
a.ravel()  # 返回展开的数组
```




    array([ 8.,  3.,  6.,  4.,  7.,  5.,  7.,  7.,  3.,  1.,  8.,  8.])




```python
a.reshape(6,2)  # 返回一个改变了shape的数组
```




    array([[ 8.,  3.],
           [ 6.,  4.],
           [ 7.,  5.],
           [ 7.,  7.],
           [ 3.,  1.],
           [ 8.,  8.]])




```python
a.T  # 返回数组的转置
```




    array([[ 8.,  7.,  3.],
           [ 3.,  5.,  1.],
           [ 6.,  7.,  8.],
           [ 4.,  7.,  8.]])




```python
a.T.shape
```




    (4, 3)




```python
a.shape
```




    (3, 4)



由ravel()产生的数组元素的顺序通常是"C-style"的，即最右边的索引“变化最快”，因此[0,0]之后的元素是[0,1]。如果一个数组变形为其他形状，数组再次被视为"C-style"。NumPy通常创建按次顺序存储的数组，因此`ravel()`通常不需要复制数组，但如果数组是通过对另一个数组进行切片操作，或者使用不寻常的方式创建的，则可能需要复制它。函数`ravel()`和`reshape()`也可以通过使用可选参数来使用FORTRAN-style的数组，其中最左侧的索引更改速度最快。

`reshape`方法返回的结果是一个变形后的数组，而`ndarray.resize`方法会更改数组本身的形状：


```python
a
```




    array([[ 8.,  3.,  6.,  4.],
           [ 7.,  5.,  7.,  7.],
           [ 3.,  1.,  8.,  8.]])




```python
a.resize((2,6))
a
```




    array([[ 8.,  3.,  6.,  4.,  7.,  5.],
           [ 7.,  7.,  3.,  1.,  8.,  8.]])



如果在reshape操作中将尺寸参数传入-1，则会自动计算这一位置的尺寸：


```python
a.reshape(3,-1)
```




    array([[ 8.,  3.,  6.,  4.],
           [ 7.,  5.,  7.,  7.],
           [ 3.,  1.,  8.,  8.]])



#### 将不同的数组粘贴起来

多个数组可以按照不同的axis来粘贴起来：


```python
a = np.floor(10*np.random.random((2,2)))
a
```




    array([[ 9.,  9.],
           [ 8.,  6.]])




```python
b = np.floor(10*np.random.random((2,2)))
b
```




    array([[ 5.,  3.],
           [ 0.,  4.]])




```python
np.vstack((a,b))
```




    array([[ 9.,  9.],
           [ 8.,  6.],
           [ 5.,  3.],
           [ 0.,  4.]])




```python
np.hstack((a,b))
```




    array([[ 9.,  9.,  5.,  3.],
           [ 8.,  6.,  0.,  4.]])



函数`column_stack`将1D数组作为列堆叠到2D数组中。它相当于仅用于2D数组的`hstack`操作。


```python
from numpy import newaxis
np.column_stack((a,b))   # 仅作用于2D数组
```




    array([[ 9.,  9.,  5.,  3.],
           [ 8.,  6.,  0.,  4.]])




```python
a = np.array([4., 2.])
b = np.array([3., 8.])
np.column_stack((a,b))  # 返回一个2D数组
```




    array([[ 4.,  3.],
           [ 2.,  8.]])




```python
np.hstack((a,b))  # 得到不同的结果
```




    array([ 4.,  2.,  3.,  8.])




```python
a[:,newaxis]  # 这将得到一个2D列向量
```




    array([[ 4.],
           [ 2.]])




```python
np.column_stack((a[:,newaxis],b[:,newaxis]))
```




    array([[ 4.,  3.],
           [ 2.,  8.]])




```python
np.hstack((a[:,newaxis],b[:,newaxis]))  # 结果是一样的
```




    array([[ 4.,  3.],
           [ 2.,  8.]])



另一方面，函数`row_stack`相当于对任何数组进行`vstack`操作。一般情况下，对于具有两个以上维度的数组，`hstack`操作沿着它的第二个axis进行堆叠，`vstack`沿着它的第一个axis堆叠，`concatenate`沿着指定axis的方向进度堆叠。

**注意**

在复杂的情况下，`r_`和`c_`可用于通过沿着一个轴堆积数字来创建数组。他们允许使用表示范围的`:`操作符。


```python
np.r_[1:4,0,4]
```




    array([1, 2, 3, 0, 4])



当使用数组作为参数时，`r_`和`c_`与默认行为的`vstack`和`hstack`类似，可以通过可选参数指定所要连接的轴的序号。

#### 将一个数组拆分成几个较小的数组

使用`hsplit`，可以沿着水平轴来切割数组，或者通过指定返回的数组的形状来切割数组，或者通过指定需要分割的列来分割数组。


```python
a = np.floor(10*np.random.random((2,12)))
a
```




    array([[ 4.,  3.,  3.,  1.,  2.,  5.,  2.,  5.,  5.,  8.,  2.,  2.],
           [ 5.,  1.,  1.,  2.,  9.,  6.,  5.,  5.,  0.,  8.,  8.,  7.]])




```python
np.hsplit(a,3)  # 将a切分成3份
```




    [array([[ 4.,  3.,  3.,  1.],
            [ 5.,  1.,  1.,  2.]]), array([[ 2.,  5.,  2.,  5.],
            [ 9.,  6.,  5.,  5.]]), array([[ 5.,  8.,  2.,  2.],
            [ 0.,  8.,  8.,  7.]])]




```python
np.hsplit(a,(3,4)) # 沿着第3和第4列来切分数组
```




    [array([[ 4.,  3.,  3.],
            [ 5.,  1.,  1.]]), array([[ 1.],
            [ 2.]]), array([[ 2.,  5.,  2.,  5.,  5.,  8.,  2.,  2.],
            [ 9.,  6.,  5.,  5.,  0.,  8.,  8.,  7.]])]



`vspilt`沿着垂直轴进行分割，`array_split`允许指定沿着那个轴来进行分割。

### 副本和视图

当操作一个数组时，它们的数据有时会被复制到一个新的数组中，有时则不会。这通常会让新手感到困惑。下面是3个例子：

#### 完全没有复制

简单的赋值不会复制数组对象或数据。


```python
a = np.arange(12)
b = a    # 没有新的对象被创建
b is a   # a 和 b 是同一个ndarray对象的两个名字
```




    True




```python
b.shape = 3,4  # 改变a的shape
a.shape
```




    (3, 4)



Python将可变对象作为引用传递，所以函数调用不会执行复制操作。


```python
def f(x):
    print(id(x))
    
id(a)      # id 是一个对象的唯一标识
```




    4449897488




```python
f(a)
```

    4449897488


#### 视图或浅拷贝

不同的数组对象可以共享相同的数据。`view`函数创建一个新的数组对象，但它和原数组持有相同的数据。


```python
c = a.view()
c is a
```




    False




```python
c.base is a  # c 是 一个a数据所创建出来的视图
```




    True




```python
c.flags.owndata
```




    False




```python
c.shape = 2,6  # a的shape并不发生改变
a.shape
```




    (3, 4)




```python
c[0,4] = 1234  # a的数据发生改变
a
```




    array([[   0,    1,    2,    3],
           [1234,    5,    6,    7],
           [   8,    9,   10,   11]])



对一个数组进行切片操作，返回它的一个视图：


```python
s = a[ : , 1:3]  # 也可以被写作 s = a[:,1:3]
s[:] = 10  # s[:] 是一个s的视图。注意这里 s = 10 和 s[:] = 10 的区别
a
```




    array([[   0,   10,   10,    3],
           [1234,   10,   10,    7],
           [   8,   10,   10,   11]])



#### 深拷贝

`copy`方法可以构造数组以及数据的完整副本。


```python
d = a.copy()    # 一个由新数据构成的新的数组对象被创建了
d is a
```




    False




```python
d.base is a     # d 与 a 不共享任何东西
```




    False




```python
d[0,0] = 9999
a
```




    array([[   0,   10,   10,    3],
           [1234,   10,   10,    7],
           [   8,   10,   10,   11]])



#### 方法预览

这里有一个NumPy中各种类型的比较有用的方法列表。

- **数组创建**
  
  `arange`,`array`,`copy`,`empty`,`empty_like`,`eye`,`fromfile`,`fromfunction`,`identity`,`linspace`,`logspace`,`mgrid`,`ogrid`,`ones`,`ones_like`,`zeros`,`zeros_like`
    
- **转换**
    
    `ndarray.astype`,`atleast_1d`,`atleast_2d`,`atleast_3d`,`mat`
    
- **手法**

    `array_split`, `column_stack`, `concatenate`, `diagonal`, `dsplit, dstack`, `hsplit`, `hstack`, `ndarray.item`, `newaxis`, `ravel`, `repeat`, `reshape`, `resize`, `squeeze`, `swapaxes`, `take`, `transpose`, `vsplit`, `vstack`
    
- **问题**

    `all`, `any`, `nonzero`, `where`
    
- **排序**

    `argmax`, `argmin`, `argsort`, `max`, `min`, `ptp`, `searchsorted`, `sort`
    
- **操作**

    `choose`, `compress`, `cumprod`, `cumsum`, `inner`, `ndarray.fill`, `imag`, `prod`, `put`, `putmask`, `real`, `sum`
    
- **基本统计**

    `cov`,`mean`,`std`,`var`
    
- **基本线性代数**

    `cross`,`dot`,`outer`,`linalg.svd`,`vdot`


### 进阶

#### 广播机制

广播允许通用方法以有意义的方式处理形状不完全相同的输入。

广播第一法则是，如果所有的输入数组维度不都相同，一个“1”将被重复地添加在维度较小的数组上直至所有的数组拥有一样的维度。

广播第二法则确定长度为1的数组沿着特殊的方向表现地好像它有沿着那个方向最大形状的大小。对数组来说，沿着那个维度的数组元素的值理应相同。

应用广播法则之后，所有数组的大小必须匹配。更多细节可以从这个[文档](https://docs.scipy.org/doc/numpy/user/basics.broadcasting.html)找到。


### 花哨的索引和索引技巧

NumPy提供比常规Python序列更多的索引功能。正如我们前面看到的，除了通过整数和切片进行索引之外，还可以使用整数和布尔数组数组对索引进行索引。

#### 通过数组索引




```python
a = np.arange(12)**2          # 前12个方格
i = np.array([1,1,3,8,5])     # 一个索引数组
a[i]                          # 一个在位置i的元素
```




    array([ 1,  1,  9, 64, 25])




```python
j = np.array([[ 3, 4], [ 9, 7]])  # 一个二维索引数组
a[j]                              # 与j的shape相同
```




    array([[ 9, 16],
           [81, 49]])



当被索引数组`a`是多维的时，每一个唯一的索引数列指向`a`的第一维。以下示例通过将图片标签用调色版转换成色彩图像展示了这种行为。


```python
palette = np.array([[0,0,0],           # 黑
                    [255,0,0],         # 红
                    [0,255,0],         # 绿
                    [0,0,255],         # 蓝
                    [255,255,255]      # 白
                   ])
image = np.array([[0,1,2,0],           # 每个值对应调色板中的颜色
                  [0,3,4,0]
                 ])

palette[image]                         
```




    array([[[  0,   0,   0],
            [255,   0,   0],
            [  0, 255,   0],
            [  0,   0,   0]],
    
           [[  0,   0,   0],
            [  0,   0, 255],
            [255, 255, 255],
            [  0,   0,   0]]])



我们也可以给出不不止一维的索引，每一维的索引数组必须有相同的形状。


```python
a = np.arange(12).reshape(3,4)
a
```




    array([[ 0,  1,  2,  3],
           [ 4,  5,  6,  7],
           [ 8,  9, 10, 11]])




```python
i = np.array([[0,1],    # indices for the first dim of a
             [1,2]])

j = np.array([[2,1],    # indices for the second dim
             [3,3]])

a[i,j]                  # i 和 j必须拥有相同的shape
```




    array([[ 2,  5],
           [ 7, 11]])




```python
a[i,2]
```




    array([[ 2,  6],
           [ 6, 10]])




```python
a[:,j]
```




    array([[[ 2,  1],
            [ 3,  3]],
    
           [[ 6,  5],
            [ 7,  7]],
    
           [[10,  9],
            [11, 11]]])



当然，我们可以将`i`和`j`放入一个序列中（比如说一个列表），然后用列表进行索引。


```python
l = [i,j]
a[l]        # 相当于一个[i,j]
```




    array([[ 2,  5],
           [ 7, 11]])



但是，我们不能将`i`和`j`放进一个数组，因为这个数组将被解读为a的第一个维度的索引。


```python
s = np.array([i,j])
a[s]     # 结果不是我们想要的
```


    ---------------------------------------------------------------------------

    IndexError                                Traceback (most recent call last)

    <ipython-input-204-79ccae1d198c> in <module>()
          1 s = np.array([i,j])
    ----> 2 a[s]     # 结果不是我们想要的
    

    IndexError: index 3 is out of bounds for axis 0 with size 3



```python
a[tuple(s)]     # 与 a[i,j]相同
```




    array([[ 2,  5],
           [ 7, 11]])



另一个常用的数组索引用法是搜索时间序列最大值。


```python
time = np.linspace(20, 145, 5)    # 时间尺度
data = np.sin(np.arange(20)).reshape(5,4)  # 4个时间依赖序列
time
```




    array([  20.  ,   51.25,   82.5 ,  113.75,  145.  ])




```python
data
```




    array([[ 0.        ,  0.84147098,  0.90929743,  0.14112001],
           [-0.7568025 , -0.95892427, -0.2794155 ,  0.6569866 ],
           [ 0.98935825,  0.41211849, -0.54402111, -0.99999021],
           [-0.53657292,  0.42016704,  0.99060736,  0.65028784],
           [-0.28790332, -0.96139749, -0.75098725,  0.14987721]])




```python
ind = data.argmax(axis=0)    # 每个序列的最大值的索引
ind
```




    array([2, 0, 3, 1])




```python
time_max = time[ind]         # 时间序列对应的最大值
data_max = data[ind, range(data.shape[1])]  # => data[ind[0],0], data[ind[1],1]...

time_max
```




    array([  82.5 ,   20.  ,  113.75,   51.25])




```python
data_max
```




    array([ 0.98935825,  0.84147098,  0.99060736,  0.6569866 ])




```python
np.all(data_max == data.max(axis=0))
```




    True



你也可以使用数组索引作为目标来赋值：


```python
a = np.arange(5)
a
```




    array([0, 1, 2, 3, 4])




```python
a[[1,3,4]] = 0
a
```




    array([0, 0, 2, 0, 0])



然而，当一个索引列表包含重复时，赋值被多次完成，保留最后一次的值：


```python
a = np.arange(5)
a[[0,0,2]]=[1,2,3]
a
```




    array([2, 1, 3, 3, 4])



这足够合理，但是小心如果你想用Python的`+=`结构，可能结果并非你所期望：


```python
a = np.arange(5)
a[[0,0,2]] += 1
a
```




    array([1, 1, 3, 3, 4])



即使0在索引列表中出现两次，索引为0的元素仅仅增加一次。这是因为Python要求`a+=1`和`a=a+1`等同。

#### 通过布尔数组索引

当我们使用整数数组索引数组时，我们提供一个索引列表去选择。通过布尔数组索引的方法是不同的我们显式地选择数组中我们想要和不想要的元素。

我们能想到的使用布尔数组的索引最自然方式就是使用和原数组一样形状的布尔数组。


```python
a = np.arange(12).reshape(3,4)
b = a > 4
b                   # b 是一个和a形状相同的boolean数组
```




    array([[False, False, False, False],
           [False,  True,  True,  True],
           [ True,  True,  True,  True]], dtype=bool)




```python
a[b]                # 经过筛选后的1维数组
```




    array([ 5,  6,  7,  8,  9, 10, 11])



这个属性在赋值时非常有用：


```python
a[b] = 0            # 将a中所有比4大的元素赋值为0
a
```




    array([[0, 1, 2, 3],
           [4, 0, 0, 0],
           [0, 0, 0, 0]])



你可以参考曼德博集合示例看看如何使用布尔索引来生成[曼德博集合](http://en.wikipedia.org/wiki/Mandelbrot_set)的图像。


```python
import numpy as np
import matplotlib.pyplot as plt
def mandelbrot(h,w,maxit=20):
    """返回一个尺寸为（h,w）的曼德博分形图"""
    y,x = np.ogrid[ -1.4:1.4:h*1j, -2:0.8:w*1j]
    c = x+y * 1j
    z = c
    divtime = maxit + np.zeros(z.shape,dtype=int)
    
    for i in range(maxit):
        z = z**2 + c
        diverge = z*np.conj(z) > 2**2    # who is diverging
        div_now = diverge & (divtime == maxit)  # who is diverging now
        divtime[div_now] = i                    # note when
        z[diverge] = 2                          # avoid diverging too much
        
    return divtime

plt.imshow(mandelbrot(400,400))
plt.show()
```


![png](/img/18_05_11/output_194_0.png)


第二种通过布尔来索引的方法更近似于整数索引；对数组的每个维度我们给一个一维布尔数组来选择我们想要的切片。


```python
a = np.arange(12).reshape(3,4)
b1 = np.array([False,True,True])    # 第一维的筛选
b2 = np.array([True,False,True,False])  # 第二维的筛选

a[b1,:]       # 选择行
```




    array([[ 4,  5,  6,  7],
           [ 8,  9, 10, 11]])




```python
a[b1]         # 和上面相同
```




    array([[ 4,  5,  6,  7],
           [ 8,  9, 10, 11]])




```python
a[:,b2]       # 选择列
```




    array([[ 0,  2],
           [ 4,  6],
           [ 8, 10]])




```python
a[b1,b2]      # 一个奇怪的结果
```




    array([ 4, 10])



注意一维数组的长度必须和你想要切片的维度或轴的长度一致，在之前的例子中，`b1`是一个秩为1长度为三的数组(`a`的行数)，`b2`(长度为4)与`a`的第二秩(列)相一致。

#### ix_()函数

`ix_`函数可以为了获得多元组的结果而用来结合不同向量。例如，如果你想要用所有向量a、b和c元素组成的三元组来计算a+b*c：


```python
a = np.array([2,3,4,5])
b = np.array([8,5,4])
c = np.array([5,4,6,8,3])
ax,bx,cx = np.ix_(a,b,c)
ax
```




    array([[[2]],
    
           [[3]],
    
           [[4]],
    
           [[5]]])




```python
bx
```




    array([[[8],
            [5],
            [4]]])




```python
cx
```




    array([[[5, 4, 6, 8, 3]]])




```python
ax.shape, bx.shape, cx.shape
```




    ((4, 1, 1), (1, 3, 1), (1, 1, 5))




```python
result = ax + bx * cx
result
```




    array([[[42, 34, 50, 66, 26],
            [27, 22, 32, 42, 17],
            [22, 18, 26, 34, 14]],
    
           [[43, 35, 51, 67, 27],
            [28, 23, 33, 43, 18],
            [23, 19, 27, 35, 15]],
    
           [[44, 36, 52, 68, 28],
            [29, 24, 34, 44, 19],
            [24, 20, 28, 36, 16]],
    
           [[45, 37, 53, 69, 29],
            [30, 25, 35, 45, 20],
            [25, 21, 29, 37, 17]]])




```python
result[3,2,4]
```




    17




```python
a[3] + b[2] * c[4]
```




    17



你也可以实行如下简化：


```python
def ufunc_reduce(ufct, *vectors):
    vs = np.ix_(*vectors)
    r = ufct.identity
    for v in vs:
        r = ufct(r,v)
    return r
```

然后这样使用它：


```python
ufunc_reduce(np.add,a,b,c)
```




    array([[[15, 14, 16, 18, 13],
            [12, 11, 13, 15, 10],
            [11, 10, 12, 14,  9]],
    
           [[16, 15, 17, 19, 14],
            [13, 12, 14, 16, 11],
            [12, 11, 13, 15, 10]],
    
           [[17, 16, 18, 20, 15],
            [14, 13, 15, 17, 12],
            [13, 12, 14, 16, 11]],
    
           [[18, 17, 19, 21, 16],
            [15, 14, 16, 18, 13],
            [14, 13, 15, 17, 12]]])



这个reduce与ufunc.reduce(比如说add.reduce)相比的优势在于它利用了广播法则，避免了创建一个输出大小乘以向量个数的参数数组。

#### 用字符串索引

参加[结构化数组](https://docs.scipy.org/doc/numpy/user/basics.rec.html#structured-arrays)。

### 线性代数


继续前进，基本线性代数包含在这里。

#### 简单数组运算

参考numpy文件夹中的linalg.py获得更多信息


```python
import numpy as np
a = np.array([[1.0,2.0], [3.0, 4.0]])
print(a)
```

    [[ 1.  2.]
     [ 3.  4.]]



```python
a.transpose()
```




    array([[ 1.,  3.],
           [ 2.,  4.]])




```python
np.linalg.inv(a)
```




    array([[-2. ,  1. ],
           [ 1.5, -0.5]])




```python
u = np.eye(2) # unit 2x2 matrix; "eye" represents "I"  单位矩阵
u
```




    array([[ 1.,  0.],
           [ 0.,  1.]])




```python
j = np.array([[0.0, -1.0], [1.0, 0.0]])
np.dot(j, j)   # 矩阵乘法
```




    array([[-1.,  0.],
           [ 0., -1.]])




```python
np.trace(u)    # trace
```




    2.0




```python
y = np.array([[5.],[7.]])
np.linalg.solve(a, y)
```




    array([[-3.],
           [ 4.]])




```python
np.linalg.eig(j)
```




    (array([ 0.+1.j,  0.-1.j]),
     array([[ 0.70710678+0.j        ,  0.70710678-0.j        ],
            [ 0.00000000-0.70710678j,  0.00000000+0.70710678j]]))



```
Parameters:
    square matrix
Returns
    The eigenvalues, each repeated according to its multiplicity.
    The normalized (unit "length") eigenvectors, such that the
    column ``v[:,i]`` is the eigenvector corresponding to the
    eigenvalue ``w[i]`` .
```

### 技巧和提示

下面我们给出简短和有用的提示。

#### “自动”改变形状

更改数组的维度，你可以省略一个尺寸，它将被自动推导出来。


```python
a = np.arange(30)
a.shape = 2,-1,3  # -1 意味着 “无论需要什么”
a.shape
```




    (2, 5, 3)




```python
a
```




    array([[[ 0,  1,  2],
            [ 3,  4,  5],
            [ 6,  7,  8],
            [ 9, 10, 11],
            [12, 13, 14]],
    
           [[15, 16, 17],
            [18, 19, 20],
            [21, 22, 23],
            [24, 25, 26],
            [27, 28, 29]]])



#### 向量组合(stacking)

我们如何用两个相同尺寸的行向量列表构建一个二维数组？在MATLAB中这非常简单：如果`x`和`y`是两个相同长度的向量，你仅仅需要做`m=[x;y]`。在NumPy中这个过程通过函数`column_stack`、`dstack`、`hstack`和`vstack`来完成，取决于你想要在那个维度上组合。例如：


```python
x = np.arange(0,10,2)      # x = ([0,2,4,6,8])
y = np.arange(5)           # y = ([0,1,2,3,4])
m = np.vstack([x,y])       # m=([[0,2,4,6,8],
                           #     [0,1,2,3,4]])
m
```




    array([[0, 2, 4, 6, 8],
           [0, 1, 2, 3, 4]])




```python
xy = np.hstack([x,y])      # xy = ([0,2,4,6,8,0,1,2,3,4])
xy
```




    array([0, 2, 4, 6, 8, 0, 1, 2, 3, 4])



#### 直方图(histogram)

NumPy中`histogram`函数应用到一个数组返回一对变量：直方图数组和箱式向量。注意：`matplotlib`也有一个用来建立直方图的函数(叫作`hist`,正如matlab中一样)与NumPy中的不同。主要的差别是`pylab.hist`自动绘制直方图，而`numpy.histogram`仅仅产生数据。


```python
import numpy as np
import matplotlib.pyplot as plt
# 简历一个拥有10000个元素的正态分布的向量，方差为0.5^2，均值为2
mu, sigma = 2, 0.5
v = np.random.normal(mu,sigma,10000)
# 绘制分成50份的正态分布直方图
plt.hist(v, bins=50, normed=1)
plt.show()
```


![png](/img/18_05_11/output_232_0.png)



```python
# 用numpy计算直方图然后绘制它
(n, bins) = np.histogram(v, bins=50, normed=True)  # NumPy version (no plot)
plt.plot(.5*(bins[1:]+bins[:-1]), n)
plt.show()
```


![png](/img/18_05_11/output_233_0.png)


