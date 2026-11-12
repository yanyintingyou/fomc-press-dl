# fomc-press-dl

简体中文 | [English](README.md)

从美联储 YouTube 频道下载 FOMC 新闻发布会视频，文件名按会议日期排序。

仓库里有两个脚本。第一个读取频道视频列表，生成元数据表；第二个从表里筛出新闻发布会并下载视频。

## 为什么需要它

研究 FOMC 沟通时，经常需要发布会视频本身。手动下载要在美联储频道的上传列表里逐个翻找、打开视频、重命名文件。频道里也不只有完整的新闻发布会，还有开场声明等较短的内容，仅凭标题匹配不够。

这个工具扫描一次频道，保留标题符合发布会特征、时长超过 13 分钟的视频，并用会议日期命名。

## 快速开始

环境要求：

- Python 3.9 及以上
- ffmpeg 已加入 PATH（yt-dlp 用它把视频流和音频流合并为 MP4）
- 安装依赖：`pip install -r requirements.txt`

```bash
git clone https://github.com/yanyintingyou/fomc-press-dl.git
cd fomc-press-dl
pip install -r requirements.txt

python 1_fetch_metadata.py
python 2_download_videos.py
```

第一步生成 `fed_raw_metadata.xlsx`。第二步列出匹配的视频，输入 `y` 后开始下载。

## 它会产出什么

| 路径 | 内容 |
|---|---|
| `fed_raw_metadata.xlsx` | 频道上传列表中的视频：上传日期、标题、时长、链接、视频 ID、播放量 |
| `fed_targets_checklist.xlsx` | 筛选出的新闻发布会清单 |
| `raw_videos/` | MP4 视频，命名为 `YYYY-MM-DD_标题.mp4` |

## 筛选规则

`2_download_videos.py` 使用三个默认条件：

| 条件 | 值 |
|---|---|
| 标题包含 | `Press Conference`（不区分大小写） |
| 标题排除 | `Introductory Statement` |
| 时长 | 大于 800 秒（约 13 分钟） |

会议日期从英文标题（如 `January 29, 2020`）中解析；识别不出日期时记为 `Unknown_Date`。

规则写在 `filter_videos()` 里，改关键词或时长阈值就改这个函数。要扫描其他频道，改 `1_fetch_metadata.py` 中的 `CHANNEL_URL`。

## 配置

| 文件 | 变量 | 默认值 | 说明 |
|---|---|---|---|
| `1_fetch_metadata.py` | `CHANNEL_URL` | `@federalreserve` 视频页 | 要扫描的频道 |
| | `OUTPUT_EXCEL` | `fed_raw_metadata.xlsx` | 元数据输出文件 |
| `2_download_videos.py` | `METADATA_FILE` | `fed_raw_metadata.xlsx` | 输入的元数据文件 |
| | `CHECKLIST_FILE` | `fed_targets_checklist.xlsx` | 筛选结果清单 |
| | `DOWNLOAD_DIR` | `raw_videos` | 下载目录 |
| | `MAX_WORKERS` | `4` | 并发下载数 |

## 下载行为

下载使用 4 个线程。每个任务开始前随机等待 0.5 到 2 秒，单个视频最多并发下载 4 个分片。yt-dlp 优先选择 1080p 及以下的 MP4 流，找不到时回退到最佳可用格式。已存在的文件会跳过，中断后重新运行即可。

## 已知限制

- 元数据来自 yt-dlp 的 flat 模式，播放量等字段可能因 YouTube 返回内容不同而为空。筛选和下载不依赖这些字段。
- 日期从标题文本解析。标题中的日期格式不常见时，文件会以 `Unknown_Date` 命名。
- YouTube 限流或人机验证会中断下载。调低 `MAX_WORKERS` 后重试，或用 `pip install -U yt-dlp` 升级 yt-dlp。
- 脚本下载的是频道公开发布的内容。再分发前请确认 YouTube 服务条款和视频本身的版权规定。
- 面向研究用途，适合个人学习和学术工作。

## License

MIT。作者：[yanyintingyou](https://github.com/yanyintingyou)。
