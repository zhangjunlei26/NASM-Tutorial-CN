# NASM 教程

中文翻译版 NASM Tutorial

英文原文链接：http://cs.lmu.edu/~ray/notes/nasmtutorial/

> NASM 是一个绝赞的汇编器。现在让我们通过一些例子来学习 NASM。 然而这里的笔记仅仅只是蜻蜓点水般地涉及了一些皮毛,所以当你看完这个页面后,你需要查看 [官方的 NASM 文档 ](http://www.nasm.us/doc/)。

![img](https://cs.lmu.edu/~ray/images/nasm-logo.png)

## 教程范围

本教程将向您展示如何在 x86-64 体系结构上编写汇编语言程序。

您将同时编写(1)独立程序和(2)与 C 集成的程序。

我们不会太花哨了。

## 您的第一个程序

在学习 nasm 之前,请确保您可以键入并运行程序。

确保同时安装了 nasm 和 gcc。将以下程序之一另存为`hello.asm`,具体取决于您的计算机平台。然后根据给定的说明运行程序。

如果您使用的是基于 Linux 的操作系统：

`hello.asm`

```asm
; ----------------------------------------------------------------------------------------
; 仅使用syscall将"Hello,World"写入控制台。仅在64位Linux上运行。
; 编译汇编代码并运行：
;
; nasm -felf64 hello.asm && ld hello.o && ./a.out
; ----------------------------------------------------------------------------------------

          global    _start

          section   .text
_start:   mov       rax,1                  ; system call for write
          mov       rdi,1                  ; 文件句柄1是stdout
          mov       rsi,message            ; 要输出的字符串地址
          mov       rdx,13                 ; 字节数
          syscall                           ; 调用操作系统进行写入
          mov       rax,60                 ; syscall退出
          xor       rdi,rdi                ; 退出代码 0
          syscall                           ; 调用操作系统退出

          section   .data
message:  db        "Hello,World",10      ; 注意最后的换行符

```

```shell
$ nasm -felf64 hello.asm && ld hello.o && ./a.out
Hello,World
```



如果您使用的是 macOS：

`hello.asm`

```asm
; ----------------------------------------------------------------------------------------
; 仅使用syscall将" Hello,World"写入控制台。仅在64位macOS上运行。
; 编译汇编代码并运行：
;
; nasm -fmacho64 hello.asm && ld hello.o && ./a.out
; ----------------------------------------------------------------------------------------

          global _start

          section .text
start: mov rax,0x02000004   ; syscall
          mov rdi,1         ; 文件句柄1是stdout
          mov rsi,message   ;要输出的字符串地址
          mov rdx,13        ; 字节数
          syscall           ; 调用操作系统进行写入
          mov rax,0x02000001; syscall退出
          xor rdi,rdi       ; 退出代码0
          syscall           ; 调用操作系统退出

          section   .data
message:  db  "Hello, World",10 ;注意最后的换行符
```
```shell
$ nasm -fmacho64 hello.asm && ld hello.o && ./a.out
Hello, World
```
> **练习**：确定两个程序之间的差异。

## NASM 程序的结构

NASM 是基于行的。大多数程序由指令后跟一个或多个部分组成。行可以具有可选标签。大多数行都有一条指令,后跟零个或多个操作数。

![nasmstructure.png](https://cs.lmu.edu/~ray/images/nasmstructure.png)

通常,您将代码放在的部分中,`.text`并将常量数据放在的部分中`.data`。

## 细节

NASM 是一个很棒的汇编器,但是汇编语言很复杂。您不仅需要教程。您需要详细信息。很多细节。准备咨询：

- [NASM 手册](http://www.nasm.us/doc/),非常好！
- [英特尔处理器手册](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)

## 您的最初几个指示

有数百条指令。您无法一次全部学习它们。从这些 start:

| `mov` _x_,_y_ | _x_ ← _y_                                                                                                        |
| ------------- | ---------------------------------------------------------------------------------------------------------------- |
| `and` _x_,_y_ | _x_ ← *x* and *y*                                                                                                |
| `or` _x_,_y_  | _x_ ← *x* or *y*                                                                                                 |
| `xor` _x_,_y_ | _x_ ← _x_ xor _y_                                                                                                |
| `add` _x_,_y_ | _x_ ← _x_ + _y_                                                                                                  |
| `sub` _x_,_y_ | _x_ ← _x_ – _y_                                                                                                  |
| `inc` _x_     | _x_ ← _x_ + 1                                                                                                    |
| `dec` _x_     | _x_ ← _x_ – 1                                                                                                    |
| `syscall`     | 调用操作系统例程                                                                                                 |
| `db`          | 一个[伪指令](http://www.nasm.us/xdoc/2.11.02/html/nasmdoc3.html#section-3.2) 声明字节,这将是在内存中的程序运行时 |

## 三种操作数

### 寄存器操作数

在本教程中,我们只关心整数寄存器和 xmm 寄存器。您应该已经知道什么是寄存器,但是这里是一个快速的回顾。16 个整数寄存器为 64 位宽,称为：

```
R0 R1 R2 R3 R4 R5 R6 R7 R8 R9 R10 R11 R12 R13 R14 R15
RAX RCX RDX RBX RSP RBP RSI RDI
```

(请注意,其中的最后 8 个寄存器具有备用名称)您可以将每个寄存器的最低 32 位视为寄存器本身,但可以使用以下名称：

```
R0D R1D R2D R3D R4D R5D R6D R7D R8D R9D R10D R11D R12D R13D R14D R15D
EAX ECX EDX EBX ESP EBP ESI EDI
```

您可以使用以下名称将每个寄存器的最低 16 位视为一个寄存器本身：

```
R0W R1W R2W R3W R4W R5W R6W R7W R8W R9W R10W R11W R12W R13W R14W R15W
AX CX DX BX SP BP SI DI
```

您可以使用以下名称将每个寄存器的最低 8 位视为一个寄存器本身：

```
R0B R1B R2B R3B R4B R5B R6B R7B R8B R9B R10B R11B R12B R13B R14B R15B
AL CL DL BL SPL BPL SIL DIL
```

由于历史原因,`R0`..的第 15 至 8 位`R3`被命名为：

```
AH CH DH BH
```

最后,有 16 个 XMM 寄存器,每个 128 位宽,名为：

```
XMM0 ... XMM15
```

研究这张照片;希望它可以帮助：

![rdx.png](https://cs.lmu.edu/~ray/images/rdx.png)

### 内存操作数

这些是寻址的基本形式：

- `[ number ]`
- `[ reg ]`
- `[ reg + reg*scale ]` _小数位数只能是 1、2、4 或 8_
- `[ reg + number ]`
- `[ reg + reg*scale + number ]`

这个数字叫做**位移** ; 普通寄存器称为**基** ; 带有刻度的寄存器称为**索引**。

例子：

```asm
[750]               ; 仅位移
[rbp]               ; 仅基址寄存器
[rcx + rsi*4]       ; 基数+指数*比例
[rbp + rdx]         ; scale is 1
[rbx-8]             ; 位移-8
[rax + rdi*8 + 500] ; 所有四个组成部分
[rbx + counter]     ; 使用变量"counter"地址作为偏移
```

### 直接操作数

这些可以用多种方式编写。以下是官方文档中的一些示例。

```asm
200         ; 十进制数
0200        ; 仍然是十进制-前导0不会使其变为八进制
0200d       ; 显式十进制-d后缀
0d200       ; 也十进制-0d prefex
0c8h        ; 十六进制-h后缀,但是前导0是必需的,因为c8h看起来像var
0xc8        ; hex-经典的0x前缀
0hc8        ; 十六进制-由于某些原因,NASM偏爱0h写法
310q        ; 八进制-q后缀
0q310       ; 八进制-0q前缀
11001000b   ; 二进制-b后缀
0b1100_1000 ; 二进制-0b前缀,顺便说一下,允许使用下划线

```

## 具有两个内存操作数的指令非常少见

实际上,在本教程中我们将看不到任何此类说明。大多数基本说明只有以下几种形式：

| `add` _reg_, _reg_ |
| ------------------ |
| `add` _reg_,_mem_  |
| `add` _reg_, _imm_ |
| `add` _mem_,_reg_  |
| `add` _mem_, _imm_ |

## 定义数据并保留空间

这些示例来自 [docs 的第 3 章](https://cs.lmu.edu/~ray/notes/nasmtutorial/)。要将数据放入内存中：

```asm
      db 0x55               ; 只是字节0x55
      db 0x55,0x56,0x57     ; 连续三个字节
      db 'a',0x55           ; 字符常量可以
      db 'hello',13,10,'$'  ; 字符串常量也是如此
      dw 0x1234             ; 0x34 0x12
      dw 'a'                ; 0x61 0x00(只是一个数字)
      dw 'ab'               ; 0x61 0x62(字符常量)
      dw 'abc'              ; 0x61 0x62 0x63 0x00(字符串)
      dd 0x12345678         ; 0x78 0x56 0x34 0x12
      dd 1.234567e20        ; 浮点常数
      dq 0x123456789abcdef0 ; 八字节常量
      dq 1.234567e20        ; 双精度浮点
      dt 1.234567e20        ; 扩展精度浮点

```

还有其他形式。请稍候自行查阅 NASM 文档。

要保留空间(无需初始化),可以使用以下伪指令。它们应该放在一个称为`.bss`的小节中(如果您试图在一个`.text`小节中使用它们,将会出现错误)：

```asm
buffer:         resb    64              ; 保留64个字节
wordvar:        resw    1               ; 保留一个字
realarray:      resq    10              ; 十个实数的数组
```

## 另一个例子

这是一个要研究的 macOS 程序：

`triangle.asm`

```asm
; ----------------------------------------------------------------------------------------
; 这是一个OSX控制台程序,将星号的小三角形写成标准
; 输出。仅在macOS上运行。
;
; nasm -fmacho64 triangle.asm && gcc hola.o && ./a.out
; ----------------------------------------------------------------------------------------

          global _start
          section .text
start:
          mov rdx, output     ; rdx保存要写入的下一个字节的地址
          mov r8, 1           ; 初始线长
          mov r9, 0           ; 到目前为止在线上写的星数
line:
          mov byte [rdx], '*' ; 写一个星形符号
          inc rdx             ; 前进指向下一个要写入的单元格的指针
          inc r9              ; 到目前为止,"计数"数字已在线
          cmp r9,r8           ; 我们达到这条线的星数了吗？
          jne line            ; 还没有,继续写这行
lineDone:
          mov byte [rdx],10   ; 写一个新行char
          inc rdx             ; 并将指针移到下一个字符的位置
          inc r8              ; 下一行将是一个字符长
          mov r9,0            ; 重置写在此行上的星星数
          cmp r8,maxlines     ; 等等,我们已经完成了最后一行吗？
          jng line            ; 如果没有,开始写这行
done:
          mov rax,0x02000004  ; syscall
          mov rdi,1           ; 文件句柄1是stdout
          mov rsi,output      ; 要输出的字符串地址
          mov rdx,dataSize    ; 字节数
          syscall             ; 调用操作系统进行写入
          mov rax,0x02000001  ; syscall退出
          xor rdi,rdi         ; 退出代码0
          syscall             ; 调用操作系统退出

          section .bss
maxlines equ 8
dataSize equ 44
output: resb dataSize

```

```shell
$ nasm -fmacho64 triangle.asm && ld triangle.o && ./a.out
*
**
***
****
*****
******
*******
********
```



此示例中的新内容：

- `cmp` 做比较
- `je`如果先前的比较相等则跳转。
- `jne`(如果不等于则跳转)
- `jl`(如果不等于则跳转)
- `jnl`(如果不小于则跳转)
- `jg`(如果大于则跳转)
- `jng`(如果不大于则跳转)
- `jle`(如果小于或等于则跳转)
- `jnle`(如果不小于或等于则跳转)
- `jge`(如果大于或等于则跳转)
- `jnge`(如果不大于或等于则跳转)
- `equ`实际上不是真正的指令。它只是定义了供汇编程序本身使用的缩写。(这是一个意义深远的想法)
- 本`.bss`节适用于*可写*数据。

## 使用 C 库

仅使用 syscall 编写独立程序就已经很酷了，但很少见。我们想使用 C 库中的好东西。

还记得在 C 中如何从 `main`函数"starts"执行吗？这是因为 C 库实际上在其`_start`内部具有标签！`_start`开始的代码进行了一些初始化，然后调用`main`，执行完成进行清理，然后发出 syscall 以退出。因此，您只需要编码实现`main`即可，我们可以在汇编语言中实现这一点！

如果您有 Linux,请尝试以下操作：

hola.asm

```asm
; ----------------------------------------------------------------------------------------
; 使用C库将" Hola,mundo"写入控制台。在Linux上运行。
;
; nasm -felf64 hola.asm && gcc hola.o && ./a.out
; ----------------------------------------------------------------------------------------        
global    main
          extern    puts

          section   .text
main:                                       ; 这里由C library启动代码调用
          mov       rdi, message            ; rdi中的第一个整数(或指针)参数
          call      puts                    ; puts(message)
          ret                               ; Return from main back into C library wrapper
message:
          db        "Hola, mundo", 0        ; 注意字符串必须在C中以0结尾
```

```shell
$ nasm -felf64 hola.asm && gcc hola.o && ./a.out
Hola, mundo
```



在 macOS 下,看起来会有些不同：

hola.asm

```asm
; ----------------------------------------------------------------------------------------
; 这是一个macOS控制台程序,它在一行上写入" Hola,mundo",然后退出。
; 它使用C库中的puts。编译汇编代码并运行：
;
; nasm -fmacho64 hola.asm && gcc hola.o && ./a.out
; ----------------------------------------------------------------------------------------
          global    _main
          extern    _puts

          section   .text
_main:    push      rbx                     ; 调用堆栈必须对齐
          lea       rdi, [rel message]      ; 第一个参数是消息的地址
          call      _puts                   ; puts(message)
          pop       rbx                     ; Fix up stack before returning
          ret

          section   .data
message:  db        "Hola, mundo", 0        ; C字符串末尾需要一个零字节结尾

```

```shell
$ nasm -fmacho64 hola.asm && gcc hola.o && ./a.out
Hola, mundo
```



在 macOS 领域中,C 函数(或实际上是从一个模块导出到另一个模块的任何函数)必须以下划线作为前缀。调用堆栈必须在 16 字节边界上对齐(稍后会对此进行更多介绍)。并且在访问命名变量时,需要`rel`前缀。

## 理解参数调用习俗

我们怎么知道`puts`的参数放在`RDI`中？答：有多个参数调用约定。

为与 C 库集成的 64 位 Linux 编写代码时,必须遵循《[AMD64 ABI 参考》中](http://www.x86-64.org/documentation/abi.pdf)说明的调用约定 。您也可以从[Wikipedia](http://en.wikipedia.org/wiki/X86_calling_conventions#x86-64_Calling_Conventions)获得此信息 。最重要的一点是：

- 从左到右,传递尽可能多的参数以适合寄存器。分配寄存器的顺序为：
  - 整数和指针：`rdi`,`rsi`,`rdx`,`rcx`,`r8`,`r9`。
  - 浮点(单精度,双精度)：`xmm0`,`xmm1`,`xmm2`,`xmm3`,`xmm4`,`xmm5`, ` xmm6`,`xmm7`
- 其他参数从右到左压入堆栈,并*在调用*后*由调用方删除*。
- 推入参数后,将生成调用指令,因此当被调用函数获得控制时,返回地址为`[rsp]`,第一个内存参数为 ,依此类推`[rsp+8]`。
- **`rsp`进行调用之前,堆栈指针必须与 16 字节边界对齐**。很好,但是进行调用的过程会将返回地址(8 个字节)压入堆栈,因此当一个函数获得控制权时,`rsp`它就不会对齐。您必须通过推入一些东西或从中减去 8 来自己腾出额外的空间`rsp`。
- 唯一的供函数调用预留寄存器(the calle-save registers)有：`rbp`,`rbx`,`r12`,`r13`,`r14`,`r15`。其他所有函数均可通过调用函数自由更改。
- 被调用者还应该保存 XMCSR 和 x87 的控制位，但是 x87 指令在 64 位代码中很少见，因此您不必担心这一点。
- 整数被返回在`rax`或`rdx:rax`,浮点值返回在`xmm0`或`xmm1:xmm0`。

以上罗列的都理解了吗？什么，还没有？没关系，接下来我们再来一些示例，练习一下。

这是一个说明如何保存和还原寄存器的程序：

fib.asm

```asm
; ----------------------------------------------------------------------------
; 一个写入前90个斐波那契数字的64位Linux应用程序。至
; 编译汇编代码并运行：
;
; nasm -felf64 fib.asm && gcc fib.o && ./a.out
; ----------------------------------------------------------------------------

        global  main
        extern  printf

        section .text
main:
        push rbx                               ; 我们必须保存它,因为我们使用它

        mov ecx,90                             ; ecx将倒数至0
        xor rax,rax                            ; rax将保留当前数字
        xor rbx,rbx                            ; rbx将保留下一个数字
        inc rbx                                ; rbx最初是1
print:
        ; 我们需要调用printf,但是我们使用的是rax,rbx和rcx。打印
        ; 可能会破坏rax和rcx,因此我们将在调用之前保存它们,并且
        ; 之后恢复它们。

        push rax                               ; caller-save register
        push rcx                               ; caller-save register

        mov rdi,printf                         ; 设置第一个参数(format)
        mov rsi,rax                            ; 设置第二个参数(current_number)
        xor rax,rax                            ; 因为printf是varargs

                                               ; 堆栈已经对齐,因为我们压入了三个8字节寄存器
        call printf                            ; printf(format, current_number)

        pop rcx                                ; restore caller-save register
        pop rax                                ; restore caller-save register

        mov rdx,rax                            ; save the current number
        mov rax,rbx                            ; next number is now current
        add rbx,rdx                            ; get the new next number
        dec ecx                                ; count down
        jnz print                              ; if not done counting, do some more

        pop rbx                                ; returing之前还原rbx
        ret
format:
        db  "％20ld",10,0

```

```shell
$ nasm -felf64 fib.asm && gcc fib.o && ./a.out
                   0
                   1
                   1
                   2
                   .
                   .
                   .
  679891637638612258
 1100087778366101931
 1779979416004714189
```



我们刚刚看到了一些新的说明：

| `push` _x_     | 减少`rsp`操作数的大小,然后将*x*存储在`[rsp]` |
| -------------- | -------------------------------------------- |
| `pop` _x_      | 移动`[rsp]`到*X*,然后增加`rsp`由操作数的大小 |
| `jnz` _label_  | 如果设置了处理器的 Z(零)标志,请跳至给定标签  |
| `call` _label_ | 按下下一条指令的地址,然后跳到标签            |
| `ret`          | 弹出指令指针                                 |

## 混合 C 和汇编语言

该程序只是一个简单的函数,它接受三个整数参数并返回最大值。

`maxofthree.asm`

```asm
; -------------------------------------------------- ---------------------------
; 一个64位函数,该函数返回其三个64位整数的最大值
; 论点。该函数具有签名：
;
; int64_t maxofthree(int64_t x,int64_t y,int64_t z)
;
; 请注意,参数已经在rdi,rsi和rdx中传递。我们
; 只需返回rax中的值即可。
; -------------------------------------------------- ---------------------------

        global  maxofthree
        section .text
maxofthree：
        mov rax,rdi    ; 结果(rax)最初持有x
        cmp rax,rsi    ; x小于y吗？
        cmovl rax,rsi  ; 如果是这样,将结果设置为y
        cmp rax,rdx    ; max(x,y)小于z吗？
        cmovl rax,rdx  ; 如果是这样,将结果设置为z
        ret            ; 最大值将为rax
```

这是一个调用汇编语言功能的 C 程序。

`callmaxofthree.c`

```c
/*
 * 一个小程序,说明如何调用我们用汇编语言编写的maxofthree函数。
 *
 */

#include <stdio.h>
#include <inttypes.h>

int64_t maxofthree (int64_t,int64_t,int64_t);

int main(){
    printf("％ld \ n", maxofthree(1,-4,-7));
    printf("％ld \ n", maxofthree(2,-6,1));
    printf("％ld \ n", maxofthree(2,3,1));
    printf("％ld \ n", maxofthree(-2,4,3));
    printf("％ld \ n", maxofthree(2,-6,5));
    printf("％ld \ n", maxofthree(2,4,6));
    return 0;
}

```

```shell
$ nasm -felf64 maxofthree.asm && gcc callmaxofthree.c maxofthree.o && ./a.out
1
2
3
4
5
6
```



## 有条件的指令

在算术或逻辑指令或比较指令之后`cmp`,处理器会设置或清除其中的位`rflags`。最有趣的标志是：

- `s` (标志)
- `z` (零)
- `c` (携带)
- `o` (溢出)

因此,执行完一条加法指令后,我们可以根据新的标志设置执行跳转,移动或设置。例如：

| `jz` _标签_      | 如果运算结果为零,则跳至标签 L                                |
| ---------------- | ------------------------------------------------------------ |
| `cmovno` _x_,_y_ | _x ← y_ 如果最后的操作确实*不*溢出                           |
| `setc` _x_       | 如果最后一个操作带有进位,则*x* ← _1_,否则,_x_ ← *0,*否则(*x*必须是字节大小的寄存器或存储器位置) |

条件指令具有三种基本形式：`j`用于条件跳转,`cmov`用于条件移动和`set`用于条件设置。指令的后缀具有 30 种形式之一： `s ns z nz c nc o no p np pe po e ne l nl le nle g ng ge nge a na ae nae b nb be nbe`。

## 命令行参数

您知道在 C 中，`main`它只是一个普通的旧函数,它有几个自己的参数：

```c
int main(int argc, char ** argv)
```

因此,您猜到了，`argc`它将以`rdi`结尾，而 `argv`(指针)将以`rsi`结尾。以下是一个遵循此规则的程序，将命令行参数简单地回显到程序,每行一个：

`echo.asm`

```asm
; -------------------------------------------------- ---------------------------
; 一个显示其命令行参数(每行一个)的64位程序。
;
; 在输入时,rdi将包含argc,而rsi将包含argv。
; -------------------------------------------------- ---------------------------

        global  main
        extern  puts
        section .text

main:
        push rdi      ; 保存放置使用的寄存器
        push rsi
        sub rsp,8     ; 调用前必须对齐堆栈

        mov rdi,[rsi] ; 要显示的参数字符串
        call puts     ; 打印它

        add rsp,8     ; 恢复％rsp到预先对齐的值
        pop rsi       ; 恢复已使用的寄存器
        pop rdi

        add rsi,8     ; 指向下一个论点
        dec rdi       ; 倒数
        jnz main      ; 如果还没递减到0就继续

        ret

```

```shell
$ nasm -felf64 echo.asm && gcc echo.o && ./a.out dog 22 -zzz "hi there"
./a.out
dog
22
-zzz
hi there
```



## 一个更长的例子

请注意,就 C 库而言,命令行参数始终是字符串。如果要将它们视为整数,请致电`atoi`。这是一个计算 x y 的简洁程序。

权力

```asm
; -------------------------------------------------- ---------------------------
; 一个用于计算x ^ y的64位命令行应用程序。
;
; 语法：电源xy
; x和y是(32位)整数
; -------------------------------------------------- ---------------------------

        global  main
        extern  printf
        extern  puts
        extern  atoi

        section .text
main:
        push    r12                     ; save callee-save registers
        push    r13
        push    r14
        ; By pushing 3 registers our stack is already aligned for calls

        cmp     rdi, 3                  ; must have exactly two arguments
        jne     error1

        mov     r12, rsi                ; argv

; We will use ecx to count down form the exponent to zero, esi to hold the
; value of the base, and eax to hold the running product.

        mov     rdi, [r12+16]           ; argv[2]
        call    atoi                    ; y in eax
        cmp     eax, 0                  ; disallow negative exponents
        jl      error2
        mov     r13d, eax               ; y in r13d

        mov     rdi, [r12+8]            ; argv
        call    atoi                    ; x in eax
        mov     r14d, eax               ; x in r14d

        mov     eax, 1                  ; start with answer = 1
check:
        test    r13d, r13d              ; we're counting y downto 0
        jz      gotit                   ; done
        imul    eax, r14d               ; multiply in another x
        dec     r13d
        jmp     check
gotit:                                  ; print report on success
        mov     rdi, answer
        movsxd  rsi, eax
        xor     rax, rax
        call    printf
        jmp     done
error1:                                 ; print error message
        mov     edi, badArgumentCount
        call    puts
        jmp     done
error2:                                 ; print error message
        mov     edi, negativeExponent
        call    puts
done:                                   ; restore saved registers
        pop     r14
        pop     r13
        pop     r12
        ret

answer:
        db      "%d", 10, 0
badArgumentCount:
        db      "Requires exactly two arguments", 10, 0
negativeExponent:
        db      "The exponent may not be negative", 10, 0

```

```shell
$ nasm -felf64 power.asm && gcc -o power power.o
$ ./power 2 19
524288
$ ./power 3 -8
The exponent may not be negative
$ ./power 1 500
1
$ ./power 1
Requires exactly two arguments
```



## 浮点指令

浮点参数进入 xmm 寄存器。这是一个简单的函数,用于对双精度数组中的值求和：

总和

```asm
; -------------------------------------------------- ---------------------------
; 一个64位函数,该函数返回浮点中的元素之和
; 数组。该函数具有原型：
;
; double sum(double []数组,uint64_t长度)
; -------------------------------------------------- ---------------------------

        global  sum
        section .text
sum:
        xorpd   xmm0, xmm0              ; 将sum初始化为0
        cmp     rsi, 0                  ; special case for length = 0
        je      done
next:
        addsd   xmm0, [rdi]             ; 加当前数组元素
        add     rdi, 8                  ; 移至下一个数组元素
        dec     rsi                     ; 递减
        jnz     next                    ; if not done counting, continue
done:
        ret                             ; 返回值已经在xmm0中
```

请注意,浮点指令具有`sd`后缀;这是最常见的一种,但稍后我们会再看到其他一些。这是一个调用它的 C 程序：

`callum.c`

```c
/**
 * 说明如何调用用汇编语言编写的sum函数。
 */

#include <stdio.h>
#include <inttypes.h>

double sum(double[], uint64_t);

int main() {
    double test[] = {
        40.5, 26.7, 21.9, 1.5, -40.5, -23.4
    };
    printf("%20.7f\n", sum(test, 6));
    printf("%20.7f\n", sum(test, 2));
    printf("%20.7f\n", sum(test, 0));
    printf("%20.7f\n", sum(test, 3));
    return 0;
}
```

```shell
$ nasm -felf64 sum.asm && gcc sum.o callsum.c && ./a.out
          26.7000000
          67.2000000
           0.0000000
          89.1000000
```



## 数据段

在大多数操作系统上,文本部分是只读的,因此您可能会发现需要数据部分。在大多数操作系统上,data 部分仅用于初始化数据,而您有一个特殊的.bss 部分用于未初始化数据。这是一个对命令行参数取平均值的程序,该参数应为整数,并将结果显示为浮点数。

`average.asm`

```asm
; -------------------------------------------------- ---------------------------
; 64位程序,将其所有命令行参数都视为整数和
; 将其平均值显示为浮点数。该程序使用数据
; 存储中间结果的部分,不是必须要存储的,而仅仅是
; 说明如何使用数据段。
; -------------------------------------------------- ---------------------------

        global   main
        extern   atoi
        extern   printf
        default  rel

        section  .text
main:
        dec      rdi              ; argc-1,因为我们不计算程序名称
        jz       nothingToAverage
        mov      [count], rdi     ; 保存真实参数的数量
accumulate:
        push     rdi              ; save register across call to atoi
        push     rsi
        mov      rdi, [rsi+rdi*8] ; argv[rdi]
        call     atoi             ; now rax has the int value of arg
        pop      rsi              ; restore registers after atoi call
        pop      rdi
        add      [sum], rax       ; accumulate sum as we go
        dec      rdi              ; count down
        jnz      accumulate       ; more arguments?
average:
        cvtsi2sd xmm0, [sum]
        cvtsi2sd xmm1, [count]
        divsd    xmm0, xmm1       ; xmm0 is sum/count
        mov      rdi, format      ; 1st arg to printf
        mov      rax, 1           ; printf is varargs, there is 1 non-int argument

        sub      rsp, 8           ; align stack pointer
        call     printf           ; printf(format, sum/count)
        add      rsp, 8           ; restore stack pointer

        ret

nothingToAverage:
        mov      rdi, error
        xor      rax, rax
        call     printf
        ret

        section  .data
count:  dq       0
sum:    dq       0
format: db       "%g", 10, 0
error:  db       "There are no command line arguments to average", 10, 0

```

```shell
$ nasm -felf64 average.asm && gcc average.o && ./a.out 19 8 21 -33
3.75
$ nasm -felf64 average.asm && gcc average.o && ./a.out
There are no command line arguments to average
```



该程序着重介绍了一些在整数和浮点值之间转换的处理器指令。一些最常见的是：

| `cvtsi2sd` _xmmreg,r/m32_     | _xmmreg [63..0]_ ← _intToDouble(r / m32)_ |
| ----------------------------- | ----------------------------------------- |
| `cvtsi2ss` _xmmreg,r/m32_     | _xmmreg [31..0]_ ← _intToFloat(r / m32)_  |
| `cvtsd2si` _reg32_,_xmmr/m64_ | _reg32_ ← _doubleToInt(xmmr / m64)_       |
| `cvtss2si` _reg32_,_xmmr/m32_ | _reg32_ ← _floatToInt(xmmr / m32)_        |

## 递归

也许令人惊讶的是,实现递归功能并没有什么不同寻常的。您只需要像往常一样小心保存寄存器。在递归调用周围推动和弹出是一种典型的策略。

`factorial.asm`

```asm
; ----------------------------------------------------------------------------
; 递归函数的实现：
;
;   uint64_t factorial(uint64_t n) {
;       return (n <= 1) ? 1 : n * factorial(n-1);
;   }
; ----------------------------------------------------------------------------

        global  factorial

        section .text
factorial:
        cmp     rdi, 1                  ; n <= 1?
        jnbe    L1                      ; if not, go do a recursive call
        mov     rax, 1                  ; otherwise return 1
        ret
L1:
        push    rdi                     ; 将n保存在堆栈上 (也对齐 %rsp!)
        dec     rdi                     ; n-1
        call    factorial               ; factorial(n-1), 结果保存到 %rax
        pop     rdi                     ; 恢复 n
        imul    rax, rdi                ; n * factorial(n-1), 保存到 %rax
        ret
```

一个调用示例：

`callfactorial.c`

```c
/**
 *一个说明调用其他地方定义的阶乘函数的应用程序。
 */

#include <stdio.h>
#include <inttypes.h>

uint64_t factorial(uint64_t n);

int main() {
    for (uint64_t i = 0; i < 20; i++) {
        printf("factorial(%2lu) = %lu\n", i, factorial(i));
    }
    return 0;
}
```

```shell
$ nasm -felf64 factorial.asm && gcc -std=c99 factorial.o callfactorial.c && ./a.out
factorial( 0) = 1
factorial( 1) = 1
factorial( 2) = 2
factorial( 3) = 6
factorial( 4) = 24
factorial( 5) = 120
factorial( 6) = 720
factorial( 7) = 5040
factorial( 8) = 40320
factorial( 9) = 362880
factorial(10) = 3628800
factorial(11) = 39916800
factorial(12) = 479001600
factorial(13) = 6227020800
factorial(14) = 87178291200
factorial(15) = 1307674368000
factorial(16) = 20922789888000
factorial(17) = 355687428096000
factorial(18) = 6402373705728000
factorial(19) = 121645100408832000
```



## SIMD 并行

XMM 寄存器可以对浮点值进行一次算术运算(一次标量)或一次执行多次运算(打包)。操作具有以下形式：

```asm
op xmmreg_or_memory,xmmreg
```

对于浮点加法,说明如下：

| `addpd` | 并行执行 2 个双精度加法（添加压缩双精度）              |
| ------- | ------------------------------------------------------ |
| `addsd` | 使用寄存器的低 64 位仅执行一次双精度加法（将标量加倍） |
| `addps` | 并行执行 4 个单精度加法（添加打包的单个）              |
| `addss` | 使用寄存器的低 32 位仅执行一个单精度加法（加标量单）   |

这是一个可以一次添加四个浮点数的函数：

`add_four_floats.asm`

```asm
; void add_four_floats(float x[4], float y[4])
; x[i] += y[i] for i in range(0..4)

        global   add_four_floats
        section  .text

add_four_floats:
        movdqa   xmm0, [rdi]            ; x的所有四个值
        movdqa   xmm1, [rsi]            ; y的所有四个值
        addps    xmm0, xmm1             ; 一次完成所有四个总和
        movdqa   [rdi], xmm0
        ret
```

一个呼叫者：

`test_add_four_floats.c`

```c
#include <stdio.h>
void add_four_floats(float[], float[]);

int main() {
    float x[] = {-29.750, 244.333, 887.29, 48.1E22};
    float y[] = {29.750,  199.333, -8.29,  22.1E23};
    add_four_floats(x, y);
    printf("%f\n%f\n%f\n%f\n", x[0], x[1], x[2], x[3]);
    return 0;
}
```

还可以参考：[nice little x86 floating-point slide deck from Ray Seyfarth](http://rayseyfarth.com/asm/pdf/ch11-floating-point.pdf)

## 饱和运算

XMM 寄存器还可以对整数进行算术运算。这些说明具有以下形式：

```asm
op xmmreg_or_memory, xmmreg
```

对于整数加法,说明如下：

| `paddb`   | 做 16 个字节加法                           |
| --------- | ------------------------------------------ |
| `paddw`   | 做 8 个单词加法                            |
| `paddd`   | 做 4 个 dword 加法                         |
| `paddq`   | 做 2 个 qword 加法                         |
| `paddsb`  | 执行 16 个字节加法并带符号饱和度（80..7F） |
| `paddsw`  | 进行 8 个带符号饱和的单词加法（8000..7F）  |
| `paddusb` | 执行 16 个字节的无符号饱和（00..FF）       |
| `paddusw` | 进行 8 个无符号饱和（00..FFFF）的单词加法  |

这是一个例子。它还说明了如何加载 XMM 寄存器。您无法加载立即值;您必须使用它`movaps`来移出内存。还有其他方法,但是我们不会在本教程中介绍所有内容。

`satexample.asm`

```asm
; ----------------------------------------------------------------------------------------
; 有符号饱和运算示例。
; ----------------------------------------------------------------------------------------
        global  main
        extern  printf

        section .text
main:
        push    rbp
        movaps  xmm0, [arg1]
        movaps  xmm1, [arg2]
        paddsw  xmm0, xmm1
        movaps  [result], xmm0

        lea     rdi, [format]
        mov     esi, dword [result]
        mov     edx, dword [result+4]
        mov     ecx, dword [result+8]
        mov     r8d, dword [result+12]
        xor     rax, rax
        call    printf
        pop     rbp
        ret
        section .data
        align   16
arg1:   dw      0x3544,0x24FF,0x7654,0x9A77,0xF677,0x9000,0xFFFF,0x0000
arg2:   dw      0x7000,0x1000,0xC000,0x1000,0xB000,0xA000,0x1000,0x0000
result: dd      0, 0, 0, 0
format: db      '%x%x%x%x',10,0
```



## 图形

TODO

## 局部变量和堆栈框架

首先,请阅读[Eli Bendersky 的文章,](http://eli.thegreenplace.net/2011/09/06/stack-frame-layout-on-x86-64/) 该概述比我的简短笔记更完整。

调用函数时,调用者将首先将参数放入正确的寄存器中,然后发出`call`指令。调用之前,超出寄存器覆盖范围的其他参数将被压入堆栈。调用指令将返回地址放在堆栈的顶部。所以如果你有功能

```
int64_t example(int64_t x, int64_t y) {
    int64_t a, b, c;
    b = 7;
    return x * b + y;
}
```

然后在进入函数时,x 将在 edi 中,y 将在 esi 中,并且返回地址将在堆栈的顶部。我们可以在哪里放置局部变量？一个简单的选择是在堆栈本身上,但是如果您有足够的寄存器,请使用它们。

如果在运行符合标准 ABI 的计算机上运行,则可以将 rsp 保留在原来的位置,并直接从 rsp 访问"额外参数"和局部变量,例如：

```shell
                +----------+
         rsp-24 |    a     |
                +----------+
         rsp-16 |    b     |
                +----------+
         rsp-8  |    c     |
                +----------+
         rsp    | retaddr  |
                +----------+
         rsp+8  | caller's |
                | stack    |
                | frame    |
                | ...      |
                +----------+
```

因此,我们的函数如下所示：

```asm
        global  example
        section .text
example:
        mov     qword [rsp-16], 7
        mov     rax, rdi
        imul    rax, [rsp+8]
        add     rax, rsi
        ret
```

如果我们的函数要进行另一个调用,则您必须调整 rsp 才能摆脱干扰。

在 Windows 上,您不能使用此方案,因为如果发生中断,则堆栈指针上方的所有内容都会被粘贴。在大多数其他操作系统上不会发生这种情况,因为在堆栈指针后面有一个 128 字节的"红色区域",可以避免这些情况。在这种情况下,您可以立即在堆栈上腾出空间：

```gas
例：
        sub rsp, 24
```

因此我们的堆栈如下所示：

```
                +----------+
         rsp    |    a     |
                +----------+
         rsp+8  |    b     |
                +----------+
         rsp+16 |    c     |
                +----------+
         rsp+24 | retaddr  |
                +----------+
         rsp+32 | caller's |
                | stack    |
                | frame    |
                | ...      |
                +----------+

```

现在是这里的功能。请注意,我们必须记住在返回之前要替换堆栈指针！

```asm
        global  example
        section .text
example:
        sub     rsp, 24
        mov     qword [rsp+8], 7
        mov     rax, rdi
        imul    rax, [rsp+8]
        add     rax, rsi
        add     rsp, 24
        ret
```

## 在 macOS 上使用 NASM

希望您已经使用基于 Linux 的操作系统(或更正确地说,是 ELF64 系统)阅读了以上整个教程。要使这些示例在 64 位 macOS 系统上工作,几乎只有五件事要了解：

- 此目标文件格式`macho64`不是`elf64`。
- 系统调用值***完全不同***。
- 模块之间**共享符号**将**以下划线作为前缀**。
- 除非您进行一些设置调整,否则 macOS 中的 gcc 链接器似乎不允许绝对寻址。因此`default rel`,在引用标记的内存位置时添加,并且始终用`lea`获取地址。
- 另外,似乎有时在 Linux 下,不强制执行 16 位堆栈对齐要求,但似乎*始终*在 macOS 下强制执行。

因此,这是上面为 macOS 编写的普通程序。

`average.asm`

```asm
; -------------------------------------------------- ---------------------------
; 64位程序,将其所有命令行参数都视为整数和
; 将其平均值显示为浮点数。该程序使用数据
; 存储中间结果的部分,不是必须要存储的,而仅仅是
; 说明如何使用数据段。
;
; 为OS X设计。组装和运行：
;
; nasm -fmacho64 average.asm && gcc average.o && ./a.out
; -------------------------------------------------- ---------------------------

        global   _main
        extern   _atoi
        extern   _printf
        default  rel

        section  .text
_main:
        push     rbx              ; 我们从未使用，但是必须的代码，对齐堆栈，以便我们调用东西
        dec      rdi              ; argc-1, 因为我们不计算程序名称
        jz       nothingToAverage
        mov      [count], rdi     ; 保存实参数量
accumulate:
        push     rdi              ; save register across call to atoi
        push     rsi
        mov      rdi, [rsi+rdi*8] ; argv[rdi]
        call     _atoi            ; now rax has the int value of arg
        pop      rsi              ; restore registers after atoi call
        pop      rdi
        add      [sum], rax       ; accumulate sum as we go
        dec      rdi              ; count down
        jnz      accumulate       ; more arguments?
average:
        cvtsi2sd xmm0, [sum]
        cvtsi2sd xmm1, [count]
        divsd    xmm0, xmm1       ; xmm0 is sum/count
        lea      rdi, [format]    ; 1st arg to printf
        mov      rax, 1           ; printf is varargs, there is 1 non-int argument
        call     _printf          ; printf(format, sum/count)
        jmp      done

nothingToAverage:
        lea      rdi, [error]
        xor      rax, rax
        call     _printf

done:
        pop      rbx              ; undoes the stupid push at the beginning
        ret

        section  .data
count:  dq       0
sum:    dq       0
format: db       "%g", 10, 0
error:  db       "There are no command line arguments to average", 10, 0
```

```shell
$ nasm -fmacho64 average.asm && gcc average.o && ./a.out
There are no command line arguments to average
$ nasm -fmacho64 average.asm && gcc average.o && ./a.out 54.3
54
$ nasm -fmacho64 average.asm && gcc average.o && ./a.out 54.3 -4 -3 -25 455.1111
95.4
```



## 在 Windows 上使用 NASM

我不确定 Windows 上的 syscall 是什么,但是我确实知道,如果您要汇编和链接 C 库,则必须了解[x64 约定](https://msdn.microsoft.com/en-us/library/7kcdt6fy.aspx)。阅读它们。您将学到以下内容：

- 前四个整数参数在 RCX,RDX,R8 和 R9 中传递。其余的将被推入堆栈。
- 被调用者(callee)必须保留 RBX,RBP,RDI,RSI,RSP,R12,R13,R14 和 R15。
- 您猜对了，前四个浮点参数被通过 XMM0,XMM1,XMM2 和 XMM3。
- 返回值进入 RAX 或 XMM0。

**重要说明**：在任何文档中都很难找到一件事：x64 调用约定要求您在每次调用之前分配 32 字节的[影子空间](http://stackoverflow.com/a/30191127/831878),并在调用之后将其删除。这意味着您的"hello world"程序如下所示：

`hello.asm`

```asm
; -------------------------------------------------- --------------------------------------
; 这是一个Win64控制台程序,它在一行上写入" Hello",然后退出。它
; 使用C库中的puts。编译汇编代码并运行：
;
; nasm -fwin64 hello.asm && gcc hello.obj && a
; -------------------------------------------------- --------------------------------------

        global  main
        extern  puts
        section .text
main:
        sub     rsp, 28h                        ; 保留影子空间
        mov     rcx, message                    ; 第一个参数是message变量的地址
        call    puts                            ; puts(message)
        add     rsp, 28h                        ; 删除影子空间
        ret
message:
        db      'Hello', 0                      ; C strings need a zero byte at the end
       
```

您是否注意到我们实际上保留了 40 个字节？最小 32 字节的影子空间是必需的。在我们的`main`函数中,我们正在调用另一个函数,因此我们的堆栈[必须在 16 字节边界处对齐](https://docs.microsoft.com/en-us/cpp/build/stack-usage)。当`main`被调用时,返回地址(8 个字节)被压入,因此我们必须向影子空间"添加"额外的 8 个字节。


