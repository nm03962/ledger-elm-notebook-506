# 树莓派：14 段数码管显示（TM1638）



## 说明

用 TM1638 模块显示数字和按键输入。学习 SPI-like 通信协议、段码编程、按键扫描。



## 硬件

- 树莓派 ×1

- TM1638 数码管+按键模块 ×1



## 电路

| TM1638 | 树莓派 GPIO |

|--------|------------|

| STB | GPIO24 |

| CLK | GPIO23 |

| DIO | GPIO25 |

| VCC | 5V |

| GND | GND |



## 代码

```python

import RPi.GPIO as GPIO

import time



STB = 24

CLK = 23

DIO = 25



GPIO.setmode(GPIO.BCM)

GPIO.setup(STB, GPIO.OUT)

GPIO.setup(CLK, GPIO.OUT)

GPIO.setup(DIO, GPIO.OUT)



def send_byte(data):

    """发送一个 8 位字节（模拟 SPI）"""

    for i in range(8):

        GPIO.output(CLK, GPIO.LOW)

        GPIO.output(DIO, (data >> i) & 0x01)

        GPIO.output(CLK, GPIO.HIGH)



def send_command(cmd):

    """发送命令字节"""

    GPIO.output(STB, GPIO.LOW)

    send_byte(cmd)

    GPIO.output(STB, GPIO.HIGH)



def display_number(num):

    """在数码管上显示数字"""

    GPIO.output(STB, GPIO.LOW)

    send_byte(0xC0)  # 地址自增模式

    # 消隐所有位

    for addr in range(16):

        send_byte(0x00)

    GPIO.output(STB, GPIO.HIGH)



    # 简化：指定显示某几位

    # 实际需查 TM1638 段码表，这里略



send_command(0x8F)  # 显示开，最大亮度

send_command(0x44)  # 写数据到显示寄存器

display_number(1234)



try:

    while True:

        time.sleep(1)

except KeyboardInterrupt:

    send_command(0x80)  # 关闭显示

    GPIO.cleanup()

```



## 教学重点

- TM1638 通过 3 线（STB/CLK/DIO）通信，类似 SPI

- Bit-banging 模拟协议是嵌入式编程的基本功

- 段码表决定每个数字对应的 LED 段亮灭组合

- 命令字节的高 4 位是功能码，低 4 位是地址/亮度



## 常见错误

- 引脚接错 → 不显示或乱码

- 5V 供电不够（树莓派 5V 引脚限流）

- 忘记初始化指令 → 屏幕全黑

- 通信时序不满足芯片要求（树莓派比 Arduino 快很多）

