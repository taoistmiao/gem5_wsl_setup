1. install wsl distro in PowerShell
    ```$ wsl --install -d Ubuntu-22.04```
    may need to [enable virtualization feature](https://www.configserverfirewall.com/windows-10/please-enable-the-virtual-machine-platform-windows-feature-and-ensure-virtualization-is-enabled-in-the-bios/)
2. [configure](https://joe.blog.freemansoft.com/2022/01/setting-your-memory-and-swap-for-wsl2.html)
   wsl2 VM memory and swap file if needed
    ``` 
        # C:\Users\<UserName>\.wslconfig
        # Settings apply across all Linux distros running on WSL 2
        # Can see memory in wsl2 with "free -m"
        # Goes in windows home directory as .wslconfig
        [wsl2]

        # Limits VM memory to use no more than 48 GB, defaults to 50% of ram
        memory=32GB

        # Sets the VM to use 8 virtual processors
        processors=8

        # Sets the amount of swap storage space to 8GB, default is 25% of available RAM
        swap=32GB
    ```
    in PowerShell `$ wsl --shutdown` then start distro to load the settings
3. update and install [dependencies](https://www.gem5.org/documentation/general_docs/building)
4. go into gem5 diresctory, run `$ pip install -r requirement.txt` for python libs, then add `~/.local/bin` to `$PATH` if needed
5. build `$ scons EXTRAS=../ATP-Engine -j $(nproc) build/ARM/gem5.opt` (remove EXTRAS option if ATP not needed)
6. build ARM disk image with Ubuntu 22.04
   ```bash
   $ sudo apt-get install qemu-user-static
   $ sudo apt-get install  gcc-arm-linux-gnueabihf gcc-aarch64-linux-gnu g++-aarch64-linux-gnu device-tree-compiler
   $ make -C system/arm/bootloader/arm64 # build bootloader
   $ mkdir disk_img
   $ cd disk_img
   $ wget https://storage.googleapis.com/dist.gem5.org/dist/develop/kernels/arm/static/arm64-vmlinux-5.15.36 # get kernel
   $ mkdir rootfs
   $ cd rootfs
   $ wget https://cdimage.ubuntu.com/ubuntu-base/releases/22.04/release/ubuntu-base-22.04.4-base-arm64.tar.gz # get base rootfs
   $ tar -xvf ubuntu-base-22.04.4-base-arm64.tar.gz
   $ cd ..
   $ # prepare the rootfs
   $ sudo cp /usr/bin/qemu-aarch64-static rootfs/usr/bin/ # ARM compiler 
   $ mkdir rootfs/mnt/etc/
   $ sudo cp /etc/resolv.conf rootfs/etc/ # internet access
   $ cd ../util/m5/
   $ scons build/arm64/out/m5
   $ cp build/arm64/out/m5 ../../disk_img/rootfs/sbin/ # m5 utility
   $ chmod 1777 /tmp
   $ ./mount.sh #chroot
    # apt-get update
    # apt-get upgrade
    # apt-get install sudo vim
    # exit
   $ ./umount.sh
   $ # create disk image
   $ sudo apt-get install qemu
   $ sudo apt install qemu-utils
   $ qemu-img create -f raw ubuntu-2204.img 8G
   $ sudo fdisk ubuntu-2204.img #create a new partition
   $ sudo losetup --show --find ubuntu-2204.img
   $ sudo partprobe /dev/loop0 # probe /dev/loopX
   $ mkdir tmp_mnt
   $ sudo mkfs.ext4 /dev/loop0p1 # create ext4 filesystem
   $ sudo mount /dev/loop0p1 tmp_mnt/
   $ sudo cp -r rootfs/* tmp_mnt/
   $ sudo umount tmp_mnt
   $ sudo losetup -d /dev/loop0
   $ ./build/ARM/gem5.opt configs/example/gem5_library/arm-ubuntu-run.py
   ```
   18. ./build/ARM/gem5.opt configs/example/arm/fs_bigLITTLE.py --kernel /home/guotong/gem5-stable/disk_img/kernels/arm64-vmlinux-5.15.36 --disk /home/guotong/gem5-stable/disk_img/ubuntu-2204.img --bootscript /home/guotong/gem5-stable/disk_img/testarm.rcS --bootloader /home/guotong/gem5-stable/system/arm/bootloader/arm64/boot.arm64 --cpu-type atomic --caches --big-cpus 0 --little-cpus 4 /home/guotong/gem5-stable/system/arm/bootloader/arm64/boot.arm64 --cpu-type atomic --caches --big-cpus 0 --little 4 