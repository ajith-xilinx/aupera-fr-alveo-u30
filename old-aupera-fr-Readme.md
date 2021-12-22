[[[ALVEO_HOST_SETUP_SECTION]]]

---

# 2. U30 Face Recognition Docker Installation
## 2.1 Prerequisites
+ One U30 board with two xcu30 devices.
+ One x86 Ubuntu 18.04 machine supports one PCIe x8 as dual x4 or PCIe x16 as 4x4 bifurcation for U30 This document will call this computer as the “X86_Host_Computer”
+ Install Aupera released face recognition docker image using docker pull command ‘docker pull hubxilinx/auperatech_face_recog_alveo_u30’ in X86_Host_Computer.
+ One windows machine hosts client software, for operation interaction and results viewing. This document will call this computer as the “Windows_Client_PC”

Enter the following commands in a terminal window to run the application:

---

## 2.2 Procedure
Install Aupera Face Recognition Docker image
+ Prepare essential software and other related packages:

```bash
$sudo apt update;sudo apt install make build-essential nfs-kernel-server docker docker-containerd docker.io
$sudo service rpcbind restart
$sudo service nfs-kernel-server restart
```

---

+ Pull Docker image and check:

```bash
$docker pull xilinxpartners/aupera_face_recognition:2.0.1
$docker images aupera_face_recognition
```

---

+ Copy firmware and driver from Docker image

```bash
$docker create --name {CONTAINER_NAME} xilinxpartners/aupera_face_recognition:2.0.1 bash
$docker cp {CONTAINER_NAME}:/root/driver {NFS_ABS_PATH}
$docker cp {CONTAINER_NAME}:/root/firmware {NFS_ABS_PATH}
```

Here **{CONTAINER_NAME}** is a user defined container name, like face, **{REPOSITORY}:{TAG}** is the repository name, like aupera_face_recognition:2.0.1, **{NFS_ABS_PATH}** is local directory where firmware and driver will copied to, like /opt/aupera/face-recognition.

---

## 2.3 Install U30 firmware
+ Source XRT env and check the current XRT version.
Currently XRT version 2.6.655 or 2.6.0 are required for the firmware installation.

```bash
$cd /opt/xilinx/xrt/
$source setup.sh
$ xbutil --version
XCLMGMT: 2.6.655
```

---

+ Run lspci command to validate the U30 board seen by the OS

```bash
$sudo lspci -d 10ee:
07:00.0 Processing accelerators: Xilinx Corporation Device 503d (rev 02)
07:00.1 Processing accelerators: Xilinx Corporation Device 503c (rev 02)
08:00.0 Processing accelerators: Xilinx Corporation Device 503d (rev 02)
08:00.1 Processing accelerators: Xilinx Corporation Device 503c (rev 02)
```
Two devices and four functions will be found by the OS if the card is successfully installed.

---

+ Flash the U30 board using XRT xbmgmt utility:

```bash
$sudo /opt/xilinx/xrt/bin/xbmgmt flash --shell --card {card_id} --path {binfile}.bin
```

Where, {card_id} is the BDF ID read from lspci, like 07:00.1, and {binfile} is the file name of the Aupera firmware QSPI flash dump file in the directory {NFS_ABS_PATH}/firmware/. After completion, flash another one with the second card_id (like 08:00:1) read from lspci and the same flash dump file.

---

+ After flash, cold reboot server, please do not use ‘sudo poweroff’ or ‘sudo reboot’ command.

---

# 3. Install U30 driver

```bash
$cd {NFS_ABS_PATH}/driver
$sudo ./install.sh
```

---

# 4. Run Docker
+ Setup Environment Variables

```bash
$git clone https://raw.githubusercontent.com/Xilinx/Xilinx_Base_Runtime
$source Xilinx_Base_Runtime/utilities/xilinx_docker_setup.sh
```

---

+ Docker Run

```bash
$docker run -dit --name {CONTAINER_NAME} $XILINX_DOCKER_DEVICES -v {NFS_ABS_PATH}:{NFS_ABS_PATH} -e NFS_ABS_PATH={NFS_ABS_PATH} -p 56108:56108 {REPOSITORY}:{TAG} bash
```

An example of the command line:
```bash
$docker run -dit --name face $XILINX_DOCKER_DEVICES -v /opt/aupera/face-recognition/:/opt/aupera/face-recognition/ -e NFS_ABS_PATH=/opt/aupera/face-recognition/ -p 56108:56108 aupera_face_recognition:2.0.1 bash
```

---

+ Refer to above section 1 to generate a license file (cred.json) and choose a configuration file (conf.json), copy them to {NFS_ABS_PATH}/drm.

---

+ Start Face Recognition service

```bash
$docker container exec -it {CONTAINER_NAME} bash start.sh
```

An example of the command line:
```bash
$docker container exec -it face bash start.sh
```

---

# 5. Install the client software on Windows_Client_PC
Detailed instructions please refer to the section 5 of [Aupera_FR_U30_User guide.](https://www.xilinx.com/content/dam/xilinx/publications/user-guide/partner/aupera-user-guide.pdf)

---

# 6. Perform the Test
Connect camera to setup live streaming or upload video clips for testing.
Detailed instructions please refer to the section 5.5 QA 3 and 4 of [Aupera_FR_U30_User guide.](https://www.xilinx.com/content/dam/xilinx/publications/user-guide/partner/aupera-user-guide.pdf)

---

# 7. Results
Once you complete above setting, you should be able to click ‘View Live Result’ on the in the Aupera’s Windows Client Software and view the face recognition result.
![Results](assets/aupera_facial_recognition/results.png)
