#python循环迭代

##1. 循环计数器: while和range

遍历字符串X：用for

	X='spam'
	for i in X:
		print i,

遍历字符串X：用while

	X='spam'
	i=0
	while i<len(X):
		print X[i],
		i += 1

两者结果一致。

**for循环内定迭代:**

* 内置range函数生成一系列连续增加的整数，可作为for的索引
* 内置zip函数返回并行元素的元组列表，可用于for内遍历数个序列

**for循环一般比wihle计数器循环快**

当然for也可以用计数器：

	>>> X='spam'
	>>> for i in range(len(X)):
	...     print X[i],
	... 
	s p a m

还可以更复杂，如想跳过序列中某些元素(分片）：

	>>> x='liaoheng'
	>>> for i in range(0, len(x), 2):
	...     print x[i],
	... 
	l a h n

用步进值为2来分片也可以：

	>>> x='liaoheng'
	>>> for i in x[::2]:
	...     print i,
	... 
	l a h n

**问题1：** 如果想创建一个序列中所有元素加1，如何操作  
一般会想到如下操作:

	>>> a=[1,2,3,4,5]
	>>> for i in a:
	...     i += 1
	... 
	>>> i
	6
	>>> print a[:]
	[1, 2, 3, 4, 5]

这结果明显不对，序列a的元素没有改变。这只是把x对应不同的对象，1，2，3, 4, 5，并没有修改序列中的元素

	>>> for i in range(len(a)):
	...     a[i] += 1
	... 
	>>> print a
	[2, 3, 4, 5, 6]

如果用while写：

	>>> a=[1,2,3,4,5]
	>>> i=0
	>>> while i < len(a):
	...     a[i] += 1
	...     i += 1
	... 
	>>> 
	>>> 
	>>> print a
	[2, 3, 4, 5, 6]

##2.并行遍历： zip和map

zip以一个或多个序列为参数，然后返回元组的列表，将这些序列的并排元素配成对。

	>>> a=[1,2,3]
	>>> b=[4,5,6]
	>>> zip(a,b)
	[(1, 4), (2, 5), (3, 6)]
	>>> type(zip(a,b))
	<type 'list'>
	>>> for (x,y) in zip(a,b):
	...     print (x, y, '----', x+y)
	... 
	(1, 4, '----', 5)
	(2, 5, '----', 7)
	(3, 6, '----', 9)

三序列用zip：

	>>> T1,T2,T3=(1,2,3),(4,5,6),(7,8,9)
	>>> T3
	(7, 8, 9)
	>>> list(zip(T1,T2,T3))
	[(1, 4, 7), (2, 5, 8), (3, 6, 9)]

**当参数的长度不同时，zip会以最短序列的长度为准来截断所得到的元组**

	>>> s1='abc'
	>>> s2='123456'
	>>> zip(s1,s2)
	[('a', '1'), ('b', '2'), ('c', '3')]

比起zip，map在面对参数长度不同时，则可以为较短的序列用None补齐：

	>>> map(None,s1,s2)
	[('a', '1'), ('b', '2'), ('c', '3'), (None, '4'), (None, '5'), (None, '6')]



使用zip构造字典

	>>> a={}
	>>> for (x,y) in zip(keys,vals):
	...     a[x]=y
	... 
	>>> a
	{'heng': 3, 'liao': 1}

不用for

	>>> d3=dict(zip(keys,vals))
	>>> d3
	{'heng': 3, 'liao': 1}

###产生偏移和元素：enumerate

	>>> s='spam'
	>>> offset=0
	>>> for i in s:
	...     print(i,'appears at offset',offset)
	...     offset += 1
	... 
	('s', 'appears at offset', 0)
	('p', 'appears at offset', 1)
	('a', 'appears at offset', 2)
	('m', 'appears at offset', 3)

偏移量有个内置函数可以用

	>>> s='spam'
	>>> for (offset,item) in enumerate(s):
	...     print(item,'appers as offset',offset)
	... 
	('s', 'appers as offset', 0)
	('p', 'appers as offset', 1)
	('a', 'appers as offset', 2)
	('m', 'appers as offset', 3)




##文件迭代器

	>>> f=open('install.log')
	>>> f.readline()
	'Installing libgcc-4.4.7-4.el6.x86_64\n'
	>>> f.readline()
	'warning: libgcc-4.4.7-4.el6.x86_64: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY\n'
	>>> f.readline()
	'Installing setup-2.8.14-20.el6_4.1.noarch\n'
	>>> f.readline()
	'Installing filesystem-2.4.30-3.el6.x86_64\n'
	>>> f.readline()
	'Installing basesystem-10.0-4.el6.noarch\n'

文件对象也有个方法 __next__,差不多有相同的效果：每次调用时，就会返回文件中的下一行。  
唯一区别，到达文件末尾时，__next__会引发内置的StopIteration异常，而不是返回空字符串
>python2.7 next()

	>>> for line in open('install.log'):
	...     print line.upper(),
	... 
	INSTALLING LIBGCC-4.4.7-4.EL6.X86_64
	WARNING: LIBGCC-4.4.7-4.EL6.X86_64: HEADER V3 RSA/SHA1 SIGNATURE, KEY ID C105B9DE: NOKEY
	INSTALLING SETUP-2.8.14-20.EL6_4.1.NOARCH
	INSTALLING FILESYSTEM-2.4.30-3.EL6.X86_64
	INSTALLING BASESYSTEM-10.0-4.EL6.NOARCH
	INSTALLING NCURSES-BASE-5.7-3.20090208.EL6.X86_64
	INSTALLING TZDATA-2013G-1.EL6.NOARCH
	INSTALLING GLIBC-COMMON-2.12-1.132.EL6.X86_64
	INSTALLING NSS-SOFTOKN-FREEBL-3.14.3-9.EL6.X86_64


	>>> for line in open('install.log').readlines():
	...     print line.upper(),
	... 
	INSTALLING LIBGCC-4.4.7-4.EL6.X86_64
	WARNING: LIBGCC-4.4.7-4.EL6.X86_64: HEADER V3 RSA/SHA1 SIGNATURE, KEY ID C105B9DE: NOKEY
	INSTALLING SETUP-2.8.14-20.EL6_4.1.NOARCH
	INSTALLING FILESYSTEM-2.4.30-3.EL6.X86_64
	INSTALLING BASESYSTEM-10.0-4.EL6.NOARCH
	INSTALLING NCURSES-BASE-5.7-3.20090208.EL6.X86_64
	INSTALLING TZDATA-2013G-1.EL6.NOARCH
	INSTALLING GLIBC-COMMON-2.12-1.132.EL6.X86_64
	INSTALLING NSS-SOFTOKN-FREEBL-3.14.3-9.EL6.X86_64

用while写：

	>>> f=open('install.log')
	>>> while True:
	...     line=f.readline()
	...     if not line: break
	...     print line.upper(),
	... 
	INSTALLING LIBGCC-4.4.7-4.EL6.X86_64
	WARNING: LIBGCC-4.4.7-4.EL6.X86_64: HEADER V3 RSA/SHA1 SIGNATURE, KEY ID C105B9DE: NOKEY
	INSTALLING SETUP-2.8.14-20.EL6_4.1.NOARCH
	INSTALLING FILESYSTEM-2.4.30-3.EL6.X86_64
	INSTALLING BASESYSTEM-10.0-4.EL6.NOARCH
	INSTALLING NCURSES-BASE-5.7-3.20090208.EL6.X86_64
	INSTALLING TZDATA-2013G-1.EL6.NOARCH
	INSTALLING GLIBC-COMMON-2.12-1.132.EL6.X86_64
	INSTALLING NSS-SOFTOKN-FREEBL-3.14.3-9.EL6.X86_64

用next()

	>>> f=open('install.log')
	>>> next(f)
	'Installing libgcc-4.4.7-4.el6.x86_64\n'
	>>> next(f)
	'warning: libgcc-4.4.7-4.el6.x86_64: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY\n'
	>>> next(f)
	'Installing setup-2.8.14-20.el6_4.1.noarch\n'
	>>> next(f)
	'Installing filesystem-2.4.30-3.el6.x86_64\n'
	>>> next(f)
	'Installing basesystem-10.0-4.el6.noarch\n'
	>>> next(f)
	'Installing ncurses-base-5.7-3.20090208.el6.x86_64\n'
	>>> next(f)
	'Installing tzdata-2013g-1.el6.noarch\n'
	>>> next(f)
	'Installing glibc-common-2.12-1.132.el6.x86_64\n'
	>>> next(f)
	'Installing nss-softokn-freebl-3.14.3-9.el6.x86_64\n'
	>>> next(f)
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	StopIteration
	>>> next(f)
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>

##其他内置类型迭代器

字典，用keys()函数遍历字典索引：

	>>> a={'a':1,'b':2,'c':3,'d':4}
	>>> a.keys()
	['a', 'c', 'b', 'd']
	>>> for key in a.keys():
	...     print a[key],
	... 
	1 3 2 4

python中有个iter函数可以把对象变成可迭代对象

	>>> I=iter(a)
	>>> type(I)
	<type 'dictionary-keyiterator'>
	>>> type(a)
	<type 'dict'>
	>>> next(I)
	'a'
	>>> next(I)
	'c'
	>>> next(I)
	'b'
	>>> next(I)
	'd'
	>>> next(I)
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	StopIteration

##列表解析
>不同于for,列表解析是生成一个新的列表，再重新赋值给变量：

	>>> L=[1,2,3,4,5]
	>>> for i in range(len(L)):
	...     L[i] += 10
	... 
	>>> L
	[11, 12, 13, 14, 15]


	>>> L=[1,2,3,4,5]
	>>> L=[x+10 for x in L]
	>>> L
	[11, 12, 13, 14, 15]

###在文件上用列表解析

	>>> f=open('install.log')
	>>> lines=f.readlines()
	>>> lines
	['Installing libgcc-4.4.7-4.el6.x86_64\n', 'warning: libgcc-4.4.7-4.el6.x86_64: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY\n', 'Installing setup-2.8.14-20.el6_4.1.noarch\n', 'Installing filesystem-2.4.30-3.el6.x86_64\n', 'Installing basesystem-10.0-4.el6.noarch\n', 'Installing ncurses-base-5.7-3.20090208.el6.x86_64\n', 'Installing tzdata-2013g-1.el6.noarch\n', 'Installing glibc-common-2.12-1.132.el6.x86_64\n', 'Installing nss-softokn-freebl-3.14.3-9.el6.x86_64\n']

readlines()一次把文件所有行读入，并以列表返回

如果想要去掉结尾的\n 换行符，可以用[:-1]分片，或者用rstrip()

	>>> lines = [ line[:-1] for line in lines]
	>>> lines
	['Installing libgcc-4.4.7-4.el6.x86_64', 'warning: libgcc-4.4.7-4.el6.x86_64: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY', 'Installing setup-2.8.14-20.el6_4.1.noarch', 'Installing filesystem-2.4.30-3.el6.x86_64', 'Installing basesystem-10.0-4.el6.noarch', 'Installing ncurses-base-5.7-3.20090208.el6.x86_64', 'Installing tzdata-2013g-1.el6.noarch', 'Installing glibc-common-2.12-1.132.el6.x86_64', 'Installing nss-softokn-freebl-3.14.3-9.el6.x86_64']
	>>> lines = [ line.rstrip() for line in lines]
	>>> lines
	['Installing libgcc-4.4.7-4.el6.x86_64', 'warning: libgcc-4.4.7-4.el6.x86_64: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY', 'Installing setup-2.8.14-20.el6_4.1.noarch', 'Installing filesystem-2.4.30-3.el6.x86_64', 'Installing basesystem-10.0-4.el6.noarch', 'Installing ncurses-base-5.7-3.20090208.el6.x86_64', 'Installing tzdata-2013g-1.el6.noarch', 'Installing glibc-common-2.12-1.132.el6.x86_64', 'Installing nss-softokn-freebl-3.14.3-9.el6.x86_64']


###列表解析扩展语法

读取Install.log文件，把开头为I的行，去掉\n，生成列表：

	>>> lines=[line.rstrip() for line in open('install.log') if line[0] == 'I']
	>>> lines
	['Installing libgcc-4.4.7-4.el6.x86_64', 'Installing setup-2.8.14-20.el6_4.1.noarch', 'Installing filesystem-2.4.30-3.el6.x86_64', 'Installing basesystem-10.0-4.el6.noarch', 'Installing ncurses-base-5.7-3.20090208.el6.x86_64', 'Installing tzdata-2013g-1.el6.noarch', 'Installing glibc-common-2.12-1.132.el6.x86_64', 'Installing nss-softokn-freebl-3.14.3-9.el6.x86_64']

分解步骤：

	>>> res=[]
	>>> for line in open('install.log'):
	...     if line[0] == 'I':
	...             res.append(line.rstrip())
	... 
	>>> res
	['Installing libgcc-4.4.7-4.el6.x86_64', 'Installing setup-2.8.14-20.el6_4.1.noarch', 'Installing filesystem-2.4.30-3.el6.x86_64', 'Installing basesystem-10.0-4.el6.noarch', 'Installing ncurses-base-5.7-3.20090208.el6.x86_64', 'Installing tzdata-2013g-1.el6.noarch', 'Installing glibc-common-2.12-1.132.el6.x86_64', 'Installing nss-softokn-freebl-3.14.3-9.el6.x86_64']


另一个例子：

	>>> [x+y for x in 'abc' for y in 'lmn']
	['al', 'am', 'an', 'bl', 'bm', 'bn', 'cl', 'cm', 'cn']

分解步骤类似：

	>>> a='abc'
	>>> b='lmn'
	>>> c=[]
	>>> for x in a:
	...     for y in b:
	...             c.append(x+y)
	... 
	>>> 
	>>> c
	['al', 'am', 'an', 'bl', 'bm', 'bn', 'cl', 'cm', 'cn']