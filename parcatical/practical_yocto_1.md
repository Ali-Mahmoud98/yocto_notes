# Common YOCTO Commands and Variables
This file contains the basic and important information about variables and commands which are mostly used.

## Contents
* [1. The basic variables](#1-the-basic-variables)
    * [1.1 Show recipe variable value](#11-the-following-command-to-know-the-recipe-variables-value)
    * [1.2 append value to variable in yocto](#12-we-can-append-and-prepend-values-to-yocto-variables-using)
* [2. Variables for do_fetch](#2-variables-for-do_fetch)
    * [2.1 SRC_URI](#21-src_uri)
    * [2.2 SRCREV](#22-srcrev)
* [3. Basic Commands](#3-baic-commands)
* [4. Patching recipe using devtools](#4-patching-recipe-using-devtools)
* [5. Patching without devtools](#5-patching-sources-of-recipe-without-devtools)
* [6. RDEPENDS, DEPENDS, and RPROVIDES Variables](#6-importent-variables-rdepends-depends-rprovides)
    * [6.1 RDEPENDS](#61-rdepends-runtime-depends)
    * [6.2 DEPENDS](#62-depends-build-time-depends)
    * [6.3 RPROVIDES](#63-rprovides-runtime-provides)
    * [6.4 Differences in simple](#64-differences-in-simple)
    * [6.5 Example Scenario](#65-example-scenario)
* [7 PACKAGES and FILES variables](#7-packages-and-files)
    * [7.1 PACKAGES](#71-packages-in-yocto)
    * [7.2 FILES:${PN}-example](#72-filespn-example-in-yocto)

### 1. The basic variables:
----
* `PN`:
This variable represents the "Package Name" of the recipe. It is typically derived from the recipe filename and is used to identify the software package being built.
* `PV`:
This stands for "Package Version." It indicates the version of the software package that is being built, allowing for version control and management.
* `PR`:
The "Package Revision" variable is used to specify the revision of the package. It is often incremented for changes that do not affect the version number, such as bug fixes or minor updates.
* `WORKDIR`:
This variable points to the working directory for the recipe. It is where the source code is unpacked and where the build process takes place.
* `S`:
The "Source Directory" variable indicates the location of the source code for the recipe. It is typically set to a subdirectory within
* `D`:
This variable represents the "Destination Directory." It is the directory where the final package files are installed during the build process, typically used for packaging the output.
* `B`:
The `B` variable stands for "Build Directory." It specifies the directory where the build process takes place. This is typically a subdirectory of the `WORKDIR` and is used to store intermediate files generated during the build, such as object files and other artifacts. The separation of the build directory from the source directory helps maintain a clean environment and allows for easier management of build outputs.

#### 1.1 The following command to know the recipe variables value 
```sh
bitbake -e recipe_name | grep ^YOCTO_VAR=
```

#### 1.2 we can append and prepend values to yocto variables using:
* new syntax:
```sh
VAR1:append = " val" # space should be added at first
VAR1:prepend = " val"
```
* old syntax:
```sh
VAR1_append = " val" # space should be added at first
VAR1_prepend = " val"
```
* another syntax:
```sh
VAR1 += "val" # append: no need to add space at first
VAR1 =+ "val" # prepend

VAR1 .= " val" # append
VAR1 =. " val" # prepend
```
* also there is remove:
```sh
VAR1:remove = "val" # new syntax
VAR1_remove = "val" # old syntax
```
### 2. Variables for do_fetch
----
In the context of building software with the Yocto Project or OpenEmbedded, the `do_fetch` task is responsible for downloading source code or other resources specified in the recipe. The variables below are used to configure what `do_fetch` retrieves and how it works:

#### 2.1 `SRC_URI`:
- **Purpose**: Specifies the location(s) where the source code or other resources should be fetched from. This is a key variable for the `do_fetch` task because it tells the system where to look for the files.
- **Format**: It can contain multiple URIs (Uniform Resource Identifiers) separated by spaces. These URIs can reference different protocols such as `http`, `https`, `git`, `ftp`, `svn`, and even local file paths.
  
  Example:
  ```bash
  SRC_URI = "https://example.com/source.tar.gz"
  ```
  
  Multiple sources example:
  ```bash
  SRC_URI = "https://example.com/source.tar.gz \
             git://github.com/example/repo.git;branch=main"
  ```

- **Common Usage**: 
  - Downloading source tarballs.
  - Cloning Git repositories.
  - Retrieving patches, documentation, or other resources.

#### 2.2 `SRCREV`:
- **Purpose**: Defines the specific revision of the source code that should be fetched from a version control system like Git, Subversion (SVN), or Mercurial. This is especially important for projects where you want to pin the recipe to a specific commit, tag, or branch in the repository.
  
  Example (Git):
  ```bash
  SRCREV = "abcdef1234567890"
  ```

  Example (Git with a branch specified in `SRC_URI`):
  ```bash
  SRC_URI = "git://github.com/example/repo.git;branch=main"
  SRCREV = "abcdef1234567890"
  ```

- **Common Usage**:
  - For Git, `SRCREV` is typically set to a commit hash, tag, or branch.
  - For SVN or other version-controlled systems, it could be a revision number.

#### 2.3 Other Related Variables for `do_fetch`

- **`FETCHCMD`**: Defines the command used to fetch files. Usually, this is set based on the protocol specified in `SRC_URI`. For example, for `git://`, it would use Git; for `http://`, it would use a tool like `wget`.

- **`SRC_URI_OVERRIDES_PACKAGE_ARCH`**: This variable ensures that the content fetched from `SRC_URI` takes priority over architecture-specific adjustments in some cases.

- **`LOCAL_MIRROR`**: This can be used to specify local or mirror locations for fetching sources if remote URIs are not reachable or if you want to cache source code.

- **`BB_NO_NETWORK`**: When set, it forces the build system to only use sources that are already available locally and avoids using the network, often used in offline or cached builds.

Would you like further clarification or specific examples on any of these variables?

#### 2.4 YOCTO tasks:
you will see the following tasks many times
* `do_fetch` -> `do_unpack` -> `do_patch` -> `do_configure` -> `do_compile` -> `do_install`

1. the `do_fetch` starts first and download or fetch sources then add them into the download directory.
2. the `do_unpack` add the files into the `WORKDIR`
    * the patches will be inside the `WORKDIR` and other folders.
    * the source files inside the code cloned from `e.g.: git` will be in `S` the source directory
3. the patchs is used to edit the sources before building in the `do_patch` task.
4. `do_configure` used to add configurations before build and compile.
5. example about `do_compile`:
    ```sh
    # very simple do compile task
    do_compile() {
        ${CC} ${CFLAGS} ${LDFLAGS} ${S}/code.c -o out_exec 
    }
    ```
    * `${CFLAGS}`: compiler flags
    * `${LDFLAGS}`: linker flags

There are also tasks, to list then use `bitbake -c listtasks recipe_name` 

### 3. Baic Commands:
----
```sh
bitbake -e recipe_name | grep ^YOCTO_VAR= # Print recipe variables value

bitbake -c listtasks recipe_name # List recipe tasks

bitbake-layers show-layers # shows the layers

bitbake-layers create-layer /path/to/create/layer/layer_name

bitbake-layers add-layer /path/to/create/layer/layer_name
```


### 4. Patching recipe using devtools:
----
```sh
devtool modify recipe-name

bitbake -c devshell recipe-name # optionally -> you can edit recipe in the workspace without devshell

# make your configuration to recipe files
# edit files

git add file # add the first file you changed

git commit -m "your message" --signoff

git add file2 # add the second file you changed

git commit -m "your message" --signoff

# each file change should have its commit
# you can edit one or more file at the same time but each file should have a commit different from another 

exit # if you use devshell, you should exit devshell 

devtool update-recipe recipe-name -a /path/to/your/custom_layer

devtool reset <recipe-name>

# then delete the recipe files from the workspace
```


### 5. Patching sources of recipe without devtools:
----
```sh
bitbake -c do_fetch recipe_name
bitbake -c do_unpack recipe_name

cd /to/your/source/dir/ # you can go to the source dire using devshell
# you can skip previous steps using: bitbake -c devshell recipe_name

git init
git add .
git commit -m "Intialization"
# add changes to the files
git add file1
git commit -m "your message1"
git format-patch HEAD~1

git add file2
git commit -m "your message2"
git format-patch HEAD~1

# copy patches to use them with the recipe
bitbake cleanall recipe_name
```



### 6. Importent Variables RDEPENDS, DEPENDS, RPROVIDES
----
In Yocto/OpenEmbedded, the build system needs to understand the relationships between different software components. This is where variables like `RDEPENDS`, `DEPENDS`, and `RPROVIDES` come into play. Each of these variables controls how recipes (packages) are related in terms of build-time and runtime dependencies.


#### 6.1 `RDEPENDS` (Runtime Depends)
- **Purpose**: Specifies **runtime dependencies**. It lists the packages that must be installed **at runtime** for a particular package to function properly.
  
- **Usage**: When the software produced by a recipe is installed on the target device, the packages specified in `RDEPENDS` will also be installed automatically to ensure all necessary functionality is available.

- **Example**: 
  If your recipe is building a Python script that depends on the `python3` interpreter, you would use:
  ```bash
  RDEPENDS_${PN} = "python3"
  ```

  Here:
  - `${PN}` is a special variable representing the name of the package being built.
  - This ensures that `python3` is installed on the target system alongside the package built by the recipe.

- **Scenario**: A typical case might be that your application depends on another package, like a library or a tool, to run. These dependencies need to be present in the target system for the software to operate.
- This used if the package needs another package at runtime.


#### 6.2 `DEPENDS` (Build-Time Depends)
- **Purpose**: Specifies **build-time dependencies**. It lists the packages (or recipes) that must be available and built **before** this recipe can be built.
  
- **Usage**: It ensures that all required packages or tools are built and available during the compilation and linking stages. These dependencies are **only necessary during the build process**, not at runtime on the target device.

- **Example**:
  If you are building a C application that relies on OpenSSL for encryption, you would specify:
  ```bash
  DEPENDS = "openssl"
  ```

  This ensures that the OpenSSL libraries are available for your application to link against when it’s being compiled.

- **Scenario**: `DEPENDS` specifies which libraries, development headers, and tools are needed to build your software. Once built, these dependencies are no longer required.


#### 6.3 `RPROVIDES` (Runtime Provides)
- **Purpose**: Defines **runtime provides**. This variable allows a package to declare that it **provides the functionality** of another package. It’s often used for virtual packages or when multiple packages offer the same functionality.

- **Usage**: If multiple packages provide similar or identical functionality, `RPROVIDES` allows one package to effectively replace another at runtime by providing the same service or feature.

- **Example**:
  Suppose your package `my-custom-shell` is a custom implementation of `bash`. You would specify:
  ```bash
  RPROVIDES_${PN} = "bash"
  ```

  This means that `my-custom-shell` can be installed in place of `bash` because it provides the same functionality.
  
  The recipe can provide an alias for its name using `RPROVIDES_${PN}`.

- **Scenario**: It is useful in cases where you have alternative implementations of a tool or service. For example, both `busybox` and `coreutils` provide basic shell commands (like `ls`, `cp`, etc.), so `busybox` might declare that it `RPROVIDES` some of the functionality provided by `coreutils`.


#### 6.4 Differences in simple:
- **`RDEPENDS`**: Declares **runtime dependencies** (packages that need to be installed alongside the software for it to run correctly).
- **`DEPENDS`**: Declares **build-time dependencies** (packages that must be available during the build process to compile or link the software correctly).
- **`RPROVIDES`**: Declares that a package **provides the same functionality** as another package at runtime, often used in scenarios where alternative implementations exist.


#### 6.5 Example Scenario:
Let’s say you have a recipe to build a custom media player:
- `DEPENDS = "ffmpeg libmpg123"` means you need `ffmpeg` and `libmpg123` at build-time to compile the media player.
- `RDEPENDS_${PN} = "alsa-lib"` means your media player depends on `alsa-lib` to function at runtime for audio output.
- `RPROVIDES_${PN} = "mediaplayer"` means that your custom media player provides the functionality of the generic `mediaplayer` package.



### 7. `PACKAGES` and `FILES`
----
#### 7.1 **`PACKAGES` in Yocto:**
- The `PACKAGES` variable in Yocto defines the set of binary packages that are produced from a single recipe.
- By default, Yocto generates several packages from one recipe, including the main package (`${PN}`), debug packages (`${PN}-dbg`), and documentation packages (`${PN}-doc`), where `${PN}` represents the name of the recipe.
- You can explicitly specify additional packages in the recipe using the `PACKAGES` variable.

**Example:**
```bash
PACKAGES = "${PN} ${PN}-dev ${PN}-doc"
```
In this example:
- `${PN}` is the main package (usually contains the main binaries).
- `${PN}-dev` is the development package (may contain headers and static libraries for development).
- `${PN}-doc` is the documentation package (contains documentation files).


#### 7.2 **`FILES:${PN}-example` in Yocto:**
- The `FILES` variable is used to specify which files should go into the packages defined in `PACKAGES`.
- The syntax `FILES:${PN}-example` tells Yocto which files should be included in the `${PN}-example` package (assuming `${PN}-example` is defined as part of the `PACKAGES` variable).
- Typically, this includes binaries, libraries, configuration files, etc., that belong to this specific package.

**Example:**
```bash
PACKAGES += "${PN}-example"
FILES:${PN}-example = "${bindir}/example ${datadir}/example"
```
In this example:
- A new package `${PN}-example` is created.
- The `FILES:${PN}-example` variable tells Yocto to include the binary `example` located in the `${bindir}` and the data files in `${datadir}/example` into the `${PN}-example` package.

- Difference between `FILES:${PN}-example` and `FILES:${PN}`:
    - `FILES:${PN}-example`: Files assigned here go into a separate subpackage (e.g., `mypackage-example`). This is typically used for optional components like example files or documentation.
    - `FILES:${PN}`: Files assigned here go into the main package (e.g., `mypackage`). These files are considered essential to the core functionality of the package.

This system allows for fine-grained control over what files belong in each package, making it easier to create modular and optimized embedded Linux images.
