# Turnip Vulkan Driver: Adreno 735 (Snapdragon 8s Gen 3)

Custom build recipes and patch stack for the open-source Mesa Turnip Vulkan driver, configured for Qualcomm Snapdragon 8s Gen 3 (`SM8635`) / Adreno 735 devices (such as POCO F6 `peridot`, Xiaomi Civi 4 Pro, and Realme GT Neo 6) on Android 14+ (`minApi: 34`).

---

## Hardware Context: Snapdragon 8s Gen 3 vs 8 Gen 3

The Snapdragon 8s Gen 3 (`SM8635`) features an Adreno 735 GPU, which differs in key architectural ways from the flagship 8 Gen 3 (`SM8650` / Adreno 750):
* **Memory & System-Level Cache (SLC)**: 4MB SLC (compared to 12MB on 8 Gen 3) and narrower memory bandwidth, making cache locality and tiled GMEM rendering especially important.
* **Shader Processor Count**: 4 physical Shader Processors (SPs) compared to 6 SPs on Adreno 750, making instruction scheduling and register pressure management important to maintain thread wave occupancy.
* **Thermal Envelope**: Tighter thermal limits on passively cooled devices where sustained memory bus traffic can trigger thermal throttling.

---

## Applied Patch Stack

* **`0001-freedreno-tu-default-to-PREFER_GMEM-autotuning-for-A.patch`**
  - Sets Turnip's autotuner default mode to `PREFER_GMEM` while preserving the `TU_AUTOTUNE_ALGO` runtime override.
  - Encourages tiled on-chip rendering to reduce external memory round-trips and keep rendering passes within GMEM buffers where feasible.

* **`0002-freedreno-ir3-cap-outstanding-texture-batches-to-4-o.patch`**
  - Scoped specifically to 4-SP Adreno 7xx hardware (`ctx->compiler->gen == 7 && ctx->compiler->info->num_sp_cores <= 4`).
  - Caps high-latency outstanding `sy` (texture/memory) batches to 6 (down from 8) to reduce register-pressure spikes while preserving latency hiding, and leaves short-latency `ss` (SFU) batches at 8.

---

## Build & Performance Configuration

* **Pinned Upstream Mesa**: Built against the exact commit in [`mesa-version.env`](mesa-version.env) for reproducible builds and clean bug bisection.
* **Toolchain & CPU Tuning**: Compiled with Android NDK `r27c` (`aarch64-linux-android34-clang`), `-O3 -march=armv8.2-a+dotprod+fp16 -mtune=cortex-a720`, and ThinLTO (`-Db_lto=true -Db_lto_mode=thin`) for faster host-side NIR/ir3 shader compilation and Vulkan command recording.
* **Dual-Variant Releases**: Tagging `v*` publishes both `turnip-adreno735-8sgen3-pocof6-tuned.zip` and `turnip-adreno735-8sgen3-pocof6-stock.zip` (plus `SHA256SUMS.txt`) so regressions can be isolated between upstream Mesa and the SM8635 patch stack.

---

## Building from Source

1. Fork or push to your repository.
2. Push/PR runs CI build verification and ELF/metadata sanity checks, uploading artifacts.
3. Push a version tag (`git tag v1.4.0 && git push origin v1.4.0`) to publish the dual-variant release under **Releases**.
4. Load the `.zip` archive into your emulator's custom graphics driver manager.

---

## Credits & Upstream
* **Mesa 3D Graphics Library**: [https://www.mesa3d.org](https://www.mesa3d.org)
* **Freedreno / Turnip Driver**: Developed by the Mesa community, Rob Clark, Danylo Piliaiev, and contributors.
