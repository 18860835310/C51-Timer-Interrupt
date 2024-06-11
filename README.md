# C51-Timer-Interrupt
使用定时器中断来产生1ms的间隔，以避免阻塞程序的其他运行。

当需要的延时不干扰其他程序的正常继续运行时，也就是说我想实现C51这边控制动态数码管在那扫描显示数字，当每过去1m，就会使得数码管显示的数字-1。
我们就要用到计时器中断了。
程序功能：使用定时器中断来产生1ms的间隔，这样可以避免阻塞程序的其他运行。可以将计时器设置为每1ms触发一次中断，并在中断服务程序中更新数码管的显示。

# 全部程序

```c
#include "reg52.h"

typedef unsigned int u16;
typedef unsigned char u8;

// 定义数码管引脚和显示变量
sbit SEG_A = P2 ^ 0;
sbit SEG_B = P2 ^ 1;
sbit SEG_C = P2 ^ 2;
sbit SEG_D = P2 ^ 3;
sbit SEG_E = P2 ^ 4;
sbit SEG_F = P2 ^ 5;
sbit SEG_G = P2 ^ 6;
sbit SEG_DP = P2 ^ 7;

u8 display_number = 9;  // 初始显示数字

void display_digit(u8 num)
{
    // 假设数码管为共阴极，定义每个数字对应的段码
    const u8 digit_codes[10] = {
        0x3F, // 0
        0x06, // 1
        0x5B, // 2
        0x4F, // 3
        0x66, // 4
        0x6D, // 5
        0x7D, // 6
        0x07, // 7
        0x7F, // 8
        0x6F  // 9
    };

    u8 code = digit_codes[num];
    
    // 设置数码管段码
    SEG_A = code & 0x01;
    SEG_B = (code >> 1) & 0x01;
    SEG_C = (code >> 2) & 0x01;
    SEG_D = (code >> 3) & 0x01;
    SEG_E = (code >> 4) & 0x01;
    SEG_F = (code >> 5) & 0x01;
    SEG_G = (code >> 6) & 0x01;
    SEG_DP = (code >> 7) & 0x01;
}

void timer0_init(void)
{
    TMOD |= 0x01;  // 选择为定时器0模式，工作方式1
    TH0 = 0xFC;    // 给定时器初始值，定时1ms
    TL0 = 0x66;
    ET0 = 1;       // 打开定时器0中断允许
    EA = 1;        // 打开总中断
    TR0 = 1;       // 启动定时器
}

void timer0_isr(void) interrupt 1
{
    static u16 count = 0;
    TH0 = 0xFC;    // 给定时器初始值，定时1ms
    TL0 = 0x66;
    count++;
    if (count >= 1000)  // 每经过1000ms
    {
        count = 0;
        if (display_number > 0)
        {
            display_number--;  // 数字减1
        }
        else
        {
            display_number = 9;  // 重置为9
        }
    }
}

void main(void)
{
    timer0_init();  // 初始化定时器0
    while (1)
    {
        display_digit(display_number);  // 显示当前数字
        // 其他程序代码
    }
}
```

# 代码说明

## 1.数码管显示：
数码管的每一段（A到G和DP）连接到P2端口。
```c
// 定义数码管引脚和显示变量
sbit SEG_A = P2 ^ 0;
sbit SEG_B = P2 ^ 1;
sbit SEG_C = P2 ^ 2;
sbit SEG_D = P2 ^ 3;
sbit SEG_E = P2 ^ 4;
sbit SEG_F = P2 ^ 5;
sbit SEG_G = P2 ^ 6;
sbit SEG_DP = P2 ^ 7;
```
使用 display_digit 函数根据数字选择段码来控制数码管显示。
```c
void display_digit(u8 num)
{
    // 假设数码管为共阴极，定义每个数字对应的段码
    const u8 digit_codes[10] = {
        0x3F, // 0
        0x06, // 1
        0x5B, // 2
        0x4F, // 3
        0x66, // 4
        0x6D, // 5
        0x7D, // 6
        0x07, // 7
        0x7F, // 8
        0x6F  // 9
    };

    u8 code = digit_codes[num];
    
    // 设置数码管段码
    SEG_A = code & 0x01;
    SEG_B = (code >> 1) & 0x01;
    SEG_C = (code >> 2) & 0x01;
    SEG_D = (code >> 3) & 0x01;
    SEG_E = (code >> 4) & 0x01;
    SEG_F = (code >> 5) & 0x01;
    SEG_G = (code >> 6) & 0x01;
    SEG_DP = (code >> 7) & 0x01;
}
```

## 2.定时器初始化：
timer0_init 函数配置定时器0为模式1，并设置定时器初始值以实现1ms的中断。
```c
void timer0_init(void)
{
    TMOD |= 0x01;  // 选择为定时器0模式，工作方式1
    TH0 = 0xFC;    // 给定时器初始值，定时1ms
    TL0 = 0x66;
    ET0 = 1;       // 打开定时器0中断允许
    EA = 1;        // 打开总中断
    TR0 = 1;       // 启动定时器
}
```
启用定时器0中断和总中断。
```c
void main(void)
{
    timer0_init();  // 初始化定时器0
}
```

## 3.定时器中断服务程序：
timer0_isr 在每次定时器0中断时执行，维持一个计数器count，每当计数器达到1000时，表示已过1秒。
每秒将 display_number 减1，若减到0则重置为9。
```c
void timer0_isr(void) interrupt 1
{
    static u16 count = 0;
    TH0 = 0xFC;    // 给定时器初始值，定时1ms
    TL0 = 0x66;
    count++;
    if (count >= 1000)  // 每经过1000ms
    {
        count = 0;
        if (display_number > 0)
        {
            display_number--;  // 数字减1
        }
        else
        {
            display_number = 9;  // 重置为9
        }
    }
}
```

## 4.主函数：
初始化定时器0并进入主循环。
主循环不断调用 display_digit 函数显示当前数字，不会阻塞程序其他部分的运行。
```c
void main(void)
{
    timer0_init();  // 初始化定时器0
    while (1)
    {
        display_digit(display_number);  // 显示当前数字
        // 其他程序代码
    }
}
```

