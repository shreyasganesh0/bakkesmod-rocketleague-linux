# Installing BakkesMod on Linux with Heroic Games Launcher (GE-Proton)

A step-by-step guide for getting BakkesMod working with Rocket League installed via Heroic Games Launcher using GE-Proton on Linux.

> **Tested on:** Pop!\_OS, Heroic Games Launcher (Flatpak), GE-Proton-latest, AMD Radeon 780M / NVIDIA RTX 4070 Laptop GPU

---

## Prerequisites

- Rocket League installed and running via Heroic Games Launcher
- GE-Proton-latest set as the Wine/Proton version in Heroic
- `winetricks` and `mingw-w64` installed on your system

```bash
sudo apt install winetricks mingw-w64
```

---

## Step 1: Locate Your Wine Binary and Prefix

### Wine binary

Find the `wine64` binary that Heroic uses:

```bash
find ~/.var/app/com.heroicgameslauncher.hgl -name "wine64" -type f 2>/dev/null
```

For Heroic Flatpak with GE-Proton, this is typically:

```
~/.var/app/com.heroicgameslauncher.hgl/config/heroic/tools/proton/GE-Proton-latest/files/bin/wine64
```

### Wine prefix

Check Heroic → Rocket League → Settings for the Wine Prefix path. For most setups this is:

```
~/Games/Heroic/Prefixes/default
```

---

## Step 2: Set the Prefix to Windows 10

```bash
WINEPREFIX="$HOME/Games/Heroic/Prefixes/default" winetricks win10
```

Or manually:

```bash
WINEPREFIX="$HOME/Games/Heroic/Prefixes/default" \
  "$HOME/.var/app/com.heroicgameslauncher.hgl/config/heroic/tools/proton/GE-Proton-latest/files/bin/wine64" \
  winecfg
```

Set Windows Version to **Windows 10** in the dropdown and click Apply.

---

## Step 3: Download and Install BakkesMod

```bash
cd ~/Downloads
wget https://github.com/bakkesmodorg/BakkesModInjectorCpp/releases/latest/download/BakkesModSetup.zip
unzip BakkesModSetup.zip
chmod u+x BakkesModSetup.exe
```

Run the installer:

```bash
WINEPREFIX="$HOME/Games/Heroic/Prefixes/default" \
  "$HOME/.var/app/com.heroicgameslauncher.hgl/config/heroic/tools/proton/GE-Proton-latest/files/bin/wine64" \
  ~/Downloads/BakkesModSetup.exe
```

Go through the installer normally. BakkesMod will be installed to:

```
~/Games/Heroic/Prefixes/default/drive_c/Program Files/BakkesMod/BakkesMod.exe
```

---

## Step 4: Create the Epic Games Manifest Directory

BakkesMod needs this directory to detect an Epic Games installation:

```bash
mkdir -p "$HOME/Games/Heroic/Prefixes/default/drive_c/ProgramData/Epic/EpicGamesLauncher/Data/Manifests"
```

---

## Step 5: Switch to WineD3D (Critical Step)

**This is the key step that prevents the game from hanging after injection.**

BakkesMod's DirectX hooks conflict with DXVK (the default Vulkan translation layer). Switching to WineD3D (OpenGL translation) resolves this.

In Heroic, go to **Rocket League → Settings → Advanced** and add this **Environment Variable**:

```
PROTON_USE_WINED3D=1
```

> **Note:** You may see a slight performance decrease compared to DXVK, but BakkesMod will work correctly with full UI and no freezing.

---

## Step 6: Launch and Inject

1. **Launch Rocket League** through Heroic normally
2. **Wait until you are at the main menu** (not just the loading screen — wait at least 30 seconds)
3. In Heroic, go to Rocket League → Settings → **"Run EXE inside prefix"**
4. Point it to:

```
C:\Program Files\BakkesMod\BakkesMod.exe
```

5. BakkesMod will open. If it warns about an unsupported version, click **Yes** to inject anyway
6. If you see "Mod is out of date, waiting for an update", go to **Settings** and disable **"Enable safe mode"**
7. Press **F2** in-game to open the BakkesMod overlay

---

## Troubleshooting

### Game hangs/freezes after injection

You are likely still using DXVK. Make sure `PROTON_USE_WINED3D=1` is set in the environment variables (Step 5).

### BakkesMod says "No RL installation detected"

Make sure you created the Epic Games manifest directory (Step 4), and that you are launching BakkesMod via **"Run EXE inside prefix"** in Heroic — not from a terminal. BakkesMod must run inside the same sandbox as Rocket League.

### BakkesMod window doesn't appear

Make sure Rocket League is fully loaded at the main menu before running BakkesMod. Launching too early can cause failures.

### "Mod is out of date" message

Go to BakkesMod → Settings → uncheck **"Enable safe mode"**. You will get a warning — click Yes to proceed with injection.

### Performance is lower than usual

This is expected when using WineD3D instead of DXVK. The trade-off is that BakkesMod works correctly. If you find the performance unacceptable, BakkesMod unfortunately may not be fully compatible with your setup under DXVK.

---

## Summary of What's Happening

Rocket League on Linux runs through Proton/Wine, which uses **DXVK** to translate DirectX calls to Vulkan. BakkesMod injects a DLL into Rocket League and hooks into DirectX for its overlay and plugin system. These hooks conflict with DXVK's translation layer, causing the game to freeze.

By setting `PROTON_USE_WINED3D=1`, we switch from DXVK (Vulkan) to WineD3D (OpenGL), which is closer to native DirectX and allows BakkesMod's hooks to work without conflict.

---

## Credits

- [CrumblyLiquid/BakkesLinux](https://github.com/CrumblyLiquid/BakkesLinux) — Original Linux guide
- [BakkesLinux Issue #17](https://github.com/CrumblyLiquid/BakkesLinux/issues/17) — Heroic-specific instructions by @valenipes
- [blastrock's custom injector](https://gist.github.com/blastrock/6958033f03a0bdffa52c6dfa2ce0e60a) — Custom DLL injector for Linux
- [Allavaz's guide](https://gist.github.com/Allavaz/23bb92ec4a69fa74f01088cb846b3029) — Steam Proton setup guide
