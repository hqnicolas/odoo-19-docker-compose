### **File:** `run.sh`

#### **Purpose**
This script automates the setup and launch of an **Odoo 19 Docker environment** by cloning a predefined Odoo Docker Compose repository, adjusting system and file permissions, configuring ports, and starting Odoo through Docker Compose.

---

### **Step-by-Step Breakdown**

#### 1. **Argument Handling**
```bash
DESTINATION=$1
PORT=$2
CHAT=$3
```
- The script expects three command-line arguments:
  1. `DESTINATION`: Directory path where the Odoo setup will be created.
  2. `PORT`: The main Odoo web service port.
  3. `CHAT`: The live chat service port.

---

#### 2. **Clone the Odoo Docker Repository**
```bash
git clone --depth=1 https://github.com/minhng92/odoo-19-docker-compose $DESTINATION
rm -rf $DESTINATION/.git
```
- Clones the Odoo 19 Docker Compose setup into the destination directory.
- Removes the `.git` folder to detach from version control, leaving only the necessary runtime files.

---

#### 3. **Create the PostgreSQL Data Directory**
```bash
mkdir -p $DESTINATION/postgresql
```
- Ensures that a directory exists for PostgreSQL data persistence.

---

#### 4. **Set Ownership and Secure Permissions**
```bash
sudo chown -R $USER:$USER $DESTINATION
sudo chmod -R 700 $DESTINATION
```
- Assigns ownership of all files to the current user.
- Applies restrictive permissions (`700`) to ensure that only the user can access the setup, improving security.

---

#### 5. **Platform Detection (macOS vs Linux)**
```bash
if [[ "$OSTYPE" == "darwin"* ]]; then
  echo "Running on macOS. Skipping inotify configuration."
else
  # System configuration ...
fi
```
- Detects if the script is running on macOS.
- If so, it skips Linux-specific system configurations (`inotify` tuning).

---

#### 6. **Linux System Configuration (inotify Watches)**
```bash
if grep -qF "fs.inotify.max_user_watches" /etc/sysctl.conf; then
  echo $(grep -F "fs.inotify.max_user_watches" /etc/sysctl.conf)
else
  echo "fs.inotify.max_user_watches = 524288" | sudo tee -a /etc/sysctl.conf
fi
sudo sysctl -p
```
- For Linux systems, increases the maximum number of file watches (`fs.inotify.max_user_watches`) to 524,288.
- This is essential for Odoo’s file change detection and live reloading.
- Applies the changes immediately using `sysctl -p`.

---

#### 7. **Update Docker Compose Ports**
```bash
sed -i 's/10019/'$PORT'/g' $DESTINATION/docker-compose.yml
sed -i 's/20019/'$CHAT'/g' $DESTINATION/docker-compose.yml
```
- Updates the `docker-compose.yml` file with user-specified ports.
- Uses `sed` to replace default port numbers (`10019` for Odoo, `20019` for chat).
- Uses macOS-compatible syntax (`sed -i ''`) if running on macOS.

---

#### 8. **Adjust File and Directory Permissions**
```bash
find $DESTINATION -type f -exec chmod 644 {} \;
find $DESTINATION -type d -exec chmod 755 {} \;
```
- Sets standard permissions after setup:
  - Files: readable/writable by the owner, readable by others.
  - Directories: executable and readable by everyone (allows navigation).

---

#### 9. **Make Entrypoint Executable**
```bash
chmod +x $DESTINATION/entrypoint.sh
```
- Ensures that the `entrypoint.sh` script has execution permissions so Docker can use it as the container entrypoint.

---

#### 10. **Start the Odoo Docker Containers**
```bash
if ! is_present="$(type -p "docker-compose")" || [[ -z $is_present ]]; then
  docker compose -f $DESTINATION/docker-compose.yml up -d
else
  docker-compose -f $DESTINATION/docker-compose.yml up -d
fi
```
- Checks whether `docker-compose` is installed.
- If not, falls back to using the newer `docker compose` command.
- Starts the Odoo and PostgreSQL services in detached mode (`-d`).

---

#### 11. **Completion Message**
```bash
echo "Odoo started at http://localhost:$PORT | Master Password: minhng.info | Live chat port: $CHAT"
```
- Prints a summary with:
  - The Odoo URL and port.
  - The default master password (`minhng.info`).
  - The live chat port.

---

### **Summary**
The `run.sh` script provides a **fully automated setup** for deploying Odoo 19 in Docker.  
It handles everything from cloning the repository and configuring ports to securing permissions and starting the services, requiring minimal user input.
```