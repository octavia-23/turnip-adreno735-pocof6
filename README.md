# Turnip Vulkan Driver: Adreno 735 (POCO F6 / Snapdragon 8s Gen 3) Tuned

Custom build recipe and optimization patches for Mesa Turnip Vulkan driver tailored specifically for Qualcomm Snapdragon 8s Gen 3 (`SM8635`) / Adreno 735 devices (e.g., POCO F6 `peridot`, Xiaomi Civi 4 Pro, Realme GT Neo 6).

## Key Hardware Bottlenecks on 8s Gen 3 vs 8 Gen 3:
1. **Narrower Memory Bandwidth & SLC**: 4MB System-Level Cache (vs 12MB on 8 Gen 3) and narrower LPDDR5X bus.
2. **Aggressive Thermal Throttling**: HyperOS thermal engine clamps CPU/GPU when DRAM bus power spikes from unbinned rendering.
3. **Fewer SPs (Shader Processors)**: High register pressure drops wave occupancy significantly faster than on Adreno 750.

## Patches Applied:
- **`0001-adreno735-gmem-binning-bias.patch`**: Increases sysmem cost multiplier (1.6x penalty) to heavily bias towards on-chip GMEM tiling and avoids Sysmem DRAM chokepoints while respecting tile capacity.
- **`0002-ir3-a7xx-fp16-occupancy.patch`**: Lowers medium-precision math to FP16 (2x ALU rate) and caps instruction scheduling register pressure to preserve maximum wave occupancy per SP without corrupting output registers.
- **`0005-kgsl-batch-submission-tuning.patch`**: Batches KGSL command submissions to minimize kernel context switching.
- **`0006-gpl-fast-linking-adreno735.patch`**: Fast-path Graphics Pipeline Library (GPL) linking to eliminate emulator shader stutter.
- **`0007-suballocator-contiguous-pools.patch`**: 16MB contiguous suballocator memory pools to prevent fragmentation.
- **`0008-autotune-gmem-lock-for-a7xx.patch`**: Locks GMEM autotuner to prevent mid-frame GMEM/sysmem flip-flop stutter.
- **`0009-flush-denorms-to-zero.patch`**: Forces Flush-To-Zero (FTZ) on subnormal floats to remove SP ALU cycle penalties.
- **`0010-d24s8-depth-fastpath.patch`**: Direct tiled resolves for D24S8 depth-stencil formats.
- **`0011-custom-border-color-alignment.patch`**: 64-byte sampler border alignment for Adreno 7xx hardware samplers.
- **`0012-embedded-drirc-emulator-profiles.patch`**: Embedded application profiles for Winlator, Mobox, Sudachi, Vita3K, and shadPS4.

*(Note: Unsafe LRZ retention and forced UBWC patches were removed in v1.3.2 to resolve visual artifacting in Bloodborne and translation layers while preserving full hardware throughput.)*

## How to Build via GitHub Actions (Zero Local Setup):
1. Push this repository's files to GitHub:
   ```bash
   git add .
   git commit -m "fix(driver): resolve Bloodborne GFX artifacts and update to v1.3.2"
   git push origin main
   ```
2. Open your repo on GitHub, go to the **Actions** tab.
3. The workflow will automatically compile the driver with Android NDK r26c and generate `Turnip-Adreno735-8sGen3-POCO-F6.zip` under **Artifacts** and **Releases**.
4. Download the zip and load it directly into your emulator's Custom Driver Manager (Winlator, Mobox, Sudachi, Suyu, shadPS4, etc.).
