title: 【Python3教程 第2章】控制结构
tags:
  - Python
categories:
  - 工程开发
  - Python
comments: true
date: 2017-1-17 00:28:58
---

## 布尔与比较

### 布尔

Python中的另一种类型是布尔类型(Boolean)。布尔类型有两种值：**True**和**False**。它们可以通过比较值来创建，例如通过等号操作符`==`。

```python
>>> my_boolean = True
>>> my_boolean
True

>>> 2 == 3
False
>>> "hello" == "hello"
True
```

> 注意不要混淆**赋值**（一个等号）和**比较**（两个等号）操作符。

### 比较

另外一个比较操作符是不等于(**!=**)，如果两者相比不相等，则结果为**False**否则为**True**。

```python
>>> 1 != 1
False
>>> "eleven" != "seven"
True
>>> 2 != 10
True
```

Python也有比较两个数值（浮点数或整数）是否大于或小于另一个数值的运算。操作符分别是`>`和`<`。

```python
>>> 7 > 5
True
>>> 10 < 10
False
```

大于或等于的运算符是`>=`，小于或等于的运算符是`<=`。

与大于小于操作符类似，除了当两者相等的时候也返回**True**。

```python
>>> 7 <= 8
True
>>> 9 >= 9.0
True
```

> 大于等于和小于等于操作符也可以用于按字典排序模式比较字符串操作（字的字母顺序是基于它们的组成字母的字母顺序排序的）。

## if语句

你可以使用**if**语句来运行满足某个条件的语句。

如果表达式的结果是**True**，某些语句会执行。否则，它们不会被执行。

一个if语句看起来像是这样的：

```python
if expression:
   statements
```

> Python使用缩进(在行首的空白)的方式来限制代码块。其他语言，例如C语言，使用花括号来表示代码块，但在Python中缩进是强制性的，程序没有缩进就不会正常工作。正如你看到的**if**里的语句应该被缩进。

下面是一个**if**语句的例子：

```python
if 10 > 5:
   print("10 greater than 5")

print("Program ended")
```

判断表达式10是否大于5，如果是，则执行缩进的语句，输出`10 greater than 5`，然后执行后面没有缩进的语句。

执行结果：

```python
>>>
10 greater than 5
Program ended
>>>
```

注意**冒号**在**if**语句的末尾处。

> 由于程序包含多行代码，因此应将其创建为单独的代码文件来运行。

### if的嵌套

为了执行更严格的条件检查，**if**语句可以嵌套使用，嵌套在另一个if语句中。这意味着内部的**if**语句是外部的**if**语句的一部分，这是一种查看是否满足多个条件的方法。

例如：

```python
num = 12
if num > 5:
   print("Bigger than 5")
   if num <=47:
      print("Between 5 and 47")
```

执行结果：

```python
>>>
Bigger than 5
Between 5 and 47
>>>
```

## else语句

一个**else**语句跟在一个**if**语句后面，并且包含当**if**条件语句执行结果为**False**时执行的代码。

与**if**语句一样，内部的代码块也应该缩进。

```python
x = 4
if x == 5:
   print("Yes")
else:
   print("No")
```

执行结果:

```pyhton
>>> 
No
>>>
```

### if和else嵌套使用

你可以将**if**和**else**连接起来，来判断一系列的对错情况，例如：

```python
num = 7
if num == 5:
  print("Number is 5")
else: 
  if num == 11:
    print("Number is 11")
  else:
    if num == 7:
      print("Number is 7")
    else: 
      print("Number isn't 5, 11 or 7")
```

执行结果:

```python
>>>
Number is 7
>>> 
```

### elif语句

**elif**(`else if`的缩写)语句是`if`和`else`连起来使用的一个缩写形式。一系列的**elif**判断需要用一个**else**来收尾，用于判断**if**或**elif**以外的情况。

例如：

```python
num = 7
if num == 5:
   print("Number is 5")
elif num == 11:
   print("Number is 11")
elif num == 7:
   print("Number is 7")
else:
   print("Number isn't 5, 11 or 7")
```

执行结果:

```python
>>>
Number is 7
>>>
```

> 在其他的编程语言中，**elif**语句有多种名称，包括**else if**,**elseif**或者**elsif**。

## 布尔逻辑

当**if**语句依赖多个条件时，使用**布尔(Boolean)逻辑**可以用来生成更复杂的条件。

Python的布尔操作符是**and**，**or**和**not**。

### 布尔逻辑 and

**and**操作符接受两个参数，并且当且仅当两个参数都为**True**时，计算结果才为**True**。否则，计算结果是**False**。

```python
>>> 1 == 1 and 2 == 2
True
>>> 1 == 1 and 2 == 3
False
>>> 1 != 1 and 2 == 2
False
>>> 2 < 1 and 3 >  6
False
```

> Python用直白的单词来表示布尔逻辑，而其他大多数编程语言使用`&&`、`||`、`!`这些符号来表示。

### 布尔逻辑 or

**or**操作符也接受两个参数，当两个参数至少有一个为**True**时，计算结果就为**True**；否则，当两个参数都为**False**时，计算结果为**False**。

```python
>>> 1 == 1 or 2 == 2
True
>>> 1 == 1 or 2 == 3
True
>>> 1 != 1 or 2 == 2
True
>>> 2 < 1 or 3 >  6
False
```

### 布尔逻辑 not

不像其他我们已经看到的操作符那样，**not**操作符只接受一个参数，并且计算结果是取反。**True**的**not**结果是**False**，而且**not False**的计算结果是**True**。

```python
>>> not 1 == 1
False
>>> not 1 > 7
True
```

> 通过布尔逻辑，你可以在if的条件语句中连接多个条件。

## 运算符优先级

**运算符优先级**是编程中非常重要的概念。这个概念是数学中运算顺序的一种延伸（例如前面提到的乘法操作），用于包含多个其他的操作符，例如布尔逻辑。

下面的代码展示了`==`运算符拥有比`or`更高的优先级：

```python
>>> False == False or True
True
>>> False == (False or True)
False
>>> (False == False) or True
True
```

> Python的运算符顺序与数学中的运算顺序是一致的：括号优先，然后是幂运算，然后是乘/除法运算，然后是加/减法运算。

下面的表按照优先级从高到低展示了Python中所有的运算符。

|运算符|描述|
|:--|:--|
| `**` |指数 (最高优先级)|
| `~` `+` `-` |按位翻转, 一元加号和减号 (最后两个的方法名为 +@ 和 -@)|
| `*` `/` `%` `//` |乘，除，取模和取整除|
| `+` `-`|加法减法|
|`>>` `<<`|右移，左移运算符|
|`&`|位 'AND'|
|`^` &#124;| 位运算符|
|`<=` `<` `>` `>=`|比较运算符|
|`<>` `==` `!=`|等于运算符|
| `=` `%=` `/=` `//=` `-=` `+=` `*=` `**=`	|赋值运算符|
|`is` `is not`|身份运算符|
|`in` `not in`|成员运算符|
|`not` `or` `and`|逻辑运算符|

## while循环

一个**if**语句在条件为**True**时会执行一次，如果为**False**则不执行。**while**语句会在条件满足时执行多次。只要条件为**True**，while内部的语句就会重复的执行下去。一旦条件为**False**，就会跳出循环，继续执行后面的代码。

下面是一个**while**循环，包含一个从1到5自增长用来计数的变量，直到i的值达到了循环的终点就会跳出循环。

```python
i = 1
while i <=5:
   print(i)
   i = i + 1

print("Finished!")
```

执行结果：

```python
>>>
1
2
3
4
5
Finished!
>>>
```

> **while**内部循环执行的的代码被称为**迭代(iteration)**。

### 无限循环

**无限循环**是一种特殊的while循环；它永远都不会停止运行。它的判断条件一直是**True**	。

下面是一个无限循环的例子：

```python
while 1==1:
  print("In the loop") 
```

这个程序会无限的打印`In the loop`。

> 你可以通过按下快捷键**Ctrl-C**来终止程序的执行，从而中断这个无限循环。

### break

通过使用**break**语句，可以提前跳出**while**循环的执行。**break**可以立刻中断**while**内部的循环。

```python
i = 0
while 1==1:
  print(i)
  i = i + 1
  if i >= 5:
    print("Breaking")
    break

print("Finished")
```

执行结果：

```python
>>>
0
1
2
3
4
Breaking
Finished
>>>
```

> 在循环外使用**break**会产生错误。

### continue

另一个可以用在循环内的语句是**continue**。

不同于**break**，**continue**会跳回到循环的顶部而不是停止循环的执行。

```python
i = 0
while True:
   i = i +1
   if i == 2:
      print("Skipping 2")
      continue
   if i == 5:
      print("Breaking")
      break
   print(i)

print("Finished")
```

执行结果：

```python
>>>
1
Skipping 2
3
4
Breaking
Finished
>>>
```

**continue**语句会停止当前这一次的迭代项，并且继续执行下一次项。

> 在循环外使用**continue**会产生错误。

## 列表(List)

**列表(List)**是另一个Python对象，它可以用于存储条目的索引列表。
列表可以通过一个**方括号(`[]`)**来构建，内部的条目用**逗号(`,`)**隔开。

可以通过使用其在方括号中的索引来访问列表中的特定项。

例如：

```python
words = ["Hello", "world", "!"]
print(words[0])
print(words[1])
print(words[2])
```

执行结果：

```
>>>python
Hello
world
!
>>>
```

> 第一个列表项的索引值是**0**，而不是**1**。

一个空的列表可以通过一对方括号来实现：

```python
empty_list = []
print(empty_list)
```

执行结果：

```python
>>>
[]
>>>
```

> 大多数情况下，列表的最后一个条目后是不用添加逗号的。但是如果你添加了逗号，那就表明在最后占了一个位置，在某些情况下这种做法也是被推荐的。

列表通常包含着同一种类型的数据，但你也完全可以添加不同类型的数据到列表中。列表本身也可以嵌套在其他列表中。

```python
number = 3
things = ["string", 0, [1, 2, number], 4.56]
print(things[1])
print(things[2])
print(things[2][2])
```

执行结果：

```python
>>>
0
[1, 2, 3]
3
>>>
```

> 列表嵌套列表是通常用于表示二维网格，Python缺少像其他语言中的**多维数组**的结构，可以通过嵌套列表来实现。

索引超出列表长度的边界会导致**IndexError**。

有些类型，如**字符串(strings)**，可以像**列表(List)**一样被索引。索引字符串的行为就好像你在索引一个包含了每个字符的列表所组成的字符串。

对于其他类型，例如**整数(integers)**，它们是不可以被索引的，如果对它使用索引会导致TypeError。

```python
str = "Hello world!"
print(str[6])
```

执行结果：

```python
>>>
w
>>>
```

## 列表操作(List Operations)

列表中的某个位置的条目可以重新赋值。

```python
nums = [7, 7, 7, 7, 7]
nums[2] = 5
print(nums)
```

执行结果：

```python
>>>
[7, 7, 5, 7, 7]
>>>
```

列表可以像字符串一样执行**加法**和**乘法**操作：

```python
nums = [1, 2, 3]
print(nums + [4, 5, 6])
print(nums * 3)
```

执行结果：

```python
>>>
[1, 2, 3, 4, 5, 6]
[1, 2, 3, 1, 2, 3, 1, 2, 3]
>>>
```

> 列表和字符串在很多情况下是相似的，字符串可以想象成是字符组成的不可变列表。

---

可以通过**in**操作符来检测某个条目是否存在于某个列表中。如果当前条目在列表中存在一份或多份时，返回**True**，否则返回**False**。

```python
words = ["spam", "egg", "spam", "sausage"]
print("spam" in words)
print("egg" in words)
print("tomato" in words)
```

执行结果：

```python
>>>
True
True
False
>>>
```

> **in**操作符也可以用来判断某个字符串是否是另一个字符串的子字符串。

---

可以通过**not**操作符来判断某个条目是否不在列表中：

```python
nums = [1, 2, 3]
print(not 4 in nums)
print(4 not in nums)
print(not 3 in nums)
print(3 not in nums)
```

执行结果：

```python
>>>
True
True
False
False
>>>
```
---

## 列表方法(List Functions)

另外一种改变列表的方式是通过**append**函数。这个函数会添加一个元素到列表的末尾。

```python
nums = [1, 2, 3]
nums.append(4)
print(nums)
```

执行结果：

```python
>>>
[1, 2, 3, 4]
>>>
```

---

可以通过**len**函数来获取列表中元素的个数。

```python
nums = [1, 3, 5, 2, 4]
print(len(nums))
```

执行结果：

```python
>>>
5
>>>
```

---

**insert**方法和**append**类似，不同的是，它可以在列表的任何位置插入指定的元素，而不是仅仅跟在列表最后。

```python
words = ["Python", "fun"]
index = 1
words.insert(index, "is")
print(words)
```

执行结果：

```python
>>>
['Python', 'is', 'fun']
>>>
```

---

**index**方法会找到元素第一次出现的所在位置索引值，如果元素不在列表中，会抛出一个ValueError。

```python
letters = ['p', 'q', 'r', 's', 'p', 'u']
print(letters.index('r'))
print(letters.index('p'))
print(letters.index('z'))
```

执行结果：

```python
>>>
2
0
ValueError: 'z' is not in list
>>> 
```

> 下面有几个列表的更有用的方法：
> 
> - **max**(list): 返回列表中最大的元素。
> - **min**(list): 返回列表中最小的元素。
> - list.**count**(obj): 返回列表中指定元素的出现次数。
> - list.**remove**(obj): 移除列表中的一个指定元素。
> - list.**reverse**(): 将列表中的元素顺序翻转。

--- 

## Range

**range**方法会创建一个包含数字序列的列表。下面的代码会生成一组0到10之间(包含0不包含10)的整数的序列：

```python
numbers = list(range(10))
print(numbers)
```

执行结果：

```python
>>>
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>>
```

---

如果**range**方法传入一个参数，那么它将产生一组包含0到那个参数的数字序列。

如果包含两个参数，那么它将产生第一个参数到第二个参数之间的数字序列。

例如：

```python
numbers = list(range(3, 8))
print(numbers)

print(range(20) == range(0, 20))
```

执行结果：

```python
>>>
[3, 4, 5, 6, 7]

True
>>>
```

---

**range**可以拥有第三个用来决定步数的参数。这个参数必须是一个**整形(integer)**。

```python
numbers = list(range(5, 20, 2))
print(numbers)
```

执行结果：

```python
>>>
[5, 7, 9, 11, 13, 15, 17, 19]
>>>
```

---

## for循环

### 循环

有时，你需要对列表中的元素依次执行运算，这种操作叫做**迭代(iteration)**。迭代可以通过**while**循环以及一个条件变量来实现。

例如：

```python
words = ["hello", "world", "spam", "eggs"]
counter = 0
max_index = len(words) - 1

while counter <= max_index:
   word = words[counter]
   print(word + "!")
   counter = counter + 1 
```

执行结果：

```python
>>>
hello!
world!
spam!
eggs!
>>>
```

---

### for循环

使用**while**来对列表进行迭代操作需要写的代码有点多，所以Python提供了一种**for**循环的方式来更简便的实现这一操作。

同样的代码用**for**循环来实现：

```python
words = ["hello", "world", "spam", "eggs"]
for word in words:
  print(word + "!")
```

执行结果：

```python
>>>
hello!
world!
spam!
eggs!
>>>
```

> Python中的**for**循环，就像其他语言中的**foreach**循环一样。

---

**for**循环通常用于重复某一代码执行一定次数。这种操作可以通过与**range**对象结合使用来完成：

```python
for i in range(5):
  print("hello!")
```

执行结果：

```python
>>>
hello!
hello!
hello!
hello!
hello!
>>>
```

---

## 一个简单的计算器

### 创建一个计算器

这节课介绍一个Python程序的示例：一个简单的计算器。

第一部分是整体菜单部分。它将持续接受用户的输入，直到用户输入了“quit”为止，因此用到了一个**while**循环。

```python
while True:
   print("Options:")
   print("Enter 'add' to add two numbers")
   print("Enter 'subtract' to subtract two numbers")
   print("Enter 'multiply' to multiply two numbers")
   print("Enter 'divide' to divide two numbers")
   print("Enter 'quit' to end the program")
   user_input = input(": ")

   if user_input == "quit":
      break
   elif user_input == "add":
      ...
   elif user_input == "subtract":
      ...
   elif user_input == "multiply":
      ...
   elif user_input == "divide":
      ...
   else:
      print("Unknown input")
```

> 这部分代码是程序的开始部分。它将接受用户的输入，并且比较**if/elif**的条件语句。
> 
> 当用户输入了“quit”时，**break**语句用来终止while循环的执行。

---

接下来一部分的代码是获取用户输入的数字。下面的代码展示了加法部分的逻辑。其他操作的代码也与此类似。

```python
elif user_input == "add":
  num1 = float(input("Enter a number: "))
  num2 = float(input("Enter another number: "))
```

现在，当用户输入了“add”时，程序提示输入两个数字，并且存储到相应的变量中。

> 当提示输入数字时，如果用户输入了一个非数字的字符，那么会产生一个崩溃。我们会在后面的章节来修复这个类似的问题。

---

最后一部分是处理用户输入，并且展示结果。

加法部分的逻辑如下：

```python
elif user_input == "add":
  num1 = float(input("Enter a number: "))
  num2 = float(input("Enter another number: "))
  result = str(num1 + num2)
  print("The answer is " + result)
```

我们现在有了一个可以正常工作程序：提示用户输入，然后计算和打印输入的和。

> 其他分支(减法，乘法，除法)的代码结构类似，可以自行补全。
> 
> 输出行可以放在**if**语句之外，以省略重复的代码。
