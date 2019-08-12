# GAS 汇编

## 汇编程序范例

```
.section .data
output:
        .ascii "The processor Vendor ID is 'xxxxxxxxxxxx'\n"

.section .text
.global _start
_start:
        movl $0, %eax
        cpuid

        movl $output, %edi
        movl %ebx, 28(%edi)
        movl %edx, 32(%edi)
        movl %ecx, 36(%edi)
        movl $4, %eax
        movl $1, %ebx
        movl $output, %ecx
        movl $42, %edx
        int $0x80
        movl $1, %eax
        movl $0, %ebx
        int $0x80

```
### 构建可执行程序

```
$ as -o cpuid.o cpuid.s 
$ ld -o cpuid cpuid.o
```

### 使用编译器进行汇编

_start 标签改成main标签

```
.section .text
.global main
main:
```


```
$ gcc -o cpuid cpuid.s
```

### 调试程序

使用gdb

```
$ as -gstabs -o cpuid.o cpuid.s
$ ld -o cpuid cpuid.o
```

```
[root@localhost test01]# gdb cpuid
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-114.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /root/study/test01/cpuid...done.
(gdb) 

```

设置断点并运行程序，break格式： break *label+offset

```
(gdb) break *_start
Breakpoint 1 at 0x4000b0: file cpuid.s, line 8.
(gdb) run
Starting program: /root/study/test01/cpuid 

Breakpoint 1, _start () at cpuid.s:8
8		movl $0, %eax
(gdb) 

```

next和step命令执行下一行源码，cont按正常方式继续运行

查看数据

|数据命令|描述|
|---|---|
|info registers|显示所有寄存器的值|
|print|显示特定寄存器或者来自程序变量的值|
|x|显示特定内存位置的内容|

print 命令

* print/d 显示十进制的值
* print/t 显示二进制的值
* print/x 显示十六进制的值

```
(gdb) print /d $rsp
$1 = 140737488348560
(gdb) print /x $rsp
$2 = 0x7fffffffe590
(gdb) print /t $rsp
```

x命令

格式： x/nyz

n是要显示的字段数, y是输出格式, 如下：

* c用于字符
* d用于十进制
* x用于十六进制

z是要显示的字段的长度

* b用于字节
* h用于16位 （半字）
* w用于32位字

```
(gdb) x/42cb &output
0x6000ea:	84 'T'	104 'h'	101 'e'	32 ' '	112 'p'	114 'r'	111 'o'	99 'c'
0x6000f2:	101 'e'	115 's'	115 's'	111 'o'	114 'r'	32 ' '	86 'V'	101 'e'
0x6000fa:	110 'n'	100 'd'	111 'o'	114 'r'	32 ' '	73 'I'	68 'D'	32 ' '
0x600102:	105 'i'	115 's'	32 ' '	39 '\''	120 'x'	120 'x'	120 'x'	120 'x'
0x60010a:	120 'x'	120 'x'	120 'x'	120 'x'	120 'x'	120 'x'	120 'x'	120 'x'
0x600112:	39 '\''	10 '\n'

```

这个命令以字符模式显示output变量(&符号表明它是一个内存位置)


### 在汇编语言中使用C库函数

使用printf

```
.section .data
output:
        .asciz "The processor Vendor ID is '%s'\n"
.section .bss
        .lcomm buffer, 12

.section .text
.global _start
_start:
        movl $0, %eax
        cpuid

        movl $buffer, %edi
        movl %ebx, (%edi)
        movl %edx, 4(%edi)
        movl %ecx, 8(%edi)
        pushq $buffer
        pushq $output
        call printf
        addq $16, %rsp
        pushq $0
        call exit

```

编译链接

```
$ as -o cpuid2.o cpuid2.s
$ ld -dynamic-linker /lib64/ld-linux-x86-64.so.2  -o cpuid2 -lc cpuid2.o

```


## 汇编语言程序设计基础

### 传送数据

#### 定义数据元素

1. 数据段

使用.data命令声明数据段。在这个段声明的任何数据元素都保留在内存中并且可以被汇编程序中的指令读取和写入。

.rodata 任何数据元素只能按照只读模式访问。

|命令|数据类型|
|---|---|
|.ascii|文本字符串|
|.asciz|以空字符结尾的文本字符串|
|.byte|字节值|
|.double|双精度浮点数|
|.float|单精度浮点数|
|.int|32位整型|
|.long|32位整型(和int相同)|
|.octa|16字节整数|
|.quad|8字节整型|
|.short|16位整型|
|.single|单精度浮点型（和float相同）|


```
output:
    .ascii "The processor Vendor ID is 'xxxxxxxxxxxx'\n"
```

```
pi:
    .float 3.14159
```

```
sizes:
.long 100, 150, 200, 250, 300
```

```
.section .data
msg:
    .ascii "This is a test message"
factors:
    .double 37.45, 45.33, 12.30
height:
    .int 54
length:
    .int 62, 35, 47
```


2. 定义静态符号

定义

```
.equ factor, 3
.equ LINUX_SYS_CALL, 0X80
```

引用静态数据元素, 标签名前必须使用$符号

```
movl $LINUX_SYS_CALL, %eax
```

3. bss段

在bss段中定义的数据元素和在数据段中定义的有些不同。无需声明特定的数据类型，只要声明为所需目的保留的原始内存部分即可。

|命令|描述|
|---|---|
|.comm|声明未初始化的数据的通用内存区域|
|.lcomm|声明未初始化的数据的本地通用内存区域|

格式：

```
.comm symbol, length
```

例子

```
.section .bss
.lcomm buffer, 10000
```

bss 中声明的数据不包含在任何可执行文件中


.fill 自动创建10000个数据元素。默认每个字段一个字节，并用0填充它。
```
.section .data
buffer:
    .fill 10000
```

#### 传送数据

1. MOV指令格式

```
movx source, destination
```

movx 其中x可以是下面字符

* q 用于64位长度值
* l 用于32位长度值
* w 用于16位长度值
* b 用于8位长度值


```
movq %rax, %rbx
movl %eax, %ebx
movw %ax, %bx
movb %al, %bl
```

2. 把立即数传送到寄存器和内存

```
movl $0, %eax
movl $0x80, %ebx
movl $100, height
```

3. 在寄存器之间传送数据

8个通用寄存器(EAX, EBX, ECX, EDX, EDI, ESI, EBP和ESP)是用于保存数据最常用的寄存器。这些寄存器内容可以传送给可用的任何其他类型的寄存器。

专用寄存器(控制、调试和段寄存器)的内容只能传送给通用寄存器，或者接收从通用寄存器传送的内容。

```
movl %eax, %ecx
movw %ax, %cx
```

应该在长度相同的寄存器之间传送数据

4. 在内存和寄存器之间传送数据

* 把数据值从内存传送到寄存器

```
movl value, %eax  ; 把位于value标签指定的内存位置的数据传送给EAX寄存器
```

* 把数据值从寄存器传送给内存
```
movl %ecx, value
```

* 使用变址的内存位置

```
values:
    .int 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60
```

内存位置有以下因素确定：

* 基址
* 添加到基址上的偏移地址
* 数据元素的长度
* 确定选择哪个数据元素的变址

表达式格式是：

base_address (offset_address, index, size)

获取的数据值位于

base_address + offset_address + index * size

offset_address 和 index的值必须是寄存器，size可以是数字值。

如：引用values数组中的值20
```
movl $2, %edi
movl values(, %edi, 4), %eax
```



* 使用寄存器间接寻址

当使用标签引用内存位置中包含的数据值时,可以通过在指令的标签前面加上$符号获得数据值的内存位置的地址。

把values标签引用的内存地址传送给EDI寄存器
```
movl $values, %edi
```

把EBX寄存器的值传递给EDI寄存器中包含的内存位置
```
movl %ebx, (%edi)
```

将EDX寄存器的值存放在EDI寄存器指向的位置之后4个字节的内存位置中
```
movl %edx, 4(%edi)
```

相反方向
```
movl %edx, -4(%edi)
```


#### 条件传送指令

1. CMOV 指令

```
cmovx source, destination
```









