dsh-session-purge 1.0.0 发布：给已归档的对话一个真正的「删除」
> DSH 里对话可以归档，但归档只是从侧栏藏起来——日志、附件、缓存照样躺在硬盘上，越攒越多。
> 这个小插件在已归档对话的「⋯」菜单里加了一个**删除对话…**，一次把该删的删干净。
---
它解决什么问题
DSH 的会话存储（`session.v4.jsonl.zstd`）没有删除接口，工作区注册表也只支持归档。于是「不想要的对话」只能一直留着：一个跑过图片、文档、长对话的会话，动辄几 MB 到几十 MB，还连带附件对象、投影缓存、临时文件。时间一长，`~/.dsh` 就是几个 GB。
这个插件补上了缺失的那一环：归档之后，你可以真的把它删掉。
两个档位
档位	删什么	留什么
档 1 · 只删对话	会话日志目录、列表/摘要缓存、归档与置顶记账、所有以该会话 id 命名的 DSH 记录	附件、临时文件、对话产出的文件全部保留
档 2 · 完全删除	档 1 的全部内容，加上你在弹窗里勾选的产物：仅被该对话引用的附件对象、已无主的历史附件、spill 临时目录、对话交付过的文件、对话在项目目录里生成的文件	你没勾选的任何东西、其它会话的数据
档 2 是档 1 的严格超集——点档 2 就是一次删完，不需要先删档 1。
用起来是什么样
入口在侧栏：已归档的对话行 →「⋯」→「删除对话…」（就在「取消归档」下面）。没归档的对话不会出现这一项。
弹窗第一屏就列出每个文件的位置和占用大小，两档合计字节数一目了然。
档 2 里每一项产物都有复选框：`对话记录` 恒删，`对话产物` 逐项可勾。文档、图片、视频默认勾上；项目里的源码或不确定来源的文件默认不勾——勾选就是删除授权。
删完那一行立刻从侧栏消失，不用刷新、不用重启。
对话还在跑也不怕：插件会先停掉它的活动，只有日志文件真被系统占用时才提示稍后再试。
⚠️ 注意事项（请一定读）
删除不可撤销。 没有回收站，没有还原。档 2 尤其如此——按下之前请确认勾选列表。
档 1 会留下产物。 这是设计：附件可能被别的对话共用。想彻底清空就用档 2，或事后跑 `scripts/sweep-orphans.mjs` 清掉无主附件。
文件扫描只限会话自己的项目目录（JSONL 头里的 `cwd`，两层深）。它不会去扫 C 盘、不会碰 DSH 安装目录、不会删你的用户目录——日志里出现一个路径不等于拥有它。
默认勾选规则：对话自己在工具调用里点名过的文件 → 默认勾；仅「在会话期间生成」但没被点名的文档 → 列出但不勾；源码/脚本 → 不勾。
运行中的会话会先被停止再删除；如果日志仍被内核占用，会返回「稍后再试」，等几秒重试即可。
建议先 dry-run：`node scripts/show-plan.mjs <session-id>` 只打印清单、不删任何东西。
搜索索引（若启用了 session-query）由 DSH 下次扫描时自动对账清理；`llm-deepseek/files-v3.json` 这类上传缓存不归本插件管，会自行过期。
安装（三步）
方式一：压缩包（Windows 最省事）
解压 `dsh-session-purge-1.0.0.zip`；
在解压目录里运行：
```powershell
   powershell -NoProfile -ExecutionPolicy Bypass -File .\install.ps1
   ```
完全退出并重新打开 DSH（宿主插件和界面脚本都是启动时加载的）。
脚本会自动找到你的 DSH profile、复制文件、登记依赖与 bundle，改之前会备份 `package.json`。卸载用 `uninstall.ps1`，同样幂等。
方式二：tgz
```powershell
dsh plugin --profile desktop add "file:C:\path\to\dsh-session-purge-1.0.0.tgz"
```
方式三：手动 —— 把包放进 `<profile>\node_modules\dsh-session-purge`，在 `package.json` 的 `dependencies` 里加 `"dsh-session-purge": "file:./vendor/dsh-session-purge"`，并把 `dsh-session-purge` 追加到 `dsh.profile.bundles`，重启。
环境要求
DSH Desktop 0.2.x（web profile）；插件用的是 `webServer` / `workspaceRegistry` / slot 这些稳定接口。
安装脚本是 PowerShell，Windows 自带 5.1 即可；插件本体与脚本逻辑是纯 Node，macOS / Linux 下手动安装同样可用。
已验证
两套自检全过：档位语义与勾选子集、归档门禁、孤儿附件、脚本生成文档的识别、canary（项目外文件绝不入列）；
前端 bundle：slot 注册、未归档不出入口、中英文字典一致；
从 zip 解压 → 安装到全新 profile → 清单正确 → 卸载还原，端到端跑通；
真实数据 dry-run：只列项目目录内文件，不出现任何系统路径。
卸载
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\uninstall.ps1
```
移除依赖、bundle 与安装目录，不动你的会话数据。已删除的东西当然不会回来。
---
有问题、误删了不该删的、或者想要更多产物类型支持，欢迎反馈——误删案例我会优先修。
