## Systemd runlevels
**Runlevels** are an old concept in Unix-like operating systems, including early versions of Linux, that define various operating states or modes of a system. In these systems, runlevels determine what services, processes, and system resources are available at a given time. A runlevel is essentially a mode of operation that dictates what should be running on the system, such as whether it's in multi-user mode, single-user mode, or even powered off.

### Traditional SysV Runlevels

In **SysV (System V)**, which was the traditional init system used before **systemd**, runlevels were used to control system states. They are identified by a single number from 0 to 6. Each number corresponds to a specific operating mode.

Here's a breakdown of the **standard runlevels**:

| **Runlevel** | **Description**                                              |
|--------------|--------------------------------------------------------------|
| **0**        | Halt the system (shutdown).                                  |
| **1**        | Single-user mode (maintenance mode, typically no networking).|
| **2**        | Multi-user mode without networking. (varies by distribution) |
| **3**        | Multi-user mode with networking (no graphical interface).    |
| **4**        | Unused or user-defined (custom configurations).              |
| **5**        | Multi-user mode with networking and a graphical interface.   |
| **6**        | Reboot the system.                                           |

#### Details of Each Runlevel:

1. **Runlevel 0 (Shutdown Mode)**:
   - This runlevel halts the system and powers it off.
   - No services or processes are running in this state.
   - It is typically reached using the `shutdown` or `poweroff` commands.
   
   Example command to switch to runlevel 0 (shut down the system):
   ```bash
   init 0
   ```

2. **Runlevel 1 (Single-User Mode)**:
   - This is a **maintenance or rescue mode**.
   - Only the root user can log in, and typically only essential system services are running.
   - Networking is usually disabled in this runlevel.
   - Useful for troubleshooting issues, performing maintenance tasks, or fixing boot problems.
   
   Example command to enter single-user mode:
   ```bash
   init 1
   ```

3. **Runlevel 2 (Multi-User Mode Without Networking)**:
   - This runlevel enables multiple users to log in, but the system does **not** start networking services.
   - This mode can vary depending on the Linux distribution; for some, networking may be enabled.
   - Suitable for a local system with multiple users but without network access.
   
   Example command:
   ```bash
   init 2
   ```

4. **Runlevel 3 (Multi-User Mode with Networking)**:
   - In this mode, the system allows multiple users to log in, and **networking is enabled**.
   - There is **no graphical interface** (GUI) in this runlevel.
   - Often used in server environments where only command-line interfaces (CLI) are needed.
   
   Example command to switch to runlevel 3:
   ```bash
   init 3
   ```

5. **Runlevel 4 (Unused/User-Defined)**:
   - This runlevel is typically **unused** by default.
   - It can be **customized** by the system administrator to perform any specialized operations.
   - You could configure it to start a specific set of services or boot into a custom environment.
   
   To configure runlevel 4, you would typically modify the init scripts found in `/etc/rc.d/`.

6. **Runlevel 5 (Multi-User Mode with GUI)**:
   - This runlevel is similar to runlevel 3, but it also starts the **graphical display manager** (e.g., GDM, LightDM, or XDM).
   - Most desktop distributions, such as Ubuntu and Fedora, boot into runlevel 5 by default.
   - Useful for desktop environments where a graphical interface is required.
   
   Example command to start the system in graphical mode:
   ```bash
   init 5
   ```

7. **Runlevel 6 (Reboot Mode)**:
   - This runlevel **reboots** the system.
   - When the system enters this runlevel, it will stop all services, cleanly unmount file systems, and then reboot.
   
   Example command to reboot:
   ```bash
   init 6
   ```

### How SysV Init Managed Runlevels

In **SysV init**, each runlevel had its own directory containing symbolic links to the services or scripts that should be started or stopped at that level. These directories are typically found under `/etc/rc.d/` or `/etc/init.d/`, and they are named according to their runlevel:

- `/etc/rc0.d/` - scripts for runlevel 0 (shutdown).
- `/etc/rc1.d/` - scripts for runlevel 1 (single-user mode).
- `/etc/rc2.d/` - scripts for runlevel 2 (multi-user mode, no networking).
- `/etc/rc3.d/` - scripts for runlevel 3 (multi-user mode, networking).
- `/etc/rc5.d/` - scripts for runlevel 5 (graphical interface).
- `/etc/rc6.d/` - scripts for runlevel 6 (reboot).

Scripts in these directories start with either:
- **K** (for "kill") to stop services.
- **S** (for "start") to start services.

For example, if you wanted a service to start at runlevel 3, you would place a symbolic link to the service script in the `/etc/rc3.d/` directory and name it something like `S20servicename`.

### Runlevels in Modern Systems (SystemD)

In modern Linux distributions using **systemd**, runlevels are replaced by **targets**. Targets serve the same purpose as runlevels but offer more flexibility and features, such as parallel startup of services.

Here’s a comparison between traditional SysV runlevels and systemd targets:

| **SysV Runlevel** | **SystemD Target**     | **Description**                             |
|-------------------|------------------------|---------------------------------------------|
| Runlevel 0        | `poweroff.target`       | System shutdown.                           |
| Runlevel 1        | `rescue.target`         | Single-user mode.                          |
| Runlevel 2        | `multi-user.target`     | Multi-user mode without graphical UI (CLI).|
| Runlevel 3        | `multi-user.target`     | Multi-user mode with networking.           |
| Runlevel 4        | `multi-user.target`     | Unused or user-defined (same as Runlevel 3).|
| Runlevel 5        | `graphical.target`      | Multi-user mode with graphical UI (GUI).   |
| Runlevel 6        | `reboot.target`         | System reboot.                             |

In **systemd**, the command to switch between these targets (formerly runlevels) is `systemctl isolate`. For example, to switch to a graphical mode (similar to Runlevel 5 in SysV):

```bash
systemctl isolate graphical.target
```

Or, to power off the system (equivalent to Runlevel 0 in SysV):

```bash
systemctl isolate poweroff.target
```

### How to Check Runlevels (SysV)

In SysV-based systems, you can check the current runlevel using the `runlevel` command:

```bash
runlevel
```

This will output two values:
- The previous runlevel.
- The current runlevel.

Example:
```bash
N 5
```
In this example, the system has moved from no previous runlevel (`N`) to runlevel 5.

### How to Change Runlevels (SysV)

To change runlevels in a SysV init system, use the `init` command followed by the desired runlevel number. For example, to change to runlevel 3 (multi-user, no GUI):

```bash
init 3
```

Or, to shut down the system (runlevel 0):

```bash
init 0
```

### Conclusion

- **Runlevels** in traditional SysV init systems define various states of system operation (shutdown, single-user, multi-user, graphical, reboot, etc.).
- SysV runlevels are managed via directories and scripts under `/etc/rc.d/`, with symbolic links controlling which services start or stop at each level.
- Modern Linux systems using **systemd** have replaced runlevels with **targets**, providing more flexibility and control over system states.
