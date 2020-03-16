# Shell

打开数据包，选择追踪tcp流，发现有cd，ls，cat等shell操作，尤其是cat，在其命令下方出现了一串字符串MXHz3TYzxzLCNnLx3nOzwWXBdfSBgWXBh0k，而cat命令为 cat flag.txt | base64 -w 0 | python -c "print raw_input().swapcase()

可以看到是先对其进行了base64编码，而后用python的swapcase（）函数进行操作后输出，查询该函数，发现是大小写替换函数，于是写代码将该字符串再调用一次swapcase函数后用base64解码后输出得到flag