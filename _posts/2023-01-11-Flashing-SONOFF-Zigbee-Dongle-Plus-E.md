I have compiled the details for flashing [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"} with router firmware on Linux 🐧 and MAC  OS.

### Why flash firmware:
- upgrade
- alternate (opensource) option
- convert a Zigbee coordinator into a Zigbee router (this was my use case)


Apparently, the [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"} acts as a very good router (I have started using it as router and so far it's reliable).

<br/>
#### Step 1:
Download the router firmware
* [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"} **router** [firmware](https://github.com/itead/Sonoff_Zigbee_Dongle_Firmware/tree/master/Dongle-E/Router){:target="_blank"}
* [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"} **coordinator** [firmware](https://github.com/itead/Sonoff_Zigbee_Dongle_Firmware/tree/master/Dongle-E/NCP){:target="_blank"} (incase we want to reflash the coordinator firmware)

<br/>
#### Step 2:
🪛 Unscrew [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"}'s case. We will only need to unscrew the 2 screws that are on the side of the USB port. This is to get access to the boot button on the board.

![boot](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/C51079D8-DC05-4C04-B209-061AA596CF41.jpeg)

<br/>
#### Step 3:
Download and install [minicom](https://packages.debian.org/sid/minicom){:target="_blank"}. There are other tools that can also be used such as
[coolterm](https://freeware.the-meiers.org/). If using `coolterm` execute the steps detailed below under `For MAC OS`. 

**For Linux OS:**
* run the below command:
```
sudo apt-get install minicom
```
[minicom](https://packages.debian.org/sid/minicom){:target="_blank"} has a dependency on [lrzsz](https://www.ohse.de/uwe/software/lrzsz.html){:target="_blank"}. `lrzsz` is needed for XMODEM,YMODEM and ZMODEM communication. On linux platform, when minicom is installed using above command, this dependency is automatically installed. However, on MAC OS, this dependency is not installed with minicom. Follow the below steps to install minicom and the dependency on MAC OS.

**For MAC OS:**
* run the below command to install minicom:
```
brew install minicom
```
* run the below command to install the dependency [lrzsz](https://www.ohse.de/uwe/software/lrzsz.html){:target="_blank"}:
```
brew install lrzsz
```
* minicom internally uses `sx` for XMODEM communication (this can be changed in Step 7 `File transfer protocols` configuration page to make it use `lsx`. If we decide to do so, we can skip below steps and jump to Step 4). If we take a look at the installation's `bin` directory of `lrzsz` there is no `sx` executable.
There is `lsx` executable which is same as `sx`. 

```
ganesh@magal:/usr/local/Cellar/lrzsz/0.12.20_1/bin$ ls
lrb	lrx	lrz	lsb	lsx	lsz	rz	sz
```
`/usr/local/Cellar/lrzsz/0.12.20_1/bin` is the location where homebrew installed `lrzsz`.
* we will create a symlink called `sx` and link it to `lsx` which is already inside the `bin` directory.

```
ln -s /usr/local/Cellar/lrzsz/0.12.20_1/bin/lsx /usr/local/Cellar/lrzsz/0.12.20_1/bin/sx

```
* should see like this now:
```
gmagal@magal:/usr/local/Cellar/lrzsz/0.12.20_1/bin$ ls
lrb	lrx	lrz	lsb	lsx	lsz	rz	sx	sz
```
* one last thing that needs to be done is to add `sx` to PATH. If using zsh as shell, create `~/.zshrc` if not already present:
```
gmagal@GMAGAL-M-M1D2:$ nano ~/.zshrc
```
* add below line to the `/.zshrc` file:
```
export PATH=/usr/local/Cellar/lrzsz/0.12.20_1/bin:$PATH
```
* reopen terminal and type `sx` see if the command was found. If `sx` command is not found, the PATH has not been set in the correct file (belowsnippet shows `sx` command was found):
```
gmagal@magal:~$ sx
sx: need at least one file to send
Try `sx --help' for more information.
```

<br/>
#### Step 4:
Run the below command and make a note of all the devices that show up:
```
ls /dev/tty*
```
Sample output:
```
gmagal@masterrpi:~ $ ls /dev/tty*
/dev/tty    /dev/tty11  /dev/tty15  /dev/tty19  /dev/tty22  /dev/tty26  /dev/tty3   /dev/tty33  /dev/tty37  /dev/tty40  /dev/tty44  /dev/tty48  /dev/tty51  /dev/tty55  /dev/tty59  /dev/tty62  /dev/tty9
/dev/tty0   /dev/tty12  /dev/tty16  /dev/tty2   /dev/tty23  /dev/tty27  /dev/tty30  /dev/tty34  /dev/tty38  /dev/tty41  /dev/tty45  /dev/tty49  /dev/tty52  /dev/tty56  /dev/tty6   /dev/tty63  /dev/ttyAMA0
/dev/tty1   /dev/tty13  /dev/tty17  /dev/tty20  /dev/tty24  /dev/tty28  /dev/tty31  /dev/tty35  /dev/tty39  /dev/tty42  /dev/tty46  /dev/tty5   /dev/tty53  /dev/tty57  /dev/tty60  /dev/tty7   /dev/ttyprintk
/dev/tty10  /dev/tty14  /dev/tty18  /dev/tty21  /dev/tty25  /dev/tty29  /dev/tty32  /dev/tty36  /dev/tty4   /dev/tty43  /dev/tty47  /dev/tty50  /dev/tty54  /dev/tty58  /dev/tty61  /dev/tty8
```

<br/>
#### Step 5:
Keeping the `boot` button pressed on [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"} connect it to the computer's USB port. Only steady red LED will glow.

![boot](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/52D26826-0C80-4EA5-98C3-D78BD44C0809.jpeg)

<br/>
#### Step 6:
Now, run the same command as Step 4:
```
ls /dev/tty*
```
Sample output:
```
gmagal@masterrpi:~ $ ls /dev/tty*
/dev/tty    /dev/tty11  /dev/tty15  /dev/tty19  /dev/tty22  /dev/tty26  /dev/tty3   /dev/tty33  /dev/tty37  /dev/tty40  /dev/tty44  /dev/tty48  /dev/tty51  /dev/tty55  /dev/tty59  /dev/tty62  /dev/tty9
/dev/tty0   /dev/tty12  /dev/tty16  /dev/tty2   /dev/tty23  /dev/tty27  /dev/tty30  /dev/tty34  /dev/tty38  /dev/tty41  /dev/tty45  /dev/tty49  /dev/tty52  /dev/tty56  /dev/tty6   /dev/tty63  /dev/ttyACM0
/dev/tty1   /dev/tty13  /dev/tty17  /dev/tty20  /dev/tty24  /dev/tty28  /dev/tty31  /dev/tty35  /dev/tty39  /dev/tty42  /dev/tty46  /dev/tty5   /dev/tty53  /dev/tty57  /dev/tty60  /dev/tty7   /dev/ttyAMA0
/dev/tty10  /dev/tty14  /dev/tty18  /dev/tty21  /dev/tty25  /dev/tty29  /dev/tty32  /dev/tty36  /dev/tty4   /dev/tty43  /dev/tty47  /dev/tty50  /dev/tty54  /dev/tty58  /dev/tty61  /dev/tty8   /dev/ttyprintk
```
This time we see the USB device - `/dev/ttyACM0`. This will be needed for configuring [minicom](https://packages.debian.org/sid/minicom){:target="_blank"}

<br/>
#### Step 7:
Run the below command to start [minicom](https://packages.debian.org/sid/minicom){:target="_blank"}

```
sudo minicom -s -c on
```
This will pop-up below screen in the terminal.

![main page](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom_1.png)

We can navigate the screen using keyboard arrows. To choose a specific suboption enter the alphabet next to it.
Below are the things that needs to be configured before we can flash the device:
* `Filenames and paths`: Provide the directory path where we have downloaded the router firmware

![Filenames and paths](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom2.png)


* `File transfer protocols`: XMODEM should be enabled (it's enabled by default). If we need to change XMODEM to use command as `lsx` that needs to changed it in this screen.

![File transfer protocols](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom_3.png)

* `Serial port setup`: This is the option where we should specify the Serial Device (which we obtained in Step 6 above). The baud rate should be 115200 (default)

![Serial port setup](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom4.png)

* `Save the Setup as df1`: Save the configuration.

![Save the Setup as df1](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom_5.png)

* `Exit`: Exit from the configuration screen. This should take to the command screen

<br/>
#### Step 8:
Now press `1` on the keyboard, we should see something like below, the upload connection is established to the USB device:

![1](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom6.png)

<br/>
#### Step 9:
Now press `control` and `a` together, leave it and press `z` on the keyboard, this should open up the available options. `s` is used for sending file to the device (on MAC OS, META key needs to be mapped to OPTIONS key which can be done from terminal > keyboard preferences).

![control+a](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom7a.png)

<br/>
#### Step 10:
Press `s`, now we should see options like below:

![s](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom7b.png)

<br/>
#### Step 11:
We need to choose `xmodem` protocol. Use keyboard arrow to navigate to `xmodem` and hit `return` on the keyboard. This should open up a dialog box inside terminal showing all the files available that are inside the directory that was configured in Step 7 `Filenames and paths`.

![files](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom7c.png)

<br/>
#### Step 12:
Using keyboard arrow key navigate to the firmware file and hit space bar to choose that file, followed by return. This should start the upload
process. Once completed we should see something like:

![upload](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom7d.png)

<br/>
#### Step 13:
Now disconnect [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"} from the computer

<br/>
#### Step 14:
🪛 Screw back the case and plug it into any USB outlet with 5V DC (minimum 1Amp current). The router should be in pairing mode.

<br/>
#### Step 15:
Using Homeassistant or zigbee2mqtt we can pair the new router with the zigbee coordinator

<script src="https://utteranc.es/client.js" repo="gmrock/gmrock.github.io" issue-term="pathname" label="Comments" theme="github-light" crossorigin="anonymous" async> </script> 
