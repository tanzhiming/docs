### 基本数学功能


#### 移位指令

1. 移位乘法

sal 向左算术移位

shl 向左逻辑移位

```
sal destination     # 向左移动1位
sal %cl, destination
sal shifter, destination
```

移位造成的超出长度的任何位首先被存放到进位标志中，然后在下一次移位操作中丢弃。


2. 移位除法

shr 逻辑右移， 清空移位造成的空位。

sar 算术右移， 右移的空位用符号位填充。


3. 循环移位

ROL 向左循环移位
ROR 向右循环移位
RCL 向左循环移位，并且包含进位标志
RCR 向右循环移位，并且包含进位标志

#### 十进制运算

1. 不打包BCD运算

* AAA: 调整加法操作结果
* AAS: 调整减法操作结果
* AAM: 调整乘法操作结果
* AAD: 准备除法操作的被除数

2. 打包BCD的运算

* DAA: 调整ADD或者ADC指令的结果
* DAS: 调整SUB或者SBB指令的结果


#### 逻辑操作

1. 布尔逻辑

* AND
* NOT
* OR
* XOR

AND, OR 和XOR指令的格式

```
and source, destination
```

2. 位测试

TEST


### 高级数学功能

#### FPU环境





