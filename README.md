# VKD3D-Proton builds

Optimized [VKD3D-Proton](https://github.com/HansKristian-Work/vkd3d-proton) (Direct3D 12 to Vulkan translation layer) built for modern CPUs.

## Requirements

- **Architecture:** amd64
- **CPU:** x86-64-v3 (AVX2) or x86-64-v4 (AVX-512) - pick the matching variant
- **Runtime:** Wine 8.0+ or Proton 8.0+

**Builds will not run** on CPUs without AVX2/AVX-512. Check support:

```bash
grep -o 'avx[0-9_]*' /proc/cpuinfo | sort -u
```

If the output is empty - **do not use these builds**.

## Package

<details>
<summary>Build details</summary>

| Component      | Description                                                                     |
|----------------|---------------------------------------------------------------------------------|
| Target         | Windows PE DLLs (x64 + x86), cross-compiled                                     |
| Compiler       | Clang from [llvm-mingw](https://github.com/mstorsjo/llvm-mingw) 20260922 (UCRT) |
| Linker         | LLD                                                                             |
| Compiler flags | `-march=x86-64-v3/znver3` or `-march=x86-64-v4/znver4`, `-O3`                   |
| Patches        | None (upstream build with custom cross-files)                                   |

Debug symbols are **not included** in releases - they do not affect performance and only take up space.
</details>

## Installation

### 1. Download and extract

```bash
mkdir -p ~/vkd3d-opt && cd ~/vkd3d-opt
gh release download --repo argonforge/vkd3d-proton-builds --pattern '*.tar.zst'

# Unpack the archive matching your CPU:
tar --zstd -xf vkd3d-proton-*-x86-64-v4.tar.zst   # AVX-512 (Zen 4/5)
# or
tar --zstd -xf vkd3d-proton-*-x86-64-v3.tar.zst   # AVX2 (Zen 1/2/3)
```

Or download the `.tar.zst` file manually from the [Releases](../../releases) page.

### 2. Install into Wine prefix

```bash
export WINEPREFIX=/path/to/your/prefix
cd vkd3d-proton-*/

./setup_vkd3d_proton.sh install
```

Use `--symlink` if you prefer symlinks over copying (convenient when updating):

```bash
./setup_vkd3d_proton.sh install --symlink
```

### 3. Clear shader caches

**Mandatory.** Old shader caches are incompatible with the new VKD3D-Proton version and will cause crashes or rendering artifacts.

```bash
rm -rf ~/.cache/vkd3d-proton/* \
       ~/.cache/dxvk/* \
       ~/.cache/mesa_shader_cache*
```

For per-game caches, remove `vkd3d-proton.cache*` next to the game's `.exe`:

```bash
rm -f /path/to/game/vkd3d-proton.cache \
      /path/to/game/vkd3d-proton.cache.write
```

First launch after clearing the cache will be slower - shaders are recompiled and a fresh cache is created.

### 4. Verify

Launch the game with the VKD3D-Proton HUD enabled:

```bash
VKD3D_CONFIG=hud WINEPREFIX=/path/to/your/prefix wine /path/to/game.exe
```

A HUD overlay with device info and shader activity confirms VKD3D-Proton is active.

## Rollback

This is a manual DLL installation - no package manager integration. To roll back:

```bash
export WINEPREFIX=/path/to/your/prefix
cd ~/vkd3d-opt/vkd3d-proton-*/

./setup_vkd3d_proton.sh uninstall

# Reset shader caches to avoid stale data
rm -rf ~/.cache/vkd3d-proton/*
```

Or simply delete the Wine prefix and recreate it:

```bash
rm -rf "$WINEPREFIX"
WINEPREFIX=/path/to/your/prefix WINEARCH=wow64 wineboot --init
```

## Expected Performance

| Component                                    | Gain   | Comment                                   |
|----------------------------------------------|--------|-------------------------------------------|
| CPU part of VKD3D-Proton (D3D12 translation) | 1-5%   | Noticeable in CPU-bound scenarios         |
| GPU-bound games                              | ~0%    | Bottleneck is GPU and memory bandwidth    |
| Shader compilation (ACO)                     | **0%** | ACO does not use VKD3D-Proton build flags |

**Honest note:** VKD3D-Proton is a translation layer, not a graphics driver. Its own CPU cost matters only in scenarios where the game is limited by D3D12 call overhead. The main FPS gains in games come from the graphics stack (Mesa, RADV), not from VKD3D-Proton itself.

## Companion projects

For a complete optimized graphics stack on **AMD Zen (x86-64-v3/v4)**:

| Project                                                                    | Purpose                           |
|----------------------------------------------------------------------------|-----------------------------------|
| [`mesa-builds`](https://github.com/argonforge/mesa-builds)                 | Mesa (radeonsi, RADV)             |
| [`dxvk-builds`](https://github.com/argonforge/dxvk-builds)                 | DXVK (D3D9/10/11 -> Vulkan)       |
| [`wine-builds`](https://github.com/argonforge/wine-builds)                 | Wine WoW64 (Clang)                |
| [`gamescope-builds`](https://github.com/argonforge/gamescope-builds)       | Micro-compositor for game scaling |

## Important

- Builds are compiled on **Arch Linux**. The resulting DLLs are Windows PE files that run inside any Wine 8.0+ or Proton 8.0+ prefix, regardless of the host distribution.
- Builds are **not signed**. Verify integrity using SHA-256 from the release description.
- **Do not use v4 builds** if you are unsure about AVX-512 support. Use v3 if your CPU has only AVX2.
- VKD3D-Proton manipulation in online multiplayer games may be considered cheating. **Use at your own risk.**
- Always keep a working VKD3D-Proton release from [upstream](https://github.com/HansKristian-Work/vkd3d-proton) as fallback.
- Not affiliated with the upstream project. Report build-specific issues in this repository's [Issues](../../issues) tracker.

## License

The build scripts and GitHub Actions workflows in this repository
are licensed under the MIT License. See LICENSE file.

The VKD3D-Proton source code is distributed under the
[VKD3D license](https://github.com/HansKristian-Work/vkd3d-proton/blob/master/LICENSE).
The compiled DLLs in Releases are redistributions of VKD3D-Proton under its original license.
