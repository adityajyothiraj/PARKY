# PARKY

![project_model1](https://github.com/user-attachments/assets/91d5ef18-e39c-4dec-bb06-7b576a64bf72)

PARKY is an innovative parking management system that allows users to book parking slots through a dedicated mobile app. The system ensures seamless entry and exit by using OCR (Optical Character Recognition) and OpenCV to scan vehicle registration numbers.
Booking: Users enter their vehicle’s registration number and name via the mobile app, which is then saved to a Firebase database.
Entry & Exit Gates: At the gates, cameras scan the vehicle’s registration number, compare it with the database, and automatically open the gate if the numbers match.
Database Management: After the vehicle exits, the registration number is removed from the database, ensuring efficient and secure space management.
Technologies used: Firebase, OpenCV, OCR, Mobile App (Frontend).

## AIM
The primary aim of the PARKY project is to develop an automated and efficient parking management system that allows users to easily book parking slots via a mobile application. The system utilizes Optical Character Recognition (OCR) and OpenCV to scan and validate vehicle registration numbers at entry and exit gates, ensuring smooth access control. By integrating Firebase for real-time database management, the system securely stores and manages vehicle registration data. The goal is to streamline the parking process, enhance user convenience, and ensure efficient space utilization by automatically removing vehicle records upon exit.

## COMPONENTS
1. ESP32
2. SG90 Servo x 2
3. IR Sensor x 2
4. System for processing OpenCV
5. Webcam x 2
6. ESP32 Breakon Board
7. Connecting Wires
8. 12V DC Power Supply

## CIRCUIT DIAGRAM

![Parky_Circuit](https://github.com/user-attachments/assets/3473e66a-a44b-4f29-90d4-fd102198b0dc)

.. PIN servoPin ---> GPIO 32  
.. PIN servoPin2 ---> GPIO 33  
.. PIN IR1 ---> GPIO 35  
.. PIN IR2 ---> GPIO 36  

# PROCEDURE
1. Open Visual Studio Code.
2. Open PlatformIO and create a new project.
3. Interface the sensors and servo to the board.
4. Upload the code.

# LIBRARY USED
https://github.com/mobizt/Firebase-ESP-Client.git

https://github.com/madhephaestus/ESP32Servo.git

# CODE
```
#include <Arduino.h>
#if defined(ESP32)
#include <WiFi.h>
#elif defined(ESP8266)
#include <ESP8266WiFi.h>
#endif
#include <Firebase_ESP_Client.h>
#include <addons/TokenHelper.h>
#include <addons/RTDBHelper.h>
#include <ESP32Servo.h>
#include <ESP32PWM.h>
#define NUM_SLOTS 6
#define SLOT_LENGTH 50

#define IR1 35
#define IR2 36
#define WIFI_SSID "realme GT 6T"
#define WIFI_PASSWORD "Connect12345"

#define API_KEY "AIzaSyAmfbORfn10-S6ygaJv0yOVGK5xCawGEvg"
#define DATABASE_URL "https://parking-field-default-rtdb.firebaseio.com/"
#define USER_EMAIL "amrithendu4@gmail.com"
#define USER_PASSWORD "Firebase123"

FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;

int servoPin = 32;
int servoPin2 = 33;
String regnumbers = "";
String opencv_entry = "";
String opencv_exit = "";
String regArray[NUM_SLOTS]; 

Servo myServo;
Servo myServo2;

void Connect_WiFi();
void Firebase_Store(String PATH, String MSG);
String Firebase_getString(String PATH);
String filter(String);
//String conversion(String,String);
void recognition_entry();
void recognition_exit();
void entryServo(int);
void exitServo(int);

void setup() {
  Connect_WiFi();
  myServo.attach(servoPin);
  myServo2.attach(servoPin2);
  myServo.write(90);
  pinMode(IR1, INPUT);
  pinMode(IR2, INPUT);
}

void loop() 
{
  Serial.print("ir1=");
  Serial.println(digitalRead(IR1));
  Serial.print("ir2=");
  Serial.println(digitalRead(IR2));

  if(digitalRead(IR1) == 0)
  {
    Firebase_Store("SCANENTRY","1");
    recognition_entry();
  }
  else if(digitalRead(IR1) == 1)
  {
    //delay(2000);
    entryServo(0);
    Firebase_Store("SCANENTRY","0");
  }

  if(digitalRead(IR2) == 0)
  {
    Firebase_Store("SCANEXIT","1");
    recognition_exit();
  }
  else if(digitalRead(IR2) == 1)
  {
    //delay(2000);
    exitServo(0);
    Firebase_Store("SCANEXIT","0");
  }
}

void recognition_entry() 
{
  String msg1 = Firebase_getString("SLOTS");
  String msg2 = Firebase_getString("OPENCV_ENTRY");

  regnumbers = filter(msg1);
  opencv_entry = filter(msg2);
 
 
  Serial.println();
  Serial.print("REGISTERS = ");
  Serial.println(regnumbers);
  Serial.println();
  Serial.print("Entry = ");
  Serial.println(opencv_entry);
  

  // Split the string based on commas and store them in regArray
  int slotIndex = 0;
  unsigned int startPos = 0;  
  int endPos = regnumbers.indexOf(',');

  while (endPos != -1 && slotIndex < NUM_SLOTS) 
  {
    String regNum = regnumbers.substring(startPos, endPos);
    regArray[slotIndex++] = regNum;
    startPos = endPos + 1;
    endPos = regnumbers.indexOf(',', startPos);
  }

   //If there's a remaining part after the last comma
  if (startPos < regnumbers.length() && slotIndex < NUM_SLOTS) 
  {
    regArray[slotIndex] = regnumbers.substring(startPos);
  }

  for (int i = 0; i <= slotIndex; i++) 
  {
    if (strcmp(opencv_entry.c_str(), regArray[i].c_str()) == 0)
    {
      Serial.println("Entry Gate Open");
      entryServo(1);
    }
  }
}

void recognition_exit()
{
  String msg1 = Firebase_getString("SLOTS");
  String msg3 = Firebase_getString("OPENCV_EXIT");
  
  regnumbers = filter(msg1);
  opencv_exit = filter(msg3);

  Serial.print("Exit = ");
  Serial.println(opencv_exit);

  int slotIndex = 0;
  unsigned int startPos = 0;  
  int endPos = regnumbers.indexOf(',');



  while (endPos != -1 && slotIndex < NUM_SLOTS) 
  {
    String regNum = regnumbers.substring(startPos, endPos);
    regArray[slotIndex++] = regNum;
    startPos = endPos + 1;
    endPos = regnumbers.indexOf(',', startPos);
  }

   //If there's a remaining part after the last comma
  if (startPos < regnumbers.length() && slotIndex < NUM_SLOTS) 
  {
    regArray[slotIndex] = regnumbers.substring(startPos);
  }

  for (int c = 0; c <= slotIndex; c++) 
  {
    if (strcmp(opencv_exit.c_str(), regArray[c].c_str()) == 0)
    {
      Serial.println("Exit Gate Open"); 
      exitServo(1);
      //regArray[c] = "";
      //reg = conversion(,regnumbers);
      msg1.replace(regArray[c], "");
      Firebase_Store("SLOTS",msg1.c_str());
      Firebase_Store("UPDATE",String(c));
    }
  }
  Serial.print("reg ="); 
  Serial.print(msg1);
}

String filter(String msg)
{
  msg.replace("[", "");
  msg.replace("]", "");
  msg.replace("\\", "");
  msg.replace("\"", "");
  return msg;
}

void entryServo(int a)
{
  if(a == 1)
  {
    myServo.write(0);
  }
  else if(a == 0)
  {
    myServo.write(90);
  }
}

void exitServo(int a)
{
  if(a == 1)
  {
    myServo2.write(90);
  }
  else if(a == 0)
  {
    myServo2.write(0);
  }
}

  void Firebase_Store(String PATH,String MSG)
    {
          Serial.print("Uploading data \" ");
          Serial.print(MSG);
          Serial.print(" \"  to the location \" ");
          Serial.print(PATH);
          Serial.println(" \"");
          Firebase.RTDB.setString(&fbdo, PATH, MSG);
          delay(50);
    }

String Firebase_getString(String PATH) {
  String msg = "";
  if (Firebase.RTDB.getString(&fbdo, PATH)) {
    msg = fbdo.to<const char *>();
  } else {
    Serial.println("Firebase read error: " + fbdo.errorReason());
    msg = "";
  }
  delay(50); 
  return msg;
}

void Connect_WiFi() {
  Serial.begin(9600);
  Serial.println("Connecting to Wi-Fi...");
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(300);
  }
  Serial.println();
  Serial.print("Connected with IP: ");
  Serial.println(WiFi.localIP());

  Serial.printf("Firebase Client v%s\n\n", FIREBASE_CLIENT_VERSION);

  config.api_key = API_KEY;
  auth.user.email = USER_EMAIL;
  auth.user.password = USER_PASSWORD;
  config.database_url = DATABASE_URL;
  config.token_status_callback = tokenStatusCallback;

  #if defined(ESP8266)
    fbdo.setBSSLBufferSize(2048, 2048);
  #endif
  
  Firebase.begin(&config, &auth);
  Firebase.reconnectWiFi(true);
  Firebase.setDoubleDigits(5); 
  config.timeout.serverResponse = 10 * 1000; 
}
```

