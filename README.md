#WafBook CN
Thomas Nagy

建立了一个QQ群, 欢迎大家来交流 : WAF Build System讨论交流 54669543

###引言
Copyright © 2010-2016 Thomas Nagy
Copies of this book may be redistributed, verbatim, and for non-commercial purposes. The license for this book is by-nc-nd license.

####关于build系统
随着软件越来越复杂，功能越来越丰富，创建软件所需的过程也越来越多。复杂性的增加自然来自项目的基本需求。
　　1.可用的软件(程序)是机器可读的但通常不是人类可读的。
	2.因此，被称为源代码的人类可读的输入数据被编译器或归档器转换为可分发的软件。
	3.源代码通常被分解为互相依赖的单元例如文件，模块，类和函数。
	4.这些单元随着时间的版本演变是通过版本控制软件管理的。
	5.其他程序由源代码生成附加数据如测试结果及静态分析结果。
	6.源代码的处理过程耗时而且容易出错所以通过其他软件帮助其自动化。
	