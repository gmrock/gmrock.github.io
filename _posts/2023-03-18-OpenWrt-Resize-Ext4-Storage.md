When we configured [Raspberry Pi As OpenWrt Router](https://gmrock.github.io/2023/03/05/Raspberry-Pi-as-OpenWrt-router.html){:target="_blank"}, we had used `FACTORY (SQUASHFS)` as the openwrt image (`Step 3` in [Raspberry Pi As OpenWrt Router](https://gmrock.github.io/2023/03/05/Raspberry-Pi-as-OpenWrt-router.html#:~:text=Now,%20the%20firmware%20by%20choosing%20FACTORY%20(SQUASHFS)%20option){:target="_blank"}). You might have noticed even though the microsd card had a much bigger capacity(16gb in my case), but what was actually available for openwrt (~250mb in my case) was much lower storage.

This is because openwrt is bundled to work on lower memory footprint devices too. In this article, I have detailed how we can resize the microsd card to utilize the entire capacity for our openwrt setup.


## HARDWARE:
- [USBC/USB2 to microSD card reader](https://www.amazon.com/UGREEN-Reader-Adapter-Portable-Windows/dp/B07D1J88CF/){:target="_blank"}. This is optional. This specific device is compatible with Mac, Windows and Linux.

<hr/>

## SOFTWARE:
- [Virtual Box](https://www.virtualbox.org/){:target="_blank"} - To create Ubuntu VM

<hr/>

## STEPS:
Instead of following the `Step 3` from [Raspberry Pi As OpenWrt Router](https://gmrock.github.io/2023/03/05/Raspberry-Pi-as-OpenWrt-router.html){:target="_blank"}. We will execute below steps.

#### Step A:
After adding the additional package, click on REQUEST BUILD (it takes few seconds to get custom build ready). Now, download the firmware by choosing `FACTORY (EXT4)` option. This will download *.img.gz file. Unzip the file, and flash the image to the microsd card using etcher.

#### Step B:
Now connect the microSD card(I used the above mentioned card reader as the one which I had wasn't getting detected in Ubuntu) to a Ubuntu machine OR to any machine that is running Ubuntu VM (it's much easier to resize the EXT4 storage on linux systems).  
I have Ubuntu VM on [Virtual Box](https://www.virtualbox.org/){:target="_blank"} that is running on my MacBook (host).  
**Note:** Make sure when starting [Virtual Box](https://www.virtualbox.org/){:target="_blank"} on your host machine(Mac OS in my case), it's started with sudo permission. Without this, the Ubuntu VM will not be able to detect the microSD card. After installing [Virtual Box](https://www.virtualbox.org/){:target="_blank"}, you can start it in sudo level by running the below command from terminal

```
sudo virtualbox &
```
This should start the [Virtual Box](https://www.virtualbox.org/){:target="_blank"} on your machine with sudo permission.

#### Step C:
Open terminal and run the below command - `lsblk`. Below is the highlighted entries that is associated with the microsd card.
![lsblk](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/lsblk.png)


#### Step D:
Now, let's unmount the the `rootfs` partition. This is needed because we are going to resize this partition. Run this command to unmount - `sudo umount /media/ganesh/rootfs`. Enter the password when prompted.

#### Step E:
Let's run this command - `sudo fdisk /dev/sdb`. Then at the prompt press `p` then press `d`, then press `2`. This should result in deleting the 2nd partition. So far it should look like below:
![press keys](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/presskeys.png)


#### Step F:
Now at the same prompt, press `n` for new followed by `p` for partition. Now press `2` for choosing 2nd partition. Now enter the sector address where the 2nd partition starts i.e. where 1st partition ends. I have highlighted that address in the below screenshot. In this case it's `147456`, enter that in the prompt. Then at the next prompt don't enter anything as we want to use the entire storage available in the microsd card. It should look like this:
![press keys 2](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/presskeys2.png)


#### Step G:
Now press `n` for no and then `w` for write. This will get us out of the prompt. It should look like this.
![write](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/write.png)


#### Step H:
Now type the command - `sudo e2fsck -f /dev/sdb2`. This is for clean up activity. Now press `y` to remove padding. It should look like this.
![write](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/cleanup.png)

#### Step I:
Now run the command to resize - `sudo resize2fs /dev/sdb2`. The resize is complete.
![resize](https://raw.githubusercontent.com/gmrock/gmrock.github.io/main/media/resize.png)


#### Step J:
Now run this command to unmount the microsd card - `sudo umount /dev/sdb1`. Remove the microsd card from the laptop and insert the microsd card in Raspberry Pi, connect the external LAN dongle to USB3 port and power it ON. 


#### Next Step:  
Now go back to `Step 4` in [Raspberry Pi As OpenWrt Router](https://gmrock.github.io/2023/03/05/Raspberry-Pi-as-OpenWrt-router.html#:~:text=Step%204:){:target="_blank"} and complete the rest of the steps.

<hr/>

## REFERENCES:

- This [video](https://www.youtube.com/watch?v=_pBf2hGqXL8){:target="_blank"} was helpful in resizing EXT4 for openwrt.
<br/>



<script src="https://utteranc.es/client.js" repo="gmrock/gmrock.github.io" issue-term="pathname" label="Comments" theme="github-light" crossorigin="anonymous" async> </script> 
