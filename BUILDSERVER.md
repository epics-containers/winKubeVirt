# Setting up as a build server

Starting point: a new install of Windows Server 2022.

## get ssh working

This means we can push build tasks to the server from a container (especially gitlab runner)

Add Openssh Server
https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui

Add a service to give access to ssh
```
kubectl virt expose vmi iso-win10-2 --port=22 --name=ssh-2
```

Check you can get to it using a cross namespace example:
```bash
root@pvc-sharing-68b57f9c99-wdvsq:~$ sshpass -p $PASS scp party.txt Administrator@ssh-2:/party.txt
root@pvc-sharing-68b57f9c99-wdvsq:~$ sshpass -p $PASS ssh Administrator@ssh-2 'type \party.txt'
Now is the time for all good men to come to the aid of their party
root@pvc-sharing-68b57f9c99-wdvsq:~$
```

TODO:
It would be nice to figure out how to do the shared PVC thing - but in truth `scp` or use of `git` on the target server would both work fine.

## build tools and libraries

These are the CMDLine build tools - install these (just the C++ option) and skip VS itself. Instead get vscode installed for a comfortable editing experience (optional).
- https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022

Get the build environ tools - just run the exe linked here.
- https://www.msys2.org/

And see the epics notes on it here
- https://docs.epics-controls.org/en/latest/getting-started/installation-windows-msys2.html

BUT don't do MinGW, instead we will use the MS compilers,
The EPICS documentation has detailed docs on 2 scenarios:
- https://docs.epics-controls.org/en/latest/getting-started/installation-windows.html#install-and-build

BUT: I'm taking the 3rd way: MSys build environment and MSVC compilers. This gives us:
- a useable build environment with bash shell
- maximum compatibility with vendor libraries (mostly MSVC only)

```bash
# get a bash shell window by running C:\msys64\ucrt64.exe

# install make
pacman -S make
# TODO this is the latest git but no https - therefore not that useful - investigate
pacman -S git
```
Because git is not allowed https and ssh keys are out of the question on this server we shall clone inside a cluster container and rcp epics-base over from my container (in lieu of git https working locally - but also since this will eventually be run in a gitlab runner that will already have the source maybe this is just OK).
```bash
sshpass -p $PASS scp -r epics-base Administrator@ssh-2.windows.svc.cluster.local:/build/epics-base
```

Use short paths for includes and libs by making symlinks. Make gets upset with spaces in the names otherwise:
```powershell
# make short names for system include folders with symlinks in PowerShell
# (for some reason msys2 'ln -s makes a copy')
mkdir C:\build
cd C:\build
mkdir lib
mkdir inc
New-Item -ItemType SymbolicLink -Path lib/um -Target 'C:\Program Files (x86)\Windows Kits\10\Lib\10.0.22621.0\um\x64'
New-Item -ItemType SymbolicLink -Path lib/ucrt -Target 'C:\Program Files (x86)\Windows Kits\10\Lib\10.0.22621.0\ucrt\x64'
New-Item -ItemType SymbolicLink -Path lib/tools -Target 'C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Tools\MSVC\14.39.33519\lib\x64'
New-Item -ItemType SymbolicLink -Path inc/ucrt -Target 'C:\Program Files (x86)\Windows Kits\10\Include\10.0.22621.0\ucrt'
New-Item -ItemType SymbolicLink -Path inc/um -Target 'C:\Program Files (x86)\Windows Kits\10\Include\10.0.22621.0\um'
New-Item -ItemType SymbolicLink -Path inc/shared -Target 'C:\Program Files (x86)\Windows Kits\10\Include\10.0.22621.0\shared'
New-Item -ItemType SymbolicLink -Path inc/tools -Target 'C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Tools\MSVC\14.39.33519\include'
```

# Now Build it

vanilla epics-base clone with no .locals at all ...

make a bash script called C:\build\build-base.sh
```bash
#!/bin/bash

export EPICS_HOST_ARCH=windows-x64

# Add latest MSVC SDK into path
export PATH='C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools/VC/Tools/MSVC/14.39.33519/bin/Hostx64/x64/:/c/Program Files (x86)/Windows Kits/10/bin/10.0.22621.0/x64':$PATH

# Add shortened paths into the SDK include and libs
# these environment vars used by Microsoft cl and link
export LIB='/c/build/lib/x64/:/c/build/lib/um:/c/build/lib/ucrt'
export INCLUDE='/c/build/inc/ucrt:/c/build/inc/VC:/c/build/inc/um:/c/build/inc/shared'

cd /c/build/epics-base
make clean
make -j 6

```

Now do:
```
C:\msys64\ucrt64.exe C:\build\build-base.sh
```

To launch from a container using ssh:
```bash
cmd='cmd.exe /c C:\\msys64\\msys2_shell.cmd -defterm -here -no-start -ucrt64'

ssh Administrator@ssh-2.windows.svc.cluster.local "$cmd C:\\build\\build-base.sh"
```