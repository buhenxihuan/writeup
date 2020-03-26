

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
 8048d83:	89 4c 24 08          	mov    %ecx,0x8(%esp)//var3 = (var2+var3)/2向下取整 - 1
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

则可能有 0->1->2/0->0->0->1->2

 0->1->2从外向内推依次推出<7,>3,=5

0->0->1->2从外向内推依次推出, <7 , >3, <5 , =4

故答案为 4 2 或 5 2