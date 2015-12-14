#optparse — Parser for command line options

##1. Tutorial

第一，需要导入OptioParser类，并创建OptinParser实例：

    from optparse import OptionParser
    [...]
    parser = OptionParser()

之后，你定义选项，基本语法如下：

    parser.add_option(opt_str, ...,
    	attr=value, ...)

每个选项可能有1个或多个字符串组成，比如： -f 或者--file

    parser.add_option("-f", "--file", ...)

把选项参数解析到程序中：

	(options, args) = parser.parse_args()
>(If you like, you can pass a custom argument list to parse_args(), but that’s rarely necessary: by default it uses sys.argv[1:].)

选项option最终要的4个属性：action, type, dest (destination), and help

###1.1 The store action



