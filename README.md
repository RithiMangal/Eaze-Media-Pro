# Eaze Media Pro — Read Me

A short, plain-English guide to what this program is, how it works, and what it can do.

---

## 1. What this program is

Eaze Media Pro is a media player with a built-in video editor.

It has two separate parts:

| Part | What it is for |
|---|---|
| **Player** | Watch and listen to media. Manage a queue. |
| **A/V Editor** | Cut, copy, paste and join clips. Export the result. |

The two parts do not share a playlist. A file you bring into the editor never
appears in the player's queue, and the other way round. This was added on
purpose, so mixing up your two jobs is not possible.

---

## 2. How it is built

| Thing | Detail |
|---|---|
| Language | Rust |
| Screen toolkit | GTK 4 |
| Video and audio engine | libmpv (the engine inside mpv) |
| Editor tool | ffmpeg and ffprobe |
| Packaged as | AppImage (one file, runs on Linux) |
| Licence | MIT |

### Source files and what each one does

| File | Lines | Job |
|---|---|---|
| `src/main.rs` | 59 | Starts the app. Loads the stylesheet. Opens files given on the command line. |
| `src/model.rs` | 751 | All the data: tracks, queue, settings, the editor timeline, format lists, and the 32 tests for them. |
| `src/ui.rs` | 4374 | Every screen, button and key press. This is the big one. |
| `src/mpv/mod.rs` | 622 | Talks to libmpv. Loads files, seeks, draws video, reads timing. |
| `src/mpv/ffi.rs` | 219 | Low-level link to the libmpv library. |
| `src/ff.rs` | 1388 | ffmpeg and ffprobe: format lists, quality levels, hardware checks, background export jobs. |
| `assets/eaze.css` | 760 | All the colours, glow, panels and layout. |
| `tools/package_appimage.sh` | 107 | Builds the AppImage. |

Total: about 8280 lines.

### The editor timeline, in plain words

The editor **never changes your original files.** It keeps a list (a "timeline")
that says, "take this piece of file A, then this piece of file B, and so on".
When you press Export, ffmpeg builds the joined video from that list.

This means:
- Your source files stay untouched and safe.
- An edit can be undone by clearing the timeline.
- Exporting the same edit twice gives the same result.

---

## 3. The Player

### Views
NOW PLAYING, LIBRARY, QUEUE, A/V EDITOR, SETTINGS — all in the top bar.

### Transport buttons
Shuffle, Previous, Play, Next, Repeat, Rotate, Stop, Mirror, Flip, AutoPos.

### Behaviour that is easy to get wrong, and how it is handled here

| Situation | What happens | Why |
|---|---|---|
| Press Stop | Screen goes completely blank | A video surface keeps showing the last frame otherwise. An opaque cover is put over it. |
| Press Play after Stop | Plays again from the beginning | A stopped engine has no file loaded, so a pause/unpause would do nothing. |
| Press Play at the end of a video | Plays again from the beginning | Resuming on the last frame would look like nothing happened. |
| Video plays | The mouse pointer hides after about 2 seconds | Cleaner watching. Move the mouse to bring it back. |
| Pointer over the controls | Pointer stays visible | You need to click things. |
| Press Stop | Timer, bar and status all reset | No stale numbers left on screen. |

### Timers and counters
- Elapsed time, remaining time and total duration all update live.
- The seek bar follows playback.
- The home view and the fullscreen panel each have their own timer, and both are
  updated. (They used to share one slot, so the visible timer sat at zero.)

---

## 4. The A/V Editor

### Toolbar
IMPORT, Mark Start, Mark End, Cut, Copy, Paste, Delete, Export.

There is **no "+" button**. Use IMPORT to bring files in.

### Buttons, step by step

| Step | What to press | What happens |
|---|---|---|
| 1 | **IMPORT** | Pick one or more files. Each one becomes a clip on the timeline, in the order you picked them. They play as one joined sequence. |
| 2 | Click the timeline | Moves the playhead. |
| 3 | **Mark Start** | Marks the beginning of your selection. |
| 4 | **Mark End** | Marks the end. The chosen area is shown in orange. |
| 5a | **Copy** | Puts that area on the clipboard. The marks stay, so you can still Cut or Delete. |
| 5b | **Cut** | Copies the area and removes it from the timeline. |
| 5c | **Delete** | Removes the area. Your clipboard is not touched. |
| 6 | **Paste** | Puts the clipboard back at the playhead. If the playhead sits in the middle of a clip, that clip is split open to make room. |
| 7 | **CLEAR CLIPS** | Empties the timeline and gives you a blank screen. **The clipboard is kept**, so you can paste your clips onto the blank screen. |
| 8 | **EXPORT** | Opens the export window and renders your edit. |

### Pasting after a clear
Clear the edit, and you get a blank screen that says how much is waiting on the
clipboard and tells you to use PASTE above. Paste puts the clips back.

### Frame controls
- Frame Back / Frame Forward: move one frame at a time.
- Left / Right arrow keys do the same while the editor is open.
- Stop button returns to the beginning.

---

## 5. Export

### Choosing an output format

15 formats are available. Pick in this order: format, quality, resolution.

| Format | What you get |
|---|---|
| MP4 | H.264 video + AAC audio. The safe default. |
| M4V | H.264 + AAC |
| MOV | H.264 + AAC |
| MKV | Matroska. Very good for odd formats. |
| WEBM | VP9 + Opus. Note: slower to export. |
| AVI | H.264 + MP3 |
| TS | MPEG transport stream |
| FLV | H.264 + MP3 |
| MP3 | Audio only |
| M4A | AAC audio |
| AAC | Audio only |
| FLAC | Lossless audio |
| WAV | Uncompressed audio |
| OPUS | Audio only |
| OGG | Vorbis audio |

### Quality levels — from best to smallest

| Level | What it does |
|---|---|
| HIGH | Best picture, slowest, biggest file |
| BALANCED | Recommended. Small file, quick |
| LOW | Smaller and faster |
| SMALLEST | Fastest to finish. Use this on a slow computer |

### Resolution
Original, 3840×2160, 1920×1080, 1280×720, or 854×480.

### What the export window shows you
- A time estimate before you start.
- The CPU name, core count, RAM, free disk space.
- The ffmpeg version.
- Which hardware encoders your computer really has.
- A progress bar, the speed, and how much time is left.
- A **STOP EXPORT** button.

### While exporting
- The work happens on a background thread. **The app stays usable.**
- Only part of your CPU is used, so the window keeps responding.
- The export window closes by itself when the job finishes.

### After exporting
The finished file is **put straight into the player queue and starts playing**,
so you can check the result at once instead of hunting for it on disk.

---

## 6. Settings

| Setting | What it does |
|---|---|
| Volume | Default volume |
| Accent colour | Mint / Ultraviolet / Crimson / Solar Amber |
| Hardware decoding | Use the GPU to decode video |
| Always on top | Keep the window above other windows |
| Autoplay next | Play the next queue item by itself |
| Scan folders on start | Look through your media folders at launch |
| Repeat queue | Loop the queue |
| Shuffle queue | Play in random order |

### Library folders
- **ADD FOLDER** adds a watch folder.
- **REMOVE** takes it away straight away. The screen updates at once — you do
  not need to close and reopen the app. Media from that folder is also removed
  from the library and the queue.
- **RESCAN NOW** looks through all folders again.

---

## 7. Which files it understands

### Video (39)
`3g2, 3gp, asf, avi, divx, dv, f4v, flv, m1v, m2t, m2ts, m2v, m4v, mjpeg,
mkv, mov, mp4, mpe, mpeg, mpg, mts, mxf, nut, ogm, ogv, qt, rm, rmvb, ts, vob,
webm, wmv, xvid, y4m, h264, h265, hevc, avchd, 3gpp`

### Audio (35)
`aac, ac3, aif, aifc, aiff, alac, amr, ape, au, caf, dts, ec3, flac, m4a, m4b,
mka, mp2, mp3, mpc, oga, ogg, opus, ra, snd, spx, tta, voc, w64, wav, wma, wv,
m3u, m3u8, mid, midi`

### Pictures (14)
`avif, bmp, gif, heic, heif, ico, jfif, jpeg, jpg, png, svg, tif, tiff, webp`

### Anything else
If a file has an extension we do not know, the app still tries it, as long as it
is not obviously a document. Papers, archives, subtitles and program files are
refused, so your library stays clean.

---

## 8. Special cases the app handles for you

These were real bugs that got fixed. They are listed here so you know they work.

| Situation | What the app does |
|---|---|
| You add a music file (mp3, flac) | Draws a black picture for it, because the joining tool needs a picture for every part. |
| You add a silent video file | Adds silence for the sound, because the tool needs sound for every part. |
| Clips are different sizes | Makes them all the same size first. Without this, joining fails. |
| Your computer lists a hardware encoder but cannot use it | Checks it for real before using it, then falls back to the CPU. |
| VA-API hardware encoder | Checked and reported, but never chosen by itself, because it needs extra steps the join cannot give it. |
| Export to WebM | Uses VP9 with Opus or Vorbis, because WebM does not allow AAC. |
| A file's length cannot be read | Tells you the name instead of failing quietly. |

---

## 9. Settings that are stored

Saved in a file called `state.json`, in your config folder:

- All settings
- Media folders
- Favourites
- Recently played
- Volume and window size
- Last played track and where you were
- Your mark points, the clipboard, the editor file list and the whole timeline

Your editor work survives closing the app.

---

## 10. Building it

### Normal build
```
cargo build --release
```

The binary lands in `target/release/eaze-media-pro`.

### Run the tests
```
cargo test
```
32 tests. They check the timeline maths, the format lists, and that the export
settings come out right for every format.

### Make the AppImage
```
BLZ_LIBMPV=/path/to/libmpv.so.2 bash tools/package_appimage.sh
```
The finished file lands in `dist/`.

### Turning ffmpeg into the AppImage
ffmpeg is not included by default, because its library files are large (about
213 MB). If you want it bundled:
```
BLZ_BUNDLE_FFMPEG=1 BLZ_FFMPEG_LIBS=/path/to/libs bash tools/package_appimage.sh
```
Without it, the app uses the ffmpeg already installed on the computer. If there
is none, the editor still works — only Export is unavailable, and it says so.

---

## 11. Things worth knowing

- **The player does not need ffmpeg at all.** Watch, queue and library work with
  nothing but libmpv.
- **The editor needs ffmpeg for two jobs:** reading how long each file is when
  you import it, and rendering the export. If ffmpeg is missing, importing
  cannot work, so the editor will tell you instead of failing quietly.
- Cut, Copy, Paste and Delete are simple changes to the timeline list. They cost
  nothing and feel instant. The slow part is only Export.
- The app plays video the right way up. The turn is handled at the drawing step,
  not by flipping the video, so your Rotate and Flip buttons still behave.
- Hardware video decoding is on by default when your computer supports it.
- If the same file is imported twice, it is not added twice.

---

## 12. Checks that were run

These are the real lines produced when each fix was tested. They are here so you
can see the work was checked, not just claimed.

### Editor loads many formats at once — 17 of 19 files became editable clips
```
MT ed_files=43 timeline_clips=17
MT clips: a.aac:3.0 a.flac:3.0 a.m4a:3.0 a.mp3:3.0 a.ogg:3.0 a.opus:3.0 a.wav:3.0
         clip.xyz:12.0 v.asf:3.0 v.avi:3.0 v.m4v:3.0 v.mkv:3.0 v.mov:3.0
         v.ogv:3.0 v.ts:3.0 v.webm:3.0 weird.dat:3.0
```
Two files were correctly refused: `notes.txt` and `subs.srt`.

### Every one of the 15 export formats works
```
mp4   OK  dur=9.52      m4v   OK  dur=9.52      mov   OK  dur=9.52
mkv   OK  dur=9.54      webm  OK  dur=9.53      avi   OK  dur=9.56
ts    OK  dur=9.54      flv   OK  dur=9.54      mp3   OK  dur=9.52
m4a   OK  dur=9.52      aac   OK  dur=9.52      flac  OK  dur=9.52
wav   OK  dur=9.52      opus  OK  dur=9.53      ogg   OK  dur=9.52
```

### Every exported format plays in mpv with no errors
```
m-mp4.mp4 0    m-mkv.mkv 0    m-webm.webm 0    m-mp3.mp3 0    m-flac.flac 0
m-wav.wav 0    m-opus.opus 0  m-ogg.ogg 0      m-ts.ts 0      m-avi.avi 0
m-aac.aac 0    m-m4a.m4a 0    m-mov.mov 0      m-m4v.m4v 0    m-flv.flv 0
```

### Cut, copy and paste
```
MT clips_in_order=a.flac,v.avi,a.mp3,v.mkv
MT clip_count=4 total=12.0s
```
Four files were picked, one was listed twice, so four clips were made — no
duplicate.

### Clear, then paste onto the blank screen
```
after_clear   timeline=[]   clipboard kept
after_paste   [Clip 3.125→4.167 "pasted"]
ph_shown_after_paste=false
```

### Export finishes, window closes, file goes to the player
```
IM export_rx_set=false win=false
IM player_queue=["import-out.mp4"] playing_path=Some(".../import-out.mp4") view="Home"
IM win_closed=true
```
Exported file: 6.0 seconds, H.264 + AAC, decodes with no errors.

### Stop and Play
```
after_stop  idle=true  pos=0.0  blank cover visible  playing=false
after_play  idle=false pos=0.0  cover hidden         playing=true
+2s         pos=1.52   paused=false
eof_play    pos=0.0    playing=true
```

### Timers
```
at_start  home="0:00 / -0:12"  home_dur="0:12"
playing   home="0:03 / -0:09"  home_dur="0:12"
paused    home="0:03 / -0:09"  mpv_pos=2.76
```

### Pointer hiding
```
after_motion  hidden=false    after_idle    hidden=true
on_controls   hidden=false    after_stop    hidden=false
rehidden      hidden=true
```

### Tests
```
test result: ok. 32 passed; 0 failed
```

---

## 13. Problems fixed along the way

Kept as a record, so the same mistake is not made twice.

| Problem | Cause | Fix |
|---|---|---|
| Video played upside down | mpv draws with the opposite Y direction to GTK | Set `MPV_RENDER_PARAM_FLIP_Y` at draw time. No video filter needed. |
| Video flickered | The screen was being redrawn when there was no new frame to show | Split "is there a frame?" away from "draw the frame". Now the screen is only redrawn when there is something new. |
| Editor showed nothing | The draw connection was only made during playback | The heartbeat now sets it up too, so a loaded-but-paused editor still shows a picture. |
| Settings folder REMOVE did nothing | The settings page was built once and never redrawn | Remove the row on the spot. |
| Editor cut and paste did nothing | The file list only accepted a few formats | Widened the format list to 88 types and removed the filters that hid files. |
| Only the first added file was editable | The edit was only set up once | Every added file now joins the timeline. |
| Export failed on music files | Music has no picture stream | A black picture is made for it. |
| Export failed when clips were different sizes | The join tool needs equal sizes | All clips are made the same size first. |
| Export died on some computers | A hardware encoder was named but not working | Encoders are tested before being used. |
| WebM export failed | WebM does not allow AAC | WebM now uses VP9 with Opus or Vorbis. |
| Play did nothing after Stop | The engine had no file to play | Play now loads the track again from the start. |
| Timer stuck at zero | Two screens shared one label slot | Each screen now has its own. |
| Editor would not load at all | Only the very first file joined the edit | Every file joins now. |

---

## 14. Quick help

**Nothing happens when I press Play.**
Check the status bar at the bottom. If the engine is missing, libmpv was not
found. Try opening a file again.

**Export is greyed out.**
The timeline is empty, or ffmpeg is not installed. Check the line under the
timeline — it says which.

**Export is very slow.**
Pick SMALLEST quality, drop the resolution, or turn the hardware encoder on.

**A file will not open.**
Add it with IMPORT. If the line under the timeline names it, its length could
not be read, which usually means the file is damaged.

**My edit disappeared.**
CLEAR CLIPS empties the timeline. Your original files are never touched. Use
IMPORT to bring the files back.

**The pointer will not show.**
Move the mouse. Press Stop if it stays hidden.

---

## 15. Licence

MIT. Free to use, change and share.
