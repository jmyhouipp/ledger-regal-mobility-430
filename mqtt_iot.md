# ESP32：MQTT 物联网通信



## 说明

ESP32 通过 MQTT 协议连接到云服务器，发布传感器数据并订阅控制指令。这是主流物联网方案的基础，学习 `PubSubClient` 库。



## 硬件

- ESP32 ×1

- DHT11 ×1



## 代码

```cpp

#include <WiFi.h>

#include <PubSubClient.h>

#include <DHT.h>



#define DHTPIN 4

#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);



const char* ssid     = "你的WiFi名";

const char* password = "你的WiFi密码";



// MQTT 服务器（可用免费的 broker.emqx.io）

const char* mqtt_server = "broker.emqx.io";

const int   mqtt_port   = 1883;

const char* mqtt_topic_temp = "esp32/sensor/temperature";

const char* mqtt_topic_humi = "esp32/sensor/humidity";

const char* mqtt_topic_ctrl = "esp32/control";



WiFiClient espClient;

PubSubClient client(espClient);



void callback(char* topic, byte* payload, unsigned int length) {

  Serial.print("收到消息 [");

  Serial.print(topic);

  Serial.print("]: ");

  String message = "";

  for (int i = 0; i < length; i++) {

    message += (char)payload[i];

  }

  Serial.println(message);



  // 处理控制指令

  if (message == "LED_ON")  digitalWrite(2, HIGH);

  if (message == "LED_OFF") digitalWrite(2, LOW);

}



void reconnect() {

  while (!client.connected()) {

    Serial.print("连接 MQTT...");

    String clientId = "ESP32-" + String(random(0xffff), HEX);

    if (client.connect(clientId.c_str())) {

      Serial.println("成功");

      client.subscribe(mqtt_topic_ctrl);  // 订阅控制主题

    } else {

      Serial.print("失败, rc=");

      Serial.print(client.state());

      delay(5000);

    }

  }

}



void setup() {

  pinMode(2, OUTPUT);

  Serial.begin(115200);



  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) delay(500);



  client.setServer(mqtt_server, mqtt_port);

  client.setCallback(callback);

}



void loop() {

  if (!client.connected()) reconnect();

  client.loop();



  // 每 5 秒发布传感器数据

  static unsigned long lastPub = 0;

  if (millis() - lastPub > 5000) {

    lastPub = millis();

    float temp = dht.readTemperature();

    float humi = dht.readHumidity();



    if (!isnan(temp) && !isnan(humi)) {

      client.publish(mqtt_topic_temp, String(temp).c_str());

      client.publish(mqtt_topic_humi, String(humi).c_str());

      Serial.printf("发布: %.1f°C, %.1f%%\n", temp, humi);

    }

  }

}

```



## 教学重点

- MQTT 的核心模型：发布者(Publish) → 服务器(Broker) → 订阅者(Subscribe)

- `client.loop()` 必须在主循环中持续调用，处理入站消息

- 回调函数 `callback()` 自动处理订阅主题的消息

- 免费 MQTT Broker：broker.emqx.io / test.mosquitto.org

- 使用 `random()` 生成唯一 ClientID 避免冲突



## 常见错误

- 忘记 `client.loop()` → 收不到消息

- ClientID 重复 → 连接断开

- Topic 层级用错（/esp32/temp vs esp32/temp）

- 发布频率太高被服务器限流

