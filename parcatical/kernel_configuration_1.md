# kernel configuration

**Kernel configuration using Yocto** is an essential part of customizing the Linux kernel for embedded systems. Yocto provides a flexible way to configure, build, and manage the Linux kernel as part of its build system. The kernel configuration process allows developers to enable or disable features, drivers, and hardware support based on the specific needs of their embedded devices.

### Contents:
* [1. Understanding Kernel Recipes in Yocto](#1-understanding-kernel-recipes-in-yocto)
* [2. Kernel Configuration Methods in Yocto](#2-kernel-configuration-methods-in-yocto)
* [3. Adding Kernel Modules and Features](#3-adding-kernel-modules-and-features)
* [4. Building and Deploying the Kernel](#4-building-and-deploying-the-kernel)
* [5. Kernel Configuration and Patches](#5-kernel-configuration-and-patches)
* [6. Example Yocto Kernel Configuration](#6-example-yocto-kernel-configuration)
* [7. Enabling GPIO for sysfs Example](#7-enabling-gpio-for-sysfs-example)

### 1. **Understanding Kernel Recipes in Yocto**

In Yocto, the Linux kernel is built using a **recipe** (a `.bb` file) that describes how to fetch, configure, compile, and install the kernel. Yocto has a set of meta-layers where kernel recipes are found. The most common kernel recipe is `linux-yocto`, but you can use other custom or vendor-specific kernels as well.

For example, the recipe for building the Linux kernel might be located in:
```
meta/recipes-kernel/linux/linux-yocto_%.bb
```

The `%.bb` file defines how Yocto should fetch the kernel source, configure it, and compile it.

### 2. **Kernel Configuration Methods in Yocto**

There are several ways to configure the Linux kernel in Yocto:

#### a) **Using the `defconfig` file**:

The **defconfig** file is a default kernel configuration file. Yocto will use this as the base configuration for building the kernel. You can create a custom `defconfig` file that includes only the configuration options you need.

Steps:
1. **Generate a defconfig file** from a running kernel:
   - Build the kernel using a configuration you want to use as the base.
   - Run the following command in the kernel source directory:
     ```bash
     make defconfig
     ```
   - This will create a `.config` file that contains the current configuration options.

2. **Use defconfig in your Yocto recipe**:
   - Copy the generated `.config` or `defconfig` to your Yocto layer and add it to your kernel recipe.
   - Specify the path to the `defconfig` in the kernel recipe by modifying the recipe `.bb` file:
     ```bash
     FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
     SRC_URI += "file://defconfig"
     ```
   - Yocto will use this configuration when building the kernel.

#### b) **Using Kernel Configuration Fragments**:

Yocto allows the use of **kernel configuration fragments** to modify or extend the kernel configuration. A fragment is a small file that contains only the configuration changes you want to apply, without the need to override the entire kernel configuration.

Steps:
1. **Create a kernel fragment**:
   - Create a `.cfg` file that contains only the options you want to modify. For example, to enable a specific driver:
     ```bash
     CONFIG_USB_SERIAL=y
     CONFIG_SOME_OTHER_DRIVER=m
     ```

2. **Add the fragment to your Yocto layer**:
   - Place the fragment in a directory (e.g., `files/`) within your Yocto layer.

3. **Reference the fragment in the kernel recipe**:
   - Modify your kernel recipe to apply the configuration fragment during the build process. You can specify it using `SRC_URI`:
     ```bash
     FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
     SRC_URI += "file://my-kernel-fragment.cfg"
     ```

4. **Bitbake will automatically merge** the fragment into the main kernel configuration when building the kernel.

#### c) **Interactive Kernel Configuration (`menuconfig`)**:

Yocto allows for interactive kernel configuration using the **menuconfig** interface. This method allows you to configure the kernel interactively by selecting options from a text-based user interface.

Steps:
1. **Start the interactive configuration**:
   - Run the following command in your Yocto environment:
     ```bash
     bitbake virtual/kernel -c menuconfig
     ```

2. **Configure the kernel interactively**:
   - The `menuconfig` interface will open, allowing you to browse through all available kernel options and enable/disable them as needed.

3. **Save the configuration**:
   - After you're done with your configuration, the `.config` file will be saved, and you can extract it for future builds.

4. **Save and use the `.config`**:
   - You can copy the resulting `.config` file to your layer and use it in future builds as described in the `defconfig` method.

#### d) **Editing the `.config` File Manually**:

You can also manually edit the kernel `.config` file after running `bitbake`. However, this approach is generally discouraged because it's less flexible and less maintainable compared to the other methods mentioned.

Steps:
1. **Extract the current kernel configuration**:
   - Run `bitbake virtual/kernel -c configure` to configure the kernel, but stop the build before compiling.
   
2. **Edit the `.config` file** manually in the kernel build directory (`tmp/work`).
   
3. **Rebuild the kernel** using the updated configuration.

### 3. **Adding Kernel Modules and Features**

In addition to configuring the kernel itself, you may also need to add kernel modules or additional features to the kernel image. Here’s how to do it:

#### a) **Add a Kernel Module**:
1. Add the module to the kernel configuration (using one of the methods above):
   ```bash
   CONFIG_MY_MODULE=m
   ```

2. Ensure that the module is included in the final root filesystem by modifying the Yocto recipe to copy it into the appropriate location:
   ```bash
   do_install_append() {
       install -m 0644 ${B}/drivers/my_module/my_module.ko ${D}/lib/modules/${KERNEL_VERSION}/kernel/drivers/my_module/
   }
   ```

3. Rebuild the kernel and ensure that the module is loaded at runtime.

#### b) **Enable Kernel Features**:
Some kernel features, such as specific filesystem support or networking options, can be enabled through kernel configuration. Use kernel fragments or `menuconfig` to enable features like:
- **Filesystem support** (e.g., ext4, BTRFS, XFS):
  ```bash
  CONFIG_EXT4_FS=y
  ```
- **Network protocols**:
  ```bash
  CONFIG_IPV6=y
  ```

### 4. **Building and Deploying the Kernel**

Once the kernel is configured, you can build and deploy it using Yocto:

1. **Build the kernel**:
   ```bash
   bitbake virtual/kernel
   ```

2. **Deploy the kernel**:
   The compiled kernel image, modules, and other related files will be placed in the `deploy` directory of your Yocto build (typically under `tmp/deploy/images/`). You can then copy these files to your target device.

3. **Deploy on target**:
   - Flash the kernel image (e.g., `zImage` or `uImage`) to the appropriate partition of the target device.
   - Copy the kernel modules to `/lib/modules/` on the root filesystem of the target.

### 5. **Kernel Configuration and Patches**

You can also patch the kernel source code if needed:

1. Create a patch file with the necessary changes.
2. Add the patch to your Yocto layer and reference it in the kernel recipe:
   ```bash
   SRC_URI += "file://my-kernel-patch.patch"
   ```

### 6. Example Yocto Kernel Configuration

Let’s consider an example where you want to enable support for a USB serial driver and configure the kernel through a fragment.

1. **Create the fragment file** (`usb-serial.cfg`):
   ```bash
   CONFIG_USB_SERIAL=y
   CONFIG_USB_SERIAL_PL2303=m
   ```

2. **Place the fragment in your Yocto layer** under `meta-my-layer/recipes-kernel/linux/files/`.

3. **Modify the kernel recipe**:
   ```bash
   FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
   SRC_URI += "file://usb-serial.cfg"
   ```

4. **Build the kernel**:
   ```bash
   bitbake virtual/kernel
   ```

5. **Deploy the kernel and modules** to your target system.

#### Conclusion
Configuring the Linux kernel using Yocto is a powerful and flexible process. You can use different methods such as `defconfig`, kernel fragments, or interactive configuration with `menuconfig` to customize the kernel for your specific use case. This flexibility is critical when developing embedded systems, as it allows for precise control over which features and drivers are included in the kernel, ensuring an optimized and stable system.

**Note:** to start the `menuconfig` in the same terminal:
* add the following line inside `local.conf`:
```bash
OE_TERMINAL = "screen"
```
* install screen:
```bash
sudo apt install screen
```
* use `menuconfig`:
```bash
# you can use the following but check the provider for virtual/kernel
bitbake -c menuconfig virtual/kernel
# or you can use if you build kernel for rpi:
bitbake -c menuconfig linux-raspberrypi
```

**Note:** the ways to configure kernel:
* Using the `defconfig` file.
* Using Kernel Configuration Fragments using `.cfg` file
* Interactive Kernel Configuration (`menuconfig`)
* Editing the `.config` File Manually -> (Not Recommended)

### 7. enabling gpio for sysfs example:
* enter the `menuconfig`:
```bash
bitbake virtual/kernel
```
* enable `Configure standard kernel features` in general setup.
    * in menuconfig: to enable module press `y` to disable module press `n` to compile it as a module press `m`
* select device drivers then go to gpio support then you will see line contains `/sys/class/gpio`, enable this option.
* save and exit the menuconfig then the configurations are saved.
* build your image again:
```bash
bitbake your_image
```