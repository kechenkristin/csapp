## 2.1 数制与编码
https://github.com/kechenkristin/csapp/blob/main/note/week2/slides/21.pdf
- 转换的概念在数据表示中的反映

- 信息的二进制编码
	- 机器级数据
		- 数值数据: 无符号整数, 带符号整数, 浮点数(实数)
		- 非数值数据: 逻辑数(位串), 西文字符，汉字
	- 01
	- reasons using binary
	- 真值&机器数
		- 真值：真正的值
		- 机器数: 用0和1编码的计算机内部的01序列

- 数值数据的表示(important)
	- 进位计数制
		- decimal, binary, octal, hexadecimal
		- 相互转换
	- 定，浮点表示(小数点)
		- 定点整数, 定点小数
		- 浮点数
	- 定点数的编码(正负号)
		- 原码 补码 移码 反码	

- 进位计数制
	- 进制表示
		- decimal
		- binary
		- octal
		- hexadecimal
	- 相互转换
		- decimal <===> binary  
	8421
		- decimal <===> octal

## 2.2 定点数的编码表示
https://github.com/kechenkristin/csapp/blob/main/note/week2/slides/22.pdf
- 原码(Sign and Magnitude)
- 补码 
	- modular 运算
	- 2's complement  
https://inst.eecs.berkeley.edu/~cs61c/sp22/pdfs/lectures/lec01.pdf  

## 2.3 C语言中的整数
## 2.4 浮点数的编码表示
![avatar](https://github.com/kechenkristin/imagesGitHub/blob/main/notes/csapp/Cdata.png)

- C语言支持的基本数据类型
	- 整数类型
		- Unsigned integer
		- Signed integer
	- 实数类型
		- 单精度浮点
		- 双精度浮点
		- 拓展精度浮点