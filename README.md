# Android Custom ROM Building Guide (DEPRECATED)

This guide offers a simple guide on how to build a custom ROM for Android. However, simple can be harder than complex: You have to work hard to get your thinking clean to make it simple *(I'm not even sure what I'm saying at this point, thanks to sleep deprivation)*. But it’s worth it in the end because once you get there, you can move mountains.

Note: This guide assumes that you know how to use Linux fluently. This guide is a compilation of what I've learned over the years. Thanks to AlaskaLinuxUser and LineageOS wiki and many other sources my tired brain is unable to remember, as this guide will not be possible without them.

Note: This guide is so outdated that you shouldn't be following these instructions word for word.

## Requirements And Setting Up

Click this link [here](https://source.android.com/setup/build/requirements) from the official AOSP website to see if your computer meets the hardware and software requirements needed to build ROMs. Also, click this link [here](https://source.android.com/setup/build/initializing) to set up your build environment properly (don't forget to update your pre-existing packages). Another package to install is OpenJDK 8, which is a required package to build any ROM that is Android Oreo or newer.

```sh
sudo apt install openjdk-8-jdk
```

I personally use a Ubuntu Server 18.04 build server, as it is minimalistic and contains the OpenJDK 8 package. I heavily recommend Ubuntu 18.04 or any version higher that contains the OpenJDK 8 package in its repo by default. It is also possible to build ROMs in macOS, but this guide is targeted specifically for Linux users.

Note: If you are building for Android Nougat, install both OpenJDK 7 and 8.

## How To Build Stock AOSP ROMs

### Initialising The Source Directory

Click [here](https://source.android.com/setup/build/downloading) to download Repo. Now, while following the instructions, it might tell you to enter the following command or something similar if you are downloading a specific version of Android rather than the latest version:

```sh
repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1
```

Take note that the following example tells Repo to initialise the AOSP repository from the "android-4.0.1_r1" branch. If you want to download another branch of Android, click [here](https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds) or the link provided in the instructions. After that, copy the branch name from the appropriate row from the table into the command and run it.

You should have configured Git if you followed the instructions carefully, or this command results in an error. If you do not want Google to know your name and username, feel free to enter a placeholder name and email address just like how the examples from the instructions show. When asked whether you want to enable colour display in this user account, feel free to enter "yes", as it does not matter.

### Download The Source Code

After running "repo init", continue to follow the instructions and run "repo sync". Note that depending on your network speed, you might want to set the number of threads that should be utilised to synchronise the repo. The default option (without specifying any arguments) would be four threads. If you have a slower internet connection, you can lower it down to two. To set the number of threads to allocate for download to two:

```sh
repo sync -j2
```

Note: You can increase the number of threads by increasing the number of X, where X in -jX is the number of threads to use while downloading the source. This is highly dependent on the quality of your internet connection with the repo server. The -c argument tells Repo to pull in only the current branch instead of all branches that are available on the repository.

This process takes about an hour or more, so feel free to grab a cup of coffee, get on with your day while having this process running in the background on your computer.

### Prepare The Device-specific Code

After running "repo sync", click [here](https://source.android.com/setup/build/building) and follow the instructions to start the process of building Android. Now, the instructions might want you to run the following command:

```sh
lunch aosp_arm-eng
```

If you did not follow the instructions carefully, your computer might spit out an error stating that it cannot understand this command. This error is because you did not initialise the environment. Initialising the environment enables you to run commands such as "lunch" (and many other custom commands found in custom ROMs) as this allows the computer to set up the necessary build environment so that all the tools are at your disposal when needed.

Now to prevent me from going off-topic, let's move on to the next part. Since you are not familiar with all the device codenames, run the command without any arguments instead:

```sh
lunch
```

The command prompt presents you with a list of devices that you can build. Just enter the appropriate option number and press enter. Now this command sets up the build environment and not start building Android until you run the "make" command further down the road.

Note: There are multiple variants of the ROM that you can build for a device. For more information, on the same webpage, there is a table labeled "buildtype" which tells you more about what ROM variant is best suited for your intention of building your ROM. Also, if you have not downloaded the proprietary binaries yet (which we are going to do so below), errors might appear. Ignore them for now.

### Downloading Proprietary Binaries

Now, if you start building Android, the process goes on smoothly with no errors, but if you flash the ROM onto your phone, you will find out that your phone is unable to boot. That is because if we look at the build directory, you find that there is no "vendor" folder there, as it does not automatically download the proprietary binaries for you. That particular folder is essential as it contains all the necessary drivers specific to your device to allow it to run. To download those files, head over to [TheMuppets repository](https://github.com/TheMuppets) and there, you find the proprietary binaries of many major phone OEMs there in separate repositories. If you are building for a Google Pixel device, head over to the repository named "proprietary_vendor_google".

Note: If you are building for a Nexus device, most of these devices are not built by Google as Google only provides the software, while other OEMs build the physical device for them. Therefore, most components that are in Nexus devices are put together by them, so head over to their repositories instead and look for your device codename.

Now head over to the correct branch (select the Lineage version that corresponds to the Android version you are building). There are multiple methods of downloading the proprietary binaries, but the best method is to create a local manifest so that every time you run "repo sync" the proprietary binaries is automatically updated together with the AOSP source code. So now head over to <BUILD_DIRECTORY>/.repo/manifests (make sure you enable the option to show hidden files) and create a .xml file named "local_manifest.xml" (though you can create a .xml file with whatever name you like). After that, copy the following in (and modifying whatever parts that need to be changed):

```
<?xml version="1.0" encoding ="UTF-8"?>
<manifest>
  <project name="TheMuppets/proprietary_vendor_google" path="vendor/google" remote="github" revision="lineage-16.0" />
</manifest>
```

Now run "repo sync" and the "lunch" command to prepare to build.

Note: If you own a LineageOS device that you wish to build for, you can also extract the proprietary binaries from the ROM. Information is provided in the build instructions of the various devices in the LineageOS Wiki.

Another option will be to directly download the proprietary binaries from [here](https://developers.google.com/android/drivers) if you are unable to find it in the TheMuppets repository. For newer phones, after extracting the .tgz file a shell script will be provided that requires you to physically plug in your phone you are building for to your computer, so that the script can extract the proprietary binaries out of the phone itself.

### Turn On Caching To Speed Up Build

Make use of ccache if you want to speed up subsequent builds by running:

```sh
export USE_CCACHE=1
```

and adding that line to your ~/.bashrc file. Then, specify the maximum amount of disk space you want ccache to use by typing this:

```sh
ccache -M 50G
```

where 50G corresponds to 50GB of cache. This needs to be run once. Anywhere from 25GB-100GB will result in very noticeably increased build speeds (for instance, a typical 1hr build time can be reduced to 20min). If you’re only building for one device, 25GB-50GB is fine. If you plan to build for several devices that do not share the same kernel source, aim for 75GB-100GB. This space will be permanently occupied on your drive, so take this into consideration.

You can also enable the optional ccache compression. While this may involve a slight performance slowdown, it increases the number of files that fit in the cache. To enable it, run:

```sh
export CCACHE_COMPRESS=1
```

or add that line to your ~/.bashrc file.

Note: If compression is enabled, the ccache size can be lower (aim for approximately 20GB for one device).

### Building Android 

After you have downloaded the necessary proprietary binaries, you can run the following command below:

```sh
make -j4
```

Note: As you can see, you can specify the number of threads to use above, just like the "repo sync" command. However, the number of threads you should use is now wholly dependent on the amount of resources you should allocate to your CPU, rather than going by your internet speed, as we are well past of downloading source code from the internet and are now building Android, which is a very CPU intensive task, so your computer is going to get real toasty really fast. So set the appropriate number of threads based on how many cores your CPU has. The more the number of CPU cores, the more the number of threads you can allocate. The default option (without specifying any arguments) is four threads. Also, this process is going to take a long while (just like the "repo sync" command earlier), so feel free to leave your computer for now. Lastly, throughout the build process, warnings might appear. Most of the time, you do not have to worry about those, until errors occur.

Best practices:

- Minimum number of threads you can allocate: Number of CPU(s) * Number of cores per CPU * Number of threads per core (most likely 2)

- Maximum number of threads you can allocate: Minimum number of threads * 2

### Creating Flashable Zips

After the build has completed enter the output directory <BUILD_DIRECTORY>/out/target/product/<DEVICE_CODENAME> and you will see a bunch of .img files. You can flash them with Fastboot. However, most of the time, you want to create a flashable zip file. To do that you need to run the following command below:

```sh
make otapackage
```

This command generates a .zip file in the same directory as mentioned above. This process takes a shorter time to execute as you should have the necessary files required after building to flash a functioning ROM into a device, so the program is smart enough not to build Android again. With that said, if you do not want to create any .img files but generate a .zip file directly, you can run "make otapackage" instead of the "make -jX" command mentioned above.

Note: To quickly enter the output directory, you can type

```sh
cd $OUT
```

into the terminal.

### Future Builds

Run the following commands below to update your ROM and device sources.

```sh
repo sync
source build/envsetup.sh
lunch
```

## How To Build Custom ROMs

In this guide, we are using LineageOS as an example of how to build custom ROMs for your device. This process is mostly the same as building AOSP ROMs, so I am not going through all the steps mentioned above again, but I am going to highlight any actions that are noticeably different.

Note: You should not use the same build directory if you are building a different ROM.

### Build Instructions

The process for building Android will differ from one custom ROM to another. So the first and most crucial step is to search for the manifest of the custom ROM (which is different from the local manifest file stated above). It is a guide specially made for the custom ROM you are building. You can typically find it in their repository by searching for the keyword "manifest", or in LineageOS's case, in a repository named "android" (which is also relatively common). Make sure that you are in the correct branch (correct LineageOS version). Over there, you might find that their instructions might be similar or even identical (except the "repo init" command which points to a different repository) to the steps mentioned earlier. Make sure to follow their instructions carefully. Mentioned below are some of the steps that will be different.

### repo init

Notice that the "repo init" command points to a different repository in the manifest as compared to the repository stated in the official AOSP website. You should copy and paste the "repo init" in the manifest instead of copy and pasting the entire "repo init" command from the official AOSP website.

### Custom Commands

Custom ROMs may employ various custom commands to simplify the building process. LineageOS (and it's derivative custom ROMs) use the "brunch" command. It sets up your build environment to be configured for your device and then commences the build process. It also automatically creates the flashable zip and the LineageOS recovery image that TWRP can flash directory in the output directory. It's generally only used for officially supported devices (ones that you use can choose through the breakfast menu). It is basically a combination of "lunch" and "make" (or more accurately "breakfast" and "mka"). "breakfast" does the same thing as "lunch", but as mentioned above, is used typically for building supported devices of a specific custom ROM. "mka" does the same job as "make", but builds Android more efficiently, as it uses the program "sched_tool" to make full use of all the threads available on your machine. For AMD, this is equivalent to the number of cores your processor has; For Intel, this is usually equivalent to twice the amount of cores your processor has, due to Hyperthreading). What this means is that ALL of your processors are working, not just one small part of it. Choose the device you want to build for by typing the "brunch" command. Type in the numbered option, and sit back and relax. You do not have to run the "make" command; the computer will do that for you.

Another example of a custom command will be "croot" (not to be confused with chroot) where it changes your current working directory into the root directory of the build environment.

Note: Take note that when you enter "brunch" in the terminal, some ROMs do not give you the option of selecting any device. You will have to search for the device codename manually.

### Proprietary Binaries

After checking your device at the TheMuppets repository, go to <BUILD_DIRECTORY>/.repo/local_manifests instead. Open the file named "roomservice.xml" with a text editor and add the following line into the .xml file (and modifying whatever parts that need to be changed):

```
  <project name="TheMuppets/proprietary_vendor_oneplus" path="vendor/oneplus" remote="github" />
```

Note: If you do not have the folder, you can simply create it.

### Custom Build Scripts

The custom ROM manifest might instruct you to run a certain shell script to start the building process instead of "brunch" and "make". Simply follow the instructions and you should be fine.

### Signing Builds

To learn more about signing your own builds, click here[https://wiki.lineageos.org/signing_builds.html].

## Resolving Errors

Time and time again, you might encounter errors while building the custom ROM. The best way to resolve it is to copy and paste the error message along with the name of the ROM you are building and Google it to figure out what other users are doing to fix the error.

## Building A Custom ROM For Any Device

Note: It is highly recommended to read the entire guide from start to finish and not to jump to this part straight away.

When I say any device, I mean any device that has their device, kernel, and proprietary binaries made public. Use Google to search for the device tree, kernel tree, and the proprietary binaries of the device you want to build for. For example, this are the links to the repositories for the OnePlus 6:

Device tree: https://github.com/LineageOS/android_device_oneplus_enchilada.git

Kernel tree: https://github.com/LineageOS/android_kernel_oneplus_sdm845.git

Vendor tree: https://github.com/TheMuppets/proprietary_vendor_oneplus.git

As you can see, since LineageOS officially supports OnePlus 6 at the time of writing, its device and kernel tree are included in their repository. You could use the same device and kernel tree to build other custom ROMs for that same device.  If other custom ROMs officially support your device, they will also have included the device and kernel tree in their repositories. Another way is to go to your device's XDA Developers forum page and look for any custom ROMs built by other users. The link to the device and kernel tree etc. should be posted there. Lastly, as I mentioned above, you could Google it.

Now that you have the links to the repositories required, assuming that you are building for the OnePlus 6, add the following repositories to the "roomservice.xml" file as I mentioned above (modify when necessary):

```
  <project name="LineageOS/android_device_oneplus_enchilada" path="device/oneplus/enchilada" remote="github" revision="lineage-16.0" />
  <project name="LineageOS/android_kernel_oneplus_sdm845" path="kernel/oneplus/sdm845" remote="github" revision="lineage-16.0" />
  <project name="TheMuppets/proprietary_vendor_oneplus" path="vendor/oneplus" remote="github" revision="lineage-16.0" />
```

After that, run "repo sync" to download the repositories to the build directory.

Since you are building a different custom ROM, go to <BUILD_DIRECTORY>/device/<DEVICE_OEM>/<DEVICE_CODENAME> and open "vendorsetup.sh". For example, if you took the device tree from AOKP, but you are building PAC ROM, change the following line from this,

```
add_lunch_combo aokp_enchilada-eng
```

to this:

```
add_lunch_combo pac_enchilada-eng
```

Now if you start building your custom ROM, you might encounter errors. The most common reason is that you are still missing some build dependencies. Look through the errors, and try running "breakfast" or "lunch" commands if you cannot spot any errors that make sense to you. Once, you identified the build dependency you need to include, simply include the necessary repository into "roomservice.xml" and run "repo sync" again.

## Cherry-picking Guide

Unfortunately, not every device runs as smooth or bug-free as its OEM ROM it is shipped with after flashing the ROM you had just built. Therefore, you might want to integrate fixes from other ROM developers into your ROM. The easiest way to do that is to cherry-pick commits from other ROM developers in the hope that it will solve the problem you are currently facing in your ROM.

To add code from other repositories to your current repository for building your custom ROM, check out this useful guide [here](https://www.google.com/url?sa=t&source=web&rct=j&url=https://forum.xda-developers.com/showthread.php%3Ft%3D2763236&ved=2ahUKEwiq3ueB_tjkAhVLfisKHcbhCu0QFjAAegQIAxAB&usg=AOvVaw1ZVNVztDPgncTqb2cqKo_B).
