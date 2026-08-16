### `docker compose -f docker-compose.yml up -d`

- **`docker compose`** → The CLI tool to compose file.*
- **`-f docker-compose.yml`** → Specifies which file to use.
- **`up`** → **Builds (if needed)** + **Creates** + **Starts** the containers. *(Not just "create" – it runs them immediately.)*
- **`-d`** → **Detach** mode – runs containers in the **background**, freeing up your terminal.

### The Opposite
- **`docker compose down`** → **Stops** AND **deletes** the containers (and networks).

---

## "Pulled" means:
- Docker downloaded the pre-built **blueprints** (images) for those services from a registry (like Docker Hub) to our local computer.
- They are **not running yet**—they are just stored locally on our disk, ready to be used.

**The difference:**
- **Image** = The "recipe" / executable file (static, stored on disk).
- **Container** = A running instance of that image (the actual process).

![pulling-docker-images.png](.\pulling-docker-images.png)