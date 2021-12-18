### Let's do it

### Raspberry Pi4: Installing latest updates and libraries

### Raspberry Pi4: Build GCC 10

### Raspberry Pi4: Prerequisites

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

Run the following command to know your binutils version:

`ld --version`

Run the following command to know your GLIBC version:

`ldd --version`

### CrossTool-NG: Downloading & Installing

Open a terminal to download and install crosstool-ng with homebrew. The formula is:

`brew install crosstool-ng`

Check that crosstool-ng has been installed typing:

`ct-ng version`

The installed version should be 1.24.0.

### CrossTool-NG: Preparing

Open the "Utility Disk" to make a new case sensitive volume.

Click on the "+" situated on the right corner of the window.
In my case I name it “crosstool-ng” which uses “APFS (case-sensitive)”. I've allocated 20GB about.

### CrossTool-NG: Making the cross-toolchain for Rpi4

Special thanks to [medium](https://medium.com/@stonepreston/how-to-cross-compile-a-cmake-c-application-for-the-raspberry-pi-4-on-ubuntu-20-04-bac6735d36df) for this.

We need to move inside the volume created in the previous step:

`cd /Volumes/crosstool-ng`

Create a folder here called "src".

`mkdir src`

Now we need to make a folder for the binutils patch downloaded with the Raspberry Pi. So run the following:

`mkdir -p patches/binutils/@TODO:BINUTILS_VERSION/`

So, move inside that directory and download the file from the Raspberry with SCP.

```
cd patches/binutils/@TODO:BINUTILS_VERSION/
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
In the same way change the "Working directory" to:

`${CT_TOP_DIR}/build`

Change the "Prefix directory" to:

`${CT_TOP_DIR}/${CT_HOST:+HOST-${CT_HOST}/}${CT_TARGET}`

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

Go back to the main menu and select the "C-compiler" sub menu. Change the version of gcc to the version on your Pi. Move to the option "gcc extra config" and change the value to

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

`SEARCH_DIR("=/lib/arm-linux-gnueabihf"); SEARCH_DIR("=/usr/lib/arm-linux-gnueabihf");`

If you do not see those directories, something is wrong. Go back and check your configuration and make sure you didn’t miss anything.

### Sync sysroot

### Qt Source: Download

### Qt Source: Configure

### Qt Source: Build

### Qt Source: Installing

### Qt: Upload compiled source to RaspberryPi

### QtCreator: Configuration

### QtCreator: Test App
