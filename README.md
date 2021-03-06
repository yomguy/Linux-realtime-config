# Linux-realtime-config

Compile and install a fast Linux realtime preemptible kernel for audio and video workstations

## Why?

Many distributions come with partially (fake?) realtime (RT) low latency kernels not as preemptible as needed for audio and video processes which require very high system and hardware access priorities.

Note that even fast realtime video operations need fast and stable audio device accesses.

## What's included?

 * Full PREEMPT options enabled (RT, 1000HZ, etc...)
 * All audio and many video drivers enabled
 * Dynamic USB IDs disabled
 * Some personal tweaks ;)
 * Well tested in production during thousands of hours on "24/24 7/7" systems

## WARNING!

 * Some last commands in the howtos below will reboot your machine. SAVE your work before doing anything!
 * Any new kernel installation will NOT remove the current one. So you can always go back to your stable kernel with the GRUB menu at boot. BUT, if the GRUB menu is not displayed at boot time, please update the GRUB config before doing anything else:

```bash
sudo sed -ie 's/GRUB\_TIMEOUT=.*/GRUB\_TIMEOUT=7/g' /etc/default/grub
```

## To install my own stable RT kernel on Debian or Ubuntu (64 bits)

```
wget -O - http://debian.parisson.com/debian/conf/parisson.gpg.key | sudo apt-key add -
echo "deb http://debian.parisson.com/debian/ stable main" | sudo tee -a /etc/apt/sources.list
sudo apt-get update
sudo apt-get install linux-image-3.10.10-amd64-yomguy-rt7 linux-headers-3.10.10-amd64-yomguy-rt7
sudo reboot
```

## To compile your own RT kernel from source

```bash
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
sudo reboot
```

Note you can do it with more recent kernels and RT patches, but the result is NOT guaranteed with my config.

## To get high audio priorities

Usually, installing jackd will configure the audio group and high priorities:

```
sudo apt-get install jackd
sudo reboot
```

If you want to get them by hand:

```
sudo adduser $USER audio
echo -e "@audio   -  rtprio     95\n@audio   -  memlock    unlimited" | sudo tee -a /etc/security/limits.d/audio.conf
sudo reboot
```

## To test your RT capabilities

As explained here: https://rt.wiki.kernel.org/index.php/Cyclictest

```
sudo apt-get install git gcc
git clone git://git.kernel.org/pub/scm/linux/kernel/git/clrkwllms/rt-tests.git
cd rt-tests
make all
./cyclictest -t1 -p 80 -n -i 10000 -l 10000
```

A good average score for RT capabilities is around 20 and the max not above 100.


