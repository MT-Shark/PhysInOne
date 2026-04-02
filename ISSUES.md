# Known Issues

## 1. Video Codec Incompatibility (2026-04-02)

**Status:** Open

**Problem:**  
Showcase videos are not playing in the browser. The console shows:
```
Video error: MediaError {code: 4, message: 'PipelineStatus::DEMUXER_ERROR_NO_SUPPORTED_STREAMS: FFmpegDemuxer: no supported streams'}
```

**Root Cause:**  
The video files are encoded with **MPEG-4 Part 2** codec (`mpeg4`), which is not supported by modern browsers. Browsers require **H.264** (also known as AVC) codec for MP4 files.

**Affected Files:**
- `static/videos/showcase/A/*.mp4`
- `static/videos/showcase/B/*.mp4`
- `static/videos/showcase/C/*.mp4`
- `static/videos/showcase/D/*.mp4`

**Solution:**  
Convert all videos to H.264 codec using FFmpeg:

```powershell
# Single file conversion
ffmpeg -i "input.mp4" -c:v libx264 -c:a aac "output.mp4"

# Batch conversion script (PowerShell)
Get-ChildItem -Path "static/videos/showcase" -Recurse -Filter "*.mp4" | ForEach-Object {
    $input = $_.FullName
    $output = $_.FullName -replace '\.mp4$', '_h264.mp4'
    ffmpeg -i "$input" -c:v libx264 -c:a aac "$output"
    # After verification, replace original with converted file
}
```

**Verification:**
```powershell
# Check video codec
ffprobe -v error -show_entries stream=codec_name,codec_type -of default=noprint_wrappers=1 "video.mp4"

# Expected output for browser-compatible video:
# codec_name=h264
# codec_type=video
```
