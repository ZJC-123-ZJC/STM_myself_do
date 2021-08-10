# STM32点灯（基于HAL库+CubeMX）

创作者：张锦程

时间：2021.7.30

以下笔记大部分摘自b站up主：成电应电科协

实战使用开发板：正点原子stm32f103mini（芯片为stm32f103rct6）

介绍了基本的点灯，GPIO，状态机思想

## 新建工程

使用CubeMX新建工程（略，网上一搜就有，可以看看野火的教学视频）

### 1.设置串行模式

![image-20210810174721884](image-20210810174721884.png)

### 2.设置时钟

选择外部晶振

![image-20210810174820366](image-20210810174820366.png)

## 配置引脚

配置时注意使用CubeMX的user-label备注引脚名，增加可读性。

首先是LED引脚配置

LED引脚为

LED0 -> DS0 -> PA8

LED1 -> DS1 -> PD2

模式为

低电平，推挽输出，既不上拉也不下拉

将灯亮宏定义为LED_ON 将灯灭宏定义为LED_OFF



------

接下来是按键引脚配置

按键引脚为

KEY0 -> PC5

KEY1 -> PA15

WK_UP -> PA0

模式为

输入，既不上拉也不下拉



![image-20210810175138124](image-20210810175138124.png)

------

## GPIO八种模式的区别与使用：

以下内容了解即可，有个印象。

**1、开漏输出和推挽输出的区别？**

开漏输出：只可以输出强低电平，高电平得靠外部电阻拉高。输出端相当于三极管的集电极。适合于做电流型的驱动，其吸收电流的能力相对强(一般20ma以内)；

推挽输出:可以输出强高、低电平，连接数字器件。

**2、在STM32中选用怎样选择I/O模式？**

浮空输入_IN_FLOATING ——浮空输入，可以做KEY识别，RX1

带上拉输入_IPU——IO内部上拉电阻输入

带下拉输入_IPD—— IO内部下拉电阻输入

模拟输入_AIN ——应用ADC模拟输入，或者低功耗下省电

开漏输出_OUT_OD ——IO输出0接GND，IO输出1，悬空，需要外接上拉电阻，才能实现输出高电平。当输出为1时，IO口的状态由上拉电阻拉高电平，但由于是开漏输出模式，这样IO口也就可以由外部电路改变为低电平或不变。可以读IO输入电平变化，实现C51的IO双向功能

推挽输出_OUT_PP ——IO输出0-接GND， IO输出1 -接VCC，读输入值是未知的

复用功能的推挽输出_AF_PP ——片内外设功能（I2C的SCL、SDA）

复用功能的开漏输出_AF_OD——片内外设功能（TX1、MOSI、MISO.SCK.SS）

## 编写main函数

### main函数区域注意：

![image-20210728202954335](C:\Users\张aa\AppData\Roaming\Typora\typora-user-images\image-20210728202954335.png)

打开工程后先编译一次

在while（1）循环中写下以下内容

```C
//如果KEY0按下，翻转LED0的状态
		if(HAL_GPIO_ReadPin(KEY0_GPIO_Port, KEY0_Pin)==GPIO_PIN_RESET)
		{
			HAL_Delay(100);
			if(HAL_GPIO_ReadPin(KEY0_GPIO_Port, KEY0_Pin)==0)
			{
					HAL_GPIO_TogglePin(LED0_GPIO_Port, LED0_Pin);
			}
			// 等待按键释放
			while(HAL_GPIO_ReadPin(KEY0_GPIO_Port, KEY0_Pin)==GPIO_PIN_RESET);
		}
	}
```

实现多种按键功能最好定义一个函数，KEY_SCAN()，将功能写入

------

## 状态机写法

这里详情请参考b站up主：成电应电科协。这里不做详细介绍。

### 状态机设计思想：

![image-20210728202718288](C:\Users\张aa\AppData\Roaming\Typora\typora-user-images\image-20210728202718288.png)

### 状态机设计：

![image-20210728202740628](C:\Users\张aa\AppData\Roaming\Typora\typora-user-images\image-20210728202740628.png)

![image-20210728202607485](C:\Users\张aa\AppData\Roaming\Typora\typora-user-images\image-20210728202607485.png)

### 状态机实现：

### ![image-20210728202819297](C:\Users\张aa\AppData\Roaming\Typora\typora-user-images\image-20210728202819297.png)

### 程序编写：

下图例程定义TIM10定时器进行定时器中断

#### 用户自定义数据类型区域（Private typedef）：

### ![image-20210728203149536](C:\Users\张aa\AppData\Roaming\Typora\typora-user-images\image-20210728203149536.png)

#### 主程序代码区：

![image-20210728203221983](C:\Users\张aa\AppData\Roaming\Typora\typora-user-images\image-20210728203221983.png)

#### 定时器回调函数：

![image-20210728203522581](C:\Users\张aa\AppData\Roaming\Typora\typora-user-images\image-20210728203522581.png)

![image-20210728203647708](C:\Users\张aa\AppData\Roaming\Typora\typora-user-images\image-20210728203647708.png)

![image-20210728203752533](C:\Users\张aa\AppData\Roaming\Typora\typora-user-images\image-20210728203752533.png)

## 板级支持包构建方法

参考b站up主 成电应电科协 https://www.bilibili.com/video/BV1y7411m7gg

也可以不用板级支持包，直接写在main函数内

