# ALVR Linux fork (`alvr-linux-fix`)

個人 fork，給 **CachyOS + NVIDIA + SteamVR + Quest 3 有線** 使用。
官方 repo：<https://github.com/alvr-org/ALVR>

`alvr-linux-fix` = 官方 master + **PR #3425**（Linux direct mode / frame sync）+ `nv-codec-headers` pin。
`master` 保持乾淨，只用來對照官方。

比較：<https://github.com/Ency0810/ALVR/compare/master...alvr-linux-fix>

## 編譯

```bash
cd ~/ALVR
git checkout alvr-linux-fix
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
git checkout alvr-linux-fix
git merge upstream/master
cargo xtask build-streamer --release
```

PR #3425 合進官方後，把本地 commit 搬到新 upstream：

```bash
git rebase --onto upstream/master 0a778405 alvr-linux-fix
```

（`0a778405` 是目前本地的 PR #3425 merge commit。）

## 本地設定

- Bitrate：`~/.config/alvr/session.json` → `session_settings.video.bitrate.mode.ConstantMbps`（目前 `600`）。改完重開 dashboard。
- Quest client 版本不符時：`cargo xtask build-client --release`，APK 在 `build/alvr_client_quest/`。

## 這個分支改了什麼

1. PR #3425：Linux direct mode / texture / frame sync 修正。
2. `alvr/xtask/src/dependencies.rs`：`nv-codec-headers` `12.1.14.0` → `13.0.19.0`。
