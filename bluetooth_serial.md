# ESP32：蓝牙串口通信



## 说明

将 ESP32 模拟为一个蓝牙串口设备，手机通过蓝牙发送指令控制板载 LED。学习 BLE 的基本概念和 BluetoothSerial 库。



## 硬件

- ESP32 ×1



## 代码

```cpp

#include <BluetoothSerial.h>



BluetoothSerial SerialBT;

const int LED_PIN = 2;



void setup() {

  pinMode(LED_PIN, OUTPUT);

  Serial.begin(115200);



  // 初始化蓝牙，设备名 "ESP32-BT"

  SerialBT.begin("ESP32-BT");

  Serial.println("蓝牙已启动，设备名: ESP32-BT");

  Serial.println("请用手机蓝牙调试助手连接并发送指令:");

  Serial.println("  ON  - 开灯");

  Serial.println("  OFF - 关灯");

  Serial.println("  BLINK - 闪烁 3 次");

}



void loop() {

  if (SerialBT.available()) {

    String cmd = SerialBT.readStringUntil('\n');

    cmd.trim();

    cmd.toUpperCase();



    Serial.print("收到蓝牙指令: ");

    Serial.println(cmd);



    if (cmd == "ON") {

      digitalWrite(LED_PIN, HIGH);

      SerialBT.println("LED 已打开");

    }

    else if (cmd == "OFF") {

      digitalWrite(LED_PIN, LOW);

      SerialBT.println("LED 已关闭");

    }

    else if (cmd == "BLINK") {

      SerialBT.println("开始闪烁...");

      for (int i = 0; i < 3; i++) {

        digitalWrite(LED_PIN, HIGH); delay(200);

        digitalWrite(LED_PIN, LOW);  delay(200);

      }

      SerialBT.println("闪烁完成");

    }

    else {

      SerialBT.println("未知指令: " + cmd);

      SerialBT.println("请使用 ON / OFF / BLINK");

    }

  }

}

```



## 教学重点

- `BluetoothSerial` 库将蓝牙模拟为普通串口，用法和 `Serial` 几乎一样

- ESP32 经典蓝牙（BT）与 BLE 不同，经典蓝牙更简单

- 手机上安装 "Serial Bluetooth Terminal" 等 App 测试

- 同时使用 `Serial`（USB）和 `SerialBT`（蓝牙）双串口调试



## 常见错误

- 手机连不上：确认蓝牙版本兼容（经典蓝牙 2.0+, BLE 4.0+）

- `SerialBT` 和 `Serial` 混淆调用

- 蓝牙名称太长或包含特殊字符

- 功率不足导致连接距离短（仅几米）

