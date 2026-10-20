# Turnip Vulkan Driver: Adreno 735 (POCO F6 / Snapdragon 8s Gen 3) Tuned

Custom build recipe and verified optimization patch stack for Mesa Turnip Vulkan driver tailored specifically for Qualcomm Snapdragon 8s Gen 3 (`SM8635`) / Adreno 735 devices (e.g., POCO F6 `peridot`, Xiaomi Civi 4 Pro, Realme GT Neo 6).

## Key Hardware Bottlenecks on 8s Gen 3 vs 8 Gen 3:
1. **Narrower Memory Bandwidth & SLC**: 4MB System-Level Cache (vs 12MB on 8 Gen 3) and narrower LPDDR5X bus.
2. **Aggressive Thermal Throttling**: HyperOS thermal engine clamps CPU/GPU when DRAM bus power spikes from unbinned rendering.
3. **Fewer SPs (Shader Processors)**: 4 SPs vs 6 SPs on Adreno 750.

## Verified Optimization Patches:
- **`0001-autotune-prefer-gmem-adreno735.patch`**: Configures Mesa Turnip's native C++ autotuner (`tu_autotune`) to default to `PREFER_GMEM` mode. This ensures render passes aggressively stay within on-chip GMEM tiles (saving 4MB SLC and external DRAM bus bandwidth) while completely eliminating runtime performance-counter profiling stutters.
- **`0002-embedded-drirc-emulator-profiles.patch`**: Embeds targeted low-latency and compatibility profiles into `00-mesa-defaults.conf` for **shadPS4** (PlayStation 4), **Winlator/Mobox** (DXVK / VKD3D-Proton), **Sudachi** (Switch), and **Vita3K** (PS Vita).

## How to Build via GitHub Actions:
1. Push updates to GitHub:
   ```bash
   git add .
   git commit -m "feat(driver): apply upstream-verified optimization stack v1.3.4"
   git push origin main
   ```
2. Open your repo on GitHub and check the **Actions** tab.
3. The workflow compiles the driver with Android NDK r26c and produces `Turnip-Adreno735-8sGen3-POCO-F6.zip` under **Artifacts** and **Releases**.
4. Load the zip into your emulator's Custom Driver Manager (Winlator, Mobox, Sudachi, Suyu, shadPS4, etc.).
