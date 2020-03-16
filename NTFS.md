# NTFS

考察ntfs隐写

 在NTFS文件系统中存在着NTFS交换数据流（Alternate Data Streams，简称ADS），这是NTFS磁盘格式的特性之一。每一个文件，都有着主文件流和非主文件流，主文件流能够直接看到；而非主文件流寄宿于主文件流中，无法直接读取，这个非主文件流就是NTFS交换数据流。

于是，再用winrar解压后，利用专门的ntfs流查看工具获得了flag