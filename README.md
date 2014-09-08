linux-RT-config
===============

a config file for compiling a fast Linux RT aka "realtime preemptible kernel" for audio and video workstations.

Why?
----

Many distributions come from RT kernels not as preemptible as needed for audio and video processes which require very high priorities.


What's included?
----------------

 * Full PREEMPT options (RT, 1000HZ, etc...)
 * All audio and many video drivers enabled
 * Disable dynamic USB IDs for USB audio devices
 * Well tested: thousands hours of testing sessions in production


Install my own stable RT kernel on Debian or Ubuntu
-----------------------------------------------------

```
wget -O - http://debian.parisson.com/debian/conf/parisson.gpg.key | sudo apt-key add -
echo "deb http://debian.parisson.com/debian/ stable main" | sudo tee -a /etc/apt/sources.list
sudo apt-get update
sudo apt-get install linux-image-3.10.10-amd64-yomguy-rt7 linux-headers-3.10.10-amd64-yomguy-rt7
```

Compile your own RT kernel from source
--------------------------------------

```
sudo apt-get install kernel-package libncurses5-dev fakeroot wget xz-utils
wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.10.10.tar.xz
wget https://www.kernel.org/pub/linux/kernel/projects/rt/3.10/older/patch-3.10.10-rt7.patch.xz
tar xJf linux-3.10.10.tar.xz
cd linux-3.10.10
xzcat ../patch-3.10.10-rt7.patch.xz | patch -p1 --dry-run
xzcat ../patch-3.10.10-rt7.patch.xz | patch -p1
wget https://github.com/yomguy/linux-RT-config/raw/master/.config
make menuconfig
export CONCURRENCY_LEVEL=4
make-kpkg --rootcmd fakeroot --initrd --revision=1 --append-to-version=-amd64-yomguy kernel_image kernel_headers
cd ..
sudo dpkg -i linux-image-3.10.10-amd64-yomguy-rt7_1_amd64.deb linux-headers-3.10.10-amd64-yomguy-rt7_1_amd64.deb
```

