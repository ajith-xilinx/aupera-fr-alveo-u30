## Prerequisites
+ [Alveo U30 Data Center Accelerator Card](https://www.xilinx.com/products/boards-and-kits/alveo/u30.html) ( The card includes two xu30 devices )
+ x86 Ubuntu 18.04 machine. This document will call this computer as the “X86_Host_Computer”. ( The server need to support bifurcation to take advantage of both xu30 Zynq® UltraScale+™ devices on the Alveo U30 card ) 
+ Xilinx Appstore Access Key & Face Recognition Product License file
+ Xilinx Runtime (XRT) host application
+ Aupera Face Recognition Docker Image
+ Aupera Client Software, for operation interaction and results viewing 
+ Windows PC to install Aupera Client Software. This document will call this computer as the “Windows_Client_PC”

---

## <a name="Section-1"></a> 1. Xilinx Appstore Account & Face Recognition Product Access Key 

+ Create an account on [Xilinx Appstore](https://appstore.xilinx.com/) 
+ Obtain an entitlement to evaluate the Aupera Facial Recognition product. See "Try or Buy" Section at the Top of this page. 
  + **Free Trial** is also available
  + The product can be purchased with a floating or node-locked license 
  + Download the License Configuration File ( conf.json ), this file will be used in install procedure later
+ Create **cred.json** file ( Access Key ) for your account 
  + Login to [Xilinx Appstore Page](https://appstore.xilinx.com/) -> Access Key -> Create an Access Key -> Download JSON 
  + This file identifies your account to the Appstore during runtime and will be used in install procedure later

---

## 2. Host Setup - XRT Installation 

### 2.1 Clone the Xilinx Base Runtime GitHub Repository:

```bash
git clone https://github.com/Xilinx/Xilinx_Base_Runtime.git 
cd Xilinx_Base_Runtime
```
---
### 2.2 Run Host Setup Script:


```bash
sudo ./host_setup.sh -v 2021.1 --skip-shell-flash
```
---

## 3. U30 Face Recognition Docker Installation

Install Aupera Face Recognition Docker image

### 3.1 Install essential software and other related packages:


```bash
sudo apt update; sudo apt install make build-essential nfs-kernel-server docker docker-containerd docker.io
sudo service rpcbind restart
sudo service nfs-kernel-server restart
```
---

### 3.2 Pull Auper Face Recognition Docker Image:

```bash
sudo docker pull xilinxpartners/aupera_face_recognition:3.0.2
sudo docker images | grep aupera_face_recognition
```
---

### 3.3 Copy firmware and driver from Docker image


```bash
sudo docker create --name {CONTAINER_NAME} xilinxpartners/aupera_face_recognition:3.0.2 bash
sudo docker cp {CONTAINER_NAME}:/root/driver {NFS_ABS_PATH}
sudo docker cp {CONTAINER_NAME}:/root/firmware {NFS_ABS_PATH}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;»&nbsp; **{CONTAINER_NAME}** is a user defined container name, like "face_recognition"<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;»&nbsp; **{NFS_ABS_PATH}** is local directory where firmware and driver will copied to, like "/opt/aupera/face-recognition"

#### An example of the command line:
```bash
sudo docker create --name face_recognition xilinxpartners/aupera_face_recognition:3.0.2 bash
sudo docker cp face_recognition:/root/driver /opt/aupera/face-recognition
sudo docker cp face_recognition:/root/firmware /opt/aupera/face-recognition
```

---

## 4. Setting up Alveo U30 Card
### 4.1 Install U30 firmware

---

#### Source XRT env and check the current XRT version. Currently XRT version 2.11 or above are required for the firmware installation.

```bash
source /opt/xilinx/xrt/setup.sh
xbutil --version
```
This will give the output similar to:
```
XCLMGMT: 2.11.634
```

---

#### Run lspci command to validate the U30 board seen by the OS

```bash
sudo lspci -d 10ee:
```
This will give the output similar to:
```bash
07:00.0 Processing accelerators: Xilinx Corporation Device 503d (rev 02)
07:00.1 Processing accelerators: Xilinx Corporation Device 503c (rev 02)
08:00.0 Processing accelerators: Xilinx Corporation Device 503d (rev 02)
08:00.1 Processing accelerators: Xilinx Corporation Device 503c (rev 02)
```
+ Two xu30 devices and four functions will be found by the OS if the card is successfully installed.

---

#### Flash the U30 board using XRT xbmgmt utility:

```bash
sudo /opt/xilinx/xrt/bin/xbmgmt flash --shell --card {card_id} --path {binfile}.bin
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;»&nbsp; **{card_id}** is the BDF ID read from lspci, like 07:00.1<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;»&nbsp; **{binfile}** is the file name of the Aupera firmware QSPI flash dump file in the directory {NFS_ABS_PATH}/firmware/<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;»&nbsp; **After completion, flash another xu30 device with the second card_id (like 08:00:1) read from lspci and the same flash dump file**

#### An example of the command lines to flash both the xu30 devices:

```bash
sudo /opt/xilinx/xrt/bin/xbmgmt flash --shell --card 07:00.1 --path /opt/aupera/face-recognition/firmware/xu30-qspi-burn-fr-mtd.bin
sudo /opt/xilinx/xrt/bin/xbmgmt flash --shell --card 08:00.1 --path /opt/aupera/face-recognition/firmware/xu30-qspi-burn-fr-mtd.bin
```

---

#### After flash, cold reboot server. Please do not use ‘sudo poweroff’ or ‘sudo reboot’ command.

---

### 4.2 Install U30 driver

```bash
cd {NFS_ABS_PATH}/driver
sudo ./install.sh
```

#### Note : Repeat this step ( 4.2 Install U30 driver ) everytime post cold reboot of the server

---

## 5. Run Docker

### 5.1 Copy Access Key & License Configuration Files to DRM Path

+ **Refer to [Section 1](#Section-1) above to generate Access Key File (cred.json) and choose a License Configuration File (conf.json)**<br>
+ Copy both the files to {NFS_ABS_PATH}/drm.

---

### 5.2 Docker Run

```bash
sudo docker run -dit --name {CONTAINER_NAME} -v {NFS_ABS_PATH}:{NFS_ABS_PATH} -e NFS_ABS_PATH={NFS_ABS_PATH} -p 56108:56108 aupera_face_recognition:3.0.2 bash
```

#### An example of the command line:
```bash
sudo docker run -dit --name face_recognition -v /opt/aupera/face-recognition/:/opt/aupera/face-recognition/ -e NFS_ABS_PATH=/opt/aupera/face-recognition/ -p 56108:56108 aupera_face_recognition:3.0.2 bash
```
---

### 5.3 Start Face Recognition Service

```bash
sudo docker container exec -it {CONTAINER_NAME} bash start.sh
```

#### An example of the command line:
```bash
sudo docker container exec -it face_recognition bash start.sh
```

---

## 6. Install the client software on Windows_Client_PC
#### Perform the below steps on Windows_Client_PC : 
+ Download **Aupera Face Recognition Client Software** from [Aupera Download Page](https://auperatechnologies.com/downloads/)
+ Extact the downloaded **dist.zip** file & go to the path : **dist\dist\client**
+ Run **client.exe** Application. This will setup Client Software on Windows_Client_PC

For detailed instructions please refer to the section 5 of [Aupera_FR_U30_User guide.](https://www.xilinx.com/content/dam/xilinx/publications/user-guide/partner/aupera-user-guide.pdf)

---

## 7. Perform the Test
#### There are two ways to perform the Test: 

+ Connect IP camera to setup live streaming
+ Upload pre-recorded video clips & stream through FFMPEG RTSP

#### Detailed Instructions are included in [Aupera_FR_U30_User guide](https://www.xilinx.com/content/dam/xilinx/publications/user-guide/partner/aupera-user-guide.pdf)
+ For instructions to connect IP Camera, please refer to section 5.3.1.1 <br>
+ For instructions to Upload pre-recorded video clip & stream through FFMPEG RTSP, please refer to 5.5 Frequent Q&A (3) and (4)

---

## 8. Results
Once you complete above setting, you should be able to click ‘View Live Result’ on the in the Aupera’s Windows Client Software and view the face recognition result.
![Results](assets/aupera_facial_recognition/results.png)
