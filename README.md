sel4-newlibs-manifest
=================
The sel4 project with no libc dependicies.

To checkout the use repo, repo sync and get the default config:
```
$ repo init -u https://github.com/winksaville/sel4-newlibs-manifest.git
$ repo sync
$ make helloworld_x86_defconfig
```

To build using CMake create a subdirectory and run "cmake .."
which creates the Makefile and then run "make"
```
$ mkdir cmake-build
$ cd cmake-build
$ cmake ..
$ make
```

For building with the root makefile stay in the root and just type make:
```
$ make
```

Then to run helloworld, which works for either cmake or root make:
```
$ make simulate
Starting node #0
Testing log
Wrote to ksLog deadbeef
Hi
```
