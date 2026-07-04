# ESP32：GPS 定位与 SD 卡记录



## 说明

用 NEO-6M GPS 模块获取经纬度，配合 SD 卡模块记录轨迹日志。学习 UART 串口解析 NMEA 数据、TinyGPS++ 库、SD 卡文件系统。



## 硬件

- ESP32 ×1

- NEO-6M GPS 模块 ×1

- Micro SD 卡模块 ×1



## 电路连接

| GPS | ESP32 | SD 模块 | ESP32 |

|-----|-------|---------|-------|

| VCC | 5V | VCC | 5V |

| GND | GND | GND | GND |

| TX | GPIO16(RX2) | CS | GPIO5 |

| RX | GPIO17(TX2) | MOSI | GPIO23 |

| - | - | MISO | GPIO19 |

| - | - | SCK | GPIO18 |



## 代码

```cpp

#include <TinyGPS++.h>

#include <SD.h>



TinyGPSPlus gps;

HardwareSerial gpsSerial(2); // 使用 UART2



const int SD_CS = 5;

File logFile;



void setup() {

  Serial.begin(115200);

  gpsSerial.begin(9600, SERIAL_8N1, 16, 17); // RX=16, TX=17



  // 初始化 SD 卡

  if (!SD.begin(SD_CS)) {

    Serial.println("SD 卡初始化失败！");

  } else {

    logFile = SD.open("/gps_log.txt", FILE_APPEND);

    if (!logFile) {

      logFile = SD.open("/gps_log.txt", FILE_WRITE);

    }

    logFile.println("====== 新记录会话 ======");

    logFile.println("日期时间,纬度,经度,高度,速度");

    logFile.close();

    Serial.println("SD 卡就绪");

  }

}



void loop() {

  // 持续读取 GPS 数据

  while (gpsSerial.available() > 0) {

    gps.encode(gpsSerial.read());

  }



  // 每 2 秒更新

  static unsigned long lastUpdate = 0;

  if (millis() - lastUpdate >= 2000) {

    lastUpdate = millis();



    if (gps.location.isValid()) {

      double lat = gps.location.lat();

      double lng = gps.location.lng();

      double alt = gps.altitude.meters();

      double spd = gps.speed.kmph();



      // 串口打印

      Serial.printf("📍 %.6f, %.6f | 海拔 %.1fm | 速度 %.1fkm/h\n",

                    lat, lng, alt, spd);



      // 写入 SD 卡

      logFile = SD.open("/gps_log.txt", FILE_APPEND);

      if (logFile) {

        if (gps.date.isValid() && gps.time.isValid()) {

          logFile.printf("%04d-%02d-%02d %02d:%02d:%02d,",

            gps.date.year(), gps.date.month(), gps.date.day(),

            gps.time.hour(), gps.time.minute(), gps.time.second());

        }

        logFile.printf("%.6f,%.6f,%.1f,%.1f\n", lat, lng, alt, spd);

        logFile.close();

      }

    } else {

      Serial.print(".");

    }

  }

}

```



## 教学重点

- GPS 通过 UART 输出 NMEA 0183 字符串，`TinyGPS++` 自动解析

- `gps.location.isValid()` 判断是否已定位（室内通常无效）

- SD 卡模块用 SPI 协议，引脚多但速度快

- CSV 格式记录方便后续用 Excel 分析轨迹

- `FILE_APPEND` 追加写入，不会覆盖历史记录



## 常见错误

- GPS 首次冷启动定位需 30 秒~数分钟（室外空旷处）

- SD 卡格式必须是 FAT32，不支持 exFAT/NTFS

- SPI 引脚顺序不对 → SD 卡初始化失败

- GPS 的 TX/RX 需交叉连接（GPS TX→ESP32 RX）

