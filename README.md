# Snapchat-Memories-Fixer
Snapchat's "download my data" export splits every Memory that had a filter,
sticker, caption, or drawing into TWO separate files:

    2016-09-14_9D8EB6C5-...-main.jpg      (the photo/video itself)
    2016-09-14_9D8EB6C5-...-overlay.png   (the filter/sticker/text, transparent PNG)

...and it ships the real date + GPS location separately, in
memories_history.json, e.g.:

    {
      "Date": "2026-08-15 18:03:48 UTC",
      "Media Type": "Video",
      "Location": "Latitude, Longitude: 34.015728, -6.841497",
      ...
    }

This script:
  1. Finds matching -main / -overlay pairs and flattens the overlay onto the
     photo (Pillow) or video (ffmpeg) into a single merged file.
  2. If you give it the JSON file too, it matches each merged file to its
     JSON entry and writes the real date + GPS into the file's own metadata
     (EXIF for photos, container metadata for videos).

MATCHING CAVEAT: the export gives no shared ID between the files and the
JSON. The only usable signal is chronological order, so this script sorts
your files and the JSON entries separately for photos and for videos, then
pairs them up in that order. This is reliable *between* days, but if you
saved several photos on the exact same day, their relative order within that
day is a best guess (filenames only carry a date, not a time). The script
warns you if photo/video counts don't match the JSON, which is the main
sign something's misaligned.

Requirements:
    pip install Pillow piexif --break-system-packages
    ffmpeg must be installed (either on PATH, or point to it via a .env file
    -- see FFMPEG_PATH / FFPROBE_PATH below)

Usage:
    python3 fix_snapchat_memories.py <input_dir> <output_dir> [memories_history.json]

ffmpeg location:
    If ffmpeg isn't on your PATH, create a file named ".env" next to this
    script (or in the folder you run it from) with:

        FFMPEG_PATH=D:\\Programs\\ffmpeg-8.1.2-essentials_build\\bin\\ffmpeg.exe
        FFPROBE_PATH=D:\\Programs\\ffmpeg-8.1.2-essentials_build\\bin\\ffprobe.exe

    (ffprobe isn't actually used by this script, but it's harmless to set.)
    No extra package needed -- this script reads .env itself.
