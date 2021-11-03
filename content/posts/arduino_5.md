---
title: Arduino基础（五）蓝牙
date: 2018-10-30
category: embeded
tags: ['arduino']
---

参考资料：

- [HC-05 與 HC-06 藍牙模組補充說明（三）：使用 Arduino 設定 AT 命令](https://swf.com.tw/?p=712)
- [HC-06 蓝牙模块设置与使用](https://blog.csdn.net/xiaoping2994/article/details/53371659?locationNum=7&fps=1)
- [实操六蓝牙串口通讯——Android 手机控制 Arduino 单片机](https://blog.sina.com.cn/s/blog_c1526256010301vz.html)
- [蓝牙（BT04A、HC-05) 使用经验](https://blog.csdn.net/m0_37575064/article/details/76419088)

### 硬件介绍

目前比较常见的是 HC-05、HC-06、BT04-A 等蓝牙模块。这次以 BT04-A 为例，其他型号的模块可以参考最上面的参考链接，有关 HC-05 和 HC-06 的资料已经很全面详细了。

BT04-A 是一个非常弱鸡的蓝牙模块，只能用作从机，并且不兼容 iOS 系统。  
BT04-A 的默认波特率为`9600`。

手上没有 BT04-A 模块的图片，以 HC-04、HC-06 通用模块图为例：

![](https://pic.rhinoc.top/15409087936893.png)

下图是「电脑-Arduino-蓝牙模块」系统的示意图：  
![](https://pic.rhinoc.top/15409090354749.png)

接线的话，VCC 接`5V`，GND 接`GND`，TXD 接`8`（Arduino 的 RX），RXD 接`9`（Arduino 的 TX），即 TX 接 RX，RX 接 TX。

### 代码

```c
#include <SoftwareSerial.h>
SoftwareSerial BluetoothSerial(8, 9); // RX, TX
char value;
void setup() {
  //打开串行通信，等待端口打开：
  Serial.begin(9600);
  while (!Serial) {
    ; // 等待串口连接。
  }
  Serial.println("Serial Connected!");

  // 设置蓝牙串口通讯的速率 BT04-A默认是9600
  BluetoothSerial.begin(9600);
}

void loop() { // 循环
  if (Serial.available()) {   //检测单片机串口状态
    value = Serial.read();
    BluetoothSerial.write(value);//蓝牙模块将数据发送给单片机
  }

  if (BluetoothSerial.available()) {//检测蓝牙模块串口状态
    value = BluetoothSerial.read();
    Serial.write(value); //单片机将指令发送到蓝牙模块
  }
}
```

### 演示

#### 电脑端

这里稍微注意下换行的设置，看看是选择`Newline`还是`No Line ending`。

![](https://pic.rhinoc.top/15409089709546.jpg)

#### 手机端

使用酷安里下载的`蓝牙串口`应用。  
我用的是 MIUI 系统，配对成功后也不会显示已连接，不知道是不是个例。蓝牙的默认密码是`1234`，有关蓝牙显示名称和密码修改可以在串口用 AT 指令修改。  
![](https://pic.rhinoc.top/15409083935420.jpg)

### 心得体会

做这个实验的时候，真的心好累。主要原因是一开始不知道手上这个买套件送的蓝牙模块的型号，所以一开始按 HC-05 写代码，HC-05 的波特率为 38400bps，所以自然在串口上得到的只有问号和乱码。然后偶然发现 9600bps 的波特率下不会乱码，又以为是 HC-06，绕了一大弯，最后才发现是 BT04-A。

还有就是实验过程中差点烧坏蓝牙模块，因为调整电路的时候顺手就反插了蓝牙，导致正负极接反。

嗯等哪天有闲有钱不用买鞋的时候可以搞几个 HC-05 来（HC-06 也只能作为从机），做两个蓝牙设备的互连实验。
