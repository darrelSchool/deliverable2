# WANTAM
![[Proof.jpeg]]
## Group members
1. 157769 Kariuki Darrel Kanyugo
2. 152066 Owiro Shawn Ochieng
3. 167074 Jaafar Latifah Jepchirchir
4. 168323 Wambari Michael Rimu
5. 166152 Precious Mwende Waweru
## Experiment A (https://wokwi.com/projects/467711131202275329)
![[1 LCD.png|231]]
![[1 Full.png|235]]
![[1 Output.png|236]]
### Code
```
#include <LiquidCrystal.h>

// LCD pins (RS, E, D4, D5, D6, D7)
LiquidCrystal lcd(2, 4, 5, 18, 19, 21);

const int MQ2_PIN = 34;     // Analog pin for MQ2
const int GAS_THRESHOLD = 485; // Adjust based on your calibration (0-4095)

void setup() {
  Serial.begin(115200);
  lcd.begin(16, 2);
  
  lcd.print("MQ2 Gas Sensor");
  lcd.setCursor(0, 1);
  lcd.print("Initializing...");
  delay(2000);
  lcd.clear();
}

void loop() {
  int gasValue = analogRead(MQ2_PIN);  // 0-4095 on ESP32
  
  // Print to Serial
  Serial.print("Gas Value: ");
  Serial.println(gasValue);
  
  // Display on LCD
  lcd.setCursor(0, 0);
  lcd.print("Gas: ");
  lcd.print(gasValue);
  lcd.print("   ");  // Clear extra digits
  
  lcd.setCursor(0, 1);
  if (gasValue > GAS_THRESHOLD) {
    lcd.print("WARNING! Gas!  ");
  } else {
    lcd.print("Air is clean   ");
  }
  
  delay(500);  // Update every 500ms
}
```
## Experiment B
![[2 Close up.jpeg|266]]
![[2 Full.png|268]]
![[2 Output.png|266]]
### Code
#### ESP32 connected to DHT
```
#include <DHT.h>
#include <HardwareSerial.h>

HardwareSerial SerialESP(1);  // UART1

// DHT22
#define DHTPIN 15
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

String receivedData = "";

void setup() {
  Serial.begin(115200);
  SerialESP.begin(9600, SERIAL_8N1, 18, 19);

  dht.begin();

  delay(2000);
}

void loop() {
  // Read DHT22
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  // Receive data from ESP32 #1
  while (SerialESP.available()) {
    receivedData = SerialESP.readStringUntil('\n');
  }

  int mq2Value = 0;
  String status = "NORMAL";
  if (receivedData.startsWith("MQ5:")) {
    int firstColon = receivedData.indexOf(':');
    int secondColon = receivedData.lastIndexOf(':');
    mq2Value = receivedData.substring(firstColon+1, secondColon).toInt();
    status = receivedData.substring(secondColon+1);
  }

  Serial.printf("Temp: %.1f°C | Hum: %.0f%% | MQ2: %d %s\n", t, h, mq2Value, status.c_str());

  delay(1000);
}

```
#### ESP32 connected to MQ5
```
#include <HardwareSerial.h>

HardwareSerial SerialESP(1);  // UART1

const int MQ2_PIN = 34;
const int GAS_THRESHOLD = 900;

void setup() {
  Serial.begin(115200);
  SerialESP.begin(9600, SERIAL_8N1, 18, 19); 

  Serial.println("ESP32 #1 - MQ2 Sender Started");
}

void loop() {
  int gasValue = analogRead(MQ2_PIN);

  // Send data to ESP32 #2
  SerialESP.print("MQ2:");
  SerialESP.print(gasValue);
  SerialESP.print(":");
  SerialESP.println(gasValue > GAS_THRESHOLD ? "HIGH" : "NORMAL");

  Serial.printf("Sent MQ2: %d\n", gasValue);

  delay(1000);
}

```

## Experiment C (https://wokwi.com/projects/468157940812693505)
### Code
#### ESP32 Connected to the DHT22 and Relay
```
#include <DHT.h>
HardwareSerial SerialESP(2);

#define DHTPIN 15
#define DHTTYPE DHT22
#define RELAY_PIN 25

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, LOW); 
  
  Serial.begin(115200);
  SerialESP.begin(9600);
  dht.begin();
}

void loop() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if (SerialESP.available()) {
    String data = SerialESP.readStringUntil('\n');
    if (data.indexOf("HIGH") != -1) {
      digitalWrite(RELAY_PIN, HIGH);
      Serial.println("Gas detected → Relay ON");
    } else {
      digitalWrite(RELAY_PIN, LOW);
    }
  }

  Serial.printf("Temp: %.1f°C | Hum: %.0f%%\n", t, h);
  delay(1000);
}
```
#### ESP32 connected to the MQ2 sensor
```
HardwareSerial SerialESP(2);

const int MQ5_PIN = 34;
const int GAS_THRESHOLD = 1200;   

void setup() {
  Serial.begin(115200);
  SerialESP.begin(9600);
}

void loop() {
  int gasValue = analogRead(MQ5_PIN);

  SerialESP.print("MQ5:");
  SerialESP.print(gasValue);
  SerialESP.print(":");
  SerialESP.println(gasValue > GAS_THRESHOLD ? "HIGH" : "NORMAL");

  Serial.printf("MQ-5 Value: %d\n", gasValue);
  delay(1000);
}
```

## Challenges Faced
1. For Experiment B we had to use a DHT11 since the DHT22s we found at the lab were not working.
2. For Experiment C we were unable to simulate 2 ESP32 at once on Wokwi since it is not possible. 