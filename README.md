# dsh-session-purge-1.0.0
用于dsh的会话归档后的删除和产物清理


安装说明（3 步）
第 1 步：解压
把压缩包解压到任意位置（例如 `D:\\Tools\\dsh-session-purge-1.0.0`）。路径里最好不要有空格。
第 2 步：安装
在本文件夹里按住 `Shift` 右键 → 在此处打开 PowerShell 窗口，然后执行：
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\\install.ps1
```
看到「安装完成 ✅」即可。脚本会自动：
把插件复制到 `<DSH\_HOME>\\profiles\\desktop\\vendor\\dsh-session-purge` 和 `node\_modules\\dsh-session-purge`；
在 profile 的 `package.json` 里登记依赖与 `dsh.profile.bundles`；
备份原 `package.json`（同名 `.bak-时间戳`），装错了可以还原。
如果你的 profile 不叫 `desktop`，加参数：`-Profile 你的profile名`。
第 3 步：重启 DSH
完全退出 DSH 桌面端（不是关窗口，是退出进程）再打开。插件在启动时加载，不重启看不到入口。
---
怎么用
侧栏里把要删的对话归档（视图切到「已归档」能看到它）；
点该行右侧「⋯」，在「归档」下面点 删除对话…；
选档位：
档位	效果
档 1 · 仅删除对话记录	只删日志、列表缓存、归档/置顶记账；附件和生成的文件都留着
档 2 · 完全删除	删掉档 1 的全部内容，再加上这个对话产生的附件、临时文件、交付文件、项目目录里它生成的文件
档 2 会逐项列出产物，可以取消勾选不想删的；
勾「我明白此操作不可撤销」→ 确认。该行立即消失，硬盘空间立刻释放。
常见问题
菜单里没有「删除对话…」？
没重启，或重启的是窗口不是进程；
这个对话还没归档——插件只对已归档的对话显示入口。
提示「还在运行」？
日志仍被一个进行中的回合占用。等它结束，或者先归档（归档会停掉当前回合）再删。
Word/Excel 没出现在产物列表里？
只扫描该对话的项目目录（`cwd`）下、会话期间新建的文件。对话如果是在别的工作目录里跑的，文件不在扫描范围。
档 1 删完才发现还有附件残留？
再对同一个对话跑一次档 2 已经不可能（行没了）。可以直接清孤儿附件：
```powershell
node .\\dsh-session-purge\\scripts\\sweep-orphans.mjs --home "$env:USERPROFILE\\.dsh"          # 预览
node .\\dsh-session-purge\\scripts\\sweep-orphans.mjs --home "$env:USERPROFILE\\.dsh" --apply  # 真删
```
卸载
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\\uninstall.ps1
```
卸载只移除插件本身，已经删掉的对话不会恢复。
安全说明
只删能证明属于该对话的东西：会话日志目录、以会话 id 命名的记录、只有它引用的附件（按 sha256 引用计数）、它 `present` 过的文件、它在项目目录里新建的文件。
不碰项目源码里的原有文件、不碰全局日志、不碰别的对话的附件。
每个待删路径都校验父目录，拒绝盘根、`C:\\Windows`、`Program Files` 与 `..` 逃逸。
