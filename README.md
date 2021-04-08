# jeb_gather

## 简介
``JEB`` 参考文档：https://www.pnfsoftware.com/jeb/manual-3/

JEB 破解版下载链接：https://down.52pojie.cn/Tools/Android_Tools/

以下均以分析 ``APK`` 文件为例，平台为 ``macOS``
### 通用操作
重命名方法名或字段名：``N``

添加注释：``/``

查看注释：``Command + /``

正向导航/反向导航：``option + <-/->``

交叉引用：``X``

转换进制：``B``

查看类型层级：``H``

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
## 插件  
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
