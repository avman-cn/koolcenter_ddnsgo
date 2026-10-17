# Koolcenter ddns-go

华硕 / 梅林（koolcenter 软件中心）平台的 **DDNS-Go** 插件，把 [ddns-go](https://github.com/jeessy2/ddns-go) 内核封装成可在路由器 Web 界面里一键安装、启停和守护的插件。

- 项目地址 / 问题反馈：<https://github.com/avman-cn/koolcenter_ddnsgo>
- 更新日志：[Changelog.txt](Changelog.txt)
- DDNS-Go 内核（上游，本仓库不修改）：<https://github.com/jeessy2/ddns-go>

> 本项目只是 koolcenter 软件中心的**插件封装层**，DDNS 解析能力全部来自上游 `ddns-go` 内核，内核的仓库地址与版本发布均以上游为准。

---

## 功能特性

- **一键启停**：在软件中心界面开启 / 重启 / 关闭 DDNS-Go，无需登录 SSH。
- **实时进程守护**：基于 `perp` 守护内核进程，异常退出后自动拉起，并显示进程运行时间。
- **公网访问开关**：开启后自动写入 `notallowwanaccess: false`，并通过 `iptables` / `ip6tables` 放行面板端口（IPv4 + IPv6），NAT 重启后自动重新放行。
- **可选跳过证书验证**：老固件根证书过期时，可用 `-skipVerify` 启动内核。
- **可视化参数**：本地检查间隔、公网比对间隔次数、面板端口均可在页面中调整。
- **状态与日志**：实时显示运行状态、内核版本、插件日志。
- **开机自启**：安装时创建 `S99ddnsgo.sh` / `N99ddnsgo.sh` 软链，随系统启动 / 关机。
- **安全提示**：开启公网访问时务必为 DDNS-Go 后台设置足够复杂的用户名 / 密码。

## 适配平台

- 仅支持 **koolcenter 梅林改 / 官改** 固件（hnd / axhnd / axhnd.675x 平台）。
- 安装脚本会校验 `/koolshare` 目录、`/usr/bin/skipd` 以及内核版本（`uname -r` 主次版本号 ≥ 41），不满足则拒绝安装。
- 已适配 ASUSWRT / ROG / TUF 皮肤。

## 安装

1. 从 [Releases](https://github.com/avman-cn/koolcenter_ddnsgo/releases) 下载 `ddnsgo.tar.gz`（或自行构建，见下文）。
2. 打开路由器 **软件中心 → 离线安装**，上传该压缩包并安装。
3. 安装完成后进入 **软件中心 → DDNS-Go**，点击「开启」。
4. 点击「访问 DDNS-Go」进入内核面板，配置 DNS 服务商与域名解析规则（配置保存在 `/koolshare/configs/ddnsgo/default.yaml`）。

卸载：在软件中心卸载插件，或执行 `/koolshare/scripts/uninstall_ddnsgo.sh`。

## 使用说明

插件页面包含三块内容：

| 区域 | 说明 |
| --- | --- |
| 状态 | 运行状态、内核版本（实时读取 `ddnsgo -v`）、访问入口 |
| 设置 | 实时进程守护、开启公网访问、跳过证书验证、本地检查间隔（秒）、公网比对间隔次数、面板端口（默认 `9876`） |
| 操作 | 开启 / 重启 / 关闭，以及插件日志查看 |

内核实际启动参数形如：

```sh
ddnsgo -l :9876 -f 300 -cacheTimes 5 -skipVerify -c /koolshare/configs/ddnsgo/default.yaml
```

相关路径：

- 内核二进制：`/koolshare/bin/ddnsgo`
- 配置文件：`/koolshare/configs/ddnsgo/default.yaml`
- 插件日志：`/tmp/upload/ddnsgo_log.txt`
- 内核日志：`/tmp/upload/ddnsgo.log`

## 目录结构

```
.
├── config.json.js              # 软件中心插件描述（名称/版本/描述/入口/md5）
├── build.py                    # 打包脚本（Python 2），生成 ddnsgo.tar.gz 并回写 md5
├── Changelog.txt               # 插件更新日志
├── LICENSE
├── README.md
└── ddnsgo/                     # 插件本体，安装时整体拷贝到 /koolshare
    ├── bin/ddnsgo              # 上游 ddns-go 内核二进制（armv7）
    ├── res/icon-ddnsgo.png     # 插件图标
    ├── scripts/ddnsgo_config.sh# 启停 / 守护 / 防火墙 / 状态 / 版本 逻辑
    ├── webs/Module_ddnsgo.asp  # 软件中心插件页面
    ├── install.sh              # 安装脚本
    ├── uninstall.sh            # 卸载脚本
    └── version                 # 插件版本号
```

## 构建

需要 Python 2（脚本使用 `print u"..."` 语法）：

```sh
python build.py
```

脚本会：

1. 读取 `config.json.js`，把 `version` 写入 `ddnsgo/version`；
2. 打包为 `ddnsgo.tar.gz`；
3. 计算压缩包 md5 并回写到 `config.json.js` 的 `md5` 字段。

产物 `ddnsgo.tar.gz` 即可用于软件中心离线安装。

## 版本号规则

- 插件版本号格式为 `主版本.内核迭代.修订`（如 `1.29.0`），与内核版本号**相互独立**。
- 跟随上游内核发布更新时，将第二位 +1 并同步 `config.json.js`、`ddnsgo/version` 与 `Changelog.txt`。

## 致谢与许可

- DDNS 解析内核：[jeessy2/ddns-go](https://github.com/jeessy2/ddns-go)（MIT）
- 插件框架参考 koolcenter 软件中心插件规范
- 文档整理与维护脚本的部分工作由吃白饭的大肥鱼（deepseek）协助完成
- 本项目基于 [MIT License](LICENSE) 开源
