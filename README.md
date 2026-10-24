# PCL2 补丁注入工具

恢复 PCL2 官方 2.99.x 被禁用的「下载」与「整合包安装」入口——就地注入 IL 补丁，自动备份原文件。
同时内置 **GitHub 更新下载**：官方更新服务器已关闭，可一键从官方仓库获取最新版 PCL2。

## 功能特点

- **单文件免安装**：内嵌 dnlib 与参考版实现，空目录双击即用
- **就地注入**：选中你的 `Plain Craft Launcher 2.exe`，点「注入补丁」即可，自动备份为 `.bak`
- **自动检测**：程序版本 / 是否 PCL2 / 是否已注入 / 能否注入
- **安全写入**：临时文件 + 原子替换，失败不损坏原文件；已注入文件不会重复修改
- **GitHub 下载最新 PCL2**：官方更新服务器关闭后，工具内直接下载官方仓库 `最新正式版.zip`
  （jsDelivr CDN 主通道 + GitHub API 兜底），下载后自动选中，再点「注入补丁」即可
- **两个按钮各管各的**：「下载最新 PCL2」只负责下载，「注入补丁」只负责打补丁，互不自动串联
- **v1.5 深色主题**：全新深色界面（GLM 优化）、更精致的按钮与状态视觉，保留全部功能

## 使用方法

1. 双击 `PCL2补丁注入工具.exe`（免安装，需 .NET Framework 4.8）
2. 选择或直接拖入 `Plain Craft Launcher 2.exe`，工具会自动检查（版本 / 类型 / 补丁状态 / 可注入性）
3. 若需要更新 PCL2：点「③ PCL2 更新」卡片里的「下载最新 PCL2」→ 选择保存位置 →
   工具从 GitHub 下载最新版并自动选中（**不会自动注入**）
4. 检查通过后点击「注入补丁」→ 确认框点「开始注入」
5. 几秒内完成，按钮变绿「补丁已生效」；启动 PCL2 进入下载页即可

> 如果下载失败，可点「在浏览器打开 GitHub」手动下载
> （<https://github.com/Meloong-Git/PCL> 根目录的 `最新正式版.zip`）。

## 适用范围

PCL2 2.99.x（如 2.99.0.0）。其它版本会提示"结构不受支持"，不会乱改。

## 补丁原理

PCL2 2.99.0.0 以"为遵守相关法律法规，PCL 暂时不再提供 MC 下载功能"为由，把 3 处入口换成了抛异常的占位代码（下载/安装核心代码其实都还在）：

1. `PCL.PageDownloadInstall::LoaderInit` —— 假加载器恢复为真实的版本清单加载器 `DlClientListLoader`
2. `PCL.ModModpack::ModpackInstall(String,String,String)` —— 整合包类型分发恢复：CurseForge / HMCL / MMC / MCBBS / Modrinth / 启动器包 / 普通压缩包
3. `PCL.ModModpack::ModpackInstall()` —— 恢复"选择文件 → 后台安装"入口

工具以 2.13.1.1（下载功能正常）为参照在 IL 层恢复上述实现；全量方法级差分确认只改了这 3 个方法。

## GitHub 更新通道说明

- PCL2 官方更新服务器已关闭，官方改为在仓库 main 分支根目录发布 `最新正式版.zip`（内含 `Plain Craft Launcher 2.exe`）
- 工具下载主通道为 jsDelivr CDN（`cdn.jsdelivr.net/gh/Meloong-Git/PCL@main/最新正式版.zip`），
  失败时自动切换 GitHub API（`api.github.com` 的 contents raw 接口）兜底
- 版本查询走 `api.github.com/repos/Meloong-Git/PCL/releases/latest`（显示 tag 与发布日期）
- 下载完成后会校验：zip 能打开、内含 PCL2 exe、`FileVersionInfo` 是 Plain Craft Launcher，才落盘
- 下载/解压均通过临时文件 + 原子替换，失败不留半成品

## 安全说明

- 注入前自动备份为 `<原文件名>.bak`；若备份已存在，会生成 `<原文件名>.yyyyMMddHHmmss.bak`，不覆盖旧备份
- 写入采用"临时文件 + 原子替换"，失败不会损坏原文件
- 已注入过的文件会被识别为「已注入 / 无需注入」，不会重复修改
- 选错文件（非 PCL2、非 exe、文件被占用）都会给出明确中文提示

## 版本历史

| 版本 | 说明 |
| --- | --- |
| v1.5.0 | 深色主题界面优化（GLM）；保留注入核心与 GitHub 下载最新 PCL2；修复 net48 下 jsDelivr TLS 握手（SecurityProtocol 精确 Tls1.2） |
| v1.3.0 | 新增「③ PCL2 更新（GitHub）」：一键下载官方最新 PCL2（jsDelivr 主 + GitHub API 兜底、百分比进度、下载后自动选中但不自动注入） |
| v1.2.0 | 界面优化版：左右分栏新界面、状态语义化配色、按钮动效；功能与原版逐字节等价 |
| v1.1.0 | 初版：注入核心 + 基础界面 |

## 系统要求

- Windows 10 / 11（.NET Framework 4.8，系统自带）

## 免责声明

本工具仅用于个人修复本机 PCL2 的可用性，补丁恢复的是 PCL2 自带的原有功能入口。请遵守 PCL2 的使用条款及相关法律法规，使用本工具产生的后果由使用者自行承担。
