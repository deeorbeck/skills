# Lumina Agent Publish API Reference

## Environment

```bash
export LUMINA_API_BASE_URL="https://api.lumin.uz"
export LUMINA_AGENT_API_KEY="lumin_agent_..."
```

## Register an Agent

Use this only when the agent does not already have a key.

```bash
curl -X POST "$LUMINA_API_BASE_URL/api/agents/register" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_name": "Deck Builder Agent",
    "agent_url": "https://example.com/agent",
    "contact_email": "team@example.com",
    "description": "Creates structured HTML slide decks."
  }'
```

Store the returned `api_key` immediately. Lumina stores only a hash.

If registration returns a network, firewall, Cloudflare, or `403`/`1010` error, do not continue as a successful local-only deck. Report that Lumina registration is blocked and ask the user for a valid `LUMINA_AGENT_API_KEY`, a reachable runtime/server, or explicit approval for local-only output.

## Publish a Presentation

Endpoint:

```http
POST /api/agent/presentations
```

Headers:

```http
Content-Type: application/json
X-Lumina-Agent-Key: lumin_agent_...
```

Request:

```json
{
  "title": "AI Agents in Customer Support",
  "topic": "How AI agents improve support operations",
  "language": "en",
  "theme_css": ".slide { font-family: Inter, sans-serif; }",
  "fonts_url": "https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap",
  "design_data": {
    "style": "executive",
    "colors": {
      "primary": "#2563eb",
      "background": "#0f172a"
    }
  },
  "metadata": {
    "source": "agent-run-123"
  },
  "slides": [
    {
      "title": "Title",
      "slide_type": "title",
      "html": "<div class=\"slide\"><h1 data-field=\"title\" contenteditable=\"true\">AI Agents in Customer Support</h1></div>",
      "css": ".slide { width: 1920px; height: 1080px; display: grid; place-items: center; background: #0f172a; color: white; }",
      "speaker_notes": "Introduce the operating model.",
      "image_url": null,
      "metadata": {
        "layout": "centered-title"
      }
    }
  ]
}
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
  },
  "agent": {
    "id": "7c8c...",
    "name": "Deck Builder Agent",
    "total_presentations": 12
  }
}
```

## Check Usage

```bash
curl "$LUMINA_API_BASE_URL/api/agent/me" \
  -H "X-Lumina-Agent-Key: $LUMINA_AGENT_API_KEY"
```

Use this as a preflight check before generating a full deck when an API key is already available. If it fails, stop and report the blocker instead of creating local files as the final output.

## Download Exports

After publishing, Lumina returns direct export URLs:

- `urls.export_pptx`
- `urls.export_pdf`

Do not download automatically. Offer the user a download first. If the user asks, save the selected format into a local `slides/` folder.

```bash
mkdir -p slides
curl -L "<export_pptx URL>" -o "slides/presentation.pptx"
curl -L "<export_pdf URL>" -o "slides/presentation.pdf"
```

If the direct export endpoint returns an async task instead of a file, use:

```http
POST /api/documents/{uuid}/export/start?export_type=pptx
POST /api/documents/{uuid}/export/start?export_type=pdf
GET /api/export/status/{task_id}
```

When the status is `completed`, download the returned `file_url`:

```bash
curl -L "$LUMINA_API_BASE_URL<file_url>" -o "slides/presentation.pptx"
```

Report the saved path to the user, for example:

```text
Saved: slides/ai-sales-assistant.pptx
```

## Limits

- Slides per request: 1-50
- Slide HTML: 200 KB per slide
- Slide CSS: 80 KB per slide
- Global theme CSS: 200 KB

## Error Handling

- `401`: missing, invalid, or inactive API key.
- `400`: invalid slide payload or missing editable field during text update.
- `404`: presentation not found.
- `422`: schema validation failed.

On validation errors, inspect the response body and fix the named field.

On auth or network errors, do not replace the hosted Lumina output with local files. A local payload can be kept only as an unpublished draft while waiting for a valid key or reachable API.
