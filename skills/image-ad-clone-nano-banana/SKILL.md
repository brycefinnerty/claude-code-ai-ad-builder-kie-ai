---
name: image-ad-clone-nano-banana
description: >-
  Reverse-engineer an existing image ad into a reusable, parameterizable prompt template validated against Nano Banana (Gemini Flash Image family) via the KIE.ai API. Appends the new template to the shared image-ad prompt library so it's reusable by chatgpt-image-ad and nano-banana-image-ad. Triggers on phrases like "clone this ad as a template for nano banana on KIE", "reverse engineer this ad with Gemini", "extract a nano-banana template", "add this ad to the nano-banana library". Anchors on input being an EXISTING ad image AND nano-banana as the validation backend — does NOT trigger for fresh generation requests (use nano-banana-image-ad) or gpt-image-2 validation (use image-ad-clone-chatgpt).
---

# image-ad-clone-nano-banana (KIE.ai)

Take an existing image ad and turn it into a reusable, parameterizable prompt template, **validated by round-tripping through `nano-banana-image-ad`'s generator** (default `nano-banana-2` on KIE). Output: a new entry appended to the shared prompt library, with `Model notes` recording Nano Banana behavior.

For the parallel skill that validates against ChatGPT Image 2 instead, see `image-ad-clone-chatgpt`.

## Read order

1. **This file** — the Nano-Banana-on-KIE-specific validation loop.
2. **[shared/skills/image-ad-clone/prompting/guide.md](../../shared/skills/image-ad-clone/prompting/guide.md)** — the full 10-phase workflow.
3. **[shared/skills/image-ad-prompting/prompting/template-format.md](../../shared/skills/image-ad-prompting/prompting/template-format.md)** — entry skeleton.
4. **[shared/skills/image-ad-prompting/prompting/prompt-library.md](../../shared/skills/image-ad-prompting/prompting/prompt-library.md)** — destination for the new entry.

## Hard rules

Inherits all 6 from the shared guide. Plus:

7. **Validation is via Nano Banana on KIE** (default `nano-banana-2`; bump to `nano-banana-pro` if `-2` can't lock structure after 2 iterations and the template is high-stakes).

## Dependencies

- The `nano-banana-image-ad` skill must be installed in this repo.
- `.env` with `KIE_API_KEY`.
- Reference ad uploaded to a public URL (KIE has no presigned-upload flow).
- Python 3.12+.

## Where this skill's generator lives

1. `~/.claude/skills/nano-banana-image-ad/scripts/generate_image.py`
2. `<repo>/skills/nano-banana-image-ad/scripts/generate_image.py`
3. If neither: install `nano-banana-image-ad` first.

## Hosting requirement for the reference ad

Same as `image-ad-clone-chatgpt`: KIE requires public URLs. Resolve hosting (R2 / S3 / Cloudinary) before Phase 1.

## Aspect ratio mapping

Nano Banana supports the full Meta ratio set. Use the exact ratio measured from the original ad whenever possible (`4:5`, `5:4`, etc. all work natively — preserve them).

## Model variant to validate with

Default: `nano-banana-2`. Use `--model nano-banana-pro` for:
- Templates requiring locked character identity across runs
- Material-realism critical work (claymation, Pixar, premium product photography)
- Hero-format ads that will see heavy reuse

`nano-banana-pro` costs more credits per validation iteration; surface in Phase 4.

## Model notes you'll write at Phase 9

```markdown
**Model notes:**
- **gpt-image-2:** {if cross-tested: actual finding. Else: "untested — validate before using on chatgpt-image-ad backend"}
- **nano-banana:** {observed behavior — e.g. "strong on KIE; -2 sufficient", "needed nano-banana-pro to lock character identity", "weak on dense table text — keep rows to 4 max"}
```

Always specify which Nano Banana variant you validated against.

## Iteration directory layout

```
<cwd>/iterations/clone-2026-05-25/
  T41-letter-board/
    prompt.txt
    v1.png, v2.png, …                # against the source ref
    test-fill-v1.png, …              # Phase 7 generalization test
    cross-chatgpt/v1.png             # Phase 8 cross-test
    notes.md
```

## Cross-skill validation (Phase 8) — strongly recommended

```bash
~/.claude/skills/chatgpt-image-ad/scripts/generate_image.py \
  --prompt "$(cat iterations/clone-2026-05-25/T41-letter-board/prompt.txt)" \
  --aspect-ratio <gpt-image-2-compatible-ratio> \
  --image-url <test-brand-product-url> \
  --out iterations/clone-2026-05-25/T41-letter-board/cross-chatgpt \
  --env-file .env
```

If the gpt-image-2 ratio mapping is different (e.g. nano-banana validated at `4:5`, gpt-image-2 needs `2:3`), document the mismatch in `Model notes`.

## See also

- **[shared/skills/image-ad-clone/prompting/guide.md](../../shared/skills/image-ad-clone/prompting/guide.md)** — full workflow
- **[nano-banana-image-ad skill](../nano-banana-image-ad/SKILL.md)** — the generator this skill uses
- **[image-ad-clone-chatgpt skill](../image-ad-clone-chatgpt/SKILL.md)** — sibling skill that validates against gpt-image-2
