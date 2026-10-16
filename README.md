# Personal Homepage — Fangdi Yu

A single-file academic homepage in the style of the classic **Jon Barron** template
(the de-facto standard for robotics/CS student homepages), with a **News timeline**
borrowed from HKU DASE MPhil homepages.

Everything lives in **one file**: `index.html`. No build step, no Jekyll, no dependencies.

---

## 1. Files

```
homepage/
├── index.html                 ← the entire site
├── README.md                  ← this file
├── assets/
│   └── photo.jpg              ← 600×600 square head-shot (36 KB)
└── files/
    ├── tacenvelope.mp4        ← looping thumbnail video (2.9 MB, intro trimmed)
    ├── tacenvelope.png        ← its poster frame
    ├── tacross.mp4            ← looping thumbnail video (2.7 MB)
    ├── tacross.jpg            ← its poster frame
    ├── tacross-task.mp4       ← second video, stacked below tacross.mp4 (0.44 MB)
    └── tacross-task.jpg       ← its poster frame
```

Both `assets/` and `files/` are already created. Just drop your files in.
If `assets/photo.jpg` is missing, the page automatically falls back to a grey
circle showing "FY" — so nothing breaks.

### The avatar (`assets/photo.jpg`)

The CSS renders the avatar as a **186×186 circle** (`border-radius:50%` +
`object-fit:cover`). Feeding it a portrait photo directly would make the browser
centre-crop the middle of the image — i.e. the chest, not the face. So the
source ID photo (854×1280) was pre-cropped to a square that puts the face at
~65% of the frame height, then resized:

```python
box = (47, 45, 807, 805)          # side 760, centred horizontally
im.crop(box).resize((600, 600), Image.LANCZOS).save(
    'assets/photo.jpg', 'JPEG', quality=88, optimize=True, progressive=True)
```

600×600 keeps it crisp on 3× retina displays while staying at 36 KB.
**To swap the photo:** re-crop to a square (face centred, ~60–70% of frame
height), resize to 600×600, save over the same filename. No HTML change needed.

### Where the videos came from

The two thumbnail videos are copies of assets already published by the projects
themselves, downloaded so the page loads from a single origin:

| Local file | Source |
|---|---|
| `tacross.mp4` / `tacross.jpg` | `https://tacross-touch-project.github.io/assets/slide1-animation.{mp4,jpg}` |
| `tacenvelope.mp4` | `https://huhuanzhang.github.io/TacEnvelope-web/assets/videos/task_usb.mp4` |
| `tacenvelope.png` | `https://huhuanzhang.github.io/TacEnvelope-web/assets/images/task_usb_row.png` |
| `tacross-task.mp4` / `.jpg` | supplied by the author (raw file was 39 MB / 1920×1560 / 60 fps) |

Total media weight ≈ 6.7 MB — well under GitHub's 100 MB per-file limit.

### Re-encoding notes (ffmpeg)

The raw TACROSS task clip was 39 MB at 16.5 Mbps — far too heavy for an
autoplaying thumbnail. It was re-encoded to 0.44 MB:

```bash
ffmpeg -i raw.mp4 -an -vf "scale=640:-2,fps=30" \
       -c:v libx264 -preset slow -crf 31 -pix_fmt yuv420p \
       -movflags +faststart files/tacross-task.mp4
```

`tacenvelope.mp4` had a 2-second dead intro. Trimming it required a re-encode
(a stream copy would snap to the previous keyframe and leave part of the wait):

```bash
ffmpeg -ss 2 -i original.mp4 -an -vf "scale=800:-2,setsar=1" \
       -c:v libx264 -preset slow -crf 23 -pix_fmt yuv420p \
       -movflags +faststart files/tacenvelope.mp4
```

Three flags are easy to forget: `-an` (drop the audio track — these loop
silently), `-movflags +faststart` (moves the moov atom to the front so playback
starts before the file finishes downloading), and `setsar=1` (keeps pixels
square after a non-integer downscale).

A static ffmpeg is available locally via `pip install imageio-ffmpeg`;
`imageio_ffmpeg.get_ffmpeg_exe()` returns its path.

### Stacking two videos in one entry

Put both `<video>` elements inside the same `.pub-media` div — the CSS rule
`.pub-media video + video{border-top:2px solid var(--rule)}` draws the divider.

---

## 2. Live status

**Deployed.** The site is live at **https://fangdiyu.github.io**

| Item | Value |
|---|---|
| GitHub account | `fangdiyu` |
| Repository | https://github.com/fangdiyu/fangdiyu.github.io |
| Live URL | https://fangdiyu.github.io |
| Pages source | branch `main`, path `/` |
| HTTPS | enforced |

> **2026-10-16 — account renamed.** The GitHub account was changed from
> `yuyfrank` to `fangdiyu`, and the repo renamed `yuyfrank.github.io` →
> `fangdiyu.github.io` (the repo rename is required — otherwise Pages demotes
> the site to a project site at `fangdiyu.github.io/yuyfrank.github.io/`).
> The old URLs (`github.com/yuyfrank`, `yuyfrank.github.io`) now return 404:
> GitHub does not redirect profile links, and the freed-up old username could
> be claimed by anyone. All links in `index.html` and the CV were updated.

### Remaining placeholders

Search `index.html` for these strings and replace them:

| Placeholder | Replace with | Status |
|---|---|---|
| `https://github.com/your-username` | — | ✅ done (`fangdiyu`) |
| `https://scholar.google.com/citations?user=YOUR_ID` | your Google Scholar URL | ⬜ TODO |
| `assets/photo.jpg` | your photo (keep the filename) | ✅ done (600×600) |

The ORCID and CV header links were removed on request. The email address is
written out in full rather than shown as the word "Email".

**Don't have a Google Scholar profile yet?** Create one — free, five minutes.
Google Scholar will auto-link your papers once they're indexed.

### Publishing updates

```bash
cd homepage
git add .
git commit -m "Update homepage"
git push
```

Pages redeploys automatically within about a minute.

---

## 3. How it was published (for reference)

This repo is a **user site**, which means the repo name must be exactly
`<username>.github.io`. GitHub enables Pages automatically for such repos —
no manual Settings → Pages step was needed.

```bash
gh repo create fangdiyu.github.io --public --source=. --remote=origin --push
```

For a **project site** (URL `https://fangdiyu.github.io/repo-name/`), any repo
name works, but Pages must be enabled manually under
**Settings → Pages → Source: Deploy from a branch → `main` / `root`**.
Relative paths (`assets/photo.jpg`) work correctly in both cases.

### Custom domain

Add a file named `CNAME` containing your domain (e.g. `fangdiyu.com`), then
configure a CNAME DNS record pointing to `fangdiyu.github.io`.

---

## 4. Editing content later

All content is plain HTML inside `<section>` blocks in `index.html`, in this order:

1. `HEADER` — name, title, affiliation, links, photo
2. `ABOUT` — three paragraphs
3. `NEWS` — reverse-chronological list (`<span class="d">` is the date column)
4. `RESEARCH INTERESTS`
5. `SELECTED PUBLICATIONS` — two featured entries (looping video + clickable title) followed by an "Other Publications" compact list
6. `RESEARCH EXPERIENCE`
7. `EDUCATION`
8. `HONORS & AWARDS`
9. `SKILLS`

**Featured publication entry** — a `.pub-feat` block is a two-column flex row:

```html
<div class="pub-feat">
  <div class="pub-media">                    <!-- 32% wide -->
    <video autoplay loop muted playsinline preload="metadata" poster="files/X.png">
      <source src="files/X.mp4" type="video/mp4">
    </video>
  </div>
  <div class="pub-body">                     <!-- remaining 68% -->
    <a class="ptitle-link" href="PROJECT_URL" target="_blank" rel="noopener">
      <p class="ptitle">Paper title</p>      <!-- clicking this opens PROJECT_URL -->
    </a>
    <p class="authors">…</p>
    <p class="venue">…</p>
    <p class="abstract">…</p>
    <p class="pub-links"><a href="…">[paper]</a><a href="…">[project]</a></p>
  </div>
</div>
```

The `<video>` attributes matter: `autoplay loop muted playsinline` — without
`muted`, browsers block autoplay; without `playsinline`, iOS opens fullscreen.
Below 660px the row stacks vertically and the video goes full width.

**Compact publication entry** — copy one `<li>` inside `<ul class="pubs-compact">`.

Use `<span class="tag pub">Published</span>` or
`<span class="tag rev">Under Review</span>` for the status chip.

**Adding a news item:** copy one `<li>` inside `<ul class="news">`. Keep the
format `<li><span class="d">Mon. YYYY</span><span>Text</span></li>`.

---

## 5. Design notes

- **Responsive** — the header stacks and the photo shrinks below 660px; the News
  date column collapses on mobile.
- **Print-friendly** — `Ctrl/Cmd + P` produces a clean PDF with no shadows or
  wasted margins. Handy if you ever need a paper copy.
- **No external requests** — no Google Fonts, no CDN, no tracking. Loads instantly
  and works offline. The favicon is an inline SVG emoji.
- **Light theme only.** A dark variant can be added via a `prefers-color-scheme`
  media query if you want it.

---

## 6. A note on accuracy

Publication statuses on this page mirror the CV exactly:

- *TacEnvelope* and *TACROSS* are marked **Under Review** (submitted to ICRA 2027).
  Update these the moment you hear back.
- *Earth–Moon Logistics* is marked **Accepted** (ICCECT 2026).
- The underwater glider and ROV papers are marked **Published**, with DOIs.
- The biomimetic underwater robot paper (ICAUS) is **deliberately not listed** —
  its acceptance has not been confirmed. Add it once your supervisor confirms.

Keep it that way. HKU's supplementary document carries a zero-tolerance
authenticity clause, and a public homepage is easy for an admissions officer to check.
