# Kernel configuration
## 1. Kernel configuration `defconfig`
we need to save our configuration to the kernel to not do the same configuration every time I want to build the kernel again.

### the flow of configuring the kernel will be like the following:
#### main steps:
1. Create a kernel `.bbappend` file.
2. Configure kernel via `menuconfig`.
3. Save kernel configuration.
4. Copy config file to recipe folder.
5. build your image

**Note: You can use `devtool` to make a `.bbappend` recipe for your kernel.**

### Steps to configure the kernel:
1. Open the `menuconfig`:
```bash
bitbake -c menuconfig virtual/kernel
```
2. make your configurations
3. save your changes then exit the `menuconfig`
4. save the defconfig file:
```bash
bitbake -c savedefconfig virtual/kernel
```
5. you can find your `defconfig` file inside the working directory of the kernel recipe
6. copy the `defconfig` file into the `bbappend` recipe like:
```bash
recipes-kernel/
├── linux/
│   ├── linux-rasbperrypi_%.bbappend
│   └── files/
│       └── defconfig
```
the `linux-rasbperrypi_%.bbappend`:
```bash
FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
SRC_URI += "file://defconfig"
KCONFIG_MODE = "alldefconfig"
```
**Note:** try rename defconfig to e.g. `my_defconfig` then make the `KCONFIG_MODE` value to be `my_defconfig`
7. build your image again:
```bash
bitbake -c cleansstate virtual/kernel
bitbake your_image
```
## 2. Kernel configuration `.cfg`(config fragment)
* config fragment has higher priority than defconfig. => generally highest priorit
* config fragment is more moduler. you can make more than one `.cfg` and each one will be responsible for enabling specific module and if you wanted disable a module again you will just remove its config fragment from `SRC_URI` values.

#### main steps:
1. Create a kernel `.bbappend` file.
2. Configure kernel via `menuconfig`.
3. Create kernel config fragment.
4. Copy config fragment file to recipe folder.
5. Edit kernel `.bbappend` file.
6. Build your image

#### steps to configure the kernel:
1. Open the `menuconfig`:
```bash
bitbake -c menuconfig virtual/kernel
```
2. make your configurations
3. save your changes then exit the `menuconfig`
4. save the config fragment file:
```bash
bitbake -c diffconfig virtual/kernel
```
5. you can find your `.cfg` file inside the working directory of the kernel recipe
6. copy the `.cfg` file into the `bbappend` recipe like then rename it (e.g. I renamed the `fragment.cfg` to `my-fragment.cfg`):
```bash
recipes-kernel/
├── linux/
│   ├── linux-rasbperrypi_%.bbappend
│   └── files/
│       └── my-fragment.cfg
```
the `linux-rasbperrypi_%.bbappend`:
```bash
FILESEXTRAPATHS_prepend := "${THISDIR}/files:"
SRC_URI += "file://my-fragment.cfg"
```