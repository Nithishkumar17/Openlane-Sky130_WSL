# Openlane-Sky130_WSL
VSD OpenLane Workshop documentation — WSL2 setup, OpenLane + Sky130A PDK install, and picorv32a full flow.

# VSD OpenLane Workshop

## System Requirements

- Windows 11
- WSL2 with Ubuntu 22.04
- 8-16GB RAM minimum
- ~50GB - 100GB free disk space (Docker image ~3–4GB + Sky130A PDK ~5GB + For future scalability)

---

## Setup

### Step 1 — Docker Desktop

Download and install Docker Desktop. During installation, enable **"Use WSL2 instead of Hyper-V"**.

After installation:
- Settings → Resources → WSL Integration → enable for **Ubuntu-22.04**
- Settings → General → enable **"Start Docker Desktop when you login"**

Verify Docker is working:

```bash
docker run hello-world
# Expected output: Hello from Docker!
```

---

### Step 2 — Directory Structure & Repository Setup

```bash
mkdir ~/openlane && cd ~/openlane
git clone https://github.com/The-OpenROAD-Project/OpenLane.git
git clone https://github.com/vsdip/vsd-openlane.git
```

This gives you two repos:

```
~/openlane/
├── OpenLane/        ← Engine (tools, Docker setup, PDK manager)
└── vsd-openlane/    ← Workshop (design files, instructions)
```

> **Note:** `vsd-openlane` contains a hidden `.openlane-designs` folder — only visible with `ls -la`

---

### Step 3 — OpenLane Installation & Sky130 PDK Setup

```bash
cd ~/openlane/OpenLane
make
```

This pulls the Docker image (~3–4GB) and installs the Sky130A PDK (~5GB). Takes ~30 minutes on first run.

PDK will be installed at: `~/.ciel/sky130A/`

Verify the installation:

```bash
make test
# Expected output:
# [SUCCESS]: Flow complete.
# [INFO]: There are no setup or hold violations in the design.
```

---

### Step 4 — Workshop Design Setup

The `.openlane-designs` folder is hidden — verify it exists first:

```bash
ls -la ~/openlane/vsd-openlane/
```

Copy the `picorv32a` design into OpenLane:

```bash
cp -r ~/openlane/vsd-openlane/.openlane-designs/picorv32a \
      ~/openlane/OpenLane/designs/
```

Verify the copy:

```bash
ls ~/openlane/OpenLane/designs/picorv32a
# Expected: config.tcl  src/
```

---

### Step 5 — Run the Full Automated Flow (picorv32a)

```bash
cd ~/openlane/OpenLane
make mount
```

Inside the container:

```bash
./flow.tcl -design picorv32a -verbose 1
```

Takes ~20–25 minutes. Expected output:

```
[SUCCESS]: Flow complete.
[INFO]: There are no setup violations in the design.
[INFO]: There are no hold violations in the design.
```

Run output is saved at:

```
~/openlane/OpenLane/designs/picorv32a/runs/RUN_<timestamp>/
├── results/final/      ← GDSII, DEF, LEF files
├── reports/            ← timing, DRC, metrics
└── logs/               ← per-step logs
```

---

### Step 6 — View Layout in Magic

Magic is launched **inside the Docker container** (the container has Magic 8.3.413 which is version-matched to Sky130A PDK; the Ubuntu apt version is too old).

```bash
cd ~/openlane/OpenLane
make mount
```

Inside the container:

```bash
magic -T $PDK_ROOT/sky130A/libs.tech/magic/sky130A.tech &
```

The GUI forwards automatically to Windows 11 via WSLg — no extra configuration needed.

In the Magic **tkcon** window, load the layout:

```tcl
cd designs/picorv32a/runs/RUN_<timestamp>/tmp/
lef read merged.nom.lef
cd ../results/final/def/
def read picorv32a.def
```

Press **`V`** to zoom fit — the full routed picorv32a layout will be visible.

---

## Key Paths Reference

| Location | Path |
|---|---|
| Workshop repo | `~/openlane/vsd-openlane/` |
| OpenLane engine | `~/openlane/OpenLane/` |
| PDK | `~/.ciel/sky130A/` |
| picorv32a design | `~/openlane/OpenLane/designs/picorv32a/` |
| Run output | `~/openlane/OpenLane/designs/picorv32a/runs/RUN_<timestamp>/` |
| Final results | `.../runs/RUN_<timestamp>/results/final/` |
| Inside container | `/openLANE_flow/` → maps to `~/openlane/OpenLane/` |
| PDK inside Docker | `$PDK_ROOT/sky130A/` |

---

## Standard Workflow (After Initial Setup)

```bash
# 1. Start Docker Desktop from Windows Start Menu

# 2. Enter the OpenLane container
cd ~/openlane/OpenLane
make mount

# 3. Run the full automated flow (inside container)
./flow.tcl -design picorv32a -verbose 1

# 4. Open Magic for layout viewing (inside container)
magic -T $PDK_ROOT/sky130A/libs.tech/magic/sky130A.tech &
```
