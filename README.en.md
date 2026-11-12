<p align="right"><a href="README.md">简体中文</a> · <b>English</b></p>

# FOMC Press Conference Video Downloader

Fetch video metadata from the Federal Reserve's official YouTube channel, filter out FOMC press conferences, and download them as local MP4 files for academic research.

## Pipeline

```
1_fetch_metadata.py  →  fed_raw_metadata.xlsx  →  2_download_videos.py  →  raw_videos/*.mp4
```

Run the two scripts in order. The first one reads the channel listing; the second filters and downloads.

## Requirements

- Python 3.9+
- ffmpeg on your PATH. yt-dlp uses it to merge video and audio into MP4.
- Python packages from `requirements.txt`: yt-dlp, pandas, openpyxl, tqdm

## Quick start

```bash
git clone https://github.com/yanyintingyou/fomc-press-conference-video-downloader.git
cd fomc-press-conference-video-downloader
pip install -r requirements.txt

python 1_fetch_metadata.py
python 2_download_videos.py
```

### Step 1: Fetch metadata

`1_fetch_metadata.py` scans the videos tab of `@federalreserve` with yt-dlp's flat extraction mode. It reads the listing without downloading video files and writes `fed_raw_metadata.xlsx` with upload date, title, duration, full URL, video ID and view count, sorted newest first.

```bash
python 1_fetch_metadata.py
```

Runtime depends on the number of videos on the channel and your network.

### Step 2: Filter and download

`2_download_videos.py` reads the spreadsheet and applies these defaults:

| Rule | Value |
| --- | --- |
| Title contains | `Press Conference` (case-insensitive) |
| Title excludes | `Introductory Statement` |
| Duration | Longer than 800 seconds (about 13 minutes) |

It parses the date from English titles such as `January 29, 2020`, saves the selection to `fed_targets_checklist.xlsx`, prints the list, and waits for `y` before downloading.

```bash
python 2_download_videos.py
```

The downloader runs 4 worker threads and prefers MP4 streams at 1080p or lower, falling back to the best available format. It writes files to `raw_videos/` as `YYYY-MM-DD_title.mp4` and skips files that already exist, so you can rerun the script after an interruption.

## Output

| Path | Contents |
| --- | --- |
| `fed_raw_metadata.xlsx` | Metadata for every video on the channel |
| `fed_targets_checklist.xlsx` | Filtered press conference list |
| `raw_videos/` | Downloaded MP4 files |

## Configuration

Edit the constants at the top of each script:

| File | Variable | Default | Purpose |
| --- | --- | --- | --- |
| `1_fetch_metadata.py` | `CHANNEL_URL` | Federal Reserve YouTube channel | Channel to scan |
| | `OUTPUT_EXCEL` | `fed_raw_metadata.xlsx` | Metadata output |
| `2_download_videos.py` | `METADATA_FILE` | `fed_raw_metadata.xlsx` | Input metadata |
| | `CHECKLIST_FILE` | `fed_targets_checklist.xlsx` | Filtered list output |
| | `DOWNLOAD_DIR` | `raw_videos` | Download folder |
| | `MAX_WORKERS` | `4` | Worker threads |

The keywords and duration threshold live in `filter_videos()`. Change the channel through `CHANNEL_URL`, and adjust the selection rules in that function.

## Notes

- Flat extraction leaves some fields empty depending on what YouTube returns. Missing view counts do not affect filtering or downloads.
- Each task waits 0.5–2 seconds before starting, and each video downloads 4 fragments in parallel. Lower `MAX_WORKERS` if YouTube rate-limits you.
- If yt-dlp fails to parse YouTube, upgrade it first: `pip install -U yt-dlp`.
- This tool is for personal study and academic research. Follow YouTube's Terms of Service and the copyright rules that apply to the videos you download.

## License

MIT
