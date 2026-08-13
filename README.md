# Smart Sensor Data Logger 📊

## Arduino UNO + LM35 + LDR + DS3231 RTC + MicroSD

This project is an **Arduino-based environmental data logger** that measures temperature and light, adds a real-time timestamp using a DS3231 RTC, and stores the readings on a MicroSD card in CSV format.

The system reads temperature from an **LM35 sensor** and light level from an **LDR**, timestamps each measurement using the **DS3231 RTC**, and saves the data into a file named `DATA.CSV`.

The logger is designed to record approximately **1 sample per second (1 Hz)**.

## 🎯 Objectives

* Measure temperature using an LM35 sensor.
* Measure light level using an LDR.
* Add accurate timestamps using a DS3231 RTC.
* Store sensor readings on a MicroSD card.
* Save the data in CSV format.
* Create a timestamped dataset of 100 samples.
* Demonstrate an IoT-style environmental data acquisition system.

## 🛠️ Components Required

| Component               |    Quantity |
| ----------------------- | ----------: |
| Arduino UNO             |           1 |
| LM35 Temperature Sensor |           1 |
| LDR / Photoresistor     |           1 |
| Resistor                |           1 |
| DS3231 RTC Module       |           1 |
| MicroSD Card Module     |           1 |
| MicroSD Card            |           1 |
| Breadboard              |           1 |
| Jumper Wires            | As required |
| USB / Power Supply      |           1 |

## 🔌 Pin Configuration

| Device              | Arduino UNO |
| ------------------- | ----------- |
| LM35 OUT            | A0          |
| LDR Voltage Divider | A1          |
| DS3231 SDA          | A4 / SDA    |
| DS3231 SCL          | A5 / SCL    |
| MicroSD CS          | D10         |
| MicroSD SPI         | D10–D13     |

### SPI Pins

| SPI Signal | Arduino UNO |
| ---------- | ----------- |
| CS         | D10         |
| MOSI       | D11         |
| MISO       | D12         |
| SCK        | D13         |

## ⚙️ System Workflow

```text
       LM35 ──────────┐
                      │
                      ↓
                  Arduino UNO
                      ↑
                      │
       LDR ───────────┘
                      │
                      ↓
                 DS3231 RTC
                      │
                 Add Timestamp
                      │
                      ↓
                  MicroSD
                      │
                      ↓
                  DATA.CSV
                      │
                      ↓
               Timestamped Data
```

## 🔄 Working Principle

The system follows these steps:

1. **Read** temperature and light sensor values.
2. **Timestamp** each reading using the DS3231 RTC.
3. **Store** the readings in `DATA.CSV`.
4. **Wait** for approximately one second.
5. **Repeat** the process continuously.

```text
READ → TIME → STORE → WAIT → REPEAT
```

## 📌 Key Specifications

| Specification      | Details       |
| ------------------ | ------------- |
| Controller         | Arduino UNO   |
| Temperature Sensor | LM35 → A0     |
| Light Sensor       | LDR → A1      |
| RTC                | DS3231 → I2C  |
| Storage            | MicroSD → SPI |
| File               | DATA.CSV      |
| Sampling Rate      | 1 Hz          |
| Samples            | 100           |

## 💻 Software Requirements

* Arduino IDE
* SPI Library
* SD Library
* Wire Library
* RTClib

Install **RTClib** through the Arduino IDE Library Manager.

## 📚 Arduino Program

```cpp
#include <SPI.h>
#include <SD.h>
#include <Wire.h>
#include <RTClib.h>

const int SD_CS = 10;
const int TEMP_PIN = A0;   // LM35: 10 mV/deg C
const int LIGHT_PIN = A1;  // LDR voltage divider

RTC_DS3231 rtc;
File logFile;

void setup() {
  Serial.begin(9600);

  pinMode(SS, OUTPUT);

  if (!rtc.begin()) {
    Serial.println("RTC not found");
    while (1);
  }

  if (rtc.lostPower()) {
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  }

  if (!SD.begin(SD_CS)) {
    Serial.println("SD initialization failed");
    while (1);
  }

  if (!SD.exists("DATA.CSV")) {
    logFile = SD.open("DATA.CSV", FILE_WRITE);

    if (logFile) {
      logFile.println("timestamp,temperature_C,light_raw");
      logFile.close();
    }
  }

  Serial.println("Logger ready");
}

void loop() {
  DateTime now = rtc.now();

  int tempRaw = analogRead(TEMP_PIN);
  int lightRaw = analogRead(LIGHT_PIN);

  // For an LM35 on a 5 V Arduino with 10-bit ADC
  float voltage = tempRaw * (5.0 / 1023.0);
  float temperatureC = voltage * 100.0;

  logFile = SD.open("DATA.CSV", FILE_WRITE);

  if (logFile) {
    char timestamp[20];

    snprintf(
      timestamp,
      sizeof(timestamp),
      "%04d-%02d-%02d %02d:%02d:%02d",
      now.year(),
      now.month(),
      now.day(),
      now.hour(),
      now.minute(),
      now.second()
    );

    logFile.print(timestamp);
    logFile.print(",");
    logFile.print(temperatureC, 2);
    logFile.print(",");
    logFile.println(lightRaw);

    logFile.close();

    Serial.print(timestamp);
    Serial.print(" T=");
    Serial.print(temperatureC, 2);
    Serial.print(" C Light=");
    Serial.println(lightRaw);

  } else {
    Serial.println("Error opening DATA.CSV");
  }

  delay(1000);
}
```

## 📄 CSV Data Format

The generated `DATA.CSV` file contains three columns:

```text
timestamp,temperature_C,light_raw
```

Example:

```text
2026-08-11 10:00:00,23.97,503
2026-08-11 10:00:01,24.05,506
2026-08-11 10:00:02,24.06,526
2026-08-11 10:00:03,24.02,540
```

The report provides a 100-sample timestamped dataset using this format.

## ⏱️ Sampling Rate

The program uses:

```cpp
delay(1000);
```

This gives an intended sampling rate of approximately:

```text
1 sample / second
1 Hz
```

Therefore, collecting 100 samples takes approximately **100 seconds**, with small additional timing overhead from sensor conversion, RTC communication, SD writing and Serial output.

## 🧪 Testing Procedure

1. Format the MicroSD card.
2. Connect all modules.
3. Install the RTClib library.
4. Upload the Arduino sketch.
5. Open Serial Monitor.
6. Set baud rate to **9600**.
7. Allow the logger to run for at least **100 seconds**.
8. Safely stop the system before removing the SD card.
9. Open `DATA.CSV`.
10. Verify that the file contains **100 or more rows**.

## 🖥️ Expected Serial Monitor

```text
Logger ready

2026-08-11 10:00:00 T=23.97 C Light=503
2026-08-11 10:00:01 T=24.05 C Light=506
2026-08-11 10:00:02 T=24.06 C Light=526
```

## 📸 Project Evidence

Add your own real project evidence:

* Physical circuit photograph
* Arduino IDE code screenshot
* Serial Monitor screenshot
* MicroSD card / `DATA.CSV` screenshot
* Circuit wiring screenshot
* 100-sample CSV file

The report specifically asks for actual hardware and Serial Monitor screenshots before submission.

## 📁 Suggested Repository Structure

```text
smart-sensor-data-logger/
│
├── README.md
├── smart_sensor_data_logger.ino
├── DATA.CSV
│
├── images/
│   ├── circuit_diagram.png
│   ├── hardware_setup.png
│   ├── arduino_ide.png
│   ├── serial_monitor.png
│   └── sd_card_data.png
│
└── docs/
    └── project_report.pdf
```

## 🔋 Power & Reliability

SD cards can consume additional current during write operations. For battery-powered applications, the sampling frequency can be reduced, Serial output can be minimized, and records can be buffered before writing.

Always use the SD module according to its specified voltage and logic-level requirements.

## 🎥 Video Demonstration

**Video Link:** `YOUR_VIDEO_LINK_HERE`

Recommended demonstration:

```text
Show Complete Circuit
        ↓
Power Arduino UNO
        ↓
Show LM35 Temperature Reading
        ↓
Show LDR Light Reading
        ↓
Show DS3231 Timestamp
        ↓
Show MicroSD Card
        ↓
Open DATA.CSV
        ↓
Show Timestamped Records
```

## ✅ Result

The Smart Sensor Data Logger successfully demonstrates environmental data acquisition using an Arduino UNO. Temperature and light readings are collected, timestamped using the DS3231 RTC, and stored on a MicroSD card as a portable CSV dataset.

The project meets the **100-sample requirement** and provides a foundation for longer-term environmental monitoring.

## 👨‍💻 Project Information

**Project:** Smart Sensor Data Logger
**Controller:** Arduino UNO
**Temperature Sensor:** LM35
**Light Sensor:** LDR
**RTC:** DS3231
**Storage:** MicroSD Card
**File Format:** CSV
**File Name:** DATA.CSV
**Sampling Rate:** 1 Hz
**Serial Baud Rate:** 9600
