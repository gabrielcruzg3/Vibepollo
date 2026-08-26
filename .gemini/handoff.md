# Vibepollo Build & Maintenance Handoff

## 1. Project & Repository Overview

- **Repository**: [gabrielcruzg3/Vibepollo](https://github.com/gabrielcruzg3/Vibepollo) (Fork of [Nonary/Vibepollo](https://github.com/Nonary/Vibepollo))
- **Active Branch**: `dev/agy` (tracks `origin/dev/agy`)
- **Current Version**: `1.19.0-beta.3-agy` (Upstream baseline: `1.19.0-beta.3` — **Latest Upstream Release**)
- **Published Releases**:
  - [`1.19.0-beta.3-agy`](https://github.com/gabrielcruzg3/Vibepollo/releases/tag/1.19.0-beta.3-agy) (Upstream `1.19.0-beta.3` — **Latest**)
  - [`1.19.0-beta.2-agy`](https://github.com/gabrielcruzg3/Vibepollo/releases/tag/1.19.0-beta.2-agy) (Upstream `1.19.0-beta.2`)
  - [`1.19.0-beta.1-agy`](https://github.com/gabrielcruzg3/Vibepollo/releases/tag/1.19.0-beta.1-agy) (Upstream `1.19.0-beta.1`)
  - [`1.19.0-alpha.2-agy`](https://github.com/gabrielcruzg3/Vibepollo/releases/tag/1.19.0-alpha.2-agy) (Upstream `1.19.0-alpha.2`)
  - [`1.19.0-alpha.1-agy`](https://github.com/gabrielcruzg3/Vibepollo/releases/tag/1.19.0-alpha.1-agy) (Upstream `1.19.0-alpha.1`)
  - [`v1.18.4-agy.1`](https://github.com/gabrielcruzg3/Vibepollo/releases/tag/v1.18.4-agy.1) (Upstream `1.18.4-stable.2`)
- **Local Directory**: `/home/g3/Vibepollo`

---

## 2. Upgrade & Synchronization Log

### Stepwise Upstream Progression
Our fork followed upstream release tags step-by-step using the naming convention `{originalTag}-agy`:

| Version / Tag | Base Upstream Tag | Merge Status | Test Suite | Release Status |
|---|---|---|---|---|
| `v1.18.4-agy.1` | `1.18.4-stable.2` | Initial fork base | 33/33 Passed | **Published** |
| `1.19.0-alpha.1-agy` | `1.19.0-alpha.1` | Clean Merge + Locale Contract Fix | 36/36 Passed (100%) | **Published** |
| `1.19.0-alpha.2-agy` | `1.19.0-alpha.2` | Clean Merge | 36/36 Passed (100%) | **Published** |
| `1.19.0-beta.1-agy` | `1.19.0-beta.1` | Clean Merge + Config Catalog Mapping | 36/36 Passed (100%) | **Published** |
| `1.19.0-beta.2-agy` | `1.19.0-beta.2` | Clean Merge + Linux Lossless Header Fix + Config Catalog Mapping | 36/36 Passed (100%) | **Published** |
| **`1.19.0-beta.3-agy`** | **`1.19.0-beta.3`** | **Clean Merge (Latest Upstream Tag Reached!)** | **36/36 Passed (100%)** | **Published (Latest)** |

---

## 3. Technical Summary of Applied Fixes

| Modified File | Root Cause & Issue | Resolution Applied |
|---|---|---|
| [src/process.cpp](file:///home/g3/Vibepollo/src/process.cpp#L70-L75) | In `1.19.0-beta.2`, upstream called `playnite_launcher::lossless::policy::should_enable_runtime` in `src/process.cpp` cross-platform, but `#include "tools/playnite_launcher/lossless_scaling_policy.h"` was guarded inside `#ifdef _WIN32`. | Moved `#include "tools/playnite_launcher/lossless_scaling_policy.h"` outside `#ifdef _WIN32` so Linux builds resolve the policy function cleanly. |
| [docs/configuration.md](file:///home/g3/Vibepollo/docs/configuration.md) & [en.json](file:///home/g3/Vibepollo/src_assets/common/assets/web/public/assets/locale/en.json) | In `1.19.0-beta.2`, upstream added `remote_monitor_terminate_on_first_request` to `src/config.cpp` and `ui/en.json`, but omitted it from `docs/configuration.md` and the `config` dictionary in `en.json`. | Documented `remote_monitor_terminate_on_first_request` in `docs/configuration.md` and added label to `en.json`. |
| [docs/configuration.md](file:///home/g3/Vibepollo/docs/configuration.md) & [en.json](file:///home/g3/Vibepollo/src_assets/common/assets/web/public/assets/locale/en.json) | In `1.19.0-beta.1`, upstream added `remote_monitor_mute_audio`, `remote_monitor_disconnect_on_stream_end`, and `remote_monitor_disconnect_on_client_disconnect` to `src/config.cpp` and `ui/en.json`, but omitted them from `docs/configuration.md` and the `config` dictionary in `en.json`. | Documented all three options in `docs/configuration.md` and added their labels to `en.json`, passing `test_component_resource_config_catalog` (36/36 tests passing). |
| [src_assets/.../en.json](file:///home/g3/Vibepollo/src_assets/common/assets/web/public/assets/locale/en.json#L946-L950) | In `1.19.0-alpha.1`, upstream added `rtss_allow_virtual_display_override` to `src/config.cpp` and `ui/en.json`, but omitted it from the `config` dictionary in `en.json`. | Added `"rtss_allow_virtual_display_override": "Allow non-Reflex RTSS modes on virtual displays"` under `"config"` in `en.json`. |
| [cmake/compile_definitions/linux.cmake](file:///home/g3/Vibepollo/cmake/compile_definitions/linux.cmake#L359-L373) | Obsolete glad v1 source file paths in `PLATFORM_TARGET_FILES`; duplicate entry with syntax error. | Cleaned up `PLATFORM_TARGET_FILES` to only reference existing platform sources. Glad v2 is dynamically generated into static libs via [glad.cmake](file:///home/g3/Vibepollo/cmake/dependencies/glad.cmake). |
| [cmake/packaging/linux.cmake](file:///home/g3/Vibepollo/cmake/packaging/linux.cmake#L69-L72) | `CPACK_DEB_COMPONENT_INSTALL` was hardcoded to `ON`, preventing single standalone package creation. | Added conditional `if(NOT DEFINED CPACK_DEB_COMPONENT_INSTALL)` so monolithic packaging can be enabled with `-DCPACK_DEB_COMPONENT_INSTALL=OFF`. |
| [src/nvenc/nvenc_config.h](file:///home/g3/Vibepollo/src/nvenc/nvenc_config.h#L15-L20) | In GCC 14, `split_encode_mode` enum name collided with struct member variable under `-Wchanges-meaning`. | Renamed `enum class split_encode_mode` to `split_encode_mode_e` with type alias `using split_encode_mode = split_encode_mode_e;`. |
| [src/nvenc/nvenc_base.cpp](file:///home/g3/Vibepollo/src/nvenc/nvenc_base.cpp#L1083) | `saved_init_params.encodeGUID == NV_ENC_CODEC_HEVC_GUID` failed because `GUID` is a plain struct on Linux without `operator==`. | Used existing `equal_guids()` helper for cross-platform GUID comparison. |
| [src/nvhttp.cpp](file:///home/g3/Vibepollo/src/nvhttp.cpp#L1184-L1230) | `has_stream_session_activity()` was omitted on Linux under `#ifndef _WIN32` (named `has_stream_session_activity_for_http_probe`). | Renamed function to `has_stream_session_activity()` so HTTP launch/resume endpoints can resolve it across all platforms. |
| [src/platform/common.h](file:///home/g3/Vibepollo/src/platform/common.h#L223-L231) | Missing `vulkan` member in `platf::mem_type_e`, causing build errors in Linux KMS and Pipewire grabbers. | Added `vulkan` to `platf::mem_type_e`. |
| [src/platform/linux/host_stats.cpp](file:///home/g3/Vibepollo/src/platform/linux/host_stats.cpp#L301-L330) | `_shutdown` was undeclared in `nvml_t` after refactoring. | Updated `reset()` and initialization in `nvml_t` to properly use member function pointer `nvmlShutdown`. |
| [src/platform/linux/publish.cpp](file:///home/g3/Vibepollo/src/platform/linux/publish.cpp#L348) | Passed `platf::SERVICE_TYPE` (`std::string_view`) where Avahi's C API expected `const char *`. | Passed `platf::SERVICE_TYPE.data()`. |
| [src/video.cpp](file:///home/g3/Vibepollo/src/video.cpp#L240-L245) | `encode_session_teardown_mutex` and `native_amf_lifecycle_gate` were nested inside an unclosed `#ifdef _WIN32` block. | Moved both declarations outside the Windows-only block so cross-platform teardown paths can access them. |
| [tests/CMakeLists.txt](file:///home/g3/Vibepollo/tests/CMakeLists.txt#L183-L188) | Standalone test targets unconditionally linked Windows socket library `-lws2_32`. | Guarded socket linking under `if(WIN32)` via `${SUNSHINE_TEST_SOCKET_LIBRARIES}`. |
| [tests/integration/test_locale_consistency.cpp](file:///home/g3/Vibepollo/tests/integration/test_locale_consistency.cpp#L23-L30) | Test failed when run from `build/` via CTest due to relative paths to `src/config.cpp` and locale assets. | Added `find_repository_root()` parent-traversal helper. |

---

## 4. Build, Test & Packaging Commands

From `/home/g3/Vibepollo`:

### Step 1: Configure CMake
```bash
export PATH="/usr/local/cuda-13.1/bin:/usr/local/bin:/usr/bin:/bin:$PATH"
export CC=gcc-14
export CXX=g++-14
export BUILD_VERSION=1.19.0-beta.3-agy

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

### Step 2: Build Web UI & Sunshine Binaries
```bash
# 1. Build Vue/Vite frontend assets
ninja -C build web_ui

# 2. Build core binaries and test targets with memory limit (peak ~8-10 GB)
ninja -C build -j2
```

### Step 3: Run Full Test Suite (100% Passing)
```bash
ctest --test-dir build --output-on-failure
# 100% tests passed, 0 tests failed out of 36
```

### Step 4: Generate `.deb` Packages
```bash
mkdir -p build/cpack_artifacts

# Generate standalone (all-in-one) package
cmake -B build -DCPACK_DEB_COMPONENT_INSTALL=OFF
cpack -G DEB --config build/CPackConfig.cmake
mv build/cpack_artifacts/Vibepollo.deb build/cpack_artifacts/Vibepollo-standalone-1.19.0-beta.3-agy.deb

# Generate split packages (core & web assets separately)
cmake -B build -DCPACK_DEB_COMPONENT_INSTALL=ON
cpack -G DEB --config build/CPackConfig.cmake
mv build/cpack_artifacts/Vibepollo-Unspecified.deb build/cpack_artifacts/Vibepollo-core-1.19.0-beta.3-agy.deb
mv build/cpack_artifacts/Vibepollo-assets.deb build/cpack_artifacts/Vibepollo-web-1.19.0-beta.3-agy.deb
```

---

## 5. Release Packages (`1.19.0-beta.3-agy`)

Located in `build/cpack_artifacts/` and published on [GitHub Releases](https://github.com/gabrielcruzg3/Vibepollo/releases/tag/1.19.0-beta.3-agy):

| Artifact Name | Size | Contents |
|---|---|---|
| **`Vibepollo-standalone-1.19.0-beta.3-agy.deb`** | `19.8 MB` | Complete standalone installer containing core binary (`sunshine-1.19.0-beta.3-agy`), Web UI assets, systemd units, icons, and shaders. |
| **`Vibepollo-core-1.19.0-beta.3-agy.deb`** | `14.8 MB` | Backend executable and system integration only (no Web UI assets). |
| **`Vibepollo-web-1.19.0-beta.3-agy.deb`** | `5.0 MB` | Frontend Web UI assets only (`/usr/share/sunshine/web/`). |

---

## 6. How to Configure Vibepollo to Run Unattended (Headless / On Boot)

### Architecture Overview: NVIDIA X11 & Screen Capture
On Linux systems using NVIDIA proprietary drivers under X11:
- **KMS Capture Limitation**: Xorg takes exclusive DRM Master access of `/dev/dri/card1`. Any attempt by Sunshine to use KMS capture before an X11 user session starts fails with `Couldn't find monitor [0]` (Error 503).
- **NvFBC Capture Requirement**: NVIDIA hardware-accelerated screencasting on X11 operates exclusively via **NvFBC**, which requires an active X11 display session (`DISPLAY=:0`) and session authorization.
- **The Production Architecture**: To achieve unattended streaming securely without requiring a physical monitor or leaving an unauthenticated desktop:
  1. Configure **SDDM Auto-login to X11 (`plasmax11`)** so Xorg and NvFBC initialize on boot.
  2. Configure an **Instant Lock Screen Autostart** so KDE Plasma locks immediately upon login.
  3. Configure a **Headless Virtual Display** so the NVIDIA GPU maintains a 1080p display buffer even when no monitor is physically connected or powered on.

---

### Step 1: Add User to Hardware Groups
Ensure your user has direct access to `/dev/uinput` and `/dev/dri/*`:
```bash
sudo usermod -aG input,video,render $USER
```

### Step 2: Grant Full KMS & EGL Capture Capabilities
Grant `cap_sys_admin` and `cap_sys_nice` to the Sunshine executable:
```bash
# For installed system package:
sudo setcap cap_sys_admin,cap_sys_nice+p $(readlink -f $(which sunshine))

# For local development build:
sudo setcap cap_sys_admin,cap_sys_nice+p $(readlink -f /home/g3/Vibepollo/build/sunshine)
```

### Step 3: Configure User Service for Graphical Session
Sunshine requires `DISPLAY=:0` and session Xauthority to initialize **NvFBC**. Binding Sunshine to `graphical-session.target` ensures it starts automatically the instant SDDM completes autologin with the full X11 environment ready:

```bash
# 1. Clean up any lingering system-level unit
sudo systemctl stop sunshine 2>/dev/null || true
sudo systemctl disable sunshine 2>/dev/null || true
sudo rm -f /etc/systemd/system/sunshine.service
sudo systemctl daemon-reload

# 2. Configure systemd user service override for graphical-session.target
mkdir -p ~/.config/systemd/user/app-dev.lizardbyte.app.Sunshine.service.d
cat > ~/.config/systemd/user/app-dev.lizardbyte.app.Sunshine.service.d/override.conf << 'EOF'
[Unit]
After=graphical-session.target xdg-desktop-autostart.target
PartOf=graphical-session.target

[Service]
Restart=on-failure
RestartSec=3s
EOF

# 3. Reload and enable the user service
systemctl --user daemon-reload
systemctl --user reenable app-dev.lizardbyte.app.Sunshine.service
systemctl --user restart sunshine.service
```

---

### Step 4: Configure SDDM Auto-Login (X11)
Configure SDDM to automatically launch the KDE Plasma X11 session (`plasmax11.desktop`). **Do not use inline `#` comments** in `/etc/sddm.conf`:

```bash
sudo tee /etc/sddm.conf << 'EOF'
[Autologin]
User=g3
Session=plasmax11
EOF
```

---

### Step 5: Configure Instant Screen Lock on Boot (Security)
To prevent physical access to your desktop when the host boots unattended, add an autostart entry that immediately locks the session upon login:

```bash
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/lock-screen.desktop << 'EOF'
[Desktop Entry]
Type=Application
Name=Lock Screen on Boot
Exec=loginctl lock-session
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
EOF
```
*Result*: The system boots, logs in, instantly locks the screen, and Sunshine initializes NvFBC. When you connect via Moonlight, you see the lock screen and enter your password remotely.

---

### Step 6: NVIDIA Headless & Virtual Dual-Display Setup (Extended Desktop)
To enable both a primary headless monitor and a virtual extended second screen on NVIDIA X11:

```bash
# 1. Extract EDID binary from active display (or use existing EDID)
sudo cp /sys/class/drm/card1-HDMI-A-1/edid /etc/X11/edid.bin

# 2. Create NVIDIA dual-screen Xorg configuration
sudo tee /etc/X11/xorg.conf.d/10-headless.conf << 'EOF'
Section "ServerLayout"
    Identifier     "Layout0"
    Screen      0  "Screen0"
EndSection

Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    Option         "AllowEmptyInitialConfiguration" "True"
    Option         "ConnectedMonitor" "DFP-0, DFP-1"
    Option         "CustomEDID" "DFP-1:/etc/X11/edid.bin"
    Option         "HardDPMS" "false"
EndSection

Section "Screen"
    Identifier     "Screen0"
    Device         "Device0"
    Monitor        "Monitor0"
    DefaultDepth    24
    Option         "MetaModes" "DFP-0: 1920x1080 +0+0, DFP-1: 1920x1080 +1920+0"
    SubSection     "Display"
        Depth       24
        Modes      "1920x1080"
    EndSubSection
EndSection

Section "Monitor"
    Identifier     "Monitor0"
    VendorName     "Unknown"
    ModelName      "DualScreen"
    Option         "DPMS" "false"
EndSection
EOF

# 3. Remove any obsolete monolithic /etc/X11/xorg.conf
sudo rm -f /etc/X11/xorg.conf
```

> [!WARNING]
> **Disclaimer on Session Reloads & Reboots**:
> Removing `/etc/X11/xorg.conf` or altering multi-monitor layouts while an active KDE Plasma/X11 desktop session is running can cause Plasma or KWin to hang if only SDDM is restarted mid-session (`sudo systemctl restart sddm`). Always perform a full system reboot (`sudo reboot`) to cleanly initialize the virtual display pipelines and auto-login smoothly.

---

### Step 7: Configuring Multi-Display Streaming in Sunshine Web UI

Once the virtual dual-display configuration is active, Sunshine detects:
- **`HDMI-0 (id: 0)`**: Primary display (+0+0)
- **`DP-0 (id: 1)`**: Virtual extended secondary display (+1920+0)

In Sunshine Web UI (`https://<HOST-IP>:47990` -> **Applications**):
- **Desktop App (Primary Screen)**: Uses default (`id: 0`).
- **Second Monitor App (Extended Screen)**:
  - Create or edit an application named `Second Monitor`.
  - Under **Setting Overrides** -> select **`Display Id`** (key: `output_name`).
  - Set value to `1`.
  - Save.

---

### Step 8: Service Management Commands

| Operation | Command |
|---|---|
| **Check Status** | `systemctl --user status sunshine` |
| **Follow Live Logs** | `journalctl --user -u sunshine -f` |
| **Restart Service** | `systemctl --user restart sunshine` |
| **Stop Service** | `systemctl --user stop sunshine` |
| **Web UI Access** | `https://<HOST-IP>:47990` |

