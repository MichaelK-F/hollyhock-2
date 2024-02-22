If you want to install gcc for sh4, look [here](https://github.com/TheRainbowPhoenix/sh4-devenv-gitpod/blob/master/Dockerfile)!

---

# Patching the fx-CP400 firmware
**You do not need to do this by hand if you just want to install programs on ypur calculator! Just use the program Snail2021.exe which is available in the releases.**

Simple instructions if you just want to install it: https://github.com/SnailMath/hollyhock-2#installation-simple 

If you don't want to youse the precompiled program and you want to compile it by hand, 
you need both a Windows and Linux machine to complete this process (as there is no updater for Linux, and the extraction tool extracts the firmware from the Windows DLLs). I'd recommend a virtual machine (or Vagrant box for simplicity) rather than two physical machines.

You'll also need a resource editor program - [Resource Hacker](http://www.angusj.com/resourcehacker/) has been tested to work ~~(Visual Studio can also be used to replace the RCDATA resource - just be careful to give the replaced resource an ID of 3070!).~~ (Snail2021 relies on 'Resource Hacker')

## 0. Clone this repository
Clone this repository onto your machine. If you're using a virtual machine, this is really great place to use the shared folders feature your VM host probably provides.

## 1. Build an SH4 cross-compiler
(This is also needed if you just want to develop programs for the Classpad II.)

Hop onto your Linux machine.

First, you'll have to define a few environment variables to ensure the process goes smoothly. Feel free to change `PREFIX` (the installation location), but don't change `TARGET`.

```sh
# folder to install our cross-compiler into
export PREFIX="$HOME/opt/cross"

# the architecture and executable format we're targeting
export TARGET=sh4-elf

# add the cross-compiler to our PATH
# you'll probably want to put this somewhere like .bash-profile or .bashrc
export PATH="$PREFIX/bin:$PATH"
```

Now, download the most recent stable version of the Binutils source code from the [Binutils website](https://gnu.org/software/binutils/). Extract the compressed file containing the source code into a directory, and `cd` into the newly created directory. Then, run these commands inside that directory. (SnailMath is using version 2.35)

```sh
mkdir build
cd build
../configure --target=$TARGET --prefix="$PREFIX" --with-sysroot --disable-nls --disable-werror
make
sudo make install
```

You'll now have tools such as `sh4-elf-as` and `sh4-elf-objcopy` available.

Now, download the most recent stable version of the GCC source code from the [GCC website](https://gnu.org/software/gcc/). Extract the compressed file containing the source code into a directory, and `cd` into the newly created directory. Then, run these commands inside that directory. Be warned - the `make` commands may take a long time, depending on your system, and the `contrib/download_prerequisites` script will download about 30 MB of archives. (SnailMath is using version 10.2.0)

```sh
contrib/download_prerequisites
mkdir build
cd build
../configure --target=$TARGET --prefix="$PREFIX" --disable-nls --enable-languages=c,c++ --without-headers --with-multilib-list=m4-nofpu
make all-gcc
make all-target-libgcc
sudo make install-gcc
sudo make install-target-libgcc
```
(Note: The `--with-multilib-list=m4-nofpu` flag is needed because the cpu doesn't have an fpu)

You’ll now have `sh4-elf-gcc` and `sh4-elf-g++` available for your usage.

_I would recommend adding `export PATH="$PATH:$HOME/opt/cross/bin"` to the file .bashrc in the home directory to add the path automatically after every login._
_I would recommend adding `export SDK_DIR="PATH-TO/hollyhock-2/sdk"` to the file .bashrc in the home directory to export the sdk dir automatically after every login._

## 2. Build the patches and the patcher
Keep working on your Linux machine.

`cd` into the `patcher/` directory. Run `make`.
(This will run `make` in `patches/` and it will merge the file mod.txt with the program Snail2021 into a file called Snail2021mod.c)

## 3. Building Snail2021
Snail2021 is the program, that will extract the firmware updater from the casio updater. To build this, go to your Windows machine.

-Install the MinGW-Installer
-Select `mingw32-base-bin` (from All Packages -> MinGW -> MinGW Base System) -> right click -> 'Mark for installation'
-Select `mingw32-libz-dev` (from All Packages -> MinGW -> MinGW Libaries -> MinGW Standard Libaries) -> right click -> 'Mark for installation'
-Click on  'Installation' -> 'Apply Changes'

-Change the path of `patcher/make.bat` to what is correct on your computer
-Double click on `patcher/make.bat`. You should not see any errors. This will create the file `Snail2021.exe` in `patcher/Snail202x/`.


## 4. Run Snail2021 and modify the firmware

- Create a new folder somewhere. For this example it will be called `build`.
- Copy `Snail2021.exe` to the `build` folder.
- Copy all of the files from the `extras` folder in the source code into the `build` folder you created.
- Now you can follow the [simple install instructions](https://github.com/MichaelK-F/hollyhock-2#installation-simple).

## 5. Check the patch
Turn on your calculator and confirm the patch was applied correctly by opening the version dialog (either from the settings menu in the top left corner of Main > Version, or in System) and confirming the string "hollyhock" is displayed.

## 6. Build the launcher
`cd` into the `launcher/` directory. Run `make`, then copy the file `run.bin` into the root directory of your GC's flash.

## 7. Celebrate!
You've successfully patched your fx-CP400's firmware, and installed the Hollyhock Launcher. You're ready to go!

You can now start [using](using.md) or [developing](developing.md) software for your GC.
