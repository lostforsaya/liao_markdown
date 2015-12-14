###python sys 

导入sys模块  
`import sys`

模块搜索路径

    >>> sys.path
    ['', '/usr/lib64/python26.zip', '/usr/lib64/python2.6', '/usr/lib64/python2.6/plat-linux2', '/usr/lib64/python2.6/lib-tk', '/usr/lib64/python2.6/lib-old', '/usr/lib64/python2.6/lib-dynload', '/usr/lib64/python2.6/site-packages', '/usr/lib/python2.6/site-packages', '/usr/lib/python2.6/site-packages/setuptools-0.6c11-py2.6.egg-info']


sys搜索路径可以用append，extend，insert，remove del这些参数去修改

    >>> sys.path.append(r'/root')
    >>> sys.path
    ['', '/usr/lib64/python26.zip', '/usr/lib64/python2.6', '/usr/lib64/python2.6/plat-linux2', '/usr/lib64/python2.6/lib-tk', '/usr/lib64/python2.6/lib-old', '/usr/lib64/python2.6/lib-dynload', '/usr/lib64/python2.6/site-packages', '/usr/lib/python2.6/site-packages', '/usr/lib/python2.6/site-packages/setuptools-0.6c11-py2.6.egg-info', '/root']

查看已导入的模块：

    >>> sys.modules
    {'tokenize': <module 'tokenize' from '/usr/lib64/python2.6/tokenize.pyc'>, 'copy_reg': <module 'copy_reg' from '/usr/lib64/python2.6/copy_reg.pyc'>, 'sre_compile': <module 'sre_compile' from '/usr/lib64/python2.6/sre_compile.pyc'>, '_collections': <module '_collections' from '/usr/lib64/python2.6/lib-dynload/_collectionsmodule.so'>, '_sre': <module '_sre' (built-in)>, 'encodings': <module 'encodings' from '/usr/lib64/python2.6/encodings/__init__.pyc'>, 'site': <modul


---
###python os

    os模块属性  
    linesep 文件中分隔行的字符串  
    sep 用来分隔文件路径名的字符串  
    pathsep 用于分隔文件路径的字符串  
    curdir 当前工作目录的字符串名称  
    pardir 当前目录的父目录的字符串名称  
    
    

**ex**：

    >>> os.sep
    '\\'
    >>> pathname = r'C:\PP4thEd\Examples\PP4E\PyDemos.pyw'
    >>> os.path.split(pathname) # split file from dir
    ('C:\\PP4thEd\\Examples\\PP4E', 'PyDemos.pyw')
    >>> pathname.split(os.sep) # split on every slash
    ['C:', 'PP4thEd', 'Examples', 'PP4E', 'PyDemos.pyw']
    >>> os.sep.join(pathname.split(os.sep))
    'C:\\PP4thEd\\Examples\\PP4E\\PyDemos.pyw'
    >>> os.path.join(*pathname.split(os.sep))
    'C:PP4thEd\\Examples\\PP4E\\PyDemos.pyw'
__os.path.join(*pathname.split(os.sep))__这个例子合并后发现第一个'C:' 与'PP4thEd'之间没用sep分隔符连起来

看这个linux下例子：

    >>> import os
    >>> pathname='/root/install.log'
    >>> pathname.split(os.sep)
    ['', 'root', 'install.log']
    >>> os.sep
    '/'
    >>> os.path.join(*pathname.split(os.sep))
    'root/install.log'


os文件操作：  
os.path.isdir 判断是否为目录  
os.path.isfile 判断是否为文件  
os.path.split 分隔文件和父目录，返回一个元组  
os.path.join 合并目录和文件，返回一个字符串序列  
os.path.dirname 取一个路径的最底层目录，返回字符串  
os.path.basename 取一个路径的最底层文件名，返回字符串  
os.path.splitext 把一个路径的文件名和扩展名分隔成一个元组返回

  
	os.path.abspath(path) #返回绝对路径
    os.path.basename(path) #返回文件名
    os.path.commonprefix(list) #返回list(多个路径)中，所有path共有的最长的路径。
    os.path.dirname(path) #返回文件路径
    os.path.exists(path)  #路径存在则返回True,路径损坏返回False
    os.path.lexists  #路径存在则返回True,路径损坏也返回True
    os.path.expanduser(path)  #把path中包含的"~"和"~user"转换成用户目录
    os.path.expandvars(path)  #根据环境变量的值替换path中包含的”$name”和”${name}”
    os.path.getatime(path)  #返回最后一次进入此path的时间。
    os.path.getmtime(path)  #返回在此path下最后一次修改的时间。
    os.path.getctime(path)  #返回path的大小
    os.path.getsize(path)  #返回文件大小，如果文件不存在就返回错误
    os.path.isabs(path)  #判断是否为绝对路径
    os.path.isfile(path)  #判断路径是否为文件
    os.path.isdir(path)  #判断路径是否为目录
    os.path.islink(path)  #判断路径是否为链接
    os.path.ismount(path)  #判断路径是否为挂载点（）
    os.path.join(path1[, path2[, ...]])  #把目录和文件名合成一个路径
    os.path.normcase(path)  #转换path的大小写和斜杠
    os.path.normpath(path)  #规范path字符串形式
    os.path.realpath(path)  #返回path的真实路径
    os.path.relpath(path[, start])  #从start开始计算相对路径
    os.path.samefile(path1, path2)  #判断目录或文件是否相同
    os.path.sameopenfile(fp1, fp2)  #判断fp1和fp2是否指向同一文件
    os.path.samestat(stat1, stat2)  #判断stat tuple stat1和stat2是否指向同一个文件
    os.path.split(path)  #把路径分割成dirname和basename，返回一个元组
    os.path.splitdrive(path)   #一般用在windows下，返回驱动器名和路径组成的元组
    os.path.splitext(path)  #分割路径，返回路径名和文件扩展名的元组
    os.path.splitunc(path)  #把路径分割为加载点与文件
    os.path.walk(path, visit, arg)  #遍历path，进入每个目录都调用visit函数，visit函数必须有
    3个参数(arg, dirname, names)，dirname表示当前目录的目录名，names代表当前目录下的所有
    文件名，args则为walk的第三个参数
    os.path.supports_unicode_filenames  #设置是否支持unicode路径名


**ex**：

    >>> os.path.isdir(r'C:\Users'), os.path.isfile(r'C:\Users')
    (False, False)
    >>> os.path.isfile(r'/root')
    False

    >>> os.path.split(r'/root/install.log')
    ('/root', 'install.log')
    >>> os.path.join(r'/liaoheng','output.txt')
    '/liaoheng/output.txt'
    >>> name='/home/liaoheng'
    >>> os.path.dirname(name)
    '/home'
    >>> os.path.basename(name)
    'liaoheng'
    >>> os.path.splitext(r'/root/install.log')
    ('/root/install', '.log')

>os的一些目录文件名操作，不会管那文件目录是不是真的存在，他只是对字符串操作


目录操作：

    >>> os.chdir(r'C:\Users')
    >>> os.getcwd()
    'C:\\Users'
    >>> os.path.abspath('') # empty string means the cwd
    'C:\\Users'
    >>> os.path.abspath('temp') # expand to full pathname in cwd
    'C:\\Users\\temp'
    >>> os.path.abspath(r'PP4E\dev') # partial paths relative to cwd
    'C:\\Users\\PP4E\\dev'
    >>> os.path.abspath('.') # relative path syntax expanded
    'C:\\Users'
    >>> os.path.abspath('..')
    'C:\\'
    >>> os.path.abspath(r'..\examples')
    'C:\\examples'
    >>> os.path.abspath(r'C:\PP4thEd\chapters') # absolute paths unchanged
    'C:\\PP4thEd\\chapters'
    >>> os.path.abspath(r'C:\temp\spam.txt')
    'C:\\temp\\spam.txt'

####在脚本中运行shell命令

python中运行shell方法：  

1. os.system
2. os.popen
3. command module
4. subprocess module
> command 和popen区别，popen返回一个类文件对象，command把输出结果返回成字符串对象


**os.system:**  
system方法会创建子进程运行外部程序，方法只返回外部程序的运行结果。这个方法比较适用于外部程序没有输出结果的情况。

    >>> import os  
    >>> os.system("echo \"Hello World\"")   # 直接使用os.system调用一个echo命令  
    Hello World ——————> 打印命令结果  
    0   ——————> What's this ? 返回值？  
    >>> val = os.system("ls -al | grep \"log\" ")   # 使用val接收返回值  
    -rw-r--r--  1 root   root   6030829 Dec 31 15:14 log——————> 此时只打印了命令结果  
    >>> print val 
    0   ——————> 注意，此时命令正常运行时，返回值是0  
    >>> val = os.system("ls -al | grep \"log1\" ")  
    >>> print val 
    256 ——————> 使用os.system调用一个没有返回结果的命令，返回值为256～  
    >>>   

__注意__：上面说了，此方法只会会外部程序的结果，也就是os.system的结果，所以如果你想接收命令的返回值，接着向下看


**os.popen:**  
当需要得到外部程序的输出结果时，本方法非常有用，返回一个类文件对象，调用该对象的read()或readlines()方法可以读取输出内容。比如使用urllib调用Web API时，需要对得到的数据进行处理。os.popen(cmd) 要得到命令的输出内容，只需再调用下read()或readlines()等 如a=os.popen(cmd).read()

    >>> os.popen('ls -lt')  # 调用os.popen（cmd）并不能得到我们想要的结果  
    <open file 'ls -lt ', mode 'r' at 0xb7585ee8>  
    >>> print os.popen('ls -lt').read() # 调用read()方法可以得到命令的结果  
    total 6064  
    -rwxr-xr-x 1 long   long23 Jan  5 21:00 hello.sh  
    -rw-r--r-- 1 long   long   147 Jan  5 20:26 Makefile  
    drwxr-xr-x 3 long   long  4096 Jan  2 19:37 test  
    -rw-r--r-- 1 root   root   6030829 Dec 31 15:14 log  
    drwxr-xr-x 2 long   long  4096 Dec 28 09:36 pip_build_long  
    drwx------ 2 Debian-gdm Debian-gdm4096 Dec 23 19:08 pulse-gylJ5EL24GU9  
    drwx------ 2 long   long  4096 Jan  1  1970 orbit-long  
    >>> val = os.popen('ls -lt').read() # 使用变量可以接收命令返回值  
    >>> if "log" in val:# 我们可以使用in来判断返回值中有木有一个字符串  
    ... print "Haha,there is the log"  
    ... else:  
    ... print "No,not happy"  
    ...  
    Haha,there is the log  

**command module:**  
使用commands模块的getoutput方法，这种方法同popend的区别在于popen返回的是一个类文件对象，而本方法将外部程序的输出结果当作字符串返回，很多情况下用起来要更方便些。
主要方法:
 
* commands.getstatusoutput(cmd) 返回(status, output)
* commands.getoutput(cmd) 只返回输出结果
* commands.getstatus(file)  返回ls -ld file的执行结果字符串，调用了getoutput，不建议使用此方法

**ex:**

    long@zhouyl:/tmp/tests$ python  
    Python 2.7.3 (default, Jan  2 2013, 16:53:07)   
    [GCC 4.7.2] on linux2  
    Type "help", "copyright", "credits" or "license" for more information.  
    >>> import commands  
    >>> commands.getstatusoutput('ls -lt')  # 返回(status, output)  
    (0, 'total 5900\n-rwxr-xr-x 1 long long  23 Jan  5 21:34 hello.sh\n-rw-r--r-- 1 long long 147 Jan  5 21:34 Makefile\n-rw-r--r-- 1 long long 6030829 Jan  5 21:34 log')  
    >>> commands.getoutput('ls -lt')# 返回命令的输出结果（貌似和Shell命令的输出格式不同哈～）  
    'total 5900\n-rwxr-xr-x 1 long long  23 Jan  5 21:34 hello.sh\n-rw-r--r-- 1 long long 147 Jan  5 21:34 Makefile\n-rw-r--r-- 1 long long 6030829 Jan  5 21:34 log'  
    >>> commands.getstatus('log')   # 调用commands.getoutput中的命令对'log'文件进行相同的操作  
    '-rw-r--r-- 1 long long 6030829 Jan  5 21:34 log'  
    >>>   


**subprocess:**  
根据Python官方文档说明，subprocess模块用于取代上面这些模块。有一个用Python实现的并行ssh工具—mssh，代码很简短，不过很有意思，它在线程中调用subprocess启动子进程来干活


subprocess模块主要使用这几个方法：

1. subprocess.Popen    
> class subprocess.Popen(args, bufsize=0, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=False, shell=False, cwd=None, env=None, universal_newlines=False, startupinfo=None, creationflags=0)

**notice：**  
args参数可以是一个字符串，也可以是一个列表  
看下面例子

    >>> import subprocess 
    >>> subprocess.Popen(["cat","install.log"]) #以列表读入参数，运行正常
    <subprocess.Popen object at 0x7fdd35724590>
    >>> liaoheng is pythoner!
    2 3
    liaoheng
    
    >>> subprocess.Popen("cat install.log") #以字符串读入参数，报异常
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "/usr/lib64/python2.6/subprocess.py", line 642, in __init__
    errread, errwrite)
      File "/usr/lib64/python2.6/subprocess.py", line 1234, in _execute_child
    raise child_exception
    OSError: [Errno 2] No such file or directory

这两个之中，后者将不会工作。因为如果是一个字符串的话，必须是程序的路径才可以。(考虑unix的api函数 exec，接受的是字符串列表)


但是下面的可以工作

    >>> subprocess.Popen("cat install.log",shell=True)
    <subprocess.Popen object at 0x7fdd357245d0>
    >>> liaoheng is pythoner!
    2 3
    liaoheng

他相当于
    
    >>> subprocess.Popen(["/bin/sh","-c","cat install.log"])
    <subprocess.Popen object at 0x7fdd35724590>
    >>> liaoheng is pythoner!
    2 3
    liaoheng

args = __["/bin/sh","-c","cat install.log"]__ ，argv[0]="/bin/sh" argv[1]="-c", argv[2]="cat install.log"

都成了sh的参数，就无所谓了


**在Windows下，确不存在这个问题**  
这是由于windows下的api函数CreateProcess接受的是一个字符串。即使是列表形式的参数，也需要先合并成字符串再传递给api函数。

开源中国[subprocess参考](http://www.oschina.net/question/234345_52660)


模块还提供了几个便利函数（这本身也算是很好的Popen的使用例子了）


call() 执行程序，并等待它完成

    def call(*popenargs, **kwargs):  
    	return Popen(*popenargs, **kwargs).wait()

check_call() 调用前面的call，如果返回值非零，则抛出异常


    def check_call(*popenargs, **kwargs):  
    	retcode = call(*popenargs, **kwargs)  
   		if retcode:  
    		cmd = kwargs.get("args")  
    		raise CalledProcessError(retcode, cmd)  
    	return 0  

check_output() 执行程序，并返回其标准输出

      def check_output(*popenargs, **kwargs):
      	process = Popen(*popenargs, stdout=PIPE, **kwargs)
      	output, unused_err = process.communicate()
      	retcode = process.poll()
      	if retcode:
      		cmd = kwargs.get("args")
      		raise CalledProcessError(retcode, cmd, output=output)
      	return output

popen对象还有很多其他方法：


    poll()
    检查是否结束，设置返回值
    wait()
    等待结束，设置返回值
    communicate()
    参数是标准输入，返回标准输出和标准出错
    send_signal()
    发送信号 (主要在unix下有用)
    terminate()
    终止进程，unix对应的SIGTERM信号，windows下调用api函数TerminateProcess()
    kill()
    杀死进程(unix对应SIGKILL信号)，windows下同上
    stdin
    stdout
    stderr
    参数中指定PIPE时，有用
    pid
    进程id
    returncode
    进程返回值


####os