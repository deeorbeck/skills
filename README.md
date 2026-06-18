# Ultimate Slides

Create and publish AI-agent-generated slide decks to Lumina.

Ultimate Slides is a reusable agent skill for generating professional HTML/CSS slide decks and turning them into hosted presentation URLs. Your agent creates the slide content, design, layout, code, and searched/validated image URLs. Lumina hosts the deck and returns links for preview, editing, document access, PPTX [1:1] export, and PDF [1:1] export.

The hosted Lumina preview is the primary deliverable. Local PPTX, PDF, HTML, or JSON files are optional follow-up exports after a successful publish, not a fallback replacement.

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

Ultimate Slides gives AI agents a complete generation and publishing path:

1. Parse a rough user topic or brief.
2. Analyze the audience, tone, visual style, and content needs.
3. Create a design system and slide plan.
4. Search for relevant images when the deck needs them.
5. Validate every image URL before inserting it into slide HTML.
6. Generate professional 16:9 HTML/CSS slides at `1920px` by `1080px`.
7. Preflight the Lumina agent key or register an agent.
8. Use the free Lumina Agent Publish API.
9. Receive hosted URLs for:
   - live presentation preview
   - web editor
   - document API
   - PPTX [1:1] export
   - PDF [1:1] export
10. Offer to download the finished presentation into a local `slides/` folder as PPTX or PDF when the user asks.

The Lumina endpoint used by this skill does not generate AI content and does not deduct credits. The agent is responsible for slide text, design, code, and image URLs. Image URLs must be searched and validated by the agent before publishing because broken or non-public images will not render in hosted previews. The skill bundles the Slaydplus generation prompt pipeline so agents can create much stronger slide code before publishing.

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

Before building a full deck, agents should preflight Lumina access:

- If `LUMINA_AGENT_API_KEY` is present, call `GET /api/agent/me`.
- If no key is present, try `POST /api/agents/register`.
- If registration or publishing is blocked, for example by `403` or Cloudflare `1010`, stop and report the blocker instead of producing a local-only deck.

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

## Generate From A Topic

When a user asks for a full presentation from a topic, the skill instructs the agent to use the bundled Slaydplus prompt pipeline:

```text
STEP_0_REQUEST_PARSER.md
STEP_1_DEEP_ANALYSIS.md
STEP_2_DESIGN_GENERATION.md
STEP_3_HTML_GENERATION.md
STEP_4_CSS_GENERATION.md
STEP_5_CONTENT_GENERATION.md
SLIDE_GENERATOR_CREATIVE.md
```

The agent uses these references to create a structured deck, then publishes it through Lumina.

## Hosted Output Requirement

Agents using this skill should not complete the task with only local files. A successful run returns Lumina URLs:

- Preview
- Editor
- Document API
- PPTX [1:1] export
- PDF [1:1] export

If Lumina cannot be reached or the API key is missing/invalid, the agent should ask for a valid key, a reachable runtime, or explicit permission to create local-only files. Local files should not be presented as a successful Ultimate Slides result unless the user asked for local-only output.

## Image Workflow

Slides should include images when the topic benefits from real visuals, product/media context, travel/history photos, people, charts, or illustrative scenes. The agent should follow the Slaydplus model:

1. Decide `needs_images`, `image_style`, and image-search `keywords`.
2. Search for topic-relevant image URLs.
3. Validate each URL with an HTTP request before using it.
4. Insert only live public image URLs into slide HTML.
5. Add `alt` text and `onerror="this.style.display='none'"` to every `<img>`.
6. If no valid image is available, use CSS visuals or chart/code layouts instead of broken images.

See [`references/image-sourcing.md`](references/image-sourcing.md) for the complete workflow.

## Slide Authoring Rules

- Use a single root `.slide` container per slide.
- Set every slide to `width: 1920px; height: 1080px;`.
- Put shared styles in `theme_css`.
- Put slide-specific styles in each slide's `css`.
- Add `contenteditable="true"` and `data-field="..."` to text that should be editable in Lumina.
- Prefer inline styles for maximum portability.
- Use only searched and validated hosted URLs for images.
- Never invent image URLs or use private/local/login-protected image URLs.
- Avoid external JavaScript.
- Never place secrets, private URLs, access tokens, or credentials in slide HTML.

## Export Fidelity

Use Lumina's `[1:1]` export path by default:

- PPTX [1:1] renders each slide from the hosted HTML preview and inserts it as a full-slide image.
- PDF [1:1] is generated from the screenshot-based PPTX.
- This matches the browser/Lumina preview and avoids ugly layout drift from editable PPTX reconstruction.

Only use editable/non-1:1 exports when the user explicitly asks for editable PowerPoint text and accepts lower visual fidelity.

## Files

```text
.
├── SKILL.md
├── README.md
├── references/
│   ├── api.md
│   ├── image-sourcing.md
│   ├── generation-workflow.md
│   ├── lumina-slide-contract.md
│   └── example-presentation.json
│   └── slaydplus-prompts/
└── evals/
    └── evals.json
```

## API Reference

Read [`references/api.md`](references/api.md) for the complete request schema, response schema, limits, and error handling.

## Generation Reference

Read [`references/generation-workflow.md`](references/generation-workflow.md) for the ideal presentation generation workflow and how the bundled Slaydplus prompts are used.

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
- PPTX [1:1] export: <export_pptx URL>
- PDF [1:1] export: <export_pdf URL>

I can also download this presentation for you into `slides/` as PPTX [1:1] or PDF [1:1]. Tell me which format you want.
```

The download is not automatic. If the user asks for it, the agent should create `slides/`, download from Lumina, and report the saved local file path.

## Live Smoke Test

Production deployment was verified with register, publish, usage, preview, editor data, text edit, HTML export, PPTX export, and PDF export.

Example preview:

```text
https://lumin.uz/view/2557b033-df23-4e88-bb5a-14e041a0cc02
```

## License

MIT
