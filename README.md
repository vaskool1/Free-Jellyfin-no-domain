# Free Jellyfin Tunnel — No Domain, No Port Forwarding

Access your Jellyfin server from anywhere in the world, completely free. No paid domain, no port forwarding, no router configuration required. This project uses a Cloudflare Quick Tunnel to expose your local Jellyfin server, and automatically keeps a GitHub Pages redirect page updated with the latest tunnel URL so your bookmark never breaks.

---

## How It Works

1. `cloudflared.exe` opens a free encrypted tunnel from Cloudflare to your local Jellyfin
2. The script reads the tunnel URL from the log and pushes it to your GitHub Pages redirect page
3. Anyone you share the GitHub Pages link with will always be redirected to the correct tunnel
4. If your internet drops or the tunnel changes, the script detects this and automatically gets a new URL and updates the redirect page

---

## Requirements

Before starting, make sure you have the following:

- A Windows PC with Jellyfin installed and running (default port: `8096`)
- `cloudflared.exe` — download free from [Cloudflare's website](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/) (no account needed)
- A free [GitHub](https://github.com) account
- A GitHub Personal Access Token (PAT) — instructions below

---

## Setup Guide

### Step 1 — Create your folder and download files

1. Create a folder at `C:\JellyfinTunnel\`
2. Place the following files inside it:
   - `cloudflared.exe` (downloaded from the Cloudflare link above)
   - `watch_tunnel.ps1` (from this repo)
   - `start_jellyfin_tunnel.bat` (from this repo)

Your folder should look like this:
```
C:\JellyfinTunnel\
    cloudflared.exe
    watch_tunnel.ps1
    start_jellyfin_tunnel.bat
```

---

### Step 2 — Create your GitHub Pages redirect repo

This is the page that will always redirect visitors to your current tunnel URL.

1. Log in to [GitHub](https://github.com) and click **New repository**
2. Name it something like `jellyfin-redirect` and set it to **Public**
3. Click **Create repository**
4. Go to **Settings → Pages**, set the source to **Deploy from a branch**, select `main`, and click **Save**
5. In the repo, click **Add file → Create new file**
6. Name the file `index.html` and paste in this starter content:
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Jellyfin</title>
</head>
<body>
  <p>Tunnel not yet configured.</p>
</body>
</html>
```
7. Click **Commit changes**
8. Click **Add file → Create new file** again
9. Name this file `_headers` (no extension) and paste in:
```
/index.html
  Cache-Control: no-store, no-cache, must-revalidate
```
10. Click **Commit changes**

This `_headers` file tells browsers never to cache the redirect page, so they always fetch the latest tunnel URL instead of an old one.

---

### Step 3 — Create a GitHub Personal Access Token (PAT)

The script needs this to update your redirect page automatically.

1. Go to [github.com/settings/tokens](https://github.com/settings/tokens)
2. Click **Generate new token → Generate new token (classic)**
3. Give it a descriptive name like `JellyfinTunnel`
4. Set an expiry date (90 days is fine — you can always regenerate it)
5. Under **Select scopes**, check the box next to **repo**
6. Scroll down and click **Generate token**
7. **Copy the token immediately** — GitHub will only show it once

> ⚠️ Keep your PAT private. Never share it or upload it to GitHub. If it gets exposed, go back to [github.com/settings/tokens](https://github.com/settings/tokens) and delete it, then generate a new one.

---

### Step 4 — Configure the script

1. Open `watch_tunnel.ps1` in Notepad or any text editor
2. Fill in the configuration section at the top of the file:

```powershell
$CloudflaredExe   = "C:\JellyfinTunnel\cloudflared.exe"   # Leave this as-is if you used the same folder
$JellyfinUrl      = "http://localhost:8096"                # Change port if your Jellyfin uses a different one
$GitHubUser       = "YOUR_GITHUB_USERNAME"                 # Your GitHub username
$GitHubRepo       = "jellyfin-redirect"                    # Your redirect repo name from Step 2
$GitHubPAT        = "YOUR_GITHUB_PAT_HERE"                 # Your PAT from Step 3
$BranchName       = "main"                                 # Leave as main unless you used a different branch
$WatchdogInterval = 300                                    # How often (seconds) to verify the redirect is current
```

3. Save the file

---

### Step 5 — Run the tunnel

1. Double-click `start_jellyfin_tunnel.bat`
2. A terminal window will open and you will see output like:
```
Starting Jellyfin tunnel watcher...
Bookmark this URL: https://YOUR_USERNAME.github.io/jellyfin-redirect

16:48:23 - Watchdog started (Job ID 1), checking every 300 seconds.
16:48:23 - Waiting for internet connection...
16:48:23 - Internet is back!
16:48:23 - Starting cloudflared...
16:48:27 - Tunnel URL: https://example-tunnel.trycloudflare.com
16:48:27 - GitHub Pages updated successfully.
```
3. Bookmark the GitHub Pages URL shown in the output — this is the only link you ever need to share

---

### Step 6 (Optional) — Run automatically on Windows startup

If you want the tunnel to start every time your PC boots without you having to do anything:

1. Press `Win + R`, type `shell:startup`, and press Enter
2. A folder will open — copy a shortcut to `start_jellyfin_tunnel.bat` into it
3. From now on, the tunnel will start automatically whenever Windows starts

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Redirect goes to an old URL | Press `Ctrl + Shift + R` to hard refresh, or open in a private/incognito tab |
| GitHub push fails | Check your PAT hasn't expired and has the `repo` scope enabled |
| Tunnel URL not found | Make sure Jellyfin is running before starting the script |
| Script won't run | Right-click `start_jellyfin_tunnel.bat` and select **Run as administrator** |
| Internet keeps showing as lost | This is a false positive — the script requires 2 consecutive failures before restarting |

---

## Found a Bug or Need Help?

Open an [issue](../../issues) and I'll help when I can!

---

*Tested on Windows 10 and Windows 11.*
