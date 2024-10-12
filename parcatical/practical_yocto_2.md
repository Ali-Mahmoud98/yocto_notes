# Compilation
## Contents
* [1. do_compile](#1-do_compile)
* [1.1 Cross-compiling Using Yocto](#11-cross-compiling-using-yocto)
* [1.2 The do_compile Task in Detail](#12-the-do_compile-task-in-detail)
* [1.2.1 What Happens in do_compile](#121-what-happens-in-do_compile)
* [1.3 Workflow Summary](#13-workflow-summary)
* [2. Autotools and Pkgconfig](#2-autotools-and-pkgconfig)
* [2.1 inherit autotools](#21-inherit-autotools)
* [2.1.1 What Does inherit autotools Provide](#211-what-does-inherit-autotools-provide)
* [2.1.2 Example of a Recipe Using inherit autotools](#212-example-of-a-recipe-using-inherit-autotools)
* [2.2 inherit pkgconfig](#22-inherit-pkgconfig)
* [2.2.1 What Does inherit pkgconfig Provide](#221-what-does-inherit-pkgconfig-provide)
* [2.2.2 Example of a Recipe Using inherit pkgconfig](#222-example-of-a-recipe-using-inherit-pkgconfig)
* [2.3 When to Use inherit autotools and pkgconfig](#23-when-to-use-inherit-autotools-and-pkgconfig)
* [3. oe_runmake](#3-oe_runmake)
* [3.1 What is oe_runmake](#31--what-is-oe_runmake)
* [3.2 Why Use oe_runmake](#32-why-use-oe_runmake)
* [3.3 How to Use oe_runmake](#33-how-to-use-oe_runmake)
* [3.3.1 Basic Usage](#331-basic-usage)
* [3.3.2 Passing Arguments to oe_runmake](#332-passing-arguments-to-oe_runmake)
* [3.3.3 Using with Parallel Builds](#333-using-with-parallel-builds)
* [3.3.4 Example: Full Recipe with oe_runmake](#334-example-full-recipe-with-oe_runmake)
* [3.3.5. oe_runmake without inherit autotools example](#335-oe_runmake-without-inherit-autotools-example)
* [4. EXTRA_OEMAKE](#4-extra_oemake)
* [4.1. What is `EXTRA_OEMAKE`?](#41-what-is-extra_oemake)
* [4.2. Why Use `EXTRA_OEMAKE`?](#42-why-use-extra_oemake)
* [4.3. How to Use `EXTRA_OEMAKE`?](#43-how-to-use-extra_oemake)
* [4.3.1. Basic Usage](#431-basic-usage)
* [4.3.2. Example in a Recipe](#432-example-in-a-recipe)
* [4.4. EXTRA_OEMAKE Relation with Makefile](#44-extra_oemake-relation-with-makefile)
* [4.4.1. Example Makefile Snippet](#441-example-makefile-snippet)
* [4.5. How to Pass Libraries and Other Parameters to Build Environment?](#45-how-to-pass-libraries-and-other-parameters-to-build-environment)
* [4.5.1. Example with Libraries](#451-example-with-libraries)
* [4.5.2. Example in a Recipe](#452-example-in-a-recipe)
* [4.6. EXTRA_OEMAKE to customize the parameters passed to the compiler](#46-extra_oemake-to-customize-the-parameters-passed-to-the-compiler)
* [4.7. Example Use Case: Optimization Flags](#47-example-use-case-optimization-flags)

## 1. do_compile
Cross-compiling with Yocto involves building software for a target platform that differs from the host machine. Yocto uses metadata, recipes, and BitBake to define how software components are compiled and configured for the target architecture. Here's an overview of how cross-compilation and the `do_compile` process works in Yocto:

### 1.1. **Cross-compiling using Yocto**
   - **Yocto Layers and Recipes**: Yocto uses layers and recipes to define software components. Recipes specify how to download, configure, compile, and package software for the target.
   - **Machine Configuration**: When cross-compiling, Yocto needs to know details about the target machine (e.g., CPU architecture, available hardware). This is specified in the `MACHINE` configuration variable, usually in a configuration file in the `conf/machine/` directory of a layer.
   - **Toolchain**: Yocto uses a cross-toolchain to build software for the target architecture. The toolchain includes a cross-compiler (like `gcc`), linker, and other tools to compile the code for the target system.
   - **Environment Setup**: Before starting a build, Yocto sets up a specific environment using the `oe-init-build-env` script. This script sets environment variables and paths required for cross-compiling.
   - **BitBake Command**: BitBake is the build tool that reads the recipes and executes tasks like fetching source code, configuring, compiling, and packaging the software. To start cross-compiling a specific recipe, you run:
     ```bash
     bitbake <recipe-name>
     ```

### 1.2. **The `do_compile` Task in Detail**
In Yocto, tasks like fetching (`do_fetch`), patching (`do_patch`), configuring (`do_configure`), and compiling (`do_compile`) are defined in each recipe. The `do_compile` task is where the actual cross-compilation occurs.

#### 1.2.1. **What Happens in `do_compile`:**
The `do_compile` task usually follows `do_configure`, where the build system (like `autotools`, `cmake`, or `meson`) has been prepared. In the `do_compile` task, the source code is compiled into binaries using the cross-compiler for the target architecture.

Here’s a breakdown of the key points for `do_compile`:

1. **Cross-Compiler Setup**:
   - Yocto ensures that the right cross-compiler, linker, and associated tools are used by setting environment variables like `CC`, `CXX`, `LD`, `AR`, `RANLIB`, etc. These point to the cross-compilation toolchain created by Yocto.

2. **Compiling the Code**:
   - Yocto recipes generally use common build systems like GNU `autotools`, `cmake`, or others. The build system command (e.g., `make`) is executed during the `do_compile` task.
   - For an `autotools` recipe, for example, the `do_compile` task might look like this:
     ```bash
     oe_runmake
     ```
     This calls `make` with the appropriate environment settings and arguments to compile the software.

3. **Handling Parallelism**:
   - Yocto supports parallel building. The number of parallel jobs can be controlled using the `PARALLEL_MAKE` variable.
     ```bash
     oe_runmake ${PARALLEL_MAKE}
     ```

4. **Customization of `do_compile`**:
   - If you need to customize the build process, you can override the default `do_compile` task in the recipe. For example, you might want to add custom build steps, or bypass the default build system commands:
     ```bash
     do_compile() {
         # Custom compile command
         ${CC} -o myapp myapp.c
     }
     ```

5. **Error Handling**:
   - If an error occurs during the `do_compile` task, BitBake will capture the error and stop the build process. You can examine the logs (`log.do_compile`) in the `temp` directory of the recipe’s build directory to troubleshoot any issues.

6. **Typical Example**:
   A simple example of an `autotools` based recipe might include the following steps:
   ```bash
   inherit autotools
   inherit pkgconfig

   do_compile() {
       oe_runmake
   }
   ```
   In this example, `oe_runmake` runs `make` in the environment set up for cross-compilation.

### 1.3. **Workflow Summary**
1. **Set Up Yocto Environment**: 
   ```bash
   source oe-init-build-env
   ```
   
2. **Define or Modify a Recipe**: 
   Create or edit a recipe to customize `do_compile`.

3. **Run BitBake**: 
   Start the cross-compilation process:
   ```bash
   bitbake <recipe-name>
   ```

4. **Examine Logs**: 
   If there are issues, logs from `do_compile` are located under the `temp/log.do_compile` in the recipe’s work directory.

**NOTE:** For detailed understanding of the build process, tasks, and variables, check the [Yocto Project Reference Manual](https://www.yoctoproject.org/docs/).

**BitBake User Manual**: Learn more about how BitBake works, including task dependencies and execution.

By properly setting up your environment, configuring your machine settings, and understanding how `do_compile` works, you can successfully cross-compile software using Yocto.

## 2. Autotools and pkgconfig
In Yocto recipes, the `inherit` keyword is used to include predefined functionality and classes into a recipe. These classes provide standard tasks, functions, and behaviors that are common across different recipes, helping to avoid redundancy and simplifying the writing of recipes.

Here’s an explanation of the two most common classes, `autotools` and `pkgconfig`, and how they work:

---

### 2.1. **`inherit autotools`**

The `autotools` class in Yocto provides support for building software that uses the GNU `autotools` build system (which typically includes `autoconf`, `automake`, and `libtool`). Most open-source projects use `autotools` to provide a cross-platform way of configuring and building software.

By including `inherit autotools` in a Yocto recipe, the build system automatically handles key tasks associated with the `autotools` build process, such as configuration, compilation, and installation.

#### 2.1.1. **What does `inherit autotools` provide?**
When you inherit the `autotools` class in a recipe, Yocto provides the following standard tasks:

1. **do_configure**:
   - Runs the `./configure` script to detect system-specific settings (e.g., paths to libraries, compilers, etc.).
   - Configures the software package to be compiled for the target architecture.
   - Sets environment variables (like `CC`, `CXX`, `LD`, etc.) for cross-compilation.

2. **do_compile**:
   - Runs the `make` command to build the software from the source code.
   - Uses `oe_runmake` to ensure the correct environment is applied and to manage parallel builds.

3. **do_install**:
   - Runs `make install` to copy the compiled files (binaries, libraries, headers, etc.) to the target installation directories (usually in `DESTDIR`).

4. **Patches for cross-compilation**:
   - The `autotools` class automatically patches some build scripts (like `configure`) to make them cross-compilation-friendly.

#### 2.1.2. **Example of a Recipe Using `inherit autotools`**
Here’s a simple Yocto recipe that uses `autotools` to build an open-source project:

```bash
SUMMARY = "Example application using autotools"
DESCRIPTION = "A simple application that uses autotools for building."
LICENSE = "MIT"

SRC_URI = "https://example.com/myapp.tar.gz"
SRCREV = "123456"

S = "${WORKDIR}/myapp"

inherit autotools

do_compile() {
    oe_runmake
}
```

In this example, the `inherit autotools` line takes care of running the `./configure` script, compiling the source code with `make`, and installing the binaries using `make install`. You don’t need to manually specify these steps in the recipe unless you need customization.

---

### 2.2. **`inherit pkgconfig`**

The `pkgconfig` class is used to help recipes that rely on `pkg-config` to find and configure dependencies. `pkg-config` is a tool commonly used in Linux environments to manage library paths, compile flags, and linker flags required by a project. It helps software packages detect the presence and versions of dependent libraries at configure time.

By inheriting `pkgconfig`, Yocto ensures that the cross-compilation environment is properly configured so that `pkg-config` will work correctly, even for target libraries that are being cross-compiled.

#### 2.2.1. **What does `inherit pkgconfig` provide?**
The `pkgconfig` class sets up and ensures the correct use of the `PKG_CONFIG` and `PKG_CONFIG_PATH` environment variables, allowing the build system to find target libraries and their respective configuration files (`.pc` files) during cross-compilation.

1. **PKG_CONFIG**:
   - Points to the Yocto-provided cross-compiled `pkg-config` tool for the target architecture.
   
2. **PKG_CONFIG_PATH**:
   - Ensures the correct paths for finding `.pc` (package configuration) files during cross-compilation.
   - This is especially important when Yocto builds multiple libraries and these libraries are required as dependencies for other projects.

Without `inherit pkgconfig`, recipes that depend on `pkg-config` for checking dependencies might fail or not detect the required libraries correctly when cross-compiling.

#### 2.2.2. **Example of a Recipe Using `inherit pkgconfig`**
```bash
SUMMARY = "Example application with pkgconfig dependencies"
DESCRIPTION = "An application that depends on libraries managed by pkg-config."
LICENSE = "MIT"

SRC_URI = "https://example.com/myapp.tar.gz"
SRCREV = "abcdef"

S = "${WORKDIR}/myapp"

inherit autotools pkgconfig
```

In this example:
- `autotools` ensures the application is built using `./configure`, `make`, and `make install`.
- `pkgconfig` ensures that the `pkg-config` tool can correctly find and use any necessary libraries (like `libpng`, `libjpeg`, or other dependencies) during the `./configure` step.

---

### 2.3. **When to Use `inherit autotools` and `pkgconfig`**

1. **`inherit autotools`**: Use this class when the software you’re building uses the GNU `autotools` build system (`./configure`, `make`, and `make install` are standard commands in the build process). Most open-source projects follow this convention.

2. **`inherit pkgconfig`**: Use this class when the software package you are building or configuring uses `pkg-config` to locate dependencies, especially when you are cross-compiling and want to ensure the dependencies are correctly found for the target architecture.

### **Summeriaing `inherit autotools` and `inherit pkgconfig`**
- **`inherit autotools`**: Automatically handles standard tasks (configuration, compilation, installation) for projects using the `autotools` build system.
- **`inherit pkgconfig`**: Helps ensure that `pkg-config` finds and configures dependencies correctly in a cross-compilation environment.

By inheriting these classes, Yocto automates much of the repetitive work of setting up and managing builds, making the development process more efficient and consistent across various projects.


## 3. OE_RUNMAKE
### 3.1.  **What is `oe_runmake`?**

`oe_runmake` is a helper function provided by the OpenEmbedded (OE) build system (used by Yocto) to simplify and standardize the use of the `make` command in recipes. It acts as a wrapper around `make`, ensuring that the correct environment is used during the compilation, especially when cross-compiling for different target architectures.

It helps handle parallel builds, cross-compilation environments, and error handling more efficiently than directly invoking `make`. Yocto uses `oe_runmake` to ensure consistent build behavior across various projects, hardware platforms, and architectures.

### 3.2. **Why Use `oe_runmake`?**

`oe_runmake` is used for several reasons:

1. **Cross-Compilation**: Yocto is primarily used to cross-compile software for target devices that have different architectures from the host machine. `oe_runmake` ensures that the correct cross-compiler and environment variables (e.g., `CC`, `CXX`, `LDFLAGS`) are passed to `make`, so the software is built for the target platform.
   
2. **Parallel Builds**: `oe_runmake` supports parallel compilation by automatically adding the appropriate options (e.g., `-jN`, where `N` is the number of parallel jobs) using the `PARALLEL_MAKE` variable. This speeds up the build process on multi-core machines.

3. **Error Handling**: If `make` fails during the build process, `oe_runmake` ensures that the error is properly captured and reported. This prevents silent failures, helping developers identify issues more easily.

4. **Consistency**: Using `oe_runmake` provides a standard approach to invoking `make` across different recipes, ensuring that the same build behavior and settings (e.g., paths, flags) are applied consistently across Yocto projects.

### 3.3. **How to Use `oe_runmake`?**

Using `oe_runmake` is simple. You typically use it in the `do_compile`, `do_install`, or other custom tasks in a Yocto recipe. Here's how:

#### 3.3.1. **Basic Usage**

In a typical recipe, `oe_runmake` is used to invoke `make`:

```bash
do_compile() {
    oe_runmake
}
```

This automatically runs the `make` command, ensuring that the appropriate environment variables are set for cross-compiling and that the correct number of jobs are executed in parallel (according to the `PARALLEL_MAKE` variable).

#### 3.3.2. **Passing Arguments to `oe_runmake`**

You can pass arguments to `oe_runmake` if you need to invoke specific targets or set custom variables. For example:

```bash
do_compile() {
    oe_runmake clean
    oe_runmake all
}
```

This will first clean the build directory by invoking `make clean`, then build the project using `make all`.

#### 3.3.3. **Using with Parallel Builds**

`oe_runmake` will automatically handle parallel builds based on the `PARALLEL_MAKE` variable. However, if you need to customize this, you can modify `PARALLEL_MAKE` in your recipe:

```bash
PARALLEL_MAKE = "-j8"
```

This forces `make` to run with 8 parallel jobs, which can speed up the build process.

#### 3.3.4. **Example: Full Recipe with `oe_runmake`**

Here’s a simple example of a Yocto recipe that uses `oe_runmake`:

```bash
SUMMARY = "My custom application"
DESCRIPTION = "This is a simple example recipe for a Yocto project."
LICENSE = "MIT"

SRC_URI = "git_hub_repo_link"
# The SRC_URI specifies the path to the git repository that holds the source tree.
SRCREV = "abcdef1234567890"
# The SRCREV variable specifies to bitbake about the SHA1 hash of the commit that should be used.

S = "${WORKDIR}/git"

inherit autotools

do_compile() {
    oe_runmake
}
```
[Writing recipes that fetch from a Git repository](https://kickstartembedded.com/2022/02/14/yocto-part-8-writing-recipes-that-fetch-from-a-git-repository/)

In this recipe:
- The source code is fetched from a Git repository.
- The `do_compile` task uses `oe_runmake` to invoke the build process.
- `inherit autotools` means the recipe will automatically handle configuration using the GNU `autotools` build system, and `oe_runmake` will call `make`.

#### 3.3.5. oe_runmake without inherit autotools example
```bash
SUMMARY = "My custom application"
DESCRIPTION = "This is a simple example recipe for a Yocto project."
LICENSE = "MIT"

SRC_URI = " \
            src1.c \
            src2.c \
            main.c \
            "
SRC_URI += "Makefile" # should be added to use runmake

S = "${WORKDIR}"
# EXTRA_OEMAKE:append = " DESTDIR=${D} BINDIR=${bindir}" # EXTRA_OEMAKE will be discussed

do_compile() {
    # without oe_runmake
    # ${CC} ${CFLAGS} -c src1.c -o src1.o
    # ${CC} ${CFLAGS} -c src2.c -o src2.o
    # ${CC} ${CFLAGS} -c main.c -o main.o
    # ${CC} ${CFLAGS} ${LDFLAGS} src1.o src2.o main.o -o out_prog -Wl,--hash-style=gnu
    #############
    # comment the 4 lines after "without oe_runmake"
    # using oe_runmake => make sure to add your Makefile to SRC_URI
    oe_runmake
}

do_install() {
    # without oe_runmake
    # install -d ${D}${bindir}
    # install -m 0755 out_prog ${D}${bindir}
    # with oe_runmake => should add a target called install in your make file
    oe_runmake install DESTDIR=${D} BINDIR=${bindir} # DESTDIR and BINDIR should be used in your target
    # you can use the following line => make sure to pass the DESTDIR and BINDIR
    # oe_runmake install # to be able to use this use the following => EXTRA_OEMAKE:append = " DESTDIR=${D} BINDIR=${bindir}"
}
```

## 4. EXTRA_OEMAKE
**`EXTRA_OEMAKE` is a variable used within yocto recipes files to customize the build process of a specific SW component or package.**

**`EXTRA_OEMAKE` allows you to pass additional argumnts to the make command during the compilation of the SW.**

**`EXTRA_OEMAKE` we can use it for customization for the giving optmization parameters to the compiler.**

* General usage of `EXTRA_OEMAKE`:
```bash
EXTRA_OEMAKE += "<make_var>=<your_value>"
```
if you have a make file contains the following variables:
```bash
TARGET ?= ''
LIBS ?= ''
```
you can add values to the variables like the following:
```bash
LIBS="lib1.c\ lib2.c\ lib3.c"
EXTRA_OEMAKE:append = " TARGET=mytarget LIBS=${LIBS}"
```
### 4.1. What is `EXTRA_OEMAKE`?

`EXTRA_OEMAKE` is a variable in Yocto used to pass additional options or parameters to the `make` command when running the `do_compile`, `do_install`, and other `make`-based tasks in a recipe. It acts as a way to append or modify the environment variables or flags passed to `make` during the build process, ensuring that the build environment is configured appropriately.

When Yocto invokes the `make` command via `oe_runmake` or directly in the `do_compile` function, the contents of `EXTRA_OEMAKE` are automatically appended to the `make` command. This allows you to fine-tune the build process without directly modifying the `Makefile` or source code.

---

### 4.2. Why Use `EXTRA_OEMAKE`?

The main reasons to use `EXTRA_OEMAKE` are:

1. **Customizing the Build**: When the `Makefile` supports extra options (such as specifying libraries, paths, or compiler flags), `EXTRA_OEMAKE` allows you to pass those options at build time.
   
2. **Cross-Compiling**: Yocto is often used for cross-compiling, so you may need to pass specific toolchain variables (e.g., `CC`, `CXX`, `LDFLAGS`) to ensure that the correct compiler and linker are used.

3. **Environment-Specific Flags**: Some builds require specific flags for optimization, debugging, or feature toggles that need to be passed to `make`. Using `EXTRA_OEMAKE`, you can specify these without hardcoding them into the `Makefile`.

4. **Avoid Modifying Makefiles**: Instead of editing the `Makefile` directly (which could lead to maintenance issues or compatibility problems when upstream versions change), you can use `EXTRA_OEMAKE` to pass necessary options or overrides.

### 4.3. How to Use `EXTRA_OEMAKE`?

You define `EXTRA_OEMAKE` in your Yocto recipe to pass additional flags or environment variables to `make`.

#### 4.3.1. **Basic Usage:**

```bash
EXTRA_OEMAKE = "CFLAGS='-O2' LDFLAGS='-L/path/to/lib' CC=${CC}"
```

This appends `CFLAGS`, `LDFLAGS`, and `CC` to the `make` command when it runs.

#### 4.3.2. **Example in a Recipe:**

```bash
SUMMARY = "My custom application using make"
DESCRIPTION = "An example recipe using EXTRA_OEMAKE to customize make flags."
LICENSE = "MIT"

SRC_URI = "git://example.com/myproject.git"
SRCREV = "abcdef1234567890"

S = "${WORKDIR}/git"

inherit autotools

EXTRA_OEMAKE = "CFLAGS='-O3' LDFLAGS='-lmylib'"

do_compile() {
    oe_runmake
}

do_install() {
    oe_runmake install
}
```

In this example:
- **`EXTRA_OEMAKE`** sets the `CFLAGS` to `-O3` (optimize for performance) and links against a custom library `-lmylib`.
- These flags are passed automatically to `make` when `oe_runmake` is called in `do_compile` and `do_install`.

### 4.4. `EXTRA_OEMAKE` Relation with Makefile

The `EXTRA_OEMAKE` variable is tightly related to the `Makefile` because it passes additional options to the `make` process that influence how the `Makefile` is executed. A typical `Makefile` often uses environment variables like `CC`, `CFLAGS`, `LDFLAGS`, and `PREFIX` to configure the build process. By using `EXTRA_OEMAKE`, you can modify these variables without modifying the `Makefile` itself.

For example:
- If a `Makefile` uses `$(CC)` to define the compiler, you can pass a custom compiler in `EXTRA_OEMAKE` by defining `CC=gcc`.
- If the `Makefile` uses `$(CFLAGS)` for optimization flags, you can customize `CFLAGS` using `EXTRA_OEMAKE`.

#### 4.4.1. Example Makefile Snippet:
```makefile
CC ?= gcc
CFLAGS ?= -O2

all:
	$(CC) $(CFLAGS) -o myapp myapp.c
```

In this example, `EXTRA_OEMAKE = "CC=arm-linux-gnueabihf-gcc CFLAGS='-O3'"` would override the `CC` and `CFLAGS` values to use a specific cross-compiler and custom optimization level.

---

### 4.5. How to Pass Libraries and Other Parameters to Build Environment?

When you need to pass additional libraries, include paths, or other build parameters, you can use `EXTRA_OEMAKE` to set the relevant environment variables. The most common variables you might need to adjust include:

- **`CFLAGS`**: Compiler flags (e.g., `-I/path/to/include` for include paths, `-O2` for optimization).
- **`LDFLAGS`**: Linker flags (e.g., `-L/path/to/libs` for library paths, `-lm` to link a specific library).
- **`CC`**: The C compiler to use (especially important for cross-compilation).
- **`CXX`**: The C++ compiler.
- **`PREFIX`**: The installation prefix (e.g., `/usr` or `/usr/local`).
- **`LIBS`**: Additional libraries to link against.

#### 4.5.1. Example with Libraries:

```bash
EXTRA_OEMAKE = "CFLAGS='-O2 -I${STAGING_DIR_TARGET}/usr/include' \
                LDFLAGS='-L${STAGING_DIR_TARGET}/usr/lib -lcustomlib'"
```

This example passes:
- **`CFLAGS`**: Adds an optimization flag `-O2` and specifies an include path for headers.
- **`LDFLAGS`**: Adds a library search path and links against a custom library (`-lcustomlib`).

#### 4.5.2. Example in a Recipe:

```bash
SUMMARY = "Example application with custom libraries"
DESCRIPTION = "An example Yocto recipe that links against a custom library."
LICENSE = "MIT"

SRC_URI = "git://example.com/myproject.git;branch=main"
SRCREV = "abcdef1234567890"

S = "${WORKDIR}/git"

inherit autotools

DEPENDS = "customlib"

EXTRA_OEMAKE = "CFLAGS='-I${STAGING_DIR_TARGET}/usr/include/customlib' \
                LDFLAGS='-L${STAGING_DIR_TARGET}/usr/lib -lcustomlib'"

do_compile() {
    oe_runmake
}

do_install() {
    oe_runmake install
}
```

In this example:
- The project depends on `customlib`, and the `EXTRA_OEMAKE` variable ensures that the compiler can find the necessary headers and libraries during the build process.
- `CFLAGS` points to the include directory for `customlib`.
- `LDFLAGS` links against the `customlib` library.

### 4.6. EXTRA_OEMAKE to customize the parameters passed to the compiler
`EXTRA_OEMAKE` is a Yocto variable that allows you to pass additional arguments or flags to the `make` command, which is invoked during the build process. One common use case is to **customize the parameters passed to the compiler**.

In a typical `Makefile`, the compiler's behavior can be controlled by environment variables like `CFLAGS` (for C code) or `CXXFLAGS` (for C++ code). These variables define flags such as optimization levels (`-O2`, `-O3`), debugging information (`-g`), or architecture-specific options. By using `EXTRA_OEMAKE`, you can modify these flags without changing the actual `Makefile`, making the build process more flexible and manageable.

### 4.7. Example Use Case: Optimization Flags

Let's say you want to optimize the code for performance using `-O3` or for size using `-Os`. You can set this in `EXTRA_OEMAKE` as follows:

```bash
EXTRA_OEMAKE = "CFLAGS='-O3'"
```

This tells Yocto to pass the `-O3` optimization flag to the compiler during the build. Similarly, if you want to optimize for size, you can use:

```bash
EXTRA_OEMAKE = "CFLAGS='-Os'"
```

This example illustrates how you can **customize compiler optimization parameters** for a build by using `EXTRA_OEMAKE`. The recipe remains simple, and you can switch between different optimization levels based on your needs.
