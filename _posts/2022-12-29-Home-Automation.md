Back in late 2016, I started working on a hobby project with the goal to monitor and automate things at 🏠. I started with a [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi){:target="_blank"} and a [reed switch](https://en.wikipedia.org/wiki/Reed_switch){:target="_blank"}.

Over the years, I have expanded the project to include multiple door sensors, garage remote control, light control, security system, notification, system health monitoring. I'm listing the hardware and software components used in my project and will provide architecture and design overview of the entire system.

<hr/>

## TABLE OF CONTENTS:
1. [Hardware](#hardware)
2. [Software](#software)
3. [Architecture](#architecture)
4. [Wiring Drawing](#wiring-drawing)
5. [Screen Captures](#screen-captures)

<hr/>

#### HARDWARE:
- [Raspberry pi](https://www.adafruit.com/product/4295){:target="_blank"}
- [Ademco contact switch 7939WG](https://www.amazon.com/7939WG-WH-Ademco-Surface-Mount-Contacts/dp/B001DEUUZC/){:target="_blank"}
- [Honeywell glass break sensor](https://www.amazon.com/Honeywell-Ademco-ASC-SS1-Shock-Sensor/dp/B000GUV1W0){:target="_blank"}
- [TP Link HS100 Wifi switch](https://www.amazon.com/TP-Link-KIT-HS100-Wall-Light-Electronic-Component-switches/dp/B01KBFWW0O){:target="_blank"}
- [Honeywell wave 2 siren](https://www.amazon.com/Honeywell-WAVE-2-Two-Tone-Siren/dp/B0006BCCAE/){:target="_blank"}
- [Garage remote control](https://www.ebay.com/p/20024769511){:target="_blank"}
- [GPIO jumper wires](https://www.amazon.com/GenBasic-Piece-Female-Jumper-Wires/dp/B077N58HFK/){:target="_blank"}
- [Wire spool](https://www.adafruit.com/product/4734){:target="_blank"}  
[🔝](#table-of-contents)

<hr/>

#### SOFTWARE:
- [Pi4j](https://pi4j.com/){:target="_blank"}
- [Java](https://www.oracle.com/java/technologies/javase/jdk11-archive-downloads.html){:target="_blank"}
- [Cloud AMQP](https://www.cloudamqp.com/){:target="_blank"} Or [RabbitMQ](https://www.rabbitmq.com/#getstarted){:target="_blank"}
- [Tomcat](https://tomcat.apache.org/){:target="_blank"}
- [Grafana](https://grafana.com/grafana/download){:target="_blank"}
- [Prometheus](https://prometheus.io/){:target="_blank"}
- [Prometheus JMX exporter](https://github.com/prometheus/jmx_exporter){:target="_blank"}
- [Telegram Bot Library](https://github.com/rubenlagus/TelegramBots){:target="_blank"}
- [otel](https://opentelemetry.io/docs/instrumentation/java/automatic/){:target="_blank"} with [Grafana Tempo](https://grafana.com/docs/tempo/latest/){:target="_blank"} 🚧  
[🔝](#table-of-contents)

<hr/>

#### ARCHITECTURE:
![Architecture](https://raw.githubusercontent.com/gmrock/website/main/media/HomeAutomation_Architecture_Diagram.png)  
[🔝](#table-of-contents)

##### `MasterRaspberryPi`: 
This pi is hosting the main web application(war) on a tomcat server. This is the central system(brain) to which all the peripheral devices such as sensors, remotes etc send status information (via other locally located raspberry pis). The web application makes the decision based on the input it receives from the devices. For example - if the main door is opened and the current time is 23:00 and the home is armed, then the siren needs to start and send notifications to users. This also provides a simple UI which lists all controls for the output devices (lights, sirens, garage remote). There are configuration pages available that allow end users to arm/dis-arm home, select time slots, intelligent control etc. There are some advanced configurations that also allow the user to set the IP address of other raspberry pis.  
[🔝](#table-of-contents)

<br/>

##### `RpiS2GlassDoorSensor`:
This is a running a java application (jar) which monitors the state of the reed switches installed on doors. I'm also using the reed switches to check on the position of the door knob. Whenever there is any state change the application will relay the state to a cloud hosted rabbitmq (cloudamqp). The action to be executed on the state change is handled by _`MasterRaspberryPi`_. I'm using this library for interacting with the GPIO pins on the raspberry pi - [Pi4j](https://pi4j.com/){:target="_blank"}  
[🔝](#table-of-contents)

<br/>

##### `RpiS3ShutterGarageSiren` & `RpiS4HallSirenGarageRemote`:
Similar to the _`RpiS2GlassDoorSensor`_ this is running a java application (jar) that monitors the reed switch. This application has an additional functionality which is to control the siren. The _`MasterRaspberryPi`_ communicates with these applications to control the state of the siren. The communication between _`MasterRaspberryPi`_ and reed switches, siren is done via rabbitmq (cloudamqp). _`RpiS4HallSirenGarageRemote`_ also has Garage remote control wired up which is again controlled via the rabbitmq.  
[🔝](#table-of-contents)

<br/>

##### `cloudamqp`:
The communication between the _`MasterRaspberryPi`_ and other peripheral applications is done over rabbitMq. I'm using the cloud hosted solution - cloudamqp.  
[🔝](#table-of-contents)

<br/>

##### `telegram`:
I have developed a telegram bot that is running on _`MasterRaspberryPi`_. The bot is listening for commands from authenticated end users. These users use the telegram app to send the commands. Based on the command received, my code will perform the necessary steps. Telegram bot is also used as notification system to provide up to date notifications. For example - whenever any door is opened/closed, siren started etc.  
[🔝](#table-of-contents)

<br/>

##### `Prometheus`:
I'm running a local instance of prometheus on one of the raspberry pi. All the individual java applications(including tomcat instance) are instrumented with Prometheus JMX exporter (-javaagent) which publishes all the JMX related metrics to their respective localhost on a specific port. The prometheus is configured to scrape for JMX metrics from all the 5 raspberry pis. Prometheus is configured to publish the metrics collected to a cloud instance of Grafana (the free tier is sufficient for my purpose).  
[🔝](#table-of-contents)

<br/>

##### `Grafana`:
I have configured a dashboard which gives an overview of the health of the entire system. I'm tracking the heap usage, thread counts, temperature, errors (using [loki](https://grafana.com/docs/loki/latest/clients/promtail/){:target="_blank"}), up time for each of the applications in my system. I have also configured alerts (to be sent to my telegram app) when there is any deviation from the baseline such as - application is down, heap usage or thread count is growing etc.  
[🔝](#table-of-contents)

<br/>

##### `otel with Grafana Tempo`:
I'm running the [otel collector](https://opentelemetry.io/docs/collector/getting-started/){:target="_blank"} on the _`MasterRaspberryPi`_. The tomcat web application is instrumented with [otel javaagent](https://opentelemetry.io/docs/instrumentation/java/automatic/){:target="_blank"} and the traces are collected by the [otel collector](https://opentelemetry.io/docs/collector/getting-started/){:target="_blank"}. The collected metrics are published to Grafana Tempo.  
[🔝](#table-of-contents)

<hr/>

#### WIRING DRAWING:
Below are the wiring drawings to connect raspberry pi GPIO to siren, reed switch and garage opener
![Wiring drawing](https://raw.githubusercontent.com/gmrock/website/main/media/Wiring_Drawings.png)  
[🔝](#table-of-contents)

<hr/>

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
[🔝](#table-of-contents)

<br/>

##### `Grafana`:
Dashboard showing various widgets for monitoring (including otel traces from _`MasterRaspberryPi`_)
![Dashboard](https://raw.githubusercontent.com/gmrock/website/main/media/grafana.png)
Heap usage and temperature monitoring
![Heap usage, temperature](https://raw.githubusercontent.com/gmrock/website/main/media/grafana_1.png)  
[🔝](#table-of-contents)

<br/>

##### `Telegram Notification`:
Telegram notifications about door sensors
<br/> <img src="https://raw.githubusercontent.com/gmrock/website/main/media/A8B57369-8EE1-43BF-BEE6-ED2A2B7BBE31.jpeg" alt="Telegram app notification" style="width:200px;"/>
<br/>Telegram message to send command signals
<br/> <img src="https://raw.githubusercontent.com/gmrock/website/main/media/5BCCB732-34F5-4BF8-8261-B5CBF5AC5724.jpeg" alt="Telegram app status and control" style="width:200px;"/>  
[🔝](#table-of-contents)

<script src="https://utteranc.es/client.js" repo="gmrock/gmrock.github.io" issue-term="pathname" label="Comments" theme="github-light" crossorigin="anonymous" async> </script> 
