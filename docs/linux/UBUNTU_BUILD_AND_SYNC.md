# Vibepollo Linux Build & Fork Synchronization Guide

This guide documents the toolchain setup, build fixes, testing, packaging, and upstream synchronization workflow for **Vibepollo** on Linux (Ubuntu 24.04 / 26.04 with GCC 14 and NVIDIA CUDA 13.1).

---

## 1. Prerequisites & Toolchain Setup

- **Compiler**: GCC / G++ 14 (`gcc-14`, `g++-14`)
- **CUDA Toolkit**: NVIDIA CUDA 13.1 (`/usr/local/cuda-13.1`)
- **Build System**: CMake >= 3.25, Ninja
- **Node.js**: Node >= 20, npm (for Web UI assets)
- **System Libraries**: `libevdev-dev`, `libdrm-dev`, `libcap-dev`, `libva-dev`, `libvulkan-dev`, `libpipewire-0.3-dev`, `libwayland-dev`, `libx11-dev`, `libayatana-appindicator3-dev`, `libnotify-dev`, `libssl-dev`, `libopus-dev`, `libpulse-dev`, `libsqlite3-dev`, `nlohmann-json3-dev`, `libboost-all-dev`.

---

## 2. Linux Compilation & Toolchain Fixes

The following fixes were implemented on branch `dev/agy`:

| File | Issue | Resolution |
|---|---|---|
| [`cmake/compile_definitions/linux.cmake`](../../cmake/compile_definitions/linux.cmake) | Obsolete glad v1 source paths in `PLATFORM_TARGET_FILES` | Removed obsolete glad files (glad v2 is generated dynamically into static libs). |
| [`src/nvenc/nvenc_config.h`](../../src/nvenc/nvenc_config.h) | `split_encode_mode` enum shadowed by member variable in GCC 14 (`-Wchanges-meaning`) | Renamed enum to `split_encode_mode_e` with `using split_encode_mode = split_encode_mode_e;` alias. |
| [`src/nvenc/nvenc_base.cpp`](../../src/nvenc/nvenc_base.cpp) | Direct `==` operator on plain C `GUID` struct failed on Linux | Replaced with `equal_guids(saved_init_params.encodeGUID, NV_ENC_CODEC_HEVC_GUID)`. |
| [`src/nvhttp.cpp`](../../src/nvhttp.cpp) | `has_stream_session_activity()` missing in non-Windows build | Defined `has_stream_session_activity()` under `#ifndef _WIN32`. |
| [`src/platform/common.h`](../../src/platform/common.h) | Missing `vulkan` enum value in `platf::mem_type_e` | Added `vulkan` to `platf::mem_type_e`. |
| [`src/platform/linux/host_stats.cpp`](../../src/platform/linux/host_stats.cpp) | Undeclared `_shutdown` variable in `nvml_t` | Replaced with member variable `nvmlShutdown`. |
| [`src/platform/linux/publish.cpp`](../../src/platform/linux/publish.cpp) | `platf::SERVICE_TYPE` (`std::string_view`) passed where `const char*` expected | Passed `platf::SERVICE_TYPE.data()`. |
| [`src/video.cpp`](../../src/video.cpp) | `encode_session_teardown_mutex` and `native_amf_lifecycle_gate` trapped inside `#ifdef _WIN32` | Moved declarations outside the Windows-only `#ifdef _WIN32` block. |
| [`tests/CMakeLists.txt`](../../tests/CMakeLists.txt) | Hardcoded `-lws2_32` link library broke Linux test linking | Set `${SUNSHINE_TEST_SOCKET_LIBRARIES}` to `ws2_32` on `WIN32` and empty on Linux. |
| [`tests/integration/test_locale_consistency.cpp`](../../tests/integration/test_locale_consistency.cpp) | Relative paths failed when CTest executed from `build/` directory | Added `find_repository_root()` parent-traversal helper. |

---

## 3. Build, Test & Package Commands

From the repository root (`/home/g3/Vibepollo`):

### A. Configure CMake
```bash
export PATH="/usr/local/cuda-13.1/bin:/usr/local/bin:/usr/bin:/bin:$PATH"
export CC=gcc-14
export CXX=g++-14

cmake -B build -G Ninja -S . \
  -DBUILD_TESTS=ON \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=/usr \
  -DSUNSHINE_ASSETS_DIR=share/sunshine \
  -DSUNSHINE_EXECUTABLE_PATH=/usr/bin/sunshine \
  -DSUNSHINE_ENABLE_DRM=ON \
  -DSUNSHINE_ENABLE_KWIN=ON \
  -DSUNSHINE_ENABLE_PORTAL=ON \
  -DSUNSHINE_ENABLE_WAYLAND=ON \
  -DSUNSHINE_ENABLE_X11=ON \
  -DSUNSHINE_ENABLE_CUDA=ON \
  -DCMAKE_CUDA_COMPILER:PATH=/usr/local/cuda-13.1/bin/nvcc \
  -DCMAKE_CUDA_HOST_COMPILER=gcc-14
```

### B. Build Web UI & Sunshine Binary (Memory Limited with `-j2`)
```bash
# Build Web UI assets first (if not already built)
ninja -C build web_ui

# Build core binary and test suite
ninja -C build -j2
```

### C. Run Test Suite
```bash
ctest --test-dir build --output-on-failure
```

### D. Generate Debian Packages
```bash
cpack -G DEB --config build/CPackConfig.cmake
```
Artifacts will be in `build/cpack_artifacts/`.

---

## 4. Fork Remote Configuration & Upstream Synchronization

### Git Remote Hierarchy
- **`origin`**: Your personal fork (`https://github.com/gabrielcruzg3/Vibepollo.git`)
- **`upstream`**: Direct upstream parent (`https://github.com/Nonary/Vibepollo.git`)
- **`upstream-apollo`**: Original Apollo repository (`https://github.com/ClassicOldSong/Apollo.git`)

### Syncing with Upstream (`Nonary/Vibepollo`)

To pull upstream changes and incorporate them into your branch:

```bash
# 1. Fetch latest commits from upstream
git fetch upstream

# 2. Update your local master branch
git checkout master
git merge upstream/master --ff-only

# 3. Push updated master to your GitHub fork
git push origin master

# 4. Rebase or merge your development branch onto updated master
git checkout dev/agy
git rebase master   # or: git merge master

# 5. Push updated dev branch to your fork
git push origin dev/agy
```

Alternatively using the GitHub CLI:
```bash
gh repo sync gabrielcruzg3/Vibepollo --source Nonary/Vibepollo
```
