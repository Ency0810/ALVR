# 分支說明

這個 repo 只有兩條長期分支，分工固定：

| 分支 | 是什麼 | 能不能 commit |
|---|---|---|
| **`linux-fix`** | **我的修改**（預設分支）—— Linux / NVENC / xtask 相關的本地修正 | ✅ 只在這條上開發 |
| **`official`** | **官方 ALVR 的鏡像**（`alvr-org/ALVR:master`），純備用對照 | ❌ 只做同步，不要在上面開發 |

兩條分支的關係：`linux-fix` 從 `official` 長出來，目前比它多 21 個 commit。

## 遠端

| remote | 指向 |
|---|---|
| `origin` | `github.com/Ency0810/alvr-linux-fix`（我的 fork） |
| `upstream` | `github.com/alvr-org/ALVR`（官方） |

## 常用指令

更新官方鏡像（`official` 永遠等於官方 master）：

```bash
git fetch upstream
git branch -f official upstream/master
git push origin official
```

把修改接到最新官方上：

```bash
git fetch upstream
git rebase upstream/master linux-fix
```

送 PR 給官方：從 `linux-fix` 對 `alvr-org/ALVR:master` 開 PR。

## 其他本地分支

`pr-3342`、`pr-3425` 是舊的 PR 工作分支，未併入官方，僅存在本機、沒有推到遠端。
