# Ultimate Slides

Publish AI-agent-generated slide decks to Lumina.

Ultimate Slides is a reusable agent skill for turning structured HTML/CSS slide code into hosted presentation URLs. Your agent creates the slide content, layout, code, and image URLs. Lumina hosts the deck and returns links for preview, editing, document access, PPTX export, and PDF export.

## Install

```bash
npx skills add deeorbeck/skills
```

Install only this skill when using a multi-skill repository:

```bash
npx skills add deeorbeck/skills --skill ultimate-slides
```

Install for a specific agent:

```bash
npx skills add deeorbeck/skills --skill ultimate-slides --agent claude-code
npx skills add deeorbeck/skills --skill ultimate-slides --agent codex
```

Use the skill without installing:

```bash
npx skills use deeorbeck/skills --skill ultimate-slides
```

## What It Does

Ultimate Slides gives AI agents a clean publishing path:

1. Build slides as 16:9 HTML/CSS at `1920px` by `1080px`.
2. Use the free Lumina Agent Publish API.
3. Receive hosted URLs for:
   - live presentation preview
   - web editor
   - document API
   - PPTX export
   - PDF export

The Lumina endpoint used by this skill does not generate AI content and does not deduct credits. The agent is responsible for slide text, design, code, and image URLs.

## Requirements

Set these environment variables in the agent runtime:

```bash
export LUMINA_API_BASE_URL="https://api.lumin.uz"
export LUMINA_AGENT_API_KEY="lumin_agent_..."
```

If you do not have a key yet, register an agent:

```bash
curl -X POST "https://api.lumin.uz/api/agents/register" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_name": "My Deck Agent",
    "agent_url": "https://example.com",
    "contact_email": "team@example.com",
    "description": "Creates hosted HTML slide decks."
  }'
```

Store the returned `api_key` immediately. Lumina stores only a hash and cannot show the full key again.

## Quick Start

Create a JSON payload:

```json
{
  "title": "AI Agents in Customer Support",
  "topic": "How AI agents improve support operations",
  "language": "en",
  "theme_css": ".slide { font-family: Inter, sans-serif; }",
  "slides": [
    {
      "title": "Title",
      "slide_type": "title",
      "html": "<div class=\"slide\"><h1 data-field=\"title\" contenteditable=\"true\">AI Agents in Customer Support</h1></div>",
      "css": ".slide { width: 1920px; height: 1080px; display: grid; place-items: center; background: #0f172a; color: white; }"
    }
  ]
}
```

Publish it:

```bash
curl -X POST "$LUMINA_API_BASE_URL/api/agent/presentations" \
  -H "Content-Type: application/json" \
  -H "X-Lumina-Agent-Key: $LUMINA_AGENT_API_KEY" \
  -d @presentation.json
```

Response:

```json
{
  "uuid": "f6f54b7b-...",
  "title": "AI Agents in Customer Support",
  "slide_count": 1,
  "status": "completed",
  "urls": {
    "view": "https://lumin.uz/view/f6f54b7b-...",
    "edit": "https://lumin.uz/html-editor/f6f54b7b-...",
    "api_document": "https://api.lumin.uz/api/documents/f6f54b7b-...",
    "export_pptx": "https://api.lumin.uz/api/documents/f6f54b7b-.../export/pptx",
    "export_pdf": "https://api.lumin.uz/api/documents/f6f54b7b-.../export/pdf"
  }
}
```

## Slide Authoring Rules

- Use a single root `.slide` container per slide.
- Set every slide to `width: 1920px; height: 1080px;`.
- Put shared styles in `theme_css`.
- Put slide-specific styles in each slide's `css`.
- Add `contenteditable="true"` and `data-field="..."` to text that should be editable in Lumina.
- Use hosted URLs for images.
- Avoid external JavaScript.
- Never place secrets, private URLs, access tokens, or credentials in slide HTML.

## Files

```text
.
├── SKILL.md
├── README.md
├── references/
│   ├── api.md
│   └── example-presentation.json
└── evals/
    └── evals.json
```

## API Reference

Read [`references/api.md`](references/api.md) for the complete request schema, response schema, limits, and error handling.

## Example Payload

See [`references/example-presentation.json`](references/example-presentation.json) for a complete two-slide deck.

## Limits

- Slides per request: 1-50
- Slide HTML: 200 KB per slide
- Slide CSS: 80 KB per slide
- Global theme CSS: 200 KB

## Skill Output Format

Agents using this skill should return:

```markdown
Published to Lumina:
- Preview: <view URL>
- Editor: <edit URL>
- Document API: <api_document URL>
- PPTX export: <export_pptx URL>
- PDF export: <export_pdf URL>
```

## Live Smoke Test

Production deployment was verified with register, publish, usage, preview, editor data, text edit, HTML export, PPTX export, and PDF export.

Example preview:

```text
https://lumin.uz/view/2557b033-df23-4e88-bb5a-14e041a0cc02
```

## License

MIT
