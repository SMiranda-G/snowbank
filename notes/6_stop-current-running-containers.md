The error has **nothing to do** with old images or containers. 

Your `docker-compose.yml` is looking for a file named **`dockerfile-airflow.dockerfile`** to build the Airflow container, but that file does not exist in your current folder (or is named differently). 

---

### To answer your question: "deactivate" images without deleting them

**Images are not "active" or "inactive"** — they are just files stored on your disk. They don't run or consume resources unless a container is created from them.

What you actually have are **running containers**. To **stop** them (without deleting them):

```bash
# Stop ALL running containers
docker stop $(docker ps -q)
```

Or if you want to stop containers from a specific previous project folder, navigate to that folder and run:
```bash
docker compose down
```
*(Note: `down` deletes containers but keeps the images intact)*

---

### The real fix for your current error

1. **Check your Dockerfile name**: 
   - Look in your current folder. Do you have a Dockerfile? Is it named exactly `dockerfile-airflow.dockerfile`?
   - If you have a file named just `Dockerfile` or `Dockerfile.airflow`, update the `docker-compose.yml` to match the correct filename under the `build` section.

2. **Temporary workaround** (if you just want to test):
   - If the file is named something else, rename it:
     ```bash
     mv your-actual-dockerfile-name dockerfile-airflow.dockerfile
     ```

Once the Dockerfile path is correct, `docker compose up -d` will work.