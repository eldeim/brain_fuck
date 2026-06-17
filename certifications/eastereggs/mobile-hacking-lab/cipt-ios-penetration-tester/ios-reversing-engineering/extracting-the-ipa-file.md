---
description: Guide to Pulling an IPA File from a Jailbroken iOS Device
---

# Extracting the IPA file

### Prerequisites <a href="#el_1726937764992_339" id="el_1726937764992_339"></a>

* **Jailbroken iOS Device**: A jailbroken device is required to access and extract app files.
* **Wi-Fi Network**: Your iOS device and your computer should be on the same Wi-Fi network for SSH access.
* **Filza File Manager (optional)**: Useful for exploring and finding app paths on the device.
* **Target App**: The **DVIA-v2** app should be installed on your device. Download it from the GitHub repository [prateek147/DVIA-v2](https://github.com/prateek147/DVIA-v2).

## Part 1: Find the App's Bundle Identifier and Directory <a href="#el_1726937770741_349" id="el_1726937770741_349"></a>

To extract the IPA, you first need to locate the app's directory on your device.

### Method 1: Using Filza File Manager <a href="#el_1726937773522_359" id="el_1726937773522_359"></a>

1\. Install **Filza** from **Cydia** or **Sileo** (if not already installed).\
2\. Open **Filza** and navigate to the **/var/containers/Bundle/Application/** directory.\
3\. Inside, you will see multiple folders named with UUIDs (random alphanumeric characters). Each of these contains an app's **.app** bundle.\
4\. Instead of tapping into each folder, you’ll notice that **Filza** already resolves the app names next to the folder, making it easy to identify the app (e.g., **DVIA-v2.app**) without opening each folder.

### Method 2: Using SSH to List Installed Apps <a href="#el_1726937780143_369" id="el_1726937780143_369"></a>

You can use SSH to directly list all installed apps and their paths.\
1\. **Open a terminal and SSH into your device**:

```
ssh root@10.11.1.1
```

2\. **Run the following command to list all installed app directories**:

```
find /var/containers/Bundle/Application/ -name "*.app"
```

This will list the UUID directories, and inside each directory, you will find the app bundles (e.g., **DVIA-2.app**).\
Take note of the app's directory (e.g., **/var/containers/Bundle/Application/70961402-4C6E-4FF6-B316-59D3ED83828F/DVIA-v2.app**).

***

## Part 2: Packaging the App into an IPA <a href="#el_1726938172103_446" id="el_1726938172103_446"></a>

> An **IPA** is essentially a zipped archive containing the app's **.app** folder and a **Payload** directory.

### Step 1: Create the Payload Directory <a href="#el_1726938517198_650" id="el_1726938517198_650"></a>

SSH into the device and navigate to a writable directory (e.g., **/tmp/**):

```
cd /tmp/
```

Create a directory named **Payload**:

```
mkdir Payload
```

Copy the app bundle into the **Payload** directory:

```
cp -r /var/containers/Bundle/Application/70961402-4C6E-4FF6-B316-59D3ED83828F/DVIA-v2.app /tmp/Payload/
```

> _Note: Replace the UUID to match the UUID on your device._

### Step 2: Compress the Payload Directory into an IPA <a href="#el_1726938323649_509" id="el_1726938323649_509"></a>

Navigate to **/tmp/**:

```
cd /tmp/
```

Compress the **Payload** folder into a **.zip** file:

```
zip -r DVIA-v2.ipa Payload
```

You should now have an IPA file located at **/tmp/DVIA-v2.ipa**.

***

## Part 3: Transferring the IPA to Your Computer <a href="#el_1726938357979_534" id="el_1726938357979_534"></a>

Now that you have the IPA file, you need to transfer it to your computer.\
On your computer, use the **scp** command to copy the IPA file from the iOS device to your computer.

```
scp root@10.11.1.1:/tmp/DVIA-v2.ipa ~/Downloads/
```

This command copies the IPA file from the device to your downloads.

## Part 4: Clean Up Temporary Files <a href="#el_1726938428413_590" id="el_1726938428413_590"></a>

After transferring the IPA, it’s a good idea to remove the temporary files from your device.1. SSH into your device (if not already connected).2. Remove the **Payload** folder and the IPA file from **/tmp/**:

```
rm -rf /tmp/Payload /tmp/DVIA-v2.ipa
```
