# Configure, Compile, and Install

### Classes in OpenEmbedded

1. **Classes and Schemes**: 
   - OpenEmbedded, which is part of the Yocto Project, uses *classes* to define how different types of software packages should be built.
   - There are standard schemes provided for building different types of packages:
     - **Make-based packages**: Packages that use `Makefile` for building.
     - **GNU Autotools–based packages**: Packages that use GNU Autotools (`configure`, `make`, `make install`).
     - **CMake-based packages**: Packages that use `CMake` as the build system.

2. **Standardized Environment Settings**:
   - These schemes (classes) provide a standardized way to specify environment settings required to build the software. This ensures consistency and ease of customization across different packages.

### Building Packages with BitBake

1. **BitBake**:
   - BitBake is the task scheduler used by OpenEmbedded and the Yocto Project to build packages.
   - It uses the recipes and classes to automate the process of configuring, compiling, and installing software.

2. **Configuration, Compilation, and Installation**:
   - **Configuring**: Preparing the package's build system to compile the software on the target platform.
   - **Compiling**: Actually building the software from source code.
   - **Installing**: Placing the built software into the correct directories for use.

### Customization and Chapter Reference

- Customizing how packages are built using these schemes is covered in Chapter 8, "Software Package Recipes." This chapter likely details how to modify and extend the standard build process for your specific needs.

### Install Step and `pseudo` Command

1. **Install Step**:
   - The installation step is crucial as it places the built software into the correct directories and sets the right permissions.

2. **`pseudo` Command**:
   - The `pseudo` command is a tool used during the installation step to handle file permissions and ownership.
   - It allows the build process to create files with specific ownership and permissions even if the build user does not have the necessary privileges.
   - This is essential for creating special files (like device nodes) and ensuring files have the correct permissions and ownerships (owner, group, others).

3. **Private System Root Directory**:
   - All files are installed into a private system root directory within the build environment.
   - This means each package is installed into its own isolated directory during the build process.
   - This helps to avoid conflicts and ensures a clean build environment.

In summary, OpenEmbedded's classes and schemes provide a standardized way to build different types of software packages. BitBake orchestrates the build process, and the `pseudo` command helps manage file permissions and ownership during installation. The details of customization are covered in a specific chapter of your study material.

In the scope of the Yocto Project and OpenEmbedded, the terms "schemes" and "packages" have specific meanings related to the build process.

### Schemes

**Schemes** refer to the standardized methods or frameworks provided by OpenEmbedded to build different types of software packages. These schemes encapsulate the procedures and rules needed to configure, compile, and install software. They ensure consistency and simplify the build process by providing predefined ways to handle various build systems. Here are some common schemes:

1. **Make-based Packages**:
   - These packages use `Makefile` for their build process.
   - The scheme for Make-based packages defines how to run the `make` command to build and install the software.

2. **GNU Autotools-based Packages**:
   - These packages use GNU Autotools (`autoconf`, `automake`, `libtool`) for the build process.
   - The scheme for Autotools-based packages includes steps to run `./configure`, `make`, and `make install`.

3. **CMake-based Packages**:
   - These packages use CMake, a cross-platform build system generator.
   - The scheme for CMake-based packages defines how to use `cmake` commands to build and install the software.

These schemes are implemented through `.bbclass` files (classes) in OpenEmbedded, providing reusable chunks of configuration and code.

### Packages

**Packages** in this context refer to the individual software components or applications that you want to build using OpenEmbedded. A package typically includes:

1. **Source Code**:
   - The actual code of the software that needs to be compiled.

2. **Recipes**:
   - BitBake recipes (`.bb` files) that provide metadata and instructions on how to build the package.
   - A recipe specifies things like where to get the source code, how to configure the build, compile the code, and install the resulting binaries.

3. **Build Instructions**:
   - Steps to configure, compile, and install the software.
   - These steps are defined in the recipe and often rely on the schemes provided by the classes.

### Putting It Together

- **Schemes**: Standardized methods (defined in classes) that outline how to build different types of software packages.
- **Packages**: Individual software components that need to be built, each with its own recipe defining the build instructions.

For example, if you have a package that uses GNU Autotools for its build system, you would write a recipe for that package, and the build process would use the Autotools scheme (class) to handle the specific steps needed to configure, compile, and install that package. This ensures a standardized and efficient way to handle the build process across various types of software.


### Example: Building a Simple Application Using GNU Autotools

#### 1. The Package: "hello-world"

Let's assume we have a simple "hello-world" application that uses GNU Autotools for its build system.

#### 2. Source Code

The source code for "hello-world" is straightforward. Here’s a minimal example:

- **hello.c**:
    ```
    #include <stdio.h>

    int main() {
        printf("Hello, world!\n");
        return 0;
    }
    ```

- **configure.ac**:
    ```
    AC_INIT([hello-world], [1.0], [your-email@example.com])
    AM_INIT_AUTOMAKE
    AC_PROG_CC
    AC_CONFIG_FILES([Makefile])
    AC_OUTPUT
    ```

- **Makefile.am**:
    ```
    bin_PROGRAMS = hello
    hello_SOURCES = hello.c
    ```

#### 3. The Recipe

Create a BitBake recipe to build the "hello-world" package.

- **hello-world_1.0.bb**:
    ```
    DESCRIPTION = "Simple Hello World application"
    LICENSE = "GPL-2.0-only"
    LIC_FILES_CHKSUM = "file://LICENSE;md5=<md5sum_of_license_file>"
    
    SRC_URI = "file://hello.c \
               file://configure.ac \
               file://Makefile.am"
    SRC_URI[md5sum] = "<md5sum_of_source_tarball>"
    SRC_URI[sha256sum] = "<sha256sum_of_source_tarball>"
    
    inherit autotools

    do_install() {
        install -d ${D}${bindir}
        install -m 0755 hello ${D}${bindir}/hello
    }
    ```

#### 4. The Class: Autotools

OpenEmbedded provides the `autotools.bbclass` to handle GNU Autotools-based packages. When you inherit `autotools` in your recipe, it automatically includes the steps for:

- Running `./configure` to configure the package.
- Running `make` to compile the package.
- Running `make install` to install the package into a staging directory.

#### 5. Build Instructions

To build the package using Yocto, follow these steps:

1. **Set up the Yocto build environment**:
    ```
    source oe-init-build-env
    ```

2. **Create the necessary directory structure** (if not already existing):
    ```
    mkdir -p meta-my-layer/recipes-example/hello-world/files
    ```

3. **Place the source files and the recipe**:
    - Copy `hello.c`, `configure.ac`, and `Makefile.am` into `meta-my-layer/recipes-example/hello-world/files`.
    - Copy `hello-world_1.0.bb` into `meta-my-layer/recipes-example/hello-world`.

4. **Build the package**:
    ```
    bitbake hello-world
    ```

This will execute the following steps:

- **Fetch the Source Code**: Download the source files specified in `SRC_URI`.
- **Configure**: Run `./configure` to prepare the build environment.
- **Compile**: Run `make` to compile the `hello-world` application.
- **Install**: Run `make install` to install the `hello-world` binary into a staging directory, using the `do_install` function to place it in the correct location.

### Summary

- **Scheme**: The `autotools` class provides a standardized way to handle the configuration, compilation, and installation steps for GNU Autotools-based packages.
- **Package**: The "hello-world" application, with its source code and a BitBake recipe that specifies how to build it using the `autotools` scheme.

This example demonstrates how the Yocto Project and OpenEmbedded use classes (schemes) and recipes to build software packages in a consistent and reusable manner.


### what is configure.ac and Makefile.am?
`configure.ac` and `Makefile.am` are key files used in the GNU Autotools build system, which is a suite of tools that make it easier to build, install, and distribute software.

#### configure.ac

`configure.ac` is a script that contains macros used by `autoconf` to generate a `configure` script. The `configure` script is responsible for setting up the build environment by checking for required tools and libraries, as well as setting up configuration options based on the system where the software will be built.

##### Example

Here’s a simple `configure.ac` for the "hello-world" application:

```
AC_INIT([hello-world], [1.0], [your-email@example.com])
AM_INIT_AUTOMAKE
AC_PROG_CC
AC_CONFIG_FILES([Makefile])
AC_OUTPUT
```

**Explanation**:
- `AC_INIT`: Initializes the configuration, specifying the package name, version, and contact email.
- `AM_INIT_AUTOMAKE`: Initializes Automake, which helps in generating Makefile templates.
- `AC_PROG_CC`: Checks for a C compiler and sets up the necessary environment.
- `AC_CONFIG_FILES`: Specifies the files to be processed by `configure` to produce output files (in this case, `Makefile`).
- `AC_OUTPUT`: Produces the `configure` script.

#### Makefile.am

`Makefile.am` is a high-level makefile used by `automake` to generate a `Makefile.in` file. The `Makefile.in` file is then used by the `configure` script to generate the final `Makefile`, which contains the actual build rules.

##### Example

Here’s a simple `Makefile.am` for the "hello-world" application:

```
bin_PROGRAMS = hello
hello_SOURCES = hello.c
```

**Explanation**:
- `bin_PROGRAMS`: Specifies the programs to be built and installed in the `bin` directory.
- `hello_SOURCES`: Lists the source files for the `hello` program.

#### How They Work Together

1. **configure.ac**: Defines the macros and checks required for setting up the build environment.
2. **autoconf**: Processes `configure.ac` to generate the `configure` script.
3. **configure script**: When run, this script checks the build environment and generates `Makefile` from `Makefile.in`.
4. **Makefile.am**: Defines the high-level build instructions.
5. **automake**: Processes `Makefile.am` to generate `Makefile.in`.
6. **Makefile**: The final makefile generated by the `configure` script, which is used by `make` to build the software.

#### Example Workflow

1. **Developer writes `configure.ac` and `Makefile.am`**.
2. **Run `autoconf`**:
   ```
   autoconf
   ```
   This generates the `configure` script from `configure.ac`.

3. **Run `automake`**:
   ```
   automake --add-missing
   ```
   This generates `Makefile.in` from `Makefile.am`.

4. **Run `configure`**:
   ```
   ./configure
   ```
   This generates the `Makefile` from `Makefile.in`, performing system checks and setting configuration options.

5. **Run `make`**:
   ```
   make
   ```
   This uses the generated `Makefile` to compile the program.

6. **Run `make install`**:
   ```
   make install
   ```
   This installs the compiled program to the specified location (usually `/usr/local/bin`).

By using `configure.ac` and `Makefile.am`, developers can ensure that their software can be easily built and installed on different systems with varying configurations. This is particularly useful for creating portable and distributable software packages.


### Output Analysis and Packaging

#### Output Analysis

**Output analysis** is a crucial step in the build process where the software that has been compiled and installed is examined and categorized. This categorization is based on the functionality and purpose of the files. Here are the common categories:

1. **Runtime Files**:
   - These are the essential files needed to run the application or software package. They include binaries, libraries, configuration files, and other necessary runtime components.

2. **Debug Files**:
   - These files contain debugging information that can be used for troubleshooting and development purposes. They often include symbols and other metadata needed for debugging the application.

3. **Development Files**:
   - These files are required for developing software that depends on the package. They typically include header files, static libraries, and other development resources.

4. **Documentation**:
   - This includes manuals, help files, and other forms of documentation that provide information on how to use and maintain the software.

5. **Locales**:
   - Locales are files that provide localization and internationalization support, including translations for different languages.

#### Packaging

Once the output analysis is complete, the categorized files are split into multiple physical packages. This separation allows for more granular control over what gets installed on a system. Each category of files can be packaged into its own package. 

Common packaging formats include:

1. **RPM** (Red Hat Package Manager):
   - Used by distributions like Fedora, RHEL, and CentOS.

2. **dpkg** (Debian Package):
   - Used by Debian and its derivatives like Ubuntu.

3. **ipkg** (Itsy Package Manager):
   - A lightweight package management system used for embedded systems.

#### BitBake and Package Management

BitBake, the build tool used by the Yocto Project, is responsible for creating these packages. The types of packages that BitBake creates are determined by the `PACKAGE_CLASSES` variable in the build environment’s configuration file `local.conf`.

**PACKAGE_CLASSES** example in `local.conf`:
```
PACKAGE_CLASSES = "package_rpm package_deb package_ipk"
```

- **Multiple Package Formats**:
  - BitBake can create packages for multiple package management systems as specified in the `PACKAGE_CLASSES` variable. This means it can generate RPM, dpkg, and ipkg packages from the same build process.

- **Primary Package Format**:
  - Although BitBake can generate packages for various formats, it will use only the first listed format in the `PACKAGE_CLASSES` variable to create the final root filesystem for the distribution. In the example above, `package_rpm` is listed first, so RPM packages would be used to create the root filesystem.

#### Example Workflow

1. **Build Process**:
   - BitBake compiles and installs the software, generating the output files.

2. **Output Analysis**:
   - The generated files are analyzed and categorized into runtime files, debug files, development files, documentation, and locales.

3. **Packaging**:
   - The categorized files are split into separate packages using the specified packaging formats (RPM, dpkg, ipkg).

4. **Final Root Filesystem**:
   - The final root filesystem for the distribution is created using the primary packaging format specified in `PACKAGE_CLASSES`.

### Summary

- **Output Analysis**: Categorizes files into different functional groups after the software is built.
- **Packaging**: Creates separate packages for each category of files in multiple packaging formats (RPM, dpkg, ipkg).
- **BitBake and `PACKAGE_CLASSES`**: Determines which packaging formats to use, with the first listed format used to create the final root filesystem.

This process ensures that the software is well-organized, easy to manage, and can be deployed on systems that use different package management systems.

### Image Creation Process

**Image Creation** involves assembling a complete root filesystem for the distribution by using the packages built in the packaging step. Here’s a detailed look at each part of the process:

### 1. Using Package Feeds

- **Package Feeds**:
  - Package feeds are repositories of pre-built packages generated during the packaging step.
  - These packages include the runtime files, debug files, development files, documentation, and locales categorized earlier.

### 2. Root Filesystem Staging Area

- **Staging Area**:
  - The root filesystem is built in a staging area, which is a temporary location where the packages are installed.
  - The package management system (e.g., RPM, dpkg, ipkg) installs packages from the package feeds into this staging area.

### 3. Image Recipes

- **Image Recipes**:
  - Image recipes are specialized BitBake recipes that define which packages should be included in the final image.
  - They assemble a functional set of packages based on the requirements of the target system.

### 4. Types of Images

- **Minimal Image**:
  - A minimal image includes just enough packages to boot the system and provide basic console operations.
  - Example: Core utilities, minimal shell, and essential system libraries.

- **Graphical User Interface (GUI) Image**:
  - A GUI image includes additional packages to support a graphical user interface.
  - Example: X server, window manager, GUI applications, and related libraries.

### 5. Core Image Class

- **core-image Class**:
  - The `core-image` class handles the creation of images.
  - It evaluates the `IMAGE_INSTALL` variable, which lists the packages to be included in the image.

### 6. IMAGE_INSTALL Variable

- **IMAGE_INSTALL**:
  - This variable specifies the list of packages to be included in the image.
  - It is defined within the image recipe and might look something like this:
    ```
    IMAGE_INSTALL += "package1 package2 package3"
    ```

### 7. Image Formats

- **Image Formats**:
  - Images can be created in various formats to suit different deployment scenarios.
  - **tar.bz2**: A compressed tarball for extraction into a pre-formatted filesystem.
  - **ext2, ext3, ext4**: Common Linux filesystem formats that can be directly copied to storage devices.
  - **jffs**: A flash filesystem format for use on flash memory devices.

### Example Workflow

1. **Build Packages**:
   - BitBake compiles and packages the software into various categories (runtime, debug, etc.).

2. **Define Image Recipe**:
   - Create an image recipe that specifies which packages to include. For example, `core-image-minimal.bb`:
    ```
    IMAGE_INSTALL += "package1 package2 package3"
    ```

3. **Use Package Feeds**:
   - The packages are fetched from the package feeds and installed into the root filesystem staging area.

4. **Create Root Filesystem**:
   - The core-image class uses the `IMAGE_INSTALL` variable to include the specified packages in the root filesystem.

5. **Generate Image**:
   - The root filesystem is bundled into the desired format (e.g., tar.bz2, ext4).

### Summary

- **Image Creation**: Assembles a complete root filesystem by installing packages from package feeds into a staging area.
- **Image Recipes**: Define which packages to include based on system requirements.
- **core-image Class**: Handles image creation and evaluates the `IMAGE_INSTALL` variable for package inclusion.
- **Image Formats**: Various formats are available to suit different deployment needs (e.g., tar.bz2, ext4, jffs).

This process ensures that the resulting images are tailored to specific requirements, whether for minimal functionality or full-featured graphical interfaces, and are prepared in formats suitable for various deployment methods.
