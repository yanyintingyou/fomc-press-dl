<p align="right"><b>简体中文</b> · <a href="README.en.md">English</a></p>

# FOMC 新闻发布会视频下载工具

从美联储官方 YouTube 频道抓取视频元数据，筛选出 FOMC 新闻发布会（Press Conference），批量下载为本地 MP4，供学术研究使用。

## 工作流程

```
1_fetch_metadata.py  →  fed_raw_metadata.xlsx  →  2_download_videos.py  →  raw_videos/*.mp4
```

两个脚本按顺序运行。第一个只抓频道列表，第二个筛选并下载。

## 环境要求

- Python 3.9 或更高版本
- ffmpeg，并已加入 PATH。yt-dlp 需要它把视频流和音频流合并为 MP4。
- Python 依赖：yt-dlp、pandas、openpyxl、tqdm（见 `requirements.txt`）

## 快速开始

```bash
git clone https://github.com/yanyintingyou/fomc-press-conference-video-downloader.git
cd fomc-press-conference-video-downloader
pip install -r requirements.txt

python 1_fetch_metadata.py
python 2_download_videos.py
```

### 第一步：抓取元数据

`1_fetch_metadata.py` 用 yt-dlp 的 flat 模式扫描 `@federalreserve` 频道的视频列表，不下载视频文件。结果写入 `fed_raw_metadata.xlsx`，包含上传日期、标题、时长、完整链接、视频 ID 和播放量，按上传日期从新到旧排列。

```bash
python 1_fetch_metadata.py
```

耗时取决于频道视频数量和网络状况。

### 第二步：筛选并下载

`2_download_videos.py` 读取上一步的表格，按以下默认条件筛选：

| 条件 | 值 |
| --- | --- |
| 标题包含 | `Press Conference`（不区分大小写） |
| 标题排除 | `Introductory Statement` |
| 时长 | 大于 800 秒（约 13 分钟） |

日期从标题中的英文日期（如 `January 29, 2020`）解析。筛选结果保存到 `fed_targets_checklist.xlsx`，同时在终端列出待下载视频，输入 `y` 确认后开始下载。

```bash
python 2_download_videos.py
```

下载使用 4 个线程，优先选择 1080p 及以下的 MP4 流，找不到时回退到最佳可用格式。文件保存到 `raw_videos/`，命名为 `YYYY-MM-DD_标题.mp4`。已存在的文件会跳过，中断后重新运行即可。

## 输出文件

| 路径 | 内容 |
| --- | --- |
| `fed_raw_metadata.xlsx` | 频道全部视频的元数据 |
| `fed_targets_checklist.xlsx` | 筛选出的新闻发布会清单 |
| `raw_videos/` | 下载好的 MP4 视频 |

## 配置

脚本顶部的常量可以直接修改：

| 文件 | 变量 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `1_fetch_metadata.py` | `CHANNEL_URL` | 美联储 YouTube 频道 | 要扫描的频道地址 |
| | `OUTPUT_EXCEL` | `fed_raw_metadata.xlsx` | 元数据输出文件 |
| `2_download_videos.py` | `METADATA_FILE` | `fed_raw_metadata.xlsx` | 输入的元数据文件 |
| | `CHECKLIST_FILE` | `fed_targets_checklist.xlsx` | 筛选结果清单 |
| | `DOWNLOAD_DIR` | `raw_videos` | 下载目录 |
| | `MAX_WORKERS` | `4` | 并发线程数 |

筛选关键词和时长阈值写在 `2_download_videos.py` 的 `filter_videos()` 里。换频道改 `CHANNEL_URL`，改筛选条件改这个函数。

## 注意事项

- 元数据通过 flat 模式获取，个别字段（如播放量）可能因 YouTube 返回内容不同而缺失，不影响筛选和下载。
- 下载任务之间随机等待 0.5–2 秒，单个视频内部并发下载 4 个分片。遇到限流或连接失败时，把 `MAX_WORKERS` 调低后重试。
- yt-dlp 无法解析 YouTube 时，先升级到最新版：`pip install -U yt-dlp`。
- 本工具用于个人学习与学术研究。下载和使用视频请遵守 YouTube 服务条款以及视频内容的版权规定。

## License

MIT
