# openwrt-netbird

[![GitHub Actions](https://github.com/jafox1024/openwrt-netbird/workflows/build/badge.svg)](https://github.com/jafox1024/openwrt-netbird/actions?query=workflow%3Abuild)

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
