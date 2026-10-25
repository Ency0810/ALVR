# ALVR Linux 本地分支（`alvr-linux-fix`）

這個 fork 是給 **CachyOS / Arch + NVIDIA + SteamVR + Quest 3 有線** 用的。

`alvr-linux-fix` = 官方 `master` + **PR #3425**（Linux direct mode / texture / frame sync 修正）+ `nv-codec-headers` pin。

- 官方 repo：<https://github.com/alvr-org/ALVR>
- 本 fork：<https://github.com/Ency0810/ALVR>
- 比較頁：<https://github.com/Ency0810/ALVR/compare/master...alvr-linux-fix>

目前的 base：`adaa16ba`（2026-10-21）。

## 分支規則

| 分支 | 用途 |
|---|---|
| `master` | 乾淨的官方 `master`，**不要在這裡 commit** |
| `alvr-linux-fix` | 日常工作分支；所有本地修改都在這 |

## 這個分支改了什麼

1. **PR #3425**：Linux direct mode 的 texture 傳遞、resource cleanup、timeline synchronization 修正。
2. **`alvr/xtask/src/dependencies.rs`**：
   `nv-codec-headers` 從 `12.1.14.0` 改成 `13.0.19.0`。
   本機 FFmpeg 13.1 / 新 NVIDIA driver 需要這個版本，否則 NVENC 相關編譯會失敗。
3. `LOCAL.md`：這份說明。

## 第一次編譯（或依賴變更後）

```bash
cd ~/ALVR
git checkout alvr-linux-fix

# Arch / CachyOS 系統依賴
sudo pacman -S --needed base-devel clang curl nasm pkgconf yasm \
  vulkan-headers libva-mesa-driver unzip ffmpeg libpipewire git cmake ninja

# Rust 版本由 rust-toolchain.toml 指定，rustup 會自動安裝
cargo xtask prepare-deps --platform linux
cargo xtask build-streamer --release
```

- NVIDIA：**不要**加 `--no-nvidia`。
- AMD：`cargo xtask prepare-deps --platform linux --no-nvidia`。
- `prepare-deps` 會刪掉 `deps/linux` 重新下載／編譯（FFmpeg、x264、Vulkan headers），跑很久是正常的。
- 只有一般程式碼改動時，跳過 `prepare-deps`，直接跑 `build-streamer`。

輸出：

```text
build/alvr_streamer_linux/
├── bin/alvr_dashboard
└── lib64/alvr/          # SteamVR driver
```

啟動 dashboard：

```bash
cargo xtask run-streamer --release
# 或
./build/alvr_streamer_linux/bin/alvr_dashboard
```

## SteamVR driver

`~/.config/openvr/openvrpaths.vrpath` 的 `external_drivers` 要指向：

```text
/home/ency/ALVR/build/alvr_streamer_linux/lib64/alvr
```

如果 dashboard 顯示 driver 未安裝：

1. 開 dashboard。
2. 到 **Installation**。
3. 按 **Register ALVR driver**。

重新 build 不需要重新註冊（路徑不變）。

## 跟進官方 upstream

```bash
cd ~/ALVR
git fetch upstream
git checkout alvr-linux-fix
git merge upstream/master
```

有衝突就解，解完重新 build：

```bash
cargo xtask build-streamer --release
```

merge 前先確認狀態：

```bash
git status
git log --oneline -1 upstream/master
```

### 同步 `master`（可選）

`master` 只拿來對照官方，平常不用動。要同步：

```bash
git checkout master
git fetch upstream
git reset --hard upstream/master
git push origin master
git checkout alvr-linux-fix
```

### PR #3425 合進官方之後

如果 upstream 已經包含 #3425，本地那個 merge commit 就不需要了。
可以把「pin + 文件」這幾個 commit 直接搬到新的 upstream 上：

```bash
git fetch upstream
git checkout alvr-linux-fix
git rebase --onto upstream/master 0a778405 alvr-linux-fix
```

- `0a778405` 是目前本地的 PR #3425 merge commit。
- rebase 有衝突就解，然後 `git rebase --continue`。
- 完成後確認 pin 還在：

```bash
git log --oneline -3
git show --stat HEAD
```

如果 upstream 的 `dependencies.rs` 已經自己更新到 13.x，就不需要 pin，可以直接丟掉那個 commit。

## Quest client

Server 版本是 `21.0.0-dev14`。

如果 merge upstream 後 protocol 有變，dashboard 會提示 client 版本不符。重建 client：

```bash
cargo xtask build-client --release
# APK 在 build/alvr_client_quest/
adb install -r build/alvr_client_quest/*.apk
```

平常只改 streamer 不需要重裝 client。

## 本機執行設定（不在 git 裡）

- **Bitrate**：`~/.config/alvr/session.json`
  → `session_settings.video.bitrate.mode.ConstantMbps`
  目前是 `600`。改完要重開 dashboard / streamer。
- Dashboard GUI 的修改有時不會立刻寫回 `session.json`；如果沒生效，直接改檔案再重開。
- CBR 圖表：
  - 白線 = 實際 encoded bitrate（應接近 requested）
  - 彩色線 = peak throughput，較高是正常的
- SteamVR 啟動選項（NVIDIA + X11 Qt）：

  ```text
  __GLX_VENDOR_LIBRARY_NAME=nvidia __NV_PRIME_RENDER_OFFLOAD=1 VK_DRIVER_FILES=/usr/share/vulkan/icd.d/nvidia_icd.json QT_QPA_PLATFORM=xcb .../vrmonitor.sh %command%
  ```

## 常見問題

- **`prepare-deps` 卡在 FFmpeg / nv-codec-headers**
  確認 `alvr/xtask/src/dependencies.rs` 的 `codec_header_version` 是 `13.0.19.0`。
  必要時：

  ```bash
  rm -rf deps/linux
  cargo xtask prepare-deps --platform linux
  ```

- **SteamVR 說 driver 版本不符**
  重新 build 後重開 SteamVR；必要時到 dashboard → Installation → Register ALVR driver。

- **bitrate 一直停在 30 Mbps**
  檢查 `~/.config/alvr/session.json` 的 `ConstantMbps`，改完重開 dashboard。

- **關閉 SteamVR 時 compositor crash**
  目前已知，和遊戲中斷線無關。先看：

  ```bash
  tail -n 100 ~/.local/share/Steam/logs/vrcompositor-linux.txt
  ```

## 快速指令卡

```bash
# 更新 + 重建
cd ~/ALVR
git fetch upstream
git checkout alvr-linux-fix
git merge upstream/master
cargo xtask build-streamer --release

# 只重建
cargo xtask build-streamer --release

# 看本地改了什麼
git log --oneline upstream/master..alvr-linux-fix
git diff upstream/master...alvr-linux-fix --stat
```
