The script `oe-init-build-env` creates and initializes build environments. It is used in two ways:
* to create an empty build environment with default settings
* to initialize a build environment that has previously been created.
```
oe-init-build-env <buildenv>
```

with `<buildenv>` being substituted for the name of the build environment. If no build environment name is provided, then the script uses the default name `build`. The script creates a subdirectory in the current directory using the provided build environment name. Inside that directory, it creates a subdirectory named `conf` in which it creates the two configuration files `bblayers.conf` and `local.conf` **that are required for every build environment**. After that, **the script (`oe-init-build-env`) initializes all necessary shell environment variables and changes directory to the build environment**.

* If the **build environment directory already exists** and is an OpenEmbedded build environment, then `oe-init-build-env` **only initializes the shell environment variables and changes directory**.


* The second script, `oe-init-build-env-memres`, also creates and initializes build environments like `oe-init-build-env` but **also launches a memory-resident BitBake server**, which is listening on a TCP port for commands. **This easily allows for running BitBake on remote build servers and controlling it from a local system over the network**. The script’s command line is:
```
oe-init-build-env <buildenv> <port>
```

* there is the subdirectory `scripts`, which contains a collection of integration and support scripts for working with Yocto Project builds. The most commonly used scripts are as follows:
    * **bitbake-whatchanged:** Lists all components that need to be rebuilt as a consequence of changes made to metadata between two builds.
    * **cleanup-workdir:** Removes build directories of obsolete packages from a build environment
    * **create-recipe:** Creates a recipe that works with BitBake
    * **hob:** Launches Hob, the graphical user interface for BitBake
    * **runqemu:** Launches the QEMU emulator
    * **yocto-bsp:** Creates a Yocto Project BSP layer
    * **yocto-kernel:** Configures Yocto Project kernel recipes inside a Yocto Project BSP layer
    * **yocto-layer:** Creates a metadata layer that works with BitBake

## Build Environment Structure
The OpenEmbedded build system carries out all its work inside a build environment. The build environment, too, has a specific layout and structure. The layout with all directories and files in it is created automatically by the build system. The build environment structure of directories and files is deeply nested. the following list shows the first two levels of the structure after a build has been run:
```
yocto@yocto-dev:~/yocto$ tree -L 2 build
build
├── bitbake-cookerdaemon.log
├── conf
│   ├── bblayers.conf
│   ├── local.conf
│   └── templateconf.cfg
├── cache
│   ├── bb_codeparser.dat
│   ├── bb_persist_data.sqlite3
│   ├── bb_unihashes.dat
│   ├── hashserv.db
│   ├── local_file_checksum_cache.dat
│   └── sanity_info
├── sstate-cache
├── tmp
│   ├── abi_version
│   ├── buildstats
│   ├── cache
│   ├── deploy
│   │   ├── images
│   │   ├── licenses
│   │   ├── deb
│   │   ├── ipk
│   │   └── rpm
│   ├── log
│   │   └── qa.log
│   ├── saved_tmpdir
│   ├── sstate-control
│   ├── stamps
│   ├── sysroots
│   └── work
│       ├── all-poky-linux
│       ├── i586-poky-linux
│       ├── qemux86-poky-linux
│       └── x86_64-linux
└── work-shared
```

* A newly created build environment contains only the subdirectory `conf` with the two files `bblayers.conf` and `local.conf`:
    * `local.conf`: This file contains all the configuration settings for the build environment. You can also add variable settings to it that locally override settings from included layers.
    * `bblayers.conf`: this file contains the layer setup for the build environment. example:
    ```
    # LCONF_VERSION: version number for bblayers.conf
    # It is increased each time build/conf/bblayers.conf
    # changes incompatibly
    LCONF_VERSION = “6”
    BBPATH = “${TOPDIR}”
    BBFILES ?= ””
    BBLAYERS ?= ” \
    /absolute/path/to/poky/meta \
    /absolute/path/to/poky/meta-yocto \
    /absolute/path/to//poky/meta-yocto-bsp \
    ”
    BBLAYERS_NON_REMOVABLE ?= ” \
    /absolute/path/to/poky/meta \
    /absolute/path/to/poky/meta-yocto \
    ”
    ```
    **Note:** The most important variable in this file is `BBLAYERS`, which is a space-delimited list of paths to all the layers included by this build environment. This is the place where you would add additional layers to be included with your build environment. The file also sets `BBPATH` to the top-level directory of the build environment and initializes the recipe file list `BBFILES` with an empty string.

### Explanation of the `tmp` Directory in Yocto Project

The `tmp` directory in the Yocto Project build environment is where all the intermediate and final build outputs are stored. Here's a breakdown of the various subdirectories and files within the `tmp` directory and their purposes:

#### Subdirectories

1. **buildstats**:
   - **Purpose**: Stores build statistics.
   - **Contents**: Organized by build target and timestamp when the target was built. These statistics help in analyzing build performance and debugging issues related to build times.

2. **cache**:
   - **Purpose**: Caches metadata parsing results.
   - **Contents**: When BitBake parses metadata for the first time, it resolves dependencies and expressions, storing the results here. On subsequent runs, if the metadata hasn't changed, BitBake retrieves the information from this cache, speeding up the build process.

3. **deploy**:
   - **Purpose**: Contains build outputs for deployment.
   - **Contents**: Includes target filesystem images, package feeds, and licensing information. This is the directory where final images and packages ready for deployment are stored.

4. **log**:
   - **Purpose**: Stores BitBake logs.
   - **Contents**: Contains logging information created by the BitBake cooker process. These logs are useful for troubleshooting build issues.

5. **sstate-control**:
   - **Purpose**: Manages the shared state cache.
   - **Contents**: Contains manifest files for the shared state cache, organized by architecture/target and task. The shared state cache helps in reusing previous build results to avoid rebuilding unchanged components.

6. **stamps**:
   - **Purpose**: Tracks task completion.
   - **Contents**: Contains completion tags and signature data for every task, organized by architecture/target and package name. These stamps indicate which tasks have been completed successfully.

7. **sysroots**:
   - **Purpose**: Contains root filesystems.
   - **Contents**: Organized by architecture/target. Includes root filesystems for the build host, cross-tool-chain, QEMU, and various tools used during the build process. Sysroots provide a consistent environment for building software.

8. **work**:
   - **Purpose**: Builds software packages.
   - **Contents**: Contains subdirectories organized by architecture/target where the actual software packages are built. This is the working area for package compilation and assembly.

9. **work-shared**:
   - **Purpose**: Manages shared software packages.
   - **Contents**: Similar to the `work` directory but used for shared software packages that are common across multiple targets or architectures. This reduces redundancy and saves build time.

#### Files

1. **abi_version**:
   - **Purpose**: Tracks the ABI (Application Binary Interface) version.
   - **Contents**: This file records the ABI version of the build system. It's used to ensure compatibility and detect changes in the build environment that might affect the binary interface of the generated software.

2. **saved_tmpdir**:
   - **Purpose**: Stores the location of the previous `tmp` directory.
   - **Contents**: Contains a reference to the location of the previous `tmp` directory. This can be useful for debugging and for incremental builds where the build system might need to reference previous build states.

### Example Scenario

Imagine you are building a custom Linux image for an embedded device using the Yocto Project. Here’s how the `tmp` directory plays a role in the process:

1. **Initial Build**:
   - **cache**: BitBake parses all metadata, resolves dependencies, and stores the results in the cache.
   - **work**: Software packages are built in architecture/target-specific subdirectories.
   - **sysroots**: The root filesystem for the build host, including cross-compilation tools, is created here.
   - **log**: Logs of the build process are stored for troubleshooting.

2. **Intermediate Stages**:
   - **stamps**: As each build task completes, a stamp is created indicating successful completion.
   - **sstate-control**: Shared state cache files are generated to allow reuse in future builds.

3. **Final Output**:
   - **deploy**: The final root filesystem images and package feeds are stored here, ready for deployment.
   - **buildstats**: Build statistics are recorded for performance analysis.

4. **Rebuilding**:
   - **cache**: If metadata hasn’t changed, BitBake uses cached information, speeding up the process.
   - **work-shared**: Shared packages are reused, avoiding redundant builds.

By organizing the build output into these subdirectories, the Yocto Project ensures a structured and efficient build process, facilitating easier debugging, faster rebuilds, and better management of build artifacts. 
