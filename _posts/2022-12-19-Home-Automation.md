Back in late 2016, I started working on a hobby project with the goal to monitor and automate things at home. I started with a [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi) and a [reed switch](https://en.wikipedia.org/wiki/Reed_switch).

Over the years, I have expanded the project to include multiple doors, garage remote control, lights, security system, notification, system health monitoring. Let me start by listing the hardware and software components used in my project and provide architecture and design overview of the entire system.

#### HARDWARE:
- [Raspberry pi](https://www.adafruit.com/product/4295){:target="_blank"}
- [Ademco contact switch 7939WG](https://www.amazon.com/7939WG-WH-Ademco-Surface-Mount-Contacts/dp/B001DEUUZC/){:target="_blank"}
- [Honeywell glass break sensor](https://www.amazon.com/Honeywell-Ademco-ASC-SS1-Shock-Sensor/dp/B000GUV1W0){:target="_blank"}
- [TP Link HS100 Wifi switch](https://www.amazon.com/TP-Link-KIT-HS100-Wall-Light-Electronic-Component-switches/dp/B01KBFWW0O){:target="_blank"}
- [Honeywell wave 2 siren](https://www.amazon.com/Honeywell-WAVE-2-Two-Tone-Siren/dp/B0006BCCAE/){:target="_blank"}
- [Garage remote control](https://www.ebay.com/p/20024769511){:target="_blank"}
- [GPIO jumper wires](https://www.amazon.com/GenBasic-Piece-Female-Jumper-Wires/dp/B077N58HFK/){:target="_blank"}
- [Wire spool](https://www.adafruit.com/product/4734){:target="_blank"}

---

#### SOFTWARE:
- [Pi4j](https://pi4j.com/){:target="_blank"}
- [Java](https://www.oracle.com/java/technologies/javase/jdk11-archive-downloads.html){:target="_blank"}
- [Cloud AMQP](https://www.cloudamqp.com/){:target="_blank"} Or [RabbitMQ](https://www.rabbitmq.com/#getstarted){:target="_blank"}
- [Tomcat](https://tomcat.apache.org/){:target="_blank"}
- [Grafana](https://grafana.com/grafana/download){:target="_blank"}
- [Prometheus](https://prometheus.io/){:target="_blank"}
- [Prometheus JMX exporter](https://github.com/prometheus/jmx_exporter){:target="_blank"}
- [Telegram Bot Library](https://github.com/rubenlagus/TelegramBots){:target="_blank"}
- [otel](https://opentelemetry.io/docs/instrumentation/java/automatic/){:target="_blank"} with [Grafana Tempo](https://grafana.com/docs/tempo/latest/){:target="_blank"} :construction:

---

#### ARCHITECTURE:
![Architecture](https://raw.githubusercontent.com/gmrock/website/main/media/HomeAutomation_Architecture_Diagram.png)

##### `MasterRaspberryPi`: 
The main web application(war) is running on a tomcat server on this pi. This is the central system(brain) to which all the peripheral devices such as sensors, remotes etc send status information (via other locally located raspberry pis). The web application makes the decision based on the input it receives from the devices. For example - if the main door is opened and the current time is 23:00 and the home is armed, then the siren needs start along with sending notifications to users. This also provides a simple UI which lists all controls for the output devices (lights, sirens, garage remote). There are configuration pages available that allows end user to arm/dis-arm home, select time ranges, wifi control etc. There are some advanced configurations that allows to set the IP address of other raspberry pis.

##### `RpiS2GlassDoorSensor`:
This is a running a java application (jar) which monitors the state of the reed switches for couple of doors and knob. Whenever there is any state change the application will relay the state to a cloud hosted rabbitmq (cloudamqp). The decision and the action what needs to be done on the state change is handled by _`MasterRaspberryPi`_. I'm using this library for interacting with the GPIO pins on the raspberry pi - [Pi4j](https://pi4j.com/){:target="_blank"} 

##### `RpiS3ShutterGarageSiren` & `RpiS4HallSirenGarageRemote`:
Similar to the _`RpiS2GlassDoorSensor`_ this is running a java application (jar) that monitors the reed switch along with controlling the siren. The action what triggers the siren and other related activities is decided by the _`MasterRaspberryPi`_. The communication between _`MasterRaspberryPi`_ and reed switches, siren is done via rabbitmq (cloudamqp). _`RpiS4HallSirenGarageRemote`_ also has Garage remote control wired up which is again controller via the rabbitmq.

##### `cloudamqp`:
The communication between the _`MasterRaspberryPi`_ and other peripheral applications is done over rabbitMq. I'm using the cloud hosted solution - cloudamqp.

##### `telegram`:
I have developed a telegram bot that is running on _`MasterRaspberryPi`_. This bot is used to get commands from end user over telegram app and based on the command control the associated applications. The notifications, are also sent to end users as telegram messages. For example - whenever any door is opened/closed, siren started etc.

##### `Prometheus`:
I'm running a local instance of prometheus on one of the raspberry pi. All the individual java applications(including tomcat instance) are instrumented with Prometheus JMX exporter (-javaagent) which publishes all the JMX related metrics to their respective localhost on a specific port. The prometheus is configured to scrape for JMX metrics from all the 5 raspberry pis. Prometheus is configured to publish the metrics collected to a cloud instance of Grafana (the free tier is sufficient for my purpose).

##### `Grafana`:
I have configured a dashboard which gives an overview on the health of the entire system. I'm tracking the heap usage, thread counts, temperature, errors (using [loki](https://grafana.com/docs/loki/latest/clients/promtail/){:target="_blank"}), up time for each of the applications in my system. I have also configured alerts (to be sent to my telegram app) when there is any deviation from the baseline such as - application is down, heap usage or thread count is growing etc.

##### `otel with Grafana Tempo`:
I'm running the [otel collector](https://opentelemetry.io/docs/collector/getting-started/){:target="_blank"} on the _`MasterRaspberryPi`_. The tomcat web application is instrumented with [otel javaagent](https://opentelemetry.io/docs/instrumentation/java/automatic/){:target="_blank"} and the traces are collected by the [otel collector](https://opentelemetry.io/docs/collector/getting-started/){:target="_blank"} and publishes the metrics to Grafana Tempo.

---

#### WIRING DRAWING:
Below are the wiring drawings to connect raspberry pi GPIO to siren, reed switch and garage remote wiring
![Wiring drawing](https://raw.githubusercontent.com/gmrock/website/main/media/Wiring_Drawings.png)

---

#### SCREEN CAPTURES:

##### `Webapplication UI`:
Login page of the web application
![Dashboard](https://raw.githubusercontent.com/gmrock/website/main/media/login.png)
<br/>Main page
![Dashboard](https://raw.githubusercontent.com/gmrock/website/main/media/homepage.png)
<br/>Configuration page
![Dashboard](https://raw.githubusercontent.com/gmrock/website/main/media/config.png)
<br/>Internal configuration page
![Dashboard](https://raw.githubusercontent.com/gmrock/website/main/media/internal_config.png)
<br/>
##### `Grafana`:
Dashboard showing various widgets for monitoring (including otel traces from _`MasterRaspberryPi`_)
![Dashboard](https://raw.githubusercontent.com/gmrock/website/main/media/grafana.png)
Heap usage and temperature monitoring
![Heap usage, temperature](https://raw.githubusercontent.com/gmrock/website/main/media/grafana_1.png)
<br/>
##### `Telegram Notification`:
Telegram notifications about door sensors
<br/> <img src="https://raw.githubusercontent.com/gmrock/website/main/media/A8B57369-8EE1-43BF-BEE6-ED2A2B7BBE31.jpeg" alt="Telegram app notification" style="width:200px;"/>
<br/>Telegram message to send command signals
<br/> <img src="https://raw.githubusercontent.com/gmrock/website/main/media/5BCCB732-34F5-4BF8-8261-B5CBF5AC5724.jpeg" alt="Telegram app status and control" style="width:200px;"/>
