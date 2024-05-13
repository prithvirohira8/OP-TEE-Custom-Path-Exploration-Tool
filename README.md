# angrXplore: OP-TEE-Custom-Path-Exploration-Tool

### Build OP-TEE kernel
We are making use of QEMUv8 for our build process. 
Build instructions
As long as you pick the v8 manifest, use the following shell sequence to fetch, build and run OP-TEE using QEMU for Armv8-A is like the one below:
```
$ mkdir optee
$ cd optee
$ repo init -u https://github.com/OP-TEE/manifest.git
$ repo sync
$ cd build
$ make toolchains
$ make run
```
Note:
If you do not want to check out the latest version of OP-TEE, but rather a specific tagged version, you can use repo init -u https://github.com/OP-TEE/manifest.git -b <branchname>. e.g., repo init -u https://github.com/OP-TEE/manifest.git -b 3.16.0. You can see valid branch names by inspecting the OP-TEE/manifest git repository on https://github.com/OP-TEE/manifest/branches.

To speed up your build, you can make use of the parallel make feature. For example, use make -j32 run to have 32 build processes running concurrently. Note that this will make it much more difficult to spot errors if something fails; therefore fall back to sequential builds to view build errors and produce logs for bug reports.

#### Consoles
After running make run you will end up in the QEMU console and it will also spawn two UART consoles. One console containing the UART for secure world and one console containing the UART for normal world. You will see that it stops waiting for input on the QEMU console. To continue, do:
```
(qemu) c
```
Host-Guest folder sharing
You can use the VirtFS QEMU feature to avoid changing rootfs CPIO archive every time you need to add additional files or modify existing files. To do this, you share a folder between the guest and host operating systems. To enable and use this feature you have to provide additional arguments when running make, example:
```
$ make QEMU_VIRTFS_ENABLE=y QEMU_USERNET_ENABLE=y
```

#### GDB - Normal world
If you need to debug a client application, using GDB in a remote debugging configuration may be useful. Remote debugging means gdb runs on your PC, where it can access the source code, while the program being debugged runs on the remote system (in this case, in the QEMU environment in normal world). Here is how to do that. On your PC, build with GDBSERVER=y:
```
$ cd <qemu-v8-project>/build
$ rm -rf <qemu-v8-project>/out-br
$ make -j8 run GDBSERVER=y
```
Boot up as usual
```
(qemu) c
```
Inside QEMU (Normal World UART), run your application with gdbserver (for example xtest 4002):
```
# gdbserver :12345 xtest 4002
```
Process xtest created; pid = 654
Listening on port 12345
Back on your PC, open another terminal, start GDB and connect to the target:
```
$ <qemu-v8-project>/out-br/host/bin/arm-buildroot-linux-gnueabihf-gdb
(gdb) set sysroot <qemu-v7-project>/out-br/host/arm-buildroot-linux-gnueabihf/sysroot
(gdb) target remote :12345
```
Now GDB is connected to the remote application. You may use GDB normally.
```
(gdb) b main
(gdb) c
```
#### GDB - Secure world
TEE core debugging
To debug TEE core running QEMU with GDB, you need to disable TEE ASLR with CFG_CORE_ASLR=n flag. Furthermore, note that it’s easier to debug if you have optimization disabled. Other than that you will have four consoles that you are working with.
1. Qemu console
2. NW UART console
3. SW UART console
4. GDB console

All of them but the GDB console are consoles you normally will see/use when running OP-TEE/xtest using QEMU. The first thing is to start QEMU, i.e.,
```
$ cd <qemu-v8-project>/build
# make run-only also works if you don't want to rebuild things
$ make run CFG_CORE_ASLR=n
```
Next launch another console for GDB and do this
```
$ cd <qemu-v8-project>/toolchains/aarch32/bin
$ ./arm-linux-gnueabihf-gdb -q
```
In the GDB console connect to the QEMU GDB server, like this (the output is included to show what you normally will see).
```
(gdb) target remote localhost:1234

Remote debugging using localhost:1234
warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
0x00000000 in ?? ()
```

Please note that we have shown how to build OP-TEE on QEMU, which is in the scope of this project. There are other systems on which we can build and test OP-TEE, which is future scope for this project. We have only described in detail the steps we took for buiding OP-TEE on our x86 system running ubuntu. 

### Build Angr

This is pretty straightforward, install PyPI packages by running:
``` pip install angr ```
and we are ready to use angr library (vanilla) on the OP-TEE tee.elf binary which we compiled in the above step.

### Run tee.elf on angrXplore

Run the ipynb notebook angrXplore.ipynb containing angrXplore extension using vanilla angr, for optimal performance. There are specific cells which contain placeholders, config values of which can be tuned according to specific function being explored. The final cell consists of the results of symbolic execution, describing the number of active, spinning, deadended states, time taken, techniques and errors, if any. If we want to explore in depth to infer the reasons for the final verdict, the step() function can be called to print each step of breadth first search, as provided in the ipynb notebook already.
