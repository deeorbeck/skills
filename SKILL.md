---
name: ultimate-slides
description: Publish AI-agent-generated slide decks to Lumina. Use this skill whenever a user wants an AI agent to create, host, preview, edit, or export a presentation using structured HTML/CSS slide code, especially when the agent already has slide content, code, or image URLs and needs an online presentation URL instead of generating a PPTX locally.
---

# Ultimate Slides

Use this skill to publish a complete slide deck to Lumina through the free Agent Publish API.

Lumina does not generate the content for this endpoint. You, the agent, prepare complete slide HTML/CSS and optional hosted image URLs, then ask Lumina to host the presentation and return preview, editor, document, PPTX, and PDF URLs.

## When To Use

Use this skill when the task needs:

- An online presentation preview URL.
- A hosted editor URL for slide review and direct text edits.
- PPTX or PDF export URLs after generating slides as HTML/CSS.
- A way for an AI agent to publish structured slide code without using Lumina's paid AI generation.

Do not use this skill if the user only wants local files, a markdown outline, or a hand-written explanation without hosted presentation output.

## Required Inputs

Before publishing, gather or create:

- `LUMINA_AGENT_API_KEY`: the agent API key.
- `LUMINA_API_BASE_URL`: defaults to `https://api.lumin.uz`.
- A deck title.
- One or more slides with complete `html`.
- Optional slide `css`, `title`, `slide_type`, `speaker_notes`, and `image_url`.
- Optional global `theme_css`, `fonts_url`, and `design_data`.

If there is no API key, register the agent first using `POST /api/agents/register`, then store the returned key as a secret. The full key is shown only once.

## Workflow

1. Create the presentation content yourself.
2. Build every slide at `1920px` by `1080px`.
3. Put reusable styling in `theme_css`.
4. Put slide-specific styling in each slide's `css`.
5. Add `contenteditable="true"` and `data-field="..."` to text elements that should be editable in Lumina.
6. Use hosted image URLs for images.
7. Publish with `POST /api/agent/presentations`.
8. Return the Lumina URLs to the user.

## API Reference

Read `references/api.md` for endpoint details, request schema, response schema, limits, and examples.

## Slide HTML Rules

Make each slide self-contained and robust:

- Use one root container with class `slide`.
- Set explicit dimensions: `width: 1920px; height: 1080px;`.
- Avoid external JavaScript.
- Avoid secrets, private URLs, and credentials.
- Keep text inside the slide bounds.
- Use `alt` text on meaningful images.
- Prefer CSS layout primitives such as grid and flexbox.

Example slide:

```html
<div class="slide">
  <div class="eyebrow" data-field="eyebrow" contenteditable="true">Market Brief</div>
  <h1 data-field="title" contenteditable="true">AI Agents in Customer Support</h1>
  <p data-field="subtitle" contenteditable="true">A practical operating model for faster support teams.</p>
</div>
```

## Publish Example

```bash
curl -X POST "$LUMINA_API_BASE_URL/api/agent/presentations" \
  -H "Content-Type: application/json" \
  -H "X-Lumina-Agent-Key: $LUMINA_AGENT_API_KEY" \
  -d @presentation.json
```

## Output Format

After publishing, respond with:

```markdown
Published to Lumina:
- Preview: <view URL>
- Editor: <edit URL>
- Document API: <api_document URL>
- PPTX export: <export_pptx URL>
- PDF export: <export_pdf URL>
```

Include the deck UUID and slide count when useful.
