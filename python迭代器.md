#python 迭代器和列表解析

###序列迭代器
列表 :  
`>>>for x in [1, 2, 3, 4, 5]: print  x ** 2,` #python的print语句会默认在末尾加上一个\n  
元组 ：  
`>>>for x in (1, 2, 3, 4, 5): print  x ** 2,`  
字符串 ：  
`>>>for x in 'spam': print  x *2,` 

###文件迭代器

用readline()：

    >>> f=open('adder.py')
    >>> f.readline()
    'import sys\n'
    >>> f.readline()
    'sum = 0\n'
    >>> f.readline()
    'while True:\n'
    >>> f.readline()
    '\ttry:\n'
    >>> f.readline()
    '\t\tline=raw_input()\n'
    >>> f.readline()
    '\texcept EOFError:\n'
    >>> f.readline()
    '\t\tbreak\n'
    >>> f.readline()

>文件有个迭代方法*\_\_next\_\_*,每次调用都会返回文件的下一行。到达文件末尾时，\_\_next\_\_会引发内置的StopIteration异常，而不是返回空字符串   
>python 3 : *\_\_next\_\_* , python2 : *next(x)*或者X.next()

所有这类对象的迭代工具原理：内部工作调用\_\_next\_\_ ,并捕捉StopIteration异常来确定何时离开。

    >>> import os
    >>> for line in open('adder.py'):
    ... print line.upper(),
    ... 
    IMPORT SYS
    SUM = 0
    WHILE TRUE:
    	TRY:
    		LINE=RAW_INPUT()
    	EXCEPT EOFERROR:
    		BREAK
    	ELSE:
    		SUM += INT(LINE)
    PRINT SUM


还可以用file.readlines（）迭代列表，又或者用while迭代  

	f = open('adder.py')
	while True:
		line = f.realine()
		if not line:
			break
		print line.upper(),

>while的速度没有for快，迭代器在python中以c语言的速度运行的，而while循环是通过python虚拟机运行python字节码的

###其他内置类型迭代器

遍历字典的方法（获取其键的列表）：  

    >>> f=open('adder.py')
    KeyboardInterrupt
    >>> D={'a':1,'b':2,'c':3}
    >>> for key in D.keys():
    ... 	print key, D[key]
    ... 
    a 1
    c 3
    b 2

新版本的python2可以直接这么用：

    >>> for key in D:
    ···		 print key, D[key]
    ... 
    a 1
    c 3
    b 2

os.popen也支持迭代协议：

    >>> p = os.popen('ls -l')
    >>> p.next()
    'total 20\n'
    >>> p.next()
    '-rw-r--r-- 1 root root 120 Oct 12 00:23 adder.py\n'
    >>> p.next()
    '-rw-r--r-- 1 root root  12 Oct 12 00:23 add.txt\n'
    >>> p.next()
    '-rw-r--r-- 1 root root 111 Oct 12 01:05 liao.py\n'
    >>> p.next()
    '-rw-r--r-- 1 root root  85 Oct 12 00:28 sort.py\n'


    >>> a=range(5)
    >>> for i in a:
    ... 	print i
    ... 
    0
    1
    2
    3
    4
    >>> I = iter(a) #将一个对象变成迭代器
   	>>> next(I)
    0
    >>> next(I)
    1
    >>> next(I)
    2
    >>> next(I)
    3
    >>> next(I)
    4
    >>> next(I)

再来看看一个偏移量方法的工作原理：  

    >>> E=enumerate('spam') #enumerate偏移量的方法，生成一个生成式对象
    >>> E
    <enumerate object at 0x7f8aa3696a50>
    >>> I=iter(E) 
    >>> next(E)
    (0, 's')
    >>> next(E)
    (1, 'p')
    >>> next(E)
    (2, 'a')
    >>> next(E)
    (3, 'm')
    >>> next(E)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    StopIteration
    >>> list(enumerate('spam')) #可迭代的生成器对象可以用list方法转变为列表
    [(0, 's'), (1, 'p'), (2, 'a'), (3, 'm')]

