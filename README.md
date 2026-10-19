# Turnip Vulkan Driver: Adreno 735 (POCO F6 / Snapdragon 8s Gen 3) Tuned

Custom build recipe and optimization patches for Mesa Turnip Vulkan driver tailored specifically for Qualcomm Snapdragon 8s Gen 3 (`SM8635`) / Adreno 735 devices (e.g., POCO F6 `peridot`, Xiaomi Civi 4 Pro, Realme GT Neo 6).

## Key Hardware Bottlenecks on 8s Gen 3 vs 8 Gen 3:
1. **Narrower Memory Bandwidth & SLC**: 4MB System-Level Cache (vs 12MB on 8 Gen 3) and narrower LPDDR5X bus.
2. **Aggressive Thermal Throttling**: HyperOS thermal engine clamps CPU/GPU when DRAM bus power spikes from unbinned rendering.
3. **Fewer SPs (Shader Processors)**: High register pressure drops wave occupancy significantly faster than on Adreno 750.

## Patches Applied:
- **`0001-adreno735-gmem-binning-bias.patch`**: Lowers binning cost thresholds to heavily bias towards on-chip GMEM tiling and avoids Sysmem DRAM chokepoints.
- **`0002-ir3-a7xx-fp16-occupancy.patch`**: Automatically lowers medium-precision math to FP16 (2x ALU rate) and caps instruction scheduling register pressure to preserve maximum wave occupancy per SP.
- **`0003-force-ubwc-compression.patch`**: Enforces Universal Bandwidth Compression across attachments and sampled textures to reduce bus traffic by 30-50%.

## How to Build via GitHub Actions (Zero Local Setup):
1. Create a new GitHub repository on your account.
2. Push this repository's files to GitHub:
   ```bash
   git init
   git add .
   git commit -m "feat: Adreno 735 / 8s Gen 3 optimization stack"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<your-repo-name>.git
   git push -u origin main
   ```
3. Open your repo on GitHub, go to the **Actions** tab.
4. The workflow will automatically run, compile the driver with Android NDK r26c, and generate `Turnip-Adreno735-8sGen3-POCO-F6.zip` under **Artifacts**.
5. Download the zip and load it directly into your emulator's Custom Driver Manager (Winlator, Mobox, Sudachi, Suyu, etc.).
