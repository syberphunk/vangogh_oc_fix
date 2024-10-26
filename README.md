# Disclaimer
[Badly-Drawn-Wizards](https://github.com/badly-drawn-wizards) Is no longer actively working on this project. Maintenance has been provided by [lividhen](https://github.com/lividhen) and [Syberphunk](https://github.com/syberphunk).

# Description
This is a linux kernel module to override the AMD VanGogh APU PowerPlay limits for its CPU. This is useful if you have overclocked your SteamDeck in the BIOS but you still want to use the [Decky plugin PowerTools](https://git.ngni.us/NG-SD-Plugins/PowerTools/) to set CPU clock limits. You do not need the vanvogh 'fix' for GPU overclocking. The Steam Deck, particularly the OLED, should scale up to any overclock values set in the BIOS by default.

Note that if you are using the [Decky plugin PowerTools](https://git.ngni.us/NG-SD-Plugins/PowerTools/) you need to setup [custom overrides](https://git.ngni.us/NG-SD-Plugins/PowerTools/wiki/Customization) to tell PowerTools that it can go higher than the default values. Conversely, the way PowerTools works as of writing this (2024-06-06) will not go past stock settings even if you have custom overrides because of limits in amdgpu/powerplay.

Similarly, you still need to set your maximum clock speed through the BIOS, this should now be introduced in the default BIOS for the Steam Deck LCD and OLED. If you are on earlier versions of the Steam Deck LCD BIOS you will need to edit or override the BIOS options using SmokelessUMAF, sdunlock or SmokelessRuntimeEFIPatcher, depending on your BIOS version.

The Steam Deck Oled BIOS has overclocking built in after BIOS 109, and after BIOS 131 on the LCD deck, and does not need to be unlocked. Except if you want to override the TDP/Fast PPT/SlowPPT which requires modifications with [SREP](https://www.stanto.com/steam-deck/how-to-unlock-the-lcd-and-oled-steam-deck-bios-for-increased-tdp-and-other-features/) due to the Advanced 'forms' in the InsydeH2O BIOS.

You will need to reinstall the module each SteamOS update as it wipes the modification in the system. Hopefully smarter people will make an easier fix, or the limit in the amdgpu driver will be made configurable.

There are issues raised about this on [gitlab](https://gitlab.freedesktop.org/drm/amd/-/issues/2638) and [github](https://github.com/ValveSoftware/SteamOS/issues/1309), and you should join in with those conversations.

# Disclaimer
This software is distributed under the terms of the GPLv3 license. Please refer to the license for the full disclaimer and understand that by using this software, you do so at your own risk.

The module does a sanity check to ensure that it is trying to modify the right value, but this may fail and write to some unknown place in kernel memory which BAD™.

# How to build & install

As of SteamOS 3.6.19 it is easier if you enable devmode and unminimize the OS. Make sure you have enough storage space tod o this. 

You also need a 'sudo' password set.

Type the following into a terminal:
- `sudo steamos-readonly disable`
- `sudo steamos-devmode enable`
- `sudo steamos-unminimize --dev`
- `sudo pacman -Sy base-devel`

Now we need to know what header packages to install.
- `uname -r`
With the output of this command you will see something like:

`6.1.52-valve16-1-neptune-61`

Now type:
- `sudo pacman -Ss linux-neptune`

Which will give you the output such as:
`jupiter-3.6/linux-neptune-65 6.5.0.valve22-1 [installed]`
`jupiter-3.6/linux-neptune-65-headers 6.5.0.valve22-1`

This will tell us the package names that relate to our kernel version.

We want the ones that match, so in our example, we're running neptune-65, we type:

- sudo pacman -S linux-neptune-65 linux-neptune-65-headers`

Now, onto installing the fix. By default SteamOS will put you into the `/home/deck` folder.

Using the commands `cd` and `pwd` you can change directory, and also check what directory
you are in, to be able to navigate to where you have extracted or cloned this repo to.

If you're unsure how to do this, [learn more about the terminal](https://ubuntu.com/tutorials/command-line-for-beginners).
Once you're in the correct folder, you can install with:

- `./install.sh`

If you get the error "install.sh not found" then you haven't downloaded the files, or navigated to the correct folder.

You should now be prompted for the CPU frequency, enter the same value as you have set in the BIOS. GPU frequency is automatically determined, this is solely for the CPU.

If this does not work, read the messages on the screen. If it does not install, then you will need to manually add support for your specific kernel (this can happem when SteamOS has updated beyond what is supplied with this repo) and then run the install again. Follow the instructions below.

# How to manually add support specific kernels

This project depends on unstable internel APIs of the amdgpu<sup>[1]</sup> driver. In version `0.0.1` I created c header files to mirror the header files in the kernel, but this not a great idea. In this version I copy the header files for a specific kernel version and store them for each version in source control.

Support provided with this repo' is stored in `/module/amd_headers/`.

Note this will download approximately 3.2gb of data.

To add support for your kernel version:
- `cd` to `/linux-header-extract directory` and
- Type `./get.sh`
- Then type `make -j$(nproc) linux-pkg-prepare`
- Then type `make -j$(nproc) extract-headers`

You can then use it for yourself or submit a pull request so others won't need to do this process.

If you're using SteamOS beta, preview or main then get.sh may not grab the correct file, until get.sh is adapted to accommodate for this you will need to manually edit the script to get the correct headers file.

If you are in need of a different version of the kernel headers is giving you, you can download it from [here](https://steamdeck-packages.steamos.cloud/archlinux-mirror/jupiter-main/os/x86_64/).
It is prefaced with linux-neptune (eg. linux-neptune-61-6.1.52.valve16-1-x86_64.pkg.tar.zst). Then run `sudo pacman -U /path/to/linux-neptune-headers`.

[1] In addition to what the name of the driver suggests, it also exposes the interface that PowerTools uses to adjust the CPU clock.
