# Ultimate Slides Generation Workflow

Use this workflow when the user asks you to create a complete presentation, not only publish already-existing slide code.

The workflow is adapted from the Slaydplus presentation-generation prompt pipeline. It is designed to make the agent think like a presentation strategist, content architect, and slide designer before publishing to Lumina.

## Reference Prompts

Read these files only when you need to generate slide content and code:

1. `references/slaydplus-prompts/STEP_0_REQUEST_PARSER.md`
2. `references/slaydplus-prompts/STEP_1_DEEP_ANALYSIS.md`
3. `references/slaydplus-prompts/STEP_2_DESIGN_GENERATION.md`
4. `references/slaydplus-prompts/STEP_3_HTML_GENERATION.md`
5. `references/slaydplus-prompts/STEP_4_CSS_GENERATION.md`
6. `references/slaydplus-prompts/STEP_5_CONTENT_GENERATION.md`
7. `references/slaydplus-prompts/SLIDE_GENERATOR_CREATIVE.md`
8. `references/lumina-slide-contract.md`

## Practical Agent Flow

### 1. Parse The Request

Use `STEP_0_REQUEST_PARSER.md` as the mental contract for extracting:

- topic
- description
- slide count
- language
- audience
- style preference
- color preference
- content requests
- tone
- specific requests

If the user gives a direct, clear request, do not over-question. Infer reasonable defaults from the prompt.

### 2. Analyze The Deck

Use `STEP_1_DEEP_ANALYSIS.md` to decide:

- category and subcategory
- mood
- visual style
- whether images are useful
- image style
- decoration type
- content suggestions

This step prevents generic decks. The topic should drive the design system.

### 3. Create A Design System

Use `STEP_2_DESIGN_GENERATION.md` to create a coherent design system:

- color palette
- typography
- decoration elements
- spacing
- effects
- slide layout principles

Convert the design system into global `theme_css` and reusable style decisions.

### 4. Plan Slides

Build a slide outline before writing HTML. For typical decks:

- Slide 1: title / promise
- Slide 2: agenda or framing
- Middle slides: one idea per slide
- Near-end slide: synthesis, roadmap, comparison, or key takeaways
- Final slide: closing / next step / thank you

Respect the requested slide count. Use varied slide types: `title`, `content`, `list`, `two_column`, `comparison`, `quote`, `timeline`, `stats`, `end`.

### 5. Generate Slide Content

Use `STEP_5_CONTENT_GENERATION.md` for content density and editability rules.

Every slide should have:

- a clear message
- concise text
- optional speaker notes
- editable text fields
- image search context if an image is needed

### 6. Source And Validate Images

If `STEP_1_DEEP_ANALYSIS.md` returns `needs_images: true`, read `references/image-sourcing.md` before writing final HTML.

Use the analysis `keywords` to search for relevant images. Validate each candidate URL with an HTTP check, follow redirects, and accept only public `2xx` image responses. Reject broken, private, login-gated, local, guessed, or `text/html` URLs.

Pass only validated image URLs into the slide-code generation step. This matches the Slaydplus rule: use provided image URLs only, never invent URLs. If no valid URL is available, generate the slide without `<img>` and use CSS visuals instead.

### 7. Generate HTML/CSS

Use `SLIDE_GENERATOR_CREATIVE.md` as the main slide-code standard.

Important adaptations for Lumina:

Read `references/lumina-slide-contract.md` before publishing. It adapts the Slaydplus creative prompt output to the exact Lumina Agent Publish API contract.

### 8. Publish To Lumina

After generating all slide objects, publish with `POST /api/agent/presentations`.

Each slide should be shaped like:

```json
{
  "title": "Slide title",
  "slide_type": "content",
  "html": "<div class=\"slide\" style=\"width:1920px;height:1080px;...\">...</div>",
  "css": "",
  "speaker_notes": "Optional notes"
}
```

Put shared CSS in `theme_css`. Use per-slide `css` only when it improves clarity.

## Quality Bar

Before publishing, inspect the deck against this checklist:

- The deck tells a coherent story.
- Every slide has one clear job.
- The design matches the subject and audience.
- Slide styles are consistent but not repetitive.
- Text is readable at presentation distance.
- No slide has clipped text or overflow.
- Editable text has `contenteditable="true"` and `data-field`.
- Images, if used, were searched, validated, and inserted as hosted URLs.
- No image URL is hallucinated, private, local, expired, or broken.
- No secrets or private URLs appear in the HTML.
