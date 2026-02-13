# Kali Linux Audio Crackling Fix (Conexant CX20724)

![Linux Audio](https://img.shields.io/badge/Linux-Audio_Fix-blue?style=for-the-badge&logo=linux)
![Platform](https://img.shields.io/badge/Hardware-Conexant_CX20724-brightgreen?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Resolved-success?style=for-the-badge)

## 📌 Overview
This repository documents a definitive fix for the persistent **audio crackling, popping, and buffer underruns** experienced on **Kali Linux** (and other Debian-based rolling releases) specifically utilizing the **Conexant CX20724** audio codec.

While hardware often performs flawlessly on Windows, the Linux implementation of PipeWire/WirePlumber often suffers from aggressive power management or insufficient buffer headroom, leading to broken audio streams.

## 🛠 The Root Cause
The issue is primarily driven by a mismatch between the **ALSA period size** and the **WirePlumber headroom** settings. On sensitive codecs like the CX20724, the default latency targets are too tight, causing the audio buffer to "underrun" during CPU-intensive tasks or standard playback.

---

## ✅ The Solution: WirePlumber ALSA Tuning

We resolve this by creating a configuration override for WirePlumber to increase the ALSA headroom and stabilize the period size.

### 1. Initialize Configuration Directory
```bash
mkdir -p ~/.config/wireplumber/wireplumber.conf.d

```

### 2. Create the Override File

Create a new configuration file named `50-alsa-config.conf`:

```bash
nano ~/.config/wireplumber/wireplumber.conf.d/50-alsa-config.conf

```

### 3. Apply Professional Audio Logic

Paste the following Lua configuration:

```lua
monitor.alsa.rules = [
  {
    matches = [
      { node.name = "~alsa_output.*" }
    ]
    actions = {
      update-props = {
        api.alsa.period-size = 1024
        api.alsa.headroom = 8192
      }
    }
  }
]

```

### 4. Restart the Audio Stack

Apply changes without rebooting:

```bash
systemctl --user restart wireplumber pipewire pipewire-pulse

```

---

## 🔍 Diagnostic Toolkit

If you encounter further issues, use these commands to audit your audio environment:

| Command | Purpose |
| --- | --- |
| `aplay -l` | List all hardware playback devices |
| `dmesg | grep -i snd` |
| `pactl info` | Verify PipeWire is currently running |
| `sudo alsa force-reload` | Hard reset the ALSA kernel modules |

## 💡 Technical Insight

> **Why 8192 headroom?** By increasing the `api.alsa.headroom`, we provide a larger "safety net" for the CPU to fill the audio buffer before the hardware consumes it. For the Conexant CX20724, this eliminates the "scratching" artifacts caused by timing jitters.

---

**Maintainer:** [Your Name/VansKE]

**Topic:** Linux Kernel Audio Optimization

**License:** MIT

```
