## CinePi-2K Clip Metadata
For generating and continuously  <i>metadata.csv</i> and <i>edl.edl</i> with all the clips synced on a timeline, for easy import to DaVinci. Uses the file <i>controls.py</i>, a slightly modified version of the original `home/pi/camera/proxy.py` to get the shutter angle and ISO from the camera. Like for <i>rec_signal.py</i> GPIO 21 has to be connected to GPIO 20. Edl file conforms to standard edl format,

Clips will be conformed to 24 fps. To change this, change the variable `fps_base`. Clips shot in a different frame than fps_base (for example when speed ramping or doing slow-mo) will be marked with a pink flag in DaVincvi Resolve.

##Wiring

For the script to work, GPIO 21 has to be connected to GPIO 20. The script uses GPIO 20 to detect when recording stops, and then writes the metadata of the clip just recorded.

## Installing an RTC

For script to work properly you need an RTC (Real Time Clock) connected to the Pi. I recommend the DS3231 type. Since I2C channel 1 is used by the camera you need to create a second I2C channel. 

Files will be written to the SSD drive so the script has to be run as sudo.

SSH into the Pi.

Go into raspi-config and enable I2C.

<code>sudo raspi-config</code>

Enable I2C

Open the file /boot/config.txt

<code>sudo nano /boot/config.txt</code>

Add the following line to the file:

<code>dtoverlay=i2c-gpio,bus=2,i2c_gpio_delay_us=1,i2c_gpio_sda=27,i2c_gpio_scl=18</code>

This will create a second I2C channel on with GPIO 27 as SDA and GPIO 18 as SCL. Feel free to change these if needed but avoid GPIO 2,3,4,20,21 as these are used by other scripts/camera system

Exit by pressing ctrl-x and then selecting "yes".

<code>sudo pip3 install smbus2</code>

Open the file `/etc/modules`

<code>sudo nano /etc/modules</code>

Add the following three lines to the end.

<code>i2c-bcm2708</code>

<code>i2c-dev</code>

<code>rtc-ds1307</code>

Exit by pressing ctrl-x and then selecting "yes".

then open /etc/rc.local

<code>sudo nano /etc/rc.local</code>

Add the following lines before the line <i>python3/home/pi/camera/cameracore3.py &</i>

<code>echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-2/new_device &</code>

<code>hwclock -s &</code>

Exit by pressing ctrl-x and then selecting "yes".

Reboot the Pi

## Installing the script

SSH into the Pi. Then,

`git clone https://github.com/Tiramisioux/cinepi-2k-clip-metadata.git`

## Make script run on start-up

`sudo nano /etc/rc.local`

add the lines

`sudo python3 /home/pi/cinemate/metadata.py &`

before the line <i>exit 0</i>

Exit by pressing ctrl-x and then selecting "yes".

<i><b>... or</b></i> add the script to the end of the file `home/pi/camera/cameracore3.py. Remember to add sudo like above.

Reboot the Pi.

## Importing metadata and edl to DaVinci Resolve

In Resolve:

Select the clips in your bin.

<i>File > Import metadata to > selected media pool clips</i>

In the dialog window, deselect <i>Match using clip start and Timecode</i> and select <i>Update all metadata fields available in the source file</i>.

<img width="640" alt="resolve" src="https://user-images.githubusercontent.com/74836180/179369440-84b2401b-047f-4a51-b7da-1ef1248c8a9e.png">

Select the file metadata.csv on the SSD.

For the EDl, go to <i>File > Import > Timeline</i> and select edl.edl on the SSD.

Note that the EDL will place the clips on a timeline using Time Of Day timecode. If you have material from different shooting days on the SSD they will get mixed up on the timeline. Will fix this in the future by making the script create a new EDL for each shooting day.
