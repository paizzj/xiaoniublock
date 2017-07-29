# bitcoind: Bitcoin Core Daemon , 比特币核心守护进程

源码文件位置: [~/bitcoin/src/bitcoind.cpp][bitcoind]

入口函数很简单
<pre><code>
int main(int argc, char* argv[])
{
    SetupEnvironment();

    // Connect bitcoind signal handlers
    noui_connect();

    return (AppInit(argc, argv) ? EXIT_SUCCESS : EXIT_FAILURE);
}
</code></pre>


		    











[bitcoind]:https://github.com/bitcoin/bitcoin/blob/master/src/bitcoind.cpp

