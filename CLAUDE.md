@shared/CLAUDE.md

# KIE.ai-specific session rules

- **API:** KIE.ai (`https://api.kie.ai`).
- **Auth:** Bearer token via `KIE_API_KEY`. Setup check: `./scripts/check-kie-env.sh`.
- **Skills:**
  - `.claude/skills/kie-external-api/SKILL.md` — main API reference (Veo / Sora / Nano Banana / jobs vs first-party endpoints).
  - `.claude/skills/generate-youtube-thumbnail/SKILL.md` — YouTube thumbnail batch workflow on Nano Banana 2.
  - **Image-ad ecosystem (Meta image creatives):** read `shared/skills/image-ad-prompting/OVERVIEW.md` FIRST. Four skills (`chatgpt-image-ad`, `nano-banana-image-ad`, `image-ad-clone-chatgpt`, `image-ad-clone-nano-banana`) + a shared 37-template prompt library. Output is image files; Meta upload is the separate `meta-ad-builder` skill. For the ChatGPT-Image variant on KIE, verify the exact marketplace `model` string at [kie.ai/market](https://kie.ai/market) on first use.
- **Reference images:** KIE has no presigned-upload flow. Reference images must be at publicly reachable URLs. If `MASTER_CONTEXT.md`'s *Image hosting* section is empty, stop and ask the user for their hosting strategy before firing any generation that needs references.
- **Cost disclosure:** Always present credit totals as **estimates**. Direct the user to [kie.ai/logs](https://kie.ai/logs) for exact charges.
- **Logging:** Log every generation call to `logs/kie-api.jsonl` (schema in `logs/README.md`).
- **Rate limits:** 20 new requests per 10s, 100+ concurrent tasks. Back off with jitter on 429.
- **First-time setup:** If `.env` is missing, run `./scripts/setup.sh`. If `MASTER_CONTEXT.md` is missing, copy `MASTER_CONTEXT.template.md` to `MASTER_CONTEXT.md`.
