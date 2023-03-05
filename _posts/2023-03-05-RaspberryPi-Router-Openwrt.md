This project started with the need for having an view into how much bandwidth each device on the home network is consuming. I currently have an old Netgear modem-router combo that doesn't support custom firmwares and it doesn't provide the insights that I'm looking for out of the box. 

I tried finding for other add-on applications which can be used for my purpose and was blown away with the capabilities [openwrt](https://openwrt.org/){:target="_blank"} provides. However, the initial problem still remained - it wasn't compatible with my existing modem-router and I didn't want to upgrade. [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi){:target="_blank"} came to the rescue. 
Currently for my home network I have a [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi){:target="_blank"} acting as router running openwrt (architecture below). There is no compormise in the internet speed with [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi){:target="_blank"} running [openwrt](https://openwrt.org/){:target="_blank"}. 
I have listed the hardware, sofware, my current setup along with details on how to configure [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi){:target="_blank"} as a router running openwrt. 



#### HARDWARE:
- [Raspberry pi 4B](https://www.adafruit.com/product/4295){:target="_blank"}
- [TP-Link USB-Ethernet-Adapter-Gigabit-Switch](https://www.amazon.com/USB-Ethernet-Adapter-Gigabit-Switch/dp/B09GRL3VCN){:target="_blank"}
- [MicroSD card](){:targe="_blank"}
- Existing modem
- Access point (any) - only if you don't want to use the raspberry pi's onboard wifi (which is not powerful enough to act like access point)
- Cat 6 ethernet cables (cat5 can give up to maximum of 100Mbps)

<hr/>

#### SOFTWARE:
- [openwrt](https://firmware-selector.openwrt.org/){:target="_blank"}
- [etcher](https://etcher.download){:target="_blank"} - any software which can flash storage with an image

<hr/>

#### ARCHITECTURE:
![Architecture](https://raw.githubusercontent.com/gmrock/website/main/media/HomeAutomation_Architecture_Diagram.png)

<hr/>

#### STEPS:

Step 1: Navigate to [openwrt firmware selector page](https://firmware-selector.openwrt.org/){:target="_blank"} and search for `Raspberry Pi`. Choose the model which you will be using. 

Step 2: Before downloading the firmware, we will add few more packages that we want to be included in the firmware (this can be done later too after the entire setup). The additional package is the driver for [TP-Link USB-Ethernet-Adapter-Gigabit-Switch](https://www.amazon.com/USB-Ethernet-Adapter-Gigabit-Switch/dp/B09GRL3VCN){:target="_blank"}. We will using this as the 2nd LAN port for [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi){:target="_blank"} (as it has only 1 LAN port on the board). Expand `Customize installed packages and/or first boot script` and under `Installed Packages` add the below line towards the end.

```
kmod-mii kmod-crypto-sha256 kmod-usb-net-cdc-ether kmod-usb-net-cdc-ncm kmod-usb-net kmod-usb-net-asix-ax88179 luci
```
![customize firmware](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/step2.png)

Step 3: After adding the additional package, click on `REQUEST BUILD` (it takes few seconds to get custom build ready). Now, download the firmware by choosing `FACTORY (SQUASHFS)` option. This will download *.img.gz file. Unzip the file, and flash the image to the microsd card using [etcher](https://etcher.download){:target="_blank"}. Insert the microsd card in [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi){:target="_blank"}, connect the external LAN dongle to USB3 port and power it ON.
![customize firmware](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/Step3.png)


Step 4: After turning on the [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi){:target="_blank"}, connect an ethernet cable to your laptop and the other end to [Raspberry Pi's](https://en.wikipedia.org/wiki/Raspberry_Pi){:target="_blank"} onboard LAN port.

Step 5: Disconnect WIFI on your laptop (as we will connecting to the LAN connected [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi){:target="_blank"}). We will SSH into the [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi){:target="_blank"} which is now running [openwrt](https://firmware-selector.openwrt.org/){:target="_blank"}. Open up a terminal(console) and enter the below command (default username: `root`, default password is empty and default IP address is `192.168.1.1`).

```
ssh root@192.168.1.1
```
![ssh openwrt](https://raw.githubusercontent.com/gmrock/website/main/media/Wiring_Drawings.png)


Step 6: Add a password by running the below command
```
passwd
```
![password change](https://raw.githubusercontent.com/gmrock/website/main/media/Wiring_Drawings.png)



Step 7: We will turn off DHCP for lan interface and remove unwanted parameters. This is done by running the below commands and making the following changes to - `/etc/config/dhcp` file.

```
vi /etc/config/dhcp
```

**Add** the below line under `config dhcp 'lan'` (this will turn off DHCP for lan interface)

```
option ignore '1'
```

**Remove** below lines under `config dhcp 'lan'` (this is not needed)

```
option start '100'
option limit '150'
```

This is how the final file will look like:
```

```
Save and close the file.


Step 8: We will change the default IP address to 192.168.0.111 (from 192.168.1.1). I'm doing this because my current modem-router's IP address is 192.168.0.1. This way I will be able to access the openwrt from my current network for configuration. We also specify the DNS and home gateway (current modem-router's) IP address. This is done by running the below commands and making the following changes to - `/etc/config/network` file.

```
vi /etc/config/network
``` 

Change the `option ipaddr` under `config interface 'lan'` to the new static IP address (default - 192.168.1.1)

```
option ipaddr '192.168.0.111'
```

Add below lines to specify the DNS(using google's DNS) and home gateway (current modem-router's) IP address.

```
option dns '8.8.8.8'
option gateway '192.168.0.1'
```

This is how the final file will look like:
```

```

Save and close the file and reboot using below command

```
reboot
```

Step 9: Now unplug the LAN cable from your laptop and plug that to home modem-router's LAN port. So the connection will be from home modem-router to raspberry pi's onboard LAN port which is running openwrt.

Step 10: Turn on the WIFI on the laptop and open the below address in the browser. This should open up the openwrt's UI (use root as username and the password which was configured in step 6 above).

```
http://192.168.0.111
```

Step 11: Now we will add a new interface for the external ethernet dongle which we are using. For that navigate to:

```
Network > Interfaces > Add new interface
```
![Interface page](https://raw.githubusercontent.com/gmrock/website/main/media/Wiring_Drawings.png)

Name the interface `wan` and protocol as `DHCP client` and device as `eth1`
![new interface config](https://raw.githubusercontent.com/gmrock/website/main/media/Wiring_Drawings.png)


In the firewall settings change it to `wan`
![wan firewall](https://raw.githubusercontent.com/gmrock/website/main/media/Wiring_Drawings.png)

Save the interface and save and apply. So the external LAN dongle is WAN i.e. it will be used to connect to modem-router and the internal LAN port will be used to connect to access point or switch or to computer to access internet.

Step 12: Now remove the LAN cable from the raspberry pi's onboard LAN port and connect that to external USB LAN dongle. So the connection will be - From modem-router TO external USB LAN dongle that is connected to Raspberry pi. We did this because, in Step 11 above we configured the external USB LAN dongle as wan (Wide Area Newtwork). wan needs to be connected to the modem-router/gateway.

Step 13: Now we should see both the interfaces will have IP address
![IP address for both interface](https://raw.githubusercontent.com/gmrock/website/main/media/Wiring_Drawings.png)

Step 14: Now we will revert some of the changes we did in Step 7. 

Remove below from `/etc/config/dhcp` file. 
```
option ignore '1'
``` 

Remove below from `/etc/config/network` file (these were added under lan, these are needed under wan)
```
option dns '8.8.8.8'
option gateway '192.168.0.1'
```

Step 14: Connect LAN cable from onboard Raspberry pi's LAN port to access point.

Step 15: All your devices will be connected automatically because we didn't do any changes to access point


Step 16: There are a lot of options(packages) that can be installed to analyze the bandwidth usage. Below are some of the packages which I found very useful
- [Bandwidthd](https://openwrt.org/docs/guide-user/services/network_monitoring/bandwidthd){:target="_blank"}
- [Netlink Bandwidth Monitor](https://openwrt.org/docs/guide-user/services/network_monitoring/bwmon){:target="_blank"}
- [Complete list](https://openwrt.org/docs/guide-user/services/network_monitoring/start){:target="_blank"}

Step 17: These can be installed from the software manager from within openwrt. Follow below steps to install [Bandwidthd](https://openwrt.org/docs/guide-user/services/network_monitoring/bandwidthd){:target="_blank"}:
- go to openwrt UI
- System > Software and click on `Update lists...`
- afer the list has been updated, search for package named `bandwidthd-sqlite` and install it
- during installation if there are some errors, it will be due to missing dependent packages. Search for the package which caused the installation to fail and search for that package in the similar way done above and install them. Then again try installing `bandwidthd-sqlite`
![installting bandwidthd](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/bandwidthd.png)
- After installation, the UI for  bandwidthd is accessible here - `http://192.168.0.111/bandwidthd`(this is the IP address which was set in Step 8)
![bandwidthd UI](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/bandwidth_ui.png)

- Similarly to install [Netlink Bandwidth Monitor](https://openwrt.org/docs/guide-user/services/network_monitoring/bwmon){:target="_blank"}, search for the package named `luci-app-nlbwmon` and install it. After installation this will be visible under Services > Bandwidth Monitor
![bandwidth monitor UI](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/bandwidthMonitor.png)


#### Other useful packages (can be installed the same way we installed the above packages):
- [Adblock](https://openwrt.org/docs/guide-user/services/ad-blocking){:target="_blank"} - this is used for blocking ads. I was using pihole for a long time and recently switched to adblock which can be bundled with openwrt. You will need to enable reporting after installing the package to view reports (you will see the options after adblock is installed on the UI).
- [SQM QOS](https://openwrt.org/docs/guide-user/network/traffic-shaping/sqm){:target="_blank"} - this is used for prioritzing traffic and help reduce bufferbloat. 


More about bufferbloat: 
Bufferbloat is a software issue with networking equipment that causes spikes in your Internet connection’s latency when a device on the network uploads or downloads files. Bufferbloat can make web browsing slower, make video calls stutter, and cause VoIP calls will break up. Real-time games will lag. Bufferbloat causes degraded connectivity anytime your Internet connection is under heavy use by any user or application. If a large upload or download of data is happening, other applications and users will slow down.
[source](https://www.waveform.com/tools/bufferbloat){:target="_blank"}

On this [source](https://www.waveform.com/tools/bufferbloat){:target="_blank"}, there is a very good explanation of bufferbloat (snippet below):
Imagine that you have a sink with a very narrow drain. Your friend pours a large bucket of water into it. The sink is full and draining very slowly.
You have a spoonful of oil that you want to get down the drain as soon as possible. You pour it into the sink. But of course, it doesn’t drain quickly - the water in the sink needs to drain first. Shoot!
The sink is a buffer - a “queue” for liquid. When your friend poured in the bucket of water, they filled the queue. Anything new will take a long time to drain.

Instead of water and oil, networks have different flows of packets. Your router is like the sink, and your connection to the ISP is the narrow drain (since it’s probably the slowest link in your network).
When someone on your network sends a large file, a lot of packets get sent all at once. The router temporarily "buffers those packets", holding them before they’re sent. Any new data packets get stuck behind the existing queue of buffered packets. They will arrive at the destination much later than if the router’s buffers hadn't been full.

Certain routers have smart algorithms (usually called "SQM") that ensure that time-sensitive packets flowing through the router don’t get delayed, even when large files are being downloaded or uploaded. Continuing with the liquid analogy, these routers offer a way to admit just the right amount of water into the sink so the drain pipe is always full, but a new spoonful of oil will drain out immediately.

Please visit [the source](https://www.waveform.com/tools/bufferbloat){:target="_blank"} to read more about bufferbloat and also do a bufferbloat test on your network.

<br/>

##### `RpiS2GlassDoorSensor`:
This is a running a java application (jar) which monitors the state of the reed switches installed on doors. I'm also using the reed switches to check on the position of the door knob. Whenever there is any state change the application will relay the state to a cloud hosted rabbitmq (cloudamqp). The action to be executed on the state change is handled by _`MasterRaspberryPi`_. I'm using this library for interacting with the GPIO pins on the raspberry pi - [Pi4j](https://pi4j.com/){:target="_blank"} 

<br/>



<script src="https://utteranc.es/client.js" repo="gmrock/gmrock.github.io" issue-term="pathname" label="Comments" theme="github-light" crossorigin="anonymous" async> </script> 
