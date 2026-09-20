# Civitai related skills

Agent Skills (`SKILL.md` folders) for working with [Civitai](https://civitai.com) from an
LLM chat, written for
[Chat-Ollama-Claude-A1111-WebUI](https://github.com/sss22213/Chat-Ollama-Claude-A1111-WebUI)
but usable by any runtime that understands the Agent Skill format.

| Skill | What it does | Needs |
|---|---|---|
| [`civitai-helper/`](civitai-helper/) | Manage the **local** Stable Diffusion model library through the A1111 **Civitai Helper** extension API: list installed LoRAs / checkpoints / embeddings with trigger words, look up a model by URL or id, download a version into the WebUI, scan for missing info and previews, check for newer versions, show and save example images / prompts, write trigger words onto the WebUI cards. Declares 14 HTTP tools in `tools.json`. | A1111 / Forge with the Civitai Helper extension; a runtime that executes `tools.json` HTTP tools (`{a1111_url}` placeholder). |
| [`civitai-api/`](civitai-api/) | Search the **public** Civitai REST API by keyword / tag / creator / hash, fetch model or version details (trigger words, example-image URLs), list images and tags, build download URLs. Pure-stdlib Python CLI in `scripts/civitai.py`. Mirror of [stanestane/civitai-api](https://clawhub.ai/stanestane/skills/civitai-api) v1.0.0 (MIT-0). | A runtime that can run the skill's Python script. Optional `CIVITAI_API_KEY` for authenticated calls. |

The two complement each other: `civitai-api` finds models on civitai.com, `civitai-helper`
installs and manages them in the WebUI.

## Install

Copy a skill folder into your runtime's skills directory. For Chat-Ollama-Claude-A1111-WebUI:

```bash
git clone git@github.com:sss22213/Civitai_related_skills.git
cp -r Civitai_related_skills/civitai-helper Civitai_related_skills/civitai-api <webui>/backend/skills/
```

Then pick the skill(s) from the ✨ Skills button. `civitai-api` also needs
**Settings → Skills → Allow skills to run scripts** turned on; put
`CIVITAI_API_KEY=…` in `backend/data/skill-work/civitai-api/.env` if you want authenticated
requests.

## Layout

```
civitai-helper/
  SKILL.md            instructions + when to use
  tools.json          HTTP tools against {a1111_url}/civitai-helper/v1
  references/api.md   endpoint notes
civitai-api/
  SKILL.md
  scripts/civitai.py  CLI wrapper over https://civitai.com/api/v1
  references/api-notes.md
  skill-card.md, _meta.json, .clawhub/   upstream metadata
```

## License

`civitai-api` is MIT-0 (upstream). `civitai-helper` is released under the same terms.
