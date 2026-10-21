# Turnip Vulkan Driver: Adreno 735 (Snapdragon 8s Gen 3)

Custom build recipes and patch stack for the open-source Mesa Turnip Vulkan driver, configured for Qualcomm Snapdragon 8s Gen 3 (`SM8635`) / Adreno 735 devices (such as POCO F6 `peridot`, Xiaomi Civi 4 Pro, and Realme GT Neo 6).

---

## Hardware Context: Snapdragon 8s Gen 3 vs 8 Gen 3

The Snapdragon 8s Gen 3 (`SM8635`) features an Adreno 735 GPU, which differs in key architectural ways from the flagship 8 Gen 3 (`SM8650` / Adreno 750):
* **Memory & System-Level Cache (SLC)**: 4MB SLC (compared to 12MB on 8 Gen 3) and narrower memory bandwidth, making cache locality and tiled GMEM rendering especially important.
* **Shader Processor Count**: 4 physical Shader Processors (SPs) compared to 6 SPs on Adreno 750, making instruction scheduling and register pressure management important to maintain thread wave occupancy.
* **Thermal Envelope**: Tighter thermal limits on passively cooled devices where sustained memory bus traffic can trigger thermal throttling.

---

## Applied Patch Stack

* **`0001-freedreno-tu-default-to-PREFER_GMEM-autotuning-for-A.patch`**
  - Sets Turnip's autotuner default mode to `PREFER_GMEM` for Adreno 7xx.
  - Encourages tiled on-chip rendering to reduce external memory round-trips and keep rendering passes within GMEM buffers where feasible.

* **`0002-freedreno-ir3-cap-outstanding-texture-batches-to-4-o.patch`**
  - Caps outstanding texture sample and SFU instructions from 8 to 4 in `ir3_sched.c` for A7xx hardware.
  - Helps limit register pressure while waiting on high-latency memory fetches to maintain warp occupancy across the 4 physical SPs.

* **`0003-drirc-embed-application-optimization-profiles-for-An.patch`**
  - Embeds configuration overrides into `00-mesa-defaults.conf` for Android emulator runtimes (including Winlator, Mobox, shadPS4, and Vita3K) to optimize swapchain presentation and synchronization settings.

---

## Building from Source

This repository builds automatically via GitHub Actions:
1. Fork or push to your repository.
2. In the GitHub **Actions** tab, the build workflow uses Android NDK r26c to compile Mesa Turnip.
3. The resulting installable driver package `Turnip-Adreno735-8sGen3-POCO-F6.zip` is published under **Releases** and **Artifacts**.
4. The `.zip` archive can be loaded into your emulator's custom graphics driver manager.

---

## Credits & Upstream
* **Mesa 3D Graphics Library**: [https://www.mesa3d.org](https://www.mesa3d.org)
* **Freedreno / Turnip Driver**: Developed by the Mesa community, Rob Clark, Danylo Piliaiev, and contributors.
