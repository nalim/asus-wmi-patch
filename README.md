# asus-wmi-patch
This repository contains a simple partch generator for asus WMI module.

To install and use patched module using dkms:

0. Please make sure that you are running the kernel that you want to install the module for. So if you did a kernel update, please reboot first so the installation uses the correct kernel version.

1. Install DKMS using the method of your distribution.
   Debian/Ubuntu/etc.: `sudo apt install dkms`
   In addition to that, you need to have the headers for your current kernels installed. Most distributions provide a package for that.
   E.g. Ubuntu: `sudo apt install linux-headers-5.4.0-37` (replace by the correct version as determined by `uname -r`)

2. Create a directory for the module and download the source code
   ```
   sudo mkdir /usr/src/asus-wmi-1.0
   cd /usr/src/asus-wmi-1.0
   sudo wget 'https://github.com/nalim/asus-wmi-patch/archive/master.zip'
   sudo unzip master.zip
   sudo mv asus-wmi-patch-master/* .
   sudo rmdir asus-wmi-patch-master
   sudo rm master.zip
   ```
   Now the source code should be in `/usr/src/asus-wmi-1.0`. It's important that the folder is called exactly like that because DKMS expects that.
   Alternatively you can of course also clone this git repository into that folder.

3. If not using kernel 5.4: Call the following script to download and patch files fitting to your kernel version
   ```
   sudo sh prepare-for-current-kernel.sh
   ```

4. Register the module with DKMS
   ```
   sudo dkms add -m asus-wmi -v 1.0
   ```

5. Build and install the module to the current kernel
   ```
   sudo dkms build -m asus-wmi -v 1.0
   sudo dkms install -m asus-wmi -v 1.0
   ```
   From now on, DKMS will automatically rebuild the module on every kernel update.

## Troubleshooting

### New kernel version
If you keep updating your kernel for a while, it might happen that the downloaded and patched module no longer fits your kernel version and the dkms build fails. In this case, it often helps to clean and reinstall everything as described in the section **Removing or reinstalling** below.

### Kernel module conflicts
On some kernels, it might happen that the built-in module overrides our compiled module.
In this case, it might help to execute the following code afterwards:
```bash
cd /lib/modules/YOURKERNELVERSION/kernel/drivers/platform/x86
sudo mv  asus-nb-wmi.ko asus-nb-wmi.ko_bak
sudo mv asus-wmi.ko asus-wmi.ko_bak
sudo ln -s ../../../../extra/asus-nb-wmi.ko ./
sudo ln -s ../../../../extra/asus-wmi.ko ./
sudo depmod -a
```
For some newer system(EG:linuxmint 22 with kernel 6.2),it might help to execute the following code afterwards:
```bash
cd /lib/modules/YOURKERNELVERSION/kernel/drivers/platform/x86
sudo mv asus-nb-wmi.ko asus-nb-wmi.ko_bak
sudo mv asus-wmi.ko asus-wmi.ko_bak
sudo ln -s ../../../../updates/dkms/asus-nb-wmi.ko ./
sudo ln -s ../../../../updates/dkms/asus-wmi.ko ./
sudo depmod -a
```

on newer Ubuntu versions it can happen that the `mfd_aaeon` kernel module is interfering.
it is only needed for asus embedded boards - more details in [#issues/32](https://github.com/nalim/asus-wmi-patch/issues/32#issuecomment-986424835)
so we can safely blacklist:
```bash
echo "blacklist mfd_aaeon" | sudo tee /etc/modprobe.d/aaeon-blacklist.conf
sudo update-initramfs -k all -u
```
then rebuild as above.



## Removing or reinstalling
If you want to re-download and reinstall the kernel module (maybe because there have been changes in the code), you have to remove the old one first, calling
```
sudo dkms remove -m asus-wmi -v 1.0 --all
sudo rm -r /usr/src/asus-wmi-1.0
```
Then repeat the steps above from step 2 on.

## Major kernel updates
After a major kernel update (e.g. from 5.8 to 5.10), DKMS cannot update the module automatically as the new kernel sources need to be downloaded and patched. In this case, please uninstall and reinstall the module as described above in the section **Removing or reinstalling**.
