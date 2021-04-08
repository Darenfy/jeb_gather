# jeb_gather

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
