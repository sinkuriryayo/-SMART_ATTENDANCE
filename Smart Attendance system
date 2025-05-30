#include <WiFi.h>
#include <SPI.h>
#include <MFRC522.h>
#include <WebServer.h>
#include <NTPClient.h>
#include <WiFiUdp.h>

#define SS_PIN 5
#define RST_PIN 22
#define RED_LED 2
#define GREEN_LED 3

MFRC522 mfrc522(SS_PIN, RST_PIN);

const char* ssid = "Chadai Developer";
const char* password = "chadai12";

WebServer server(80);
WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, "pool.ntp.org", 3600, 60000); // +1 hour offset for Africa/Kigali

String attendees = "";
String fullLog = "";
bool isLoggedIn = false;

const String adminUser = "admin";
const String adminPass = "1234";
String loginPageHtml;

void setup() {
  Serial.begin(115200);
  SPI.begin();
  mfrc522.PCD_Init();

  pinMode(RED_LED, OUTPUT);
  pinMode(GREEN_LED, OUTPUT);
  ledcAttachPin(RED_LED, 0);
  ledcAttachPin(GREEN_LED, 1);
  ledcSetup(0, 5000, 8);  // Red LED
  ledcSetup(1, 5000, 8);  // Green LED

  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected!");
  Serial.println(WiFi.localIP());

  timeClient.begin();

  loginPageHtml = R"rawliteral(
  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="UTF-8">
    <style>
      body {
        font-family: Arial;
        background-image: url('https://www.w3schools.com/w3images/forest.jpg');
        background-size: cover;
        background-position: center;
        padding: 40px;
        text-align: center;
        color: white;
      }
      input, button {
        padding: 10px;
        margin: 5px;
        width: 200px;
        background-color: rgba(0, 0, 0, 0.5);
        color: white;
        border: 1px solid #fff;
        border-radius: 5px;
      }
      button {
        cursor: pointer;
        background-color: #2196F3;
      }
      button:hover {
        background-color: #0b7dda;
      }
      p {
        color: red;
      }
    </style>
  </head>
  <body>
    <h2>WELCOME IN ATTENDANCE OF MUHANGA HANGAHUB</h2>
    <input type="text" id="user" placeholder="Username" /><br>
    <input type="password" id="pass" placeholder="Password" /><br>
    <button onclick="login()">Login</button>
    <p id="msg"></p>
    <script>
      function login() {
        const user = document.getElementById('user').value;
        const pass = document.getElementById('pass').value;
        fetch(`/auth?u=${user}&p=${pass}`).then(res => {
          if (res.status === 200) window.location.href = "/dashboard";
          else document.getElementById('msg').innerText = "Invalid credentials!";
        });
      }
    </script>
  </body>
  </html>
  )rawliteral";

  server.on("/", HTTP_GET, []() {
    server.sendHeader("Location", "/login");
    server.send(303);
  });

  server.on("/login", HTTP_GET, []() {
    server.send(200, "text/html", loginPageHtml);
  });

  server.on("/auth", HTTP_GET, []() {
    String u = server.arg("u");
    String p = server.arg("p");
    if (u == adminUser && p == adminPass) {
      isLoggedIn = true;
      server.send(200, "text/plain", "OK");
    } else {
      server.send(401, "text/plain", "Unauthorized");
    }
  });

  server.on("/dashboard", HTTP_GET, []() {
    if (!isLoggedIn) {
      server.sendHeader("Location", "/login");
      server.send(303);
      return;
    }
    server.send(200, "text/html", createWebPage());
  });

  server.on("/export", HTTP_GET, []() {
    String csv = "Name,Time\n";
    csv += fullLog;
    server.send(200, "text/csv", csv);
  });

  server.begin();
  Serial.println("Server started");
}

void loop() {
  server.handleClient();
  timeClient.update();

  if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
    String uid = "";
    for (byte i = 0; i < mfrc522.uid.size; i++) {
      uid += String(mfrc522.uid.uidByte[i] < 0x10 ? "0" : "");
      uid += String(mfrc522.uid.uidByte[i], HEX);
    }

    uid.toUpperCase();
    Serial.println("Card UID: " + uid);

    String name = getNameFromUID(uid);
    String timestamp = getTimeStamp();

    if (attendees.indexOf(name) == -1) {
      attendees += "<tr><td>" + name + "</td><td>" + timestamp + "</td></tr>";
      fullLog += name + "," + timestamp + "\n";
    }

    if (name == "Unknown User") {
      blinkLED(RED_LED, 0, "RED");
    } else {
      blinkLED(GREEN_LED, 1, "GREEN");
    }

    delay(1000); // Prevent double scan
  }
}

void blinkLED(int pin, int channel, const char* color) {
  for (int i = 0; i < 2; i++) {
    ledcWrite(channel, 255);
    Serial.printf("%s LED ON - Approx Voltage: %.2fV\n", color, 3.3);
    delay(200);

    ledcWrite(channel, 0);
    Serial.printf("%s LED OFF - Approx Voltage: 0.00V\n", color);
    delay(200);
  }
}

String createWebPage() {
  String html = R"rawliteral(
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <meta http-equiv="refresh" content="5">
      <style>
        body {
          font-family: Arial;
          background-image: url('https://images.unsplash.com/photo-1600708857499-ec8a3d3b93a1?auto=format&fit=crop&w=1470&q=80');
          background-size: cover;
          background-position: center;
          color: white;
          padding: 20px;
        }
        h1 { color: white; }
        table {
          width: 100%;
          border-collapse: collapse;
          background-color: rgba(0, 0, 0, 0.5);
        }
        th, td {
          border: 1px solid #ddd;
          padding: 10px;
          text-align: left;
          color: white;
        }
        th {
          background: #2196F3;
        }
        .top {
          margin-bottom: 20px;
        }
        button {
          padding: 10px 15px;
          background-color: #2196F3;
          color: white;
          border: none;
          border-radius: 5px;
          cursor: pointer;
        }
        button:hover {
          background-color: #0b7dda;
        }
      </style>
    </head>
    <body>
      <div class="top">
        <h1>Smart Attendance Dashboard</h1>
        <a href="/export"><button>Download CSV</button></a>
      </div>
      <table>
        <tr><th>Name</th><th>Time</th></tr>
        ATTENDEE_TABLE
      </table>
    </body>
    </html>
  )rawliteral";

  html.replace("ATTENDEE_TABLE", attendees);
  return html;
}

String getNameFromUID(String uid) {
  if (uid == "4589C801") return "John Doe";
  if (uid == "AED4F060") return "Jane Smith";
  if (uid == "DDDC3103") return "El-chadai munyaneza";
  return "Unknown User";
}

String getTimeStamp() {
  return timeClient.getFormattedTime();
}
