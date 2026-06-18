# Image Sourcing And Validation

Use this reference whenever generated slides need photos, illustrations, icons, abstract visuals, product images, charts, maps, or other external media.

Lumina does not search for or validate images. The agent must provide live, public image URLs before publishing slide HTML to Lumina.

## Decision Flow

1. Use `references/slaydplus-prompts/STEP_1_DEEP_ANALYSIS.md` to decide:
   - `needs_images`
   - `image_style`
   - 5-7 English `keywords`
2. If `needs_images` is `false`, do not force images. Use CSS decoration, charts, icons, or clean visual structure.
3. If `needs_images` is `true`, search for images before final HTML generation.
4. Pass the validated image URLs into the slide-code step as the only allowed image inputs.

## Search Requirements

Search with topic-specific English keywords from the analysis step. Prefer precise, visual queries:

- Good: `silk road caravan samarkand architecture photo`
- Good: `solar panel installation workers photo`
- Good: `customer support agent headset office photo`
- Weak: `business`
- Weak: `technology`

Use images that match the selected `image_style`:

- `photos`: real-world photos from public image hosts, official sites, Wikimedia, or stable CDN URLs.
- `illustrations`: public illustration URLs or generated/agent-provided hosted illustrations.
- `icons`: hosted SVG/PNG icon URLs or CSS/icon-font alternatives when the URL license/source is unclear.
- `abstract`: hosted abstract visuals, generated hosted images, or CSS-generated abstract backgrounds.
- `charts`: prefer agent-created chart HTML/CSS/SVG when exact data matters; use image URLs only for illustrative chart visuals.

## Validation Requirements

Before adding an image URL to slide HTML, validate it:

1. Check the final URL with an HTTP request.
2. Follow redirects.
3. Accept only `2xx` responses.
4. Accept only image content types such as `image/jpeg`, `image/png`, `image/webp`, `image/gif`, or `image/svg+xml`.
5. Reject pages that return `text/html`, login pages, hotlink-blocked URLs, local files, `data:` URLs, private network URLs, and URLs that require cookies or auth headers.
6. Prefer stable direct asset URLs over search-result thumbnails.

Example validation command:

```bash
curl -I -L --max-time 10 "https://example.com/image.jpg"
```

For sources that do not support `HEAD`, use a small ranged `GET`:

```bash
curl -L --max-time 10 -r 0-1024 -o /dev/null -D - "https://example.com/image.jpg"
```

## HTML Rules

Use only validated URLs. Do not hallucinate or guess image URLs.

Every `<img>` tag should include:

- `src` with the validated public URL.
- Descriptive `alt` text.
- `onerror="this.style.display='none'"`.
- `object-fit: cover` or `object-fit: contain`, depending on the design.
- Explicit dimensions or layout constraints so the slide does not shift.

Example:

```html
<img
  src="https://upload.wikimedia.org/wikipedia/commons/example.jpg"
  alt="Historic Silk Road architecture in Samarkand"
  onerror="this.style.display='none'"
  style="position:absolute;right:120px;top:170px;width:680px;height:720px;object-fit:cover;border-radius:18px;"
/>
```

## Slaydplus Compatibility

`SLIDE_GENERATOR_CREATIVE.md` expects image URLs to be provided before HTML generation. Follow that contract:

- Provide a dedicated image URL list to the generation step.
- Use only those provided URLs in `<img>` tags.
- If no validated URLs are available, generate the slide with no `<img>` tags.
- Keep z-index between decoration and content, usually `z-index: 5` to `8`.

## Fallbacks

If image search or validation fails:

1. Do not use broken or unvalidated images.
2. Use a CSS illustration, icon system, chart, timeline, map-like layout, pattern, or abstract visual structure.
3. Mention in internal notes that the slide intentionally uses non-image visuals because no valid public image URL was available.

## Pre-Publish Checklist

- Each needed image was searched from topic-specific keywords.
- Each used image URL passed HTTP validation.
- Each `<img>` has `alt` and `onerror`.
- No `<img>` uses a local path, private URL, login-protected URL, or guessed URL.
- Slides remain visually strong even if an image fails to load.
