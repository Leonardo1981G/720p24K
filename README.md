LOCAL VIDEO UPSCALER
Waifu2x + NVIDIA DLSS 5 neural rendering, with a browser interface. Everything runs on your own PC.

1. REQUIREMENTS
================

- Windows 10/11, 64-bit
- NVIDIA RTX graphics card (DLSS 5 here runs on RTX 30, 40 and 50 series; RTX 20 and older are refused)
- A current NVIDIA GeForce driver
- About 3 GB of free disk space for the program files, plus space for your videos
- Internet access while running setup.bat (once)

2. SETUP (once)
================

1. Unzip the folder anywhere (a short path such as C:\Upscaler is easiest).
2. Double-click setup.bat. It installs, INSIDE this folder only:
     - a private Python 3.12 (your other Python installs are not touched; nothing is added to PATH/registry)
     - numpy, PyAV and OpenCV
     - Waifu2x (ncnn / Vulkan) and, if you answer "y", Real-ESRGAN
     - FFmpeg
     - the DLSS 5 runtime, in one of two ways (you are asked):
         [1] COPY it from an existing ComfyUI install that has the ComfyUI-DLSS5-Enhancer node pack.
             Recommended. It is the safe choice for RTX 30-series cards, which only work with one exact,
             tested pair of runtime files (the setup script checks this for downloads).
         [2] DOWNLOAD it from its upstream project (about 470 MB). You must accept its licence notice.
             Windows Defender may flag the file "nvngx.dll" (it is a worker program); that is expected.
               If it is blocked, add the "runtime" folder to your antivirus exclusions.
3. Let the self-test run when asked (y). It sends 3 synthetic frames through the DLSS worker.
4. Double-click run.bat. The browser opens at http://127.0.0.1:8765. Close the black window to stop the app.

UPDATING: unzip a newer version over the same folder (replace files). The folders python, tools, runtime,
ffmpeg and data (your profiles and history) are kept.

3. QUICK START
===============

1. Paste the path of a video into "Source" (Shift + right-click the file in Explorer > Copy as path).
2. Click "Preview frame": processes one frame with the current settings and shows it against the original.
3. Try a few settings changes with Preview until the frame looks right (save them as a Profile).
4. Set "Duration" to 3 or 4 seconds and click "Render video" for a short test.
5. Render the whole video (blank Duration), or add several videos in the "Batch queue" tab.
Press Cancel at any time. Results are written next to the source video unless you set an Output folder.

HOW THE PIPELINE WORKS
  decode -> centre-crop to exact 16:9 (9:16 for vertical video) -> Waifu2x 2x -> resize to the exact size
  DLSS wants -> DLSS 5 -> one single final encode, with the original audio added back.
  Example (1280x736 clip, 4K target, DLSS 1.5x): crop to 1280x720 -> Waifu2x 2560x1440 -> DLSS 1.5x -> 3840x2160.
  Frames travel in small chunks, so disk use stays low and the video is only encoded once.
  Colours: the source's colour matrix is used when decoding and encoding (BT.709 for HD) and the output is tagged,
  so contrast and saturation should not shift. HDR sources are not tone-mapped.

4. THE TABS AND CARDS
======================

PROFILES (top left)
  A profile stores the Output, DLSS 5, temporal and Export settings (not the video path or time range).
  Load / Save (overwrite the selected one) / Save as... / Rename / Duplicate / Delete.
  Export saves the selected profile as a small .json file; Export all saves every profile; Import... reads
  such files (names that already exist get " (2)" added, nothing is overwritten). Profiles live in data\profiles.json.

FRAME COMPARE
  After "Preview frame": the left/first picture is the ORIGINAL frame (cropped to 16:9 at its real size, e.g. 1280x720),
  the other is the result. Slider mode or Side-by-side mode (the two panes scroll and zoom together).
  The browser enlarges the original only so the two line up; "hard pixels" switches off its smoothing.

VIDEO COMPARE
  Load two videos (A = left of the slider, B = right), for example your previous result and the new render.
  One Play/Pause button controls both; frame-step buttons, speed, loop, a seek bar, a wipe slider and zoom
  (move the mouse over the picture to choose where to zoom) are included. The videos are scaled to the same
  width so they line up; "Fill" crops like the app does, "Show everything" letterboxes.
  Browsers cannot play every file. HEVC inside MKV usually does not play. If a video stays black, click
  "Browser copy" for A or B: the app makes an H.264 MP4 copy (same resolution) just for viewing.
  That is why MP4 is the default container for new renders.

BATCH QUEUE
  Paste one video path per line and click "Start queue". Every video is rendered one after the other with the
  CURRENT settings (tip: load a profile first). By default whole videos are rendered; tick the box to apply the
  Start/Duration boxes to every video. Cancel stops the current video and skips the rest.
  All runs of one queue share a batch id (B-month-day-time).

HISTORY
  Every preview and render (also failed or cancelled ones) is logged in data\history.json: date, source, settings,
  time, frames, average frames per second, GPU, peak VRAM, result and any error.
  "VRAM peak" is the highest memory use of the WHOLE GPU seen while the run was active (checked every 1.5 s),
  so it also includes other programs using the GPU.
  Click rows to select them. One row: shows its full settings. Two rows: shows only the settings that differ,
  with the time and fps of each - the quickest way to see what a settings change did.
  "Load settings of selected" copies a logged run's settings back into the form.
  Deleting: "Delete selected" (the ticked rows), "Delete batch of selected" (all logs that share the batch id of
  the selected row), "Delete this date" (all logs of the day picked in the date box) and "Delete all".

5. DLSS 5 NEURAL RENDERING - WHAT EACH SETTING DOES
====================================================

DLSS 5 is NVIDIA's neural rendering pass. It rebuilds material detail (skin, hair, fabric, light on surfaces)
instead of just sharpening edges. Video has none of the extra information a game supplies, so here it works only
from the picture and from motion estimated between frames, which makes it fairly conservative.
Tip: always judge settings with "Preview frame" at 400% zoom, and use Profiles to keep the good ones.
(Numbers like "0.56x detail" below come from the node pack's own measurements on one generated clip - treat them
as a guide, not a promise. Lines marked "rule of thumb" are practical advice, not measurements.)

--- UPSCALING (1x, 1.5x, 2x, 3x) --------------------
The factor is how many times larger DLSS's OUTPUT is than the frame it RECEIVES. Because this app always
produces an exact 4K or 1080p result, the factor decides what size the picture must be before DLSS:

      mode                     4K target gets      1080p target gets
      1x  (DLAA / native)      3840x2160           1920x1080
      1.5x (Quality)           2560x1440           1280x720
      2x  (Performance)        1920x1080           960x540
      3x  (Ultra Performance)  1280x720            640x360

  1x  Enhance only. Nothing is enlarged, DLSS works at the final size. Most faithful, but the slowest and the
      most VRAM hungry, because every frame is processed at full output size. Use it when your frames are ALREADY
      at the target size (for example a video that is already 4K, or after another upscaler did the enlarging).
  1.5x  A gentle enlargement; DLSS has plenty of real pixels to work from, so little is invented. The best
      balance of quality and speed and the setting this app is designed around (Waifu2x to 1440p, then 1.5x to
      4K; or 720p to 1080p). Rule of thumb: use this unless you have a reason not to.
  2x  Roughly half the work of 1.5x but the model must invent more detail. Good for 1080p sources going to 4K
      (set the engine to "No pre-upscale"), or when you need more speed.
  3x  Fastest and most aggressive: 720p straight to 4K with no Waifu2x. The model has the least real information
      and the highest risk of invented or odd detail. Rule of thumb: for quick drafts or already-clean sources.
  (DLSS also has a 1.724x "Balanced" mode. It is not offered here because it cannot land exactly on 3840x2160
  or 1920x1080.)
  Speed example: on an RTX 3090, 1440p to 4K at 1.5x ran at about 3 frames per second in the DLSS stage.

--- STYLE (Default, Natural, Cinematic) ---
Changes the LOOK of the neural result, not how much detail is rebuilt.
  Default    The renderer's own look. Start here.
  Natural    Stays closest to the source: gentler contrast, softer result. Use it when the output must match the
             original colours and contrast (documentary-style footage, skin tones that must not shift, anything
             you will colour-grade yourself later).
  Cinematic  Pushes contrast and deepens shadows, shifting the grade. Use it on flat or dull footage when you
             want more punch, and check the shadows for crushed detail.

--- MODEL PRESET (Default, J, K, L, M) ---
Chooses which trained DLSS model does the work. They differ in how much texture they rebuild.
  Default  Does not force a model; the worker chooses. If the log ever says the worker applied a different
           preset than the one you asked for, set this back to Default.
  J and K  The softest. Measured at about 0.56x of the source's detail energy: clean, calm, few invented
           details, but skin can look smooth or waxy. Use them when the source is noisy or has generator
           artifacts and you want it cleaned up, or when L/M look over-detailed.
  L and M  Rebuild noticeably more skin, hair and fabric texture (about 0.70x for L and 0.73x for M): pores, brow
           lines, separated hair strands. Use them for faces, people and anything that should look real.
           M is the upstream recommendation. Rule of thumb: compare K and M in Preview on a close-up.
  (This app starts with K because that is what your original workflow used.)

--- NR INTENSITY (0.00 to 2.00) ---
How strongly the neural-rendering result is applied. 1.00 = full effect, 0 = none (you get the plain upscale).
Values below 1 blend progressively back toward the source (0.5 is about halfway). Values ABOVE 1 currently
do nothing extra: 1.0, 1.5 and 2.0 give identical output, so there is no point going over 1.0.
  Lower it (0.5-0.8) when the result looks over-processed, plastic or "AI-ish". Raise it to 1.0 for the
  strongest effect.

--- LOCAL TONE (0.00 to 2.00, default 1.00) ---
Local tone mapping: how much the neural pass re-balances brightness and contrast region by region inside the
frame (shadows lifted or deepened, highlights shaped), not across the whole picture at once.
  Higher: more local contrast and depth ("punchier"), with a risk of crushed shadows or hot highlights.
  Lower: tones stay closer to the original. Rule of thumb: if the result's contrast or colours drift from
  the source, lower this (try 0.5) and use the Natural style. The effect is subtle - check it in Preview.

--- LOCAL STRUCTURE (0.00 to 2.00, default 1.50) ---
How strongly fine structure is rebuilt: hair, fabric weave, micro-contrast in surfaces and edges.
  Higher: more visible fine detail (the upstream notes suggest 2.0 for the most realistic faces); at the top
  of the range it can look over-etched or exaggerate noise.
  Lower (about 1.0): softer, more natural, less rebuilt. Use it when the result looks too strong.

--- SKIN STRUCTURE (-1.00 to 2.00, default 2.00) ---
How strongly pores and skin micro-texture are rebuilt, only inside the regions the skin mask finds. It needs
"Automatic skin mask" to be ON; with the mask off, every value gives identical output.
  2.0: the strongest pore and skin texture. Lower values give smoother skin, and values near 0 add little.
  -1: leaves the decision to the model.
  Rule of thumb: raise it when faces look waxy (also try model preset L or M), lower it for a smoother, more
  "beauty" look or if skin looks noisy.

--- AUTOMATIC SKIN MASK (checkbox) ---
  ON   The model detects skin regions (faces, hands, bare skin) and applies the Skin structure setting there.
  OFF  No skin detection: Skin structure has no effect at all. The other settings still work normally.
  Leave it ON for footage with people. Turn it OFF for landscapes, animation or products, or if the mask
  mistakes skin-coloured objects (sand, wood, orange surfaces) for skin and gives them odd texture.

--- VERIFY NEURAL RENDERING AFTER THE RUN (checkbox) ---
After the run the app checks the worker's log for proof that the real neural renderer ran (and not a silent
fallback to plain upscaling). The video is written either way; a failed check only shows a warning.
Messages such as "NR upscaling fell back to native" and repeated "vtable::Hook(Failed to find ...)" lines in the
worker log are normal on current runtimes.

6. TEMPORAL CONTROLS (Advanced, under the DLSS 5 settings)
============================================================

Video frames have no motion data, so the app measures how things moved between each frame and the previous one
(optical flow) and gives DLSS that information. DLSS uses it to carry detail over from earlier frames, which keeps
skin and hair steady instead of shimmering from frame to frame. The defaults are the node pack's defaults and are
fine for normal single-shot clips. Change them only if you see shimmer or ghosting.

--- MOTION (Auto / Always estimate motion / Off) ---
  Auto     Motion is used for videos and skipped for single images (the Preview frame). Leave on Auto.
  Always   Forces motion estimation even for a single frame.
  Off      No motion data is sent (all zero) and history is reset only once at the start. Only suitable for a
           perfectly static camera and scene; moving subjects can ghost or smear.

--- SCENE-CHANGE THRESHOLD (0.01 to 1.00, default 0.24) ---
Before measuring motion, each frame is compared with the previous one. If the average brightness change is above
this number, the app treats it as a hard cut and tells DLSS to forget what it learned from the previous shot.
  Too LOW: fast action, flashes or sudden lighting changes are mistaken for cuts; detail visibly "pops" or goes
           soft at those moments, because DLSS starts over.
  Too HIGH: real cuts are missed and the end of the old shot can smear into the start of the new one (ghosting).
  Clips with hard cuts (compilations): if you see ghosting after a cut, lower it a little (0.15-0.20).
  Clips with flashes or very fast motion that "reset" when they should not: raise it (0.3-0.4).

--- MOTION DETAIL / FLOW WIDTH (64 to 4096, default 640) ---
The width in pixels at which motion is measured (the result is scaled up to the full frame).
  Higher: more precise motion for fine, fast detail, but slower. Lower: faster and smoother, less precise.
  640 is a good balance; going far above 1280 rarely helps.

--- WARM-UP FRAMES (0 to 16, default 0) ---
Extra lead-in frames that are rendered and thrown away before the first real frame, so the neural model has
settled when your video starts. It can reduce a slightly different look in the very first frames, at the cost of
a little extra time. Leave at 0 unless the first frames look different from the rest.

7. EXPORT SETTINGS
====================

  Codec      HEVC (H.265): small files, encoded on the GPU. H.264: most compatible. AV1: needs an RTX 40/50 card.
  Container  MP4: plays in browsers, phones and most players (non-AAC audio is converted to AAC).
             MKV: keeps audio untouched, but most browsers cannot play HEVC in MKV (use "Browser copy").
  Quality    Auto picks a bitrate from resolution and frame rate. Good = twice that, Best = four times,
             Max = constant quality (largest files).
  Pre-upscale engine: Waifu2x photo (natural footage) or cunet (cleaner, slightly softer), Real-ESRGAN (optional,
             4x), or "No pre-upscale" (a plain high-quality resize before DLSS).

8. TROUBLESHOOTING
====================

- "FFmpeg: missing" or "DLSS 5 runtime: ..." at the left: run setup.bat again (it only does what is missing).
- Black video in Video compare: use "Browser copy" for that video.
- "needs the tested experimental Ampere pair": RTX 30 cards need that exact pair of runtime files. Re-run setup.bat
  and choose option 1 (copy from a working ComfyUI install).
- "worker could not be started" / file disappears: antivirus. Add the runtime folder to the exclusions.
- Out of memory or a crash: close other GPU programs (games, ComfyUI, browsers with video) and try the
  1.5x mode or a shorter test clip. Another program using the GPU can also make verification fail.
- A job stops with an error: the black window and the log box at the top show the details.
- Delete the folder "work" any time; it only holds temporary files.

9. FOLDER CONTENTS AND SHARING
=================================

Files that belong to the app: app.py, ui.html, dlss5\, setup.bat, setup.ps1, run.bat, make-share-zip.bat,
README.txt, THIRD_PARTY_NOTICES.txt.
Created on your PC by setup.bat or while running: python\, tools\, ffmpeg\, runtime\, data\ (profiles, history),
work\ (temporary).
To give the app to someone else, double-click make-share-zip.bat. It creates Local-Video-Upscaler-share.zip
one folder up, with ONLY the app files. Never include runtime\, python\, tools\, ffmpeg\ or data\: they are
recreated by setup.bat on the other PC, and the DLSS runtime consists of third-party files that must not be
redistributed. See THIRD_PARTY_NOTICES.txt for the licences.
