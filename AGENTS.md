<!-- DO NOT EDIT — this file is auto-generated.
     The repo-specific section lives in AGENTS.tail.md; edit there. -->

# Agent instructions

This repository is set up for AI coding agents (Cursor, Claude Code, Copilot-style tools, etc.) to generate AI video and image assets via the API documented in this repo.

## First-time setup

If `.env` or `MASTER_CONTEXT.md` do not exist, tell the user to run `./scripts/setup.sh`.

## Every session

1. Read **[MASTER_CONTEXT.md](MASTER_CONTEXT.md)** for brand voice, credit costs, and accumulated learnings.
2. Follow the skill at `.cursor/skills/` or `.claude/skills/` (synced from `skills/` via `scripts/sync-skill.sh`).
3. If `MASTER_CONTEXT.md` has empty fields (credit costs, defaults), offer to populate them — ask the user and write the values back so future sessions have them.
4. After material changes, add a dated entry to **MASTER_CONTEXT.md** Changelog.

## Image-ad skill ecosystem (cross-API)

This repo ships a 3-skill ecosystem for generating standalone Meta image-ad creatives. **Read [shared/skills/image-ad-prompting/OVERVIEW.md](shared/skills/image-ad-prompting/OVERVIEW.md) before invoking any of these skills** — it explains the decision tree (gpt-image-2 vs Nano Banana), the shared 37-template library, the hand-off to the separate `meta-ad-builder` skill, and what's out of scope.

Quick map:
- **Generate from a brief** → `chatgpt-image-ad` (typography / UI mimicry) or `nano-banana-image-ad` (photoreal / lifestyle / multi-ref).
- **Clone an existing ad into a reusable template** → `image-ad-clone` (single backend-agnostic skill; asks you which generator to validate against at Phase 1, optionally cross-validates against the other backend at Phase 8).
- **Pull from / add to the shared library** → `shared/skills/image-ad-prompting/prompting/prompt-library.md` (37 ready-to-use validated prompts).
- **Hand off finished images to Meta** → separate `meta-ad-builder` skill; the image-ad skills produce images only.


## This repo specifically

- **API:** KIE.ai (`https://api.kie.ai`).
- **Auth:** Bearer token via `KIE_API_KEY` (single token, no pre-encoded header).
- **Skills:**
  - `kie-external-api` — main API reference (Veo/Sora/Nano Banana endpoints, auth, polling, jobs vs first-party paths).
  - `generate-youtube-thumbnail` — YouTube thumbnail batch workflow on top of Nano Banana 2.
  - **Image-ad ecosystem** (3 skills + shared 37-template library) — see [shared/skills/image-ad-prompting/OVERVIEW.md](shared/skills/image-ad-prompting/OVERVIEW.md):
    - `chatgpt-image-ad` — generate via KIE's dedicated `/api/v1/gpt4o-image/generate` endpoint (typography / UI-mimicry creatives).
    - `nano-banana-image-ad` — generate via KIE `nano-banana-2`/`-pro`/`-edit` (photoreal / lifestyle creatives).
    - `image-ad-clone` — single backend-agnostic skill that reverse-engineers existing ads into reusable templates (asks which backend to validate against at Phase 1; optionally cross-validates at Phase 8).
- **Setup check:** `./scripts/check-kie-env.sh`.
- **Reference images:** KIE has no presigned-upload flow. Hosted public URLs only — see *Image hosting* in MASTER_CONTEXT.md.
- **Logging:** Log every generation call to `logs/kie-api.jsonl` (schema in `logs/README.md`).
- **Dashboards:** [kie.ai/logs](https://kie.ai/logs) · [kie.ai/api-key](https://kie.ai/api-key) · [kie.ai/pricing](https://kie.ai/pricing) · [kie.ai/market](https://kie.ai/market).
