### Introduction

This repo provides an integration which allows [LISA](https://github.com/ARM-software/lisa) to work with QEMU VMs.<br/>
[LISA's](https://github.com/ARM-software/lisa) goal is to help Linux kernel developers to measure the impact of modifications in core parts of the kernel.<br/>
Integration with [QEMU](https://www.qemu.org/) will allow developers to test  wide variety of hardware configurations including ARM architecture<br/>
and complex NUMA topologies.

### Getting Started
```
git clone https://github.com/rf972/lisa-qemu.git
cd lisa-qemu
git submodule update --init --recursive
```

### Required packages
The following packages are required on ubuntu.<br/>
```
apt-get build-dep -y qemu
apt-get install -y libfdt-dev flex bison git apt-utils
apt-get install -y python3-yaml wget qemu-efi-aarch64 qemu-utils genisoimage qemu-user-static
```
### Note on below commands
```
Each of the below commands is expected to be issued from the top level of the lisa-qemu repo.
In most cases please use a separate shell for each command.
```
### Building virtual machine
The following command will build the image and use the defaults.<br/>
At the top level run:<br/>
```
python3 scripts/build_image.py
```
### Launch the VM
To launch the VM, and open an SSH connection to it, using all default arguments:
```
python3 scripts/launch_image.py
```

### LISA installation
```
cd external/lisa
sudo ./install_base.sh
source init_env
lisa-install
```
In case the venv becomes unusable for some reason,<br/>
the `lisa-install` shell command available after sourcing init_env<br/>
will allow to create a new clean venv from scratch.<br/>

##### For more information please refer to LISA documentation
https://lisa-linux-integrated-system-analysis.readthedocs.io/en/master/setup.html

### Run LISA test
```
# From top level run the following:
source init_lisa_env
lisa-test NUMAMultipleTasksPlacement:test_task_remains --conf conf/lisa/qemu_target_default.yml
```

### Build kernel
We have a script, which automates the process of putting a new kernel into your image.

To build the kernel we suggest the following steps.  <br/>
Note: these steps assume cross compiling for ARM on ubuntu.<br/>
1) Install required packages: <br/>
 ```
sudo apt-get install -y gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
sudo apt-get build-dep -y linux
sudo apt-get install -y libncurses-dev flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf git fakeroot
```
2) download the kernel from https://github.com/torvalds/linux <br/>
3) copy one of the config files from lisa-qemu/linux-config over to your top level kernel .config file. <br/>
4) make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- olddefconfig <br/>
5) make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bindeb-pkg -j [number of jobs] <br/>

The resulting .deb package will be named something like this: <br/>
linux-image-5.4.0+_5.4.0+-4_arm64.deb<br/>

### Install kernel into virtual machine <br/>
A script is provided to simplify the process of adding a kernel image to the virtual machine. <br/>

Note that this script requires sudo in order to be able to mount the image.<br/>
To install the kernel using defaults, only specify the kernel location.
```
sudo python3 scripts/install_kernel.py -p linux-image-5.4.0+_5.4.0+-4_arm64.deb
```
### Launch VM with new kernel
launch_image.py will launch a specific vm image if we use the --image_path option<br/>
```
python3 scripts/launch_image.py --image_path ./build/VM-ubuntu.aarch64/ubuntu.aarch64.img.kernel-5.4.0+
```

### Tips
You may want to consider disabling SSH StrictHostKeyChecking  
This can be done by changing your ssh config as following:
```
[LISAShell lisa-qemu] \> cat ~/.ssh/config
Host 127.0.0.1
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
    LogLevel QUIET
```

If you create a new image, host key may change and you will get following error:
``` 
devlib.exception.TargetStableError: Bad host key: Host key for server '127.0.0.1' does not match
```
This can be fixed by running:
```
ssh-keyscan -p 5555  127.0.0.1 > ~/.ssh/known_hosts
```


### License
This project is licensed under Apache-2.0.<br/>
This project includes some third-party code under other open source licenses.<br/>

### Contributions / Pull Requests
Contributions are accepted under Apache-2.0.<br/>
Only submit contributions where you have authored all of the code.<br/>
If you do this on work time make sure your employer is cool with this.<br/>
