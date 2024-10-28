# How to Read Data from a Soil Moisture Sensor into ThingSpeak Using ESP32
In this project, we’ll build an IoT soil moisture monitoring system that uses an ESP32 and a soil moisture sensor to measure levels and send the data to ThingSpeak for remote monitoring.

## Prerequisites
### Hardware
- **ESP32 Board**: Provides Wi-Fi connectivity, acting as the brain of the project.
- **Soil Moisture Sensor**: Measures the moisture levels in the soil.
- **Soil Moisture Probes**: Insert into the soil to obtain readings.
- **Jumper Wires**: Connects the sensor to the ESP32.
- **Breadboard**: Used for prototyping circuit connections.

### Software
- **Arduino IDE**: For programming the ESP32. Make sure to install the ESP32 add-on (see instructions below).
- **ThingSpeak**: A cloud platform that allows you to publish and visualize sensor data using charts with timestamps, enabling remote access to sensor readings.

## Step-by-Step Guide

### Step 1: Preparing the Arduino IDE
1. Install the **ESP32 add-on** in Arduino IDE:
   - Go to *File > Preferences*.
   - In the *Additional Board Manager URLs* field, enter: `https://dl.espressif.com/dl/package_esp32_index.json`.
   - Then go to *Tools > Board > Board Manager*, search for "ESP32," and install it.

2. Install the **ThingSpeak Library**:
   - Go to *Sketch > Include Library > Manage Libraries…*.
   - Search for “ThingSpeak” and install the library by MathWorks.

![Library](https://miro.medium.com/v2/resize:fit:720/format:webp/1*9aVUJwSDtYCB2oyjjyWYZQ.png)   

### Step 2: Building the Circuit
To collect soil moisture data, connect the soil moisture sensor to your ESP32 as follows:

- Connect **VCC** of the sensor to **3.3V** on the ESP32.
- Connect **GND** of the sensor to a **GND** pin on the ESP32.
- Connect the **Signal** output of the sensor to **GPIO pin 34** on the ESP32.

![Schematic](https://miro.medium.com/v2/resize:fit:640/format:webp/1*QPkRnJNHW9RnXl4Sj2ONtA.jpeg)   

### Step 3: Setting Up ThingSpeak
1. **Create a ThingSpeak Account**:
   - Visit the ThingSpeak website and click on *Get Started for Free*.
   - Sign up or log in using your MathWorks account.

2. **Create a New Channel**:
   - Navigate to the *Channels* tab and click on *My Channels*.
   - Press the *New Channel* button, fill in the channel details (e.g., name, description), and add fields as necessary for multiple data types.
   - Click *Save Channel*.   

3. **Get the Write API Key**:
   - Open the *API Keys* tab.
   - Copy the Write API Key; you’ll need this in the code to send data from the ESP32 to ThingSpeak.

![API](https://miro.medium.com/v2/resize:fit:720/format:webp/1*6XKNp0P0-gBdrt-KLtEVBg.png)  

### Step 4: Code to Publish Sensor Readings to ThingSpeak
Copy the following code into your Arduino IDE:
```cpp 
#include <WiFi.h>
#include <HTTPClient.h>
#include "ThingSpeak.h"

const char* ssid = "REPLACE_WITH_YOUR_SSID";
const char* password = "REPLACE_WITH_YOUR_PASSWORD";
const char* server = "http://api.thingspeak.com/update";
const char* api_key = "YOUR_API_KEY";

#define SOIL_MOISTURE_PIN 34

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("Connected!");
}

void loop() {
  int soilMoistureValue = analogRead(SOIL_MOISTURE_PIN);
  float soilMoisturePercentage = (100 - ((soilMoistureValue / 4095.0) * 100));
  
  Serial.print("Soil Moisture: ");
  Serial.println(soilMoisturePercentage);

  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    String url = server + "?api_key=" + String(api_key) + "&field1=" + String(soilMoisturePercentage);
    http.begin(url);
    int httpResponseCode = http.GET();

    if (httpResponseCode > 0) {
      String payload = http.getString();
    } else {
      Serial.println("Error in sending request");
    }
    http.end();
  } else {
    Serial.println("WiFi Disconnected");
  }

  delay(30000);
}
  ```

## Explanation

### Wi-Fi Initialization
The ESP32 connects to your Wi-Fi network using the `ssid` and `password` variables. Once connected, it prints "Connected!" to the Serial Monitor.

### Reading Soil Moisture Levels
The soil moisture sensor data is read using `analogRead(SOIL_MOISTURE_PIN)`, and the result is converted to a percentage for easier interpretation.

### Uploading Data to ThingSpeak
The code sends a GET request to ThingSpeak, including the moisture data. If the request is successful, the data is stored in your ThingSpeak channel for visualization.

## Step 5: Testing and Monitoring
1. Upload the code to your ESP32.
2. Open the Serial Monitor at a baud rate of 115200.
3. Press the reset button on the ESP32 and observe as the sensor readings are published to ThingSpeak every 30 seconds.

### Visualizing the Data
To view the data, log in to your ThingSpeak account. Open your channel to see a chart that visualizes soil moisture readings over time.

## Wrapping Up
This project demonstrates how to use an ESP32 and a soil moisture sensor to create a cloud-connected monitoring system with ThingSpeak. By automating soil moisture data collection, you can optimize irrigation management for gardens or farms. You can further enhance this project by adding features such as alert triggers or automated pump controls based on soil moisture levels.

---

[For more information, you can watch the video here to see a hands-on demonstration](https://www.youtube.com/watch?v=WqP3gK8o5pA)

