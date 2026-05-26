---
name: image-ad-clone-chatgpt
description: >-
  Reverse-engineer an existing image ad into a reusable, parameterizable prompt template validated against ChatGPT Image 2 (gpt-image-2) via the KIE.ai API. Appends the new template to the shared image-ad prompt library so it's reusable by chatgpt-image-ad and nano-banana-image-ad. Triggers on phrases like "clone this ad as a template for gpt-image-2 on KIE", "reverse engineer this ad with ChatGPT Image", "extract a gpt-image-2 template", "add this ad to the chatgpt library". Anchors on input being an EXISTING ad image AND gpt-image-2 as the validation backend — does NOT trigger for fresh generation requests (use chatgpt-image-ad) or nano-banana validation (use image-ad-clone-nano-banana).
---

# image-ad-clone-chatgpt (KIE.ai)

Take an existing image ad and turn it into a reusable, parameterizable prompt template, **validated by round-tripping through `chatgpt-image-ad`'s generator** (gpt-image-2 on KIE). Output: a new entry appended to the shared prompt library, with `Model notes` recording gpt-image-2 behavior.

For the parallel skill that validates against Nano Banana instead, see `image-ad-clone-nano-banana`.

## Read order

1. **This file** — the gpt-image-2-on-KIE-specific validation loop.
2. **[shared/skills/image-ad-clone/prompting/guide.md](../../shared/skills/image-ad-clone/prompting/guide.md)** — the full model-agnostic 10-phase workflow.
3. **[shared/skills/image-ad-prompting/prompting/template-format.md](../../shared/skills/image-ad-prompting/prompting/template-format.md)** — entry skeleton.
4. **[shared/skills/image-ad-prompting/prompting/prompt-library.md](../../shared/skills/image-ad-prompting/prompting/prompt-library.md)** — destination for the new entry.

## Hard rules

Inherits all 6 from the shared guide. Plus model-specific:

7. **Validation is via gpt-image-2 on KIE.** If KIE's marketplace string for ChatGPT Image 2 differs from `gpt-image-2`, the user must set it before this skill runs (see `chatgpt-image-ad` SKILL.md § Model string verification).

## Dependencies

- The `chatgpt-image-ad` skill must be installed in this repo.
- `.env` with `KIE_API_KEY` (and `CHATGPT_IMAGE_MODEL` if non-default).
- Reference ad uploaded to a public URL (KIE has no presigned-upload flow).
- Python 3.12+.

## Where this skill's generator lives

When Phase 1 of the [shared guide](../../shared/skills/image-ad-clone/prompting/guide.md) tells you to locate the companion generator, look here in order:

1. `~/.claude/skills/chatgpt-image-ad/scripts/generate_image.py`
2. `<repo>/skills/chatgpt-image-ad/scripts/generate_image.py`
3. If neither: stop and ask the user to install `chatgpt-image-ad` first.

## Hosting requirement for the reference ad

Because KIE accepts only public URLs as references, **the original reference ad must be uploaded to a public host** before this skill can validate. If the user only has a local file:

1. Stop and ask which host they use (R2 / S3 / Cloudinary / etc.).
2. Document the host in `MASTER_CONTEXT.md` for future runs.
3. Either help them upload (if you have credentials documented) or instruct them to upload and share the URL.

Do not proceed past Phase 1 without a reachable URL.

## Aspect ratio mapping

gpt-image-2 supports `{1:1, 2:3, 3:2, 9:16, 16:9}`. When measuring the original ad's aspect (Phase 2), map to the nearest:
- `4:5` ad → use `2:3` (taller; document the map)
- `1.91:1` ad → use `16:9`
- `5:4` ad → use `1:1` (small re-flow)

If the original requires a ratio gpt-image-2 can't approximate well, use `image-ad-clone-nano-banana` instead (Nano Banana supports the full set).

## Model notes you'll write at Phase 9

```markdown
**Model notes:**
- **gpt-image-2:** {observed behavior on KIE — e.g. "clean on KIE — same fidelity as Arcads-hosted gpt-image-2", "tends to add a 4th Slack message — explicit count in prompt"}
- **nano-banana:** {if cross-tested: actual finding. Else: "untested — validate before using on nano-banana-image-ad backend"}
```

## Iteration directory layout

```
<cwd>/iterations/clone-2026-05-25/
  T40-lifestyle-hero/
    prompt.txt
    v1.png, v2.png, …        # against the source ref
    test-fill-v1.png, …      # Phase 7 generalization test against a different brand
    cross-nano-banana/       # Phase 8 cross-test (optional)
    notes.md
```

## Cross-skill validation (Phase 8) — strongly recommended

```bash
~/.claude/skills/nano-banana-image-ad/scripts/generate_image.py \
  --prompt "$(cat iterations/clone-2026-05-25/T40-lifestyle-hero/prompt.txt)" \
  --aspect-ratio <matched_ratio_for_nano_banana> \
  --image-url <test-brand-product-url> \
  --out iterations/clone-2026-05-25/T40-lifestyle-hero/cross-nano-banana \
  --env-file .env
```

Read the result image and write the `nano-banana:` note.

## See also

- **[shared/skills/image-ad-clone/prompting/guide.md](../../shared/skills/image-ad-clone/prompting/guide.md)** — full workflow
- **[chatgpt-image-ad skill](../chatgpt-image-ad/SKILL.md)** — the generator this skill uses
- **[image-ad-clone-nano-banana skill](../image-ad-clone-nano-banana/SKILL.md)** — sibling skill that validates against Nano Banana
