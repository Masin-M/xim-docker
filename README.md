# Xim - Dockerized FFXI Browser Client

A fully self-contained Docker setup for running Xim offline. Everything is built inside Docker - no need to install Java, Node.js, or any other dependencies on your system.

## Prerequisites

1. **Docker** and **docker-compose**
2. **FFXI Game Files** (~8-15 GB) - the entire "FINAL FANTASY XI" folder

### Install Docker on Ubuntu

```bash
sudo apt update
sudo apt install docker.io docker-compose
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
# Log out and back in for group changes to take effect
```

## Directory Setup

Your directory should look like this:

```
xim-docker/
├── Dockerfile
├── docker-compose.yml
├── nginx.conf
├── run.sh
├── .env.example
├── build.gradle.kts        ← From source.zip
├── settings.gradle.kts     ← From source.zip
├── gradle.properties       ← From source.zip
├── gradle/                 ← From source.zip
├── src/                    ← From source.zip
└── webpack.config.d/       ← From source.zip
```

**Important:** Extract the Xim source code into this same directory so the Dockerfile can access it.

## Quick Start

### Option 1: Using the run script (easiest)

```bash
# Make executable
chmod +x run.sh

# Run with your FFXI folder path
./run.sh "/path/to/your/FINAL FANTASY XI"
```

### Option 2: Using docker-compose directly

```bash
# Set your FFXI path (the folder containing ROM/, ROM2/, sound/, etc.)
export XIM_FFXI_PATH="/path/to/your/FINAL FANTASY XI"

# Build the image (first time only)
docker-compose build

# Start the server
docker-compose up -d

# Open http://localhost:8080
```

### Option 3: Using .env file

```bash
# Create .env file
cp .env.example .env

# Edit and set your FFXI path
nano .env

# Build and run
docker-compose build
docker-compose up -d
```

## Required FFXI Folder Structure

Your FFXI folder should contain these directories:

```
FINAL FANTASY XI/
├── ROM/              ← Base game data (required)
│   ├── 0/
│   ├── 1/
│   ├── ...
│   └── 384/
├── ROM2/             ← Expansion content
├── ROM3/
├── ROM4/
├── ROM5/
├── ROM6/
├── ROM7/
├── ROM8/
├── ROM9/
├── sound/            ← Audio files (for music/sound effects)
│   └── win/
│       ├── se/       ← Sound effects
│       └── music/    ← Background music
├── VTABLE.DAT
├── FTABLE.DAT
└── ...
```

## Commands

| Command | Description |
|---------|-------------|
| `docker-compose up -d` | Start in background |
| `docker-compose down` | Stop |
| `docker-compose logs -f` | View logs |
| `docker-compose ps` | Check status |
| `docker-compose build` | Rebuild image |
| `docker-compose build --no-cache` | Full rebuild |

## Access

- **Local:** http://localhost:8080
- **Game Mode:** http://localhost:8080/?mode=game
- **Asset Viewer:** http://localhost:8080/

### From Other Devices

Find your Ubuntu machine's IP:
```bash
hostname -I
```

Then access from any device on your network:
```
http://YOUR_IP:8080
```

## Auto-Start on Boot

Create a systemd service:

```bash
sudo nano /etc/systemd/system/xim.service
```

Paste:
```ini
[Unit]
Description=Xim FFXI Browser Client
After=docker.service
Requires=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/path/to/xim-docker
Environment="XIM_FFXI_PATH=/path/to/your/FINAL FANTASY XI"
ExecStart=/usr/bin/docker-compose up -d
ExecStop=/usr/bin/docker-compose down

[Install]
WantedBy=multi-user.target
```

Enable:
```bash
sudo systemctl daemon-reload
sudo systemctl enable xim
sudo systemctl start xim
```

## Getting FFXI Game Files

The game files are copyrighted and must come from a legitimate FFXI installation.

### Option A: Copy from Windows

Copy this entire folder from a Windows PC:
```
C:\Program Files (x86)\PlayOnline\SquareEnix\FINAL FANTASY XI\
```

To your Ubuntu machine:
```
/home/yourusername/ffxi/FINAL FANTASY XI/
```

### Option B: Install via Lutris

```bash
sudo apt install lutris
# Search for "Final Fantasy XI" in Lutris and install
```

Game files typically end up in:
```
~/Games/final-fantasy-xi-online/drive_c/Program Files (x86)/PlayOnline/SquareEnix/FINAL FANTASY XI/
```

### Option C: Wine installation

```bash
sudo apt install wine64
wine FFXIFullSetup_US.exe
```

Game files will be in:
```
~/.wine/drive_c/Program Files (x86)/PlayOnline/SquareEnix/FINAL FANTASY XI/
```

## Troubleshooting

### Build fails

```bash
# Full rebuild without cache
docker-compose build --no-cache

# Check build logs
docker-compose build 2>&1 | tee build.log
```

### Container won't start

```bash
# Check logs
docker-compose logs

# Verify FFXI path exists and has correct structure
ls -la "/path/to/your/FINAL FANTASY XI/"
# Should show: ROM/, ROM2/, ROM3/, sound/, VTABLE.DAT, etc.
```

### Missing game content

If some zones, sounds, or effects are missing, ensure you have:
- All ROM folders (ROM through ROM9)
- The sound folder with music and sound effects
- VTABLE.DAT and FTABLE.DAT in the root

### Permission denied for Docker

```bash
sudo usermod -aG docker $USER
# Log out and back in
```

### Port 8080 in use

Edit `docker-compose.yml`:
```yaml
ports:
  - "9000:80"  # Use port 9000 instead
```

### Slow first load in browser

Normal! The browser caches DAT files. Subsequent loads are much faster.

## Resource Usage

- **Image size:** ~200 MB (after build)
- **Runtime memory:** ~50-100 MB
- **FFXI files:** ~8-15 GB (mounted, not copied)
- **CPU:** Minimal when idle

## Updating Xim

When a new version is released:

```bash
# Get new source files
# Replace src/ folder with updated version

# Rebuild
docker-compose build --no-cache

# Restart
docker-compose down
docker-compose up -d
```

Your save data is in your browser's localStorage and won't be affected.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Docker Container                  │
│  ┌───────────────────────────────────────────────┐  │
│  │                   nginx                        │  │
│  │            (serves static files)               │  │
│  └───────────────────────────────────────────────┘  │
│                         │                            │
│  ┌──────────────────────┼────────────────────────┐  │
│  │   /usr/share/nginx/html/                      │  │
│  │   ├── index.html  (built into image)          │  │
│  │   ├── xim.js      (built into image)          │  │
│  │   ├── env.json    (configures ffxi/ path)     │  │
│  │   ├── landsandboat/ (built into image)        │  │
│  │   └── ffxi/       ← mounted from host         │  │
│  │       ├── ROM/                                │  │
│  │       ├── ROM2/ ... ROM9/                     │  │
│  │       ├── sound/                              │  │
│  │       ├── VTABLE.DAT                          │  │
│  │       └── FTABLE.DAT                          │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
                          │
                          ▼
              ┌───────────────────────────────┐
              │       Host Filesystem         │
              │  /path/to/FINAL FANTASY XI/   │
              │      (your game files)        │
              └───────────────────────────────┘
```

The multi-stage Docker build:
1. **Stage 1:** Uses JDK to compile Kotlin/JS with Gradle
2. **Stage 2:** Copies only the built files to a lightweight nginx image

This keeps the final image small (~200 MB) while building everything inside Docker.

## How It Works

1. Your browser loads `index.html` and `xim.js` from nginx
2. The JavaScript (compiled from Kotlin) runs in your browser
3. It fetches DAT files via HTTP: `GET /ffxi/ROM/0/0.DAT`
4. nginx serves those files from your mounted FFXI folder
5. The JavaScript parses the DAT files and renders using WebGL
6. All rendering happens client-side in your browser
