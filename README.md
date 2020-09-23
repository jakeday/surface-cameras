# Surface Cameras

Support for the Surface Cameras under Linux.

### Instructions

### Compiling the Kernel from Source

0. (Prep) Install the required packages for compiling the kernel:
  ```
  sudo apt install build-essential binutils-dev libncurses5-dev libssl-dev ccache bison flex libelf-dev
  ```
1. Clone the surface-cameras repo:
  ```
   git clone https://github.com/jakeday/surface-cameras.git ~/surface-cameras
  ```
2. Clone the mainline stable kernel repo:
  ```
  git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git ~/linux-stable
  ```
3. Go into the linux-stable directory:
  ```
  cd ~/linux-stable
  ```
4. Checkout the version of the kernel you wish to target (replacing with your target version):
  ```
  git checkout v5.y.z
  ```
5. Apply the kernel patches from the surface-cameras repo:
  ```
  for i in ~/surface-cameras/patches/[VERSION]/*.patch; do patch -p1 < $i; done
  ```
5. Use config for kernel series (may need to manually change for your distro):
  ```
  cp ~/surface-cameras/configs/[VERSION]/config .config
  ```
6. Compile the kernel and headers (for ubuntu, refer to the build guide for your distro):
  ```
  make -j `getconf _NPROCESSORS_ONLN` deb-pkg LOCALVERSION=-linux-surface
  ```
7. Install the headers, kernel and libc-dev:
  ```
  sudo dpkg -i ../linux-headers-[VERSION].deb ../linux-image-[VERSION].deb ../linux-libc-dev-[VERSION].deb
  ```

### Patching the Device DSDT

0. (Prep) Install the required packages for patching the dsdt:
  ```
  sudo apt install acpica-tools
  ```
1. Create a working directory:
  ```
   mkdir ~/surface-dsdt
  ```
2. Go into the working directory:
  ```
   cd ~/surface-dsdt
  ```
3. Dump your current ACPI:
  ```
   sudo acpidump > acpidump.out
  ```
4. Extract the ACPI parts:
  ```
   acpixtract acpidump.out
  ```
5. Disassemble the DSDT:
  ```
   iasl -d DSDT.dat
  ```
6. Patch the DSDT:
  ```
   patch -p1 < ~/surface-cameras/patches/dsdt/[DEVICE_FILE].patch
  ```
7. Build the patched DSDT:
  ```
   iasl -tc DSDT.dsl
  ```
8. Make a CPIO Archive to boot with patched DSDT
  ```
   mkdir -p kernel/firmware/acpi
   cp DSDT.aml kernel/firmware/acpi
   find kernel | cpio -H newc --create > acpi_override
   sudo cp acpi_override /boot
  ```
9. Configure your bootloader to load the CPIO archive, normally by updating your grub config to add the path to the new acpi override.
