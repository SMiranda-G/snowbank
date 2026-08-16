
## 🐳 Container-Level Commands (single container)

| Command | Simple Definition |
| :--- | :--- |
| `docker ps` | Show only **running** containers (active). |
| `docker ps -a` | Show **all** containers (including stopped). |
| `docker stop <name/id>` | **Stop** a running container gracefully (like pressing "shutdown"). |
| `docker kill <name/id>` | **Force stop** a container immediately (like pulling the plug). |
| `docker pause <name/id>` | **Freeze** a container's processes without stopping them (keeps memory). |
| `docker start <name/id>` | **Start** a stopped container (reactivate it). |
| `docker stop $(docker ps -q)` | Stop **all** currently running containers at once. |
| `docker logs -f <name/id>` | Show **live logs** streaming from a container (like tail -f). |

---

## 🏗️ Docker Compose Commands (whole project)

| Command | Simple Definition |
| :--- | :--- |
| `docker compose up` | Build (if needed) + Create + Start **all** services in the compose file (attached to terminal). |
| `docker compose up -d` | Same as above, but runs **in the background** (detached mode). |
| `docker compose up --build` | Force rebuild images before starting (use when you changed code). |
| `docker compose up --force-recreate` | Force create **new** containers even if old ones exist (wipes old ones). |
| `docker compose down` | Stop **and delete** all containers, networks, and default volumes (clean slate). |
| `docker compose start` | Start **existing** stopped containers for this project (does NOT create new ones). |
| `docker compose start <service>` | Start **only one service** by its name (e.g., `docker compose start postgres`). |
| `docker compose logs -f` | Show **live logs** for all services in the project. |

---

## Key Difference to Remember

| Level | Command | What it affects |
| :--- | :--- | :--- |
| **Container-level** | `docker start/stop ...` | Single container (you need its exact name/ID). |
| **Compose-level** | `docker compose up/down ...` | All containers defined in `docker-compose.yml` (project-aware). |

