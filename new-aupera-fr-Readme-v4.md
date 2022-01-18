[[[ALVEO_U30_HOST_SETUP_SECTION]]]

---

# 2. U30 Face Recognition Docker Installation

<b> Install Aupera Face Recognition Docker image </b>

## 2.1 Install essential software and other related packages


```bash
sudo apt update; sudo apt install make build-essential nfs-kernel-server
sudo service rpcbind restart
sudo service nfs-kernel-server restart
```
---

## 2.2 Pull Auper Face Recognition Docker Image

```bash
sudo docker pull auperastor/aupera_face_recognition:3.0.2
sudo docker images | grep aupera_face_recognition
```
---

## 2.3 Copy Firmware, Driver & License configuration file from Docker image


```bash
sudo docker create --name {CONTAINER_NAME} auperastor/aupera_face_recognition:3.0.2 bash
sudo docker cp {CONTAINER_NAME}:/root/firmware {NFS_ABS_PATH}
sudo docker cp {CONTAINER_NAME}:/root/driver {NFS_ABS_PATH}
sudo docker cp {CONTAINER_NAME}:/root/drm {NFS_ABS_PATH}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;»&nbsp; **{CONTAINER_NAME}** is a user defined container name, like "face_recognition"<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;»&nbsp; **{NFS_ABS_PATH}** is local directory where firmware and driver will copied to, like "/opt/aupera/face-recognition"

<b> An example of the command line: </b>
```bash
sudo docker create --name face_recognition auperastor/aupera_face_recognition:3.0.2 bash
sudo docker cp face_recognition:/root/firmware /opt/aupera/face-recognition
sudo docker cp face_recognition:/root/driver /opt/aupera/face-recognition
sudo docker cp face_recognition:/root/drm /opt/aupera/face-recognition
```

---

# 3. Setting up Alveo U30 Card
## 3.1 Install U30 Firmware

<b> Source XRT env and check the current XRT version. Currently XRT version 2.11 or above are required for the firmware installation. </b>

```bash
source /opt/xilinx/xrt/setup.sh
xbutil --version
```
<b> This will give the output similar to: </b>
```
XCLMGMT: 2.11.634
```

---

<b> Run "lspci" command to validate the U30 board seen by the OS </b>

```bash
sudo lspci -d 10ee:
```
<b> This will give the output similar to: </b>
```bash
07:00.0 Processing accelerators: Xilinx Corporation Device 503d (rev 02)
07:00.1 Processing accelerators: Xilinx Corporation Device 503c (rev 02)
08:00.0 Processing accelerators: Xilinx Corporation Device 503d (rev 02)
08:00.1 Processing accelerators: Xilinx Corporation Device 503c (rev 02)
```
+ Two xu30 devices and four functions will be found by the OS if the card is successfully installed.

---

<b> Flash the U30 board using XRT xbmgmt utility: </b>

```bash
sudo /opt/xilinx/xrt/bin/xbmgmt flash --shell --card {card_id} --path {binfile}.bin
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;»&nbsp; **{card_id}** is the BDF ID read from lspci, like 07:00.1<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;»&nbsp; **{binfile}** is the file name of the Aupera firmware QSPI flash dump file in the directory {NFS_ABS_PATH}/firmware/<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;»&nbsp; **Flash another xu30 device with the second card_id read from lspci (like 08:00:1) using the same flash dump binfile**

<b> An example of the command lines to flash both the xu30 devices: </b>

```bash
sudo /opt/xilinx/xrt/bin/xbmgmt flash --shell --card 07:00.1 --path /opt/aupera/face-recognition/firmware/xu30-qspi-burn-fr-mtd.bin
sudo /opt/xilinx/xrt/bin/xbmgmt flash --shell --card 08:00.1 --path /opt/aupera/face-recognition/firmware/xu30-qspi-burn-fr-mtd.bin
```

---

<b> After flash, cold reboot server. Please do not use ‘sudo poweroff’ or ‘sudo reboot’ command. </b>

---

## 3.2 Install U30 Driver

```bash
cd {NFS_ABS_PATH}/driver
sudo ./install.sh
```

<b> Note : Repeat this step ( 4.2 Install U30 driver ) everytime post cold reboot of the server. </b>

---

# 4. Run Docker

## 4.1 Copy Access Key & License Configuration File to DRM Path

+ Refer to **[Section 1](#Section-1)** above to generate Access Key File (cred.json) and copt to `{NFS_ABS_PATH}/drm` path
+ Choose suitable License configuration file ( Either Floating Licence or Nodelocked Licence ) & copy to `{NFS_ABS_PATH}/drm` path  

For Floating Licence Configuration :

```bash
sudo cp {NFS_ABS_PATH}/drm/floating/conf.json {NFS_ABS_PATH}/drm/conf.json
```

For Nodelocked Licence Configuration : 
```bash
sudo cp {NFS_ABS_PATH}/drm/nodelocked/conf.json {NFS_ABS_PATH}/drm/conf.json
```
---

## 4.2 Docker Run

```bash
sudo docker run -dit --name {CONTAINER_NAME} -v {NFS_ABS_PATH}:{NFS_ABS_PATH} -e NFS_ABS_PATH={NFS_ABS_PATH} -p 56108:56108 aupera_face_recognition:3.0.2 bash
```

<b> An example of the command line: </b>
```bash
sudo docker run -dit --name face_recognition -v /opt/aupera/face-recognition/:/opt/aupera/face-recognition/ -e NFS_ABS_PATH=/opt/aupera/face-recognition/ -p 56108:56108 aupera_face_recognition:3.0.2 bash
```
---

## 4.3 Start Face Recognition Service

```bash
sudo docker container exec -it {CONTAINER_NAME} bash start.sh
```

<b> An example of the command line: </b>
```bash
sudo docker container exec -it face_recognition bash start.sh
```

---

# 5. Install the Client Software on Windows_Client_PC
### Perform below steps on Windows_Client_PC : 
+ Download **Aupera Face Recognition Client Software** from [Aupera Download Page](https://auperatechnologies.com/downloads/)
+ Extact the downloaded **dist.zip** file & go to the path : **dist\dist\client**
+ Run **client.exe** Application. This will setup Client Software on Windows_Client_PC

For detailed instructions please refer to the section 5 of [Aupera_FR_U30_User guide.](https://www.xilinx.com/content/dam/xilinx/publications/user-guide/partner/aupera-user-guide.pdf) </br></br>
<b> Note : Turn off firewall settings that are blocking Client-Server communication. ( Eg : McAfee Endpoint Security -> Firewall -> OPTIONS : Untick 'Enable Firewall' ) 

---

# 6. Perform the Test
### There are two ways to perform the Test: 

+ Connect IP camera to setup live streaming
+ Upload pre-recorded video clips & stream through FFMPEG RTSP

### Detailed Instructions are included in [Aupera_FR_U30_User guide](https://www.xilinx.com/content/dam/xilinx/publications/user-guide/partner/aupera-user-guide.pdf)
+ For instructions to connect IP Camera, please refer to section 5.3.1.1 <br>
+ For instructions to Upload pre-recorded video clip & stream through FFMPEG RTSP, please refer to 5.5 Frequent Q&A (3) & (4)

---

# 7. Results
Once you complete above setting, you should be able to click ‘View Live Result’ on the in the Aupera’s Windows Client Software and view the face recognition result.
![Results](assets/aupera_facial_recognition/results.png)
