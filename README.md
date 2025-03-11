# PARKY
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
4. Webcam x 2
5. ESP32 Breakon Board
6. Connecting Wires
7. 12V DC Power Supply

## CIRCUIT DIAGRAM
![PARKY](.md/home/aditya-jyothiraj/Downloads/)

.. PIN servoPin ---> GPIO 32  
.. PIN servoPin2 ---> GPIO 33  
.. PIN IR1 ---> GPIO 35  
.. PIN IR2 ---> GPIO 36  

# PROCEDURE
1. Open Visual Studio Code.
2. Open PlatformIO and create a new project.
3. 

