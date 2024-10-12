# SystemD Service

## 1. What is a SystemD Service?

A **SystemD service** is a unit of work managed by the **SystemD init system**, commonly found in most modern Linux distributions. Services could include anything from background daemons (like web servers) to system-level tasks (like network management). Each SystemD service is defined by a **unit file** (usually with a `.service` extension), which contains detailed instructions on how to manage that service, such as starting, stopping, or restarting it.

### Example:  
A web server like **Apache** can be managed as a SystemD service using the command:
```bash
systemctl start apache2.service
```
Here, `apache2.service` is the name of the unit file that defines how the Apache service behaves.

## 2. Why Use SystemD Services?

SystemD provides many advantages over older init systems like SysV init. Here are the key reasons to use SystemD:

- **Parallelization**:  
  Unlike older init systems, SystemD can start multiple services at the same time, which significantly reduces the boot time of the system.  
  Example: SystemD can start network services, file systems, and background daemons concurrently rather than waiting for one to finish before starting another.
  
- **Dependency Management**:  
  SystemD understands which services depend on others and ensures that services start in the correct order.  
  Example: A database service (like MySQL) will only start after the filesystem and network services it depends on are fully running.

- **Logging**:  
  SystemD integrates logging through **journald**, capturing logs for all services in a central place, making it easier to troubleshoot issues.  
  Example: You can review the logs for a service like NGINX using:
  ```bash
  journalctl -u nginx.service
  ```

- **Resource Management**:  
  SystemD allows setting CPU, memory, and other resource limits to control how much system resources a service can use, improving system stability.  
  Example: You can configure a service to use no more than 50% of the CPU or limit memory usage to avoid overloading the system.

- **Service Recovery**:  
  If a service crashes, SystemD can automatically attempt to restart it, increasing system reliability.  
  Example: If a web server crashes, SystemD can be set to automatically restart it, reducing downtime.

- **Standardization**:  
  Since SystemD is widely adopted across most Linux distributions, the way services are managed is standardized, providing consistency across different systems.

## 3. How to Create a SystemD Service Recipe?

To create a SystemD service, you need to define its behavior in a **.service unit file** and then place this file in the correct location in your system. Here's how to do it:

1. **Create the service script or executable**:  
   Write the code or script that performs the task you want to run as a service. For example, a Python script that acts as a web server.
   
2. **Create the `.service` unit file**:  
   This file defines how SystemD will manage your service (start, stop, restart, etc.). A basic `.service` file looks like this:
   ```ini
   [Unit]
   Description=My Custom Web Service
   After=network.target
   
   [Service]
   ExecStart=/usr/bin/python3 /path/to/web_server.py
   Restart=on-failure
   
   [Install]
   WantedBy=multi-user.target
   ```

3. **Place the files in the correct location**:  
   The `.service` file should be placed in the correct location, typically `/etc/systemd/system/` or `${systemd_system_unitdir}/system` in Yocto-based recipes.

4. **Use the `do_install` task in Yocto**:  
   In Yocto, you need to use the `do_install` task to ensure the `.service` file and any related scripts are copied to the correct directory during the installation process. Example:
   ```bash
   do_install() {
       install -m 0644 my_service.service ${D}${systemd_system_unitdir}/system/
   }
   ```

5. **Define dependencies and metadata**:  
   In the Yocto recipe, define any other dependencies (like libraries or network services) and metadata required by the service.

## 4. Where to Place the Service File in the Root Filesystem?

The `.service` file should be placed in the appropriate directory, which follows the **Filesystem Hierarchy Standard (FHS)**. For SystemD services, this is usually:
```
/etc/systemd/system/
```
Or, in a Yocto-based system, use the `${systemd_system_unitdir}/system` variable to place it in the correct directory.

## 5. How to Enable the Service by Default?

If you want the service to start automatically when the system boots, you need to enable it. In Yocto recipes, this is done by setting the following parameters:
```bash
SYSTEMD_AUTO_ENABLE = "enable"
SYSTEMD_SERVICE:${PN} = "my_service.service"
```
This ensures that the service is enabled and will run on boot without needing to manually enable it after installation.

## 6. Some Basic SystemD Commands

Here are some common SystemD commands to manage services:

- **Check status**:  
  ```bash
  systemctl status my_service.service
  ```
  Example: View the status of the web server service.

- **Start a service**:  
  ```bash
  systemctl start my_service.service
  ```
  Example: Start a web server like NGINX manually.

- **Stop a service**:  
  ```bash
  systemctl stop my_service.service
  ```
  Example: Stop the NGINX web server.

- **Restart a service**:  
  ```bash
  systemctl restart my_service.service
  ```
  Example: Restart a crashed service.

- **Enable a service to start at boot**:  
  ```bash
  systemctl enable my_service.service
  ```
  Example: Make sure the service starts automatically when the system boots.

- **Disable a service from starting at boot**:  
  ```bash
  systemctl disable my_service.service
  ```
  Example: Prevent a service from starting automatically.

- **Reload SystemD to apply changes**:  
  After modifying or adding new `.service` files, run:
  ```bash
  systemctl daemon-reload
  ```
  This reloads the SystemD manager configuration.

---

## 7. Check the following Link:
* [Systemd service](https://github.com/Munawar-git/YoctoTutorials/blob/master/25_SystemD_Service_Recipe/25_SystemD_Service_Recipe.md)

## 8. Practical example with Yocto Project:

#### 1. **Create a Python Web Server Script**

First, create a simple Python HTTP server script. This script will run a web server that serves files from a directory.

```python
#!/usr/bin/env python3

from http.server import SimpleHTTPRequestHandler
from socketserver import TCPServer

PORT = 8080

Handler = SimpleHTTPRequestHandler

with TCPServer(("", PORT), Handler) as httpd:
    print(f"Serving at port {PORT}")
    httpd.serve_forever()
```

Save this script as `web_server.py`. This script listens on port 8080 and serves the current directory.

#### 2. **Create a SystemD Unit File**

Next, create a `.service` unit file for the Python web server. The unit file will tell SystemD how to start, stop, and manage the service.

```ini
[Unit]
Description=Simple Python Web Server
After=network.target

[Service]
ExecStart=/usr/bin/python3 /usr/local/bin/web_server.py
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

- **ExecStart**: Specifies the command to start the Python web server.
- **Restart=on-failure**: Ensures the service will restart automatically if it crashes.
- **After=network.target**: Ensures the service starts only after the network is up.

Save this file as `web_server.service`.

#### 3. **Create a Yocto Recipe to Install the Service**

Now, we need to create a **Yocto recipe** to package this Python web server script and the SystemD unit file. The recipe will handle installing these files into the appropriate locations during the build process.

Create a file called `simple-web-server.bb` inside your Yocto layer’s recipe directory (e.g., `recipes-example/simple-web-server/`):

```bitbake
SUMMARY = "Simple Python Web Server"
DESCRIPTION = "A basic HTTP server using Python"
LICENSE = "MIT"
SRC_URI = "file://web_server.py \
           file://web_server.service"

inherit systemd

S = "${WORKDIR}"

# Install the Python script and systemd service file
do_install() {
    install -d ${D}${bindir}
    install -m 0755 ${WORKDIR}/web_server.py ${D}${bindir}/web_server.py

    install -d ${D}${systemd_system_unitdir}
    install -m 0644 ${WORKDIR}/web_server.service ${D}${systemd_system_unitdir}/web_server.service
}

# Ensure the service is enabled and starts at boot
SYSTEMD_SERVICE:${PN} = "web_server.service"
SYSTEMD_AUTO_ENABLE = "enable"
```

### Breakdown of Recipe:
- **SRC_URI**: Points to the Python script (`web_server.py`) and the SystemD service file (`web_server.service`) that should be included in the build.
- **inherit systemd**: Tells Yocto that this recipe needs to manage SystemD services.
- **do_install()**: This task installs the Python script to `/usr/local/bin/` and the `.service` file to the SystemD directory (usually `/lib/systemd/system/` or `/etc/systemd/system/`).
- **SYSTEMD_SERVICE:${PN}**: Defines which service file should be managed by SystemD for this recipe.
- **SYSTEMD_AUTO_ENABLE = "enable"**: Automatically enables the service, ensuring it starts on boot.

#### 4. **Add the Recipe to Your Image**

Once the recipe is ready, you need to add it to your Yocto image. In your image recipe file (e.g., `core-image.bb` or `my-custom-image.bb`), add the following line:

```bitbake
IMAGE_INSTALL += "simple-web-server"
```

This ensures that Yocto builds the Python web server and includes it in the final image.

#### 5. **Build the Image**

Now, you can build the image with the Python web server service:

```bash
bitbake my-custom-image
```

Yocto will build your image, including the Python web server and SystemD service.

#### 6. **Deploy and Test**

After flashing the image to your target device (like a Raspberry Pi or any embedded system), the Python web server should automatically start on boot.

You can manage the service using the following commands:

- **Check if the service is running**:
  ```bash
  systemctl status web_server.service
  ```

- **Start the service manually**:
  ```bash
  systemctl start web_server.service
  ```

- **Stop the service**:
  ```bash
  systemctl stop web_server.service
  ```

- **Restart the service**:
  ```bash
  systemctl restart web_server.service
  ```

- **Enable the service to start at boot**:
  ```bash
  systemctl enable web_server.service
  ```

- **Disable the service from starting at boot**:
  ```bash
  systemctl disable web_server.service
  ```

#### 7. **Access the Web Server**

Once the service is running, you can access the web server by opening a browser and navigating to the IP address of your target device at port 8080:

```
http://<device-ip>:8080
```

You should see a list of files being served from the directory where the `web_server.py` script is running.
