sel4-newlibs-manifest
=====================
The seL4 project with no libc dependicies. This is a very very
subset of seL4 as of today it only able to create applications
on the order of the complexity and functionality of "helloworld",
so do not expect to much.

Dependencies
------------

My development system is Ubuntu 15.04 and here are the
primary dependencies and their versions that I'm using:

* gnu make v3.81
* gcc 4.9.2
* [git](https://github.com/git/git) v2.5.0
* [CMake](http://www.cmake.org/download/) v3.2.3
* [ninja](https://github.com/martine/ninja) v1.6.0
* [repo](https://code.google.com/p/git-repo/) v1.12.26


Checkout build and run
----------------------

To checkout create a directory and use repo, repo sync (the repo
selfupdate is required as I'm using features that aren't available
unless you have the latest version):
```
mkdir sel4-newlibs
cd sel4-newlibs
repo init -u https://github.com/winksaville/sel4-newlibs-manifest.git
repo selfupdate
repo sync
```

By default this manifest setups a default configuration (.config) in
the root of the repos which builds all of the apps for x86 and can be
tested using qemu. So now use make to build and then run helloworld
under qemu:
```
make
make run app=helloworld
```

Other build options
-------------------

As I'm exploring build systems there are actually several options for
building. The "original" make system as we did above plus I've added
and tested two CMake generators. One for "Unix Makefiles" and the other
for "Ninja". To build for "Unix Makefiles" do the following:
```
mkdir cmake-build
cd cmake-build
cmake ..
make
```

Then to run helloworld using qemu, which works for either cmake or root make:
```
make run_helloworld
```

The output should be something like, type 'ctrl-a' then 'x' to exit qemu:
```
...
Starting node #0
Testing log
Wrote to ksLog deadbeef
Hello, World!
```

To use "Ninja" its the same as "Unix Makefiles" except you need to specify
the generator:
```
cd ..
mkdir ninja-build
cd ninja-build
cmake -G Ninja ..
ninja
ninja run_helloworld
```

Build performance
-----------------
I've choosen to use CMake because it allows for faster build times. A clean build with
ccache cleared and clean takes about 14 seconds on my desktop:

```
ccache -C -c -z -s; make clean; time make
$ ccache -C -c -z -s; make clean; time make
Cleared cache
Cleaned cache
Statistics cleared
cache directory                     /home/wink/.ccache
cache hit (direct)                     0
cache hit (preprocessed)               0
cache miss                             0
files in cache                         0
cache size                             0 Kbytes
max cache size                       1.0 Gbytes
[CLEAN] in /home/wink/prgs/seL4-projects/sel4-newlibs
 [INCLUDE] /home/wink/prgs/seL4-projects/sel4-newlibs/include/generated
 [INCLUDE] /home/wink/prgs/seL4-projects/sel4-newlibs/include/config
 [INCLUDE] /home/wink/prgs/seL4-projects/sel4-newlibs/include
[GEN] /home/wink/prgs/seL4-projects/sel4-newlibs/Makefile
  /usr/bin/ccache gcc  -o tools/kbuild/kconfig/conf tools/kbuild/kconfig/conf.o tools/kbuild/kconfig/zconf.tab.o  
[KERNEL]
 [MKDIR] arch/object
 [PBF_GEN] arch/object/structures.pbf

...

 [STAGE] libsel4assert.a
[libs/libsel4assert] done.
[apps/helloworld] building...
 [HEADERS]
 [STAGE] autoconf.h
 [CC] src/main.o
 [LINK] helloworld.elf
 [STAGE] helloworld.bin
 [STAGE] helloworld
[apps/helloworld] done.
[GEN_IMAGE] helloworld-image

real	0m14.550s
user	0m8.364s
sys 0m1.864s
```

Using CMake "Unix Makefiles" takes about 8 seconds:
```
$ ccache -C -c -z -s; make clean; time make
Cleared cache
Cleaned cache
Statistics cleared
cache directory                     /home/wink/.ccache
cache hit (direct)                     0
cache hit (preprocessed)               0
cache miss                             0
files in cache                         0
cache size                             0 Kbytes
max cache size                       1.0 Gbytes
Scanning dependencies of target conf
[  3%] Building C object tools/kbuild/kconfig/CMakeFiles/conf.dir/conf.c.o
[  7%] Building C object tools/kbuild/kconfig/CMakeFiles/conf.dir/zconf.tab.c.o
[ 10%] Linking C executable conf
[ 10%] Built target conf
Scanning dependencies of target autoconf.h
[ 14%] Generating ../../../include/generated/autoconf.h

...

 [CPP_GEN] kernel_all.c
 [CPP] kernel_all.c_pp
 [CP] kernel_final.c
 [CC] kernel_final.s
 [AS] kernel.o
 [LD] kernel.elf
[KERNEL] done.
[ 92%] Built target sel4_kernel
Scanning dependencies of target helloworld-main
[ 96%] Building C object apps/helloworld/CMakeFiles/helloworld-main.dir/src/main.c.o
[100%] Linking C executable helloworld-main
[100%] Built target helloworld-main

real	0m7.758s
user	0m5.808s
sys 0m0.576s
```

The most significant difference happens if there are no or only minor changes, thus
significantly reducing the test, edit, compile cycle. Here, if I change
apps/helloworld/src/main.c to print "Hello, Wink!\n" the standard make
takes almost 7 seconds:
```
$ time make
[KERNEL]
 [LD] kernel.elf
[KERNEL] done.
[libs/libsel4] building...
 [GEN] include/interfaces/sel4_client.h
 [GEN] include/sel4/types_gen.h
 [GEN] include/sel4/syscall.h

...

about about/helloworld] building...
 [HEADERS]
 [STAGE] autoconf.h
 [CC] src/main.o
 [LINK] helloworld.elf
 [STAGE] helloworld.bin
 [STAGE] helloworld
[apps/helloworld] done.
[GEN_IMAGE] helloworld-image

real	0m6.903s
user	0m2.768s
sys 0m1.056s
```

But for CMake "Unix Makefiles" it takes 0.3 seconds:
```
$ time make
[ 10%] Built target conf
[ 14%] Built target autoconf.h
[ 17%] Built target arch_invocation.h
[ 21%] Built target syscall.h
[ 25%] Built target sel4_client.h
[ 28%] Built target invocation.h
[ 32%] Built target types_gen.h
[ 39%] Built target libsel4
[ 50%] Built target libsel4startstop
[ 57%] Built target libsel4putchar
[ 67%] Built target libsel4string
[ 75%] Built target libsel4printf
[ 82%] Built target libsel4assert
[ 89%] Built target libsel4benchmark
[ 92%] Built target sel4_kernel
Scanning dependencies of target helloworld-main
[ 96%] Building C object apps/helloworld/CMakeFiles/helloworld-main.dir/src/main.c.o
[100%] Linking C executable helloworld-main
[100%] Built target helloworld-main

real	0m0.306s
user	0m0.056s
sys 0m0.040s
```

A world about configurations
----------------------------

Other configurations can be created using menuconfig or using an editor.
If the configuration ends with "defconfig" and exists in configs/ it
will be listed in the "make help" information and you can use it by
simply doing "make xxxx_defconfig":
```
make helloworld_x86_defconfig
```
