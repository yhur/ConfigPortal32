# Config Portal

With this library, the developer can create a WiFi ESP32 device which 
1. on power on, it provides with the Captive Portal if the WiFi is not configured, where the user can enter the configuration information such as SSID/password, and save to the ESP32 flash filesystem.
2. or boots with the stored WiFi SSID/password and other programmed information if configured already.
3. connects to the WiFi and help run the programmed setup function
4. it will serve as written in the loop function
5. (factory reset) if needed, you can connect GPIO0 to ground for more than 5 seconds, then the configuration will be deleted and the device can be reconfigured for the WiFi.

This is the screenshot of a sample captive portal.

<img width="381" alt="Captive Portal" src="https://user-images.githubusercontent.com/13171662/150663373-ffb1d4e9-e128-41a6-9231-3cb9f6f495f7.png">

# How to use the ConfigPortal32
You can create a PlatformIO project with the example directory and modify the src/main.cpp for your purpose and build it.

## src/main.cpp 
The following code is the example to use the library. 
```c
#include <Arduino.h>
#include <ESPmDNS.h>
#include <ConfigPortal32.h>

char*               ssid_pfix = (char*)"CaptivePortal";
String              user_config_html = "";      
/*
 *  ConfigPortal library to extend and implement the WiFi connected IOT device
 *
 *  Yoonseok Hur
 *
 *  Usage Scenario:
 *  0. copy the example template in the README.md
 *  1. Modify the ssid_pfix to help distinquish your Captive Portal SSID
 *          char   ssid_pfix[];
 *  2. Modify user_config_html to guide and get the user config data through the Captive Portal
 *          String user_config_html;
 *  2. declare the user config variable before setup
 *  3. In the setup(), read the cfg["meta"]["your field"] and assign to your config variable
 *
 */

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

<img width="799" alt="Screenshot 2024-01-30 at 8 43 22 AM" src="https://github.com/yhur/ConfigPortal32/assets/13171662/e1e32a79-89b8-4ed7-915f-930216a2d818">


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

<img width="1815" alt="Screenshot 2024-11-06 at 11 28 18 AM" src="https://github.com/user-attachments/assets/9457b71f-9717-487e-96a3-2bc9bb3797c7">

# Advanced Use
There is one custom user exit call back function pointer and one optional html file `postSave.html`.

1. If you want/need to provide some intermediate function to run during the configuration stage, you can implement a function and assign it to `userConfigLoop` function pointer.
```c
    // *** If no "config" is found or "config" is not "done", run configDevice ***
    if(!cfg.containsKey("config") || strcmp((const char*)cfg["config"], "done")) {
        userConfigLoop = &customConfigLoop;
        configDevice();
    }
```
For example, if you want to update the sensor data on OLED or if you need to provide some information to the browser through the websocket even during the configuration process.

2. If you want/need to show the post configuration information, you can create a html file with name `postSave.html` and place under data folder, this page will be sent to the browser when the user submit the configuation information. The built-in default page is as follows.


<img width="381" alt="config.json" src="https://user-images.githubusercontent.com/13171662/199158213-4e4e572a-4110-4333-8058-42a774a54cf5.PNG">
