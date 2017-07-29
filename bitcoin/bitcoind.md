# bitcoind: Bitcoin Core Daemon , 比特币核心守护进程

源码文件位置: [~/bitcoin/src/bitcoind.cpp][bitcoind]

[TOC]


## 入口函数
<pre><code>
int main(int argc, char* argv[])
{
    SetupEnvironment();

    // Connect bitcoind signal handlers
    noui_connect();

    return (AppInit(argc, argv) ? EXIT_SUCCESS : EXIT_FAILURE);
}
</code></pre>

### SetupEnviroment()
SetupEnvironment()见文件util.h/util.cpp     
功能: 主要设置语言环境。    
详细的语言设置参见：[c++标准库]第16章   


### noui_connect()   
noui_connect()  见文件noui.h/nohui.cpp    
功能:主要是设置信号处理函数。    
详细的信号处理函数用法 ：参见[boost程序库完全参考指南]第11章(函数与回调)signals2   


### AppInit()











[bitcoind]:https://github.com/bitcoin/bitcoin/blob/master/src/bitcoind.cpp

