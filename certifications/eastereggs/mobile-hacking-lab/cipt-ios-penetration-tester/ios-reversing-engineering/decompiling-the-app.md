---
description: Guide to Decompiling an IPA File Using IPSW Tools (class-dump and swift-dump)
---

# Decompiling the App

## Part 1: Install the IPSW Toolset and Swift <a href="#el_1727116005491_372" id="el_1727116005491_372"></a>

You can install the **IPSW Toolset** using **Homebrew** on macOS. Follow the steps below to set it up.

### Step 1: Install IPSW Toolset <a href="#el_1727116014153_392" id="el_1727116014153_392"></a>

You can install the IPSW Toolset with the command below:\
\
**MacOS**:

```
brew install blacktop/tap/ipsw
```

**Ubuntu**:

```
sudo snap install ipsw
```

This installs the IPSW Toolset, including **class-dump** and **swift-dump**.

### Step 2: Verify IPSW Toolset Installation <a href="#el_1727116023875_407" id="el_1727116023875_407"></a>

Check if IPSW was installed successfully by running:

```
ipsw --help
```

This command should display a list of available commands, such as **class-dump** and **swift-dump**.

### Step 3: Install Swift <a href="#el_1730391143658_393" id="el_1730391143658_393"></a>

You can install the Swift with the instructions below:\
**MacOS**:

* Install **XCode** (which bundles Swift) or use the standalone [macOS Package Installer](https://www.swift.org/install/macos/package_installer/).

\
**Ubuntu**:

```
sudo apt install -y curl
```

```
curl -L https://swiftlygo.xyz/install.sh | bash
```

```
sudo swiftlygo install latest
```

### Step 4: Verify Swift Installation <a href="#el_1730392076395_470" id="el_1730392076395_470"></a>

Check if Swift was installed successfully by running:

```
swift --help
```

This command should display a list of available commands, such as **build** and **run**.

***

## Part 2: Extract the IPA File <a href="#el_1727116036249_427" id="el_1727116036249_427"></a>

The next step is to extract the contents of your IPA file.

### Step 1: Unzip the IPA File <a href="#el_1727116111850_471" id="el_1727116111850_471"></a>

An **IPA** is essentially a **ZIP archive** containing the app’s code and resources. You can extract it using **unzip**:\
1\. Navigate to the directory where your IPA is located and run:

```
unzip ./DVIA-v2.ipa
```

This extracts the contents of the IPA to the specified output directory. The extracted contents will include a folder named **Payload** which contains the **.app** directory for the app.

### Step 2: Locate the App Binary <a href="#el_1727116133283_491" id="el_1727116133283_491"></a>

Inside the **Payload** folder, you'll find the **.app** folder, which contains the app’s binary (usually a file without an extension). And the app binary is located inside the **.app** folder.\
\
For example, if the app is called **DVIA-v2**, the binary would be found here:

```
./Payload/DVIA-v2.app/DVIA-v2
```

This file is the main binary that you will use for decompiling.

***

## Part 3: Dumping Objective-C Classes Using class-dump <a href="#el_1727116268659_568" id="el_1727116268659_568"></a>

Now that you've located the app binary, you can use **class-dump** to extract the **Objective-C** classes.

### Step 1: Run class-dump on the App Binary <a href="#el_1727116270867_573" id="el_1727116270867_573"></a>

Use the **ipsw class-dump** command to dump the Objective-C classes from the binary:

```
ipsw class-dump ./Payload/DVIA-v2.app/DVIA-v2 --headers -o ./class_dump
```

This command extracts all the **Objective-C class headers** from the app binary and saves them in the specified output directory.

### Step 2: Review Dumped Headers <a href="#el_1727116291663_610" id="el_1727116291663_610"></a>

Once the **class-dump** process is complete, you can navigate to the output directory and review the extracted header files, which will contain information about the app's Objective-C classes, methods, and properties.

> _Note: we will do in-depth analysis in the next workbook:_ [_Analyzing the App_](https://academy.mobilehackinglab.com/path-player?courseid=ios-appsec\&unit=66ded39f2bae13a9b9093c22)

***

## Part 4: Dumping Swift Classes Using swift-dump <a href="#el_1727116301199_620" id="el_1727116301199_620"></a>

If the app is written in **Swift**, you can use **swift-dump** to extract and demangle Swift class and method names.

### Step 1: Run swift-dump on the App Binary <a href="#el_1727116303948_625" id="el_1727116303948_625"></a>

Use the **ipsw swift-dump** command to extract Swift class information from the binary:\
\
**macOS**:

```
ipsw swift-dump ./Payload/DVIA-v2.app/DVIA-v2 > ./swift_dump_mangled.txt
```

```
ipsw swift-dump ./Payload/DVIA-v2.app/DVIA-v2 --demangle > ./swift_dump_demangled.txt
```

**Ubuntu**:

```
SWIFT_DEMANGLE="$(find /usr/libexec -type f -executable -name "swift-demangle")"
```

```
​
```

```
ipsw swift-dump ./Payload/DVIA-v2.app/DVIA-v2 > ./swift_dump_mangled.txt
```

```
ipsw swift-dump ./Payload/DVIA-v2.app/DVIA-v2 | $SWIFT_DEMANGLE --simplified > ./swift_dump_demangled.txt
```

This command extracts all **Swift classes** and mangled and demangled method names, saving them in the specified file.

> _Note: At the time of writing the tool is still heavily under development and some features are not implemented/working yet._

### Step 2: Review Demangled Swift Symbols <a href="#el_1727116313968_640" id="el_1727116313968_640"></a>

Once **swift-dump** finishes running, go to the output directory and review the dumped Swift symbols. The demangled symbols will include class names, method names, and other Swift metadata that you can analyze.

> _Note: we will do in-depth analysis in the next workbook:_ [_Analyzing the App_](https://academy.mobilehackinglab.com/path-player?courseid=ios-appsec\&unit=66ded39f2bae13a9b9093c22)

***

## Part 5: Automating the Process for a Single IPA File <a href="#el_1727116321632_650" id="el_1727116321632_650"></a>

If you want to automate this process for a single IPA file, you can use the following script:

```bash
#!/bin/bash

# Check if an IPA file was provided
if [ -z "$1" ]; then
  echo "Usage: $0 <path_to_ipa_file>"
  exit 1
fi

IPA_FILE="$1"

# Check if the IPA file exists
if [ ! -f "$IPA_FILE" ]; then
  echo "[@] Error: IPA file not found!"
  exit 1
fi

# Get the app name from the IPA file
APP_NAME="$(basename ""$IPA_FILE"" .ipa)"
OUTPUT_DIR="$(dirname ""$IPA_FILE"" | xargs readlink -f)"

# Create output directory
OUTPUT_DIR="$OUTPUT_DIR/$APP_NAME"
mkdir -p "$OUTPUT_DIR"

# Unzip the IPA contents
UNZIP_DIR="$OUTPUT_DIR/_extracted"
echo "[*] Extracting IPA contents..."
mkdir -p "$UNZIP_DIR"
unzip -q "$IPA_FILE" -d "$UNZIP_DIR"

# Locate the .app directory
APP_PATH=$(find "$UNZIP_DIR" -name "*.app" -type d)

if [ -z "$APP_PATH" ]; then
  echo "[@] No .app found in $UNZIP_DIR, exiting..."
  exit 1
fi

BINARY="$APP_PATH/$(basename ""$APP_PATH"" .app)"

# Check if the binary exists (file without an extension in the .app folder)
if [ ! -f "$BINARY" ]; then
  echo "[@] No binary found in $APP_PATH, exiting..."
  exit 1
fi

# Create directories for class dumps
CLASS_DUMP_OUTPUT="$OUTPUT_DIR/class_dump"
SWIFT_DUMP_OUTPUT="$OUTPUT_DIR/swift_dump"
mkdir -p "$CLASS_DUMP_OUTPUT"
mkdir -p "$SWIFT_DUMP_OUTPUT"

# Dump Objective-C classes using class-dump
echo "[*] Dumping Objective-C classes for $APP_NAME..."
ipsw class-dump "$BINARY" --headers -o "$CLASS_DUMP_OUTPUT"

# Dump Swift classes using swift-dump
echo "[*] Dumping Swift classes for $APP_NAME..."
ipsw swift-dump "$BINARY" > "$SWIFT_DUMP_OUTPUT/$APP_NAME-mangled.txt"
ipsw swift-dump "$BINARY" --demangle > "$SWIFT_DUMP_OUTPUT/$APP_NAME-demangled.txt"

echo "[+] Decompilation completed for $APP_NAME"
```

> Note: This script automates the process of dumping both **Objective-C** and **Swift** classes for apps.
