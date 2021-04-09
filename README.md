# jeb_gather

## 简介
``JEB`` 参考文档: https://www.pnfsoftware.com/jeb/manual-3/

JEB 破解版下载链接: https://down.52pojie.cn/Tools/Android_Tools/

JEB 3.0版本下载: https://pan.baidu.com/s/1mx8NQMddVSTb3GTKIed8nw 提取码: fdiy

以下均以分析 ``APK`` 文件为例，平台为 ``macOS``
### 通用操作
重命名方法名或字段名：``N``

添加注释：``/``

查看注释：``Command + /``

正向导航/反向导航：``option + <-/->``

交叉引用：``X``

转换进制：``B``

查看类型层级：``H``

查看图：``Space``

反编译：``Tab``

### 反编译
JEB Pro 支持以下类型的反编译：
- Dalvik (Android DEX files)
- Java (classfile)
- WebAssembly modules (wasm)
- Ethereum contracts (EVM code)
- Intel x86 32-bit (all x86 - x87/mmx/sse/avx support coming in JEB 4)
- Intel x86 64-bit (all x86 - x87/mmx/sse/avx support coming in JEB 4)
- ARM 32-bit (and common ISA extensions)
- ARM 64-bit (v8 / aarch64)
- MIPS 32-bit

### 插件
用来分析 Android apps 的插件主要包括：
- APK 插件负责处理 APK 文件。能够解码 arsc 文件
- DEX 插件负责 DEX 分析、反编译、调试
- Native 代码分析器：反汇编、反编译等
- other plugins：Certificate 解析、XML/HTML/JSON 等解析


---
## 相关内容
### [动态 JNI 检测插件](https://github.com/pnfsoftware/jnihelper)  
在注册 Native 方法时，可以弃用命名规则，而选用 JNIEnv->RegisterNatives 方法将函数绑定至 Java 层

由于动态的性质，当命名符号被删除、修改，代码被混淆时，静态解析这些绑定可能会很困难，因此并非所有对 RegisterNatives 的调用都可以找到或被成功处理

插件运行的顺序：
- 注释所有带有目标地址的 dex 代码
- 重命名目标（使用 __jni_ 作为前缀）
- 允许无缝调试（从 Java 跳转到 JNI 方法）

插件使用了几种启发式方法，为 ARM 和 ARM64 实现
1. JNIEnv->RegisterNatives 方法通常从标准 JNI 初始化函数 JNI_OnLoad 调用
    JEB 搜索此方法，并尝试获取对 RegisterNatives 的调用，一旦发现 ``BL RegisterNatives``，则使用反编译器创建 IR 块并确定 R2 和 R3 寄存器的值，R2 是一个指向 JNI native methods 数组的指针，该数组结构包含方法名、方法签名、native function bound
    但是通过寄存器跳转或方法名隐藏时，该方法无效
2. 基于方法名
   在 Dalvik 中，搜索所有 native 方法的调用;对于每一个找到的方法，在二进制中搜索是否有相匹配的字符串引用;查找此字符串的交叉引用，并检查是否有相应的 JNI 结构
   注：该启发式方法存在危险性，但是能够有相当好的结果
3. 基于参数 —— 与上一种相同
   由于命名可以被缩短，可能不会被解释成字符串，因此不会被引用

仅当将方法定义为静态数组变量时，这三种启发式方法才起作用。 动态变量将需要模拟JNI_OnLoad方法才能解决

### [JEB 本地类型和类型库](https://www.pnfsoftware.com/blog/native-types-and-typelibs-with-jeb/)  
什么是类型库？  
类型库是存储在 ``JEB`` typelibs 中的 ``*.typelib`` 文件，包含给定组件（操作系统或者 ``SDK``）的类型信息，如：
- 类型（别名、结构、枚举等）和原型（虚函数指针）
- publicly exported custom  
- 常量

**UI 客户端使用本地类型**
- 类型使用  
    JEB 使用类型非常直接，如果文件目标环境能够被识别，然后对应的类型库将会被加载，其中的类型将对用户来说是可用的  
    可以通过 ``File -> Engine -> TypeLibraries`` 查看所用到的类型库，并在代码中右键修改数据类型
- 类型编辑  
    JEB 允许编辑已经存在的复杂类型（结构和派生物）以及新类型的定义，快捷键: ``Cmd+Alt+T``
- 常量  
    常见的常量通常是在 ``#DEFINE`` 预处理命令中定义在头文件的，可以使用他们替换在汇编或反编译视图中的立即数

**定制类型库**  
JEB 存在脚本允许用户创建自己的类型库  
应用场景：
- 分析 ``Windows`` 内核组件时使用 ``Driver Kit headers``，他们的类型没用添加到预构建的 ``WDK`` 类型库中
- X platform 的类型没有被编译到指定架构中
- 待分析的二进制文件所需的第三方 ``SDK`` 没有类型库

**使用 JEB API 访问**  
本地类型像 ``JEB`` 的其他组件一样都可以通过 ``API`` 访问，脚本和插件可以使用 API 以编程方式获取、定义、使用类型，操作类型库

### 脚本  
https://github.com/LeadroyaL/JebScript  
> ``GotoClass``  便捷跳转  
> ``FastXposed/FastFrida`` 方便生成 ``Xposed`` 和 ``Frida`` 代码  

https://github.com/acbocai/jeb_script  
一些常用的基础代码分析操作,可用于反混淆/路径分析/代码定位等

https://github.com/Hamz-a/jeb2frida  
使用 ``JEB`` 自动化生成 ``Frida hook`` 脚本，应用方法为 apk 字符串未经混淆时，对特定字符串进行查询并获取所属类信息
