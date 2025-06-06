# CISC与RISC
## CISC 复杂指令集
## RISC 精简指令集



# RSIC特性与发展：RISC-V

# 硬件语言Verilog基础
## 硬件描述语言的作用
芯片设计时，根据不同模块的功能特点，通常把它们分为数字电路模块和模拟电路模块
- 模拟电路模块
  - 使用传统的电路设计方法
- 数字电路模块
  - 使用专门用于数字电路的硬件描述语言进行处理

## 学习硬件描述语言的思路
在Verilog代码里，基本所有写出来的东西都对应着实际电路
所以，我们用Verilog的时候，必须理解每条语句实际上对应着什么电路，并且要从电路的角度来思考他为何这样设计。（高级编程语言通常只要功能实现就行）

通过使用Verilog语言，我们就能完成芯片的数字电路设计工作了。
芯片前端设计工程师写Verilog代码的目的，就是把一份电路用代码的形式表示出来，然后由计算机把代码转换为所对应的逻辑电路

## 芯片设计思想
在开始一个大的芯片设计时，往往需要先从整个芯片系统做好规划，在写具体的Verilog代码之前，把系统划分为几个大的基本的功能模块。之后，每个功能模块在按一定的规则，划分出下一个层次的基本单元

## Verilog代码示例
```Verilog
module counter (
  // 端口定义
  input                 reset_n,   // 复位端，低有效
  input                 clk,       // 输入时钟
  output[3:0]           cnt,       // 计数输出
  output                cout       // 溢出位
);

reg [3:0]               cnt_r ; //计数器寄存器

always@(posedge clk or negedge reset_n) begin
  if(!reset_n) begin //复位时，计时归0
    cnt_r        <= 4'b0000 ;
    end
    else if (cnt_r==4'd9) begin //计时10个cycle时，计时归0
      cnt_r <=4'b0000;
    end
  else begin
    cnt_r <= cnt_r + 1'b1 ; //计时加1
  end
end
  assign cout = (cnt_r==4'd9) ; //输出周期位
  assign cnt = cnt_r ;          //输出实时计时器

endmodule
```

### 模块结构
```Verilog
module counter(
  // 接口，模块接口必须要指定它的输入信号还是输出信号
    input          reset_n,    // 输入信号用input声明
    input          clk,
    output[3:0]    cnt,        // 输出信号用output声明
    output         count
    // inout  ...             // 双向信号inout,既可以输入又可以输出的特殊端口信号
);
......     // 逻辑功能部分
endmodule
```

### 数据类型
```verilog
parameter     SIZE = 2'b01;
reg     [3:0]        cnt_r;   // 定义了位宽为4bit的寄存器reg类型信号，信号名称为cnt-r
wire [1:0]           cout_w;
```

#### reg类型
- 声明
```verilog
reg  [3:0]     cnt_r
```
- 赋值
  - reg类型只能在always和inital语句中被赋值，如果描述语句是时序逻辑，即always语句中带有时钟信号，寄存器变量对应为触发器电路。
  - 如果描述语句是组合逻辑，即always语句不带有时钟信号，寄存器变量对应为锁存器

#### wire类型
```shell
wire[1:0]     cout_w;
```
- Verilog代码用线网wire类型表示结构实体（各种逻辑门）之间的物理连线
- wire不能存储数值，它的值是由驱动它的元件决定的
  - 驱动线网类型变量的有
    - 逻辑门
    - 连续赋值语句
    - assign等
  - 没有驱动元件连接的线网上
    - 线网默认为高阻态"Z"

#### 符号常量
作用：提高代码的可读性和可维护性
```verilog
parameter      SIZE = 2'b01
```
- 通过parameter来声明一个标识符，用来代表一个常量参数，我们称之为符号常量
- 这个常量，实际上就是电路中一串有高低电平排列组合的一个固定数值

### 数值表达
```verilog
cnt_r  <=4'b0000;
// 这行代码的意思是：给寄存器cnt_r赋予4'b0000的值
```

- 其中的逻辑"0"低电平，对应电路接地(GND)
- 逻辑"1"高电平，对应电路接电源VCC
- 特殊值"X"，表示电平未知，输入端存在多种情况，可能是高电平，也可能是低电平
- 特殊值"Z"，逻辑"Z"表示高阻态，外部没有激励信号，是一个悬空状态

#### Verilog中同样数值的不同格式
```verilog
// 表示4位宽的数值"10"，位宽表示存储时二进制占用宽度
二进制写法：4'b1010
十进制写法：4'd10
十六进制写法：4'ha
```


### 运算符
大部分运算符和高级编程语言一致
部分位运算和高级编程语言不一致
```verilog
~   按位取反
&   按位与
|   按位或
^   异或
<<  左移(生成实际电路，左移增加位宽)
>>  右移(右移位宽不变)
```

### 条件，分支，循环语句
#### if和case
用if设计的语句，所对应的电路是有优先级的，也就是多级串联的MUX电路
用case语句对应的电路是没有优先级的，是一个多输入的MUX电路

#### 循环语句
while, for, repeat和forever循环
注意，循环语句只能在always块或initial块中使用

### 过程结构
#### always语句
从0时刻开始执行其中的行为语句；每当满足设定的always块触发条件时，再次执行语句块中的语句，如此循环反复

芯片设计师通常把always块的触发条件，设置为时钟信号的上升或下降沿。这样每收到一个时钟信号，always块内的逻辑电路都会执行一次

#### initial语句
从0时刻开始执行，且内部逻辑语句只按顺序执行一次，多个inital块执行相互独立
理论上inital语句不可以综合成实际电路，多用于初始化，信号检测等，也就是在编写验证代码时使用