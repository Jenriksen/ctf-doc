# Hardware (200pts)
## #2011 #CTF

We are given a file called "capture.pcap" and the instructions "MD5 of the image."

Upon analyzing the image with wireshark we see that it is a capture of USB traffic from the USB controller on a linux system.

In the 2nd package we see information about one of the devices connected to the usb controller.
![USB Device information] (./images/USB-Device-Descriptor.png)



In the 32nd package we see a USB device connected which responds with the information about the vendor being Kingston Technology Company Inc.
((Insert picture of "Device Descriptor Response"))

Packet 63 shows the USBMS (USB Mass Storage) protocol which concludes the hint from 32nd package about it being a storage device.

Searching for "jpg" in "packet bytes" using string search reveals package 430 which holds information about 
((insert picture of package bytes))

Since its a USB stick connectec to a Linux system it makes sense that it should be running a FAT filesystem.
Searching for "FAT" using string search in package bytes reveals package 254 showing reference to "MSDOS5.0" and "FAT32" confirming the filesystem.

Looking through the file reveals that the USBMS protocol uses SCSI commands and LUN for accessing the USB stick. Using this information we can see that the SCSI Read(10)
 ([Seagate](https://www.seagate.com/staticfiles/support/disc/manuals/scsi/100293068a.pdf) - p.133)

Using this we can see which Large Blocks to examine further as the SCSI commands list a specific "Large Block Address" and length they want to read.

Last 8 packages from 1534 to 1542 we can see in hex that the jpg file signature is the first 6 numbers in ` SCSI Payload(READ(10) Response Data` which is what the actual data is referred to as.

Exporting this data to text files leaves us with a large hexadecimal file where we can see 117 sets of zeroes at the end, which we will remove together with the linebreaks created by wireshark in the export stage.

Now you can use a hexadecimal-to-binary tool called xxd with the following command: 
```
xxd -r -p input.txt output.jpg
```

Use MD5sum on the corresponding file and you have solved the challenge.

```
md5sum output.jpg
```
