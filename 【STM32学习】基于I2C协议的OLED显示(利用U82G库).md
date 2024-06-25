

#  【STM32学习】基于I2C协议的OLED显示(利用U82G库)

# 一，[I2C协议](https://so.csdn.net/so/search?q=I2C协议&spm=1001.2101.3001.7020)、时序协议

## 1，I2C协议：

I2C总线是Philips公司在八十年代初推出的一种串行、半双工的总线，主要用于近距离、低速的芯片之间的通信；I2C总线有两根双向的信号线，一根数据线SDA用于收发数据，一根时钟线SCL用于通信双方时钟的同步；I2C总线硬件结构简单，简化了PCB布线，降低了系统成本，提高了系统可靠性，因此在各个领域得到了广泛应用。

I2C总线是一种多主机总线，连接在 I2C总线上的器件分为主机和从机。主机有权发起和结束一次通信，从机只能被动呼叫；当总线上有多个主机同时启用总线时，I2C也具备冲突检测和仲裁的功能来防止错误产生；每个连接到I2C总线上的器件都有一个唯一的地址（7bit），且每个器件都可以作为主机也可以作为从机（但同一时刻只能有一个主机），总线上的器件增加和删除不影响其他器件正常工作；I2C总线在通信时总线上发送数据的器件为发送器，接收数据的器件为接收器。

I2C总线可以通过外部连线进行在线检测，便于系统故障诊断和调试，故障可以立即被寻址，软件也有利于标准化和模块化，缩短开发时间。

I2C总线上可挂接的设备数量受总线的最大电容400pF限制。

串行的8位双向数据传输速率在标准模式下可达100Kbit/s，快速模式下可达400Kbit/s，高速模式下可达3.4Mbit/s。

总线具有极低的电流消耗，抗噪声干扰能力强，增加总线驱动器可以使总线电容扩大10倍，传输距离达到15m；兼容不同电压等级的器件，工作温度范围宽。

![img](https://img-blog.csdnimg.cn/direct/e15a59e3ca3e4c32aa4201e5b4b303eb.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑



![img](https://img-blog.csdnimg.cn/direct/05fbb4f1a49941c48cbe2aa5adde7d5b.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

起始信号(S)：当 SCL 线是高电平时，SDA 线从高电平向低电平切换；停止信号(P)：当 SCL 是高电平时，SDA 线由低电平向高电平切换。

![img](https://img-blog.csdnimg.cn/direct/48ba349634a242019529156005e852b5.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑



帧地址：I2C总线上的每个设备都有自己的独立地址，主机发起通讯时，通过SDA信号线发送设备地址(SLAVE_ADDRESS)来查找从机。I2C协议规定设备地址可以是7位或10位，实际中7位的地址应用比较广泛。

I2C使用SDA信号线来传输数据，使用SCL信号线进行数据同步。SDA数据线在SCL的每个时钟周期传输一位数据。传输时，SCL为高电平的时候SDA表示的数据有效，即此时的SDA为高电平时表示数据“1”，为低电平时表示数据“0”。当SCL为低电平时，SDA的数据无效，一般在这个时候SDA进行电平切换，为下一次表示数据做好准备。

I2C的数据和地址传输都带响应。响应包括“应答(ACK)”和“非应答(NACK)”两种信号。

作为数据接收端时，当设备(无论主从机)接收到I2C传输的一个字节数据或地址后，若希望对方继续发送数据，则需要向对方发送“应答(ACK)”信号，发送方会继续发送下一个数据；若接收端希望结束数据传输，则向对方发送“非应答(NACK)”信号，发送方接收到该信号后会产生一个停止信号，结束信号传输。

![img](https://img-blog.csdnimg.cn/direct/cd6a3880ffcd40e7b86b4466cddea5c6.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

## 2，时序协议：

时序协议指的是确保I2C通信中时序关系正确的规则和标准。它包括：

起始条件（Start Condition）： 主设备发出的信号，标志着通信的开始，即SDA从高电平切换到低电平时，SCL处于高电平状态。

停止条件（Stop Condition）： 主设备发出的信号，表示通信的结束，即SDA从低电平切换到高电平时，SCL处于高电平状态。

数据传输的时钟频率： 由主设备控制，通常能达到100 kHz、400 kHz甚至更高的速率，取决于设备的支持和电路的设计。

数据位的传输顺序： 每个数据字节的传输顺序由主设备决定，可以是最高位先传输或最低位先传输。

应答位（ACK）： 每个数据字节传输结束后，接收设备发送一个应答位，表示是否成功接收数据



# 二、[OLED](https://so.csdn.net/so/search?q=OLED&spm=1001.2101.3001.7020)原理

## 1，OLED的定义和优势


 OLED，即有机发光二极管（Organic Light-Emitting Diode），又称为有机电激光显示（Organic Electroluminesence Display， OELD）。OLED由于同时具备自发光，不需背光源、对比度高、厚度薄、视角广、反应速度快、可用于挠曲性面板、使用温度范围广、构造及制程较简单等优异之特性，被认为是下一代的平面显示器新兴应用技术。

OLED显示技术具有自发光的特性，采用非常薄的有机材料涂层和玻璃基板，当有电流通过时，这些有机材料就会发光，而且OLED显示屏幕可视角度大，并且能够节省电能，从2003年开始这种显示设备在MP3播放器上得到了应用。

LCD都需要背光，而OLED不需要，因为它是自发光的。这样同样的显示，OLED效果要来得好一些。以目前的技术，OLED的尺寸还难以大型化，但是分辨率确可以做到很高。

![img](https://img-blog.csdnimg.cn/direct/020623a37d04490789be5f6f571d6cc0.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑



## **2，OLED发光原理：**

![img](https://img-blog.csdnimg.cn/direct/9f14bb60911d4555a788c8587a51dbba.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

（1）电子和空穴的注入。处于阴极中的电子和阳极中的空穴在外加驱动电压的驱动下会向器件的发光层移动，在向器件发光层移动的过程中，若器件包含有电子注入层和空穴注入层，则电子和空穴首先需要克服阴极与电子注入层及阳极与空穴注入层之间的能级势垒，然后经由电子注入层和空穴注入层向器件的电子传输层和空穴传输层移动；电子注入层和空穴注入层可增大器件的效率和寿命。关于OLED器件电子注入的机制还在不断的研究当中，目前最常被使用的机制是穿隧效应和界面偶极机制。

（2）电子和空穴的传输。在外加驱动电压的驱动下，来自阴极的电子和阳极的空穴会分别移动到器件的电子传输层和空穴传输层，电子传输层和空穴传输层会分别将电子和空穴移动到器件发光层的界面处；与此同时，电子传输层和空穴传输层分别会将来自阳极的空穴和来自阴极的电子阻挡在器件发光层的界面处，使得器件发光层界面处的电子和空穴得以累积。 [5]

（3）电子和空穴的再结合。当器件发光层界面处的电子和空穴达到一定数目时，电子和空穴会进行再结合并在发光层产生激子。 

（4）激子的退激发光。在发光层处产生的激子会使得器件发光层中的有机分子被活化，进而使得有机分子最外层的电子从基态跃迁到激发态，由于处于激发态的电子极其不稳定，其会向基态跃迁，在跃迁的过程中会有能量以光的形式被释放出来，进而实现了器件的发光。



# 三、U8G2

U8G2 是一个用于嵌入式设备的强大的图形库，特别是针对单色和彩色 OLED 显示屏。它提供了广泛的功能和灵活性，使开发者能够轻松地控制和管理这些显示屏。

![img](https://img-blog.csdnimg.cn/direct/25b58e79a89e429a916c9076369b9efa.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

主要特点和功能：

支持的显示屏类型：

OLED 显示屏： U8G2 支持各种单色和彩色 OLED 显示屏，包括不同的分辨率和尺寸。
 LCD 显示屏： 一些版本的 U8G2 也支持某些类型的 LCD 显示屏。
 驱动控制：

硬件支持： U8G2 可以通过不同的硬件接口（如 I2C、SPI）连接到微控制器。
 软件实现： 它提供了软件模拟的驱动器，使得即使在没有专用硬件接口的情况下也可以进行模拟开发和测试。
 字体和图形支持：

内置字体： U8G2 包含了多种字体，可以满足不同大小和风格的需求。
 自定义字体： 开发者可以添加和使用自定义的字体，以适应特定的应用需求。
 图形绘制： 支持直线、矩形、圆形、位图等基本图形绘制功能，方便用户进行界面设计和元素展示。
 操作简便性：

API 设计： U8G2 的 API 设计简单直观，易于使用和理解。
 功能丰富： 提供了丰富的功能集合，如滚动显示、页式显示等，以便有效地管理显示内容。
 社区支持和文档：

开源项目： U8G2 是一个开源项目，其代码和文档在 GitHub 上公开维护。
 文档和示例： 提供了详细的文档和示例代码，帮助开发者快速上手和解决问题。



# 四、代码移植

## 1，创建工程

![img](https://img-blog.csdnimg.cn/direct/ede153b431e348d1baf416b953c6fce1.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

CubexMX配置：

![img](https://img-blog.csdnimg.cn/direct/ba5b486e43454c3b8e8d478ac1b2a9cf.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

![img](https://img-blog.csdnimg.cn/direct/0ced2d26bdc84ca78672dbe60ab83d6e.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑



![img](https://img-blog.csdnimg.cn/direct/dd936388173247648ea2cb051a710ad4.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

**时钟树配置：**

![img](https://img-blog.csdnimg.cn/direct/1035247052db4cfd90c0be488a6fb9bb.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

**工程配置**

![img](https://img-blog.csdnimg.cn/direct/f0932d51d7554a6fb7b76ce4d86839c0.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

![img](https://img-blog.csdnimg.cn/direct/c62a934439854566b347bbba720724e1.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

## 2,U8G2代码移植：

![img](https://img-blog.csdnimg.cn/direct/f71b2eded72e4cfea9b019a400861a1d.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

![img](https://img-blog.csdnimg.cn/direct/796103cdfe374ef9bcc582e45716c3cd.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

![img](https://img-blog.csdnimg.cn/direct/7476d92f295f4026bacdee4015fc7d2a.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

![img](https://img-blog.csdnimg.cn/direct/049b35207c7a4ce5add6f20a67034006.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

![img](https://img-blog.csdnimg.cn/direct/51aa5666db0b41f6b293d696a7b9aab8.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑



![img](https://img-blog.csdnimg.cn/direct/097b669e2fa7483a9a73fba9fe5cdbee.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

## 3,代码修改：

**stm32_u8g2.h：**

```
#ifndef __STM32_U8G2_H
#define __STM32_U8G2_H
 
/* Includes ------------------------------------------------------------------*/
#include "main.h"
#include "u8g2.h"
/* USER CODE BEGIN Includes */
 
/* USER CODE END Includes */
 
 
 
/* USER CODE BEGIN Private defines */
 
/* USER CODE END Private defines */
#define u8         unsigned char  // ?unsigned char ????
#define MAX_LEN    128  //
#define OLED_ADDRESS  0x78 // oled
#define OLED_CMD   0x00  // 
#define OLED_DATA  0x40  // 
 
/* USER CODE BEGIN Prototypes */
uint8_t u8x8_byte_hw_i2c(u8x8_t *u8x8, uint8_t msg, uint8_t arg_int, void *arg_ptr);
uint8_t u8x8_gpio_and_delay(u8x8_t *u8x8, uint8_t msg, uint8_t arg_int, void *arg_ptr);
void u8g2Init(u8g2_t *u8g2);
void draw(u8g2_t *u8g2);
void testDrawPixelToFillScreen(u8g2_t *u8g2);
 
#endif
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**stm32_u8g2.c：**

```
#include "stm32_u8g2.h"
#include "tim.h"
#include "i2c.h"
 
 
uint8_t u8x8_byte_hw_i2c(u8x8_t *u8x8, uint8_t msg, uint8_t arg_int, void *arg_ptr)
{
    /* u8g2/u8x8 will never send more than 32 bytes between START_TRANSFER and END_TRANSFER */
    static uint8_t buffer[128];
    static uint8_t buf_idx;
    uint8_t *data;
 
    switch (msg)
    {
    case U8X8_MSG_BYTE_INIT:
    {
        /* add your custom code to init i2c subsystem */
        MX_I2C2_Init(); //I2C初始化
    }
    break;
 
    case U8X8_MSG_BYTE_START_TRANSFER:
    {
        buf_idx = 0;
    }
    break;
 
    case U8X8_MSG_BYTE_SEND:
    {
        data = (uint8_t *)arg_ptr;
 
        while (arg_int > 0)
        {
            buffer[buf_idx++] = *data;
            data++;
            arg_int--;
        }
    }
    break;
 
    case U8X8_MSG_BYTE_END_TRANSFER:
    {
        if (HAL_I2C_Master_Transmit(&hi2c2, OLED_ADDRESS, buffer, buf_idx, 1000) != HAL_OK)
            return 0;
    }
    break;
 
    case U8X8_MSG_BYTE_SET_DC:
        break;
 
    default:
        return 0;
    }
 
    return 1;
}
 
 
 
uint8_t u8x8_gpio_and_delay(u8x8_t *u8x8, uint8_t msg, uint8_t arg_int, void *arg_ptr)
{
    switch (msg)
    {
    case U8X8_MSG_DELAY_100NANO: // delay arg_int * 100 nano seconds
        __NOP();
        break;
    case U8X8_MSG_DELAY_10MICRO: // delay arg_int * 10 micro seconds
        for (uint16_t n = 0; n < 320; n++)
        {
            __NOP();
        }
        break;
    case U8X8_MSG_DELAY_MILLI: // delay arg_int * 1 milli second
        HAL_Delay(1);
        break;
    case U8X8_MSG_DELAY_I2C: // arg_int is the I2C speed in 100KHz, e.g. 4 = 400 KHz
        Tims_delay_us(5);
        break;                    // arg_int=1: delay by 5us, arg_int = 4: delay by 1.25us
    case U8X8_MSG_GPIO_I2C_CLOCK: // arg_int=0: Output low at I2C clock pin
        break;                    // arg_int=1: Input dir with pullup high for I2C clock pin
    case U8X8_MSG_GPIO_I2C_DATA:  // arg_int=0: Output low at I2C data pin
        break;                    // arg_int=1: Input dir with pullup high for I2C data pin
    case U8X8_MSG_GPIO_MENU_SELECT:
        u8x8_SetGPIOResult(u8x8, /* get menu select pin state */ 0);
        break;
    case U8X8_MSG_GPIO_MENU_NEXT:
        u8x8_SetGPIOResult(u8x8, /* get menu next pin state */ 0);
        break;
    case U8X8_MSG_GPIO_MENU_PREV:
        u8x8_SetGPIOResult(u8x8, /* get menu prev pin state */ 0);
        break;
    case U8X8_MSG_GPIO_MENU_HOME:
        u8x8_SetGPIOResult(u8x8, /* get menu home pin state */ 0);
        break;
    default:
        u8x8_SetGPIOResult(u8x8, 1); // default return value
        break;
    }
    return 1;
}
 
//U8g2的初始化，需要调用下面这个u8g2_Setup_ssd1306_128x64_noname_f函数，该函数的4个参数含义：
//u8g2：传入的U8g2结构体
//U8G2_R0：默认使用U8G2_R0即可（用于配置屏幕是否要旋转）
//u8x8_byte_sw_i2c：使用软件IIC驱动，该函数由U8g2源码提供
//u8x8_gpio_and_delay：就是上面我们写的配置函数
 
void u8g2Init(u8g2_t *u8g2)
{
	u8g2_Setup_ssd1306_i2c_128x64_noname_f(u8g2, U8G2_R0, u8x8_byte_hw_i2c, u8x8_gpio_and_delay); // 初始化u8g2 结构体
	u8g2_InitDisplay(u8g2);                                                                       // 
	u8g2_SetPowerSave(u8g2, 0);                                                                   // 
	u8g2_ClearBuffer(u8g2);
}
 
 
void draw(u8g2_t *u8g2)
{
	u8g2_ClearBuffer(u8g2); 
	
    u8g2_SetFontMode(u8g2, 1); /*字体模式选择*/
    u8g2_SetFontDirection(u8g2, 0); /*字体方向选择*/
    u8g2_SetFont(u8g2, u8g2_font_inb24_mf); /*字库选择*/
    u8g2_DrawStr(u8g2, 0, 20, "U");
    
    u8g2_SetFontDirection(u8g2, 1);
    u8g2_SetFont(u8g2, u8g2_font_inb30_mn);
    u8g2_DrawStr(u8g2, 21,8,"8");
        
    u8g2_SetFontDirection(u8g2, 0);
    u8g2_SetFont(u8g2, u8g2_font_inb24_mf);
    u8g2_DrawStr(u8g2, 51,30,"g");
    u8g2_DrawStr(u8g2, 67,30,"\xb2");
    
    u8g2_DrawHLine(u8g2, 2, 35, 47);
    u8g2_DrawHLine(u8g2, 3, 36, 47);
    u8g2_DrawVLine(u8g2, 45, 32, 12);
    u8g2_DrawVLine(u8g2, 46, 33, 12);
  
    u8g2_SetFont(u8g2, u8g2_font_4x6_tr);
    u8g2_DrawStr(u8g2, 1,54,"github.com/olikraus/u8g2");
		
	u8g2_SendBuffer(u8g2);
	HAL_Delay(1000);
}
 
//画点填充
void testDrawPixelToFillScreen(u8g2_t *u8g2)
{
  int t = 1000;
	u8g2_ClearBuffer(u8g2);
 
  for (int j = 0; j < 64; j++)
  {
    for (int i = 0; i < 128; i++)
    {
      u8g2_DrawPixel(u8g2,i, j);
    }
  }
  HAL_Delay(1000);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

上述编写的移植函数代码属于HAL库下的代码，标准库的代码其实差不多，有个别地方需要注意修改。有一定MCU编程基础的朋友应该很简单就可以做到仿写，如果有疑问可以留言。**移植代码的本质****：这些函数代码就是对应的U8g2图形库的接口函数，通过这些函数去启用U8g2图形库。**



## 4，**U8g2图形库测试函数**

**test.h：**

```
#ifndef __TEST_H
#define __TEST_H
 
#include "main.h"
#include "u8g2.h"
 
void testDrawProcess(u8g2_t *u8g2);
void testShowFont(u8g2_t *u8g2);
void testDrawFrame(u8g2_t *u8g2);
void testDrawRBox(u8g2_t *u8g2);
void testDrawCircle(u8g2_t *u8g2);
void testDrawFilledEllipse(u8g2_t *u8g2);
void testDrawMulti(u8g2_t *u8g2);
void testDrawXBM(u8g2_t *u8g2);
 
void u8g2DrawTest(u8g2_t *u8g2);
 
#endif
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

test.c

```
#include "test.h"
 
//---------------U8g2测试函数
 
#define SEND_BUFFER_DISPLAY_MS(u8g2, ms)\
  do {\
    u8g2_SendBuffer(u8g2); \
    HAL_Delay(ms);\
  }while(0);
 
 
//进度条显示
void testDrawProcess(u8g2_t *u8g2)
{
	for(int i=10;i<=80;i=i+2)
	{
		u8g2_ClearBuffer(u8g2); 
			
		char buff[20];
		sprintf(buff,"%d%%",(int)(i/80.0*100));
		
		u8g2_SetFont(u8g2,u8g2_font_ncenB12_tf);
		u8g2_DrawStr(u8g2,16,32,"STM32 U8g2");//字符显示
		
		u8g2_SetFont(u8g2,u8g2_font_ncenB08_tf);
		u8g2_DrawStr(u8g2,100,49,buff);//当前进度显示
		
		u8g2_DrawRBox(u8g2,16,40,i,10,4);//圆角填充框矩形框
		u8g2_DrawRFrame(u8g2,16,40,80,10,4);//圆角矩形
		
		u8g2_SendBuffer(u8g2);
	}
	HAL_Delay(500);
}
 
 
//字体测试 数字英文可选用 u8g2_font_ncenB..(粗) 系列字体
//u8g2_font_unifont_t_symbols/u8g2_font_unifont_h_symbols(细 圆润)
void testShowFont(u8g2_t *u8g2)
{
	int t = 1000;
	char testStr[14] = "STM32F103C8T6";
	
	u8g2_ClearBuffer(u8g2);
	
	u8g2_SetFont(u8g2,u8g2_font_u8glib_4_tf);
	u8g2_DrawStr(u8g2,0,5,testStr);
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	
	u8g2_SetFont(u8g2,u8g2_font_ncenB08_tf);
	u8g2_DrawStr(u8g2,0,30,testStr);
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	
    u8g2_SetFont(u8g2,u8g2_font_ncenB10_tr);
	u8g2_DrawStr(u8g2,0,60,testStr);
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
}
 
//画空心矩形
void testDrawFrame(u8g2_t *u8g2)
{
	int t = 1000;
	int x = 16;
	int y = 32;
	int w = 50;
	int h = 20;
	u8g2_ClearBuffer(u8g2);
	u8g2_DrawStr(u8g2,0, 15, "DrawFrame");
 
	u8g2_DrawFrame(u8g2, x, y, w, h);
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	u8g2_DrawFrame(u8g2, x+w+5, y-10, w-20, h+20);
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
}
 
//画实心圆角矩形
void testDrawRBox(u8g2_t *u8g2)
{
	int t = 1000;
	int x = 16;
	int y = 32;
	int w = 50;
	int h = 20;
	int r = 3;
	u8g2_ClearBuffer(u8g2);
	u8g2_DrawStr(u8g2,0, 15, "DrawRBox");
 
	u8g2_DrawRBox(u8g2, x, y, w, h, r);
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	u8g2_DrawRBox(u8g2, x+w+5, y-10, w-20, h+20, r);
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
}
 
//画空心圆
void testDrawCircle(u8g2_t *u8g2)
{
	int t = 600;
	int stx = 0;  //画图起始x
	int sty = 16; //画图起始y
	int with = 16;//一个图块的间隔
	int r = 15;   //圆的半径
	u8g2_ClearBuffer(u8g2);
	u8g2_DrawStr(u8g2, 0, 15, "DrawCircle");
 
	u8g2_DrawCircle(u8g2, stx, sty - 1 + with, r, U8G2_DRAW_UPPER_RIGHT); //右上
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	u8g2_DrawCircle(u8g2, stx + with, sty, r, U8G2_DRAW_LOWER_RIGHT); //右下
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	u8g2_DrawCircle(u8g2, stx - 1 + with * 3, sty - 1 + with, r, U8G2_DRAW_UPPER_LEFT); //左上
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	u8g2_DrawCircle(u8g2, stx - 1 + with * 4, sty, r, U8G2_DRAW_LOWER_LEFT); //左下
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	u8g2_DrawCircle(u8g2, stx - 1 + with * 2, sty - 1 + with * 2, r, U8G2_DRAW_ALL);//整个圆
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	
    u8g2_DrawCircle(u8g2, 32*3, 32, 31, U8G2_DRAW_ALL);//右侧整个圆
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
}
 
//画实心椭圆
void testDrawFilledEllipse(u8g2_t *u8g2)
{
	int t = 800;
	int with = 16;//一个图块的间隔
	int rx = 27;  //椭圆x方向的半径
	int ry = 22;  //椭圆y方向的半径
	u8g2_ClearBuffer(u8g2);
	u8g2_DrawStr(u8g2,0, 14, "DrawFilledEllipse");
 
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	u8g2_DrawFilledEllipse(u8g2, 0, with, rx, ry, U8G2_DRAW_LOWER_RIGHT);//右下
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	u8g2_DrawFilledEllipse(u8g2, with * 4 - 1, with, rx, ry, U8G2_DRAW_LOWER_LEFT); //左下
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	u8g2_DrawFilledEllipse(u8g2, 0, with * 4 - 1, rx, ry, U8G2_DRAW_UPPER_RIGHT); //右上
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	u8g2_DrawFilledEllipse(u8g2, with * 4 - 1, with * 4 - 1, rx, ry, U8G2_DRAW_UPPER_LEFT); //左上
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
	u8g2_DrawFilledEllipse(u8g2, with * 6, with * 2.5, rx, ry, U8G2_DRAW_ALL);//整个椭圆
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
}
 
//环形测试
void testDrawMulti(u8g2_t *u8g2)
{
	u8g2_ClearBuffer(u8g2);
	for (int j = 0; j < 64; j+=16)
	{
		for (int i = 0; i < 128; i+=16)
		{
			u8g2_DrawPixel(u8g2, i, j);
			u8g2_SendBuffer(u8g2);
		}
	}
  
	//实心矩形逐渐变大
    u8g2_ClearBuffer(u8g2);
	for(int i=30; i>0; i-=2)
	{
		u8g2_DrawBox(u8g2,i*2,i,128-i*4,64-2*i);
		u8g2_SendBuffer(u8g2);
	}
	//空心矩形逐渐变小
	u8g2_ClearBuffer(u8g2);
	for(int i=0; i<32; i+=2)
	{
		u8g2_DrawFrame(u8g2,i*2,i,128-i*4,64-2*i);
		u8g2_SendBuffer(u8g2);
	}
	
	//实心圆角矩形逐渐变大
	u8g2_ClearBuffer(u8g2);
	for(int i=30; i>0; i-=2)
	{
		u8g2_DrawRBox(u8g2,i*2,i,128-i*4,64-2*i,10-i/3);
		u8g2_SendBuffer(u8g2);
	}
    //空心圆角矩形逐渐变小
	u8g2_ClearBuffer(u8g2);
	for(int i=0; i<32; i+=2)
	{
		u8g2_DrawRFrame(u8g2,i*2,i,128-i*4,64-2*i,10-i/3);
		u8g2_SendBuffer(u8g2);
	}
	
	//实心圆逐渐变大
	u8g2_ClearBuffer(u8g2);
	for(int i=2; i<64; i+=3)
	{
		u8g2_DrawDisc(u8g2,64,32,i, U8G2_DRAW_ALL);
		u8g2_SendBuffer(u8g2);
	}
	//空心圆逐渐变小
	u8g2_ClearBuffer(u8g2);
	for(int i=64; i>0; i-=3)
	{
		u8g2_DrawCircle(u8g2,64,32,i, U8G2_DRAW_ALL);
		u8g2_SendBuffer(u8g2);
	}
	
	//实心椭圆逐渐变大
    u8g2_ClearBuffer(u8g2);
	for(int i=2; i<32; i+=3)
	{
		u8g2_DrawFilledEllipse(u8g2,64,32, i*2, i, U8G2_DRAW_ALL);
		u8g2_SendBuffer(u8g2);
	}
    //空心椭圆逐渐变小
    u8g2_ClearBuffer(u8g2);
	for(int i=32; i>0; i-=3)
	{
		u8g2_DrawEllipse(u8g2,64,32, i*2, i, U8G2_DRAW_ALL);
		u8g2_SendBuffer(u8g2);
	}
}
 
 
// width: 128, height: 48
const unsigned char bilibili[] U8X8_PROGMEM = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x03, 0x00, 0x07, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x07, 0x80, 0x07, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x0f, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x1e, 0xe0, 0x03, 0x00, 0xe0, 0x00, 0x00, 0x00, 0x00, 0x0e, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x3c, 0xf0, 0x00, 0x00, 0xf8, 0x00, 0x00, 0x00, 0x80, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00, 0xfc, 0xff, 0xff, 0xff, 0x01, 0xfc, 0x00, 0x00, 0x00, 0xc0, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00, 0xff, 0xff, 0xff, 0xff, 0x03, 0xfc, 0x00, 0x00, 0x3c, 0xc0, 0x0f, 0x00, 0x80, 0x03, 0x00, 0x80, 0xff, 0xff, 0xff, 0xff, 0x07, 0xfc, 0x00, 0x00, 0x3c, 0xc0, 0x0f, 0x00, 0xc0, 0x07, 0x00, 0x80, 0xff, 0xff, 0xff, 0xff, 0x0f, 0xfc, 0x00, 0x00, 0x3c, 0x80, 0x0f, 0x00, 0xc0, 0x07, 0x00, 0xc0, 0x0f, 0x00, 0x00, 0x80, 0x0f, 0xf8, 0x00, 0x00, 0x3c, 0x80, 0x0f, 0x00, 0x80, 0x07, 0x00, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x1f, 0xf8, 0x00, 0x00, 0x78, 0x80, 0x0f, 0x00, 0x80, 0x07, 0x00, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x1f, 0xf8, 0x00, 0x00, 0x78, 0x80, 0x0f, 0x00, 0x80, 0x07, 0x00, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x1f, 0xf8, 0x00, 0x80, 0x79, 0x80, 0x0f, 0x00, 0x98, 0x07, 0x00, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x1f, 0xf8, 0x00, 0xe0, 0x79, 0x9f, 0x0f, 0x00, 0xbe, 0xe7, 0x01, 0xc0, 0x07, 0x10, 0x40, 0x00, 0x1f, 0xf8, 0x00, 0xe0, 0x7b, 0x1f, 0x0f, 0x00, 0xbe, 0xe7, 0x01, 0xc0, 0x87, 0x1f, 0xe0, 0x0f, 0x1f, 0xf8, 0x00, 0xe0, 0x7b, 0x1e, 0x0f, 0x00, 0x3e, 0xe7, 0x01, 0xc0, 0xe7, 0x3f, 0xe0, 0x3f, 0x1f, 0xf0, 0x00, 0xe0, 0x7b, 0x1e, 0x0f, 0x00, 0x3e, 0xe7, 0x01, 0xc0, 0xe7, 0x3f, 0xe0, 0x3f, 0x1f, 0xf0, 0x00, 0x60, 0x71, 0x1e, 0x0f, 0x00, 0x34, 0xe7, 0x01, 0xc0, 0xe7, 0x07, 0x00, 0x3f, 0x1f, 0xf0, 0x00, 0x00, 0x70, 0x00, 0x1f, 0x00, 0x00, 0x07, 0x00, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x1f, 0xf0, 0x00, 0xc0, 0x73, 0x1e, 0x1f, 0x00, 0x3c, 0xc7, 0x01, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x1f, 0xf0, 0x00, 0xc0, 0x73, 0x1e, 0x1f, 0x00, 0x7c, 0xe7, 0x01, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x1f, 0xf0, 0x00, 0xc0, 0x73, 0x1e, 0x1f, 0x00, 0x7c, 0xef, 0x01, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x1f, 0xf0, 0x01, 0xc0, 0x77, 0x1e, 0x1e, 0x00, 0x7c, 0xef, 0x01, 0xc0, 0x07, 0x00, 0x03, 0x00, 0x1f, 0xf0, 0xff, 0xc1, 0xf7, 0x1e, 0xfe, 0x1f, 0x78, 0xef, 0x01, 0xc0, 0x07, 0x70, 0x37, 0x00, 0x1f, 0xe0, 0xff, 0x87, 0xf7, 0x1e, 0xfe, 0xff, 0x78, 0xee, 0x01, 0xc0, 0x07, 0xe0, 0x3f, 0x00, 0x1f, 0xe0, 0xff, 0x9f, 0xf7, 0x1e, 0xfe, 0xff, 0x79, 0xce, 0x01, 0xc0, 0x07, 0xc0, 0x18, 0x00, 0x1f, 0xe0, 0xff, 0xbf, 0xe7, 0x1e, 0xfe, 0xff, 0x7b, 0xce, 0x01, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x1f, 0xe0, 0xc7, 0xbf, 0xe7, 0x1e, 0xfe, 0xf8, 0x77, 0xce, 0x01, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x1f, 0xe0, 0x0f, 0x3f, 0xe7, 0x1c, 0xfe, 0xf0, 0x77, 0xce, 0x03, 0xc0, 0x07, 0x00, 0x00, 0x00, 0x1f, 0xe0, 0xcf, 0x3f, 0xe7, 0x1c, 0xfe, 0xf8, 0xf3, 0xce, 0x03, 0xc0, 0xff, 0xff, 0xff, 0xff, 0x0f, 0xe0, 0xef, 0x1f, 0xe7, 0x1c, 0xfe, 0xfe, 0xf1, 0xce, 0x03, 0x80, 0xff, 0xff, 0xff, 0xff, 0x0f, 0xe0, 0xff, 0x0f, 0xcf, 0x1c, 0xfc, 0xff, 0xf0, 0xc0, 0x03, 0x00, 0xff, 0xff, 0xff, 0xff, 0x07, 0xe0, 0xff, 0x03, 0x06, 0x1c, 0xfc, 0x7f, 0x60, 0xc0, 0x01, 0x00, 0xfe, 0xff, 0xff, 0xff, 0x03, 0xe0, 0xff, 0x00, 0x00, 0x00, 0xfc, 0x0f, 0x00, 0x00, 0x00, 0x00, 0xfc, 0xff, 0xff, 0xff, 0x00, 0xc0, 0x0f, 0x00, 0x00, 0x00, 0xfc, 0x00, 0x00, 0x00, 0x00, 0x00, 0xc0, 0x03, 0x00, 0x1e, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x80, 0x01, 0x00, 0x0c, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };
// width: 128, height: 48
const unsigned char three_support[] U8X8_PROGMEM = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xc0, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x06, 0x00, 0x00, 0x00, 0x00, 0xc0, 0x01, 0x00, 0x00, 0x00, 0xe0, 0x07, 0x00, 0x00, 0x00, 0x00, 0x07, 0x00, 0x00, 0x00, 0x00, 0xe0, 0x03, 0x00, 0x00, 0x00, 0xfc, 0x1f, 0x00, 0x00, 0x00, 0x00, 0x0f, 0x00, 0x00, 0x00, 0x00, 0xe0, 0x03, 0x00, 0x00, 0x00, 0xfe, 0x7f, 0x00, 0x00, 0x00, 0x80, 0x0f, 0x00, 0x00, 0x00, 0x00, 0xe0, 0x03, 0x00, 0x00, 0x00, 0xff, 0xff, 0x00, 0x00, 0x00, 0x80, 0x1f, 0x00, 0x00, 0x00, 0x00, 0xf0, 0x03, 0x00, 0x00, 0x80, 0x0f, 0xf0, 0x00, 0x00, 0x00, 0xc0, 0x3f, 0x00, 0x00, 0x00, 0x00, 0xf8, 0x03, 0x00, 0x00, 0x80, 0x0f, 0xf0, 0x01, 0x00, 0x00, 0xfc, 0xff, 0x01, 0x00, 0x00, 0xc0, 0xfd, 0xff, 0x00, 0x00, 0xc0, 0x7f, 0xfe, 0x01, 0x00, 0x00, 0xff, 0xff, 0x0f, 0x00, 0x00, 0xe0, 0xfd, 0xff, 0x01, 0x00, 0xc0, 0x1f, 0xf8, 0x03, 0x00, 0x00, 0xff, 0xff, 0x0f, 0x00, 0x00, 0xe0, 0xfd, 0xff, 0x01, 0x00, 0xc0, 0x0f, 0xf0, 0x03, 0x00, 0x00, 0xfe, 0xff, 0x07, 0x00, 0x00, 0xe0, 0xfd, 0xff, 0x01, 0x00, 0xc0, 0x67, 0xe6, 0x03, 0x00, 0x00, 0xfc, 0xff, 0x03, 0x00, 0x00, 0xe0, 0xfd, 0xff, 0x01, 0x00, 0xc0, 0x67, 0xe6, 0x03, 0x00, 0x00, 0xf8, 0xff, 0x01, 0x00, 0x00, 0xe0, 0xfd, 0xff, 0x00, 0x00, 0xc0, 0x67, 0xe6, 0x03, 0x00, 0x00, 0xf0, 0x7f, 0x00, 0x00, 0x00, 0xe0, 0xfd, 0xff, 0x00, 0x00, 0xc0, 0x67, 0xee, 0x03, 0x00, 0x00, 0xe0, 0x7f, 0x00, 0x00, 0x00, 0xe0, 0xfd, 0xff, 0x00, 0x00, 0x80, 0x7f, 0xfe, 0x01, 0x00, 0x00, 0xe0, 0xff, 0x00, 0x00, 0x00, 0xe0, 0xfd, 0xff, 0x00, 0x00, 0x80, 0x7f, 0xfe, 0x01, 0x00, 0x00, 0xf0, 0xff, 0x00, 0x00, 0x00, 0xe0, 0xfd, 0x7f, 0x00, 0x00, 0x00, 0xff, 0xff, 0x00, 0x00, 0x00, 0xf0, 0xff, 0x00, 0x00, 0x00, 0xe0, 0xfd, 0x7f, 0x00, 0x00, 0x00, 0xfe, 0xff, 0x00, 0x00, 0x00, 0xf8, 0xf9, 0x01, 0x00, 0x00, 0xe0, 0xfd, 0x7f, 0x00, 0x00, 0x00, 0xfe, 0x7f, 0x00, 0x00, 0x00, 0xf8, 0xf0, 0x00, 0x00, 0x00, 0xe0, 0xfd, 0x1f, 0x00, 0x00, 0x00, 0xf8, 0x1f, 0x00, 0x00, 0x00, 0x30, 0xc0, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xc0, 0x03, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };
 
void testDrawXBM(u8g2_t *u8g2)
{
	int t = 1000;
	u8g2_ClearBuffer(u8g2);
	u8g2_DrawStr(u8g2,0, 14, "DrawXBM");
 
	u8g2_DrawXBM(u8g2,0, 16, 128, 48, bilibili);
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
 
	u8g2_ClearBuffer(u8g2);
	u8g2_DrawStr(u8g2,0, 14, "bilibili");
	u8g2_DrawXBM(u8g2,0, 16, 128, 48, three_support);
	SEND_BUFFER_DISPLAY_MS(u8g2,t);
}
 
void u8g2DrawTest(u8g2_t *u8g2)
{
	testDrawProcess(u8g2);
	testDrawMulti(u8g2);
	//testDrawFrame(u8g2);
	//testDrawRBox(u8g2);
	//testDrawCircle(u8g2);
	//testDrawFilledEllipse(u8g2);
	testShowFont(u8g2);
	testDrawXBM(u8g2);
 
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 五、实验

## 1，完成显示任务1

OLED基础的初始化

oled.c中的代码：

```
#include "oled.h"
#include "asc.h"    //字库（可以自己制作）
#include "main.h"
 
void WriteCmd(unsigned char I2C_Command) //写命令利用I2C通讯
 {
	HAL_I2C_Mem_Write(&hi2c1,OLED0561_ADD,COM,I2C_MEMADD_SIZE_8BIT,&I2C_Command,1,100);
 }
		
void WriteDat(unsigned char I2C_Data)    //写数据利用I2C通讯
 {
		HAL_I2C_Mem_Write(&hi2c1,OLED0561_ADD,DAT,I2C_MEMADD_SIZE_8BIT,&I2C_Data,1,100);
  }
 
void OLED_Init(void)
{
	HAL_Delay(100); //这里的延时很重要
	
	WriteCmd(0xAE); //display off
	WriteCmd(0x20);	//Set Memory Addressing Mode	
	WriteCmd(0x10);	//00,Horizontal Addressing Mode;01,Vertical Addressing Mode;10,Page Addressing Mode (RESET);11,Invalid
	WriteCmd(0xb0);	//Set Page Start Address for Page Addressing Mode,0-7
	WriteCmd(0xc8);	//Set COM Output Scan Direction
	WriteCmd(0x00); //---set low column address
	WriteCmd(0x10); //---set high column address
	WriteCmd(0x40); //--set start line address
	WriteCmd(0x81); //--set contrast control register
	WriteCmd(0xff); //亮度调节 0x00~0xff
	WriteCmd(0xa1); //--set segment re-map 0 to 127
	WriteCmd(0xa6); //--set normal display
	WriteCmd(0xa8); //--set multiplex ratio(1 to 64)
	WriteCmd(0x3F); //
	WriteCmd(0xa4); //0xa4,Output follows RAM content;0xa5,Output ignores RAM content
	WriteCmd(0xd3); //-set display offset
	WriteCmd(0x00); //-not offset
	WriteCmd(0xd5); //--set display clock divide ratio/oscillator frequency
	WriteCmd(0xf0); //--set divide ratio
	WriteCmd(0xd9); //--set pre-charge period
	WriteCmd(0x22); //
	WriteCmd(0xda); //--set com pins hardware configuration
	WriteCmd(0x12);
	WriteCmd(0xdb); //--set vcomh
	WriteCmd(0x20); //0x20,0.77xVcc
	WriteCmd(0x8d); //--set DC-DC enable
	WriteCmd(0x14); //
	WriteCmd(0xaf); //--turn on oled panel
}
 
void OLED_SetPos(unsigned char x, unsigned char y) //设置起始点坐标
{ 
	WriteCmd(0xb0+y);
	WriteCmd(((x&0xf0)>>4)|0x10);
	WriteCmd((x&0x0f)|0x01);
}
 
void OLED_Fill(unsigned char fill_Data)//全屏填充
{
	unsigned char m,n;
	for(m=0;m<8;m++)
	{
		WriteCmd(0xb0+m);		//page0-page1
		WriteCmd(0x00);		//low column start address
		WriteCmd(0x10);		//high column start address
		for(n=0;n<128;n++)
			{
				WriteDat(fill_Data);
			}
	}
}
 
 
void OLED_CLS(void)//清屏
{
	OLED_Fill(0x00);
}
 
void OLED_ON(void)
{
	WriteCmd(0X8D);  //设置电荷泵
	WriteCmd(0X14);  //开启电荷泵
	WriteCmd(0XAF);  //OLED唤醒
}
 
void OLED_OFF(void)
{
	WriteCmd(0X8D);  //设置电荷泵
	WriteCmd(0X10);  //关闭电荷泵
	WriteCmd(0XAE);  //OLED休眠
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

main函数修改：

```
int main(void)
{
  /* USER CODE BEGIN 1 */
 
  /* USER CODE END 1 */
 
  /* MCU Configuration--------------------------------------------------------*/
 
  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();
 
  /* USER CODE BEGIN Init */
 
  /* USER CODE END Init */
 
  /* Configure the system clock */
  SystemClock_Config();
 
  /* USER CODE BEGIN SysInit */
 
  /* USER CODE END SysInit */
 
  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_I2C2_Init();
  MX_TIM1_Init();
  /* USER CODE BEGIN 2 */
  u8g2_t u8g2;
  u8g2Init(&u8g2);	
  /* USER CODE END 2 */
 
  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */
 
    /* USER CODE BEGIN 3 */
 
       u8g2_FirstPage(&u8g2);
       do
       {
				 draw(&u8g2);
 
				 u8g2DrawTest(&u8g2);
       } while (u8g2_NextPage(&u8g2));
		
 
  }
  /* USER CODE END 3 */
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdnimg.cn/direct/20be39dfbd5043aa82be24e65aee5dd0.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑





## 2，显示汉字：

![img](https://img-blog.csdnimg.cn/direct/53def98768d94ffca0d5a51cb0396254.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑



![img](https://img-blog.csdnimg.cn/direct/6384208045f54c7e9549753a3c64a99d.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

main代码修改：

```
	OLED_ShowCN_STR(10,3,0,7);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```
// Parameters     : x,y -- 起始点坐标(x:0~127, y:0~7); N:汉字在.h中的索引
// Description    : 显示ASCII_8x16.h中的汉字,16*16点阵
void OLED_ShowCN(unsigned char x, unsigned char y, unsigned char N)
{
	unsigned char wm=0;
	unsigned int  adder=32*N;
	OLED_SetPos(x , y);
	for(wm = 0;wm < 16;wm++)
	{
		WriteDat(F16x16[adder]);
		adder += 1;
	}
	OLED_SetPos(x,y + 1);
	for(wm = 0;wm < 16;wm++)
	{
		WriteDat(F16x16[adder]);
		adder += 1;
	}
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)