#python stream

###打印流重定向：  


    >>>print 'hello world'
    hello world 
    >>>'hello world'
    'hello world'
    
在交互模式下，表达式'hellow world'直接回显到屏幕上,原因是因为*sys.stdout* ：  

    >>>import sys
    >>>sys.stdout.write('hello world\n')
    hello world

调用了sys.stdout的write方法（当python启动链接输出流的文件对象时，这个就预先设置好了）  

###重定向输出流 ：

    python x, y  
等价于

    import sys
    sys.stdout.write(str(x) + ' ' + str(Y) + '\n')

把sys.stdout重设成已打开的文件对象 ：

	import sys
	sys.stdout = open('log.txt', 'a')
	...
	print x, y #这里输出实际上写入到log.txt里
重设之后，程序中任何地方的print语句都会将文字写至文件log.txt的末尾，而不是原始输出流

###自动化流重定向：
先把原始输出流备份，然后把sys.stdout输出流连接到log.txt文件，打印spam和1 2 3到log.txt，再把原始输出流还原成默认，之后打印就显示在交互屏幕下  

	>>>import sys
	>>>temp = sys.stdout
	>>>sys.stdout = open('log.txt, 'a')
	>>>print 'spam'
	>>>print 1, 2, 3
	>>>sys.stdout.close()
	>>>sys.stdout = temp
	
	>>>print 'backup here'
	>>>print open('log.xtx'.read())
	spam
	1 2 3

###重定向输出流到文件或程序：

    "read numbers till eof and show squares"
    def interact():
    	print('Hello stream world') # print sends to sys.stdout
    	while True:
    		try:
    			reply = input('Enter a number>') 			# input reads sys.stdin
    		except EOFError:
    			break 					# raises an except on eof
    		else: 						# input given as a string
    			num = int(reply)
    			print("%d squared is %d" % (num, num ** 2))
    	print('Bye')
    if __name__ == '__main__':
    	interact() 						# when run, not imported


输出结果：

    C:\...\PP4E\System\Streams> python teststreams.py
    Hello stream world
    Enter a number>12
    12 squared is 144
    Enter a number>10
    10 squared is 100
    Enter a number>^Z
    Bye

从文件中导入输入流：

    C:\...\PP4E\System\Streams> type input.txt   # type 类似linux的cat命令
    8
    6
    C:\...\PP4E\System\Streams> python teststreams.py < input.txt
    Hello stream world
	Enter a number>8 squared is 64
	Enter a number>6 squared is 36
	Enter a number>Bye


    C:\...\PP4E\System\Streams> python teststreams.py < input.txt > output.txt
    C:\...\PP4E\System\Streams> type output.txt
    Hello stream world
    Enter a number>8 squared is 64
    Enter a number>6 squared is 36
    Enter a number>Bye


###管道连接程序：
python的管道用法和shell差不多：

    C:\...\PP4E\System\Streams> python teststreams.py < input.txt | more
    Hello stream world
    Enter a number>8 squared is 64
    Enter a number>6 squared is 36
    Enter a number>Bye

再看个例子：  

    C:\...\PP4E\System\Streams> type writer.py
    print("Help! Help! I'm being repressed!")
    print(42)
    
	C:\...\PP4E\System\Streams> type reader.py
    print('Got this: "%s"' % input())
    import sys
    data = sys.stdin.readline()[:-1]
    print('The meaning of life is', data, int(data) * 2)
    
    C:\...\PP4E\System\Streams> python writer.py
    Help! Help! I'm being repressed!
    42
    C:\...\PP4E\System\Streams> python writer.py | python reader.py
    Got this: "Help! Help! I'm being repressed!"
    The meaning of life is 42 84
>这个是python3的例子，python3默认输入流inpu()会自动转变为字符串格式
,python2需要用raw_input()

python2 的reader.py要这么写：

    print 'Got this: %s' % raw_input()
    import sys
    data = sys.stdin.readline()[:-1]
    print 'The meaning of life is', data, int(data) * 2
	
	[root@node1 ~]# python writer.py | python reader.py 
	Got this: Help! Help! I'm being repressed!
	The meaning of life is 42 84

这里把write.py执行第一个print语句，生成第一条输出流，立马作为reader.py的输入，被print 'Got this...'输出到屏幕上。  
当writer.py再执行第二句 又重复以上过程。

*vim sort.py*

    import sys
    lines=sys.stdin.readlines()
    lines.sort()
    for line in lines:
            print line,

*vim sort.txt*

    1123
    2423
    1121
    1923
    2444

*vim adder.py*

    import sys
    sum = 0
    while True:
            try:
                    line=raw_input()
            except EOFError:
                    break
            else:
                    sum += int(line)
    print sum

*vim add.txt*

    123
    231
    222

excute:

    [root@node1 liaoheng]# python sort.py < sort.txt| python adder.py 
    9034
	
    
再看个例子：

    import sys
    sum = 0
    while True:
        line = sys.stdin.readline()
        if not line:
            break
        sum += int(line)

    print sum

	[root@node1 liaoheng]# cat <<EOF | python liao.py 
	123
	214
	556
	129
	EOF
	1022

