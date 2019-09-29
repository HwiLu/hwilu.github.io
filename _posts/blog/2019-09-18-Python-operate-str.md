---
layout: post
title: Python 操作字符串总结
categories: [Python]
description: Python基础
keywords: Python3
---

Python如何操作字符串总结，Python字符串处理，有本文就够了 :smile: 。

- 截取字符串
```
str[0:3] #截取第一位到第三位的字符
str[:] #截取字符串的全部字符
str[6:] #截取第七个字符到结尾
str[:-3] #截取从头开始到倒数第三个字符之前
str[2] #截取第三个字符
str[-1] #截取倒数第一个字符
str[::-1] #创造一个与原字符串顺序相反的字符串
str[-3:-1] #截取倒数第三位与倒数第一位之前的字符
str[-3:] #截取倒数第三位到结尾</pre>
```
```
>>> str='1234567890'
>>> str[::-1]
'0987654321'
>>> str[-1]
'0'
>>> str[-3:]
'890'
>>> str[6:]
'7890'
>>>
```
- 查找
```
>>> a = 'test'
>>> a.find('s')
2
>>> 
```
- join
str.join()方法用于将序列中的元素以指定的字符连接生成一个新的字符串
```
>>> test = ['a','b','c','d']
>>> out = '+'.join(test)
>>> out
'a+b+c+d' 
>>>str = '-'
>>>seq = ("a", "b", "c"); # 字符串序列
>>>str.join(seq)  
'a-b-c'
```
- replace
```
>>> a  = 'hello world'
>>> b = a.replace('l','t')
>>> b
'hetto wortd'
>>> str = hello new new
>>>str.replace('n','N',1)
'hello New new'
```
- 字符串重复
```
# str * n, n * str
# n 为一个 int 数字
str = "hi"
print str*2   # hihi
print 2*str   # hihi
```
- 输出格式对齐
```
>>>  str.center(20)         #生成20个字符长度，str排中间
>>> str.ljust(20)             #生成20个字符长度，str左对齐
>>>  str.rjust(20)            #生成20个字符长度，str右对齐
```
- 检测字符串组成
```
# 检测数字
str.isdigit()    # 检测字符串是否只由数字组成
str.isnumeric()  # 检测字符串是否只由数字组成,这种方法是只针对unicode对象
str.isdecimal()  # 检查字符串是否只包含十进制字符。这种方法只存在于unicode对象
# 检测字母
str.isalpha()   # 检测字符串是否只由字母组成
# 检测字母和数字
str.isalnum()   # 检测字符串是否由字母和数字组成
# 检测其他
str.isspace()   # 检测字符串是否只由空格组成
str.islower()   # 检测字符串是否由小写字母组成
str.isupper()   # 检测字符串中所有的字母是否都为大写
str.istitle()   # 检测字符串中所有的单词拼写首字母是否为大写，且其他字母为小写
```

判断字符串开头结尾
```python
>>> str='hello world you'
>>> str.startswith('hello')
True

>>> str.endswith('you')        　　#判读字符串以'you'结尾
True

```



- 处理字符串
```
str.capitalize()   # 将字符串的第一个字母变成大写,其他字母变小写
str.lower()        # 转换字符串中所有大写字符为小写
str.upper()        # 将字符串中的小写字母转为大写字母
str.swapcase()     # 对字符串的大小写字母进行转换
max(str)    # 返回字符串 str 中最大的字母
min(str)    # 返回字符串 str 中最小的字母
len(str)    # 返回字符串的长度
str(arg) # 将 arg 转换为 string
---------------------------------
>>> b='1212344444439'
>>> max(b)
'9'
>>> a =1
>>> a + 1
2
>>> str(a)
'1'
```
- 字符串去燥
```
# 去除字符串中相同的字符
s = '\tabc\t123\tisk'
print(s.replace('\t', ''))
```
```
import re
# 去除\r\n\t字符
s = '\r\nabc\t123\nxyz'
print(re.sub('[\r\n\t]', '', s))
```

** refer:[简书blog](https://www.jianshu.com/p/b758332c44bb) **
