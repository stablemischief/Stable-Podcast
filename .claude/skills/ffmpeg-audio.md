# Skill: FFmpeg Audio Processing for Podcasts

## Purpose

Reference for FFmpeg audio operations used in podcast production pipelines. Covers concatenation, normalization, mixing, format conversion, and a complete Python production class.

## Prerequisites

```bash
brew install ffmpeg      # macOS
apt-get install ffmpeg   # Linux/Debian
winget install FFmpeg    # Windows
```

Verify: `ffmpeg -version` and `ffprobe -version`

## Podcast Production Standards

| Parameter | Recommended |
|---|---|
| Format | MP3 |
| Codec | libmp3lame |
| Bitrate | 128 kbps CBR |
| Sample rate | 44100 Hz |
| Channels | Stereo (or mono at 64-96 kbps) |
| Loudness | -16 LUFS (stereo) / -19 LUFS (mono) |
| True peak | -1.5 dBTP |
| Loudness range | <= 11 LU |
| ID3 version | v2.3 |

## Common Operations

### Generate Silence

```bash
ffmpeg -f lavfi -i anullsrc=r=44100:cl=stereo -t 1.0 -c:a pcm_s16le silence.wav
```

### Concatenate Audio (Demuxer — Fast, Same Format)

```bash
# Create file list
echo "file 'clip1.mp3'" > list.txt
echo "file 'clip2.mp3'" >> list.txt
echo "file 'clip3.mp3'" >> list.txt

ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp3
```

### Concatenate Audio (Filter — Mixed Formats)

```bash
ffmpeg -i clip1.wav -i clip2.mp3 -i clip3.wav \
  -filter_complex "[0:a][1:a][2:a]concat=n=3:v=0:a=1[out]" \
  -map "[out]" -c:a libmp3lame -b:a 128k output.mp3
```

### Loudness Normalization (EBU R128, Two-Pass)

```bash
# Pass 1: Measure
ffmpeg -i input.wav -af "loudnorm=I=-16:TP=-1.5:LRA=11:print_format=json" -f null -

# Pass 2: Apply (use measured values from pass 1)
ffmpeg -i input.wav -af "loudnorm=I=-16:TP=-1.5:LRA=11:measured_I=-23:measured_TP=-2.5:measured_LRA=8:measured_thresh=-33:offset=1.5:linear=true" -c:a libmp3lame -b:a 128k output.mp3
```

### Mix Background Music Under Speech

```bash
ffmpeg -i speech.wav -stream_loop -1 -i music.mp3 \
  -filter_complex "[1:a]volume=0.12,afade=t=in:st=0:d=3,afade=t=out:st=297:d=3[music];[0:a][music]amix=inputs=2:duration=first:dropout_transition=2[out]" \
  -map "[out]" -c:a libmp3lame -b:a 128k output.mp3
```

### Add Fades

```bash
# Fade in 2s, fade out 3s (for a 300s file)
ffmpeg -i input.wav -af "afade=t=in:st=0:d=2,afade=t=out:st=297:d=3" output.wav
```

### Adjust Volume

```bash
ffmpeg -i input.wav -af "volume=6dB" louder.wav      # +6dB
ffmpeg -i input.wav -af "volume=-3dB" quieter.wav     # -3dB
ffmpeg -i input.wav -af "volume=0.5" half.wav          # 50%
```

### Crossfade Between Segments

```bash
ffmpeg -i seg1.wav -i seg2.wav \
  -filter_complex "acrossfade=d=1:c1=tri:c2=tri" \
  -c:a libmp3lame -b:a 128k output.mp3
```

Curve options: `tri` (linear), `qsin` (smooth), `esin`, `log`, `par`

### Convert Formats

```bash
ffmpeg -i input.wav -c:a libmp3lame -b:a 128k -ar 44100 -ac 2 output.mp3
ffmpeg -i input.mp3 -c:a pcm_s16le -ar 44100 output.wav
```

### Trim/Cut Audio

```bash
ffmpeg -i input.mp3 -ss 00:01:30 -to 00:04:00 -c copy trimmed.mp3
ffmpeg -i input.mp3 -t 30 -c copy first_30s.mp3
```

### ID3 Metadata

```bash
ffmpeg -i input.wav -c:a libmp3lame -b:a 128k -id3v2_version 3 \
  -metadata title="Episode 1" -metadata artist="My Podcast" \
  -metadata album="My Podcast" -metadata genre="Podcast" output.mp3
```

### Embed Cover Art

```bash
ffmpeg -i episode.mp3 -i cover.jpg -map 0:a -map 1:v \
  -c:a copy -c:v mjpeg -id3v2_version 3 \
  -metadata:s:v title="Album cover" episode_with_art.mp3
```

## Python Integration

### Approach 1: subprocess (Recommended for Production)

```python
import subprocess, json, shutil
from pathlib import Path

def check_ffmpeg() -> bool:
    return shutil.which("ffmpeg") is not None

def run_ffmpeg(args: list[str], timeout: int = 300) -> subprocess.CompletedProcess:
    cmd = ["ffmpeg", "-y", "-hide_banner"] + args
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)
    if result.returncode != 0:
        raise subprocess.CalledProcessError(result.returncode, cmd, result.stdout, result.stderr)
    return result

def get_audio_duration(file_path: str) -> float:
    cmd = ["ffprobe", "-v", "quiet", "-print_format", "json", "-show_format", file_path]
    result = subprocess.run(cmd, capture_output=True, text=True)
    return float(json.loads(result.stdout)["format"]["duration"])
```

### Approach 2: pydub (Simpler API)

```python
from pydub import AudioSegment

combined = AudioSegment.empty()
silence = AudioSegment.silent(duration=500)
for f in files:
    combined += AudioSegment.from_file(f) + silence
combined.export("output.mp3", format="mp3", bitrate="128k")
```

**Tradeoff:** pydub lacks `loudnorm` (EBU R128), advanced filter graphs, and fine-grained codec control. Use subprocess for production.

## Complete PodcastProducer Class

```python
class PodcastProducer:
    """
    Pipeline:
      1. Validate inputs and FFmpeg
      2. Normalize each segment (two-pass EBU R128)
      3. Concatenate with silence padding
      4. Optionally mix background music
      5. Apply episode-level fades
      6. Final loudness normalization + MP3 export
    """

    DEFAULT_SAMPLE_RATE = 44100
    DEFAULT_CHANNELS = 2
    DEFAULT_BITRATE = "128k"
    DEFAULT_LUFS_STEREO = -16.0
    DEFAULT_TRUE_PEAK = -1.5
    DEFAULT_LRA = 11.0

    def produce_episode(
        self,
        segments: list[str],
        output_path: str,
        silence_between: float = 1.0,
        background_music: str | None = None,
        music_volume: float = 0.12,
        intro_fade: float = 2.0,
        outro_fade: float = 3.0,
        normalize_segments: bool = True,
        final_normalize: bool = True,
    ) -> str: ...
```

See the full implementation in the research output. Key methods:
- `normalize_segment()` — Two-pass EBU R128 normalization
- `produce_episode()` — Complete pipeline from segments to final MP3
- `cleanup()` — Remove temp working directory

## Quick Reference: FFmpeg Flags

| Flag | Meaning | Example |
|---|---|---|
| `-i` | Input file | `-i input.wav` |
| `-y` | Overwrite without asking | (always use) |
| `-c:a` | Audio codec | `-c:a libmp3lame` |
| `-b:a` | Audio bitrate | `-b:a 128k` |
| `-ar` | Sample rate | `-ar 44100` |
| `-ac` | Channel count | `-ac 2` |
| `-af` | Audio filter | `-af "volume=0.5"` |
| `-filter_complex` | Complex filter graph | (multi-input) |
| `-map` | Select output stream | `-map "[out]"` |
| `-t` | Duration limit | `-t 30` |
| `-ss` | Seek/start time | `-ss 00:01:30` |
| `-to` | End time | `-to 00:04:00` |
| `-f` | Force format | `-f concat` |
| `-safe 0` | Allow unsafe paths | (with `-f concat`) |
| `-stream_loop -1` | Loop input infinitely | (for music beds) |
| `-hide_banner` | Suppress version info | (always use) |
| `-loglevel error` | Only errors | (production) |
| `-id3v2_version 3` | ID3v2.3 tags | (podcast MP3s) |
