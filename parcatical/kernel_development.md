# Kernel Develpoment
In Linux, kernel modules are pieces of code that can be loaded and unloaded into the kernel at runtime. A kernel module allows you to extend the functionality of the kernel without the need to modify the kernel source code itself. There are two types of kernel modules:
1. **In-tree modules**: These are developed and maintained as part of the official Linux kernel source code.
2. **Out-of-tree modules**: These are developed separately from the official kernel source tree.

### Contents:
* [1. Out-of-Tree Kernel Module](#1-out-of-tree-kernel-module)
* [2. Detailed Example: Writing and Building an Out-of-Tree Kernel Module](#2-detailed-example-writing-and-building-an-out-of-tree-kernel-module)
    * [2.1. Creating the Kernel Module Source Code](#21-creating-the-kernel-module-source-code)
    * [2.2. Creating the Makefile](#22-creating-the-makefile)
    * [2.3. Building the Kernel Module](#23-building-the-kernel-module)
    * [2.4. Loading the Kernel Module (Using `insmod`)](#24-loading-the-kernel-module-using-insmod)
    * [2.5. Removing the Kernel Module (Using `rmmod`)](#25-removing-the-kernel-module-using-rmmod)
* [3. `modprobe`, `rmmod`, and `insmod`](#3-modprobe-rmmod-and-insmod)
    * [3.1. `insmod`](#31-insmod)
    * [3.2. `modprobe`](#32-modprobe)
    * [3.3. `rmmod`](#33-rmmod)
    * [3.4. `modprobe -r`](#34-modprobe--r)
* [4. NOTES](#4-notes)
* [5. Hello World kernel module example](#5-hello-world-kernel-module-example)
* [6. Character Devive Driver](#6-character-devive-driver)
    * [6.1. What is the character device driver/module?](#61-what-is-the-character-device-drivermodule)
    * [6.2. What are character device driver operations?](#62-what-are-character-device-driver-operations)
    * [6.3. How to integrate character device driver with yocto build?](#63-how-to-integrate-character-device-driver-with-yocto-build)
    * [6.4. How to load and test char module?](#64-how-to-load-and-test-char-module)

----
**Section 6.3 will be continued**

----

### 1. Out-of-Tree Kernel Module

An **out-of-tree kernel module** is a module that is developed outside the main kernel source tree. This means the module's source code resides independently from the kernel's source code and is typically built using the installed kernel headers, rather than being included directly in the kernel source tree.

Out-of-tree modules are useful when:
- You want to develop a driver or feature that should be maintained independently from the kernel.
- The module is proprietary or specific to a certain application.

Developing an out-of-tree kernel module usually involves writing the kernel module code, creating a `Makefile` for building the module, compiling it against the kernel headers, and then loading it into the running kernel.

---

### 2. Detailed Example: Writing and Building an Out-of-Tree Kernel Module

Let’s walk through the creation of a simple out-of-tree kernel module.

#### 2.1. **Creating the Kernel Module Source Code**

Create a file called `hello_world_km.c` with the following content:

```c
#include <linux/module.h>    // Needed by all kernel modules
#include <linux/kernel.h>    // Needed for KERN_INFO
#include <linux/init.h>      // Needed for the macros

MODULE_LICENSE("GPL"); // mandatory
MODULE_AUTHOR("Ali Mahmoud");
MODULE_DESCRIPTION("My simple Hello World kernel module");

// Initialization function, called when the module is loaded
static int __init hello_world_init(void) {
    printk(KERN_INFO "Hello, World! Kernel module loaded.\n");
    return 0; // 0 means successful initialization
}

// Cleanup function, called when the module is removed
static void __exit hello_world_exit(void) {
    printk(KERN_INFO "Goodbye, World! Kernel module removed.\n");
}

// Registering the init and exit functions
module_init(hello_world_init);
module_exit(hello_world_exit);
```

#### 2.2. **Creating the Makefile**

Create a `Makefile` in the same directory as `hello_world_km.c` to define how to compile the module.

```makefile
obj-m += hello_world_km.o

all:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

- `obj-m`: This tells the build system which object file to generate.
- `-C /lib/modules/$(shell uname -r)/build`: This tells `make` to use the kernel build system located at `/lib/modules/$(uname -r)/build` for the currently running kernel.
- `M=$(PWD)`: This indicates that the module to be built resides in the current directory.

#### 2.3. **Building the Kernel Module**

To compile the out-of-tree module, run:

```bash
make
```

This will generate a file called `hello_world_km.ko` (the `.ko` stands for **kernel object**), which is your compiled kernel module.

#### 2.4. **Loading the Kernel Module (Using `insmod`)**

Once you have the compiled `.ko` file, you can load the module into the running kernel:

```bash
sudo insmod hello_world_km.ko
```

This will insert the module into the kernel. If the module loads successfully, you should see a message in the kernel log:

```bash
dmesg | tail
```

You should see something like:

```
[ ... ] Hello, World! Kernel module loaded.
```

#### 2.5. **Removing the Kernel Module (Using `rmmod`)**

To remove the module from the kernel:

```bash
sudo rmmod hello_world_km
```

Check the kernel log again:

```bash
dmesg | tail
```

You should see:

```
[ ... ] Goodbye, World! Kernel module removed.
```

---

### 3. `modprobe`, `rmmod`, and `insmod`

#### 3.1. **`insmod`**

- **Command**: `insmod` is used to manually insert a kernel module into the kernel.
- **Usage**: You provide the `.ko` file as an argument. For example:
  
  ```bash
  sudo insmod hello_world_km.ko
  ```

- **Limitations**: `insmod` does not resolve dependencies between kernel modules. If your module depends on other kernel modules, you’ll need to load those manually beforehand.

#### 3.2. **`modprobe`**

- **Command**: `modprobe` is a more advanced command for loading modules compared to `insmod`. It automatically resolves and loads any module dependencies.
  
- **Usage**: Instead of specifying the `.ko` file, you specify the module name. For example:

  ```bash
  sudo modprobe hello_world_km
  ```

  This will load the `hello_world_km` module if it’s available in one of the standard module directories (such as `/lib/modules/$(uname -r)`).

- **Advantages**: `modprobe` automatically handles dependencies between modules, making it easier to load complex modules.

#### 3.3. **`rmmod`**

- **Command**: `rmmod` is used to manually remove a module from the kernel. It unloads the module and frees the memory/resources associated with it.
  
- **Usage**: You provide the module name (not the `.ko` file). For example:

  ```bash
  sudo rmmod hello_world
  ```

  This will remove the `hello_world_km` module from the kernel.

#### 3.4. **`modprobe -r`**

- **Command**: `modprobe -r` is used to remove a module, but it also handles dependencies. It automatically removes any dependent modules that were loaded by `modprobe` when the main module was loaded.

- **Usage**: 

  ```bash
  sudo modprobe -r hello_world_km
  ```

  This will remove the `hello_world_km` module and any dependent modules if necessary.

---

### 4. NOTES:

- **Out-of-Tree Kernel Module**: A kernel module developed outside the kernel source tree. It’s useful for independently developing kernel features or drivers.
- **Building the Module**: Use a `Makefile` to build the module against the current kernel headers.
- **Loading/Unloading the Module**: Use `insmod` for simple loading or `modprobe` for more complex loading with dependency resolution. Use `rmmod` or `modprobe -r` to remove modules from the kernel.

### 5. Hello World kernel module example
You can find an example of how to write a kernel module recipe in `meta-skeleton/recipes-kernel/hello-mod`, in this path you will find a simple hello world kernel module yocto recipe.

You can add a recipe like that in your custom meta-layer.

The recipe you will find is like the following:
```bash
SUMMARY = "Example of how to build an external Linux kernel module"
DESCRIPTION = "${SUMMARY}"
LICENSE = "GPL-2.0-only"
LIC_FILES_CHKSUM = "file://COPYING;md5=12f884d2ae1ff87c09e5b7ccc2c4ca7e"

inherit module

SRC_URI = "file://Makefile \
           file://hello.c \
           file://COPYING \
          "

S = "${WORKDIR}"

# The inherit of module.bbclass will automatically name module packages with
# "kernel-module-" prefix as required by the oe-core build environment.

RPROVIDES:${PN} += "kernel-module-hello"
```
As shown in this simple recipe which inherits module.bbclass. The inherit of module.bbclass will automatically name module packages with `kernel-module-` prefix as required by the `oe-core` build environment.

The `.c` file name is `hello.c` so you will find inside the `Makefile` this `obj-m := hello.o`, so for example if you created a file called `my_ldd.c` you will do the following:
* in `Makefile`:
```bash
obj-m := my_ldd.o
```
* in the kernel module recipe:
```bash
RPROVIDES:${PN} += "kernel-module-my_ldd"
```

**NOTE:** It is good if you took a look into a file `meta/classes/module.bbclass`

### 6. Character Devive Driver

A **character device driver** is a type of Linux device driver that allows user-space applications to interact with hardware devices or virtual devices through simple read, write, and other I/O operations. Character devices are accessed through device files located in the `/dev` directory, and they typically support sequential data transfer operations. These drivers are commonly used for peripherals like serial ports, input devices, and even virtual devices for testing purposes.

#### 6.1. What is the character device driver/module?

A **character device driver** is a module that provides access to devices that can be read or written as a stream of data, one character at a time. These drivers are different from block device drivers, which work with fixed-size blocks of data. Character devices are suitable for hardware where the data flow is linear, such as serial ports, keyboard input, or mice.

The driver communicates with the device through kernel-space functions and exposes an interface for user-space programs. Character devices are represented by entries in `/dev`, such as `/dev/ttyS0` for a serial port.

**Example**: A simple character device driver could handle keyboard input, reading one keystroke at a time and passing it to the user-space application.

A character device driver is a driver that sends and recieves the characters. Most of I/O devices are character device driver.

#### 6.2. What are character device driver operations?
It is good to visit the following:
* [Link1](https://olegkutkov.me/2018/03/14/simple-linux-character-device-driver/)
* [Link2](https://embetronicx.com/tutorials/linux/device-drivers/device-file-creation-for-character-drivers/)
* [The Linux Kernel Module Programming Guide](https://tldp.org/LDP/lkmpg/2.6/html/x569.html)
* [Character device drivers - The Linux Kernel documentation](https://linux-kernel-labs.github.io/refs/heads/master/labs/device_drivers.html)

Character device drivers interact with the Linux kernel and the user-space applications through a set of standard operations. These operations define how data is read, written, and managed. They are implemented as function pointers in a `file_operations` structure, which is registered with the kernel during module initialization.

The following contains the character device driver operations and prototype of each operation.
```c
struct file_operations {
       struct module *owner;
       loff_t (*llseek) (struct file *, loff_t, int);
       ssize_t (*read) (struct file *, char *, size_t, loff_t *);
       ssize_t (*write) (struct file *, const char *, size_t, loff_t *);
       int (*readdir) (struct file *, void *, filldir_t);
       unsigned int (*poll) (struct file *, struct poll_table_struct *);
       int (*ioctl) (struct inode *, struct file *, unsigned int, unsigned long);
       int (*mmap) (struct file *, struct vm_area_struct *);
       int (*open) (struct inode *, struct file *);
       int (*flush) (struct file *);
       int (*release) (struct inode *, struct file *);
       int (*fsync) (struct file *, struct dentry *, int datasync);
       int (*fasync) (int, struct file *, int);
       int (*lock) (struct file *, int, struct file_lock *);
         ssize_t (*readv) (struct file *, const struct iovec *, unsigned long,
          loff_t *);
         ssize_t (*writev) (struct file *, const struct iovec *, unsigned long,
          loff_t *);
    };
```
you can make your module like using recipe like `meta-skeleton/recipes-kernel/hello-mod` then insert the module using `insmod` or you can integrate it directly with the kernel and this will be explained in the following section.

#### 6.3. How to integrate character device driver with yocto build?

Here integration of the device driver in mainline kernel. This section explains how to integrate a particular device driver inside the kernel which means inside your image (`uImage` or `zImage`) and we can see the device driver inside `menuconfig`.

In this section, we will outline how to integrate a character device driver into the Linux kernel so that it is part of your system image (such as `uImage` or `zImage`) and configurable via `menuconfig`. This process involves adding your custom driver to the kernel source and ensuring that it is included in the Yocto build process.

Steps:
1. get kernel source.
    * Get the Linux kernel source used by your Yocto project. You can either extract it from your Yocto build environment or download it directly from the kernel repository that matches your Yocto configuration.

2. initialize git.
    * Navigate to the kernel source directory and initialize a Git repository to track your changes. This will make it easier to manage updates and changes.
    ```bash
    cd /path/to/kernel/source
    git init
    ```

3. Find driver Folder inside kernel sources.
    * Kernel drivers are organized by functionality within the drivers/ directory of the kernel source tree. For example:
        * GPIO drivers are typically found in drivers/gpio/
        * SPI drivers in drivers/spi/
        * I2C drivers in drivers/i2c/
    
    Identify the relevant folder based on the type of device your character driver supports.

4. Find the relevant Driver Folder. => relevant folder means: for example driver related to GPIO, driver related to SPI, or driver related to I2C, so we have some there are a different folder for them. or we can add our device folder inside the `driver` folder
    * Inside the relevant `drivers/` subdirectory, create a new folder for your custom character device driver.
    ```bash
    mkdir /path/to/kernel/source/drivers/my_driver
    ```

5. Create our driver folder. (4 & 5 are the same)
6. Create a `Kconfig` inside the folder we created (e.g. `my_driver`)
    * `Kconfig`: is a configuration file that we get for example when we invoke the `menuconfig` so the menu that we see it is due to the `Kconfig` file. So this file will help us to see our driver settings on the `menuconfig`.

7. Create a `Makefile` file to compile our driver.
8. Create or Copy our driver files inside our folder (e.g. `my_driver`).
9. Do `Kconfig` entry in the parent `Kconfig`. (there is a `Kconfig` hierarchy)
9. Do `Makefile` entry in the parent `Makefile`. (there is a `Makefile` hierarchy)
10. Create a patch using Git.
11. Apply the patch using `bbappend`.

See the following image:
<p align="center">
    <img src="Screenshot from 2024-09-29 21-36-34.png" alt="Driver Hierarchy" />
</p>
<p align="center">
    This photo adapted from <a href="https://www.youtube.com/watch?v=0BaNjmVsXNY&list=PLwqS94HTEwpQmgL1UsSwNk_2tQdzq3eVJ&index=48">this YouTube video</a> at 4:20.
</p>

##### ---------------
##### TO BE CONTINUED
##### ---------------


#### 6.4. How to load and test char module?
Once the Yocto build is complete, and the image with the character device driver is running on your target hardware (e.g., Raspberry Pi or other embedded system), you can manually load and test the character driver.

1. **Loading the Module**:
   To load the character device module into the kernel, use the `modprobe` or `insmod` command:

   ```bash
   sudo insmod /lib/modules/$(uname -r)/extra/mychar.ko
   ```

   Alternatively, if the driver has dependencies that need to be automatically resolved:

   ```bash
   sudo modprobe mychar
   ```

2. **Check if the Device is Created**:
   After loading, a device file should be created in `/dev` (this may require additional configuration in the driver, such as calling `register_chrdev`).

   Check the device file with:

   ```bash
   ls /dev/mychar
   ```

   If the device isn't automatically created, you can manually create it:

   ```bash
   sudo mknod /dev/mychar c <major_number> 0
   ```

   Replace `<major_number>` with the major device number returned during module initialization.

3. **Testing the Character Driver**:

   - **Write to the Device**:
   
     ```bash
     echo "Hello, Device" > /dev/mychar
     ```

   - **Read from the Device**:
   
     ```bash
     cat /dev/mychar
     ```

4. **Unload the Module**:
   Once testing is complete, you can unload the module with:

   ```bash
   sudo rmmod mychar
   ```

   Check `dmesg` for any log messages related to loading/unloading the module.

----
- **Character Device Driver**: Provides sequential data I/O and is accessed through device files in `/dev`.
- **Operations**: Standard operations include `open`, `read`, `write`, and `ioctl`, which allow the user-space to interact with the device.
- **Yocto Integration**: Custom kernel modules can be integrated into Yocto builds using a `.bb` recipe.
- **Testing**: Once compiled, modules are loaded with `modprobe` or `insmod`, and can be tested by interacting with the corresponding `/dev` device file.
