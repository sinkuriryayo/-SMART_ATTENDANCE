# -SMART_ATTENDANCE
Project Overview
The Smart Attendance System is an IoT-based solution designed to track and log attendance using RFID cards. Built around the ESP32 microcontroller, this system uses an RFID-RC522 reader to identify users, captures real-time timestamps via NTP (Network Time Protocol), and displays attendance records on a secure web dashboard. It also allows exporting logs in CSV format and filtering by date.

Features
RFID-based attendance tracking
Web-based dashboard with real-time updates
Admin login system
Real-time timestamping using NTP
CSV export of attendance logs
Date-based filtering (optional enhancement)
Background themes:
Tree background for Login Page
Ice forest background for Dashboard
Components Used
ESP32 (with built-in Wi-Fi)
MFRC522 RFID Reader Module
RFID Tags/Cards
NTP (Network Time Protocol) for timestamps
HTML/CSS/JavaScript for web interface
MicroSD Card (optional, for storage)
Arduino IDE or PlatformIO
Wiring connectons
This project connects an ESP32 to an MFRC522 RFID module along with Red and Green LEDs to indicate RFID tag detection.

ESP32 to MFRC522 RFID Module Connections
MFRC522 Pin	ESP32 Pin	Function
SDA	D5	SS (Slave Select)
SCK	D18	SPI Clock
MOSI	D23	SPI MOSI
MISO	D19	SPI MISO
RST	D22	Reset
3.3V	3.3V	Power
GND	GND	Ground
LED Connections
LED Color	ESP32 Pin	Notes
Red	D2	With 220Ω resistor to GND
Green	D3	With 220Ω resistor to GND
Notes
Ensure the ESP32 provides 3.3V power to the RFID module (not 5V).
Use current-limiting resistors for LEDs to prevent damage.
Make sure the SPI pins match your ESP32 board's pinout if using a different variant.
How It Works
User Scans RFID Card: The ESP32 reads the UID of the RFID card using the MFRC522 module.
UID Matching: The UID is checked against predefined authorized users.
Timestamp Logging: If valid, the user’s name and current time (from NTP) are logged.
Web Dashboard: The log is displayed in a table format and updates every 2 seconds.
Admin Login: Secured access to the dashboard is provided via a custom login page.
Export: Attendance records can be downloaded as a CSV file.
User Interface
Login Page
Background: Tree scenery
Fields: Username & Password
Action: Redirects to dashboard if credentials are correct
Dashboard
Background: Ice forest scene
Table Columns: Name | Time
Buttons: "Export CSV", "Filter by Date" (optional)
