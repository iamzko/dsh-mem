搜索文件用 fdfind（fd），比 find 高效；文件名/路径模式匹配优先 fdfind，内容搜索用 rg。
§
从 GitHub 下载任何东西，链接一律加代理前缀 https://gh-proxy.com/ —— 即 https://github.com/xxx 写成 https://gh-proxy.com/https://github.com/xxx（raw.githubusercontent.com 同理）。实测：直连 github 超时(HTTP 000)，该前缀直连可用(HTTP 200, ~3.8s)，无需依赖本机 127.0.0.1:7897。
§
OfficeCLI v1.0.160（.NET 单二进制，无依赖）已装于 ~/.local/bin/officecli，技能在 ~/.workbuddy/skills/officecli/SKILL.md；~/.dsh/skills 是 ~/.workbuddy/skills 的符号链接，一份落盘 DSH 与 WorkBuddy 同时生效。scan/渲染/截图均本地完成，create 时无外网出站。
§
局域网计算节点 192.168.10.7（fedora-keda：i7-7700 4C8T / 31G / 无 CUDA / gcc16 / 无 node·docker），已用专用密钥 ~/.ssh/rc_10_7 免密接入，工作区 ~/rc/work。统一入口技能 lan-compute-offload（~/.dsh/skills/lan-compute-offload，与 ~/.workbuddy/skills 同目录）。其外网经本机 WiFi NAT（scripts/setup-nat.sh，规则不持久需 root 重放）；本机 agent 沙箱 no_new_privileges 导致无法执行本机 sudo。
§
dsh 插件安装：`dsh plugin --profile web add <pkg>[@ver]` 靠 PATH 解析 pnpm。本机 pnpm 由 npm 全局提供（/home/kai/.npm-global/bin/pnpm 11.21.0，另有 ~/.local/share/pnpm/bin 副本）；dsh-web.service 的 unit 已显式声明 PATH，不再需要手动前置。装完必须核三件事：package.json dependencies 版本、node_modules 实际版本、dsh.profile.bundles 是否登记该包——安装中断时只留 dependencies 条目、bundles 未登记则插件不生效。更新时看 "Packages: +N -M"，M 是传递依赖瘦身，不等于顶层丢包，按顶层依赖清单核对。
§
DSH Web UI 由 systemd 用户服务 dsh-web.service 托管；unit 真身已入库 ~/agent-home/systemd/dsh-web.service（2026-11-09），安装位 ~/.config/systemd/user/dsh-web.service 是指向它的符号链接（与 agent-home-sync 同模式），改配置直接编辑仓内文件。ExecStart=/home/kai/.npm-global/bin/dsh web --no-open，127.0.0.1:3080，Restart=on-failure。注意：unit 里的 Environment= 只对新进程生效，改完 daemon-reload 后仍需 restart 才进入运行中的服务。profile 目录（~/.dsh/profiles/desktop，web 为其符号链接）的 package.json 改动后客户端资产 F5 即从磁盘重读；服务端 host runtime 是否换新无热加载证据，稳妥做法是重启一次。
§
插件市场报 "spawn pnpm ENOENT / pnpm was not found" 的根因（2026-11-09 复核）：systemd 用户服务 dsh-web.service 的 PATH 只有系统默认 9 目录，而 pnpm 与 dsh 都装在用户级 npm 全局前缀 ~/.npm-global/bin（pnpm 11.21.0）。已在该 unit 显式加 Environment=PATH=/home/kai/.npm-global/bin:/home/kai/.local/share/pnpm/bin:...，daemon-reload 已做，需重启服务才进入运行中进程。旧方案（/home/kai/node_modules/.bin/pnpm 转发垫片）已失效并删除：ExecStart 由 npx 改为绝对路径 /home/kai/.npm-global/bin/dsh 后不再注入 node_modules/.bin。UI 里装插件仍报错时，先在正确 PATH 下用 CLI 装。
§
已在 desktop profile 安装 dsh-folder-explorer 0.2.1（npm 包，registry 依赖，VSCode 式文件树），安装后 dsh plugin 自动完成 dsh.profile.bundles 注册，包变更触发服务热加载，浏览器 F5 即生效；无需重启 dsh-web。
§
DSH 验证配置是否被 loader 采纳：`dsh --profile desktop --dump-config` 会报 "profile desktop is managed exclusively by the Electron application"；Web 服务实际跑 profile web（desktop 的符号链接），改用 `npx --yes @deepseek-ai/dsh --profile web --dump-config` 可导出完整组装 YAML（约 4700 行，含 !!js 标签），grep 目标字段即可确认 cordis.patch.yml 改动生效。
§
nemo 右键「在终端打开」总是落主目录的根因在 GNOME Terminal 侧，不在 nemo：profile 键 org.gnome.Terminal.Legacy.Profile:/org/gnome/terminal/legacy/profiles:/:b1dcc9dd-5262-4d8d-a863-c897e6d979b9/ preserve-working-directory 被设为 never（schema 默认 safe），新窗口不保留发起进程 cwd。nemo 6.4.5 src/nemo-view.c:7131 open_in_terminal() 用 g_spawn_async(path, argv=[exec], ...) 已正确传目录。修复：gsettings set 该键 'safe'（2026-11-09 已改并验证）。
§
局域网 NAS 在 192.168.10.7（fedora-keda，登录 keda）：sda+sdb 各 1TB 组 btrfs RAID1 挂 /srv/nas（可用 932G，Data/System/Metadata 全 RAID1），sdc 512G 为系统盘；服务 Samba(445/139)、Cockpit(9090)、krdp(3389)、ZeroTier 常驻；每月 1 号 03:00±1h 自动 btrfs scrub（btrfs-scrub-nas.timer）。
