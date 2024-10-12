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


## make a telegram account
- Download the Telegram app on your smartphone or use the desktop version.
- Create an account by registering your phone number and following the instructions.

## make a telegram bot
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
- Copy and paste this code
- At #define WIFI_SSID“” and #Define WIFI_PASSWORD “” change your own wifi data
- then put by #define BOT_TOKEN "" your token from the telgram bot you have make
- By String city = "";  Replace with your city
- By String apiKey = ""; Replace with your OpenWeatherMap API-key
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





