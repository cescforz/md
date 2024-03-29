

**模块名：** 
小写字母，单词之间用_分割 
参考python：logging

**包名:** 
小写字母，单词之间用_分割 
参考python：logging

**类名：** 
单词首字母大写 
参考：python class LogRecord(object):

**普通变量：** 
小写字母，单词之间用_分割 
参考：exc_info

**实例变量：** 
以*开头，小写字母，单词之间用*分割 
参考:_exc_info 
以一个下划线开头的标识符(_xxx)，不能访问的类属性，但可通过类提供的接口进行访问， 
不会被语句 “from module import *” 语句加载

**私有实例变量：** 
以*_开头（2个下划线），小写字母，单词之间用*分割 
参考:__private_var 
外部访问会报错

**专有变量：** 
**开头，**结尾，一般为python的自有变量，不要以这种方式命名 
参考:**doc** 
是系统定义的，具有特殊意义的标识符

**普通函数：** 
小写字母，单词之间用_分割： 
参考:get_name()

**私有函数**： 
以__开头（2个下划线），小写字母，单词之间用分割 
参考:__get_name() 
外部访问会报错

**注意：** 
_单下划线开头：弱“内部使用”标识，如：”from M import *”，将不导入所有以下划线开头的对象，包括包、模块、成员 
单下划线结尾_：只是为了避免与python关键字的命名冲突 
__双下划线开头：模块内的成员，表示私有成员，外部无法直接调用 
包和模块：模块应该使用尽可能短的、全小写命名，可以在模块命名时使用下划线以增强可读性。同样包的命名也应该是这样的，虽然其并不鼓励下划线。 
以上这些主要是考虑模块名是与文件夹相对应的，因此需要考虑文件系统的一些命名规则的，比如Unix系统对大小写敏感，而过长的文件名会影响其在Windows/Mac/Dos等系统中的正常使用。 
类：几乎毫无例外的，类名都使用首字母大写开头(Pascal命名风格)的规范。使用_单下划线开头的类名为内部使用，上面说的from M import *默认不被告导入的情况。 
异常：因为异常也是一个类，所以遵守类的命名规则。此外，如果异常实际上指代一个错误的话，应该使用“Error”做后缀