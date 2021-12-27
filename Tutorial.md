### Tutorial: Compile Qt Framework Binaries & Qt Application for Raspberry Pi 4B from a Mac host machine

### Raspberry Pi4:

##### Installation of a fresh OS

Download <b>Raspberry Pi Imager</b> from [here](https://downloads.raspberrypi.org/imager/imager_latest.dmg) so plug your SD card in your PC and make a fresh installation of Raspberry PI OS. I personally used a 32 bit version with desktop environment. Once done, plug your SD into the Raspberry and install the OS into it.

##### Installing latest updates and libraries

Once the OS is installed, enable SSH so connect to your Raspberry using it and edit the file "/etc/apt/sources.list", so run the folling command:

`sudo nano /etc/apt/sources.list`

And remove the comment in the third line (delete the # at the beginning), the file should appear as follow:

```
deb http://raspbian.raspberrypi.org/raspbian/ bullseye main contrib non-free rpi
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
deb-src http://raspbian.raspberrypi.org/raspbian/ bullseye main contrib non-fre>
```

Save the file pressing "CTRL + X" then press the "Y" button, then "Enter". Now run the following commands:

```
sudo apt update
sudo apt full-upgrade
sudo reboot
sudo rpi-update
sudo reboot

sudo apt-get install libboost-all-dev libudev-dev libinput-dev libts-dev libmtdev-dev libjpeg-dev libfontconfig1-dev libssl-dev libdbus-1-dev libglib2.0-dev libxkbcommon-dev libegl1-mesa-dev libgbm-dev libgles2-mesa-dev mesa-common-dev libasound2-dev libpulse-dev gstreamer1.0-omx libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev  gstreamer1.0-alsa libvpx-dev libsrtp0-dev libsnappy-dev libnss3-dev
sudo apt-get install "^libxcb.*"
sudo apt-get install flex bison libxslt-dev ruby gperf libbz2-dev libcups2-dev libatkmm-1.6-dev libxi6 libxcomposite1 libfreetype6-dev libicu-dev libsqlite3-dev libxslt1-dev libavcodec-dev libavformat-dev libswscale-dev gstreamer1.0-tools libraspberrypi-dev libx11-dev libglib2.0-dev freetds-dev libsqlite0-dev libpq-dev libiodbc2-dev firebird-dev libgst-dev libxext-dev libxcb1 libxcb1-dev libx11-xcb1 libx11-xcb-dev libxcb-keysyms1 libxcb-keysyms1-dev libxcb-image0 libxcb-image0-dev libxcb-shm0 libxcb-shm0-dev libxcb-icccm4 libxcb-icccm4-dev libxcb-sync1 libxcb-sync-dev libxcb-render-util0 libxcb-render-util0-dev libxcb-xfixes0-dev libxrender-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-glx0-dev libxi-dev libdrm-dev libssl-dev libxcb-xinerama0 libxcb-xinerama0-dev libssl-dev libxcursor-dev libxcomposite-dev libxdamage-dev libfontconfig1-dev libxss-dev libxtst-dev libpci-dev libcap-dev libsrtp0-dev libxrandr-dev libnss3-dev libdirectfb-dev libaudio-dev libxfixes-dev libxcb-util-dev libxcb-xkb-dev libxkbcommon-x11-dev
```

The following steps are needed to install the libatspi library:

```
sudo pip install meson
sudo apt install -y ninja-build
wget https://github.com/GNOME/at-spi2-core/archive/refs/tags/AT_SPI2_CORE_2_42_0.tar.gz
tar -xvf AT_SPI2_CORE_2_42_0.tar.gz
cd at-spi2-core-AT_SPI2_CORE_2_42_0/
mkdir build
cd build
meson ..
ninja
sudo ninja install
```

Once that all library are installed, create the directory where we will put the QT compiled binaries:

```
sudo mkdir /usr/local/qt6
sudo chown pi:pi /usr/local/qt6

sudo reboot
```

##### GCC 11: Build & Install

@TODO

##### Prerequisites

To configure our toolchain, with a vanilla toolchain created by crosstool-NG, the linker does not have the necessary library search paths. On the Pi, there are libraries stored inside /usr/lib/arm-linux-gnueabihf/ and /lib/arm-linux-gnueabihf/. This is because of Debian Multiarch. However, the linker from the binutils that gets downloaded by crosstool-NG will only search inside /usr/lib and /lib, which means that libraries in the arm-linux-gnueabihf subdirectories wont be found.
To fix this issue, we need to provide a patch for binutils (the package that contains the linker among other things) to crosstool-NG. This patch is provided by Debian, and we can get a copy of it on the Pi by installing binutils-source.

So, start an SSH session with the api running:

`ssh pi@ip_address`

Run the following command to download and install the binutils sources:

`sudo apt install binutils-source`

Check that the patch is downloaded,looking for a file named "129_multiarch_libpath.patch" after that you run this command:

`ls /usr/src/binutils/patches/`

Run the following command to know your kernel version:

`uname -r`
In my case is: 5.10.88-v7l+

Run the following command to know your binutils version:

`ld --version`
In my case is: 2.35.2

Run the following command to know your GLIBC version:

`ldd --version`
In my case is: 2.31

### MacOS

#### CrossTool-NG

##### Prerequisites
First of all install the required packages with brew running the following commands:

```
brew install gnu-sed
brew install binutils
brew install help2man
brew install gawk
brew install libtool
brew install ncurses
brew install autoconf
brew install automake
brew install bison
brew install diffutils
brew install flex
brew install git
brew install gperf
brew install make
brew install texinfo
brew install wget
brew install xz
brew install gmp
brew install mpfr
brew install openssl
brew install pcre
brew install readline
brew install bash
```

After that, open the file ".bash_profile" with the following command:

`sudo nano /Users/yourUserName/.bash_profile`

Add this line at the end of the file:

```
export PATH="/usr/local/opt/libtool/libexec/gnubin:$PATH"
export PATH="/usr/local/opt/ncurses/bin:$PATH"
export PATH="/usr/local/opt/flex/bin:$PATH"
export PATH="/usr/local/opt/openssl@3/bin:$PATH"

export LDFLAGS="-L/usr/local/opt/binutils/lib -L/usr/local/opt/ncurses/lib -L/usr/local/opt/flex/lib -L/usr/local/opt/openssl@3/lib"
export CPPFLAGS="-I/usr/local/opt/binutils/include -I/usr/local/opt/ncurses/include -I/usr/local/opt/flex/include -I/usr/local/opt/openssl@3/include"
```

Close the file with "CTRL+X" and press "Y". Reload your terminal.

##### Downloading & Installing

Open a terminal to download, build and install crosstool-ng from source code. Clone the crosstool-ng repository with the the following command:

`git clone https://github.com/crosstool-ng/crosstool-ng`

Move into the downloaded folder and type the following commands:

```
cd crosstool-ng
./bootstrap
./configure --prefix=/usr/local
make
sudo make install
```

Check that crosstool-ng has been installed typing:

`ct-ng --version`

##### Preparing

Open the "Utility Disk" to make a new case sensitive volume.

Click on the "+" situated on the right corner of the window.
In my case I name it “crosstool-ng” which uses “APFS (case-sensitive)”. I've allocated 20GB about.

##### Making the cross-toolchain for Rpi4

Special thanks to [medium](https://medium.com/@stonepreston/how-to-cross-compile-a-cmake-c-application-for-the-raspberry-pi-4-on-ubuntu-20-04-bac6735d36df) for this.

We need to move inside the volume created in the previous step:

`cd /Volumes/crosstool-ng`

Create a folder here called "src".

`mkdir src`

Now we need to make a folder for the binutils patch downloaded with the Raspberry Pi. So run the following:

`mkdir -p patches/binutils/2.35.1/`

I called the folder "2.35.1" instead of "2.35.2" because crosstool-ng doesn't offer the possibility to choose the "2.35.2" version of binutils.

So, move inside that directory and download the file from the Raspberry with SCP.

```
cd patches/binutils/2.35.1/
scp pi@ip_address:/usr/src/binutils/patches/129_multiarch_libpath.patch ./
cd ../../../
```

Now we are ready to start the configuration of the toolchain.
Run the following command to see the available crosstool-ng's configuration:

`ct-ng list-samples`

You should see a sample named "armv8-rpi3-linux-gnueabihf". This is the base configuration for the 32bit armv8 of RaspberryPi 3. We use this as base configuration.
So, if you see this configuration, then run the following command:

`ct-ng armv8-rpi3-linux-gnueabihf`

As we said, this is the configuration of the Raspberry Pi 3 so we need to make some changes to that.
Run:

`ct-ng menuconfig`

Select "Path and misc options" and press "Enter".
Move to "Local tarballs directory" and change the value to: 

`${CT_TOP_DIR}/src`

CT_TOP_DIR is a variable pointing to the staging directory, it is the directory where the config file is placed.
In the same way change the "Prefix directory" to:

`${CT_PREFIX:-${CT_TOP_DIR}}/${CT_HOST:+HOST-${CT_HOST}/}${CT_TARGET}`

Move to "remove prefix dir prior to building option" and enable it pressing the spacebar.
Move to "Render the toolchain read-only" and disable it pressing the spacebar.

Scroll down to the Extracting section of the menu to the "Patches origin" option and press enter. Change the option from "Bundled only" to "Bundled, then local".

Select the option called "Local patch directory" and change the value to: 

`${CT_TOP_DIR}/patches`

Press the right arrow key to move over to exit and press enter. This will take you back to the main menu. Now select the "Target options" menu.

Change the option "Emit assembly for CPU" to the following value:

`cortex-a72`

Go back to the main menu and go to the "Toolchain options" menu.
Change the "Tuple's vendor string" option to the following value:

`rpi4`

Go back to the main menu and open the "Operating System" menu. Change the version of Linux to the closest version to what you found on the Pi earlier.

Go back to the main menu and open the "Binary utilities" menu. Change the version of the binutils to whatever was on your Pi.

Go back to the main menu and select the "C-Library" sub menu. Change the version of glibc to the version from your Pi.

Go back to the main menu and select the "C-compiler" sub menu. Change the version of gcc to the version to "11.2.0". Move to the option "gcc extra config" and change the value to

`--enable-multiarch`

Make sure that C++ is enabled in the section called "Addition supported language".

Exit back to the main menu. Select the save option using the right arrow key and accept the default file name. Configuration is complete. You can then exit the main menu to return to your terminal.
The last step we need to complete before building is exporting the Debian multiarch target variable:

`export DEB_TARGET_MULTIARCH=arm-linux-gnueabihf`

Now we can build the toolchain running:

`ct-ng build`

Once complete, you can test that the linker searches the correct paths using this command:

`armv8-rpi4-linux-gnueabihf/armv8-rpi4-linux-gnueabihf/bin/ld --verbose | grep -i "search"`

The output should include:

```
SEARCH_DIR("=/lib/arm-linux-gnueabihf");
SEARCH_DIR("=/usr/lib/arm-linux-gnueabihf");
```

If you do not see those directories, something is wrong. Go back and check your configuration and make sure you didn’t miss anything.

### Sync sysroot
@TODO

#### Qt
##### Download

Download the Qt 6.2.2 source code from here: [link](https://download.qt.io/archive/qt/6.2/6.2.2/single/qt-everywhere-src-6.2.2.zip).
Extract the downloaded archive.

##### Configure

First of all, create a new file called "toolchain.cmake" and put the following inside it:

```
SET(CMAKE_SYSTEM_NAME Linux)
SET(CMAKE_SYSTEM_VERSION 1)

set(TOOLCHAIN_ROOT_DIR /Volumes/crosstool-ng/armv8-rpi4-linux-gnueabihf)
set(RASPBERRY_ROOT_FS ${TOOLCHAIN_ROOT_DIR}/armv8-rpi4-linux-gnueabihf/sysroot)

set(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} ${RASPBERRY_ROOT_FS}/usr/lib ${RASPBERRY_ROOT_FS}/usr/lib/arm-linux-gnueabihf)
set(CMAKE_INCLUDE_PATH ${CMAKE_INCLUDE_PATH} ${RASPBERRY_ROOT_FS}/usr/include)

# Specify the cross compiler
SET(CMAKE_C_COMPILER ${TOOLCHAIN_ROOT_DIR}/bin/armv8-rpi4-linux-gnueabihf-gcc)
SET(CMAKE_CXX_COMPILER ${TOOLCHAIN_ROOT_DIR}/bin/armv8-rpi4-linux-gnueabihf-g++)

# Where is the target environment
SET(CMAKE_SYSROOT ${TOOLCHAIN_ROOT_DIR}/armv8-rpi4-linux-gnueabihf/sysroot)
SET(CMAKE_FIND_ROOT_PATH ${TOOLCHAIN_ROOT_DIR})

#SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} --sysroot=/Volumes/crosstool-ng/")
#SET(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} --sysroot=${CMAKE_FIND_ROOT_PATH}")
#SET(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} --sysroot=${CMAKE_FIND_ROOT_PATH}")

set(CMAKE_C_FLAGS
  "-lGLESv2 "
  CACHE STRING "" FORCE
)

set(CMAKE_CXX_FLAGS
  "-lGLESv2 "
  CACHE STRING "" FORCE
)

# Search for programs only in the build host directories
SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)

# Search for libraries and headers only in the target directories
SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
```

Use the path of this file for the argument "DCMAKE_TOOLCHAIN_FILE" of the next step. In my case this was: "/Users/davide/qt6/rpi4-cross/toolchain.cmake".
To compile the Qt binaries for RaspberryPi you must have the Qt binaries compiled for Mac OS. In my case that are inside the folder "/opt/Qt6", and this path must be specified inside the "qt-host-path" argument of the next step.
The argument "extprefix" specify where to install the binaries inside your Mac, in my case i used "/opt/Qt6Rpi4".
The argument "prefix" specify the path where the Qt6 binaries must be installed. I used "/usr/local/qt6".

In the end, this is the entire configure command:

`../qt-everywhere-src-6.2.2/configure -release -opengl es2 -qt-harfbuzz -qt-doubleconversion -no-use-gold-linker -skip qtwayland -skip qtdatavis3d -skip qtwebengine -device linux-rasp-pi4-v3d-g++ -nomake examples -nomake tests -qt-host-path /opt/Qt6 -extprefix /opt/Qt6Rpi4 -prefix /usr/local/qt6
-- -DCMAKE_TOOLCHAIN_FILE=/Users/davide/qt6/rpi4-cross/toolchain.cmake`

##### Build & Install

After that configuration is completed, just run these commands:

```
cmake --build . --parallel
cmake --install .
```

Sometimes could happen that the build process stuck, if it happen, press "CTRL + C" and run this:

`cmake --build . --parallel 4`

##### Upload compiled binaries to RaspberryPi
@TODO

### QtCreator

##### Configuration
@TODO

##### Test App
@TODO
