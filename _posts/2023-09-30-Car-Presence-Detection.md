In this post, I'll be sharing the project which I developed to detect if the parking spot(garage) is occupied using ultrasonic sensor. The [Home Automation](https://gmrock.github.io/2022/12/29/Home-Automation.html){:target="_blank"} project that I have developed is in java and running on [Raspberry pi](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/){:target="_blank"}. However, for this project I'm using [ESP32](https://en.wikipedia.org/wiki/ESP32){:target="_blank"} microcontroller. For developing this application I had few options (Java is never an option on microcontrollers for various reasons - memory constraints, large JVM foot print etc):
- [ESPHome](https://esphome.io/){:target="_blank"} which generates c++ compiled code based on yaml configuration file
- [micropython](https://micropython.org/){:target="_blank"}
- [Arduino Sketch](https://www.arduino.cc/en/software){:target="_blank"} (which is bascially c++) 

I decided to go with [Arduino Sketch](https://www.arduino.cc/en/software){:target="_blank"} as that would give me the freedom to develop based on my requirements (unlike yaml configuration files). That also meant, I would get to dust off my c++ coding skills ;)

<hr/>

## TABLE OF CONTENTS:
1. [Hardware](#hardware)
2. [Software](#software)
3. [Architecture](#architecture)
4. [Problems Encountered](#problems-encountered)
5. [Screen Captures](#screen-captures)

<hr/>

#### HARDWARE:
- [ESP32](https://www.amazon.com/DORHEA-Development-Microcontroller-NodeMCU-32S-ESP-WROOM-32/dp/B086MJGFVV/){:target="_blank"} or any other microcontroller can also be used such as [Raspberry pico](https://www.raspberrypi.com/products/raspberry-pi-pico/){:target="_blank"}
- [HCSR04 Ultrasonic sensor](https://www.adafruit.com/product/3942){:target="_blank"}
- [USB C to micro USB data cable](https://www.amazon.com/AGVEE-Braided-Charger-Charging-Controller/dp/B094RDLNDB/){:target="_blank"} -for uploading code to microcontroller
- [5V power supply micro usb](https://www.adafruit.com/product/1995){:target="_blank"}
- [GPIO jumper wires](https://www.amazon.com/GenBasic-Piece-Female-Jumper-Wires/dp/B077N58HFK/){:target="_blank"}
- [Wire spool](https://www.adafruit.com/product/4734){:target="_blank"}  
[🔝](#table-of-contents)

<hr/>

#### SOFTWARE:
- [Arduino Sketch IDE](https://www.arduino.cc/en/software){:target="_blank"}  
     - Libraries used in the project:  
        - [PubSubClient](https://www.arduino.cc/reference/en/libraries/pubsubclient/){:target="_blank"}  
        - [WiFiClientSecure](https://github.com/espressif/arduino-esp32/tree/master/libraries/WiFiClientSecure){:target="_blank"}  
        - [Syslog](https://github.com/arcao/Syslog){:target="_blank"}
- [HiveMQ Cloud](https://console.hivemq.cloud/){:target="_blank"}
[🔝](#table-of-contents)

<hr/>

#### ARCHITECTURE:

![Architecture](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/car_presence_architecture.png)  

The ultrasonic sensor sends an ultrasonic pulse-out (trigger) which travels through the air and if there is an obstacle, it bounces back to the sensor. By calculating the travel time and the speed of sound, the distance can be calculated.  
I'm making use of this to determine if there is car parked in the garage. I had to find out the distance between the sensor and the car to determine if the car is in the garage. For example - if the distance calculated is greater than 5ft, I know the car is not parked in the garage (and if less than 5ft, the car is parked). I have created a dead zone of 0.4feet to filter out erroneous readings (which is common if the object is not flat).  
The code keeps sending pulse out/trigger every milliseconds and computes the distance. If the distance measured is current distance+-0.4ft, the actual measured distance is sent to the MQTT broker. I'm using [HiveMQ Cloud](https://console.hivemq.cloud/){:target="_blank"} as the MQTT broker and using [PubSubClient](https://www.arduino.cc/reference/en/libraries/pubsubclient/){:target="_blank"} c++/arduino library to make the calls to MQTT broker.  
I'm using [home assistant](https://www.home-assistant.io/){:target="_blank"} to take further action based on this message. Example - send a notification if a car has arrived or departed etc.  
[🔝](#table-of-contents)

<hr/>

#### PROBLEMS ENCOUNTERED:  
1. **MQTT secure encrypted connection to HiveMQ:** I started this project using locally hosted MQTT broker named [Mosquitto MQTT](https://mosquitto.org/){:target="_blank"}. However, because of the below issue, I switched to cloud based MQTT broker. Since the broker was hosted externally (cloud), the connection had to be encrypted.  
For creating encrypted secure connection I had to create the Wifi object using [WiFiClientSecure](https://github.com/espressif/arduino-esp32/tree/master/libraries/WiFiClientSecure){:target="_blank"} and pass the certificate. Without this the broker was rejecting the connection.

2. **Randomly stopped receiving metric from sensor/microcontroller:** Randomly I would stop getting metrics about the distance between sensor and car. Since memory and processing is very limited on microcontrollers, intitially I didn't have any logging. However, later I figured a way to log to [rsyslog](https://www.rsyslog.com/){:target="_blank"} server (will be publishing more details on my setup which makes use of rsyslog and grafana loki). With the help of logging, I was able to fing out that the issue was with the microcontroller/host unable to reach (even couldn't ping) the locally hosted MQTT broker.
I'm using Orbi Mesh network at my home and have seen this behavior where randonly devices on the same home network can no longer ping each other. However, they are still up and running and can ping hosts on the internet. This is the reason I moved to cloud based MQTT broker.  
[🔝](#table-of-contents)

<hr/>

#### SCREEN CAPTURES:  

![installation](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/car_presencescreen_capture_1.png)  
I have used an empty cardboard box(green color above) to mount the ultrasonic sensor along with the microcontroller which runs the application code.  

![internal](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/car_presence_inside_2.jpg)  
This is the inside of the cardboad box, it has the microcontroller and the ultrasonic sensor (hcsr04). The reason there are more wires is because, I have one more hcsro4 sensor that is also connected to this microcontroller. So I'm tracking the presence of 2 cars using 2 hcsr04 sensors and 1 esp32 (microcontroller).  
[🔝](#table-of-contents)
<br/>


<script src="https://utteranc.es/client.js" repo="gmrock/gmrock.github.io" issue-term="pathname" label="Comments" theme="github-light" crossorigin="anonymous" async> </script> 
