# Docker-Volumes-Final-Recap-Deep-Dive

### 🗂️ 1. **Types of Volumes Recap**

| Type                 | Created How                          | Managed by Docker | Visible with `docker volume ls` | Persistent                    |
| -------------------- | ------------------------------------ | ----------------- | ------------------------------- | ----------------------------- |
| **Anonymous Volume** | `-v /app/temp` or `VOLUME /app/temp` | ✅ Yes             | ✅ Yes (cryptic name)            | ❌ No (removed with container) |
| **Named Volume**     | `-v feedback:/app/feedback`          | ✅ Yes             | ✅ Yes (named)                   | ✅ Yes                         |
| **Bind Mount**       | `-v $(pwd):/app:ro`                  | ❌ No              | ❌ No                            | ✅ Yes (lives in host folder)  |

---

### 🔍 2. **How to View & Inspect Volumes**

* List all Docker-managed volumes:

  ```bash
  docker volume ls
  ```

* Inspect a specific volume:

  ```bash
  docker volume inspect feedback
  ```

  You'll see:

  * Creation date
  * Mount path in the Docker internal filesystem
  * If it's read-only (would show in `Options`)
  * Driver info (not important in most dev scenarios)

> 📌 **Note**: The mount path shown is inside Docker’s internal Linux VM (especially on Windows/macOS). You can't directly browse it like a normal folder.

---

### 🛠️ 3. **Creating Volumes Manually (Optional)**

You *can* create named volumes manually:

```bash
docker volume create my-data
```

But you **don’t have to** — Docker automatically creates it if you reference a non-existent name in `-v`.

---

### 🧹 4. **Removing Volumes**

#### ❌ Remove a specific volume:

```bash
docker volume rm volume_name
```

> 🛑 Will fail if a container is still using it. Stop and remove the container first.

#### 🔥 Prune (Delete All Unused Volumes):

```bash
docker volume prune
```

> This deletes **all unused** Docker-managed volumes — useful to clean up dangling volumes and save space.

#### 🧼 Anonymous volumes auto-delete

If the container is run with `--rm`, its **anonymous volumes** are also deleted when the container stops.

---

### 🧠 5. **The Override Rule**

Docker applies this **override logic** for volumes:

> **More specific path wins**

That means:

* If you mount `/app:ro` (read-only bind mount)
* And also mount `/app/feedback` (named, writable)
* Then **`/app/feedback` stays writable**, even though `/app` is read-only

This rule allows you to:

* Protect most of your app from writing (like source code)
* Still allow writing to feedback, temp, and node\_modules folders

---

### 🚀 Summary of Your Final Docker Run Setup

Here’s a final, full `docker run` example based on everything you’ve learned:

```bash
docker run -d --rm \
  -p 3000:80 \
  --name feedback-app \
  -v feedback:/app/feedback \            # Writable named volume
  -v /app/temp \                         # Writable anonymous volume
  -v /app/node_modules \                 # Preserve dependencies
  -v "$(pwd)":/app:ro \                  # Read-only bind mount of local code
  feedback-app
```

---

### 💡 Final Tips

* Use `named volumes` to persist user data across restarts
* Use `anonymous volumes` to prevent overwriting internal folders (like `node_modules`)
* Use `bind mounts` during development to auto-sync code changes
* Use `:ro` to prevent accidental file edits from inside the container
* Use `docker volume ls`, `inspect`, `rm`, and `prune` to manage volumes
* **Don’t forget**: `--rm` deletes anonymous volumes automatically
