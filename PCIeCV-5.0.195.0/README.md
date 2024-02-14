# PCIe Configuration Verifier (PCIeCV)

The current version of PCIeCV is 5.0.195.0

PCIeCV 5.0 is compatible with the following drivers:

- PCI-SIG Hardware Access (HWA) driver v1.42 (Windows/Linux)
- Generic PCIe Driver Interface (ARC/GPDI) driver v1.10 (Linux)

Please note the following:

- PCIeCV 5.0 on Linux lacks a GUI.
- PCIeCV 5.0 on Windows (ARM64) requires the [latest](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170#visual-studio-2015-2017-2019-and-2022) Visual C++ Redistributable package for ARM64.
- PCIeCV 5.0 on Windows (ARM64) lacks a GUI and must be run from an elevated PowerShell or Command Prompt session.

## Linux

PCIeCV 5.0 can be compiled against the PCI-SIG HWA or ARC/GPDI driver. The PCI-SIG HWA driver is approved for PCIe 5.0 IL testing. It supports Legacy IO and ECAM (in which case ECAM entries must be present in the ACPI MCFG table). The ARC/GPDI driver is only approved for RC testing on platforms that cannot use the PCI-SIG HWA driver. It uses the Linux PCIe Subsystem and therefore does not require ACPI. 

### Supported Distributions

PCIeCV 5.0 is officially supported on the following Linux distributions:

- Debian 12 (x86/64, ARM64)
- Ubuntu 20.04 LTS (x64, ARM64)
- Ubuntu 22.04 LTS (x64, ARM64)
- Ubuntu 23.10 (x64, ARM64)
- Fedora 37 (x64, ARM64)
- Fedora 38 (x64, ARM64)

### Build Instructions (Native)

Install the kernel headers and build tools for your Linux distribution.

1. Debian-based distributions

```
~$ sudo apt-get install g++ build-essential linux-headers-$(uname -r)
```

2. RPM-based distributions

```
~$ sudo dnf install make gcc gcc-c++ kernel-devel
```

#### PCI-SIG HWA Driver

Download the latest PCI-SIG HWA driver source code from the PCI-SIG GitLab [instance](https://gitlab.pcisg.com/config/gendrv/dd):

```
~$ git clone https://gitlab.pcisig.com/config/gendrv/dd.git
```

The PCI-SIG HWA driver source code is also available on the PCI-SIG Causeway [instance](https://members.pcisig.com/wg/PCI-SIG/document/16355).

```
~$ unzip dd-master.zip && mv dd-master dd
```

Build the driver and shared library:

```
~/dd/linux$ make
```

Install the driver and shared library:

```
~/dd/linux$ sudo make install
```

To uninstall the driver and shared library:

```
~/dd/linux$ sudo make uninstall
```

Set `DEBUG=1` to compile with debug information:

```
~/dd/linux$ make DEBUG=1
```

Set `PCISIG_MODULE_INSTALL_PATH=$INSTALL_PREFIX` to change the driver install path:

```
~/dd/linux$ sudo make PCISIG_MODULE_INSTALL_PATH=$INSTALL_PREFIX <target>
```

Set `PCISIG_LIB_INSTALL_PATH=$LIB_INSTALL_PREFIX` to change the shared library install path:

```
~/dd/linux$ sudo make PCISIG_LIB_INSTALL_PATH=$LIB_INSTALL_PREFIX <target>
```

#### ARC/GPDI Driver

Download the latest ARC/GPDI driver source code from the PCI-SIG GitLab [instance](https://gitlab.pcisg.com/config/gendrv/arc):

```
~$ git clone https://gitlab.pcisig.com/config/gendrv/arc.git
```

The ARC/GPDI driver source code is also available on the PCI-SIG Causeway [instance](https://members.pcisig.com/wg/PCI-SIG/document/20298).

```
~$ unzip arc-main.zip && mv arc-main arc
```

Build the driver and shared library:

```
~/arc$ make
```

Install the driver and shared library:

```
~/arc$ sudo make install
```

To uninstall the driver and shared library:

```
~/arc$ sudo make uninstall
```

Set `DEBUG=1` to compile with debug information:

```
~/arc$ make DEBUG=1
```

Set `GPDI_INSTALL_PATH=$INSTALL_PREFIX` to change the driver install path:

```
~/arc$ sudo make GPDI_INSTALL_PATH=$INSTALL_PREFIX <target>
```

Set `GPDI_LIB_INSTALL_PATH=$LIB_INSTALL_PREFIX` to change the shared library install path:

```
~/arc$ sudo make GPDI_LIB_INSTALL_PATH=$LIB_INSTALL_PREFIX <target>
```

#### PCIeCV 5.0

Download the latest PCIeCV 5.0 source code from the PCI-SIG GitLab [instance](https://gitlab.pcisg.com/config/gendrv/dd):

```
~$ git clone https://gitlab.pcisig.com/config/cv/cv5.git
```

Build PCIeCV 5.0 against the PCI-SIG HWA or ARC/GPDI driver. Use [`Makefile`](out/Makefile) to build PCIeCV 5.0 against the PCI-SIG HWA driver. Use [`Makefile.gpdi`](out/Makefile.gpdi) to build PCIeCV 5.0 aginst the ARC/GPDI driver. In the following examples, parameters in square brackets are required if compiling PCIeCV 5.0 against the ARC/GPDI driver and should be omitted otherwise.

```
~/cv5/out$ make [-f Makefile.gpdi]
```

Set `DEBUG=1` to compile with debug information:

```
~/cv5/out$ make DEBUG=1 [-f Makefile.gpdi]
```

Set `PCISIG_LIB_PATH` if the PCI-SIG HWA driver library is located outside the linker search path. For example:

```
~/cv5/out$ make PCISIG_LIB_PATH=../../dd/linux
```

Set `GPDI_LIB_PATH` if the ARC/GPDI driver library is located outside the linker search path. For example:

```
~/cv5/out$ make GPDI_LIB_PATH=../../arc -f Makefile.gpdi
```

#### Troubleshooting

##### ARM64

The `_M_ARM64` preprocessor flag must be defined when compiling the PCI-SIG HWA driver and PCIeCV 5.0 for ARM64. If not, the driver will attempt to perform misaligned accesses to device memory. The default `make` targets compile with `-D_M_ARM64` if `uname -m` is one of `armv8`, `arm64` or `aarch64`. We strongly suggest reviewing the build logs to ensure that `_M_ARM64` was defined when compiling. If not, invoke `make` with `TARGET_ARCH=arm64` to ensure that `_M_ARM64` is defined:

```
~/dd/linux$ make TARGET_ARCH=arm64
```

```
~/cv5/out$ make TARGET_ARCH=arm64
```

##### Insufficient Virtual Memory (32-bit)

The default `vmalloc` size of `128M` causes compiler errors on some 32-bit systems. If `make` fails and `dmesg -k` contains virtual memory allocation failures, override the default `vmalloc` size by setting the corresponding kernel boot parameter to `512M`. The following command displays the current `vmalloc` size:

```
~$ grep Vmalloc /proc/meminfo
```

To modify the kernel boot parameters, append `vmalloc=512M` to `GRUB_CMDLINE_LINUX` in `/etc/default/grub` and update your GRUB configuration by running `sudo update-grub`. The change should be reflected in `/proc/meminfo` after rebooting.

### Build Instructions (Cross-Compilation)

The PCI-SIG HWA and ARC/GPDI driver can be cross-compiled by specifying the target architecture, kernel build directory and toolchain prefix.

The following examples illustrate cross-compiling the PCI-SIG HWA driver and PCIeCV 5.0 for ARM64 against a precompiled Linux 4.15 kernel located at `../../linux-4.15` using the default Debian/Ubuntu GNU toolchain for ARM64.

To cross-compile the PCI-SIG HWA driver and shared library for ARM64, set the `ARCH`, `KDIR` and `CROSS_COMPILE` variables. For example:

```
~/dd/linux$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- KDIR=../../linux-4.15
```

To cross-compile PCIeCV 5.0 for ARM64, only the `library` target is required. In that case, `KDIR` is not required. For example:

```
~/dd/linux$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- library
```

To cross-compile PCIeCV 5.0 for ARM64, set the `ARCH`, `CROSS_COMPILE` and `PCISIG_LIB_PATH` variables. For example:

```
~/cv5/out$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- PCISIG_LIB_PATH=../../dd/linux
```

### Test Instructions

#### System Configuration

Ensure that the current ASPM policy is `performance`. Note that `pcie_aspm=off` and `pcie_aspm=performance` are not equivalent (the kernel will not disable ASPM in the first case). The current ASPM policy can be configured by writing `performance` to the `sys/module/pcie_aspm/parameters/policy` file. For example:

```
~$ cat /sys/module/pcie_aspm/parameters/policy
[default] performance powersave powersupersave
~$ echo performance | sudo tee /sys/module/pcie_aspm/parameters/policy
~$ cat /sys/module/pcie_aspm/parameters/policy
default [performance] powersave powersupersave
```

Add `pcie_aspm=force` and `pcie_aspm.policy=performance` to your kernel boot parameters to persist this setting.

#### DUT Configuration

If testing a PCIe Type 0 device (e.g., an Endpoint or Legacy Endpoint), all drivers used by the device must be unloaded or unbound from the device. To do this, either: (a) blacklist the drivers; (b) unbind the drivers from the device; or (c) unload the drivers using `rmmod` or `modprobe -r`. The latter options do not persist across reboots.

In the following examples, we consider a graphics card located at `0000:47:00.0` that uses the `amdgpu` driver. The driver must be unloaded or unbound from the device before testing.

To unbind a driver from a device, write the device's SBDF to the driver's `unbind` file in the `sysfs` filesystem. The SBDF must include the PCIe domain and the device and function number must be separated by a period. See the Linux kernel ABI [documentation](https://www.kernel.org/doc/Documentation/ABI/testing/sysfs-bus-pci) for details. This is comparable to disabling the device driver in Device Manager on Windows. For example:

```
~$ lspci -vs 0000:47:00.0 | grep "Kernel driver in use"
	Kernel driver in use: amdgpu
~$ echo "blacklist amdgpu" | sudo tee -a /etc/modprobe.d/amdgpu.conf
~$ sudo update-initramfs -u
~$ sudo reboot
~$ lspci -vs 0000:47:00.0 | grep "Kernel driver in use"
```

The equivalent of `update-initramfs` on RPM-based distributions is:

```
~$ sudo dracut -f
```

Do not blacklist drivers required for your system to function. For example, if your primary graphics card uses `amdgpu` and you are unable to access the system remotely, then blacklisting the `amdgpu` module would be counterproductive. However, blacklisting your audio drivers would not interfere with testing.

If the device has multiple functions, no driver should be bound to any function before testing. In some cases, the above methods fail to completely disable the DUT or cannot be performed. If so, consider using alternative drivers for conflicting devices or `initramfs` hooks to disable the device during boot. Finally, modules such as `virtio_pci` are compiled into the kernel and cannot be removed or blacklisted. In that case, the only option is recompiling the kernel without the module in question.

#### Driver Configuration

Load the PCI-SIG HWA or ARC/GPDI driver. 

- The PCI-SIG HWA driver is named `pcisig_module`.
- The ARC/GPDI driver is named `gpdi`.

In the following examples, `<driver>` ranges over `pcisig_module` and `gpdi`. If the driver was installed to `/lib/modules/$(uname -r)` and registered with `depmod`, use the `modprobe` versions of the following commands. If not, use the `insmod` and `rmmod` versions.

To load the driver:

```
~$ sudo modprobe <driver>
```

```
~$ sudo insmod $DRIVER_PATH/<driver>.ko
```

The previous command might fail with the following error message if the driver is incompatible with your kernel. In that case, try recompiling the driver. The `modinfo` and `readelf -p .comment` commands provide useful information about the driver file.

```
modprobe: ERROR: could not insert module <driver>.ko: Invalid module format
```

The previous command might fail with the following error message if Secure Boot is enabled on your system. If so, disable Secure Boot in the BIOS and reload the driver:

```
modprobe: ERROR: could not insert '<driver>': Operation not permitted
```

Ensure that the driver was successfully loaded by searching `/proc/modules` or the output of `lsmod`. The driver logs messages in the kernel ring buffer, which can be monitored by running `dmesg -kw` during testing:

```
~$ grep <driver> /proc/modules
```

```
~$ lsmod | grep <driver>
```

To unload the driver:

```
~$ sudo modprobe -r <driver>
```

```
~$ sudo rmmod <driver>
```

#### PCIeCV 5.0 Configuration

The default PCIeCV 5.0 [configuration file](pciecvapp/ini/PCIECVapp.ini) is installed to `ini/PCIECVapp.ini`. It controls the PCIe segment, start bus number, end bus number and various timing parameters. The timing parameters should not be modified for official testing unless noted otherwise.

To modify the default PCIe segment, set `PCI_SegmentNumberValue`. For example:

```
[PCI_SegmentNumber]
PCI_SegmentNumberValue=1
```

To modify the default start bus number, set `PCI_StartBridgeValue`. For example:

```
[PCI_StartBridgeNumber]
PCI_StartBridgeValue=1
```

To modify the default end bus number, set `PCI_MaxBridgeValue`. For example:

```
[PCI_MaxBridgeNumber]
PCI_MaxBridgeValue=15
```

#### PCIeCV 5.0 Testing

The following examples are intended to illustrate usage of PCIeCV 5.0 relevant to workshop settings. Run `./PCIECVApp -help` for documentation of additional functionality. The following examples assume the PCI-SIG HWA or ARC/GPDI driver are loaded.

Note that the BDF is specified in deimal, does not accept a PCIe domain prefix, and uses a colon to separate the device and function number.

1. Scan PCIe bus topology.

```
~/cv5/out$ sudo ./PCIECVApp -scan
```

2. Run TD 1.4 on DUT with BDF 11:00:0.

```
~/cv5/out$ sudo ./PCIECVApp -bdf 11:00:0 -td 1.4
```

3. Run the compliance test suite on DUT with BDF 11:00:0.

```
~/cv5/out$ sudo ./PCIECVApp -bdf 11:00:0 -td comp
```

4. Run the full test suite on DUT with BDF 11:00:0 at all supported reset mechanisms and speeds.

```
~/cv5/out$ sudo ./PCIECVApp -bdf 11:00:0 -td all -r all -s all
```

## Windows

### Supported Versions

PCIeCV 5.0 is officially supported on the following Windows versions:

- Windows 10 (x86/64)
- Windows 11 (x64, ARM64)
- Windows Server 2019 (x86/64)
- Windows Server 2022 (x86/64)

### Build Instructions

Building PCIeCV 5.0 for ARM64 on Windows requires the MSVC v142 build tools.

#### PCIeCV 5.0 (x86/64)

Building [CV4](https://gitlab.pcisig.com/config/cv/cv4) and [CV5](https://gitlab.pcisig.com/config/cv/cv5) on Windows requires the Visual Studio 2010 Service Pack 1 Platform Toolset. This can be obtained by installing Visual Studio Professional 2010 and Visual Studio Professional 2010 Service Pack 1. Both are available from [my.visualstudio.com](https://my.visualstudio.com) with a free subscription.

Download Visual Studio Professional 2010 and Visual Studio Professional 2010 Service Pack 1 from the above links. You do not require a license for Visual Studio Professional 2010 because the trial period does not extend to the toolset. The Service Pack does not interoperate correctly with the Express edition of Visual Studio 2010.

Mount the Visual Studio Professional 2010 ISO and run `setup.exe` (if the installer did not automatically run). Select only the "X64 Compilers and Tools" component when choosing which features to install. This installs Visual Studio Professional 2010, MSVC 10.0 (v100) and the Visual C++ 10.0 Runtime. Run the Visual Studio 2010 Service Pack 1 installer. If the Visual Studio 2010 version ends with "SP1Rel" then the Service Pack was installed correctly.

Download and install Visual Studio 2022 from [my.visualstudio.com](https://my.visualstudio.com) or [visualstudio.microsoft.com](https://visualstudio.microsoft.com/vs). The edition (Community, Professional, Enterprise) does not matter. Select the "Desktop development with C++" workload and the "C++ MFC for latest v143 build tools" and "C++/CLI support for latest v143 build tools" components. Visual Studio 2022 is not required but configuration of other development environments is beyond the scope of these instructions.

The installer project depends on Advanced Installer 21.1 or greater and the Advanced Installer for Visual Studio 2022 extension. The installer uses features that require a license and will not build correctly if unlicensed. However, installers can be built and used locally with unlicensed copies of Advanced Installer. If you do not intend to build the installer, do not install Advanced Installer or the extension and unload the `installer` project during the following step.

Clone PCIeCV by launching Visual Studio 2022 and selecting "Clone a repository" from the sidebar. Enter

```
https://gitlab.pcisig.com/unicv/cv.git
```

for the repository location. If not using the graphical interface or Visual Studio complains that the repository is owned by another user, you might need to add an [exception](https://git-scm.com/docs/git-config/2.35.2#Documentation/git-config.txt-safedirectory) for the repository. Do this by running the following command.

```
$ git config --global --add safe.directories $PATH
```

Open the CV4 or CV5 solution file in Visual Studio 2022. You must observe the following points.

- The CV5 solution file is `pciecvapp/PCIECVApp.sln`. The CV4 solution file is `PCIECVApp/PCICEVApp.sln`.
- Do not retarget projects to a newer Windows SDK or platform toolset when prompted. Select "Cancel" when prompted to retarget projects. If you accidentally retarget any projects, reset to a prior commit rather than manually reverting the changes.
- If `pciecvapp` is not the default project, select "Set as Startup Project" from its context menu (i.e., right click on `pciecvapp` in the "Solution Explorer" window and select "Set as Startup Project").
- Change the Platform from `x64` to `x86`. Both CV4 and CV5 are 32-bit applications and will not compile with `Platform=x64`.
- If the solution file is not named `PCIECVApp.sln` then you are working from an unsupported revision.

The following command builds the `Release` configuration of PCIeCV. Substitute `Debug` for `Release` to build with debugging symbols. Only the `Release` configuration depends on Advanced Installer. The "Visual Studio Command Prompt (2010)" shortcut launches `cmd.exe` with environment variables configured for Visual Studio 2010. Follow [these](https://learn.microsoft.com/en-us/cpp/build/building-on-the-command-line) instructions to configure a build environment in another shell.

```
$ MSBuild pciecvapp/pciecvapp.sln -t:Rebuild -p:Platform=x86 -p:Configuration=Release -v:Normal
```
