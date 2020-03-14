# junior math

下载附件，使用ida pro打开,主函数反汇编如下

```
  __isoc99_scanf("%32s", &byte_602080);
  sub_4005D6();
  sub_40063F();
  sub_4006A8();
  sub_400711();
  sub_40077A();
  sub_4007E3();
  sub_40084C();
  sub_4008B5();
  sub_400918();
  sub_400981();
  sub_4009EA();
  sub_400A53();
  sub_400ABC();
  sub_400B25();
  if ( dword_602064 )
    puts("Wrong!");
  else
    puts("Right!");
  return 0LL;
```

可以看到flag被放在602080地址的位置，并且经过了14个函数的处理，最后需要令602064位置的值为0，查看任一处理函数

```
 v0 = (byte_602080 - 147) * byte_602080 != -4590 || (byte_602082 - 147) * byte_602082 != -4850;
  result = v0 | (unsigned int)dword_602064;
  dword_602064 |= v0;
  return result;
```

其余函数格式基本相同，就是||两端的表达式不同，不难知道602064一共 | 了14次，而要令结果为0，只有每一次的v0都为0，即所有的不等式均不成立，即所有等式成立，于是解方程可得flag