# Manual-airco-prototype

## Introduction
In this document you will learn how to control the LED using a weather API. you can also control the LED using a telegram bot and a button. learning and failure is part of it good luck!

## Requirements

- Esp82666
- LED
- Button
- Laptop

## Connecting Hardware
For connecting the LED to the ESP you have a table here:

|     LED       |    ESP8266    |
| ------------- | ------------- |
|     Geel      |      D2       |
|     Rood      |    3 volt     |
|     Zwart     |     gng       |


For connecting the Button to the ESP you have a table here:

|     LED       |    ESP8266    |
| ------------- | ------------- |
|     Zwart     |      D1       |
|     Rood      |     gng       |

make sure they are all properly connected otherwise they won't do it later when running code 


## make a Telegram account
- Download the Telegram app on your smartphone or use the desktop version.
- Create an account by registering your phone number and following the instructions.

## make a Telegram bot
- Search for the bot named BotFather in the Telegram app.
- Start a chat with BotFather and follow the instructions to create a new bot.
- After creating the bot, you will receive a unique API key (bot token). Keep this, as you will need it in your code to connect the bot.
 follow.
- Note that this is how you select the bot and not the bothfather so in this case “drizzle bot”
<img src="https://github.com/user-attachments/assets/2a7bc7d9-1d25-4cab-b5bc-9b7aeec2ddb4" alt="IMG_6610" width="200" />

## Install the libraries
- Make sure you have the appropriate Arduino libraries installed. You need at least the following libraries:
1. ESP8266WiFi: For connecting to WiFi.
2. WiFiClientSecure: For secure connections.
3. UniversalTelegramBot: For communicating with the Telegram bot.
4. Adafruit_neopixel.h (for operating the LED).
- You can install these libraries through the Arduino IDE:
1. Go to Sketch > Include Library > Manage Libraries.
2. Find the required libraries and install them.

<img width="150" alt="Scherm­afbeelding 2024-10-12 om 16 30 24" src="https://github.com/user-attachments/assets/00d29f85-3280-4cfd-9e68-ce46e8d89957">

## weather API key

- ga naar [OpenWeatherMap](https://home.openweathermap.org/)
- create een account door je gegevens in te voeren
- Als je een account heb ga je naar de dropdown van je naam en klik je op “My API keys” 
- Als je een Key wilt toevoegen dan zet je een naam hier en klik je op generate
- Nu komt de API key hier te staan deze heb je later nodig

## write the code

- Open de Arduino IDE en ga naar File > Examples > ESP8266 > EchoBot.
- remove the example code
- Copy and paste this code:
<details>
<summary>click here to copy the code</summary>

```cpp
#include <ESP8266WiFi.h>
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>
#include <Adafruit_NeoPixel.h>    // Voeg de NeoPixel library toe

// Wifi network station credentials
#define WIFI_SSID ""
#define WIFI_PASSWORD ""
// Telegram BOT Token (Get from Botfather)
#define BOT_TOKEN ""

// OpenWeatherMap API-instellingen
const char* server = "api.openweathermap.org";
String city = "Amsterdam,NL";        // Vervang door jouw stad
String apiKey = "";      // Vervang door jouw OpenWeatherMap API-sleutel

const unsigned long BOT_MTBS = 5000;   // Tijd tussen berichten checks (in milliseconden)

X509List cert(TELEGRAM_CERTIFICATE_ROOT);
WiFiClientSecure secured_client;
UniversalTelegramBot bot(BOT_TOKEN, secured_client);
WiFiClient client;

unsigned long bot_lasttime = 0;         // Variabele om de tijd van de laatste Telegram-berichten-check bij te houden

// LED-strip instellingen
#define LED_STRIP_PIN D2               // Pin voor de LED-strip
#define NUM_LEDS 8                     // Aantal LEDs op de strip
Adafruit_NeoPixel strip(NUM_LEDS, LED_STRIP_PIN, NEO_GRB + NEO_KHZ800);

// Knopinstellingen
#define BUTTON_PIN D1                  // Pin voor de knop
bool ledStatus = false;                // Houd bij of de LED-strip aan of uit is
bool lastButtonState = HIGH;           // Vorige status van de knop (gestart als niet-ingedrukt)
bool apiEnabled = true;                // Variabele om te controleren of de API actief is

void setup() {
  Serial.begin(115200);
  Serial.println();

  // Verbinding maken met het Wifi-netwerk
  Serial.print("Connecting to Wifi SSID ");
  Serial.print(WIFI_SSID);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  secured_client.setTrustAnchors(&cert);

  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(500);
  }
  Serial.print("\nWiFi connected. IP address: ");
  Serial.println(WiFi.localIP());

  // LED- en knopinitialisatie
  strip.begin();
  strip.show();

  pinMode(BUTTON_PIN, INPUT_PULLUP);

  // Tijd instellen via NTP
  configTime(0, 0, "pool.ntp.org");
  time_t now = time(nullptr);
  while (now < 24 * 3600) {
    delay(100);
    now = time(nullptr);
  }
}

void loop() {
  // Controleer op nieuwe Telegram-berichten
  if (millis() - bot_lasttime > BOT_MTBS) {
    int numNewMessages = bot.getUpdates(bot.last_message_received + 1);
    while (numNewMessages) {
      handleNewMessages(numNewMessages);
      numNewMessages = bot.getUpdates(bot.last_message_received + 1);
    }
    bot_lasttime = millis();
  }

  // Controleer op weerupdate als de API actief is
  if (apiEnabled) {
    getWeatherData();
  }

  // Knop polling
  checkButton();
}

void getWeatherData() {
  if (client.connect(server, 80)) {
    Serial.println("Connected to weather server");
    client.println("GET /data/2.5/weather?q=" + city + "&appid=" + apiKey + "&units=metric HTTP/1.1");
    client.println("Host: api.openweathermap.org");
    client.println("Connection: close");
    client.println();

    unsigned long timeout = millis();
    while (client.available() == 0) {
      if (millis() - timeout > 5000) {
        Serial.println(">>> Client Timeout!");
        client.stop();
        return;
      }
    }

    String line;
    while (client.available()) {
      line = client.readStringUntil('\n');
      if (line.startsWith("{")) {
        parseWeatherData(line);
        break;
      }
    }
  } else {
    Serial.println("Connection to weather server failed");
  }
}

void parseWeatherData(String jsonString) {
  DynamicJsonDocument doc(1024);
  DeserializationError error = deserializeJson(doc, jsonString);

  if (error) {
    Serial.print("JSON parsing failed: ");
    Serial.println(error.c_str());
    return;
  }

  float temperature = doc["main"]["temp"];
  Serial.print("Current temperature: ");
  Serial.println(temperature);

  // LED-strip activeren bij temperaturen boven 20 graden
  if (temperature < 20.0) { // Stel de temperatuurdrempel in
    setStripColor(0, 0, 255);  // blauw als temperatuur te hoog is
    Serial.println("LED on (cold)");
  } else {
    setStripColor(0, 0, 0);    // LED uit bij hogere temperatuur
    Serial.println("LED off (warm)");
  }
}

void setStripColor(uint8_t r, uint8_t g, uint8_t b) {
  for (int i = 0; i < NUM_LEDS; i++) {
    strip.setPixelColor(i, strip.Color(r, g, b));
  }
  strip.show();
}

void checkButton() {
  bool buttonState = digitalRead(BUTTON_PIN);
  // Controleer of de knop is ingedrukt
  if (buttonState == LOW && lastButtonState == HIGH) {
    // Toggle de LED status en API aan/uit logica
    if (!apiEnabled && !ledStatus) {
      ledStatus = true; // Zet de airco aan zonder API in te schakelen
      setStripColor(255, 255, 255); // Zet LED aan
      Serial.println("Button pressed, LED on and API remains disabled");
    } else {
      apiEnabled = false; // Zet de API uit wanneer de knop wordt ingedrukt
      ledStatus = !ledStatus; // Toggle de LED status
      if (ledStatus) {
        setStripColor(255, 255, 255); // Zet LED aan
        Serial.println("Button pressed, LED on and API disabled");
      } else {
        setStripColor(0, 0, 0); // Zet LED uit
        Serial.println("Button pressed, LED off and API disabled");
      }
    }
  }
  lastButtonState = buttonState; // Update de vorige knopstatus
}

void handleNewMessages(int numNewMessages) {
  for (int i = 0; i < numNewMessages; i++) {
    String message = bot.messages[i].text;
    Serial.print("Received message: ");
    Serial.println(message);

    // Reactie op Telegram berichten
    if (message == "airco on") {
      ledStatus = true;
      setStripColor(255, 255, 255);
      bot.sendMessage(bot.messages[i].chat_id, "airco is nu aan!", "");
    } else if (message == "airco off") {
      ledStatus = false;
      setStripColor(0, 0, 0);
      bot.sendMessage(bot.messages[i].chat_id, "airco is nu uit!", "");
    } else if (message == "API") {
      apiEnabled = true;  // Zet de API weer aan
      bot.sendMessage(bot.messages[i].chat_id, "API is nu ingeschakeld!", "");
      Serial.println("API enabled by Telegram command.");
    } else {
      bot.sendMessage(bot.messages[i].chat_id, "Ik begrijp het niet. Typ 'airco on' om de LED-strip aan te zetten of 'airco off' om hem uit te zetten.", "");
    }
  }
}

- At #define WIFI_SSID“” and #Define WIFI_PASSWORD “” **change your own wifi data**
- then put by #define BOT_TOKEN "" **your token from the telgram bot you have make**
- By String city = "";  **Replace with your city**
- By String apiKey = ""; **Replace with your OpenWeatherMap API-key**
<img width="500" alt="Scherm­afbeelding 2024-10-12 om 17 24 53" src="https://github.com/user-attachments/assets/396a8234-3d08-4fec-9a08-8b0cf501017f">

## Upload de code to the ESP8266
- Connect your ESP8266 to the computer.
- Choose the correct board and port in the Arduino IDE.
- Click the upload button to upload the code to your ESP8266.


## open the serial monitor

- Open the serial monitor in the Arduino IDE by clicking right above on the magnifying glass.
- Set the baud rate to 115200.
- Check if any error messages appear. If any strange characters appear, check your WiFi credentials:
- Make sure you have entered the correct SSID and password.

## fix errors troubleshooting:
If no connection is made to the esp8266 check the following:

- check that you have entered the correct wifi data
- If you are on a hotspot try using a wifi network
- Make sure you have connected the right pins 

If the API is not working then in the serial monitor you will see “client not detect” check:
- if you copied and pasted the correct API key. 
- Otherwise create a new key and try that.

## LED on

If all goes well, the LED is now connected to the API and if the temperature is above this many degrees the LED turns blue. 

## Start a conversation with the Telegram bot

- Start a chat with your bot in Telegram.
You should see a confirmation in the serial monitor that the bot is active.
- Now if you write “airco off” in the Telegram bot then the LED would now turn off and you will see in the serial monitor that this it is actually off. If you want to turn it on again then write “airco on”. - Note that if you write “airco off” then the API is no longer active. If you want to activate the API again then write “API” then the API is on again and the LED turns blue at the given value.
- If the LED is enabled then the LED is blue if you control the LED via telegram then it is white so you can distinguish when the API is enabled or not.

## Button click

- If the API is disabled then you could manually press the button so that it comes back on. Note that the API is then disabled and the LED turns white in color.

<img width="1394" alt="Scherm­afbeelding 2024-10-12 om 15 16 28" src="https://github.com/user-attachments/assets/fbe6b377-783f-4cbe-936f-bccc6e6f1a65">


 






