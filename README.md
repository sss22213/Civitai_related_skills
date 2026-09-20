# Civitai related skills

Two [Agent Skills](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills) (`SKILL.md`
folders) that let an LLM chat find models on [Civitai](https://civitai.com) and manage them
inside a local Stable Diffusion WebUI. Written for
[Chat-Ollama-Claude-A1111-WebUI](https://github.com/sss22213/Chat-Ollama-Claude-A1111-WebUI)
(Ollama / Claude CLI / Codex CLI chat with A1111 image generation), but the folders follow
the standard skill layout, so any runtime that reads `SKILL.md` can use the instructions.

| Skill | Talks to | What you can ask |
|---|---|---|
| [`civitai-api/`](civitai-api/) | The public Civitai REST API (`https://civitai.com/api/v1`) through a stdlib-only Python CLI | "find me an Illustrious school-uniform LoRA", model / version details, trigger words, example-image URLs, reverse lookup by file hash, tags, creators, download links |
| [`civitai-helper/`](civitai-helper/) | Your **local** WebUI, through the HTTP API of the Civitai Helper extension | "what LoRAs do I have?", trigger words of an installed LoRA, download a Civitai link into the WebUI, scan for missing info / previews, check for updates, show and save example images, write prompts onto the WebUI cards |

They complement each other: `civitai-api` searches civitai.com, `civitai-helper` installs and
maintains what you picked. Enable both and one conversation can go from "find a LoRA" to
"it is installed, here is a prompt".

## Works with

| Project | Role | Version |
|---|---|---|
| [sss22213/Chat-Ollama-Claude-A1111-WebUI](https://github.com/sss22213/Chat-Ollama-Claude-A1111-WebUI) | The chat app that loads the skills, executes `tools.json` HTTP tools and (when allowed) skill scripts | commit `7b95a6d` or later (skill scripts + multi-skill selection) |
| [sss22213/Stable-Diffusion-Webui-Civitai-Helper](https://github.com/sss22213/Stable-Diffusion-Webui-Civitai-Helper) | Fork of the Civitai Helper extension that exposes the `/civitai-helper/v1` HTTP API used by `civitai-helper` (A1111 and Forge / Forge Neo) | 1.12.0 or later |
| Stable Diffusion WebUI (A1111 / Forge) | Hosts the extension and generates images | any version the extension supports |
| Ollama, Claude Code CLI or OpenAI Codex CLI | The model. Tool calls need Ollama (function calling) or Claude CLI; Codex only receives the instructions | — |
| [stanestane/civitai-api on ClawHub](https://clawhub.ai/stanestane/skills/civitai-api) | Upstream of `civitai-api/` (mirrored here unchanged, MIT-0) | 1.0.0 |

## Install

```bash
git clone git@github.com:sss22213/Civitai_related_skills.git
cp -r Civitai_related_skills/civitai-api Civitai_related_skills/civitai-helper \
      <Chat-Ollama-Claude-A1111-WebUI>/backend/skills/
```

Or copy them into the folder you chose in **⚙️ Settings → Skills → Skills folder**. Docker
users: rebuild (`docker compose up -d --build`) or point the skills folder at a path inside
the mounted `backend/data` volume.

Then, in the chat app:

1. **✨ Skills** button → enable `civitai-api`, `Civitai Helper`, or both (or **Auto** to let
   the model decide per request).
2. For `civitai-api`: **⚙️ Settings → Skills → Allow skills to run scripts** must be on. The
   script runs on the app server; no shell, no pip.
3. For `civitai-helper`: the WebUI must be running with the forked extension, and the app's
   A1111 URL must point at it (Settings → Service sources). Downloads of gated models need
   a Civitai API key in the extension's own settings tab.
4. Optional: put `CIVITAI_API_KEY=<your key>` in
   `backend/data/skill-work/civitai-api/.env` for authenticated `civitai-api` calls
   (needed for `download-url` on gated models; search works without it).

## Usage examples

Prompts in any language work; the model answers in yours.

**Search Civitai (`civitai-api`)**

> 用 civitai 搜尋 school uniform 的 LoRA，只要 1 筆、依下載數排序。回我模型名稱、版本 id、觸發詞、第一張範例圖網址。

The model calls `run_skill_script` with `civitai.py models --query "school uniform" --types LORA --limit 1 --sort "Most Downloaded"` and answers with the name, version id, trigger words and the example-image URL (the search result already includes `modelVersions[].images[]`; replace `original=true` with `width=450` in the URL for a thumbnail).

> Which Illustrious LoRAs did creator X publish? / What is the model behind this file hash 6FY1AM8D…? / Give me the download link for version 269110.

→ `models --username X --base-models Illustrious`, `by-hash <sha256>`, `download-url 269110` (the token is masked in the chat output).

**Manage the local library (`civitai-helper`)**

> 我裝了哪些 LoRA？有沒有校服相關的？

→ `civitai_lora_inventory` (`q: "uniform"`): file name, Civitai name, base model, trigger words and a ready `<lora:name:0.8>` tag.

> https://civitai.com/models/238657 這個是什麼？幫我裝起來

→ `civitai_lookup_model` (summary, newest version, base model, NSFW flag), then, because you asked to install, `civitai_download_model` → `refresh_loras`, and finally a prompt snippet with the LoRA tag and trigger words. Long downloads run in the background; the model polls `civitai_task_status` when you ask.

> 這個 LoRA 怎麼用？給我範例提示詞

→ `civitai_model_examples`: 1–3 example prompts with negative prompt, steps / sampler / CFG / seed and the checkpoint they were made with, optionally embedding the sample image.

> 把範例圖存下來 / 把觸發詞和範例提示詞寫到卡片上 / 卡片沒有預覽圖

→ `civitai_download_examples`, `civitai_write_card_info` (never overwrites hand-written descriptions unless you say so), `civitai_fetch_previews`.

> 我手動複製了一些模型進去，幫我補資料 / 我的模型有更新嗎？

→ `civitai_scan_models` (SHA256 match against Civitai), `civitai_check_new_versions` (offers to download the new version).

**Both together**

> 找一個 Illustrious 的校服 LoRA，裝進 WebUI，然後用它畫一張圖

→ `civitai-api` search → you pick one → `civitai_download_model` → `refresh_loras` → the app's own image generation with the LoRA tag + trigger words.

## What is in each folder

```
civitai-api/
  SKILL.md                 when to use it, quick start, workflow, auth rules
  scripts/civitai.py       CLI: models | model | version | by-hash | creators | tags | images | download-url
  references/api-notes.md  endpoint cheat sheet (cursor pagination, filters)
  skill-card.md, _meta.json, .clawhub/   upstream ClawHub metadata

civitai-helper/
  SKILL.md                 tool-by-tool guide, workflows, safety rules (no download without being asked, one call at a time)
  tools.json               14 HTTP tools against {a1111_url}/civitai-helper/v1
  references/api.md        endpoint / task / file-layout reference
```

`tools.json` uses the `{a1111_url}` placeholder, which the chat app replaces with the
configured WebUI address. Other runtimes can substitute their own base URL.

## Notes

- `civitai-api` prints raw Civitai JSON. The chat app caps script output (8000 characters),
  so keep `--limit` small (1–5) when searching; one result is roughly 4 KB.
- `civitai_download_examples` on a whole model type can fetch thousands of images; the skill
  asks for confirmation and a `max_images` cap in that case.
- Review third-party skills before enabling script execution for them. `civitai-api` only
  reads a `.env` file and talks to `civitai.com`.

## License

`civitai-api` is MIT-0 (upstream). `civitai-helper` is released under the same terms.
