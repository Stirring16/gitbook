# Linux

## Steps to create Linux Profile



#### 1.  Check profile of dump file

```
ubuntu@ubuntu:~/Desktop/$ strings memory.raw | grep "Linux version"
Linux version 4.4.0-72-lowlatency (buildd@lcy01-17) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.4) ) #93-Ubuntu SMP PREEMPT Fri Mar 31 15:25:21 UTC 2017 (Ubuntu 4.4.0-72.93-lowlatency 4.4.49)
```

So we need to build profile  linux-4.4.0-72-lowlatency on Ubuntu 16.04.4

#### 2. Download package of 4.4.0-72-lowlatency

We need install image package

```
sudo apt-get install linux-image-4.4.0-72-lowlatency 
```

And header package&#x20;

```
sudo apt-get install linux-headers-4.4.0-72-lowlatency 
```

#### 3. Change profile for a  system

```
sudo gedit /etc/default/grub
```

Fix grub like this:

```
GRUB_DEFAULT=0
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=5
GRUB_GFXMODE=1024x768
```

Update grub

```
sudo update-grub
sudo reboot
```

Spam shift or enter C, select \`Ubuntu, with Linux 4.4.0-72-lowtatency\`

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Make the profile&#x20;

Download and install volatility

```
git clone https://github.com/volatilityfoundation/volatility.git
```

```
cd volatility/tools/linux
```

Install some package necessary and create vtypes

```
 sudo apt install dwarfdump build-essential libelf-dev zip
 make
```

To create the profile, place both the `module.dwarf` and the `system.map` file into a zip file.&#x20;

```
sudo zip $(lsb_release -i -s)_$(uname -r)_profile.zip module.dwarf /boot/System.map-$(uname -r)
```

Then move this zip file under '`volatility/plugins/overlays/linux/`'

```
mv Ubuntu_4.4.0-72-lowlatency_profile.zip /home/ubuntu/Desktop/volatility/volatility/plugins/overlays/linux
```

Everything done for create profile. Check profile with volatility

#### Get profile with volatility

```
ubuntu@ubuntu:~/Desktop/volatility$ python2 vol.py --info | grep Linux
Volatility Foundation Volatility Framework 2.6.1
LinuxUbuntu_4_4_0-72-lowlatency_profilex64 - A Profile for Linux Ubuntu_4.4.0-72-lowlatency_profile x64
linux_aslr_shift           - Automatically detect the Linux ASLR shift
linux_banner               - Prints the Linux banner information
linux_yarascan             - A shell in the Linux memory image
LinuxAMD64PagedMemory          - Linux-specific AMD 64-bit address space.
```
