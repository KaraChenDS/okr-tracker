# VisOKR · 个人 OKR 可视化管理

一个**单文件 PWA**：周期 → 目标（O）→ 关键结果（KR）→ 每周检视，配进度环、趋势图、检视热力图。数据默认存本机 localStorage，可选通过**私有 GitHub 仓库**在手机 / iPad / 电脑之间自动同步——零后端、零账号、零费用。

> 线上地址：<https://karachends.github.io/okr-tracker/>
> 本地文件：`~/Documents/AI Works/做一些小应用/okr-tracker/`（双击 `index.html` 也能直接用）

## 功能框架

| 模块 | 内容 |
|---|---|
| 总览 | 周期时间进度 vs OKR 进度对照、大进度环、风险清单（落后/偏慢目标）、进度趋势线、近 91 天检视热力图 |
| 看板 | 周期切换、目标卡片（进度环 + 状态徽章 + 迷你趋势）、KR 行内直接改当前值、归档/删除 |
| 检视 | 每个目标一条时间线；写检视时同步更新各 KR 当前值并存**快照**，趋势图按快照重建历史进度 |
| 设置 | 主题、导入/导出 JSON、示例数据、清空、多设备同步配置与网络诊断 |

### KR 四种度量类型

- **递增 ↗**：从起始值爬到目标值（`0 → 12 本`）
- **递减 ↘**：从起始值降到目标值（`82 → 76 kg`）
- **百分比 %**：直接按 0–100 记
- **是否 ✓**：0/1 完成型

目标进度 = 各 KR 进度按**权重**加权平均；状态徽章 = 目标进度 ÷ 时间进度（≥90% 正常，≥65% 偏慢，否则落后）。

### 使用节奏建议

每周给每个目标写 ≥1 条检视（看板上的「✎ 写检视」），写清楚：发生了什么 / 阻碍 / 下周动作。总览页的「本周检视 x/y」和「上次检视 N 天前」会提醒你谁掉线了。

## 多设备同步（可选）

原理：整份 OKR 状态 JSON 作为一个文件存在你的**私有**数据仓库 `KaraChenDS/okr-tracker-data`（路径 `okr.json`），走 GitHub Contents API，客户端做分层合并（按 id + updatedAt 时间戳，删除以墓碑传播），服务端只是哑文件存储。

**安全规则**（已在代码与测试中强制）：

- 代码里只有 `owner/repo/path`，**没有任何令牌**；
- 令牌只存在每台设备的 localStorage，导出文件与同步载荷都不含令牌（有测试断言）；
- 建议用 **Fine-grained PAT**：只勾选 `okr-tracker-data` 这一个仓库，权限只给 **Contents: Read and write**。

### 开启步骤（每台设备一次）

1. GitHub → Settings → Developer settings → **Fine-grained tokens** → 新建令牌：
   - Repository access：Only select repositories → `KaraChenDS/okr-tracker-data`
   - Permissions：Contents → **Read and write**
2. 打开 <https://karachends.github.io/okr-tracker/> → 设置 → 粘贴令牌 → 「保存并测试连接」。
3. 其余设备：在已配好的设备上点设置页的「复制链接（含令牌）」，把链接发到自己的聊天（**发完记得撤回**），新设备打开链接即自动填好令牌。
4. 之后完全无感：启动拉取、改动 2.5 秒防抖推送、切回前台/重新联网自动拉取，顶栏 ⟳ 可手动全量同步。

### 合并语义

- 同一条记录（周期/目标/KR/检视）按 `id` 合并，谁的 `updatedAt` 新谁生效；
- 删除是**墓碑**（deleted + 时间戳），能跨设备生效，不会「删了又同步回来」；
- 拉取后若本机比云端更全，会自动回写——不会出现「手机记了、电脑永远看不到」。

## 测试

- 本地冒烟（21 项）：渲染、KR 内联编辑、检视弹层、刷新持久化、导出载荷无令牌、390px 无横向溢出、空态引导等；
- 多设备 E2E（17 项，Chrome headless + CDP）：A 首连自动建档、自动推送、B 全新设备自动拉取、**删除墓碑跨设备生效**、空转不产生新提交、无效令牌 401 优雅降级不崩不丢数据、全程请求审计（只打 `okr-tracker-testdata` 测试仓库，真实数据仓库零请求，测后已清理）。

## 文件结构

```
okr-tracker/
├── index.html            # 全部逻辑（单文件）
├── manifest.webmanifest  # PWA 清单
├── sw.js                 # 离线缓存（HTML 绕开 HTTP 缓存；跨域 API 不缓存）
├── icons/                # 应用图标（进度环 + 上升柱）
└── README.md
```

## 部署

GitHub Pages（`main` 分支根目录）。改动后：

```sh
cd okr-tracker && git add -A && git commit -m "update" && git push
```

约 1 分钟后线上生效（SW 对 HTML 走 no-store，刷新即最新）。
