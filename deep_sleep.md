# ESP32：深度睡眠低功耗



## 说明

ESP32 通过深度睡眠（Deep Sleep）模式实现超低功耗运行。用定时器唤醒 + 触摸唤醒，适合电池供电的传感器节点。



## 硬件

- ESP32 ×1

- DHT22 ×1

- 可选项：18650 电池



## 代码

```cpp

#include <DHT.h>

#define DHTPIN 4

#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);



// 唤醒原因枚举

// 0 - 上电启动

// 1 - 定时器唤醒

// 2 - 触摸唤醒

// 3 - 外部引脚唤醒



void setup() {

  pinMode(2, OUTPUT);

  Serial.begin(115200);

  delay(100);  // 等待串口稳定



  // 判断唤醒原因

  esp_sleep_wakeup_cause_t wakeReason = esp_sleep_get_wakeup_cause();

  switch (wakeReason) {

    case ESP_SLEEP_WAKEUP_TIMER:

      Serial.println("⏰ 定时器唤醒");

      break;

    case ESP_SLEEP_WAKEUP_TOUCHPAD:

      Serial.println("👆 触摸唤醒");

      break;

    default:

      Serial.println("🔄 上电启动");

      break;

  }



  dht.begin();

  float temp = dht.readTemperature();

  if (!isnan(temp)) {

    Serial.print("温度: "); Serial.print(temp); Serial.println(" °C");

    // 实际应用：通过 WiFi/MQTT 上报然后休眠

  }



  // 闪烁 LED 表示工作

  digitalWrite(2, HIGH); delay(100);

  digitalWrite(2, LOW);



  // ===== 进入深度睡眠 =====

  Serial.println("进入深度睡眠...");



  // 方式 1: 定时器唤醒（30 秒后醒来）

  esp_sleep_enable_timer_wakeup(30 * 1000000ULL); // 微秒



  // 方式 2: 触摸唤醒（触碰 T0 引脚即 GPIO4 醒来）

  // esp_sleep_enable_touchpad_wakeup();

  // touchAttachInterrupt(T0, NULL, 40);



  esp_deep_sleep_start();

  // 以下代码在唤醒前不会执行

}



void loop() {

  // 深度睡眠下不会执行 loop()

}

```



## 教学重点

- ESP32 有多种睡眠模式：Modem Sleep / Light Sleep / Deep Sleep

- Deep Sleep 下电流仅 10µA，适合电池供电

- `esp_sleep_get_wakeup_cause()` 判断唤醒来源

- 唤醒时 `setup()` 重新执行（类似复位），但某些 RTC 内存保留

- `esp_sleep_enable_timer_wakeup(us)` 微秒单位



## 常见错误

- 忘记 `delay(100)` → 串口数据还没来得及打印就休眠了

- 定时器单位是微秒，多写一个 0 变成睡几百秒

- GPIO 在深度睡眠时状态不确定 → 先设为合理状态

- WiFi 在 Deep Sleep 下断开 → 醒来需重连

