

# 拆弹

#### phase_1

```
08048b90 <phase_1>:
 8048b90:	83 ec 1c             	sub    $0x1c,%esp
 8048b93:	c7 44 24 04 e4 a1 04 	movl   $0x804a1e4,0x4(%esp)//将一个地址放在esp+4处
 8048b9a:	08 
 8048b9b:	8b 44 24 20          	mov    0x20(%esp),%eax
 8048b9f:	89 04 24             	mov    %eax,(%esp)//将读入的参数的首地址放在esp处
 8048ba2:	e8 03 05 00 00       	call   80490aa <strings_not_equal>
 8048ba7:	85 c0                	test   %eax,%eax
 8048ba9:	74 05                	je     8048bb0 <phase_1+0x20>//调用函数判断字符串是否相等
 8048bab:	e8 05 06 00 00       	call   80491b5 <explode_bomb>
 8048bb0:	83 c4 1c             	add    $0x1c,%esp
 8048bb3:	c3 
```

从上面的分析可以看出，phase_1主要就是一个判断输入字符串和地址0x804a1e4处的字符串相等，不相等则引爆，故只要知道0x804a1e4处的数据即可以过关，使用gdb的x命令获得字符串如下

```
(gdb) x /s 0x804a1e4
0x804a1e4:	"Border relations with Canada have never been better."
```

#### phase_2

```
08048bb4 <phase_2>:
 8048bb4:	56                   	push   %esi
 8048bb5:	53                   	push   %ebx
 8048bb6:	83 ec 34             	sub    $0x34,%esp
 8048bb9:	8d 44 24 18          	lea    0x18(%esp),%eax
 8048bbd:	89 44 24 04          	mov    %eax,0x4(%esp)//将esp+18的地址放在esp+4处
 8048bc1:	8b 44 24 40          	mov    0x40(%esp),%eax
 8048bc5:	89 04 24             	mov    %eax,(%esp)//将读入的数据的首地址放在esp处
  //将读入的六个数字依次放在从esp+18处开始的位置
 8048bc8:	e8 0f 06 00 00       	call   80491dc <read_six_numbers>
 //esp+18处应为0，否则爆炸，即第一个参数为0
 8048bcd:	83 7c 24 18 00       	cmpl   $0x0,0x18(%esp)
 8048bd2:	75 07                	jne    8048bdb <phase_2+0x27>
 //esp+1c处应为1，否则爆炸，即第二个参数为1
 8048bd4:	83 7c 24 1c 01       	cmpl   $0x1,0x1c(%esp)
 8048bd9:	74 1f                	je     8048bfa <phase_2+0x46>
 8048bdb:	e8 d5 05 00 00       	call   80491b5 <explode_bomb>
 8048be0:	eb 18                	jmp    8048bfa <phase_2+0x46>
 8048be2:	8b 43 f8             	mov    -0x8(%ebx),%eax//将ebx-8的内容放入eax
 8048be5:	03 43 fc             	add    -0x4(%ebx),%eax//eax = (ebx-4)+(ebx-8)
 //比较后两单元的值加起来是否等于本单元的值，不等则爆炸，很容易知道是斐波那契数列，故答案可知
 8048be8:	39 03                	cmp    %eax,(%ebx)
 8048bea:	74 05                	je     8048bf1 <phase_2+0x3d>
 8048bec:	e8 c4 05 00 00       	call   80491b5 <explode_bomb>
 8048bf1:	83 c3 04             	add    $0x4,%ebx
 8048bf4:	39 f3                	cmp    %esi,%ebx
 8048bf6:	75 ea                	jne    8048be2 <phase_2+0x2e>
 8048bf8:	eb 0a                	jmp    8048c04 <phase_2+0x50>
 8048bfa:	8d 5c 24 20          	lea    0x20(%esp),%ebx//第三个参数的地址放入ebx
 8048bfe:	8d 74 24 30          	lea    0x30(%esp),%esi//将esp+30的地址放入esi
 8048c02:	eb de                	jmp    8048be2 <phase_2+0x2e>
 8048c04:	83 c4 34             	add    $0x34,%esp
 8048c07:	5b                   	pop    %ebx
 8048c08:	5e                   	pop    %esi
 8048c09:	c3                   	ret    
```

根据上述分析，知道输入的应为斐波那契数列的前六项，即为 0 1 1 2 3 5

#### phase_3

```
08048c0a <phase_3>:
 8048c0a:	83 ec 3c             	sub    $0x3c,%esp
 8048c0d:	8d 44 24 2c          	lea    0x2c(%esp),%eax
 8048c11:	89 44 24 10          	mov    %eax,0x10(%esp)//将esp+2c的地址放到esp+10处
 8048c15:	8d 44 24 27          	lea    0x27(%esp),%eax
 8048c19:	89 44 24 0c          	mov    %eax,0xc(%esp)//将esp+27的地址放到esp+c处
 8048c1d:	8d 44 24 28          	lea    0x28(%esp),%eax
 8048c21:	89 44 24 08          	mov    %eax,0x8(%esp)将esp+28的地址放到esp+8处
 8048c25:	c7 44 24 04 42 a2 04 	movl   $0x804a242,0x4(%esp)//将0x804a242放到esp+4处，利用gdb，可知0x804a242处开始的字符为 %d %c %d
 8048c2c:	08 
 8048c2d:	8b 44 24 40          	mov    0x40(%esp),%eax
 8048c31:	89 04 24             	mov    %eax,(%esp)//将读入数据的首地址放入esp处
 //调用sscanf函数，查询该函数的参数格式，可知该函数是把输入的数据按照 %d %c %d 的格式放到了上面注释
 //的esp+28，esp+27，esp+2c处
 8048c34:	e8 27 fc ff ff       	call   8048860 <__isoc99_sscanf@plt>
 8048c39:	83 f8 02             	cmp    $0x2,%eax//返回值小于等于2爆炸
 8048c3c:	7f 05                	jg     8048c43 <phase_3+0x39>
 8048c3e:	e8 72 05 00 00       	call   80491b5 <explode_bomb>
 8048c43:	83 7c 24 28 07       	cmpl   $0x7,0x28(%esp)//第一个参数大于7则爆炸
 8048c48:	0f 87 f5 00 00 00    	ja     8048d43 <phase_3+0x139>
 8048c4e:	8b 44 24 28          	mov    0x28(%esp),%eax//将第一个参数放入eax
 //switch case的跳转表地址，根据eax的值跳转到相应的位置，故假设是0，则直接跳到0x8048c59，其余类似
 8048c52:	ff 24 85 60 a2 04 08 	jmp    *0x804a260(,%eax,4)
 8048c59:	b8 75 00 00 00       	mov    $0x75,%eax//将0x75放入eax
 8048c5e:	81 7c 24 2c d7 01 00 	cmpl   $0x1d7,0x2c(%esp)//第三个参数和471比较，不等爆炸
 8048c65:	00 
 8048c66:	0f 84 e1 00 00 00    	je     8048d4d <phase_3+0x143>
 8048c6c:	e8 44 05 00 00       	call   80491b5 <explode_bomb>
 8048c71:	b8 75 00 00 00       	mov    $0x75,%eax
 8048c76:	e9 d2 00 00 00       	jmp    8048d4d <phase_3+0x143>
 8048c7b:	b8 6d 00 00 00       	mov    $0x6d,%eax
 8048c80:	81 7c 24 2c e1 01 00 	cmpl   $0x1e1,0x2c(%esp)
 8048c87:	00 
 8048c88:	0f 84 bf 00 00 00    	je     8048d4d <phase_3+0x143>
 8048c8e:	e8 22 05 00 00       	call   80491b5 <explode_bomb>
 8048c93:	b8 6d 00 00 00       	mov    $0x6d,%eax
 8048c98:	e9 b0 00 00 00       	jmp    8048d4d <phase_3+0x143>
 8048c9d:	b8 6b 00 00 00       	mov    $0x6b,%eax
 8048ca2:	81 7c 24 2c 64 02 00 	cmpl   $0x264,0x2c(%esp)
 8048ca9:	00 
 8048caa:	0f 84 9d 00 00 00    	je     8048d4d <phase_3+0x143>
 8048cb0:	e8 00 05 00 00       	call   80491b5 <explode_bomb>
 8048cb5:	b8 6b 00 00 00       	mov    $0x6b,%eax
 8048cba:	e9 8e 00 00 00       	jmp    8048d4d <phase_3+0x143>
 8048cbf:	b8 64 00 00 00       	mov    $0x64,%eax
 8048cc4:	81 7c 24 2c a7 02 00 	cmpl   $0x2a7,0x2c(%esp)
 8048ccb:	00 
 8048ccc:	74 7f                	je     8048d4d <phase_3+0x143>
 8048cce:	e8 e2 04 00 00       	call   80491b5 <explode_bomb>
 8048cd3:	b8 64 00 00 00       	mov    $0x64,%eax
 8048cd8:	eb 73                	jmp    8048d4d <phase_3+0x143>
 8048cda:	b8 67 00 00 00       	mov    $0x67,%eax
 8048cdf:	81 7c 24 2c 0a 03 00 	cmpl   $0x30a,0x2c(%esp)
 8048ce6:	00 
 8048ce7:	74 64                	je     8048d4d <phase_3+0x143>
 8048ce9:	e8 c7 04 00 00       	call   80491b5 <explode_bomb>
 8048cee:	b8 67 00 00 00       	mov    $0x67,%eax
 8048cf3:	eb 58                	jmp    8048d4d <phase_3+0x143>
 8048cf5:	b8 6b 00 00 00       	mov    $0x6b,%eax
 8048cfa:	81 7c 24 2c f6 02 00 	cmpl   $0x2f6,0x2c(%esp)
 8048d01:	00 
 8048d02:	74 49                	je     8048d4d <phase_3+0x143>
 8048d04:	e8 ac 04 00 00       	call   80491b5 <explode_bomb>
 8048d09:	b8 6b 00 00 00       	mov    $0x6b,%eax
 8048d0e:	eb 3d                	jmp    8048d4d <phase_3+0x143>
 8048d10:	b8 72 00 00 00       	mov    $0x72,%eax
 8048d15:	83 7c 24 2c 39       	cmpl   $0x39,0x2c(%esp)
 8048d1a:	74 31                	je     8048d4d <phase_3+0x143>
 8048d1c:	e8 94 04 00 00       	call   80491b5 <explode_bomb>
 8048d21:	b8 72 00 00 00       	mov    $0x72,%eax
 8048d26:	eb 25                	jmp    8048d4d <phase_3+0x143>
 8048d28:	b8 73 00 00 00       	mov    $0x73,%eax
 8048d2d:	81 7c 24 2c d4 01 00 	cmpl   $0x1d4,0x2c(%esp)
 8048d34:	00 
 8048d35:	74 16                	je     8048d4d <phase_3+0x143>
 8048d37:	e8 79 04 00 00       	call   80491b5 <explode_bomb>
 8048d3c:	b8 73 00 00 00       	mov    $0x73,%eax
 8048d41:	eb 0a                	jmp    8048d4d <phase_3+0x143>
 8048d43:	e8 6d 04 00 00       	call   80491b5 <explode_bomb>
 8048d48:	b8 66 00 00 00       	mov    $0x66,%eax
 //将eax的低8位和第二个参数比较，即第二个参数应为eax的低8位对应的字符，当参数为0时，字符应为u，其余类推
 8048d4d:	3a 44 24 27          	cmp    0x27(%esp),%al
 8048d51:	74 05                	je     8048d58 <phase_3+0x14e>
 8048d53:	e8 5d 04 00 00       	call   80491b5 <explode_bomb>
 8048d58:	83 c4 3c             	add    $0x3c,%esp
 8048d5b:	c3                   	ret  
```

经过上述分析，我选择0 u 471作为第三个输入

#### phase_4

```
08048dbd <phase_4>:
 8048dbd:	83 ec 2c             	sub    $0x2c,%esp
 8048dc0:	8d 44 24 1c          	lea    0x1c(%esp),%eax
 8048dc4:	89 44 24 0c          	mov    %eax,0xc(%esp)//将esp+1c的地址放在esp+c
 8048dc8:	8d 44 24 18          	lea    0x18(%esp),%eax
 8048dcc:	89 44 24 08          	mov    %eax,0x8(%esp)//将esp+18的地址放在esp+8处
 //将0x804a3cf放在esp+4处，使用gdb查看，发现是该地址处是字符串 %d %d
 8048dd0:	c7 44 24 04 cf a3 04 	movl   $0x804a3cf,0x4(%esp)
 8048dd7:	08 
 8048dd8:	8b 44 24 30          	mov    0x30(%esp),%eax
 8048ddc:	89 04 24             	mov    %eax,(%esp)//将输入的参数的地址放在esp处
 //将输入的数据按%d %d的格式读到esp+1c和esp+18处，如果没有读到两个，爆炸
 8048ddf:	e8 7c fa ff ff       	call   8048860 <__isoc99_sscanf@plt>
 8048de4:	83 f8 02             	cmp    $0x2,%eax
 8048de7:	75 07                	jne    8048df0 <phase_4+0x33>
 //如果第一个参数大于14，爆炸，且jbe为无符号数比较，故参数1的范围[0,14]
 8048de9:	83 7c 24 18 0e       	cmpl   $0xe,0x18(%esp)
 8048dee:	76 05                	jbe    8048df5 <phase_4+0x38>
 8048df0:	e8 c0 03 00 00       	call   80491b5 <explode_bomb>
 8048df5:	c7 44 24 08 0e 00 00 	movl   $0xe,0x8(%esp)//将14放入esp+8
 8048dfc:	00 
 8048dfd:	c7 44 24 04 00 00 00 	movl   $0x0,0x4(%esp)//将0放入esp+4
 8048e04:	00 
 8048e05:	8b 44 24 18          	mov    0x18(%esp),%eax
 8048e09:	89 04 24             	mov    %eax,(%esp)//将第一个参数放入esp
 8048e0c:	e8 4b ff ff ff       	call   8048d5c <func4>//调用func4函数
 8048e11:	83 f8 02             	cmp    $0x2,%eax//如果返回值不等于2||第2个参数不等于2，爆炸
 8048e14:	75 07                	jne    8048e1d <phase_4+0x60>
 8048e16:	83 7c 24 1c 02       	cmpl   $0x2,0x1c(%esp)
 8048e1b:	74 05                	je     8048e22 <phase_4+0x65>
 8048e1d:	e8 93 03 00 00       	call   80491b5 <explode_bomb>
 8048e22:	83 c4 2c             	add    $0x2c,%esp
 8048e25:	c3                   	ret 
```

通过分析phase_4的代码，可以知道输入应为两个数字，且第二个为2，并且调用了func4(var1,var2(0),var3(14))的函数，且函数返回值应为2，下面分析func4代码

```
08048d5c <func4>:
//将保存特征值的寄存器压栈
 8048d5c:	56                   	push   %esi
 8048d5d:	53                   	push   %ebx
 8048d5e:	83 ec 14             	sub    $0x14,%esp//开辟临时变量等的使用空间
 8048d61:	8b 54 24 20          	mov    0x20(%esp),%edx//第一个参数放入edx，即var1
 8048d65:	8b 44 24 24          	mov    0x24(%esp),%eax//将第二个参数放入eax，即var2
 8048d69:	8b 5c 24 28          	mov    0x28(%esp),%ebx//将第三个参数放入ebx，将var3
 8048d6d:	89 d9                	mov    %ebx,%ecx
 8048d6f:	29 c1                	sub    %eax,%ecx
 8048d71:	89 ce                	mov    %ecx,%esi//ecx = esi = var3-var2
 8048d73:	c1 ee 1f             	shr    $0x1f,%esi
 8048d76:	01 f1                	add    %esi,%ecx
 8048d78:	d1 f9                	sar    %ecx//经过验证，是一个除2操作，即ecx=ecx/2
 //ecx = var2+(var3-var2)/2,考虑到/2时会去尾，ecx == (var2+var3)/2
 8048d7a:	01 c1                	add    %eax,%ecx
 8048d7c:	39 d1                	cmp    %edx,%ecx//ecx和var1比较大小
 8048d7e:	7e 17                	jle    8048d97 <func4+0x3b>//ecx <= var1,跳转到8048d97
 //ecx > var1
 8048d80:	83 e9 01             	sub    $0x1,%ecx //ecx = ecx -1
 8048d83:	89 4c 24 08          	mov    %ecx,0x8(%esp)//var3 = (var2+var3)/2 - 1
 8048d87:	89 44 24 04          	mov    %eax,0x4(%esp)//var2 = var2
 8048d8b:	89 14 24             	mov    %edx,(%esp)//var1 = var1
  //调用func4(var1,var2,(var2+var3)/2 - 1)
 8048d8e:	e8 c9 ff ff ff       	call   8048d5c <func4>
 8048d93:	01 c0                	add    %eax,%eax//返回值翻倍
 8048d95:	eb 20                	jmp    8048db7 <func4+0x5b>//实际操作结束
 8048d97:	b8 00 00 00 00       	mov    $0x0,%eax // eax  = 0
 //ecx >= var1转移，因为首先经过了一层<=的判断，这个表达式只有在ecx=var1是才会转移到，且返回值应为0
 8048d9c:	39 d1                	cmp    %edx,%ecx
 8048d9e:	7d 17                	jge    8048db7 <func4+0x5b>
 //ecx < var1
 8048da0:	89 5c 24 08          	mov    %ebx,0x8(%esp)//var3 = var3
 8048da4:	83 c1 01             	add    $0x1,%ecx
 8048da7:	89 4c 24 04          	mov    %ecx,0x4(%esp)//var2 = (var2+var3)/2 + 1
 8048dab:	89 14 24             	mov    %edx,(%esp)//var1 = var1
 //func4(var1,(var2+var3)/2 + 1,var3)
 8048dae:	e8 a9 ff ff ff       	call   8048d5c <func4>
 //返回值翻倍+1
 8048db3:	8d 44 00 01          	lea    0x1(%eax,%eax,1),%eax
 8048db7:	83 c4 14             	add    $0x14,%esp
 8048dba:	5b                   	pop    %ebx
 8048dbb:	5e                   	pop    %esi
 8048dbc:	c3                   	ret    
```



func4转成c代码如下

```
int func4(var1,var2,var3){
	half = (var2 + var3)/2;
	if(var1 >= half){
		if(var1 == half)
			return 0;
		else
			return 2 * func4(var1,half+1,var3) + 1;		
	}
	else
		return 2 * func4(var1,var2,half-1);
}
```

可以看出，func4的本质其实是二分查找的一种变形，可以看出，最内层跳出时必定返回1，而最外层需要为2,且二分查找的时间复杂度为log2n(n=14)，上取整为4,即最多要四次调用

则可能有 0->1->2/0->0->1->2

 0->1->2从外向内推依次推出<7,>3,=5

0->0->1->2从外向内推依次推出, <7 , >3, <5 , =4

故答案为 4 2 或 5 2

#### phase_5

```
08048e26 <phase_5>:
 8048e26:	53                   	push   %ebx
 8048e27:	83 ec 18             	sub    $0x18,%esp
 8048e2a:	8b 5c 24 20          	mov    0x20(%esp),%ebx
 8048e2e:	89 1c 24             	mov    %ebx,(%esp)
 8048e31:	e8 55 02 00 00       	call   804908b <string_length>//获得输入字符串的长度
 8048e36:	83 f8 06             	cmp    $0x6,%eax//长度不等于6则爆炸
 8048e39:	74 05                	je     8048e40 <phase_5+0x1a>
 8048e3b:	e8 75 03 00 00       	call   80491b5 <explode_bomb>
 8048e40:	ba 00 00 00 00       	mov    $0x0,%edx//edx = 0
 8048e45:	b8 00 00 00 00       	mov    $0x0,%eax// eax = 0
 8048e4a:	0f b6 0c 03          	movzbl (%ebx,%eax,1),%ecx//将第eax个字符放入ecx
 8048e4e:	83 e1 0f             	and    $0xf,%ecx//与0xf做按位与，即保留后四位
 //将以0x804a280位基址，4ecx为偏移量位置的数据与原edx的数据相加后放入edx
 8048e51:	03 14 8d 80 a2 04 08 	add    0x804a280(,%ecx,4),%edx
 8048e58:	83 c0 01             	add    $0x1,%eax
 8048e5b:	83 f8 06             	cmp    $0x6,%eax
 8048e5e:	75 ea                	jne    8048e4a <phase_5+0x24>//循环6次
 8048e60:	83 fa 45             	cmp    $0x45,%edx//最后结果为0x45即69则结束，否则爆炸
 8048e63:	74 05                	je     8048e6a <phase_5+0x44>
 8048e65:	e8 4b 03 00 00       	call   80491b5 <explode_bomb>
 8048e6a:	83 c4 18             	add    $0x18,%esp
 8048e6d:	5b                   	pop    %ebx
 8048e6e:	c3                   	ret  
```

经过上述分析，可以知道只要让以0x804a280位基址，4ecx为偏移量的6个数据的和相加为69则可以破解，而ecx为一个输入字符的低四位，即只要让每一个输入字符的低四位满足左移2位后加上0x804a280对应的数字的和为69即可，0x804a280对应的数组如下

```
(gdb) x /d 0x804a280
0x804a280 <array.3144>:	2
(gdb) 
0x804a284 <array.3144+4>:	10
(gdb) 
0x804a288 <array.3144+8>:	6
(gdb) 
0x804a28c <array.3144+12>:	1
(gdb) 
0x804a290 <array.3144+16>:	12
(gdb) 
0x804a294 <array.3144+20>:	16
(gdb) 
0x804a298 <array.3144+24>:	9
(gdb) 
0x804a29c <array.3144+28>:	3
(gdb) 
0x804a2a0 <array.3144+32>:	4
(gdb) 
0x804a2a4 <array.3144+36>:	7
(gdb) 
0x804a2a8 <array.3144+40>:	14
(gdb) 
0x804a2ac <array.3144+44>:	5
(gdb) 
0x804a2b0 <array.3144+48>:	11
(gdb) 
0x804a2b4 <array.3144+52>:	8
(gdb) 
0x804a2b8 <array.3144+56>:	15
(gdb) 
0x804a2bc <array.3144+60>:	13
```

我选择5+6+13+14+15+16，它们对应的偏移量为44，8，60，40，56，20，除以4后对应的二进制为1011，0010，1111，1010，1110，0101，在ascii表的大写字母处选择低四位符号的字母，则可以是KROJNU

```
08048e6f <phase_6>:
 8048e6f:	56                   	push   %esi
 8048e70:	53                   	push   %ebx
 8048e71:	83 ec 44             	sub    $0x44,%esp
 8048e74:	8d 44 24 10          	lea    0x10(%esp),%eax
 8048e78:	89 44 24 04          	mov    %eax,0x4(%esp)//将esp+10的地址放到esp+4处
 8048e7c:	8b 44 24 50          	mov    0x50(%esp),%eax
 8048e80:	89 04 24             	mov    %eax,(%esp)//将读入数据的地址放入esp
 8048e83:	e8 54 03 00 00       	call   80491dc <read_six_numbers>//从esp+10开始读六个数
 8048e88:	be 00 00 00 00       	mov    $0x0,%esi
 8048e8d:	8b 44 b4 10          	mov    0x10(%esp,%esi,4),%eax//依次取出读入的数据
 8048e91:	83 e8 01             	sub    $0x1,%eax
 8048e94:	83 f8 05             	cmp    $0x5,%eax
 8048e97:	76 05                	jbe    8048e9e <phase_6+0x2f>//读入数据在[1,6],否则爆炸
 8048e99:	e8 17 03 00 00       	call   80491b5 <explode_bomb>
 8048e9e:	83 c6 01             	add    $0x1,%esi//计数+1
 8048ea1:	83 fe 06             	cmp    $0x6,%esi
 8048ea4:	75 07                	jne    8048ead <phase_6+0x3e>//esi<=5时跳转
 8048ea6:	bb 00 00 00 00       	mov    $0x0,%ebx //两层循环结束，令ebx = 0
 8048eab:	eb 38                	jmp    8048ee5 <phase_6+0x76>//跳到0x8048ee5处执行
 8048ead:	89 f3                	mov    %esi,%ebx
 8048eaf:	8b 44 9c 10          	mov    0x10(%esp,%ebx,4),%eax//读入下一个数
 8048eb3:	39 44 b4 0c          	cmp    %eax,0xc(%esp,%esi,4)//比较当前数和偏移量4ebx数
 8048eb7:	75 05                	jne    8048ebe <phase_6+0x4f>//相等则爆炸，不相等跳转
 8048eb9:	e8 f7 02 00 00       	call   80491b5 <explode_bomb>
 8048ebe:	83 c3 01             	add    $0x1,%ebx//ebx + 1，
 8048ec1:	83 fb 05             	cmp    $0x5,%ebx
 //ebx <=5跳转，开始循环，即当前数和后面所有数都不相等
 8048ec4:	7e e9                	jle    8048eaf <phase_6+0x40>
 //整个循环为确定输入的六个数各不相等，即可判定输入必为1 2 3 4 5 6，但顺序未知
 8048ec6:	eb c5                	jmp    8048e8d <phase_6+0x1e>
 //使用gdb查看0x804c13c处开始的数据，发现是一个链表，每一个节点分别存储 一个整数 一个标号 下一个节点地址
 8048ec8:	8b 52 08             	mov    0x8(%edx),%edx//edx+8即下一个节点地址放入edx
 8048ecb:	83 c0 01             	add    $0x1,%eax//eax = eax + 1
 8048ece:	39 c8                	cmp    %ecx,%eax//比较ecx和eax，直至相等，ecx为输入数据
 8048ed0:	75 f6                	jne    8048ec8 <phase_6+0x59>
 8048ed2:	eb 05                	jmp    8048ed9 <phase_6+0x6a>
 8048ed4:	ba 3c c1 04 08       	mov    $0x804c13c,%edx//将0x804c13c放入edx
 8048ed9:	89 54 b4 28          	mov    %edx,0x28(%esp,%esi,4)//将地址放到esp+4esi+0x28
 8048edd:	83 c3 01             	add    $0x1,%ebx//ebx+1
 8048ee0:	83 fb 06             	cmp    $0x6,%ebx
 8048ee3:	74 17                	je     8048efc <phase_6+0x8d>//等于6跳转，即循环了6次
 8048ee5:	89 de                	mov    %ebx,%esi//esi = ebx
 8048ee7:	8b 4c 9c 10          	mov    0x10(%esp,%ebx,4),%ecx//开始依次取esp+10处的数据
 8048eeb:	83 f9 01             	cmp    $0x1,%ecx//如果ecx为1，跳转
 8048eee:	7e e4                	jle    8048ed4 <phase_6+0x65>
 8048ef0:	b8 01 00 00 00       	mov    $0x1,%eax//eax = 1
 8048ef5:	ba 3c c1 04 08       	mov    $0x804c13c,%edx//将0x804c13c放入edx
 8048efa:	eb cc                	jmp    8048ec8 <phase_6+0x59>//跳转至0x8048ec8
 //上面一段的主要功能就是将链表的节点的首地址按照输入数据的顺序依次放在从esp+28起始的连续空间内
 //例如输入为 6 5 4 3 2 1，则6个节点的首地址则倒序放入
 8048efc:	8b 5c 24 28          	mov    0x28(%esp),%ebx//将esp+28的放在ebx
 8048f00:	8d 44 24 2c          	lea    0x2c(%esp),%eax//将esp+2c的地址放入eax
 8048f04:	8d 74 24 40          	lea    0x40(%esp),%esi//将esp+40的地址放入esi
 8048f08:	89 d9                	mov    %ebx,%ecx//ecx = ebx
 8048f0a:	8b 10                	mov    (%eax),%edx//将eax指示的地址处的内容放入edx
 //将edx放入ecx+8的地址处，因为ecx存的是链表某个节点的地址，ecx+8对应的即为下一个节点的地址处的位置
 //而edx也是某一节点的首地址，将edx放入，即开始重置链表节点的顺序
 8048f0c:	89 51 08             	mov    %edx,0x8(%ecx)
 8048f0f:	83 c0 04             	add    $0x4,%eax//指向下一个存放节点地址的单元
 8048f12:	39 f0                	cmp    %esi,%eax//检查是否遍历完六个节点的地址
 8048f14:	74 04                	je     8048f1a <phase_6+0xab>//是跳转
 8048f16:	89 d1                	mov    %edx,%ecx//否，令ecx=edx，开始循环
 8048f18:	eb f0                	jmp    8048f0a <phase_6+0x9b>
 //上面循环的功能是将链表的节点重新按照输入数据的顺序构成一个新的链表
 8048f1a:	c7 42 08 00 00 00 00 	movl   $0x0,0x8(%edx)//链表末节点的下一个节点指针置空
 8048f21:	be 05 00 00 00       	mov    $0x5,%esi//esi = 5
 //将ebx+8处的内容放入eax，即将ebx指向的节点的下一个节点的地址放入eax
 8048f26:	8b 43 08             	mov    0x8(%ebx),%eax
 8048f29:	8b 00                	mov    (%eax),%eax//将eax指向节点的第一项放到eax
 8048f2b:	39 03                	cmp    %eax,(%ebx)//比较eax和ebx指向节点处的第一项
 8048f2d:	7d 05                	jge    8048f34 <phase_6+0xc5>//大于等于就跳转，否则爆炸
 8048f2f:	e8 81 02 00 00       	call   80491b5 <explode_bomb>
 8048f34:	8b 5b 08             	mov    0x8(%ebx),%ebx//将下一节点地址放入ebx
 8048f37:	83 ee 01             	sub    $0x1,%esi
 8048f3a:	75 ea                	jne    8048f26 <phase_6+0xb7>//esi！= 0循环，共5次
 //上述循环要求重新排序的链表的第一项应按从大到小的顺序排列
 8048f3c:	83 c4 44             	add    $0x44,%esp
 8048f3f:	5b                   	pop    %ebx
 8048f40:	5e                   	pop    %esi
 8048f41:	c3                   	ret 
```

经过上述分析，可以知道，phase_6就是一个按照输入的1 2 3 4 5 6 的顺序重新排列了一下链表，并要求链表的第一项按从大到小顺序排列，而链表的各项数据如下

```
(gdb) x /3x 0x804c13c
0x804c13c <node1>:	0x00000259	0x00000001	0x0804c148
(gdb) 
0x804c148 <node2>:	0x00000339
(gdb) 
0x804c14c <node2+4>:	0x00000002
(gdb) 
0x804c150 <node2+8>:	0x0804c154
(gdb) 
0x804c154 <node3>:	0x00000284
(gdb) 
0x804c158 <node3+4>:	0x00000003
(gdb) 
0x804c15c <node3+8>:	0x0804c160
(gdb) 
0x804c160 <node4>:	0x0000018f
(gdb) 
0x804c164 <node4+4>:	0x00000004
(gdb) 
0x804c168 <node4+8>:	0x0804c16c
(gdb) 
0x804c16c <node5>:	0x000001ee
(gdb) 
0x804c170 <node5+4>:	0x00000005
(gdb) 
0x804c174 <node5+8>:	0x0804c178
(gdb) 
0x804c178 <node6>:	0x000000f1
(gdb) 
0x804c17c <node6+4>:	0x00000006
(gdb) 
0x804c180 <node6+8>:	0x00000000

```

最后可知顺序应为2 3 1 5 4 6

#### secret_phase

```
08048f42 <fun7>:
 8048f42:	53                   	push   %ebx
 8048f43:	83 ec 18             	sub    $0x18,%esp
 8048f46:	8b 54 24 20          	mov    0x20(%esp),%edx
 8048f4a:	8b 4c 24 24          	mov    0x24(%esp),%ecx
 8048f4e:	85 d2                	test   %edx,%edx
 8048f50:	74 37                	je     8048f89 <fun7+0x47>
 8048f52:	8b 1a                	mov    (%edx),%ebx
 8048f54:	39 cb                	cmp    %ecx,%ebx
 8048f56:	7e 13                	jle    8048f6b <fun7+0x29>
 8048f58:	89 4c 24 04          	mov    %ecx,0x4(%esp)
 8048f5c:	8b 42 04             	mov    0x4(%edx),%eax
 8048f5f:	89 04 24             	mov    %eax,(%esp)
 8048f62:	e8 db ff ff ff       	call   8048f42 <fun7>
 8048f67:	01 c0                	add    %eax,%eax
 8048f69:	eb 23                	jmp    8048f8e <fun7+0x4c>
 8048f6b:	b8 00 00 00 00       	mov    $0x0,%eax
 8048f70:	39 cb                	cmp    %ecx,%ebx
 8048f72:	74 1a                	je     8048f8e <fun7+0x4c>
 8048f74:	89 4c 24 04          	mov    %ecx,0x4(%esp)
 8048f78:	8b 42 08             	mov    0x8(%edx),%eax
 8048f7b:	89 04 24             	mov    %eax,(%esp)
 8048f7e:	e8 bf ff ff ff       	call   8048f42 <fun7>
 8048f83:	8d 44 00 01          	lea    0x1(%eax,%eax,1),%eax
 8048f87:	eb 05                	jmp    8048f8e <fun7+0x4c>
 8048f89:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
 8048f8e:	83 c4 18             	add    $0x18,%esp
 8048f91:	5b                   	pop    %ebx
 8048f92:	c3                   	ret    

08048f93 <secret_phase>:
 8048f93:	53                   	push   %ebx
 8048f94:	83 ec 18             	sub    $0x18,%esp
 8048f97:	e8 90 02 00 00       	call   804922c <read_line>
 8048f9c:	c7 44 24 08 0a 00 00 	movl   $0xa,0x8(%esp)
 8048fa3:	00 
 8048fa4:	c7 44 24 04 00 00 00 	movl   $0x0,0x4(%esp)
 8048fab:	00 
 8048fac:	89 04 24             	mov    %eax,(%esp)
 8048faf:	e8 1c f9 ff ff       	call   80488d0 <strtol@plt>
 8048fb4:	89 c3                	mov    %eax,%ebx
 8048fb6:	8d 40 ff             	lea    -0x1(%eax),%eax
 8048fb9:	3d e8 03 00 00       	cmp    $0x3e8,%eax
 8048fbe:	76 05                	jbe    8048fc5 <secret_phase+0x32>
 8048fc0:	e8 f0 01 00 00       	call   80491b5 <explode_bomb>
 8048fc5:	89 5c 24 04          	mov    %ebx,0x4(%esp)
 8048fc9:	c7 04 24 88 c0 04 08 	movl   $0x804c088,(%esp)
 8048fd0:	e8 6d ff ff ff       	call   8048f42 <fun7>
 8048fd5:	83 f8 05             	cmp    $0x5,%eax
 8048fd8:	74 05                	je     8048fdf <secret_phase+0x4c>
 8048fda:	e8 d6 01 00 00       	call   80491b5 <explode_bomb>
 8048fdf:	c7 04 24 1c a2 04 08 	movl   $0x804a21c,(%esp)
 8048fe6:	e8 05 f8 ff ff       	call   80487f0 <puts@plt>
 8048feb:	e8 36 03 00 00       	call   8049326 <phase_defused>
 8048ff0:	83 c4 18             	add    $0x18,%esp
 8048ff3:	5b                   	pop    %ebx
 8048ff4:	c3                   	ret    
 8048ff5:	66 90                	xchg   %ax,%ax
 8048ff7:	66 90                	xchg   %ax,%ax
 8048ff9:	66 90                	xchg   %ax,%ax
 8048ffb:	66 90                	xchg   %ax,%ax
 8048ffd:	66 90                	xchg   %ax,%ax
 8048fff:	90                   	nop

```

secret_phase的进入方法在phase_defused中可以知道，只要当前字符串的输入个数为6，即破解了前六个阶段，且第四阶段的输入后附加字符串DrEvil即可进入

secret_phase考察的仍然是递归，通过fun7函数的递归，最终要使返回结果为5 ，且fun7的主要功能是在一颗搜索二叉树中查找数据

而fun7可写成如下c函数

```
int fun7(node root,int input){
	int result;
	if(!root)
		return -1;
	if(root.value <= input)
		if(root.value == input)
			result = 0;
		else
			result = 2*fun7(root->right,input)+1;
	else
		result = 2 * fun7(root->left,input);
	return result;
}
```

因为最终返回值是5，则最里层的返回值必然为0，即最终找到了这个数，否则不可能返回5

而要从0到5，且最多进行四层搜索(因为存储的二叉树只有四层)，则只有0->1->2->5符合要求

即为n1->n21->n33->n46,所以最终结果为n46节点的值，为47