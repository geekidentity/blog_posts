
# 远程调试命令参数

* -Xdebug: 启用高度特性。
* -Xrunjdwp:<sub-options>: 在目标VM中加载JDWP实现。它通过传输和JDWP协议与独立的调试器应用程序通信。下面介绍一些特定的子选项。
* 从Java5开始，您可以使用-agentlib:jdwp 选项，而不是-Xdebug 和-Xrunjdwp。但如果连接到Java5 以前的VM，只能选择-Xdebug 和-Xrunjdwp。
* -Xrunjdwp 子选项
    * transport: 这里通常使用套接字传输。但是在Windows平台上也可以使用共享内存传输。
    * Server: 如果值为y, 目标应用程序监听将连接的调试器应用程序。否则，它将连接到特定地址上的调试器应用程序。
    * address: 这是连接的传输地址。如果服务器民n, 将尝试连接到该地址上的调试应用程序。否则，将在这个端口监听连接。
    * suspend: 如果值为y, 目标VM将暂停，直到调试器应用程序进行连接。