# ESP32：BMP280 气压传感器+OLED 显示



## 说明

通过 I2C 总线同时连接 BMP280 气压传感器和 0.96 寸 OLED 屏，制作一个便携气象站。学习 I2C 总线的一主多从通信架构。



## 硬件

- ESP32 ×1

- BMP280 气压传感器 ×1

- SSD1306 0.96" OLED ×1



## 电路连接

| 器件 | SDA | SCL | VCC | GND |

|------|-----|-----|-----|-----|

| BMP280 | GPIO21 | GPIO22 | 3.3V | GND |

| OLED | GPIO21 | GPIO22 | 3.3V | GND |



I2C 设备并联共享 SDA/SCL，通过不同地址区分。



## 代码

```cpp

#include <Wire.h>

#include <Adafruit_BMP280.h>

#include <Adafruit_SSD1306.h>



#define SCREEN_WIDTH 128

#define SCREEN_HEIGHT 64

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

Adafruit_BMP280 bmp;



void setup() {

  Serial.begin(115200);

  Wire.begin();



  // 初始化 OLED

  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {

    Serial.println("OLED 初始化失败!");

    while (1);

  }



  // 初始化 BMP280

  if (!bmp.begin(0x76)) {  // BMP280 的 I2C 地址通常是 0x76

    Serial.println("BMP280 初始化失败!");

    while (1);

  }



  bmp.setSampling(Adafruit_BMP280::MODE_NORMAL,

                  Adafruit_BMP280::SAMPLING_X2,  // 温度过采样

                  Adafruit_BMP280::SAMPLING_X16, // 压力过采样

                  Adafruit_BMP280::FILTER_X16,

                  Adafruit_BMP280::STANDBY_MS_500);



  display.clearDisplay();

  display.setTextSize(1);

  display.setTextColor(SSD1306_WHITE);

  display.setCursor(0, 0);

  display.println("气象站启动!");

  display.display();

  delay(1000);

}



void loop() {

  float temp = bmp.readTemperature();

  float pressure = bmp.readPressure() / 100.0F; // Pa → hPa

  float altitude = bmp.readAltitude(1013.25);   // 海拔（参考海平面气压）



  // 串口输出

  Serial.printf("温度: %.1f°C | 气压: %.1f hPa | 海拔: %.1f m\n",

                temp, pressure, altitude);



  // OLED 显示

  display.clearDisplay();

  display.setCursor(0, 0);

  display.printf("=== 气象站 ===");

  display.setCursor(0, 16);

  display.printf("温度: %.1f C", temp);

  display.setCursor(0, 32);

  display.printf("气压: %.1f hPa", pressure);

  display.setCursor(0, 48);

  display.printf("海拔: %.1f m", altitude);

  display.display();



  delay(2000);

}

```



## 教学重点

- I2C 总线允许多个设备共享 2 根线（SDA/SCL）

- 每个设备有唯一 7-bit I2C 地址（BMP280 = 0x76, OLED = 0x3C）

- Adafruit 库统一了常用传感器的 API 风格

- `readAltitude(seaLevelPressure)` 用当前气压推算海拔



## 常见错误

- 两个设备 I2C 地址冲突 → 更换地址或接不同总线

- 3.3V vs 5V：ESP32 用 3.3V，切勿接 5V

- 显示乱码：`display.display()` 漏调

- BMP280 地址可能是 0x76 或 0x77（看模块丝印）

