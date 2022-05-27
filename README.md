# macemu-appimage-builder
Builds the popular classic Macintosh emulators BasiliskII and SheepShaver
from source for 64-bit x86 Linux and creates AppImages which run instantly on
many Linux desktops.

Download the [latest build](https://github.com/Korkman/macemu-appimage-builder/releases/latest)

[![Compile and release](https://github.com/Korkman/macemu-appimage-builder/actions/workflows/compile-and-release.yml/badge.svg)](https://github.com/Korkman/macemu-appimage-builder/actions/workflows/compile-and-release.yml)

## Download contents

```
BasiliskII       - launcher for the 68k emulator
BasiliskIIGUI    - launcher for the 68k emulator GUI
SheepShaver      - launcher for the PPC emulator
SheepShaverGUI   - launcher for the PPC emulator GUI
macemuAppImages/ - contains the actual AppImages

Install          - script to install as applications into home directory
installFiles/    - contains .desktop files and .png icons for the installation

SheepShaverMmap  - helper to fix mmap allocation permissions
```

## About the launchers
They are mostly optional. They make GUI and non-GUI startup easily accessible and ensure proper
working directories. For SheepShaver the sysctl variable vm.mmap_min_addr is set to 0 if required. 
The main point is to generate the AppImages in the directory `macemuAppImages`. You
can skip the launchers if you want to.

## Install (optional)
The install script will copy launchers and AppImages to `$HOME/.local/bin/`[^1] and create
menu entries for the application menu for convenience.

## Where to put startup.wav
Download your favorite startup chime in WAVE format and name it startup.wav to have it play on startup. Place it right into `macemuAppImages`. When installed it is located in `$HOME/.local/bin/macemuAppImages`[^1].

## Where will preferences be saved
In your home directory. Unless…

…you create directories named `SheepShaver.AppImage.home` and `BasiliskII.AppImage.home` within `macemuAppImages`. See [AppImage portable mode](https://docs.appimage.org/user-guide/portable-mode.html).

## AppImage compatibility
AppImages are portable Linux applications containing all the libraries required to run[^2]. Their
only dependency is having FUSE available (which basically every sane Linux desktop environment has).[^3]

## If SheepShaver doesn't start
Try running SheepShaver in a terminal to see constructive error messages. One common problem is that Linux by default disallows low memory access, which SheepShaver unfortunately requires. The launchers provide further instructions.

# Building your own

## Prerequisites
* 64-bit x86 Linux system
* Docker

If you don't meet the prerequisites the [Dockerfile](https://github.com/Korkman/macemu-appimage-builder/blob/main/build-stage1/Dockerfile) might still be of help.

## Compile
```
git clone https://github.com/Korkman/macemu-appimage-builder.git
cd macemu-appimage-builder
sudo ./compile
```
The process will delete and reproduce all files in the directory "output".

## Purge docker images
Once you are satisfied with your build you can purge the created Docker images to reclaim
disk space with

```
docker rmi macemu-build-stage1
```

and possibly

```
docker system prune
```
(this is a generic command to delete all unused
and untagged images)

## The build process explained
Stage 1 builds while creating a Docker image. This makes base distribution easily swappable and keeps
build dependencies in an easy to purge image. Also each step inside the Dockerfile is cached, so iterations
are quick when something breaks or changes are made.

Stage 2 runs linuxdeploy inside a Docker container (with elevated privileges so FUSE works) to create
the AppImages. This is because FUSE cannot be used in the Docker build process, unfortunately.


[^1]: The path `$HOME/.local/bin` can in fact vary. See output of `systemd-path user-binaries`.
[^2]: Some libraries are in fact excluded and this can sometimes cause issues. Please file an issue if you encounter "missing symbol" or similar errors that point to ".so" files
[^3]: More specifically, at the time of writing, this is FUSE2 opposed to FUSE3. FUSE3 support is [on its way](https://github.com/AppImage/AppImageKit/issues/877). Some users (most notably Ubuntu 22.04 users) may need to manually `sudo add-apt-repository universe && sudo apt apt install libfuse2` to run AppImages for now.
