<h1 id="toc_0">Dash IoT Guide (Adafruit Bluefruit SPI Friend)</h1>

**12 February 2024**

This guide demonstrates how to make BLE, TCP and MQTT connections to an Adafruit Bluefruit SPI friend using the Arduino or PlatformIO IDE and the **DashIO** Adruino library. It also shows how to load user controls (widgets) into your mobile device to monitor and control an IoT device.

<h2 id="toc_1">Getting Started</h2>

For the big picture on **Dash**, take a look at our website: <a href="https://dashio.io">dashio.io</a>

<p>For the Dash arduino library: <a href="https://github.com/dashio-connect/arduino-dashioBluefruit">github.com/dashio-connect/arduino-dashioBluefruit</a></p>

<h2 id="toc_2">Requirements</h2>

<p>Grab a development board, <strong>Adafruit Bluefruit SPI friend</strong>, Arduino IDE and follow this guide.</p>

You will need to install the <strong>Dash</strong> app on your mobile phone or tablet. 

<h2 id="toc_3">Install</h2>

<h3 id="toc_4">Arduino IDE</h3>

You will need to add the **DashioBluefruit** library into your project.  It is included in the Arduino IDE Library manager. Search the library manager for the library titled "DashioBluefruit" and install. The Arduino IDE may also ask you to install the following libraries (dependencies). Please make sure they are all installed from the Arduino IDE Library Manager.

- **Dashio** is the core messageing library used to manage messages.
- **Adafruit BluefruitLE nRF51** by Adafruit is required to communicate with the Bluefruit SPI friend.

Make sure you edit the **bluefruitConfig.h** file so the pins and settings match your hardware wiring.

<h3 id="toc_5">PlatformIO</h3>

Using **Dash** with the **PlatformIO** IDE is easy.

1. Install the **Dash** library into your project. The easiest way to do this is to download the <a href="https://github.com/dashio-connect/arduino-dashio">**arduino-dashio**</a> library  into the ***lib*** directory of your project.
2. From the **arduino-dashio** library, go to the directory **arduino-dashio/examples/DashIO_ArduinoBluefruit** and copy the three ***bluefruit*** files in the directory into the **src** directory of your project.
3. Edit the **bluefruitConfig.h** file so the pins and settings match your hardware wiring.
4. Install the **Adafruit BluefruitLE nRF51** library using the PlatformIO **Libraries** manager.
5. Edit the ***platformio.ini*** file for you project by adding ```lib_ldf_mode = deep``` to enable all the required files to be found and also add ```monitor_speed = 115200``` to set the required serial monitor speed.

Your finished ***platformio.ini*** file should look something like this:

```
[env:teensy36]
platform = teensy
board = teensy36
framework = arduino
lib_deps = adafruit/Adafruit BluefruitLE nRF51@^1.10.0
lib_ldf_mode = deep
monitor_speed = 115200
```

<h2 id="toc_6">Guide</h2>

<h3 id="toc_7">BLE Basics</h3>

<p>Lets start with a simple BLE connection.</p>

```
#include "DashioBluefruitSPI.h"

#define DEVICE_TYPE "BF_SPI_BLE_Type"
#define DEVICE_NAME "Betty Name"

DashDevice dashDevice(DEVICE_TYPE);
DashBluefruit_BLE  ble_con(&dashDevice, true);

void setup() {
    Serial.begin(115200);

    dashDevice.name = DEVICE_NAME;
    ble_con.begin(true);
}

void loop() {
    ble_con.run();
}
```

<p>This is about the fewest number of lines of code necessary to get talking to the <strong>Dash</strong> app. There is a lot happening under the hood to make this work. After the <code>#include &quot;DashioBluefruitSPI.h&quot;</code> we create a device with the <em>device_type</em> as its only attribute. We also create a BLE connection, with the newly created device being an attribute to the connection. The second attribute of the BLE connection enables the received and transmitted messages to be printed to the serial monitor to make debugging easier.</p>

<p>In the <code>setup()</code> function we assign the device name to the device and start the BLE connection <code>ble_con.begin(true);</code>. The <em>device_ID</em> is automagically obtained from the macAddress of the Bluefruit SPI Friend. </p>

<p>The &quot;true&quot; attribute of the <code>ble_conn.begin(true);</code> causes the Bluefruit SPI friend to perform a factory reset to make sure everything is in a known state. This can be set to &quot;false&quot; when you have finished your code.</p>

<p>This device is discoverable by the <strong>Dash</strong> app. You can also discover your IoT device using a third party BLE scanner (e.g. BlueCap or &quot;nRF Connect&quot;). The BLE advertised name of your IoT device will be a concatentation of &quot;DashIO_&quot; and the <em>device_type</em>.</p>

<p>Setup the Arduino IDE serial monitor with 115200 baud and run the above code. Then run the <strong>Dash</strong> app on your mobile device and you will see connection &quot;WHO&quot; messages on the Arduino serial monitor as the Dash app detects and communicates with your IoT device.</p>

<p><img src="https://dashio.io/wp-content/uploads/2022/09/attention_44_yellow.png" width="20"> <strong>Troubleshooting:</strong> Occasionally, the <strong>Dash</strong> app is unable to discover a BLE connection to the IoT device. If this occurs, try deleting the the IoT device from the Bluetooth Settings of your phone or tablet.</p>

<p>If you like, you can assign your own <strong>device_ID</strong> by changing the BLE connection begin function call with the following:</p>

```
dashDevice.deviceID = &quot;myUniqueDevieID&quot;;
ble_con.begin(true, false);
```

<p>Lets add Dial control messages that are sent to the <strong>Dash</strong> app every second. To do this we create a new task to provide a 1 second time tick and then send a Dial value message from the loop every second.</p>

```
#include "DashioBluefruitSPI.h"

#define DEVICE_TYPE "BF_SPI_BLE_Type"
#define DEVICE_NAME "Betty Name"

DashDevice dashDevice(DEVICE_TYPE);
DashBluefruit_BLE  ble_con(&dashDevice, true);

void setup() {
    Serial.begin(115200);

    dashDevice.name = DEVICE_NAME;
    ble_con.begin(true);
}

void loop() {
    ble_con.run();
    ble_con.sendMessage(dashDevice.getDialMessage("D01", int(random(0, 100))));
    delay(1000);
}
```

<p>The line <code>dashDevice.getDialMessage(&quot;D01&quot;, int(random(0, 100)))</code> creates the message with two parameters. The first parameter is the <em>control_ID</em> which identifies the specific Dial control in the <strong>Dash</strong> app and the second parameter is simply the Dial value.</p>

<p>Once again, run the above code and the <strong>Dash</strong> app. This time, a new &quot;DIAL&quot; messages will be seen on the serial monitor every second.</p>

<p>The next step is to show the Dial values from the messages on a control on the <strong>Dash</strong> app.</p>

<p>In the <strong>Dash</strong> app, tap the <img src="https://dashio.io/wp-content/uploads/2021/07/iot_blue_44.png" width="20"> <strong>All Devices</strong> button, followed by the <img src="https://dashio.io/wp-content/uploads/2021/07/magnifying_glass_44_blue.png" width="20"> <strong>Find New Device</strong> button. Then select the <strong>BLE Scan</strong> option to show a list of new IoT devices that are BLE enabled. Your IoT device shpuld be shown in the list. Select your device and from the next menu select <strong>Create Device View</strong>. This will create an empty Device View for your new IoT deivce. You can now add controls to the Device View:</p>

<h3 id="toc_8">Adding Controls to Dash App</h3>

Once you have discovered your IoT device in the **Dash** app and have a **Device View** available for editing, you can add controls to the **Device View**:

<ul>
<li><strong>Start Editing</strong>: Tap the <img src="https://dashio.io/wp-content/uploads/2021/07/pencil_44_blue.png" width="20"> <strong>Edit</strong> button (it not already in editing mode).</li>
<li><strong>Add Dial Control</strong>: Tap the <img src="https://dashio.io/wp-content/uploads/2021/07/add_44_blue.png" width="20"> <strong>Add Control</strong> button and select the Dial control from the list.</li>
<li><strong>Edit Controls</strong>: Tap the Dial control to select it. The <img src="https://dashio.io/wp-content/uploads/2021/07/spanner_44_button.png" width="20"> <strong>Control Settings Menu</strong> button will appear in the middle of the Control. The Control can then be dragged and resized (pinch). Tapping the <img src="https://dashio.io/wp-content/uploads/2021/07/spanner_44_button.png" width="20"> button allows you to edit the Control settings where you can setup the style, colors and other characteristics of your Control. Make sure the <em>Control_ID</em> is set to the same value that is used in the Dial messages (in this case it should be set to &quot;D01&quot;).</li>
<li><strong>Quit editing</strong>: Tap the <img src="https://dashio.io/wp-content/uploads/2021/07/pencil_quit_44.png" width="20"> <strong>Edit</strong> button again.</li>
</ul>

<p>The Dial on the <strong>Dash</strong> app will now show the random Dial values as they arrive.</p>

<p>The next piece of the puzzle to consider is how your IoT device can receive data from the <strong>Dash</strong> app. Lets add a Knob and connect it to the Dial. </p>

<p>In the <strong>Dash</strong> app you will need to add a <strong>Knob</strong> control onto your <strong>Device View</strong>, next to your <strong>Dial</strong> control. Edit the <strong>Knob</strong> to make sure the <strong>Control ID</strong> of the Knob matches what you have used in your Knob messages (in this case it should be &quot;KB01&quot;), then quit edit mode.</p>

<p>Next, in the Arduino code we need to respond to messages coming in from a Knob control that we just added to the <strong>Dash</strong> app. To make the changes to your IoT device we add a callback, <code>processIncomingMessage</code>, into the BLE connection with the <code>setCallback</code> function.</p>

```
#include "DashioBluefruitSPI.h"

#define DEVICE_TYPE "BF_SPI_BLE_Type"
#define DEVICE_NAME "Betty Name"

DashDevice dashDevice(DEVICE_TYPE);
DashBluefruit_BLE  ble_con(&dashDevice, true);

void processIncomingMessage(MessageData *messageData) {
    switch (messageData->control) {
    case knob:
        if (messageData->idStr == "KB01") {
            String message = dashDevice.getDialMessage("D01", (int)messageData->payloadStr.toFloat());
            ble_con.sendMessage(message);
        }
        break;
    }
}

void setup() {
    Serial.begin(115200);

    dashDevice.name = DEVICE_NAME;
    ble_con.setCallback(&processIncomingMessage);
    ble_con.begin(true);
}

void loop() {
    ble_con.run();
}
```

<p>We obtain the Knob value from the message data payload that we receive in the <code>processIncomingMessage</code> function. We then create a Dial message with the value from the Knob and send this back to the <strong>Dash</strong> app. And remember to remove the timer and the horrible delay from the loop:</p>

<p>When you adjust the Knob on the <strong>Dash</strong> app, a message with the Knob value is sent your IoT device, which returns the Knob value into the Dial control, which you will see on the <strong>Dash</strong> app.</p>

<p>Finally, we should respond to the STATUS message from the <strong>Dash</strong> app. STATUS messages allows the IoT device to send initial conditions for each control to the <strong>Dash</strong> app as soon as a connection becomes active. Once again, we do this from the <code>processIncomingMessage</code> function and our complete code looks like this:</p>

```
#include "DashioBluefruitSPI.h"

#define DEVICE_TYPE "BF_SPI_BLE_Type"
#define DEVICE_NAME "Betty Name"

DashDevice dashDevice(DEVICE_TYPE);
DashBluefruit_BLE  ble_con(&dashDevice, true);

int dialValue = 0;

void processStatus(ConnectionType connectionType) {
    String message((char *)0);
    message.reserve(1024);

    message = dashDevice.getKnobMessage("KB01", dialValue);
    message += dashDevice.getDialMessage("D01", dialValue);

    ble_con.sendMessage(message);
}

void processIncomingMessage(MessageData *messageData) {
    switch (messageData->control) {
    case status:
        processStatus(messageData->connectionType);
        break;
    case knob:
        if (messageData->idStr == "KB01") {
              dialValue = messageData->payloadStr.toFloat();
            String message = dashDevice.getDialMessage("D01", dialValue);
            ble_con.sendMessage(message);
        }
        break;
    }
}

void setup() {
    Serial.begin(115200);

    dashDevice.name = DEVICE_NAME;
    ble_con.setCallback(&processIncomingMessage);
    ble_con.begin(true);
}

void loop() {
    ble_con.run();
}
```

<p>This is just the beginning and there is a lot more we can do. Take a look at the examples in the library to see more details.</p>

<h2 id="toc_9">Layout Configuration</h2>

<p><strong>Layout configuration</strong> allows the IoT device to hold a copy of the complete layout of the device as it should appear on the <strong>Dash</strong> app. It includes all infomration for the device, controls (size, colour, style etc.), device views and connections.</p>

<p>When the <strong>Dash</strong> app discovers a new IoT device, it will download the Layout Configuration from the IoT device to display the device controls they way they have been designed by the developer. This is particularly useful developing commercial products and distributing your IoT devices to other users.</p>

<p>To include the Layout Configuration in your IoT device, simply follow these steps:</p>

<ul>
<li><strong>Design your layout</strong> in the <strong>Dash</strong> app and include all controls and connections that you need in your layout.</li>
<li><strong>Export the layout</strong>: Tap on the <strong>Device</strong> button, then tap the <strong>Export Layout</strong> button.</li>
<li><strong>Select the provisioning setup</strong> that you want (see below for provisioning details) and tap the <strong>Export</strong> button. The Layout Configuration will be emailed to you.</li>
<li><strong>Copy and paste</strong> the C64 configuration text from the email into your Arduino code, assigning it to a pointer to store the text in program memory. Your C64 configuration text will be different to that shown below.</li>
<li><strong>Add the pointer</strong> to the C64 configuration text (configC64Str) as a second attribute to the DashDevice object.</li>
<li><strong>Add the Layout Config Revision</strong> integer (CONFIG_REV) as a third attribute to the DashDevice object.</li>
</ul>

```
const char configC64Str[] PROGMEM =
"lVNNb9swDP0rhQ47CUPiLM2WW2S7bhEndh3V3TAMgxursRZH8mQ5Hy3y30f5I83W7lD4QpOPpPge+Yx84qPx9x8YTecBAesZVRVP"
"0RhNh2RbrarLW5HE9zNvE4ZeFa8QRqU+5AwA8yCaTXxwLKXQSuY3jskivb7BMJEGIj8EImI5S0rAa1UxjDbJHo37vR5GRaKY0HWS"
"E9dJiqVxkleAHUFcc123mQr5AMGM8VWmo0Rzica9j9ZliwhlycEnAEmD0LTO5G7Gxcw0anoeOswpe/gJviFGa6hty1wq8wjGiouQ"
"izXUSHmSX8k8l7uybt8WMu4O/o2ZMGB3PNXZWWWM9q/6WdbIGg1geg7v7B2B7YXvRA3v1P1KG8u5aV2zSdga7vyusfzAa4xJ7DTG"
"wrdpi7f91nDi+L7WsNOIBJHjRgsjklY5cHIFWi34E8QGwPFK8RQGqjaiRGPLMuSBKo2nG1pUmxOk/7fYrW6mNJEqZaoj5z7jmnWR"
"9UqkXYAYeRv/f7FUJaKst2N5ALZMyy45WRp1us1wQI6LDxftgph8aiJE7t9o14VeVTccRCA0MDI4g74Q1f+MEYex58nGdL0r0LEW"
"a+Kfn8uXXqx+E3JNPe/GyTLuFtNf+9uzcyGTCP4KxZa8rPd18A+ZNZfvvw/Dwnvuo95tnp/49xRjAr11Jf2h+eAlkgv9Iu5p82Ez"
"Hpg6a2C7c+pGb97ge66kElyDHAidH4x9HbVnQr0ovG5MQum8s7z2ZuwruJRnlLItX7IF01UBpQRoh3f8kWO9LH4WUmmcJmWGZ7eU"
"YuK7mNphM5FT58Wc7dp9f1xFbAvm8fgH";


DashDevice    dashDevice(DEVICE_TYPE, configC64Str, CONFIG_REV);
```

Here is our Knob and Dial example, with Layout Configuration added:


```
#include "DashioBluefruitSPI.h"

#define DEVICE_TYPE "BF_SPI_BLE_Type"
#define DEVICE_NAME "Betty Name"

const char configC64Str[] PROGMEM =
"lVNNb9swDP0rhQ47CUPiLM2WW2S7bhEndh3V3TAMgxursRZH8mQ5Hy3y30f5I83W7lD4QpOPpPge+Yx84qPx9x8YTecBAesZVRVP"
"0RhNh2RbrarLW5HE9zNvE4ZeFa8QRqU+5AwA8yCaTXxwLKXQSuY3jskivb7BMJEGIj8EImI5S0rAa1UxjDbJHo37vR5GRaKY0HWS"
"E9dJiqVxkleAHUFcc123mQr5AMGM8VWmo0Rzica9j9ZliwhlycEnAEmD0LTO5G7Gxcw0anoeOswpe/gJviFGa6hty1wq8wjGiouQ"
"izXUSHmSX8k8l7uybt8WMu4O/o2ZMGB3PNXZWWWM9q/6WdbIGg1geg7v7B2B7YXvRA3v1P1KG8u5aV2zSdga7vyusfzAa4xJ7DTG"
"wrdpi7f91nDi+L7WsNOIBJHjRgsjklY5cHIFWi34E8QGwPFK8RQGqjaiRGPLMuSBKo2nG1pUmxOk/7fYrW6mNJEqZaoj5z7jmnWR"
"9UqkXYAYeRv/f7FUJaKst2N5ALZMyy45WRp1us1wQI6LDxftgph8aiJE7t9o14VeVTccRCA0MDI4g74Q1f+MEYex58nGdL0r0LEW"
"a+Kfn8uXXqx+E3JNPe/GyTLuFtNf+9uzcyGTCP4KxZa8rPd18A+ZNZfvvw/Dwnvuo95tnp/49xRjAr11Jf2h+eAlkgv9Iu5p82Ez"
"Hpg6a2C7c+pGb97ge66kElyDHAidH4x9HbVnQr0ovG5MQum8s7z2ZuwruJRnlLItX7IF01UBpQRoh3f8kWO9LH4WUmmcJmWGZ7eU"
"YuK7mNphM5FT58Wc7dp9f1xFbAvm8fgH";

DashDevice dashDevice(DEVICE_TYPE, configC64Str, 1);
DashBluefruit_BLE  ble_con(&dashDevice, true);

int dialValue = 0;

void processStatus(ConnectionType connectionType) {
    String message((char *)0);
    message.reserve(1024);

    message = dashDevice.getKnobMessage("KB01", dialValue);
    message += dashDevice.getDialMessage("D01", dialValue);

    ble_con.sendMessage(message);
}

void processIncomingMessage(MessageData *messageData) {
    switch (messageData->control) {
    case status:
        processStatus(messageData->connectionType);
        break;
    case knob:
        if (messageData->idStr == "KB01") {
              dialValue = messageData->payloadStr.toFloat();
            String message = dashDevice.getDialMessage("D01", dialValue);
            ble_con.sendMessage(message);
        }
        break;
    }
}

void setup() {
    Serial.begin(115200);

    dashDevice.name = DEVICE_NAME;
    ble_con.setCallback(&processIncomingMessage);
    ble_con.begin(true);
}

void loop() {
    ble_con.run();
}
```

<h2 id="toc_10">Jump In and Build Your Own IoT Device</h1>

When you are ready to create your own IoT device, the Dash Arduino C++ Library will provide you with more details about what you need to know:

<a href="https://dashio.io/arduino-library/">https://dashio.io/arduino-library/</a>