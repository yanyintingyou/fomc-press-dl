# fomc-press-dl

[简体中文](README.zh-CN.md) | English

Download FOMC press conference videos from the Federal Reserve's YouTube channel, named so they sort by date.

The repo holds two scripts. The first reads the channel listing and writes a metadata workbook. The second filters that workbook down to press conferences and downloads the videos.

## Why this exists

FOMC research often needs the press conference footage itself. Pulling it by hand means scrolling the Fed's uploads, opening each video, and renaming the file. The channel mixes full press conferences with shorter introductory statements and other uploads, so a title match alone is not enough.

This tool scans the channel once, keeps videos with a press conference title and a runtime over 13 minutes, and names each file with its meeting date.

## Quick start

Requirements:

- Python 3.9+
- ffmpeg on your PATH (yt-dlp merges separate video and audio streams into MP4)
- `pip install -r requirements.txt`

```bash
git clone https://github.com/yanyintingyou/fomc-press-dl.git
cd fomc-press-dl
pip install -r requirements.txt

python 1_fetch_metadata.py
python 2_download_videos.py
```

Step 1 writes `fed_raw_metadata.xlsx`. Step 2 prints the matching videos and waits for `y` before downloading.

## What you get

| Path | Contents |
|---|---|
| `fed_raw_metadata.xlsx` | Videos found on the channel's uploads tab: upload date, title, duration, URL, video ID, view count |
| `fed_targets_checklist.xlsx` | The filtered press conference list |
| `raw_videos/` | MP4 files named `YYYY-MM-DD_title.mp4` |

## How the filter works

`2_download_videos.py` applies three rules:

| Rule | Value |
|---|---|
| Title contains | `Press Conference` (case-insensitive) |
| Title excludes | `Introductory Statement` |
| Duration | Over 800 seconds (about 13 minutes) |

It reads the meeting date from English titles such as `January 29, 2020`. Titles without a recognizable date save under `Unknown_Date`.

The rules live in `filter_videos()`. Edit that function to change keywords or the duration threshold. Change `CHANNEL_URL` in `1_fetch_metadata.py` to scan a different channel.

## Configuration

| File | Variable | Default | Purpose |
|---|---|---|---|
| `1_fetch_metadata.py` | `CHANNEL_URL` | `@federalreserve` videos tab | Channel to scan |
| | `OUTPUT_EXCEL` | `fed_raw_metadata.xlsx` | Metadata output |
| `2_download_videos.py` | `METADATA_FILE` | `fed_raw_metadata.xlsx` | Input metadata |
| | `CHECKLIST_FILE` | `fed_targets_checklist.xlsx` | Filtered list output |
| | `DOWNLOAD_DIR` | `raw_videos` | Download folder |
| | `MAX_WORKERS` | `4` | Concurrent downloads |

## Download behavior

The downloader runs four threads. Each task waits 0.5 to 2 seconds before it starts, and each video downloads up to four fragments in parallel. yt-dlp prefers MP4 streams at 1080p or lower and falls back to the best available format. Existing files are skipped, so an interrupted run can be restarted.

## Known limits

- Metadata comes from yt-dlp's flat extraction. Some fields, such as view counts, can be empty depending on what YouTube returns. Filtering and downloads do not depend on them.
- Dates come from title text. A title with an unusual date format saves under `Unknown_Date`.
- YouTube rate limits or bot checks can interrupt a run. Lower `MAX_WORKERS` and retry, or upgrade yt-dlp with `pip install -U yt-dlp`.
- The scripts download what the channel publishes. Check YouTube's terms and the rights that apply to each video before you redistribute it.
- Built for research use, including personal study and academic work.

## License

MIT. Built by [yanyintingyou](https://github.com/yanyintingyou).
