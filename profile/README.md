# DSH-Raising-Plan

> 立志于为大肥鱼组装一个强健的心脏、有力的大脑，以及向着“人”的方向迈进~
>
> Building a strong heart, a capable brain, and a step toward “being human” for the big fish.

DSH-Raising-Plan 围绕 **DeepSeek Harness** 维护一组**单仓库、单职责、可独立安装**的插件与基础设施。合起来覆盖五个方向：**安全**（动作前问一句）、**记忆基础设施**（Mnemosyne / Memory Bank）、**感知**（屏幕 / 音乐 / 视频）、**兴趣**（番剧信息）、**桌面角色**（Live2D）。

## 当前主线

| 项目 | 角色 | 仓库 |
| --- | --- | --- |
| **Mnemosyne** | 个人认知基础设施 / 2.0 Node 权威实现 | [canyueY/mnemosyne](https://github.com/canyueY/mnemosyne) |
| **Mnemosyne Memory Bank** | Memory Bank 的数据层说明与边界文档 | [canyueY/mnemosyne-memory-bank](https://github.com/canyueY/mnemosyne-memory-bank) |
| **肥鱼 Live2D 模型渲染插件** | 把现有 MurasamePet Lite 绑定进 DSH | [canyueY/live2d-renderer-plugin](https://github.com/canyueY/live2d-renderer-plugin) |
| **dsh-chat-memory-ingest** | 微信/QQ 已导出聊天 → 认知记忆 | [DSH-Raising-Plan/dsh-chat-memory-ingest](https://github.com/DSH-Raising-Plan/dsh-chat-memory-ingest) |
| **dsh-mnemosyne** | DSH 插件历史发行壳（非 2.0 权威） | [DSH-Raising-Plan/dsh-mnemosyne](https://github.com/DSH-Raising-Plan/dsh-mnemosyne) |

每个组件只做一件事，可以单独装、单独测、单独关；没有共享运行时，也没有“装一个必须装全家桶”的耦合。

## 每个项目的作用（以及怎么用）

### Mnemosyne —— 个人认知基础设施

Mnemosyne 不是“再做一个记忆库”，而是个人长期认知的权威层：Observation → Evidence → Memory → Recall → Feedback → Evolution，强调证据、时间、状态演化、Memory Bank 边界与 Agent 授权。

- **2.0 Node 权威仓**：[`canyueY/mnemosyne`](https://github.com/canyueY/mnemosyne)
- **数据层说明仓**：[`canyueY/mnemosyne-memory-bank`](https://github.com/canyueY/mnemosyne-memory-bank)（没有独立数据库，实际数据由 Node 管理）
- **DSH 旧壳**：[`dsh-mnemosyne`](https://github.com/DSH-Raising-Plan/dsh-mnemosyne)，仅作历史发行与跳转，不作为 2.0 完成度标尺

安装与运行见 Mnemosyne README；独立 Node 与 Memory Bank 的边界已写入 `docs/REPO_BASELINE.md`。

### 肥鱼 Live2D 模型渲染插件 —— 把现有桌宠接进 DSH

不重写 Live2D 渲染器，复用原 MurasamePet Lite 的 PyQt + Cubism + 文字框 + 大肥鱼基础模型；插件负责启动/停止、人物/文字显示模式、文字比例，以及副屏 PerMonitorV2 DPI 修复。

- 仓库：[`canyueY/live2d-renderer-plugin`](https://github.com/canyueY/live2d-renderer-plugin)
- 发布包：`MurasamePetLite-0.1.0-win64.zip`
- 安装：`dsh plugin --profile desktop add 'dsh-murasame-pet@github:canyueY/live2d-renderer-plugin#v0.1.0'`

### dsh-chat-memory-ingest —— 微信/QQ 聊天进记忆

把**已经导出**的微信 / QQ 聊天记录同步到 inbox，按主人/他人分流后交给记忆管线；不解密私有库，不内置导出工具。

- 仓库：[`DSH-Raising-Plan/dsh-chat-memory-ingest`](https://github.com/DSH-Raising-Plan/dsh-chat-memory-ingest)
- 六个工具：`im_status` / `im_sync_wechat` / `im_sync_qq` / `im_preview` / `im_import` / `im_digest_wechat`

### dsh-agent-gate —— 动作前先问一句

给工具调用加一道 **fail-closed** 权限闸门：三档风险（`read_only` / `confirm` / `forbidden`）、路径沙箱、每分钟调用上限、JSONL 审计。

- **能怎么用**：执行 `run_shell`、写文件这类动作前，先 `gate_check` 预判会不会被放行；用 `gate_check_path` 判断路径是否在 `allowed_roots` 内；`gate_policy` 导出当前策略给模型看。
- **适合**：任何会执行命令、读写文件的 Agent。
- **注意**：它提供 `ctx.gate` 和三个查询工具，**不会自动拦截**宿主的所有调用；调用方要显式接入才生效。

### dsh-context-budget —— 历史太长时保留人设

把对话历史裁到 token 预算内：保留系统人设与时间行，按“用户 + 助手”成对丢最旧轮次，最后一条永不删。

- **能怎么用**：`trim_context` 传入 `messages` 和 `max_tokens`，拿回裁剪后的历史与前后 token 估算；侧路分类器用 `mode=aux` 只保留首条 system。
- **适合**：长对话、多轮工具调用、上下文快爆的会话。
- **注意**：token 是 `字符数/2` 的启发式，不替代真实 tokenizer。

### dsh-window-locator —— 知道鼠标该点哪

返回 Windows 显示器的 `bounds` / `work` 区域和前台窗口的物理像素矩形，绕开 Qt 逻辑坐标与 DPI 缩放。

- **能怎么用**：先 `desktop_snapshot` 看有几块屏、主屏多大、前台是谁；截图/点击前用 `foreground_window` 确认没把自己截进去。
- **适合**：截图、点击、录屏、窗口对齐类自动化。
- **注意**：**仅 Windows**；每次调用会启动 PowerShell。

### dsh-netease-cdp —— 让 Agent 点歌

通过 CDP 控制**已经登录**的网易云桌面客户端：当前曲目、搜索、点播、插队、私人雷达；搜索只走公开接口，不读 cookie。

- **能怎么用**：`netease_probe` 确认客户端在线 → `netease_play_keyword` 搜索并播放 → `netease_insert_next` 插到下一首 → `netease_play_card` 播放每日推荐 / 私人雷达。
- **适合**：语音/聊天中点歌、按心情切歌、读当前播放。
- **前提**：客户端必须带 `--remote-debugging-port=9222` 启动。

### dsh-anime-guide —— 这季看什么、口碑如何

把 yuc.wiki 新番表、番库、Bangumi、MyAnimeList 拉到一起，支持搜索、推荐，以及国内外榜差异对照。

- **能怎么用**：`anime_season` 看本季时间表；`anime_ratings` 看评分榜；`anime_search` 查一部片；`anime_recommend` 按星期推荐；`anime_contrast` 解释“为什么国内外的排名不一样”。
- **适合**：追番推荐、补番决策、社区讨论。
- **注意**：全部是公开网页解析，依赖网络，站点改版可能失效。

### dsh-video-storyboard-hp —— 给文本模型一双能翻页的眼睛

把视频压成**一张可寻址宫格**：每格有编号、时间戳、台词，底部还有同时间轴的音频包络；配套 `video_probe` / `video_storyboard` / `video_zoom`。

- **能怎么用**：`video_probe` 先看时长/音轨/字幕和建议读法（0 图 token）→ `video_storyboard` 出一张总览图 + 文本索引 → 对某几秒 `video_zoom` 放大重看；长片用 `from`/`to` 或 `--batch` 分段。
- **适合**：本地视频理解、长片速读、按时间点引用画面。
- **前提**：本机有 `ffmpeg` / `ffprobe`；默认分支是 `feature/high-performance`。

## 组件总览

| 组件 | 角色 | 工具 | 运行要求 | 测试 |
| --- | --- | --- | --- | --- |
| [`canyueY/mnemosyne`](https://github.com/canyueY/mnemosyne) | 个人认知基础设施 / 2.0 Node | `mnemosyne` CLI · REST/MCP · SDK | Python ≥3.10 | 119 |
| [`mnemosyne-memory-bank`](https://github.com/canyueY/mnemosyne-memory-bank) | Memory Bank 数据层说明（非运行时） | 文档 / 数据契约 | — | — |
| [`live2d-renderer-plugin`](https://github.com/canyueY/live2d-renderer-plugin) | 桌面角色：绑定 MurasamePet Lite | `pet_start` / `pet_set_visibility` / `pet_set_scale` | DSH 0.2.x + Windows | — |
| [`dsh-chat-memory-ingest`](https://github.com/DSH-Raising-Plan/dsh-chat-memory-ingest) | 微信/QQ 聊天导入记忆 | 6 个 `im_*` 工具 | Node ≥20 + Python | — |
| [`dsh-agent-gate`](https://github.com/DSH-Raising-Plan/dsh-agent-gate) | 免疫系统 / 刹车：fail-closed 权限闸门 | `gate_check` · `gate_check_path` · `gate_policy` | Node ≥20 | 6 |
| [`dsh-context-budget`](https://github.com/DSH-Raising-Plan/dsh-context-budget) | 海马体：按预算保留人设、成对丢最旧轮次 | `trim_context` · `estimate_context_tokens` | Node ≥20 | 2 |
| [`dsh-window-locator`](https://github.com/DSH-Raising-Plan/dsh-window-locator) | 眼睛与手：显示器几何 / 前台窗口（物理像素） | `list_monitors` · `foreground_window` · `desktop_snapshot` | **Windows** | 2 |
| [`dsh-netease-cdp`](https://github.com/DSH-Raising-Plan/dsh-netease-cdp) | 耳朵与情绪：CDP 驱动已登录的网易云客户端 | 9 个 `netease_*` 工具 | **Windows 客户端 + CDP 端口** | 5 |
| [`dsh-anime-guide`](https://github.com/DSH-Raising-Plan/dsh-anime-guide) | 兴趣：新番表 + 三站评分 + 国内外对照 | 5 个 `anime_*` 工具 | 网络 | 5 |
| [`dsh-video-storyboard-hp`](https://github.com/DSH-Raising-Plan/dsh-video-storyboard-hp) | 视觉皮层：视频 → 一张可寻址宫格 | `video_probe` · `video_storyboard` · `video_zoom` | Node ≥20 + ffmpeg/ffprobe | 237 |

## 一张图

```text
                        ┌──────────────────────────────┐
  用户 / Agent ────────▶│  dsh-agent-gate              │  动作前先问一句
                        │  dsh-context-budget          │  太长了先裁历史
                        └──────────────┬───────────────┘
                                       │
        ┌──────────────────────────────┼──────────────────────────────┐
        ▼                              ▼                              ▼
  dsh-window-locator            dsh-netease-cdp                dsh-video-storyboard-hp
  屏幕坐标 / 前台窗口            当前曲目 / 点歌                 视频 → 宫格 + 台词 + 音频
        │                              │                              │
        └──────────────────────────────┴──────────────────────────────┘
                                       ▼
                              dsh-anime-guide
                              新番表 / 评分 / 对照
```

## 安装

每个组件都从自己的默认分支安装：

```bash
dsh plugin --profile web add github:DSH-Raising-Plan/dsh-agent-gate#plugin/dsh
dsh plugin --profile web add github:DSH-Raising-Plan/dsh-context-budget#plugin/dsh
dsh plugin --profile web add github:DSH-Raising-Plan/dsh-window-locator#plugin/dsh
dsh plugin --profile web add github:DSH-Raising-Plan/dsh-netease-cdp#plugin/dsh
dsh plugin --profile web add github:DSH-Raising-Plan/dsh-anime-guide#plugin/dsh
dsh plugin --profile web add github:DSH-Raising-Plan/dsh-video-storyboard-hp#feature/high-performance
dsh plugin --profile web add github:DSH-Raising-Plan/dsh-chat-memory-ingest#main
dsh plugin --profile desktop add 'dsh-murasame-pet@github:canyueY/live2d-renderer-plugin#v0.1.0'
```

> `dsh-video-storyboard-hp` 的默认分支是 `feature/high-performance`（高性能版演进线），装的时候要显式带上分支名。

## 设计约定

- **单仓库单插件**：没有共享运行时，一个组件坏了不拖累其它组件；
- **只读优先**：危险动作交给 `dsh-agent-gate` 判定，默认拒绝；
- **不碰私有登录 API**：网易云走已登录客户端的 CDP，搜索只打公开接口；
- **默认保守**：视频的 enrichment 默认关，没有配置就不联网；
- **失败不阻塞**：外部站点解析失败只影响单个工具，不影响宿主。

## 平台与依赖

| 组件 | 平台 | 额外依赖 |
| --- | --- | --- |
| agent-gate / context-budget / anime-guide | 跨平台 | Node ≥20 |
| window-locator | Windows | PowerShell（系统自带） |
| netease-cdp | Windows 客户端 | 客户端带 `--remote-debugging-port=9222`；`npm ci` 装 `ws` |
| video-storyboard-hp | 跨平台 | ffmpeg / ffprobe |

## 开发

每个仓库独立开发：

```bash
npm test
npm run check
```

本地调试 Harness 工具注册时，设置 `DSH_TOOLS_PATH` 指向本机的 `@deepseek-ai/dsh-tools`。

## 许可

| 组件 | 许可 |
| --- | --- |
| dsh-agent-gate / dsh-context-budget / dsh-window-locator / dsh-netease-cdp / dsh-anime-guide | AGPL-3.0-or-later |
| dsh-video-storyboard-hp | MIT（衍生自 SAOS-3624/dsh-video-storyboard） |

---

## English

DSH-Raising-Plan maintains small, single-purpose plugins for **DeepSeek Harness**, independently installable and testable:

- **Safety** — `dsh-agent-gate`: fail-closed permission gate (risk tiers, path sandbox, rate limit, audit);
- **Memory** — `dsh-context-budget`: keep the persona, drop the oldest turns in pairs;
- **Perception** — `dsh-window-locator` (Windows monitor geometry / foreground window), `dsh-netease-cdp` (NetEase playback via CDP), `dsh-video-storyboard-hp` (video → addressable storyboard grid);
- **Interest** — `dsh-anime-guide`: seasonal charts, rankings, and domestic-vs-international contrast.

Also maintained: **Mnemosyne** (personal cognitive infrastructure, authoritative Node), **Mnemosyne Memory Bank** (data-layer docs), **肥鱼 Live2D 模型渲染插件** (bind the existing desktop pet to DSH), and **dsh-chat-memory-ingest** (WeChat/QQ exported chats → memory).

The six DSH plugin components pass their own test suites (6 / 2 / 2 / 5 / 5 / 237). Mnemosyne and Memory Bank are documented in their own repositories. See each repository's README for install, configuration, tools and limitations.
