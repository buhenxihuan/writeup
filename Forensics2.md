# Forensics2

同样是流量分析题，使用wireshark打开，首先尝试过滤http，发现没有，于是继续观察，发现一个nfs协议，搜索NFS协议相关内容，得知其全称为Network file system，相当于一个远程文件共享协议，进一步查找发现其传输方式一般为明文，且读写操作一般含有参数 read 和  write，同时发现一个这样的数据

```
391	58.281933	10.0.0.22	10.0.0.2	NFS	298	V4 Call (Reply In 395) OPEN DH: 0xfc0e5415/flag.txt.gz
```

于是搜寻，终于在一个含wirte的报文中发现了参数date有数据，将其提取得到flag