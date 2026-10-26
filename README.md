# Redpen

Mark student answer copies like a teacher: boxes with feedback, ticks, crosses, underlines, strike-through, circled scores, stamps, handwritten text, circle and pen. Output is a JSON file (golden dataset) and an annotated PDF.

Static site. No build step, no backend, no CDN: everything is in `public/`.

## Run locally

```
python3 scripts/serve.py 3000     # serves public/ with the same headers as production
```

Open http://localhost:3000. Click **Demo** to try without a file.

## Deploy (Vercel)

1. Push this folder to a Git repo.
2. Vercel → New Project → import the repo. Framework: **Other**. Leave build command empty.
3. `vercel.json` sets the output folder (`public`), security headers and caching.
4. Turn on **Deployment Protection** (Vercel SSO or password). Copies contain student data.

## Settings

`public/js/config.js`

| Key | Purpose |
|---|---|
| `submitUrl` | If set, **Submit** POSTs the review JSON to this URL. If empty, Submit offers JSON/PDF downloads. |
| `submitHeaders` | Extra headers for that request (e.g. an auth token). |

If `submitUrl` is on another origin, it must allow CORS and be reachable under the CSP `connect-src` in `vercel.json`.

## How it works

- Pages are drawn on canvas (PDF via pdf.js, photos as images). Annotations are stored per page in coordinates relative to page size, so they survive zoom and export.
- Work autosaves in the browser (`localStorage`) per file. The JSON export is the source of truth. Closing with unsaved changes shows a warning.
- **Open saved JSON** resumes a review. **Open AI draft** loads an AI-evaluated JSON and marks what the reviewer edited or added.

## JSON format (`redpen/v1`)

```
{ schema, file, exported_at, cohort, meta{copyId,test,reviewer}, pages[{width,height}],
  questions[{id,max_marks,marks_obtained,attempted,objective,parameters?}],
  overall_summary?, totals{marks_obtained,total_marks,percent,remark},
  annotations[{id,type,page,question,x,y,width?,height?,color?,polarity?,number?,feedback?,
               score?,stamp?,text?,font_size?,align?,rotation?,points?,scale?,source}] }
```

- `x`, `y`, `width`, `height` are fractions (0–1) of the page, origin top-left. `page` is 1-based.
- `type`: box, tick, cross, underline, strike, score, stamp, text, circle, pen.
- `polarity` (box/underline): negative = red, neutral = yellow, positive = green.
- `source`: `reviewer`, `ai`, or `ai_edited`.

## Project layout

```
public/
  index.html
  css/styles.css
  js/config.js    settings, tools, stamps, colours
  js/model.js     pure data helpers
  js/export.js    JSON, import, annotated PDF
  js/app.js       UI and interaction
  vendor/         pdf.js 5.x (legacy build), pdf-lib. Self-hosted.
  assets/         sprites (tick, cross, stamps), favicon
scripts/serve.py  local static server with production headers
vercel.json
```

## Notes

- Fonts (Kalam, Reddit Sans) load from Google Fonts. To self-host, add the font files and update `index.html` and the CSP in `vercel.json`.
- Browsers: current Chrome, Edge, Safari, Firefox.
- Sprites are PNG. For sharper output at large zoom, replace them with SVG.
