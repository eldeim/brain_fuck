# Interacting with iOS Devices

<figure><img src="../../../../../../.gitbook/assets/image (1443).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../../.gitbook/assets/image (1444).png" alt=""><figcaption></figcaption></figure>

***

## Interaction via SSH + libimobiledevice <a href="#el_1726216875237_340" id="el_1726216875237_340"></a>

#### Prerequisites <a href="#el_1726217380186_371" id="el_1726217380186_371"></a>

* **A Jailbroken iOS Device**: SSH access requires a jailbroken iOS device,
* **Network access**: Both your iOS device and the computer should be on the same network, via VPN (for a virtual device) or Wi-Fi (for a physical device).
* **USB device access**, as explained in the device setup sections.<br>
* **libimobiledevice tools**: Install **libimobiledevice** on your host machine
* **Target App**: The **DVIA-v2** app is recommended. Download it from the GitHub repository [prateek147/DVIA-v2](https://github.com/prateek147/DVIA-v2).

### libimobiledevice installation <a href="#el_1727986108639_397" id="el_1727986108639_397"></a>

Follow the instructions on [https://libimobiledevice.org/#get-started](https://libimobiledevice.org/#get-started) to install the toolset, which are in short:

#### Linux (Ubuntu)

```
sudo apt-get install usbmuxd libimobiledevice6 libimobiledevice-utils
```

#### MacOS <a href="#el_1727986186892_467" id="el_1727986186892_467"></a>

```
brew install libimobiledevice
```

### Setup SSH access <a href="#el_1727986429955_559" id="el_1727986429955_559"></a>

#### Virtual Mobile Hacking Lab Device <a href="#el_1727986444493_571" id="el_1727986444493_571"></a>

Setup a VPN connection as explained in the [Mobile Hacking Lab Device setup](https://www.mobilehackinglab.com/path-player?courseid=ios-appsec\&unit=66dee0a8e0d02c8199018fe9).\
\
Connect to the device via SSH from a terminal (IP might be different regarding the last .1):

```
ssh root@10.11.1.1
```

#### Physical device via iproxy <a href="#el_1727986600461_605" id="el_1727986600461_605"></a>

iproxy forwards a local TCP port to your iOS device over USB,enabling SSH access without Wi-Fi.Forward port 22 (SSH) from the device to your computer’s local port 2222, and connect via localhost:

```
iproxy 2222 22
```

```
ssh root@localhost -p 2222
```

### Part 1: Common SSH and libimobiledevice Commands for iOS <a href="#el_1726217500257_393" id="el_1726217500257_393"></a>

#### 1. File System Exploration via SSH <a href="#el_1726218211364_586" id="el_1726218211364_586"></a>

Once connected to the iOS device via SSH, you can explore the file system using standard Unix commands.\
<br>

* **List directories**:

```
ls -la
```

* **Change directory**:

```
cd /var/mobile
```

* **Find the application bundle directory of all (user) installed apps:**

```
find /private/var/containers/Bundle/Application/ -name "*.app"
```

* **Copy files from the device to your computer**:

```
scp root@10.11.1.1:/etc/master.passwd ~/Downloads/
```

* **Upload files to the iOS device**:

```
scp ./payload.txt root@10.11.1.1:/var/mobile/Documents/
```

### Part 2: Using libimobiledevice Tools <a href="#el_1726217532447_403" id="el_1726217532447_403"></a>

#### 1. ideviceinfo: Retrieve Device Information <a href="#el_1726218467360_760" id="el_1726218467360_760"></a>

This command fetches detailed information about the iOS device, such as hardware details and the iOS version.

```
ideviceinfo
```

#### 2. ideviceinstaller: Install or Uninstall Apps <a href="#el_1726218489179_781" id="el_1726218489179_781"></a>

Manage apps on the iOS device using the following commands:\
<br>

* **List installed apps**:

```
ideviceinstaller --list-apps
```

* **Install an app (IPA)**:

```
ideviceinstaller --install ./DVIA-v2.ipa
```

* **Uninstall an app:**

```
ideviceinstaller --uninstall com.highaltitudehacks.DVIAswiftv2
```

#### 3. idevicebackup2: Create Backups of the iOS Device <a href="#el_1726218568983_862" id="el_1726218568983_862"></a>

Use this tool to back up or restore your iOS device:<br>

* **Create a full backup (with encryption enabled)**:

```
idevicebackup2 encryption on "[PASSWORD]"
```

```
idevicebackup2 backup --full ~/Backups/
```

> _NOTE: Backing up a Corellium device is not supported, restoring a backup to one is._

* **Restore a backup**:

```
idevicebackup2 restore \
```

```
  --system \
```

```
  --settings \
```

```
  --password "[PASSWORD]" \
```

```
  ~/Backups/
```

* **View backup information**:

```
idevicebackup2 info ~/Backups/
```

#### 4. idevicesyslog: View iOS System Logs <a href="#el_1726218648097_916" id="el_1726218648097_916"></a>

Stream system logs in real-time for monitoring or debugging app behavior:

```
idevicesyslog
```

#### 5. idevicecrashreport: Retrieve Crash Reports <a href="#el_1726218672147_954" id="el_1726218672147_954"></a>

Download crash reports for detailed app crash analysis.<br>

* **Download a copy of all crash reports**:

```
idevicecrashreport --keep ~/Reports/
```

### Part 3: Automating Tasks with SSH and libimobiledevice <a href="#el_1726217754762_430" id="el_1726217754762_430"></a>

You can automate tasks such as backups and file extraction using scripts. Here’s an example that backs up the iOS device and extracts data from an app’s directory.

#### Example: Backup and Data Extraction Script <a href="#el_1726218136594_536" id="el_1726218136594_536"></a>

```bash
#!/bin/bash

BACKUP_PATH="$HOME/Device/Backups/"
EXTRACT_PATH="$HOME/Device/Data/"
APP_UID="5CAF9854-AE84-4ABB-A856-5DE570E96171"

# Create a backup
echo "Creating backup..."
idevicebackup2 backup "$BACKUP_PATH"

# Extract data from app directory
echo "Extracting data from app ($APP_UID)..."
scp -r root@10.11.1.1:/var/mobile/Containers/Data/Application/$APP_UID/ "$EXTRACT_PATH"
```

This script automates the backup process and pulls app data for analysis.
