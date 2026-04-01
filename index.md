---
layout: default
---

# 🐳 Switching Unraid Docker Storage from vDisk to Directory

> **⏱️ Time required:** ~30 minutes &nbsp;|&nbsp; **📊 Difficulty:** Easy &nbsp;|&nbsp; **⚠️ Risk:** Low

---

## 📖 What This Guide Covers

By default, Unraid stores all Docker data inside a single file called `docker.img` — a **virtual disk (vDisk)**. This file has a fixed size (commonly 20–40GB), and when it fills up, Docker stops working.

**Directory mode** eliminates the vDisk entirely. Docker writes directly to a folder on your cache drive — no size cap, no artificial limits. It just uses whatever free space is available.

---

## ✅ What You Keep vs. 🗑️ What Gets Wiped

| &nbsp; | ✅ Kept (safe) | 🗑️ Wiped (re-downloaded) |
|---|---|---|
| 🎬 | Media files | Container images |
| ⚙️ | All app configs (Plex, Sonarr, Radarr, etc.) | Container layers & build cache |
| 💾 | Databases, watch history, settings | Running container state |
| 📋 | Docker container templates | |

> 💡 **Why your configs are safe:** App data lives in mapped volumes — typically `/mnt/user/appdata/` — which is completely separate from the Docker vDisk. The vDisk only holds the container images themselves (think of it like the "installer"), not your settings or data.

---

## 🔍 Before You Start

### 1. 📁 Confirm your appdata location
Go to the Unraid web UI and check a few containers. Click on a container name and look for a volume mapping pointing to `/mnt/user/appdata/`. If you see it — your configs are safe.

### 2. 🕐 Pick a low-traffic time
All containers will be offline for ~20–30 minutes while images re-download.

### 3. 💾 Back up your appdata *(optional but recommended)*
From the terminal:
```bash
cp -r /mnt/user/appdata/ /mnt/user/appdata-backup/
```
Or use **Settings → User Scripts** in the Unraid UI.

### 4. 📸 Note your installed containers
Go to the **Docker** tab and screenshot or write down which containers you have running. You'll re-add them from your templates after the switch.

---

## 🚀 Step-by-Step Migration

### Step 1 — 🛑 Stop Docker

1. Open the Unraid web UI
2. Navigate to **Settings → Docker**
3. Set **Enable Docker** to **No**
4. Click **Apply**

⏳ Wait for all containers to stop and the Docker service to shut down.

---

### Step 2 — 🔄 Switch to Directory Mode

On the same **Settings → Docker** page:

1. Change **Docker storage driver** to **overlay2** *(if not already set)*
2. Change **Docker data-root** (or **Docker directory**) — confirm it points to your cache drive, e.g. `/mnt/cache/docker/`
3. ☑️ Check the box to **delete the docker.img file** when prompted *(this reclaims the space)*

> 📝 **Unraid 7 note:** The UI labels may differ slightly. Look for the storage type toggle between "Image" and "Directory."

---

### Step 3 — ▶️ Start Docker

1. Set **Enable Docker** back to **Yes**
2. Click **Apply**

Docker will start fresh with an empty directory. This is expected — don't panic! 😄

---

### Step 4 — 📦 Re-Add Your Containers

1. Go to the **Docker** tab
2. Click **Add Container**
3. At the bottom, click **Template** and select a previously configured container
4. Verify the settings look correct *(they should be pre-filled from your saved template)*
5. Click **Apply**
6. 🔁 Repeat for each container

Each container will pull its image from Docker Hub. Speed depends on your internet connection.

> 💡 **Pro tip:** Go to **Apps → Previous Apps** to see everything you had installed and re-add them with one click each!

---

### Step 5 — ✅ Verify Everything Works

- [ ] All containers show as **running** on the Docker tab
- [ ] Open a few services (Plex, Sonarr, etc.) and confirm settings & data are intact
- [ ] Confirm the old `docker.img` is gone:
  ```bash
  df -h /mnt/cache
  ```

🎉 **That's it — you're done!**

---

## 🔧 Troubleshooting

<details>
<summary><strong>🚫 Container won't start after migration</strong></summary>

Click on the container name → **Edit**. Verify all volume mappings are correct and the paths exist on disk.
</details>

<details>
<summary><strong>❌ "No such image" errors</strong></summary>

The image just needs to re-download. Click the container name → **Force Update**, or remove and re-add from your template.
</details>

<details>
<summary><strong>💽 Old docker.img still taking up space</strong></summary>

If you didn't check the delete box during migration:
```bash
rm /mnt/cache/docker.img
```
*(Adjust the path if your vDisk was stored elsewhere.)*
</details>

<details>
<summary><strong>📋 Containers missing from the Templates list</strong></summary>

Go to **Apps → Previous Apps** to find and reinstall them.
</details>

<details>
<summary><strong>📂 Appdata seems empty</strong></summary>

Double-check your volume mappings. A common mistake is mapping to `/mnt/user/appdata/ContainerName` vs `/mnt/cache/appdata/ContainerName`. Use whatever path you had before.
</details>

---

## ❓ FAQ

<details>
<summary><strong>⏱️ How much downtime should I expect?</strong></summary>

About 20–30 minutes total, mostly waiting for container images to re-download.
</details>

<details>
<summary><strong>🔄 Can I switch back to vDisk mode?</strong></summary>

Yes! Same process in reverse — stop Docker, switch back to Image mode, set a vDisk size, start Docker, re-add containers.
</details>

<details>
<summary><strong>💿 Does this affect my Unraid array or parity?</strong></summary>

Nope. Docker storage sits on the cache drive, completely separate from the array.
</details>

<details>
<summary><strong>⚡ Is directory mode faster?</strong></summary>

Slightly — it removes the loopback mount overhead. In practice, most people won't notice a difference day-to-day.
</details>

<details>
<summary><strong>🗄️ Should I use btrfs or xfs for my cache drive?</strong></summary>

Either works great. If your cache pool is already formatted, don't change it just for this.
</details>

---
