# Galicaster-NDI
Documentation for installation of a Teltek Galicaster Lecture Capture agent for NDI sources

Resources Required:

[1] https://github.com/teltek/gst-plugin-ndi
[2] https://github.com/teltek/gst-plugin-ndi/releases
[3] https://github.com/teltek/Galicaster/tree/master


Install Guide 64 Bit Ubuntu:

1. Install / Update Ubuntu LTS 16.04.5 with 'galicaster' as user [UK HEI mirror http://www.mirrorservice.org/sites/releases.ubuntu.com/] 

if installed in a VM use: 

sudo apt-get install open-vm-tools open-vm-tools-desktop 

instead of VMware tools.

Restart to enable vm tools with:

sudo shutdown -r now


2. Install NDI SDK by running the install .sh Linux SDK from:
 
http://pages.newtek.com/NDI-Developers-SDK-Download-Link.html 
- click the 'linux' link.

this will download InstallNDISDK_v3_Linux.sh to ~/Downloads

then:

cd ~/Downloads

sudo chmod +x InstallNDISDK_v3_Linux.sh

sudo ./InstallNDISDK_v3_Linux.sh

[keep pressing the space bar until you see a Y/n prompt and type 'y' then press return]

The Linux NDI 3.7.1 SKD will be extracted to ~/Downloads 

3. Move the files from the extracted 'NDI SDK for Linux' folder in ~/galicaster/Downlods into place with:

sudo cp -r 'NDI SDK for Linux/lib/x86_64-linux-gnu' /usr/lib


4. Install Galicaster from the repository (to ensure dependencies are met) by running the following commands (each as a separate command):

echo "deb https://packages.galicaster.org/apt xenial main" | sudo tee --append /etc/apt/sources.list.d/galicaster.list

wget -O - https://packages.galicaster.org/apt/galicaster.gpg.key  | sudo apt-key add -

sudo apt-get update

sudo apt-get install gstreamer1.0-plugins-good gir1.2-gstreamer-1.0 #Required to solve #298 and #29

sudo apt-get install galicaster

[select 'y' press return when prompted]

Installs in: 
/usr/share/galicaster 

with config files at: 
/etc/galicaster/


4.5 Install Galicaster with NDI Feature in a separate location [e.g. ~/Desktop]:

[Note - the 'feature_ndi' from https://github.com/teltek/Galicaster/tree/feature_ndi is no longer required as the feature has been folded into the Master Branch, the version in the repository in the previous step does not yet have the NDI feature] 

use:
https://github.com/teltek/Galicaster/archive/master.zip
[click / download a zip and the extract to the desktop preserving folder structure]


5. Download the compiled GStreamer element 'libgstndi.so' available from: 

https://github.com/teltek/gst-plugin-ndi/releases 

to Ubuntu ~/Downloads and move libgstndi.so - copy this to the /usr/lib/x86_64-linux-gnu/gstreamer-1.0/ directory for GStreamer to load the University of the Arts and University of Manchester funded element:

sudo cp '/home/galicaster/Downloads/libgstndi.so'  /usr/lib/x86_64-linux-gnu/gstreamer-1.0/

sudo ldconfig

test with:

gst-inspect-1.0 ndi

Then Test an NDI Source with Gstreamer works outside of Galicaster:
(opt-tab into terminal and control-c to close stream)

[1] GStreamer basic pipelines (Change stream-name to yours):

Video Only [run first - note you will need to change the NDI source inside the double quotes to a local NDI source on the same subnet]:

gst-launch-1.0 ndivideosrc stream-name="UALRETINA.LOCAL (NDI Signal Generator)" ! videoconvert ! autovideosink

Video + Audio:
gst-launch-1.0 ndivideosrc stream-name="GC-DEV2 (OBS)" ! videoconvert ! autovideosink ndiaudiosrc stream-name="GC-DEV2 (OBS)" ! autoaudiosink

Video + Video:
gst-launch-1.0 ndivideosrc stream-name="GC-DEV2 (OBS)" ! videoconvert ! autovideosink ndivideosrc stream-name="GC-DEV2 (OBS)" ! videoconvert ! autovideosink

test with: 
dpkg -l | grep gstreamer


Test Teltek Element:
gst-inspect-1.0 ndi

Should return:

galicaster@galicaster-ndi:/usr/lib/x86_64-linux-gnu/gstreamer-1.0$ gst-inspect-1.0 ndi
Plugin Details:
  Name                     ndi
  Description              NewTek NDI Plugin
  Filename                 /usr/lib/x86_64-linux-gnu/gstreamer-1.0/libgstndi.so
  Version                  1.0
  License                  MIT/X11
  Source module            ndi
  Source release date      2018-04-09
  Binary package           ndi
  Origin URL               https://gitlab.teltek.es/rubenrua/ndi-rs.git

  ndiaudiosrc: NewTek NDI Audio Source
  ndivideosrc: NewTek NDI Video Source

  2 features:
  +-- 2 elements


for more detail of the two sources:

gst-inspect-1.0 ndiaudiosrc
gst-inspect-1.0 ndiviseosrc





6. Add the ndi.ini profile to Galicaster - a plain txt file at:

/etc/galicaster/profiles

Edit to fit the target NDI source. You can use the mDNS NDI syntax: SOMETHING.LOCAL (NDI Stream Name) 
or the IP followed by the port for the steam: 10.80.80.12:596x (5961 is the first stream, 5962 is the second from the same IP etc).

Note - The mDNS name is case sensitive

You can add multiple NDI sources - performance above 1x video track will vary. 

Make sure the permissions for the resource folder are correct:
sudo chown -R galicaster:galicaster /etc/galicaster

Example NDI confugration profile 'ndiexample.ini' :


[data]
name = NDI (audio track + video track)

[track1]
name = OBS NDI
device = ndi
location = GC-DEV2 (OBS)
file = WEBCAM.avi
flavor = presenter
audio = False
videosink = xvimagesink

[track2]
name = NDI Audio
device = ndi_audio
location = GC-DEV2 (OBS)
file = sound.mp3
flavor = presenter
player = True
vumeter = True
amplification = 1.0


7 Run the NDI feature version of Galicaster-master from the desktop:

'/home/galicaster/Desktop/Galicaster-master/run_galicaster.py'

[Opt Tab out of GC to lock the 2.2.1-rc2 version to the dock for easy access]

(Currently, you may need to select an NDI source profile, quit, then reopen Galicaster if the track count is less than the previously used profile)


