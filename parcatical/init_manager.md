# Init manager
Init manager or init system is a fundamental SW component in a a Unix-like operating systems whch is responsible for initializing the system dusing the boot process, managing the system processes and services throughout the system rate. So it is the first program to run when the operating system starts and has process ID(`PID`) 1.
### 1. **A Brief Introduction to Init Manager**

The **init manager** is the first process started by the Linux kernel during the booting process. It has the process identifier (PID) of 1 and remains active until the system shuts down. Its primary role is to initialize the user space and manage system processes, ensuring that all necessary services and applications start in the correct order. Historically, the traditional System V `init` was used, but modern systems have adopted more advanced init systems like **systemd**, **Upstart**, and **OpenRC** to handle complex dependencies and improve boot times.

---

### 2. **What Does the Init Manager Do?**

The init manager performs several vital functions:

- **System Initialization:** Sets up the user space by mounting file systems, setting up device nodes, and initializing hardware components.
  
- **Service Management:** Starts, stops, and supervises system services (daemons) required for the operating system and applications to function.
  
- **Process Supervision:** Monitors system processes, ensuring they are running correctly and restarting them if they fail.
  
- **Runlevel Management:** Defines different modes of operation (runlevels) such as single-user mode, multi-user mode, graphical mode, etc., and transitions the system between these states.
  
- **Dependency Handling:** Manages dependencies between services to ensure they start and stop in the correct order.
  
- **Logging:** Often integrates with or provides logging mechanisms to record system and service events.


Init managers are responsible for system boot and initialization:
* initializing the system boot.
* initializing the system H.W.
* mounting file system.
* setting of essential system parameters.
* starting essential system services.
* services management:
    * manages system services and daemons(starting, stopping, restarting, shutdown, reboot, and monitoring services)
    * dependency resolution => manages services dependencies:
        * ensuring that services are starting in the correct order

---

### 3. **Available Init Managers**

Several init managers are available for Linux systems, each with its own design philosophy and feature set:

- **System V Init (`sysvinit`):** The traditional init system based on runlevels. It uses shell scripts to start and stop services. Widely used in older Linux distributions.
  
- **systemd:** A modern, feature-rich init system that uses unit files for configuration, parallelizes service starts, handles dependencies automatically, and integrates closely with other system components.
  
- **Upstart:** An event-based init system developed by Ubuntu, designed to handle starting services based on events. It aimed to be more flexible than `sysvinit` but has largely been superseded by systemd.
  
- **OpenRC:** A dependency-based init system used primarily by Gentoo and some other distributions. It uses shell scripts and does not require the use of a specific programming language.
  
- **runit:** A lightweight, cross-platform init scheme with a focus on simplicity and speed. It provides process supervision and service management.
  
- **s6:** A small, secure, and efficient init system and process supervisor, suitable for containers and minimal environments.

- **BusyBox init**

---

### 4. **What Is systemd Init Manager?**

**systemd** is a comprehensive and widely adopted init system and service manager for Linux operating systems. Introduced to address the limitations of traditional init systems, systemd offers several advanced features:

- **Parallelization:** systemd can start multiple services simultaneously, significantly reducing boot times.
  
- **Dependency Management:** Automatically handles service dependencies, ensuring services start and stop in the correct order.
  
- **Unit Files:** Uses declarative unit files to describe services, sockets, devices, mounts, and other system components, making configuration more straightforward and consistent.
  
- **Socket and D-Bus Activation:** Starts services on-demand when they receive network or IPC traffic, conserving system resources.
  
- **Journal Logging (`journald`):** Provides a centralized and structured logging system integrated with systemd.
  
- **Target Units:** Replaces traditional runlevels with target units, allowing for more flexible system states.
  
- **Resource Control:** Integrates with Linux control groups (cgroups) to manage and limit resources allocated to services.
  
- **Extensibility:** Offers a rich set of APIs and hooks for extending functionality and integrating with other system components.

**Advantages of systemd:**

- **Fast Boot Times:** Due to parallelization and efficient dependency handling.
  
- **Unified Management:** Centralizes service and system management, providing consistent tools like `systemctl`.
  
- **Active Development:** Continually updated with new features and improvements.
  
- **Wide Adoption:** Used by major Linux distributions such as Fedora, Ubuntu (since 15.04), Debian, CentOS, and Arch Linux.

**Criticisms of systemd:**

- **Complexity:** Its extensive feature set makes it more complex than traditional init systems.
  
- **Monolithic Design:** Some argue it violates the Unix philosophy of "doing one thing well" by integrating multiple functionalities.
  
- **Adoption Resistance:** Certain communities prefer simpler, more modular init systems.

---

### 5. **How to Integrate systemd in Yocto**

**By default yocto use SysV init as init system.**

**Yocto Project** is an open-source collaboration project that provides templates, tools, and methods to create custom Linux distributions for embedded systems. Integrating `systemd` into a Yocto-based build involves the following steps:

#### **a. Ensure systemd is Supported:**

Most modern Yocto layers include support for `systemd`. Ensure that your Yocto environment is updated and compatible.

#### **b. Modify Local Configuration:**

In your `conf/local.conf` file, set the `DISTRO` variable to use a distribution that includes `systemd` or explicitly set the init manager.

For example:

```bash
DISTRO_FEATURES_append = " systemd"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = "systemd"
DISTRO_FEATURES_BACKFILL_CONSIDERED += "sysvinit"
```

#### **c. Select systemd as the Init Manager:**

Ensure that `systemd` is selected as the init manager by adding the following to `local.conf` or your distribution configuration file:

```bash
DISTRO_FEATURES_append = " systemd"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = "systemd"
DISTRO_FEATURES_BACKFILL_CONSIDERED += "sysvinit"
```

#### **d. Configure Recipes for systemd:**

Make sure that all system services use `systemd` service files. Most modern recipes already include `systemd` support. If not, you may need to modify the recipes to add appropriate `systemd` service files.

#### **e. Enable systemd Services:**

Use the `SYSTEMD_AUTO_ENABLE` variable in your recipes to ensure services are enabled or disabled as needed. For example, in a .bb recipe file:

```bitbake
SYSTEMD_AUTO_ENABLE = "enable"
```

#### **f. Rebuild the Image:**

After making the necessary configuration changes, rebuild your Yocto image:

```bash
bitbake <your-image>
```

#### **g. Post-Deployment Configuration:**

After deploying the image, verify that `systemd` is running as PID 1:

```bash
ps -p 1 -o comm=
```

It should output `systemd`.

**Additional Tips:**

- **Read Yocto Documentation:** Yocto provides detailed documentation on integrating `systemd`.
  
- **Use meta-systemd Layer:** If your Yocto setup doesn't include `systemd` support by default, consider adding the `meta-systemd` layer.
  
- **Test Services:** Ensure that all your required services have compatible `systemd` unit files and are properly configured.

---

### 6. **Comparison Between Init Managers**

Below is a comparison of various init managers based on different criteria:

-----
| Feature                  | SysV Init                      | systemd                             | Upstart                            | OpenRC                             | runit                                | s6                                   |
|--------------------------|--------------------------------|-------------------------------------|------------------------------------|------------------------------------|--------------------------------------|--------------------------------------|
| **Initialization Style** | Sequential (shell scripts)     | Parallel (unit files)               | Event-driven                       | Dependency-based                   | Parallel/service supervision         | Minimal/process supervision          |
| **Configuration**        | Shell Scripts                  | Declarative Unit Files               | Event Scripts                      | Shell Scripts                      | Simple Service Scripts               | Simple Service Scripts               |
| **Dependency Handling**  | Manual scripting               | Automatic and robust                 | Event-based dependencies            | Manual with some automation         | Minimal, relies on script order      | Minimal, relies on script order      |
| **Service Supervision**  | Limited                        | Built-in                              | Basic                               | Limited                            | Included (supervise)                  | Included (supervise)                  |
| **Logging**              | External tools (syslog)        | Built-in (`journald`)                | External tools                      | External tools                      | External tools                        | External tools                        |
| **Resource Control**     | Not integrated                 | Integrated with cgroups              | Limited                             | Limited                            | Limited                                | Limited                                |
| **Extensibility**        | Low                            | High                                  | Moderate                            | Moderate                            | Low                                    | Low                                    |
| **Adoption**             | Legacy systems, older distros  | Most major modern Linux distributions | Ubuntu (historically)              | Gentoo, Alpine Linux, others        | Minimalist distributions, containers | Minimalist distributions, containers |
| **Complexity**           | Low to Moderate                 | High                                   | Moderate                            | Low                                 | Low                                    | Low                                    |
| **Boot Speed**           | Slower due to sequential start | Faster due to parallelization         | Moderate                            | Comparable to SysV, some parallelism| Very fast due to simplicity           | Very fast due to simplicity           |
| **Key Strengths**        | Simplicity, stability           | Feature-rich, fast boot, widespread   | Event-based flexibility             | Lightweight, versatility            | Speed, simplicity                      | Minimalism, security                  |
| **Key Weaknesses**       | Slow boot, limited features     | Complexity, potential for bugs        | Less adoption, superseded by systemd| Less automated dependency handling  | Limited features, less flexibility    | Limited features, requires expertise  |
----
- **SysV Init:** Traditional, simple but slow and less feature-rich. Suitable for legacy systems.
  
- **systemd:** Modern, feature-rich, high adoption, fast boot times, but complex and monolithic.
  
- **Upstart:** Event-driven, was a bridge between SysV and systemd, now largely replaced by systemd.
  
- **OpenRC:** Lightweight, dependency-based without heavy integration, suitable for environments valuing simplicity.
  
- **runit & s6:** Minimalist and fast, ideal for containers and systems requiring minimal overhead.

**Choosing the Right Init Manager:**

The choice depends on the specific requirements of your system:

- **For modern desktop/server systems:** **systemd** is typically the best choice due to its extensive features and support.
  
- **For embedded or minimal systems:** Lightweight options like **OpenRC**, **runit**, or **s6** might be more appropriate.
  
- **For legacy systems:** **SysV Init** may still be in use, but migrating to a more modern system is advisable if possible.

### 7. **Most used init manager comparison**:
------
| Feature                     | **systemd init**                         | **SysV init**                          | **BusyBox init**                      |
|-----------------------------|------------------------------------------|----------------------------------------|---------------------------------------|
| **Type**                     | Modern init system with parallelization | Traditional init system                | Lightweight init system               |
| **Initialization Speed**     | Fast, uses parallel execution            | Slower, sequential startup             | Fast, designed for minimal systems    |
| **Dependency Management**    | Full dependency-based startup            | Script-based with no dependency management | Simple, minimal dependencies          |
| **Service Management**       | Centralized with `systemctl` commands    | Manual, uses `/etc/init.d` scripts     | Basic service management via scripts  |
| **Target Audience**          | General-purpose systems (servers, desktops) | Older UNIX/Linux systems               | Embedded systems and lightweight distros |
| **Resource Usage**           | Higher due to complex features           | Lower, but less efficient handling     | Extremely low, designed for embedded systems |
| **Logging**                  | Integrated with `journald`               | Logs using syslog or similar           | Limited or depends on syslog          |
| **Process Supervision**      | Built-in (automatically restarts services) | No built-in supervision                | Limited supervision                   |
| **Configuration Complexity** | Higher, uses `.service` files and units  | Lower, uses shell scripts              | Simple, very minimal shell scripts    |
| **Modularity**               | Modular, with many components            | Monolithic (single-layer scripts)      | Highly minimalistic                   |
| **Parallel Startup**         | Yes, services start in parallel          | No, services start sequentially        | Limited parallelization               |
| **Hotplug Hardware Support**  | Yes, through `udev` integration          | No native support                      | Minimal support                       |
| **Widely Used In**           | Most modern Linux distros (Ubuntu, Fedora, Arch Linux) | Older Linux systems (RHEL 6, Debian 6) | Lightweight and embedded systems (Alpine Linux, TinyCore) |
------

#### Key Takeaways:

<p align="center"> Consized comparison </p>
<p align="center">
  <img src="Screenshot%20from%202024-09-29%2010-43-01.png" alt="Small comparison" />
</p>

- **systemd** is a modern, feature-rich init system aimed at providing faster boot times and managing services effectively, but it comes at the cost of complexity and higher resource usage.
- **SysV init** is the older, simpler system that relies on sequential startup and manual management of services. It’s still used in some older or simpler environments but lacks advanced features.
- **BusyBox init** is highly minimal, designed for environments where resources are scarce, like embedded systems or minimal Linux distributions. It sacrifices many features for simplicity and speed.
