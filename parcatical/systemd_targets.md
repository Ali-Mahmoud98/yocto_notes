# Systemd targets
In **systemd**, targets are a way of grouping and managing services and other system units together. Targets define a specific state that the system should reach, such as multi-user mode, graphical mode, or system shutdown. They replace the concept of "runlevels" from the older SysV init system.

Each target is essentially a group of units (services, devices, sockets, mounts, etc.) that need to be started or stopped together to bring the system into a particular state.

### Common SystemD Targets

Here are some of the most commonly used **systemd targets** and their purposes:

| **Target**               | **Description**                                                                                     | **Equivalent SysV Runlevel** |
|--------------------------|-----------------------------------------------------------------------------------------------------|------------------------------|
| **`default.target`**      | The default system state after booting. This is a symbolic link to another target, usually `graphical.target` or `multi-user.target`. | Varies (based on configuration) |
| **`graphical.target`**    | Boots the system into multi-user mode with a graphical user interface (GUI). This is typically used on desktop systems. | Runlevel 5 (with GUI)         |
| **`multi-user.target`**   | Boots the system into multi-user mode, but without a graphical interface. Used for servers and systems without a GUI. | Runlevel 3 (without GUI)      |
| **`rescue.target`**       | A minimal environment for system recovery, with only basic services running and a single user shell. | Runlevel 1 (single-user mode) |
| **`emergency.target`**    | Boots the system into an emergency shell. This mode has very few services running and is used for critical system recovery. | N/A (like a more limited Runlevel 1) |
| **`poweroff.target`**     | Shuts down the system.                                                                              | Runlevel 0                    |
| **`reboot.target`**       | Reboots the system.                                                                                 | Runlevel 6                    |
| **`halt.target`**         | Halts the system without powering it off.                                                           | Runlevel 0 (halt)             |
| **`suspend.target`**      | Suspends the system to RAM (low-power state, resume quickly).                                        | N/A                           |
| **`hibernate.target`**    | Hibernates the system, saving the state to disk and powering off.                                   | N/A                           |
| **`hybrid-sleep.target`** | Combines both suspend to RAM and hibernation. The system will resume from RAM if power remains, or from disk if power is lost. | N/A                           |
| **`shutdown.target`**     | Prepares the system for shutting down. Usually triggers `poweroff.target`.                          | N/A                           |

### How Targets Work

Targets are defined by `.target` unit files. For example, the `multi-user.target` unit file might look like this:

```ini
[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Wants=network.target
Conflicts=rescue.target
After=basic.target rescue.target
```

- **Requires**: Lists units that must be started for the target to be reached.
- **Wants**: Lists units that should be started if possible but are not critical.
- **After**: Specifies the order in which units should be started.
- **Conflicts**: Specifies units that should not run together with this target.

### Switching Between Targets

You can use the `systemctl` command to change the system state by switching to a different target:

- **Change to graphical mode** (with GUI):
  ```bash
  systemctl isolate graphical.target
  ```

- **Change to multi-user mode** (no GUI):
  ```bash
  systemctl isolate multi-user.target
  ```

- **Reboot the system**:
  ```bash
  systemctl isolate reboot.target
  ```

- **Power off the system**:
  ```bash
  systemctl isolate poweroff.target
  ```

### Setting the Default Target

You can set the system’s default target, which defines the state it will boot into by default. For example, if you want your system to boot into **multi-user mode** instead of graphical mode:

1. Set the default target:
   ```bash
   systemctl set-default multi-user.target
   ```

2. Verify the default target:
   ```bash
   systemctl get-default
   ```

### Examples of Custom Targets

SystemD also allows you to define custom targets for more specialized use cases. For example, you could create a custom target to start specific services needed for a special environment, like a maintenance mode:

1. Create a custom target file (e.g., `maintenance.target`):
   ```ini
   [Unit]
   Description=Maintenance Mode
   Requires=basic.target
   After=basic.target
   ```

2. Enable the target and switch to it when needed:
   ```bash
   systemctl isolate maintenance.target
   ```

---

- **SystemD targets** are like "runlevels" in older systems, defining system states.
- Common targets include `multi-user.target`, `graphical.target`, `reboot.target`, etc.
- Targets group services and other units to manage the system state, such as whether the system is in a graphical environment or just a basic multi-user mode.
- You can easily switch between targets using `systemctl` and define custom targets for specific needs.
- The command `systemctl isolate` is used in systemd to change the current system state by isolating a particular target. Essentially, it stops all the units (services, devices, etc.) that are not required by the target you are isolating, and starts only the units that are required by that target. This allows you to move the system to a new state or mode while stopping all other services not needed in that mode.

### Important Note:
When you use `systemctl isolate`, it immediately stops any units not needed by the target you're isolating to. Therefore, it's important to be careful when isolating to targets like `poweroff.target` or `reboot.target`, as this will shut down or reboot the system without warning.

### Conclusion:
The `isolate` command is a powerful way to switch between different system states or modes, like graphical and non-graphical modes, without rebooting. It ensures that only the necessary services for the target you are isolating are running, stopping all others.
