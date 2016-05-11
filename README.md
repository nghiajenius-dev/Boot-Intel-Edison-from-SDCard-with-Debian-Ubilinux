# Boot-Intel-Edison-from-SDCard-with-Debian-Ubilinux

## Boot Intel Edison board with the Root Filesystem on an external storage



Before following any of these steps, make sure you have booted **_ubilinux or debian_** os 
(from [Emutexlabs](http://www.emutexlabs.com/)) on Intel Edison.

## Prepare External Storage (MicroSD Card)


* You need to have a SD card formatted with **ext4 file system** to store the root file system
* Download and unpack Debian image from [emutexlabs website](http://www.emutexlabs.com/)
* In the unpacked directory (with the "flashall.sh" script) run the following commands to
   prepare the SD card. These commands mount the edison-image-edison.ext4 rootfs image and 
   copy the contents onto the SD card

  	
        sudo su
        mkdir Rootfs
        mount ./edison-image-edison.ext4 Rootfs
        cp -a Rootfs/* media/saru/eb361b92-c285-44a6-9f32-da393c879487
        sync
  	
* Eject the SD Card and now it is ready to boot


## Find out the SD card device name

* On a running Edison board, plug your formatted SD card and get the device name.

        dmesg | tail -n 10
        
  
<p align="center">
<img src="https://github.com/sarweshkumar47/Boot-Intel-Edison-from-SDCard-with-Debian-Ubilinux/blob/master/Screenshots/s1.png" alt="sdcard";>
</p>
  
  
* Here, the SD card device is **“/dev/mmcblk1”** and the partition we’ve created is **"/dev/mmcblk1p1"**
 

## Boot the board using SD card

Debian system on Edison, does not mount the microsd card automatically. To mount sdcard at boot, modify the /etc/fstab system file in edison.

* Find the UUID for the sd card

         blkid
 
   
<p align="center">
<img src="https://github.com/sarweshkumar47/Boot-Intel-Edison-from-SDCard-with-Debian-Ubilinux/blob/master/Screenshots/s2.png" alt="blkid" style="width:304px;height:228px;">
</p>
      
* Add an entry for the partition to automount it at startup. Add file system, mount point, type, options, dump and pass information of sdcard partition to fstab file and save it. Example, the format looks like below,

   #####/dev/sdcX      /media/sdcard    ext4    defaults    0     0
           
         nano /etc/fstab
         #Add an entry for the partition
         UUID=eb361b92-c285-44a6-9f32-da393c879487    /media/sdcard    ext4    defaults    0 0
         mkdir -p /media/sdcard/
         reboot

<p align="center">
<img src="https://github.com/sarweshkumar47/Boot-Intel-Edison-from-SDCard-with-Debian-Ubilinux/blob/master/Screenshots/s3.png" alt="blkid" style="width:304px;height:228px;">
</p>
    
    
After reboot, debian os automatically mounts the sdcard.
   

<p align="center">
<img src="https://github.com/sarweshkumar47/Boot-Intel-Edison-from-SDCard-with-Debian-Ubilinux/blob/master/Screenshots/s4.png" alt="Mountain View" style="width:304px;height:228px;">
</p>
   
   
_To boot using the external device, you need to modify the U-Boot environment variable named **"mmc-bootargs"** with kernel boot arguments. We cannot modify U-Boot environment variables from edison linux console in debian os. It is possible only in U-Boot command prompt._



## Note:  

* To modify U-Boot env variables, u-boot and u-boot-tools packages are available in debian repository. The tools           fw_printenv/fw_setenv in the u-boot source tree normally work with MTD partitions but they don’t support MMC.

-----------------------------------------------------------------------------------------------------------------

To modify u-boot env variables ,during the boot, on the serial console, when it prompts to press a key to stop    booting, press any key and then run the following commands in the U-Boot command prompt.
   

<p align="center">
<img src="https://github.com/sarweshkumar47/Boot-Intel-Edison-from-SDCard-with-Debian-Ubilinux/blob/master/Screenshots/s5.png" alt="Mountain View" style="width:304px;height:228px;">
</p>
  
  Run the following commands in boot command prompt
  
      > setenv mmc-bootargs 'setenv bootargs root=${myrootfs} rootdelay=3 rootfstype=ext4 ${bootargs_console} ${bootargs_debug} systemd.unit=${bootargs_target}.target hardware_id=${hardware_id} g_multi.iSerialNumber=${serial#} g_multi.dev_addr=${usb0addr}'

      > setenv myrootfs '/dev/mmcblk1p1'

      > setenv myrootfs_sdcard /dev/mmcblk1p1

      > setenv myrootfs_emmc PARTUUID=012b3303-34ac-284d-99b4-34e03a2335f4

      > setenv do_boot_emmc 'setenv myrootfs ${myrootfs_emmc}; run do_boot'

      > setenv do_boot_sdcard 'setenv myrootfs ${myrootfs_sdcard}; run do_boot'

      > setenv bootdelay 5

      > saveenv

      > run do _boot_sdcard

<p align="center">
  <img src="https://github.com/sarweshkumar47/Boot-Intel-Edison-from-SDCard-with-Debian-Ubilinux/blob/master/Screenshots/s6.png" alt="Mountain View" style="width:304px;height:228px;">
</p>


####After boot, verify that you are using rootfs stored on your external device.

<p align="center">
  <img src="https://github.com/sarweshkumar47/Boot-Intel-Edison-from-SDCard-with-Debian-Ubilinux/blob/master/Screenshots/s7.png" alt="Mountain View" style="width:304px;height:228px;">
</p>

## Note: 

* If you want to switch to eMMC just occasionally, during the boot, on the serial console, when it prompts to press a key to stop booting, press any key and then run the below command in the U-Boot command prompt that appears:

        > run do_boot_emmc


	
## Reference

 * [https://communities.intel.com/thread/61048](https://communities.intel.com/thread/61048)

 * [http://www.denx.de/wiki/view/DULG/UBootCmdGroupEnvironment](http://www.denx.de/wiki/view/DULG/UBootCmdGroupEnvironment)
