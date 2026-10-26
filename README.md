# ALVR Linux fork (`alvr-linux-fix`)

個人 fork，給 **CachyOS + NVIDIA + SteamVR + Quest 3 有線** 使用。
官方 repo：<https://github.com/alvr-org/ALVR>

`linux-fix` = 官方 `official` + **PR #3425**（Linux direct mode / frame sync）+ `nv-codec-headers` pin。
`official` 是官方 master 的鏡像，保持乾淨，只用來對照。

比較：<https://github.com/Ency0810/alvr-linux-fix/compare/official...linux-fix>

## 預編譯版本

不想自己編譯的話，直接抓 [Releases](https://github.com/Ency0810/alvr-linux-fix/releases)：

- `alvr_streamer_linux.tar.gz` — Linux streamer
- `alvr_client_android.apk` — Quest client

**streamer 與 APK 必須來自同一個 release**（版本 `21.0.0-dev14`，與官方 `20.14.1` 不相容）。
APK 用 debug key 簽章，安裝前要先解除安裝官方版本。

## 編譯

```bash
cd ~/ALVR
git checkout linux-fix
cargo xtask build-streamer --release
```

第一次編譯或依賴變更後，先跑（會重新編譯 FFmpeg，很久）：

```bash
cargo xtask prepare-deps --platform linux
```

輸出在 `build/alvr_streamer_linux/`，dashboard 是 `bin/alvr_dashboard`。

## 跟進官方

```bash
git fetch upstream
git checkout linux-fix
git merge upstream/master
cargo xtask build-streamer --release
```

PR #3425 合進官方後，把本地 commit 搬到新 upstream：

```bash
git rebase --onto upstream/master 0a778405 linux-fix
```

（`0a778405` 是目前本地的 PR #3425 merge commit。）

## 本地設定

- Bitrate：`~/.config/alvr/session.json` → `session_settings.video.bitrate.mode.ConstantMbps`（目前 `600`）。改完重開 dashboard。
- Quest client 版本不符時：`cargo xtask build-client --release`，APK 在 `build/alvr_client_quest/`。

## 這個分支改了什麼

1. PR #3425：Linux direct mode / texture / frame sync 修正。
2. `alvr/xtask/src/dependencies.rs`：`nv-codec-headers` `12.1.14.0` → `13.0.19.0`。

## 小技巧：NVENC header 版本

```bash
# 目前 driver
nvidia-smi --query-gpu=driver_version --format=csv,noheader

# 目前 pin 的 headers 要求
curl -fsSL https://raw.githubusercontent.com/FFmpeg/nv-codec-headers/n13.0.19.0/README | grep -A2 "Minimum required"
```

`13.0.19.0` 需要 driver **570.0+**；headers 不能比 driver 新，比 driver 舊通常可以。
查其他版本就把 `n13.0.19.0` 換成對應 tag。
