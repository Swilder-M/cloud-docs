# 连接 ESP32

本文主要介绍如何在 ESP32 项目中使用 `PubSubClient` ，实现客户端与 MQTT 服务器的连接、订阅、收发消息、取消订阅等功能。

作为 ESP8266 的升级版本，[ESP32](https://www.espressif.com/zh-hans/products/socs/esp32) 是物联网项目的理想选择。除了 Wi-Fi 模块，该模块还包含蓝牙 4.0 模块。双核 CPU 工作频率为 80 至 240 MHz，包含两个 Wi-Fi 和蓝牙模块以及各种输入和输出引脚。 

## 前提条件

> 1. 已经创建了部署，在部署概览下可以查看到连接相关的信息，请确保部署状态为运行中。同时你可以使用 WebSocket 测试连接到 MQTT 服务器。
> 2. 在 `认证鉴权` > `认证` 中设置用户名和密码，用于连接验证。

本文中使用 [Arduino IDE](https://www.arduino.cc/en/guide/environment?setlang=cn) 作为代码编辑和上传，ArduinoIDE 包含了一个用于写代码的文本编辑器、一个消息区、一个文本控制台以及一个带有常用功能按钮和文本菜单的工具栏。软件连接 Arduino 和 Genuino 之后，能给所连接的控制板上传程序，还能与控制板相互通信。

## 安装依赖

在 Arduino IDE 中完成以下安装。

1. 安装 ESP32 开发板。

   点击**工具** -> **开发板** -> **开发板管理**。搜索 ESP32，点击**安装**。

2. 安装 PubSub client 库。

   点击**项目** -> **加载库** -> **管理库...**。搜索 PubSubClient，安装 PubSubClient by Nick O’Leary。

## 连接 MQTT 服务器
> 请在控制台的部署概览找到相关的地址以及端口信息，需要注意如果是基础版，端口不是 1883 或 8883 端口，请确认好端口。

### 连接设置

本文将使用 EMQX 提供的 [免费公共 MQTT 服务器](https://www.emqx.com/zh/mqtt/public-mqtt5-broker)，该服务基于 EMQX 的 [MQTT 物联网云平台](https://www.emqx.com/zh/cloud) 创建。服务器接入信息如下：

- Broker: **broker.emqx.io**
- TCP Port: **1883**
- WebSocket Port: **8083**

连接设置完成后，在 Arduino IDE 中按以下步骤编写代码：

1. 导入 WiFi 和 PubSubClient 库。

```c
#include <WiFi.h>
#include <PubSubClient.h>
```

2. 设置 Wi-Fi 名称和密码，以及 MQTT 服务器连接地址和端口。

```c
// WiFi
const char *ssid = "mousse"; // Enter your WiFi name
const char *password = "qweqweqwe";  // Enter WiFi password

// MQTT Broker
const char *mqtt_broker = "broker-cn.emqx.io";
const char *topic = "esp32/test";
const char *mqtt_username = "emqx";
const char *mqtt_password = "public";
const int mqtt_port = 1883;
```

3. 打开串行连接，以便于输出程序的结果并且连接到 Wi-Fi 网络。

```c
// Set software serial baud to 115200;
Serial.begin(115200);
// connecting to a WiFi network
WiFi.begin(ssid, password);
while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.println("Connecting to WiFi..");
}
```

4. 使用 PubSubClient 连接到公共 MQTT Broker。

```c
client.setServer(mqtt_broker, mqtt_port);
client.setCallback(callback);
while (!client.connected()) {
    String client_id = "esp32-client-";
    client_id += String(WiFi.macAddress());
    Serial.printf("The client %s connects to the public mqtt broker\n", client_id.c_str());
    if (client.connect(client_id.c_str(), mqtt_username, mqtt_password)) {
        Serial.println("Public emqx mqtt broker connected");
    } else {
        Serial.print("failed with state ");
        Serial.print(client.state());
        delay(2000);
    }
}
```

5. MQTT 服务器连接成功后，ESP32 将向 MQTT 服务器发布和订阅基于主题 `esp/test` 的消息。

```c
// publish and subscribe
client.publish(topic, "Hi EMQX I'm ESP32 ^^");
client.subscribe(topic);
```

6. 设置回调函数将主题名称打印到串行端口并打印从 `esp32/test` 主题接收的消息。

```c
void callback(char *topic, byte *payload, unsigned int length) {
    Serial.print("Message arrived in topic: ");
    Serial.println(topic);
    Serial.print("Message:");
    for (int i = 0; i < length; i++) {
        Serial.print((char) payload[i]);
    }
    Serial.println();
    Serial.println("-----------------------");
}
```

完整代码示例如下：

```c
#include <WiFi.h>
#include <PubSubClient.h>

// WiFi
const char *ssid = "mousse"; // Enter your WiFi name
const char *password = "qweqweqwe";  // Enter WiFi password

// MQTT Broker
const char *mqtt_broker = "broker-cn.emqx.io";
const char *topic = "esp32/test";
const char *mqtt_username = "emqx";
const char *mqtt_password = "public";
const int mqtt_port = 1883;

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
 // Set software serial baud to 115200;
 Serial.begin(115200);
 // connecting to a WiFi network
 WiFi.begin(ssid, password);
 while (WiFi.status() != WL_CONNECTED) {
     delay(500);
     Serial.println("Connecting to WiFi..");
 }
 Serial.println("Connected to the WiFi network");
 //connecting to a mqtt broker
 client.setServer(mqtt_broker, mqtt_port);
 client.setCallback(callback);
 while (!client.connected()) {
     String client_id = "esp32-client-";
     client_id += String(WiFi.macAddress());
     Serial.printf("The client %s connects to the public mqtt broker\n", client_id.c_str());
     if (client.connect(client_id.c_str(), mqtt_username, mqtt_password)) {
         Serial.println("Public emqx mqtt broker connected");
     } else {
         Serial.print("failed with state ");
         Serial.print(client.state());
         delay(2000);
     }
 }
 // publish and subscribe
 client.publish(topic, "Hi EMQX I'm ESP32 ^^");
 client.subscribe(topic);
}

void callback(char *topic, byte *payload, unsigned int length) {
 Serial.print("Message arrived in topic: ");
 Serial.println(topic);
 Serial.print("Message:");
 for (int i = 0; i < length; i++) {
     Serial.print((char) payload[i]);
 }
 Serial.println();
 Serial.println("-----------------------");
}

void loop() {
 client.loop();
}
```

## 测试连接

在成功连接 MQTT 服务器后，您可以使用 Arduino IDE 和 MQTT X 测试连接。

1. 使用 Arduino IDE 将完整代码上传到 ESP32，并打开串口监视器，选择 115200 波特率查看 ESP32 连接情况。
   ![esp32_connection](./_assets/esp32_connection.png)
2. 建立 MQTT X 客户端 与 MQTT 服务器的连接, 并向 ESP32 发送消息。
   ![esp32_mqttx](./_assets/esp32_mqttx.png)

## 更多内容

综上所述，我们实现了在 ESP32 项目中创建 MQTT 连接，模拟了使用客户端与 MQTT 服务器进行连接、订阅、收发消息的场景。您可以在 [这里](https://github.com/emqx/MQTT-Client-Examples/tree/master/mqtt-client-ESP32) 下载到示例的源码，同时也可以在 [GitHub](https://github.com/emqx/MQTT-Client-Examples) 上找到更多其他语言的 Demo 示例。
