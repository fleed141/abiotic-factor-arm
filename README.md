# Abiotic Factor Dedicated Server (ARM64)

This repository provides a containerized environment to deploy an **Abiotic Factor** dedicated server, optimized for ARM architectures using Wine. 

The deployment consists of two containers: a micro-service that automatically configures local folder permissions, and the main game server running from a custom-built image.

##  Features

* **Custom Wine Environment:** Uses a custom `Dockerfile` to configure Windows SteamCMD and Wine with proper user permissions (`steam`) to prevent Box64 errors.
* **Automatic Permissions Fix:** A preliminary container (`fix-permissions`) ensures that all mapped folders have the correct owner (UID `1001`) before the server starts.
* **Wine Optimizations:** `esync` and `fsync` are enabled to improve server performance.
* **Automated Backups:** Automatically creates and rotates world backups every time the server starts.
* **Built-in Updates:** Support for updating the game via SteamCMD on every restart.
* **SELinux Support:** Volumes use `:z` and `:Z` flags, ideal for environments using Podman or Linux systems with strict security policies.

---

##  Prerequisites

* Docker and Docker Compose (or Podman Compose).
* Open ports on your firewall/router:
  * `7777` (UDP) - Game traffic.
  * `27015` (UDP) - Server Query.

---

##  Installation and Usage

1. **Clone this repository** (or download the project files).
2. **Review the configuration:** Open the `docker-compose.yml` file and modify the environment variables in the `environment` section (make sure to change the `SERVER_PASSWORD`).
3. **Build and start the server:** Because this setup relies on a custom `Dockerfile` to properly set up the Wine and SteamCMD environment, **you must build the container image** before running it. Use the `--build` flag:

```bash
docker compose up -d --build
```
*(If you are using Podman, use `podman compose up -d --build`)*

**Alternative (Manual Build):**
If you prefer, you can build the image manually first and then start the containers:
```bash
docker compose build
docker compose up -d
```

The `fix-permissions` service will run first, adjusting the permissions for the `./steamapps`, `./steam`, and `./backups` directories, and then exit. Immediately after, the `abiotic-factor` container will begin its deployment.

---

##  Configuration (Environment Variables)

You can customize the server's behavior by editing the variables inside the `docker-compose.yml` file:

### Server Configuration

| Variable | Description | Default Value |
|---|---|---|
| `SERVER_NAME` | The public name of your server in the server browser. | `MyAbioticoReaction` |
| `SERVER_PASSWORD` | Password required to join the server. **Change this!** | `locuron123` |
| `MAX_PLAYERS` | Maximum number of allowed players. | `6` |
| `WORLD_SAVE_NAME` | Name of your world/save file. | `Cascade` |
| `TZ` | Server timezone. | `America/Asuncion` |

### Behavior and Maintenance

| Variable | Description | Default Value |
|---|---|---|
| `UPDATE_ON_START` | Set to `1` to check for SteamCMD updates on startup. | `1` |
| `VALIDATE` | Set to `1` to verify game file integrity (useful if you suspect corruption). | `0` |
| `BACKUP_ON_START` | Set to `1` to create a world backup before starting. | `1` |
| `BACKUP_KEEP` | Maximum number of old backups to keep before rotation. | `5` |

### System Environment (Advanced)

| Variable | Description | Default Value |
|---|---|---|
| `ARM64_DEVICE` | Target ARM device or profile used. | `adlink` |
| `WINEDEBUG` | Wine logging level (recommended to keep disabled). | `-all` |
| `WINEESYNC` / `WINEFSYNC` | Synchronization optimizers for Wine. | `1` |

---

##  Volume Structure

The Compose file will generate and map the following directories in the same folder where it is executed:

* `./steamapps`: Stores the game files downloaded by SteamCMD.
* `./steam`: Stores Steam client data and configurations.
* `./backups`: Folder where your world backups are saved.

---

##  Stopping the Server

To safely shut down the server and its processes:

```bash
docker compose down
```

