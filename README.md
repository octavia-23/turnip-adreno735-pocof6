# Turnip Vulkan Driver: Adreno 735 (POCO F6 / Snapdragon 8s Gen 3) Tuned

Custom build recipe and optimization patch stack for Mesa Turnip Vulkan driver tailored specifically for Qualcomm Snapdragon 8s Gen 3 (`SM8635`) / Adreno 735 devices (e.g., POCO F6 `peridot`, Xiaomi Civi 4 Pro, Realme GT Neo 6).

## Key Hardware Bottlenecks on 8s Gen 3 vs 8 Gen 3:
1. **Narrower Memory Bandwidth & SLC**: 4MB System-Level Cache (vs 12MB on 8 Gen 3) and narrower LPDDR5X bus.
2. **Aggressive Thermal Throttling**: HyperOS thermal engine clamps CPU/GPU when DRAM bus power spikes from unbinned rendering.
3. **Fewer SPs (Shader Processors)**: High register pressure drops wave occupancy significantly faster than on Adreno 750 (4 SPs vs 6 SPs).

## Applied Optimization Patches:
- **`0001-adreno735-gmem-binning-bias.patch`**: Increases sysmem cost multiplier (1.6x penalty) to heavily bias towards on-chip GMEM tiling, preventing DRAM bus power spikes while respecting tile capacity.
- **`0002-ir3-a7xx-fp16-occupancy.patch`**: Late algebraic FP16 optimization and strict instruction scheduling register pressure caps (< 28 registers) to preserve maximum wave occupancy across the 4 physical SPs.
- **`0003-autotune-bypass-profiling-a7xx.patch`**: Disables runtime autotuner performance counter sampling, eliminating frame-pacing micro-stutter while safely delegating to the tuned GMEM cost model.
- **`0004-flush-denorms-to-zero.patch`**: Forces Flush-To-Zero (FTZ) on subnormal floating-point numbers to eliminate microcode exception penalties on SP ALUs.
- **`0005-embedded-drirc-emulator-profiles.patch`**: Embedded application profiles for Winlator, Mobox, Sudachi, Vita3K, and shadPS4 (dual-color blending and zero-latency presentation flags).

## How to Build via GitHub Actions:
1. Push updates to GitHub:
   ```bash
   git add .
   git commit -m "fix(driver): apply Part 1 architectural fixes and bump to v1.3.3"
   git push origin main
   ```
2. Open your repo on GitHub and check the **Actions** tab.
3. The workflow compiles the driver with Android NDK r26c and produces `Turnip-Adreno735-8sGen3-POCO-F6.zip` under **Artifacts** and **Releases**.
4. Load the zip into your emulator's Custom Driver Manager (Winlator, Mobox, Sudachi, Suyu, shadPS4, etc.).
