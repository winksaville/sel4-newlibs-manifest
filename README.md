sel4-newlibs-manifest
=================
The sel4 project with no libc dependicies.

To checkout the use:
```
$ repo init -u https://github.com/winksaville/sel4-newlibs-manifest.git
```
For building with the root makefile:
```
$ make helloworld_x86_defconfig
...
$ make
...
$ make simulate
...
Starting node #0
Testing log
Wrote to ksLog deadbeef
Hi
```
For building with CMake, currently the result doesn't run but it "compiles".
```
$ mkdir cmake-build
$ cd cmake-build
$ make
```
These three lines will rebuild the world from within cmake-build and what I'm using to work on the CMakeLists.txt files.
```
$ rm -rf * ; cmake -G "Unix Makefiles" --build . ../
$ make -C ../ ; make clean
$ make
```
