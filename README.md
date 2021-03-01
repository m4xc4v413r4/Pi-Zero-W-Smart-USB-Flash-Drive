# Pi Zero W Smart USB Flash Drive
SMB enabled network share that appears as a USB drive to connected device

## Acknowledgements

This project is not an original idea.  It is based on the work documented in the MagPi magazine [article](https://magpi.raspberrypi.org/articles/pi-zero-w-smart-usb-flash-drive) by the same name.  It was written by [Russell Barnes](https://magpi.raspberrypi.org/articles/author/77pb3df8MQLs3i8qTd0C8Q). It also barrows some ideas from this [project](https://github.com/gilesdavison/RadaDASH) by [Giles Davison](https://github.com/gilesdavison).

## Features
* Remotly accessabel USB device (uploading, deleting)
* 2GB shared storage pre configured (default)
* Create AnyCubic WIFI.txt file based on the Raspberry PI's configuration

## Printer Support
This project should work for most (if not all) 3D printers.  However, it was created with the [ANYCUBIC Photon Mono X](https://www.anycubic.com/products/photon-mono-x-resin-printer) in mind. There is an optional service that can be enabled to dynamically create a WIFI.txt file in the ANYCUBIC format to help configure the printer to match the netowrk configured on the Raspberry Pi W.

## Prerequisites

* MicroSD Card >= 8GB
* MicroSD Card Adapter/Reader
* USB to Micro USB Cable
* [Raspberry Pi Zero W](https://www.raspberrypi.org/products/raspberry-pi-zero-w/)*

*Note, will not work on other Raspberry Pi's since they do not support USB OTG.

## Setup Instructions



Step 1. Download the latest [release](https://github.com/tds2021/Pi-Zero-W-Smart-USB-Flash-Drive/releases/tag/v1.0-beta).

Step 2. Extract the image from the zip archive.

Step 3. 

(MacOS) Connect the MicroSD card to your computer and flash the usb_share image using [balenaEtcher](https://www.balena.io/etcher/).

#### or

Connect the MicroSD card to your computer and flash the usb_share image.
```
diskutil list
sudo diskutil unmountDisk /dev/diskN
sudo dd bs=1m if=path_of_your_image of=/dev/rdiskN; sync
```

(Windows) Connect the MicroSD card to your computer and flash the usb_share image using [Win32DiskImager](https://sourceforge.net/projects/win32diskimager/).

Step 4. Edit the [wpa_supplicant.conf](https://github.com/tds2021/Pi-Zero-W-Smart-USB-Flash-Drive/blob/main/wpa_supplicant.conf) file provided, changing the placeholders network and password to your wireless networks SSID and password.

Step 5. Copy the *wpa_supplicant.conf* file to the boot partion of the SD card.


## Powering the USB Drive / Raspberry Pi

On the Pi Zero W, you’ll see two micro USB ports. One is marked ‘USB’ and the other ‘PWR IN’. You can supply power through either port, but the USB port is for data as well. There are two options.

You can connect your 3D printer into the Pi Zero W USB port, not the PWR IN port, using a standard micro USB cable. The cable will both supply power from the printer and make the USB data connection. The disadvantage is that the printer must be switched on to supply power to the Pi. When someone turns the printer off, the Pi will also lose power, which cwill allow make it no longer available on teh network.

Alternatively, you can connect a separate, always-on power supply to the PWR IN port, and use a slightly modified micro USB cable to connect the the printer to the USB port. The modification is to cut the red wire inside the micro USB cable. This protects the Pi from damage that could be caused by drawing power from two different power sources. The advantage of this method is that the Pi is powered independently from the printer. It will be available on the network even if the printer is off, and there is a reduced risk of sudden power loss and SD card corruption.

You might want to test the system with the first option, and then move onto the second when you want a more permanent setup. **Don’t forget to cut the red wire if you use the second option.**

Connect the Pi Zero W USB port to the printer using your chosen method, power everything up.

## Connecting to the Network Drive

Default Host Name: 3dprinter

Now we can try to access the share from a Windows PC or a Mac. You’ll need the host name the Raspberry Pi is using or it's IP Address. By default the host name will be **3DPRINTER**.

In Windows, you can bring up Explorer (Windows key + E) and type \3dprinter (or \<<IP ADDRESS>>) into the address bar at the top. The Run dialogue also works (Windows key + R).

On macOS, the Raspberry Pi will show up in the Finder sidebar. Alternatively, from the Finder menu, select Go Connect to server (Apple key + K) and type smb://3dprinteras (or smb://<<IP ADDRESS>>) as the server address.

Once connected, you will see a shared named _USB_ where you can load your files. In order for your updaed to appear on teh printer, it must first disconnect from teh printer and then reconnect. Whenever you copy files over to the network share, or delete them, the USB device should automatically reconnect to the printer after 30 seconds of inactivity.



# Optional configurations

The following optional configurations can be made from the Pi console, or over SSH.

Default userID: pi

Default password: raspberry

#### Enabling the ANYCUBIC WIFI service

The ANYCUBIC Mono X printer has built in WIFI capabilites.  This printer option is configured using a text file named _WIFI.txt_ that is _printed_ on the printer.  If you wish to have this file automatically created and added to the avaialbe files on the USB drive, based on what was configured in the wpa_supplicant.conf file, follow these instructions.

Step 1. Login to the Raspberry Pi using the UserID/Password combination provided above.

Step 2. Run the following commands:
```
sudo systemctl daemon-reload
sudo systemctl enable monox_wifi.service
sudo systemctl start monox_wifi.service
```


#### Changing the Host Name

By decault, teh USB device will be named #DPRINTER.  However, you can cahnge this if you wish by updating teh hostname on the Pi.

Step 1. Login to the Raspberry Pi using the UserID/Password combination provided above.

Step 2. Change the name in the /etc/hostname file
```
sudo nano /etc/hostname
```
Press CTRL+O followed by Enter to save, and then CTRL+X to quit.

Step 3. Change the name in the /etc/hosts file
```
sudo nano /etc/hosts
```
Press CTRL+O followed by Enter to save, and then CTRL+X to quit.

Step 3. Restart the Raspberry Pi
```
sudo reboot
```


#### Change the size of the USB drive (based on avaialbe space)

By Default, the USB drive is configured as a 2GB drive.  This is set to accomidate smaller SD Cards.  however, you can change this by recreatign the image file on the Raspberry Pi.  

Step 1. Login to the Raspberry Pi using the UserID/Password combination provided above.

Step 2. Stop the USB OTG service
```
sudo modprobe -r g_mass_storage
```

Step 3. Unmount the exisitng disk image
```
sudo umount /home/pi/USB_Share/upload
```

Step 4. Remove the existing disk image file
```
sudo rm /home/pi/USB_Share/usbdisk.img
```
Step 5. Check the avaialbe free space

The command df -h shows your disk space usage. Look at the Avail column for /dev/root to see how much free space you have. 
```
df -h
```

Step 6. Create a new disk image file

The command below will create an empty 2GB binary file (change the count=2048 parameter if you want a different size). **keep a couple GB free**; if you fill the card completely full the image will become corupt while running. This will take a long time to complete; the larger the file site the longer it will take.
```
sudo dd bs=1M if=/dev/zero of=/home/pi/USB_Share/usbdisk.img count=2048
```

Step 7. Formate the disk image as FAT32
```
sudo mkdosfs /home/pi/USB_Share/usbdisk.img -F 32 -I
```

Step 8. Remount the disk image
```
sudo mount -a
```

Step 9. Start the USB OTG service
```
sudo modprobe g_mass_storage file=/home/pi/USB_Share/usbdisk.img stall=0 ro=0 removable=1
```

Step 10. Restart the Raspberry Pi
```
sudo reboot
```
