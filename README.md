# WafBook CN
Thomas Nagy

建立了一个QQ群, 欢迎大家来交流 : WAF Build System讨论交流 54669543

### 引言
Copyright © 2010-2016 Thomas Nagy
Copies of this book may be redistributed, verbatim, and for non-commercial purposes. The license for this book is by-nc-nd license.

#### 关于build系统
随着软件越来越复杂，功能越来越丰富，创建软件所需的过程也越来越多。复杂性的增加自然来自项目的基本需求。  
　　　　1. 可用的软件(程序)是机器可读的但通常不是人类可读的。  
　　　　2. 因此，被称为源代码的人类可读的输入数据被编译器或归档器转换为可分发的软件。  
　　　　3. 源代码通常被分解为互相依赖的单元例如文件，模块，类和函数。  
　　　　4. 这些单元随着时间的版本演变是通过版本控制软件管理的。  
　　　　5. 其他程序由源代码生成附加数据如测试结果及静态分析结果。  
　　　　6. 源代码的处理过程耗时而且容易出错所以通过其他软件帮助其自动化。
虽然有时编译器有时会提供完全自动的build，但他们通常仅限于非常特别的功能上。例如，Java编译器（javac）能够一次生成整个源码树，但另一种编译器需要生成归档文件（jar）。文本编辑器和集成开发环境（IDE如Xcode或Eclipse）能够提供自动生成功能，但是他们这样的用户界面最适合的工作是编写软件而不是运行生成。版本控制系统例如Git最适合管理终端用户的文件，但通常不适合调用编译器或者运行脚本。虽然解决方案集成系统（Jenkins, Teamcity）有时被理解为生成系统，但他们通常无法独立生成软件。他们需要生成脚本以及运行在他们生成代理（build agents Maven, Make, etc）上的生成软件。