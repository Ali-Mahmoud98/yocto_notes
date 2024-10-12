## RESOURCES
* [Variables Glossary](https://docs.yoctoproject.org/ref-manual/variables.html)
* [Yocto Tasks](https://docs.yoctoproject.org/dev/ref-manual/tasks.html)
* [Clean tasks](https://docs.yoctoproject.org/1.7.3/ref-manual/ref-manual.html#ref-tasks-clean)

## BBMASK
In Yocto, `bbmask` is a variable used to exclude or ignore specific BitBake recipe files or classes based on a regular expression pattern. This feature is helpful when you want to prevent certain layers or recipes from being processed during a build, such as when you're dealing with conflicting recipes, unnecessary layers, or deprecated components.

### Example Usage:
You typically define `BBMASK` in the `conf/local.conf` or `layer.conf` file of your Yocto project:

```bash
BBMASK = "meta-foo/recipes-bar/*"
```

In this example:
- The `BBMASK` is defined to ignore all the recipes inside the `recipes-bar` folder of the `meta-foo` layer.
- BitBake will not consider any recipes under the specified folder during the build process.

`BBMASK` supports regular expressions, so you can apply more complex patterns if needed, such as ignoring multiple directories or specific versions of recipes.

### Typical Use Cases:
- **Conflicting layers:** When two or more layers contain recipes that conflict, you can use `BBMASK` to exclude one of them.
- **Customization:** If you want to selectively disable recipes from a layer but not the entire layer, you can target specific directories or recipes within the layer.
- **Debugging:** During development, you can mask certain recipes to isolate build issues.

This feature provides better control over the build process when working with multiple layers or handling custom requirements in Yocto.

## bbappend recipe files
### 1. **What is the Original/Base Recipe?**
A Yocto recipe (`.bb` file) is the core component that defines how to build a particular package or software. It specifies the source code location, dependencies, build instructions, configurations, and how to install the output. Each recipe resides in a specific layer and is responsible for building one or more software components.

A typical recipe might include:
- **Source location** (via `SRC_URI`)
- **Build steps** (via `do_compile`, `do_install`, etc.)
- **Package metadata** (e.g., `DESCRIPTION`, `LICENSE`)
- **Dependencies** (e.g., `DEPENDS`, `RDEPENDS`)

For example, a base recipe to build `helloworld` could look like:

```bash
# helloworld_1.0.bb
SUMMARY = "A simple hello world application"
LICENSE = "MIT"
SRC_URI = "http://example.com/helloworld-1.0.tar.gz"

do_compile() {
    # Compile commands go here
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 helloworld ${D}${bindir}/helloworld
}
```

### 2. **What is a `.bbappend` File?**
A `.bbappend` file is an additional file used to modify or extend an existing recipe without directly changing the original `.bb` file. The `.bbappend` file allows you to "append" or "override" certain parts of the base recipe, making it a flexible way to customize the behavior of a recipe.

Think of `.bbappend` files as a way to:
- Add additional tasks or configurations.
- Change or replace values like dependencies, source locations, or patches.
- Customize the behavior of the recipe for specific layers, distributions, or machines.

### 3. **Why `.bbappend` Files are Needed?**
The key reasons for using `.bbappend` files are:
- **Layering and Modularity:** If you want to modify a recipe from a different layer (e.g., a community layer), you shouldn't directly edit the base recipe because that would break the modular structure and make future updates harder. `.bbappend` lets you apply changes without touching the original recipe.
- **Customization for Devices:** When creating custom distributions, you may need to tweak recipes for your specific hardware platform, software environment, or business needs.
- **Maintainability:** If you modify a recipe directly, upgrading or maintaining the Yocto system becomes difficult. `.bbappend` files allow you to make changes while keeping the upgrade path easy by leaving the original recipe intact.

### 4. **Where to Place `.bbappend` Files?**
`.bbappend` files should be placed in the same layer structure as the original recipe's `.bb` file. The path should mirror the original recipe's layer.

For example:
- If the original recipe `helloworld_1.0.bb` is in `meta-foo/recipes-bar/helloworld/helloworld_1.0.bb`, the `.bbappend` file should be placed like this: `meta-custom/recipes-bar/helloworld/helloworld_1.0.bbappend`.

The naming convention for the `.bbappend` file follows the original recipe name, with an identical version number or wildcard:
- `helloworld_1.0.bbappend`
- `helloworld_%.bbappend` (where `%` matches any version of the recipe)

### 5. **A Simple Example**

#### Base Recipe:
Let's say you have a base recipe called `helloworld_1.0.bb` like the example below:

```bash
# helloworld_1.0.bb
SUMMARY = "A simple hello world application"
LICENSE = "MIT"
SRC_URI = "http://example.com/helloworld-1.0.tar.gz"

do_compile() {
    # Compilation steps for helloworld
}
```

#### `.bbappend` File:
Now, you want to apply a patch or change the source location without modifying the base recipe. You can create a `.bbappend` file for this:

```bash
# helloworld_1.0.bbappend
SRC_URI += "file://my-custom-patch.patch"

do_compile_prepend() {
    echo "Building helloworld with my customization"
}

# Optionally, you can also modify the build behavior
EXTRA_OECONF += "--enable-custom-feature"
```

In this `.bbappend` file:
- You added a new patch to the original `SRC_URI`.
- The `do_compile_prepend` task runs before the original `do_compile` function.
- The `EXTRA_OECONF` variable is extended to pass additional flags to the build configuration process.

This way, your changes apply on top of the existing recipe without modifying it directly.

- **Original/Base Recipe:** The core `.bb` file defining the build process.
- **`.bbappend` File:** An extension or modification of the base recipe, allowing customization without altering the original.
- **Why Needed:** To maintain modularity, allow recipe customization, and ensure that updates can be applied without overwriting local changes.
- **Placement:** In the same folder structure as the original recipe but in your custom layer.
- **Example:** Adding a patch or custom build configuration via a `.bbappend` file to modify an existing recipe.

## FILESEXTRAPATHS
`FILESEXTRAPATHS` is a variable used in the Yocto Project's metadata layers to specify additional paths where files (such as patches, configuration files, or other resources) can be found during the build process. It extends the default search paths and allows developers to include files that are not located in the standard directories.

### Why is it Needed?

1. **Customization**: It allows developers to customize or override default resources without modifying the original layer contents.
2. **Modularity**: Projects often consist of multiple layers, and `FILESEXTRAPATHS` facilitates the integration of resources from these layers seamlessly.
3. **Patch Management**: It’s useful for managing patches or other files that may need to be applied to software recipes, ensuring that the build system can locate them easily.

### Syntax for Using `FILESEXTRAPATHS`

The syntax for setting `FILESEXTRAPATHS` is as follows:

```bash
FILESEXTRAPATHS_prepend := "/path/to/your/custom/files:"
```

- `prepend` means that the specified path is added to the beginning of the existing paths, ensuring that it takes precedence over the default ones.
- Use a colon `:` to separate multiple paths.

### Name Convention for Resource Folder

When naming resource folders for use with `FILESEXTRAPATHS`, it’s common to follow a convention that enhances clarity. Here are some general guidelines:

- **Descriptive Names**: Use clear and descriptive names that indicate the contents or purpose of the folder (e.g., `patches`, `configs`, `files`).
- **Layer Name Prefix**: If applicable, prefix the folder name with the layer's name to avoid conflicts (e.g., `my-layer_patches`).
- **Lowercase**: Use lowercase letters to maintain consistency and avoid issues on case-sensitive filesystems.

### A Simple Example

Here’s a simple example of how to use `FILESEXTRAPATHS` in a Yocto recipe.

Assume you have a recipe for a software package called `mysoftware`, and you want to include custom patches located in a directory named `mysoftware_patches`.

1. **Directory Structure**:

```bash
my-layer/
├── recipes-example/
│   └── mysoftware/
│       └── mysoftware_1.0.bb
└── mysoftware_patches/
    ├── fix-bug.patch
    └── update-feature.patch
```

2. **Content of `mysoftware_1.0.bb`**:

```bash
SUMMARY = "My Software Package"
LICENSE = "MIT"

# Extend the FILESEXTRAPATHS variable to include the custom patches
FILESEXTRAPATHS_prepend := "${THISDIR}/../../mysoftware_patches:"

SRC_URI = "git://example.com/mysoftware.git \
           file://fix-bug.patch \
           file://update-feature.patch"
```

In this example, `FILESEXTRAPATHS` allows the build system to locate `fix-bug.patch` and `update-feature.patch` in the `mysoftware_patches` folder when building `mysoftware`.

### Conclusion

`FILESEXTRAPATHS` is a powerful mechanism in Yocto and similar build systems that enhances the flexibility and modularity of software development. By properly managing paths and resources, developers can create more maintainable and customizable software layers.

## clean, cleansstate, and cleanall => Manual Tasks
### do_cleanall
Removes all output files, shared state (sstate) cache, and downloaded source files for a target (i.e. the contents of `DL_DIR`). Essentially, the `do_cleanall` task is identical to the `do_cleansstate` task with the added removal of downloaded source files.

You can run this task using BitBake as follows:
```bash
bitbake -c cleanall recipe_name
```

### do_cleansstate
Removes all output files and shared state ([sstate](https://docs.yoctoproject.org/1.7.3/ref-manual/ref-manual.html#shared-state-cache)) cache for a target. Essentially, the `do_cleansstate` task is identical to the `do_clean` task with the added removal of shared state ([sstate](https://docs.yoctoproject.org/1.7.3/ref-manual/ref-manual.html#shared-state-cache)) cache.

You can run this task using BitBake as follows:
```bash
$ bitbake -c cleansstate recipe
```     

When you run the `do_cleansstate` task, the OpenEmbedded build system no longer uses any sstate. Consequently, building the recipe from scratch is guaranteed.

**Note:**

The `do_cleansstate` task cannot remove sstate from a remote sstate mirror. If you need to build a target from scratch using remote mirrors, use the "-f" option as follows:
```bash
$ bitbake -f -c do_cleansstate target
```

### do_clean
Removes all output files for a target from the `do_unpack` task forward (i.e. `do_unpack`, `do_configure`, `do_compile`, `do_install`, and `do_package`).

You can run this task using BitBake as follows:
```bash
$ bitbake -c clean recipe
```     
Running this task does not remove the **sstate** cache files. Consequently, if no changes have been made and the recipe is rebuilt after cleaning, output files are simply restored from the sstate cache. If you want to remove the sstate cache files for the recipe, you need to use the `do_cleansstate` task instead (i.e. bitbake -c cleansstate recipe).

