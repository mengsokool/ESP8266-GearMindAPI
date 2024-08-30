## ESP8266-GearMindAPI: ปลดล็อกพลัง AI สำหรับงาน IoT 

**GearMind API** ช่วยให้คุณเชื่อมต่ออุปกรณ์ IoT เข้ากับ AI Chatbot อัจฉริยะ เพื่อวิเคราะห์ข้อมูล แก้ไขปัญหา และ แจ้งเตือน แบบ Real-time  

**ตัวอย่างการใช้งาน:**

* ระบบ Predictive Maintenance สำหรับเครื่องจักรในโรงงาน 
* ระบบแจ้งเตือน Error สำหรับ EV Charge Point 

**จุดเด่น:**

* ใช้งานง่าย ด้วย HTTP API (หรือ HTTPS ก็ได้)
* รองรับ JSON Payload 
* ตอบสนองรวดเร็ว 
* เชื่อมต่อกับ GearMind Chatbot  ที่ทรงพลัง 
* เหมาะสำหรับงาน IoT ทุกประเภท 

## เริ่มต้นใช้งาน 

### 1. รับ API Token จาก GearMind 

เข้าไปที่ [คู่มือการใช้งาน GearMind API](https://docs.geworn.cloud/dev-docs/gearmind-api) เพื่อดูขั้นตอนการรับ API Token

### 2. ติดตั้ง Library 

```
arduino-cli lib install ArduinoJson
```

### 3. ตัวอย่างโค้ด (ESP8266)

```cpp
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <ArduinoJson.h>

// WiFi Configuration 
const char* ssid = "YOUR_WIFI_SSID";    
const char* password = "YOUR_WIFI_PASSWORD"; 

// GearMind API Configuration 
const char* apiEndpoint = "http://gearmind.geworn.cloud/api/v1/chat";  
const char* bearerToken = "YOUR_API_TOKEN"; 

void setup() {
 Serial.begin(115200); 
 
 // เชื่อมต่อ WiFi 
 Serial.println("Connecting to WiFi..."); 
 WiFi.begin(ssid, password); 
 while (WiFi.status() != WL_CONNECTED) { 
  delay(1000); 
  Serial.print("."); 
 }
 Serial.println("\nConnected to WiFi!"); 
}

void loop() {
 // ตรวจสอบ WiFi Connection 
 if (WiFi.status() == WL_CONNECTED) { 

  // สร้าง HTTP Client 
  WiFiClient client; 
  HTTPClient http; 

  // เริ่มต้น HTTP Request ไปยัง GearMind API 
  if (http.begin(client, apiEndpoint)) { 

   // ตั้งค่า Timeout สำหรับ API Request (GearMind อาจใช้เวลาคิดนาน)
   http.setTimeout(60000); 

   // ตั้งค่า Header 
   http.addHeader("Content-Type", "application/json"); 
   http.addHeader("Authorization", String("Bearer ") + bearerToken); 

   // สร้าง JSON Payload (คำถาม) 
   StaticJsonDocument<200> doc; 
   doc["messages"][0]["role"] = "user"; 
   doc["messages"][0]["content"] = "สถานีชาร์จรถยนต์ ABB ขึ้น Error Code Isolation Test Failed แก้ไขอย่างไร"; 

   // แปลง JSON เป็น String 
   String requestBody; 
   serializeJson(doc, requestBody); 

   // ส่ง HTTP POST Request ไปยัง GearMind API 
   int httpCode = http.POST(requestBody); 

   // ประมวลผล Response 
   if (httpCode > 0) { 
    String payload = http.getString(); 
    Serial.println("Status code: " + String(httpCode)); 
    Serial.println("Response from GearMind: " + payload); 
   } else { 
    Serial.println("Error on HTTP request: " + http.errorToString(httpCode)); 
   }

   // ปิด HTTP Connection 
   http.end(); 
  } else { 
   Serial.println("Unable to connect to GearMind API"); 
  }
 }

 delay(60000); // รอ 1 นาที ก่อนส่ง Request ถัดไป 
}
```

### 4. ทดสอบ API 

* อัพโหลดโค้ดไปยัง ESP8266  
* เปิด Serial Monitor เพื่อดู Response จาก GearMind API 

## หมายเหตุ 

* อย่าลืมเปลี่ยน `YOUR_WIFI_SSID`, `YOUR_WIFI_PASSWORD`, และ `YOUR_API_TOKEN` เป็นค่าของคุณ 
* โค้ดนี้เป็นเพียงตัวอย่าง คุณสามารถปรับแต่ง JSON Payload และ Logic ตามความต้องการ 
