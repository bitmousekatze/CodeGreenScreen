==============================================================
  GREENSCREEN MEME PIPELINE — QUICK START
==============================================================

Turn a greenscreen meme clip + an image into a finished meme,
entirely in code. No video editor.

Full spec (read this for the why + all the code + examples):
  -> GREENSCREEN_MEME_DESIGN_DOC.html   (open in a browser)

--------------------------------------------------------------
  WHAT YOU NEED
--------------------------------------------------------------
  1. ffmpeg + ffprobe on PATH        (ffmpeg -version)
  2. Windows PowerShell              (System.Drawing ships with it)
  3. A greenscreen video             (the meme template)
  4. The image(s) to drop on the green

--------------------------------------------------------------
  THE 6 STEPS
--------------------------------------------------------------
  1. (optional) render branded cards from cards.json   -> render_cards.ps1
  2. explode video into PNG frames                     -> ffmpeg
  3. scan each frame for green, find its box
  4. (optional) group frames into segments / 2-card
  5. composite: green -> image, then despill edges     -> card_composite.ps1
  6. re-encode frames -> mp4 (carry original audio)    -> ffmpeg

--------------------------------------------------------------
  COMMANDS (copy/paste, edit paths)
--------------------------------------------------------------
  # 0. probe fps + size
  ffprobe -v error -select_streams v:0 \
    -show_entries stream=r_frame_rate,width,height -of csv=p=0 meme.mp4

  # 2. extract frames
  mkdir frames_in frames_out
  ffmpeg -i meme.mp4 -vsync 0 frames_in\frame_%05d.png

  # 3 + 5. detect + composite (single region)
  powershell -ExecutionPolicy Bypass -File card_composite.ps1
  #   point it at: frames_in (in), frames_out (out), content.png (image)

  # 6. rebuild video at the SOURCE fps, keep audio
  ffmpeg -framerate 24000/1001 -i frames_out\frame_%05d.png \
         -i meme.mp4 -map 0:v -map 1:a? -c:v libx264 \
         -pix_fmt yuv420p -crf 18 -shortest final_meme.mp4

--------------------------------------------------------------
  THE CORE TRICK (green detection)
--------------------------------------------------------------
  A pixel is "green" when green clearly beats red AND blue:
      core   : g>100 && g>r+60 && g>b+60   (find box + fill)
      fill   : g>80  && g>r+30 && g>b+30   (replace)
      despill: g>r+12 && g>b+12  -> g = max(r,b)   (kill halo)

  Scale the image to COVER the box (max ratio, centered),
  pad with the image's background color (bone for cards, black for screens).

--------------------------------------------------------------
  SCRIPTS IN THIS FOLDER
--------------------------------------------------------------
  render_cards.ps1     render cards from cards.json
  card_composite.ps1   single green region per frame
  psycho_composite.ps1 two regions + segment assignment (American Psycho)
  build_feed.ps1       compose a scrolling feed image from posts.json

--------------------------------------------------------------
  GOTCHAS
--------------------------------------------------------------
  - Use the exact fps fraction (24000/1001), not 23.976 -> audio drift.
  - PNG between steps. JPEG reintroduces green-edge artifacts.
  - Edges still green? raise the despill threshold, re-run steps 5-6 only.
  - Keep the $probe + reference-all-assemblies block in the .ps1 files;
    Add-Type needs it to load GDI+ correctly.
  - frames_in / frames_out get large -> delete them when done.

==============================================================
  Built for Prompted. Hand the .html to a Claude with a clip +
  an image and it can make the same memes.
==============================================================
