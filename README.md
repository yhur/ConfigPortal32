# Config Portal

With this library, the developer can create a WiFi ESP8266 device which 
1. on power on, it provides with the Captive Portal if not configured, where the user can enter the configuration information such as SSID/password, and save to the ESP8266 flash filesystem.
2. or boots with the stored SSID/password and other programmed information if configured already.
3. connects to the WiFi and run the programmed setup function
4. it will serve as written in the loop function
5. (factory reset) if needed, you can connect GPIO0 to ground for more than 5 seconds, then the configuration will be deleted and the device can be reconfigured for the WiFi.

This is the screenshot of a sample captive portal.

<img width="381" alt="Captive Portal" src="https://user-images.githubusercontent.com/13171662/150663373-ffb1d4e9-e128-41a6-9231-3cb9f6f495f7.png">

# How to use the ConfigPortal8266
You can create a PlatformIO project with the example directory and modify the src/main.cpp for your purpose and build it.

## src/main.cpp 
The following code is the example to use the library. 
```c
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <ESP8266mDNS.h>
#include <ConfigPortal8266.h>

char*               ssid_pfix = (char*)"CaptivePortal";
String              user_config_html = "";      

void setup() {
    Serial.begin(115200);

    loadConfig();
    // *** If no "config" is found or "config" is not "done", run configDevice ***
    if(!cfg.containsKey("config") || strcmp((const char*)cfg["config"], "done")) {
        configDevice();
    }
    WiFi.mode(WIFI_STA);
    WiFi.begin((const char*)cfg["ssid"], (const char*)cfg["w_pw"]);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    // main setup
    Serial.printf("\nIP address : "); Serial.println(WiFi.localIP());

    if (MDNS.begin("miniwifi")) {
        Serial.println("MDNS responder started");
    }    
}

void loop() {
    MDNS.update();
}
```

## Custom Configuration Parameter
If additional configuration parameter is needed, you can modify the `user_config_html` for your variable, and handle the variable as shown below. The custom varialbe here is yourVar in the `user_html` and handling in setup code as below

In the global section.
```c
String              user_config_html = ""
    "<p><input type='text' name='yourVar' placeholder='Your Variable'>";
char                yourVar[50];
```

To read and set the configuration parameter to a variable(Refer to the ArduinoJson library for type handling).
```c
    sprintf(yourVar, (const char*)cfg["yourVar"]);
```

![스크린샷 2022-11-01 오전 9 37 26](https://user-images.githubusercontent.com/13171662/199134241-0fecd9a7-2978-4be4-87ab-d8d2db8b66ba.png)

## Configuration File Upload Feature
In addition to the Captive Portal configuration capability, it has a feature that enables you to upload the configuration using the LittleFS file upload functions in both of Arduino IDE extension and the VSCode/PlatformIO.

The format of the configuration is as follows, and you place this under the folder named 'data' to upload.
<pre>
{
   "ssid":"APID",
   "w_pw":"APPassword",
   "yourVar":"sample data",
   "config":"done"
}
</pre>

<img width="381" alt="config.json" src="https://user-images.githubusercontent.com/13171662/199135726-7cd44226-7f76-4d32-98d6-ae5ceb9d016a.jpg">


Once `config.json` is ready, you can upload with PIO's 'Upload Filesystem Image' tool. Note: It uses the serial connection, so you need to stop the Serial Monitor if running.

<img width="500" alt="Upload Filesystem Image" src="https://user-images.githubusercontent.com/13171662/199135635-ab11cdb4-3908-4a57-ad92-a3e00657f869.png">

# Advanced Use
There is one custom user exit call back function pointer and one optional html file `postSave.html`.

* If you want/need to provide some intermediate function to run during the configuration stage, you can implement a function and assign it to `userConfigLoop` function pointer.
```c
    // *** If no "config" is found or "config" is not "done", run configDevice ***
    if(!cfg.containsKey("config") || strcmp((const char*)cfg["config"], "done")) {
        userConfigLoop = &customConfigLoop;
        configDevice();
    }
```
For example, if you want to update the sensor data on OLED or if you need to provide some information to the browser through the websocket even during the configuration process.

* If you want/need to show the post configuration information, you can create a html file with name `postSave.html` and place under data folder, this page will be sent to the browser when the user submit the configuation information. The built-in default page is as follows.


<img width="381" alt="config.json" src="https://user-images.githubusercontent.com/13171662/199158213-4e4e572a-4110-4333-8058-42a774a54cf5.PNG">
