# ESP32：FreeRTOS 多任务并行



## 说明

ESP32 内置 FreeRTOS，可以创建真正并行的多任务（而 Arduino 只有单核循环）。学习 `xTaskCreate` 创建任务、任务优先级、互斥锁。



## 硬件

- ESP32 ×1

- LED ×3，蜂鸣器 ×1



## 代码

```cpp

// 任务句柄

TaskHandle_t taskLedHandle;

TaskHandle_t taskBuzzerHandle;

TaskHandle_t taskSerialHandle;



// 互斥锁（保护共享资源）

SemaphoreHandle_t serialMutex;



// 任务 1: LED 闪烁

void taskLed(void* pvParameters) {

  int pin = (int)pvParameters;

  pinMode(pin, OUTPUT);

  while (1) {

    digitalWrite(pin, !digitalRead(pin));

    vTaskDelay(200 / portTICK_PERIOD_MS); // FreeRTOS 延时

  }

}



// 任务 2: 蜂鸣器周期性响

void taskBuzzer(void* pvParameters) {

  pinMode(15, OUTPUT);

  while (1) {

    digitalWrite(15, HIGH);

    vTaskDelay(100 / portTICK_PERIOD_MS);

    digitalWrite(15, LOW);

    vTaskDelay(2000 / portTICK_PERIOD_MS);

  }

}



// 任务 3: 串口输出系统信息

void taskSerialMonitor(void* pvParameters) {

  while (1) {

    if (xSemaphoreTake(serialMutex, portMAX_DELAY)) {

      Serial.println("===== ESP32 系统信息 =====");

      Serial.printf("空闲堆: %d bytes\n", ESP.getFreeHeap());

      Serial.printf("任务 1 优先级: %d\n", uxTaskPriorityGet(taskLedHandle));

      Serial.printf("任务 2 优先级: %d\n", uxTaskPriorityGet(taskBuzzerHandle));

      Serial.println("=========================");

      xSemaphoreGive(serialMutex);

    }

    vTaskDelay(5000 / portTICK_PERIOD_MS);

  }

}



void setup() {

  Serial.begin(115200);

  serialMutex = xSemaphoreCreateMutex();



  // 创建 3 个并行任务

  xTaskCreate(taskLed, "LED", 1024, (void*)2, 1, &taskLedHandle);

  xTaskCreate(taskBuzzer, "Buzzer", 1024, NULL, 1, &taskBuzzerHandle);

  xTaskCreate(taskSerialMonitor, "Serial", 2048, NULL, 2, &taskSerialHandle);

  // 参数: 函数, 名称, 栈大小, 参数, 优先级, 句柄



  Serial.println("FreeRTOS 启动，3 个任务并行运行...");

}



void loop() {

  // ESP32 的 loop() 运行在最低优先级

  // FreeRTOS 任务优先级更高时会抢占 loop()

  delay(1000);

}

```



## 教学重点

- `xTaskCreate` 创建独立任务，有自己的栈空间

- 优先级数值越高越优先执行

- `vTaskDelay(ms / portTICK_PERIOD_MS)` 任务延时（非阻塞）

- 互斥锁 `xSemaphoreTake/Give` 保护共享资源（如 Serial）

- Arduino 的 `delay()` 和 FreeRTOS 的 `vTaskDelay()` 可以混用



## 常见错误

- 栈空间不足 → 程序崩溃（增大参数中的 1024/2048）

- 忘记 `portTICK_PERIOD_MS` → 延时不正确

- 高优先级任务死循环 → 低优先级任务餓死

- 没用互斥锁保护 Serial → 输出乱序

