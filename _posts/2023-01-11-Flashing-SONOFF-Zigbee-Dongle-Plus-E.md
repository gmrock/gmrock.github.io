I was able to succesfully flash [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"} with router firmware. Detailing the steps below, as I wasn't able to find a guide on how to do this.

### Why you might need to flash firmware:
- upgrade
- to use a different (opensource) option
- convert a Zigbee coordinator into a Zigbee router (this was my use case)


Apparently, the [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"} acts as a very good router (I have just started using it as router and so far it's reliable).

<br/>
#### Step 1:
Unscrew [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"}'s case. Will need to only unscrew the 2 screws which are on the side of the USB port. This is to get access to the boot button.

![boot](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/C51079D8-DC05-4C04-B209-061AA596CF41.jpeg)

<br/>
#### Step 2:
Download and install [minicom](https://packages.debian.org/sid/minicom){:target="_blank"}. There are other tools that can also be used such as
[coolterm](https://freeware.the-meiers.org/). If using `coolterm` on MAC OS make sure you take care of steps detailed below `For MAC OS`. 

**For linux OS:**
* run the below command:**
```
sudo apt-get install minicom
```
minicom has a dependency on [lrzsz](https://www.ohse.de/uwe/software/lrzsz.html){:target="_blank"} which is needed for XMODEM,YMODEM and ZMODEM communication. On linux platform, when minicom is installed using above command, this dependency is automatically installed. However, on MAC OS, this dependency is not installed with minicom. Follow the below steps to install minicom and the dependency on MAC OS.

**For MAC OS:**
* run the below command to install minicom:
```
brew install minicom
```
* run the below command to install the dependency [lrzsz](https://www.ohse.de/uwe/software/lrzsz.html){:target="_blank"}:
```
brew install lrzsz
```
* minicom internally uses `sx` to use XMODEM communication (this can be changed in Step 6 > #2 configuration page to make it use lsx. If you do so
skip below points and jump to Step 3). However, if you look at the installation directory, inside `bin` directory for lrzsz there is no `sx` command.
There is `lsx` command which is same as `sx`. 

```
ganesh@magal:/usr/local/Cellar/lrzsz/0.12.20_1/bin$ ls
lrb	lrx	lrz	lsb	lsx	lsz	rz	sz
```
`/usr/local/Cellar/lrzsz/0.12.20_1/bin` is the location where homebrew installed lrzsz.
* we will create a symlink called `sx` and link it to `lsx` which is already inside the `bin` directory.

```
ln -s /usr/local/Cellar/lrzsz/0.12.20_1/bin/lsx /usr/local/Cellar/lrzsz/0.12.20_1/bin/sx

```
* should see like this now:
```
gmagal@magal:/usr/local/Cellar/lrzsz/0.12.20_1/bin$ ls
lrb	lrx	lrz	lsb	lsx	lsz	rz	sx	sz
```
* one last thing that needs to be done is to add `sx` to PATH. When using shell as zsh. Create ~/.zshrc if not already present:
```
gmagal@GMAGAL-M-M1D2:$ nano ~/.zshrc
```
* add below to the ~/.zshrc file, save and exit terminal:
```
export PATH=/usr/local/Cellar/lrzsz/0.12.20_1/bin:$PATH
```
* reopen terminal and type `sx` see if the command was found. If not found, the PATH has been set in the correct file (below command was found):
```
gmagal@magal:~$ sx
sx: need at least one file to send
Try `sx --help' for more information.
```

<br/>
#### Step 3:
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
#### Step 4:
Keeping the `boot` button pressed connect it to the computer's USB port. Only steady red LED will glow

![boot](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/52D26826-0C80-4EA5-98C3-D78BD44C0809.jpeg)

<br/>
#### Step 5:
Now, run the same command as Step 3. This time we will take a note of the serial device i.e. [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"}:
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
If you notice, we see the USB device - `/dev/ttyACM0`. This will be needed for configuring [minicom](https://packages.debian.org/sid/minicom){:target="_blank"}

<br/>
#### Step 6:
Run the below command to start [minicom](https://packages.debian.org/sid/minicom){:target="_blank"}

```
sudo minicom -s -c on
```
This will pop-up below screen in the terminal.

![main page](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom_1.png)

You can navigate the screen using keyboard arrows and choose specific suboption choose the alphabet next to it.
Below are the things that needs to be configured before we can flash the device:
* `Filenames and paths`: Provide the directory path where you have downloaded the new firmware which you want to flash

![Filenames and paths](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom2.png)


* `File transfer protocols`: XMODEM should be enabled (it's enabled by default). This is how it should look (if you need to change XMODEM to use command as `lsx` you need to change it in this screen)

![File transfer protocols](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom_3.png)

* `Serial port setup`: This is the place where we specify our Serial Device (which we obtained in Step 5 above). The baud rate should be 115200 (default)

![Serial port setup](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom4.png)

* `Save the Setup as df1`: Save our configuration, so that next time the settings are saved

![Save the Setup as df1](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom_5.png)

* `Exit`: Exit from the configuration screen. This should take to the command screen

<br/>
#### Step 7:
Now press `1` on the keyboard, you should see something like below, the upload connection is established to the USB device:

![1](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom6.png)

<br/>
#### Step 8:
Now press `control` and `a` together, leave it and press `z` on your keyboard, this should open up the options and you can `s` is used for sending file to the device (on MAC OS, META key needs to be mapped to OPTIONS key which can be done from terminal > keyboard preferences).

![control+a](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom7a.png)

<br/>
#### Step 9:
Press `s`, you should see options like below:

![s](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom7b.png)

<br/>
#### Step 10:
We need to choose `xmodem` protocol. Use keyboard arrow to navigate to `xmodem` and hit `return` on your keyboard. This should open up a dialog inside terminal showing all the files available inside the directory which we had configured in Step 6 (under 1).

![files](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom7c.png)

<br/>
#### Step 11:
Using keyboard arrow key navigate to the firmware file and hit space bar to choose that file, followed by return. This should start the upload
process and you should see something like:

![upload](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/minicom7d.png)

<br/>
#### Step 12:
Now disconnect [SonOff Zigbee dongle plus E](https://sonoff.tech/product/gateway-and-sensors/sonoff-zigbee-3-0-usb-dongle-plus-e/){:target="_blank"} from your computer

<br/>
#### Step 13:
Screw back the case and plug it into any USB outlet with 5V DC (minimum 1Amp current). You have the router ready in pairing mode.

<br/>
#### Step 14:
Use Homeassistant or zigbee2mqtt or whatever software you use to pair the new router with your zigbee coordinator
