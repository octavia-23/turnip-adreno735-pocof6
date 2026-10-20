# Turnip Vulkan Driver: Adreno 735 (POCO F6 / Snapdragon 8s Gen 3) Tuned

Custom build recipe and optimization patch stack for Mesa Turnip Vulkan driver tailored specifically for Qualcomm Snapdragon 8s Gen 3 (`SM8635`) / Adreno 735 devices (e.g., POCO F6 `peridot`, Xiaomi Civi 4 Pro, Realme GT Neo 6).

## Key Hardware Bottlenecks on 8s Gen 3 vs 8 Gen 3:
1. **Narrower Memory Bandwidth & SLC**: 4MB System-Level Cache (vs 12MB on 8 Gen 3) and narrower LPDDR5X bus.
2. **Aggressive Thermal Throttling**: HyperOS thermal engine clamps CPU/GPU when DRAM bus power spikes from unbinned rendering.
3. **Fewer SPs (Shader Processors)**: 4 SPs vs 6 SPs on Adreno 750, making thread wave occupancy critical for latency hiding.

## Applied Optimization Stack (v1.3.5):
- **`0001-freedreno-tu-default-to-PREFER_GMEM-autotuning-for-A.patch`**: Configures Turnip's native C++ autotuner to default to `PREFER_GMEM` mode. This ensures render passes aggressively stay within on-chip GMEM tiles (saving 4MB SLC and external DRAM bus bandwidth) while completely eliminating runtime performance-counter profiling stutters.
- **`0002-freedreno-ir3-cap-outstanding-texture-batches-to-4-o.patch`**: Caps outstanding texture sample and SFU instructions from 8 down to 4 in `ir3_sched.c` for A7xx hardware. Prevents excessive live registers waiting on memory returns, preserving maximum thread wave occupancy across Adreno 735's 4 physical SPs.
- **`0003-drirc-embed-application-optimization-profiles-for-An.patch`**: Embeds targeted low-latency and compatibility profiles into `00-mesa-defaults.conf` for **shadPS4** (PlayStation 4), **Winlator/Mobox** (DXVK / VKD3D-Proton), **Sudachi** (Switch), and **Vita3K** (PS Vita).

## How to Build via GitHub Actions:
1. Push updates to GitHub:
   ```bash
   git add .
   git commit -m "feat(driver): apply Part 2 hardware optimizations v1.3.5"
   git push origin main
   ```
2. Open your repo on GitHub and check the **Actions** tab.
3. The workflow compiles the driver with Android NDK r26c and produces `Turnip-Adreno735-8sGen3-POCO-F6.zip` under **Artifacts** and **Releases**.
4. Load the zip into your emulator's Custom Driver Manager (Winlator, Mobox, Sudachi, Suyu, shadPS4, etc.).
