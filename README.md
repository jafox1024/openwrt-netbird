# openwrt-netbird

> **分支说明**：仓库默认分支 `main` 已是 OpenWrt **25.12 主线**（包管理器 apk，产物 `.apk`）。
> 本分支 `openwrt-24.10` 是 **24.10.x 备份线**（opkg，产物 `.ipk`，SDK 24.10.8），内容冻结、不再跟随上游自动更新；
> 需要重新出 24.10 的包时，在 Actions → `build` → `Run workflow` 里选择本分支手动运行即可。

OpenWrt package for [netbird](https://github.com/netbirdio/netbird)

## Usage

After installing the package, login the peer with setup key:

```bash
netbird login --setup-key <SETUP_KEY>
```

Start the netbird background service then you're good to go:

```bash
/etc/init.d/netbird enable
/etc/init.d/netbird start
```
