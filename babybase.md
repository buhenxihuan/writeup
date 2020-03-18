# BabyBase

考察base系列的编码和解码，简单的用base64，base32和base16都解码了一遍，发现只有base16解码成功，但没有出现flag，考虑到提示，猜测可能不只用了一种base编码，可能是base系列编码的套娃，考虑到base系列是将数据全部转换成可打印的ASCII码，则base的一系列套娃产生的密文应该都是可打印的ASCII码，则考虑使用python进行循环解码，并使用它的异常处理进行判断，关键代码如下

```
from base64 import *
import binascii

flag = ""
with open('base.txt','rb') as f:
	flag = f.read()

while True:
	try:
		flag = b16decode(flag)
		print("b16")
		if b'flag' in flag:
			break
	except binascii.Error:
		try:
			flag = b32decode(flag)
			print("b32")
			if b'flag' in flag:
				break
		except binascii.Error:
			try:
				flag = b64decode(flag)
				print("b64")
				if b'flag' in flag:
					break
			except binascii.Error:
				f.close()
with open('flag.txt','wb') as f2:
	f2.write(flag)
print(flag)
```

最终解出flag