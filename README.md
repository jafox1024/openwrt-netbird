# openwrt-netbird

OpenWrt package for [netbird](https://github.com/netbirdio/netbird)

## 分支与支持的 OpenWrt 版本

| 分支 | OpenWrt | 包管理器 / 包格式 | 目标架构 |
| --- | --- | --- | --- |
| `main` | 24.10.x | opkg / `.ipk` | x86_64, rockchip_armv8 |
| `openwrt-25.12` | 25.12.x（SDK 25.12.5） | apk / `.apk` | x86_64, rockchip_armv8 |

> OpenWrt 25.12 起包管理器由 **opkg 换成 apk**，`.ipk` 已停止使用，因此 25.12 上必须使用本分支产出的 `.apk` 包。

## 安装

下载 Release 中对应架构的 `.apk`（文件名带架构后缀，如 `netbird-0.78.1-r1-x86_64.apk`），拷到设备上安装：

```bash
apk add --allow-untrusted /tmp/netbird-0.78.1-r1-x86_64.apk
```

注意 `--allow-untrusted`：本包未加入 OpenWrt 官方签名链，属于本地/第三方包。

## 使用

用 setup key 登录 peer：

```bash
netbird login --setup-key <SETUP_KEY>
```

启动 netbird 后台服务：

```bash
/etc/init.d/netbird enable
/etc/init.d/netbird start
```

## 构建与发布（GitHub Actions）

流水线定义在 `.github/workflows/build.yml`，产物为 `.apk`：

1. `check` job：拉取 `netbirdio/netbird` 最新 release tag，比对本分支 `netbird/Makefile` 里的 `PKG_VERSION`；有更新就改版本号与 `PKG_HASH`、提交并推回 `openwrt-25.12` 分支。
2. `build` job：矩阵编译 `x86_64`、`rockchip_armv8` 两个架构（OpenWrt SDK 25.12.5），编译前用 [sbwml/packages_lang_golang](https://github.com/sbwml/packages_lang_golang) 的 `26.x`（Go 1.26）feed 替换官方 `lang/golang`——netbird 0.78.1 的 `go.mod` 要求 `go >= 1.26`。
3. 产物同时上传到 Actions Artifacts 和 Release（tag 形如 `v0.78.1-openwrt25.12`）。

手动构建：`Actions` → `build` → `Run workflow` → 选择 `openwrt-25.12` 分支。

### 关于定时自动更新

GitHub 只在**默认分支**上运行 `schedule`，所以 `openwrt-25.12` 分支上的 cron 不会触发。需要每日自动检查新版本时，在默认分支加一个调度入口即可：

```yaml
# .github/workflows/schedule-openwrt-25.12.yml（放在默认分支）
name: schedule-openwrt-25.12
on:
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:
permissions:
  actions: write
jobs:
  dispatch:
    runs-on: ubuntu-latest
    steps:
      - run: gh workflow run build.yml --ref openwrt-25.12
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 升级 OpenWrt SDK

改 `.github/workflows/build.yml` 里两处（`env.OPENWRT_RELEASE` / `env.OPENWRT_SERIES` 以及 matrix 中的 `sdk` 下载地址）即可。
