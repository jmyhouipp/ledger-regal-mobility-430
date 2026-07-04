# ESP32：OTA 无线升级



## 说明

让 ESP32 支持 OTA (Over-The-Air) 无线升级固件。一次烧录后，后续可通过 WiFi 远程更新程序，无需 USB 线。是物联网设备必备功能。



## 硬件

- ESP32 ×1



## 代码

```cpp

#include <WiFi.h>

#include <ArduinoOTA.h>



const char* ssid     = "你的WiFi名";

const char* password = "你的WiFi密码";



unsigned long lastBlink = 0;

int ledState = LOW;



void setup() {

  pinMode(2, OUTPUT);

  Serial.begin(115200);



  // 连接 WiFi

  WiFi.mode(WIFI_STA);

  WiFi.begin(ssid, password);

  while (WiFi.waitForConnectResult() != WL_CONNECTED) {

    Serial.println("WiFi 连接失败！重启...");

    ESP.restart();

  }



  // 配置 OTA

  ArduinoOTA.setHostname("ESP32-OTA");

  ArduinoOTA.setPassword("admin");  // 可选：设置升级密码



  ArduinoOTA.onStart([]() {

    Serial.println("OTA 开始更新...");

  });

  ArduinoOTA.onEnd([]() {

    Serial.println("\nOTA 更新完成！即将重启...");

  });

  ArduinoOTA.onProgress([](unsigned int progress, unsigned int total) {

    Serial.printf("\r进度: %u%%", (progress * 100) / total);

  });

  ArduinoOTA.onError([](ota_error_t error) {

    Serial.printf("OTA 错误[%u]: ", error);

    if (error == OTA_AUTH_ERROR) Serial.println("认证失败");

    else if (error == OTA_BEGIN_ERROR) Serial.println("开始失败");

    else if (error == OTA_CONNECT_ERROR) Serial.println("连接失败");

    else if (error == OTA_RECEIVE_ERROR) Serial.println("接收失败");

    else if (error == OTA_END_ERROR) Serial.println("结束失败");

  });



  ArduinoOTA.begin();



  Serial.println("OTA 已就绪");

  Serial.print("IP 地址: "); Serial.println(WiFi.localIP());

}



void loop() {

  ArduinoOTA.handle();  // 处理 OTA 请求（必须持续调用）



  // 正常业务逻辑（LED 闪烁）

  if (millis() - lastBlink >= 1000) {

    lastBlink = millis();

    ledState = !ledState;

    digitalWrite(2, ledState);

  }

}

```



## 教学重点

- `ArduinoOTA` 库让 ESP32 支持 IDE 直接无线烧录

- 必须确保 `ArduinoOTA.handle()` 在 `loop()` 中执行

- OTA 的生命周期回调：`onStart/onEnd/onProgress/onError`

- 升级时 ESP32 会临时占约 2 倍固件的 Flash 空间

- OTA 后如需保留数据，可配置分区表使用 NVS/SPIFFS



## 常见错误

- OTA 上传中断 → 可能变砖（需 USB 重烧 bootloader）

- 没有 `ArduinoOTA.handle()` → IDE 找不到设备

- WiFi 不稳定导致 OTA 失败 → 靠近路由器

- 固件太大超出 OTA 分区 → 修改分区表

