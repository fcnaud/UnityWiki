### addr2line

是 NDK 自带的调试工具，用来分析 so 崩溃时输出的地址。

在 NDK 目录下，查找 `arm-linux-androideabi-addr2line.exe`。

### 命令行参数

`arm-linux-androideabi-addr2line -C -f -e so库文件的路径 具体的内存地址`