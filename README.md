# PyGP
GP(Galgame Platform)的Python实现方式。YSP的高级逻辑脚本GPOL现在通过Python实现。

最初，我们设想通过C++完全重构一个Python解释器专门用于实现GPOL，但这样做的工作量过大，而且也不便于对语法进行标准化以及后期维护。于是我们通过在Python中构建GP库环境来完成GPOL脚本的实现。

欲编写GPOL脚本，需要先安装Python，然后将本项目里PyGP文件下的py文件完整拷贝到Python项目中。并使用这些文件中定义的GP对象完成GPOL编写。编写出的.py文件在修改后缀为.gpol后可以被cYSP程序识别，并启用cYSP附带的Python环境动态执行。

PyGP的实现方式非常简单粗暴，PyGP背后的函数均能够对本地端口11451发送由操作类型、命名空间、函数\对象名称、参数内容、返回要求等3~5个字段组成的文本。cYSP会监听11451端口上的这些文本，并解析为cYSP程序内部的函数，当需要返回值时，cYSP会在11452端口上返回数值，此时Python程序会监听11452端口并获取数值返回给PyGP函数。

这样通过本地端口通信来实现程序功能是比较低效的，但由于GPOL本身只是一个用来开发视觉小说逻辑过程与前端过程的脚本，本身不涉及到密集运算与密集通信，因此这样的实现速度是可以接受的（至少目前如此）

对于PyGP，有正在开发的适用例如下：
'''python
import GPplatform.GPCore as platform
import GPplatform.GPObject as GObject
import GPplatform.GPWidgets as GWidgets

platform.Tag.GPOL_MAIN()
platform.MainPageUI.setStyleSheet("QWidget{color:#FFFFFF;}")
platform.MainPageUI.setStyleGPOL("Style.gpol")
if platform.MainPageUI.getStyleGPOLName() != "Style":
    platform.Console.print("can not set Style Correctly")
    platform.Program.exit()
platform.StoryControll.setStoryEntrance("TS1.spol")
platform.StoryControll.start()
platform.Tag.GPOL_END()
'''
在上面的例子中，GPOL脚本从platform.Tag.GPOL_MAIN()开始，从platform.Tag.GPOL_END()结束。可以将其视作所谓的主函数。Python程序从GPOL_MAIN()开始连接11451端口，监听11452端口，从GPOL_END()开始断开11451端口，停止监听11452端口。所以在这两个Tag之外的其他PyGP语句都不能正常执行，会引起报错。

platform.MainPageUI是设置程序主页对象，拥有多个成员函数。这些成员函数的名字大多直接来自于Qt。若cYSP提供了一些Qt函数之外的成员函数，则这些函数的名字是新命名的。由于cYSP由Qt编写，依赖Qt机制，因此GPOL中一切设置StyleSheet的文本均为Qt的QSS文本。后期可能会通过转换过程弱化Qt的概念，但至少目前如此。

platform.Console是cYSP程序的控制台窗口。直接使用Python的print会把信息打印在Python的控制台上，这在开发时无伤大雅，但作为.gpol文件被cYSP附带的Python执行时，该信息会不可见。所以需要将输出打印在cYSP的控制台上。

其他内容不再多做解释，由于PyGP还在开发，因此可能会有较大的后续变更。总之，目前已经搭建起的框架形如此，仅供参考。
