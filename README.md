# InnGo App

<p align="center">
  <img src="logo.png" alt="InnGo" width="280">
</p>

> **InnGo App** is the hotel-side Property Management System (PMS) of the InnGo platform.
> It is designed to run locally inside the hotel and provides day-to-day hotel operations.

---

# Current Status

**Current Version:** Development Preview

The current release is built and tested on:

- Raspberry Pi 3
- ARM64 Architecture (aarch64)
- Raspberry Pi OS Lite (64-bit)

The application currently runs as a standalone binary and uses a `.env` file for configuration.

> **Note**
>
> There is currently **no setup/install wizard**.
> Configuration is performed manually through the `.env` file.
> A complete setup script and installer will be added in future releases.

---

# Current Modules

The current version includes:

- Front Office
- Administration

Additional modules will be added in upcoming releases.

---

# Requirements

- Raspberry Pi 3 (ARM64)
- Raspberry Pi OS Lite (64-bit)
- MariaDB Server
- Nginx

---

# Default Installation Directory

The recommended installation directory for linux is:

```text
/opt/inngo/
```

After setting up the application, your deployment directory `/opt/inngo/` (or `C:\inngo\` on Windows) should look like this:

```text
/opt/inngo/
├── dist/                # Frontend production build static files
├── inngo-server-arm64   # InnGo App (inngo-server-amd64 for standard Linux / .exe for Windows)
├── .env                 # Environment configurations and database keys
└── storage/             # Application storage (Automatically created on startup)
    ├── backups/         # Database automatic and manual backups
    └── temp/            # Temporary files used by the system
```

---

# Installing MariaDB

Update packages:

```bash
sudo apt update
sudo apt upgrade -y
```

Install MariaDB:

```bash
sudo apt install mariadb-server -y
```

Enable the service:

```bash
sudo systemctl enable mariadb
sudo systemctl start mariadb
```

Verify status:

```bash
sudo systemctl status mariadb
```

Run secure installation:

```bash
sudo mysql_secure_installation
```

Create the database:

```sql
CREATE DATABASE inngo_app CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER 'inngo'@'localhost' IDENTIFIED BY 'your_password';

GRANT ALL PRIVILEGES ON inngo_app.* TO 'inngo'@'localhost';

FLUSH PRIVILEGES;
```

---

# Installing Nginx

Install:

```bash
sudo apt install nginx -y
```

Enable:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

# Nginx Reverse Proxy

InnGo App listens on:

```
127.0.0.1:8080
```

Nginx exposes the application on **port 80**, allowing users to access it using only the server IP address without specifying `:8080`.

Example:

```
http://192.168.1.100
```

instead of

```
http://192.168.1.100:8080
```

Example Nginx configuration:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8080;

        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

# Environment Configuration

The application requires a `.env` file located in the same directory as the executable binary to manage configuration variables and database connections.

### ⚙️ How to Create the `.env` File

Follow the steps below according to your Operating System to create and populate the configuration file:

### 🐧 For Linux & Raspberry Pi
1. Navigate to your installation directory:
   ```bash
   cd /opt/inngo
   ```
2. Create and open a new .env file using the nano editor:
   ```bash
   sudo nano .env
   ```
3. Paste the configuration template below, update your credentials, then save and exit
   ```bash
   (Ctrl+O, Enter, then Ctrl+X).
   ```

### 🪟 For Windows Server
1. Navigate to your installation folder (e.g., C:\inngo\).
2. Right-click inside the folder, select New -> Text Document.
3. Name the file .env (make sure to remove the .txt extension).
4. Open the file with Notepad, paste the template below, update your credentials, and save the file.

### 📝 Configuration Template
Copy this sample configuration and update the values to match your environment:

```env
APP_ENV=production
GIN_MODE=release

APP_NAME=InnGo
APP_PORT=8080

DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=inngo_app
DB_USER=inngo
DB_PASSWORD=your_password

JWT_SECRET=change_this_secret
JWT_EXPIRES_IN=24h
```

---

# Running the Application

First, create your installation directory and move the downloaded binary and your `.env` file into it. Follow the specific instructions for your Operating System below:

### 🐧 For Linux & Raspberry Pi (ARM64 / AMD64)

1. Create the installation directory and copy the files:
   ```bash
   sudo mkdir -p /opt/inngo
   sudo cp inngo-server-arm64 /opt/inngo/    # Or inngo-server-amd64 depending on your CPU
   sudo cp .env /opt/inngo/
   cd /opt/inngo
   ```
   
2. Make the binary executable:
   ```bash
   # For Raspberry Pi 3:
   chmod +x inngo-server-arm64
   
   # For Linux Standard Server:
   chmod +x inngo-server-amd64
   ```

3. For testing or development, you can run the application manually:
    ```bash
    # For Raspberry Pi 3:
    ./inngo-server-arm64
    
    # For Linux Standard Server:
    ./inngo-server-amd64
    ```

For Windows Server (AMD64)
Create an installation folder (e.g., C:\inngo\).

Move the inngo-server-windows.exe binary and your .env file into that folder.

Open PowerShell or Command Prompt (CMD) as Administrator, and navigate to your directory:
  ```bash
  cd C:\inngo\
  ```

Run the application manually for testing:
  ```bash
  .\inngo-server-windows.exe
  ```

Note: For production environments, it is highly recommended to configure the application to run as a system service (using systemd on Linux/Pi or NSSM on Windows Server) so it stays running continuously in the background.

---

# Configure as a System Service for Linux & Raspberry Pi

For production deployments, it is recommended to run InnGo App as a **systemd service** so it starts automatically when the Raspberry Pi boots and restarts automatically if the application exits unexpectedly.

Create a new service file:

  ```bash
  sudo nano /etc/systemd/system/inngo.service
  ```

Paste the following configuration:

  ```ini
  [Unit]
  Description=InnGo Management System Service
  After=network.target mariadb.service
  
  [Service]
  Type=simple
  WorkingDirectory=/opt/inngo
  
  # For Raspberry Pi 3 (ARM64), use:
  ExecStart=/opt/inngo/inngo-server-arm64
  
  # For Standard Linux Server (AMD64), uncomment the line below and comment the one above:
  # ExecStart=/opt/inngo/inngo-server-amd64
  
  Restart=always
  RestartSec=5
  User=root
  
  # Environment variables can also be explicitly loaded if needed
  Environment=NODE_ENV=production
  
  [Install]
  WantedBy=multi-user.target
  ```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable automatic startup:

```bash
sudo systemctl enable inngo
```

Start the service:

```bash
sudo systemctl start inngo
```

Check the service status:

```bash
sudo systemctl status inngo
```

View application logs:

```bash
journalctl -u inngo -f
```

Restart the service:

```bash
sudo systemctl restart inngo
```

Stop the service:

```bash
sudo systemctl stop inngo
```

After enabling the service, InnGo App will automatically start whenever the Raspberry Pi or Linux Server boots.

---

# 🔄 Configure as a Background Service (Windows Server)

On Windows Server, you can run the application as a background Windows Service so it starts automatically on system boot, even if no user is logged in. 

We will use **NSSM** (Non-Sucking Service Manager), a trusted and lightweight tool, to achieve this:

1. **Download NSSM:**
   * Download the latest release from [nssm.cc](https://nssm.cc/download).
   * Extract the zip file and copy the `nssm.exe` (from the `win64` folder) to your installation directory (e.g., `C:\inngo\`).

2. **Install the Service via PowerShell:**
   Open **PowerShell as Administrator** and run the following command to install the service:
   ```powershell
   cd C:\inngo\
   .\nssm.exe install InnGoService "C:\inngo\inngo-server-windows.exe"
   ```

# System Architecture

```text
                 Client
                    │
                    ▼
           http://SERVER_IP
                    │
                    ▼
             Nginx (Port 80)
                    │
                    ▼
        InnGo App (Systemd Service)
          127.0.0.1:8080
                    │
                    ▼
                MariaDB
```

---

# Notes

- Current build targets **Raspberry Pi 3 ARM64**.
- The application is intended to be installed under **`/opt/inngo`**.
- Configuration is currently performed through the `.env` file.
- Database migrations run automatically on startup.
- In production, InnGo App should run as a **systemd service**.
- A complete setup script and installer will be introduced in a future release.

---

# Roadmap

Upcoming features include:

- Housekeeping
- Reservations
- Accounting
- Inventory
- POS
- Reports
- Setup Wizard
- Automatic Updates

---

# License

Copyright © InnGo.

All rights reserved.
