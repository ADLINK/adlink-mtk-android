# Android 16 for ADLINK OSM-MTK520

## Preparation

The build environment is based on **Ubuntu 22.04 LTS (64-bit)**. Follow the steps below to configure the host system.

### 1. Enable 32-bit Architecture & Add PPAs
```shell
$ sudo dpkg --add-architecture i386
$ sudo apt-get update && sudo apt-get install -y software-properties-common
$ sudo add-apt-repository -y ppa:ubuntu-toolchain-r/test
$ sudo add-apt-repository -y ppa:deadsnakes/ppa
```

### 2. Installing Dependency Packages
```shell
$ sudo apt-get update
$ sudo apt-get install -y \
    gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential \
    chrpath socat cpio python3 python3-pip python3-pexpect xz-utils \
    debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa \
    libsdl1.2-dev pylint xterm flex bison libncurses5-dev libncurses5 \
    libstdc++6 zlib1g:i386 zlib1g-dev:i386 python2 sudo vim curl \
    gnupg rsync zip
```

### 3. Configure Python Alternatives
The MediaTek Android build system requires `python` to point to `python2.7`:
```shell
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 2
$ sudo update-alternatives --set python /usr/bin/python2.7
```

### 4. Install Repo Tool
```shell
$ sudo curl https://storage.googleapis.com/git-repo-downloads/repo > /usr/local/bin/repo
$ sudo chmod a+x /usr/local/bin/repo
```

### 5. Setup GIT
```shell
$ git config --global user.name "First Last"
$ git config --global user.email "first.last@company.com"
```

## Download Android source from MediaTek and patches from ADLINK GitHub

Contact MediaTek to obtain the Android 16 SDK.
The SDK package consists of downloading:
* `alps-release-b0.mp5-aiot-V4.82`
* `alps-release-u0.mp5-aiot-V7.174`

Clone the ADLINK MTK Android repository containing the BSP patches:
```shell
$ cd ${HOME}
$ git clone https://github.com/ADLINK/adlink-mtk-android.git -b Android-16
```

## Apply OSM-MTK520 Patches

Apply the patches provided in the `Patches` directory to the respective source trees:

### 1. alps-release-b0.mp5-aiot-V4.82

#### 1.1 External (spidevtest support)
```shell
$ cd ${HOME}/alps-release-b0.mp5-aiot-V4.82/external
$ git apply ${HOME}/adlink-mtk-android/Patches/osm-mtk520/alps-release_b0-V4.82_u0-V7.174/alps-release-b0.mp5-aiot-V4.82/external/0001-osm-mtk520-build-external-add-spidevtest-support.patch
```

#### 1.2 Build (update build ID and include spidevtest)
```shell
$ cd ${HOME}/alps-release-b0.mp5-aiot-V4.82/build
$ git apply ${HOME}/adlink-mtk-android/Patches/osm-mtk520/alps-release_b0-V4.82_u0-V7.174/alps-release-b0.mp5-aiot-V4.82/build/0001-osm-mtk520-build-update-build-ID-and-include-spidevt.patch
```

### 2. alps-release-u0.mp5-aiot-V7.174

#### 2.1 Kernel Modules (initial board support)
```shell
$ cd ${HOME}/alps-release-u0.mp5-aiot-V7.174/kernel/kernel_device_modules-6.1
$ git apply ${HOME}/adlink-mtk-android/Patches/osm-mtk520/alps-release_b0-V4.82_u0-V7.174/alps-release-u0.mp5-aiot-V7.174/kernel/kernel_device_modules-6.1/0001-osm-mtk520-kernel-add-initial-board-support.patch
```

#### 2.2 Kernel (RTL8211 Ethernet PHY LED behavior)
```shell
$ cd ${HOME}/alps-release-u0.mp5-aiot-V7.174/kernel-6.1
$ git apply ${HOME}/adlink-mtk-android/Patches/osm-mtk520/alps-release_b0-V4.82_u0-V7.174/alps-release-u0.mp5-aiot-V7.174/kernel-6.1/0001-osm-mtk520-kernel-fix-RTL8211-Ethernet-PHY-LED-behav.patch
```

#### 2.3 Vendor MediaTek (WLAN firmware and LK2 display settings)
```shell
$ cd ${HOME}/alps-release-u0.mp5-aiot-V7.174/vendor/mediatek
$ git apply ${HOME}/adlink-mtk-android/Patches/osm-mtk520/alps-release_b0-V4.82_u0-V7.174/alps-release-u0.mp5-aiot-V7.174/vendor/mediatek/0001-osm-mtk520-add-WLAN-firmware-and-disable-LVDS-in-lk2.patch
```

#### 2.4 Device (initial board support)
```shell
$ cd ${HOME}/alps-release-u0.mp5-aiot-V7.174/device
$ git apply ${HOME}/adlink-mtk-android/Patches/osm-mtk520/alps-release_b0-V4.82_u0-V7.174/alps-release-u0.mp5-aiot-V7.174/device/0001-osm-mtk520-device-add-initial-board-support.patch
```

#### 2.5 Bluetooth HAL (correct RFKILL block/unblock state)
```shell
$ cd ${HOME}/alps-release-u0.mp5-aiot-V7.174/hardware/interfaces/bluetooth
$ git apply ${HOME}/adlink-mtk-android/Patches/osm-mtk520/alps-release_b0-V4.82_u0-V7.174/alps-release-u0.mp5-aiot-V7.174/hardware/interfaces/bluetooth/0001-osm-mtk520-android-bluetooth-correct-rfkill-block-un.patch
```

## Compile Android 16 BSP

Build the Android images using the MediaTek split build helper:

```shell
$ cd ${HOME}/alps-release-b0.mp5-aiot-V4.82
$ vendor/mediatek/proprietary/scripts/releasetools/split_build_helper.py full_aiot8391p2_64_bsp-userdebug --vf ../alps-release-u0.mp5-aiot-V7.174
$ vendor/mediatek/proprietary/scripts/releasetools/split_build_helper.py --run full_aiot8391p2_64_bsp-userdebug --vf-path ../alps-release-u0.mp5-aiot-V7.174
```

Once the build is complete, the generated Android image is located in:
```
alps-release-u0.mp5-aiot-V7.174/out/target/product/aiot8391p2_64_bsp/merged
```

## Flashing the Built Image
Instructions to flash the generated image using SP Flash Tool are covered in [README.md (Section 4: Flashing the Image and Booting)](README.md#4-flashing-the-image-and-booting).
