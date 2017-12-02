# Android && Reverse

## Reverse

### Assembly
- 指令集
    - x86 / x64
	- ARM / ARM64
	- 一些小众架构 如：MIPS

- CPU运行方式
	
	- 取出下一条指令、执行当前指令、输出结果
	
    - 单位
        - ·BIT（位）：电脑数据量的最小单元，可以是0或者1。
        - ·BYTE（字节）：一个字节包含8个位，所以一个字节最大值是255(0-255)。为了方便阅读，我们通常使用16进制来表示。
        - ·WORD（字）：一个字由两个字节组成，共有16位。一个字的最大值是0xFFFF。
        - ·DWORD（双字DOUBLE WORD）：一个双字包含两个字，共有32位。最大值为0xFFFFFFFF。
        - .QWORD（四字 QUAD-WORD）：一个四字包含四个字，共64位。
	- ![Assembly.jpg](https://s1.ax2x.com/2017/11/21/FxZSl.jpg)

- 常用汇编指令
    
	- 运算指令：加 `ADD` 减 `SUB` 乘 `MUL` 除 `DIV` 异或 `XOR` 与 `AND` 或 `OR` 非 `NOT`
	- 数据转移指令：读取/写入内存 写入寄存器 `MOV`
	- 跳转指令：无条件跳转 `JMP` /条件跳转指令 `JZ`
	- 栈操作指令：出栈 `POP` 入栈 `PUSH`
	- 调用指令：`CALL`
	- 空指令：`NOP`
	

- x86 / x64
    - 寄存器
	    - 寄存器是CPU中的存储结构，用于暂存指令、数据、地址

	    - ![x86寄存器](https://s1.ax2x.com/2017/11/21/Fx40y.png)

	    - x64架构在x86的基础上又增加了R8-R15这8个寄存器
	    ```
	    高地址 ---------------------------- 低地址
        
	    [==== ==== ==== ==== ==== ==== ==== ====] RAX
	                        [==== ==== ==== ====] EAX
	                                  [==== ====] AX
	                                  [====]      AH
	                                       [====] AL
        ```
        
    - 标志寄存器
        - Z-Flag：零标志，ZF是我们在破解中用的最多的寄存器，因为 `CMP` 指令会修改ZF的值
        - O-Flag：溢出标志，当上一步操作改变了某一寄存器的最高有效位时，OF会被设置为1。
        - C-Flag：进位标志，假设某一32位寄存器值为0xFFFFFFFF，再加1就会产生溢出，寄存器的值会变为0x00000000。
    - [wiki](https://zh.wikibooks.org/wiki/X86_%E6%B1%87%E7%BC%96)

- ARM
    - 寄存器
	    - R0-R15，其中R13与ESP相当，R11与EBP相当
	    - R14是LR(link register)，用于存储返回地址
	    - R15是PC(program counter)，其值为当前执行的指令地址+8
	- [ARM汇编详解](https://azeria-labs.com)
	
	- ![ARM](https://azeria-labs.com/downloads/cheatsheetv1-1920x1080.png)

- 堆&栈
    -  push 入栈 pop 出栈，先入后出
    - TODO: @XuCcc
	

### Tools
- [010Editor](https://www.52pojie.cn/thread-399761-1-1.html)
![010Editor](https://s1.ax2x.com/2017/11/21/FxyJB.jpg)


- [CFF Explorer](https://down.52pojie.cn/Tools/PEtools/CFF_Explorer.zip)
![CFF Explorer](https://s1.ax2x.com/2017/11/21/FxvTJ.jpg)



- IDA Pro: Interactive DisAssembler
![IDA Pro](https://s1.ax2x.com/2017/11/21/FxJaX.jpg)



- 调试器：gdb、WinDBG、OllyDBG、x64dbg、IDA的内置调试器



## Android
### Android系统背景
- Android 操作系统结构
    ![Android](https://s1.ax2x.com/2017/11/23/H9T7z.png)
    
- 碎片化
    ![Android Dashboard](https://s1.ax2x.com/2017/11/23/H9zyh.jpg)
    
    ![IOS](https://s1.ax2x.com/2017/11/23/H9V8H.jpg)
    
- 签名
    
- root
    - 直接使用Recovery刷入su和daemon-su
    - 使用BootLoader替换原Recovery之后，再刷入su和daemon-su
    - 使用内核漏洞Exploit获取root权限
- Andoird Application
    - .java -> .class -> .dex -> .apk （`smali` / `baksmali` `dex2jar`）
    - JNI调用Native代码（`IDA`）

### Andoird 逆向
- 安卓逆向工具：
反汇编，重汇编： `apktool` ， `shakaApktool`， `Apktoolbox` 等
静态分析： `jeb`，`apk改之理(ApkIDE)` ， `Android Killer` 等
动态分析：`IDA`，`Android Studio`，`Eclipse` 等
Android sdk自带：`adb` ， `ddms`

- smali
    - Dalvik虚拟机中的汇编语言
    - [语法](http://lib.csdn.net/article/android/7043)

- 代码保护与逆向技术的对抗
    1. 编译与反编译
        - Java层
            - java -> jvm字节码 -> dalvik字节码
            - .dex -> smali（`smali`）
            - .dex -> smali, java（`Jeb`）
            - .dex -> java字节码（`dex2jar`）
        - Native层
            - C/C++ -> .so
            - .so -> 汇编/伪C（`IDA`）
    2. 加壳与脱壳
        - 基于加固平台/开发者自行加固
        - 脱壳
            - Hook技术：基于Xposed的ZJDroid
            - Dump内存重建dex文件 
            - DexHunter/AppSpear：基于修改Android系统runtime的通用脱壳机

### 配置Android开发环境
- 安装jdk与jdk环境变量配置([参考资料](http://jingyan.baidu.com/article/6dad5075d1dc40a123e36ea3.html))
    1. 配置jdk环境变量：
    ```
	计算机 -> 属性 -> 更改设置 -> (系统属性)高级 -> 环境变量 -> 系统变量
	```
    2. 新建：
    ```
    JAVA_HOME -> JDK安装路径
    CLASSPATH -> ...;%JAVA_HOME%/lib/dt.jar;%JAVA_HOME%/lib/tools.jar
    ```
    3. 编辑:
    ```
    PATH -> %JAVA_HOME%/bin;%JAVA_HOME%/jre/bin
    ```

- 安装Android sdk与Android sdk环境变量配置

    **配置Andriod环境变量前提是要先安装好JAVA环境**
    
    1. 下载 [Android SDK](http://developer.android.com/sdk/index.html)，点击安装，直接默认路径即
    
    2. 修改默认路径安装后，安装完成，开始配置环境变量。
    
    3. 打开计算机属性——高级系统设置——环境变量
    
    4. 新建一个环境变量，变量名：ANDROID_HOME，变量值：D:\adt-bundle-windows-x86_64-20140702\sdk（以你安装目录为准,确认里面有tools和add-ons等多个文件夹），点击确认。
    
    5. 在用户变量PATH后面加上变量值;%ANDROID_HOME%\platform-tools;点击确认即可。在系统变量path中添加
    D:\adt-bundle-windows-x86_64-20140702\sdk\tools
    
    6. Android SDK配置完成，接下来验证配置是否成功。
    
    7. 点击运行——输入cmd——回车——输入adb——回车，如果没有提示未找到命令即表示配置成功，在输入Android，启动Android SDK Manager。

- 虚拟机安装
安卓虚拟机目前种类繁多，推荐使用 `Android Studio` 作为开发环境，自带虚拟机，当然，有条件上实体机也是不错的选择，毕竟虚拟机有时会发生奇怪的问题。

- 前面提到的一些工具 [百度网盘](http://pan.baidu.com/s/1nuYwLJVx) 密码：ar60

## How To Reverse
- 去除软件保护
    1. 查壳
        - PEiD、ExeInfoPE等等
    2. 脱壳
        - 搜索对应的脱壳机
        - ESP定律脱壳（仅适用于压缩壳）
    3. 去除花指令
        - 使用OllyDBG脚本或IDC&IDA Python脚本
        - 手动patch花指令
    4. 去除混淆
- 定位验证代码
    1. 正面硬肛
        - 从程序入口（main函数）开始，逐函数分析
        - 慢慢深入，达到验证代码位置
        - 静态分析
        
    2. 从输入输出处寻找
        - 查找调用输入输出函数的地方，如：scanf、get、cin等等
        - 进而回溯处理（验证）过程
    3. 查询关键字符串
        - 搜索关键字符串直接定位验证函数
- 比赛中常见算法
    1. 没算法
    2. 简单的异或
    3. 雪崩式的异或（前一步的结果影响下一步的结果）
    4. 加密算法（RSA、AES、RC4）
    5. 摘要算法（SHA1、MD5）
    6. 编码（Base64）
    7. 解方程
    8. 脑洞算法（走迷宫……）
- 小技巧
    1. 快速定位关键代码
        - 从 Function List 前面开始翻
        - 从 main 函数周围翻
        - 一般来说，用户代码部分不会有一些奇怪的算法，如果看到了奇怪的算法，就有可能是进入了系统函数
        
    2. MFC 逆向
        - 使用 xspy 工具查看消息处理函数
        - 之后就与正常逆向一致了
- 动态调试
    - IDA Pro
        1. Windows
            一般来说，使用 `Local Windows Debugger`，直接在本机调试，如果需要在虚拟机中调试可以参照Linux的设置方法
        2. Linux
            - 这里以VMWare为例，配置网络为NAT模式， 关闭虚拟机防火墙。调试器选择 `Remote Linux Debugger` ，点击Debugger->Process Options，填入虚拟机的ip和端口。
            - 将IDA目录下的 `dbgsrv` 文件夹中对应的调试服务端放入虚拟机，运行，然后就可以开始调试了。
            - **注意待调试程序是32位和64位，使用对应调试服务端**
        3. Android
            - 虚拟机调试
                - 首先按照前文说明搭建好 `Android SDK` 环境，并新建一台虚拟机
                - 通过 `adb push` 命令将 `dbgsrv` 下的调试服务端推送至虚拟机内
                - 利用 `adb shell` 进入对应目录，修改权限并启动
                ```
                chown 0.0 ./android_server
                chmod 755 ./android_server
                ./android_server
                ```
                
                - 新开一个命令行，执行端口转发
                ```
                adb forward tcp:23946 tcp:23946
                ```
                
                - 调试器选择 `Remote ARMLiunx/Android Debugger`，进入Process Options，ip填127.0.0.1，端口为23946
                
            - 真机调试
                - 建议先root手机，否则会由于应用设置了 `debuggable=false` 而无法调试
                - 安装 Xposed 框架，在其他设置中勾选调试其他应用
                - 其余步骤同虚拟机 
                
            - 调试过程
                1. 解压apk文件，将其中的dex文件用IDA打开
                2. 修改Debugger Option
                    - 勾选 'Suspend on process entry point'
                    - ![Step1](https://s1.ax2x.com/2017/11/21/Fg9WH.jpg)
                    - 配置虚拟机ID和apk包名
                    ```
                    查看虚拟机ID
                    adb devices
                    ```
                    
                    - ![Step2](https://s1.ax2x.com/2017/11/21/Fg3nh.jpg)
    
- 现实与比赛的不同
    1. 现实：
        - 代码量巨大
        - 大量使用其他库
        - 各种奇怪的编码&语言
        - 大量现代语言特性
        - 性能优化&加密壳
        - 一些冷门的编程语言
    2. CTF比赛：
        - 代码量小
        - 结构简单
        - 现代语言特性少
        - 很少使用加密壳或优化
        - 语言比较常见

## THE END
- **注意保护自己**，比赛一般不会遇到，但在分析现实中软件（特别是病毒和外挂）的时候一定要注意，开发者可能会留有暗桩（关机重启，甚至格盘）
    - 修改可执行文件后缀名，防止误运行
    - 绝对！不要！在真机运行不可信的软件！同时，也不要在真机进行动态调试，很容易跑飞。
    - 推荐打开360或与其同等强度的HIPS（Host-based Intrusion Prevention System）防御软件
- Books
    - 《加密与解密》
    - 《IDA Pro权威指南》
    - 《Android软件安全与逆向分析》

部分内容参考 @Misty & 腾讯玄武实验室@刘惠明
    








