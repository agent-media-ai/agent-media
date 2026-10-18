# Changelog

## 2.8.0

- Crazy characters take any trait the user describes: `create_crazy_character` `trait` is a quick-pick id or the user's own words for one absurd look (for example "small head, big body" or "six arms"), screened and moderated, at the same price. The catalog traits stay as quick picks.

## 2.7.1

- The installed skill is self-contained: `npx skills add agent-media-ai/agent-media` now brings `reference/` inside `skills/agent-media/`, with one page per live model, and every link in the skill resolves inside the installed folder.
- The model index and the model pages list only models an agent can pick today (the set `list_models` returns). Planned and unavailable models are gone from the pack.
- The directory manifests (`plugin.json`, `glama.json`, `skills.sh.json`), the ai-ugc-test-loop skill and the README sections on the free first ad, the test loop and crazy characters are generated with the pack, so a sync keeps them.
- Corrected: a 5 s clip plus a 5 s reference clip on seedance-2.5 is 600 credits at its default 480p and 1,250 at 720p; the default quality is per model (480p on seedance-2.5, 720p on seedance-2.0) and the live video models take 480p, 720p and 1080p. The free first ad text follows the server: an eligible new account (a confirmed email and no paid plan yet). Leaving out `model` on generate_image picks gpt-image-2.5.

## 2.7.0

- Add crazy characters for agents: `list_crazy_traits`, `create_crazy_character` (one call: one take, a named sheet saved as a star, and a 10 second clip with sound, quoted first and confirmed with `confirm: true`), `crazy_scene` (a new clip of a saved star) and `list_my_stars`. A who is a catalog id or a described new adult; agents may cast every castable trait. Needs a plan or credits.
- Add the `make-viral-ad` skill, built around the one-call flow.

## 2.5.1

- Make Seedance 2.5 the default and recommended paid video model while keeping Seedance 2.0 selectable as the older faster option.

## 2.5.0

- Document the free first ad for eligible new OAuth connector accounts and how `get_account`, `quote`, `quote_ugc`, `generate_video` and `make_ugc` expose or consume it.
- Add the `ai-ugc-test-loop` skill for research-first UGC testing: outlier scoring, one-variable variants, one Agent Media test take, approval-gated batching, real app-screen finishing and UTM publish packets.

## 2.4.2

- Define Agent Media as one composable connection for uploads, image inspection, images, storyboards and frames, video, voice, characters, quotes and run tracking.

## 2.4.1

- Treat make_ugc as an optional composed shortcut. Agents can freely choose and combine direct image, video and audio generators when they need custom models, modes, references, intermediate assets or shot structure.

## 2.4.0

- Add make_ugc as the recommended complete first-video workflow on the hosted connector, with quote_ugc as its no-charge price and balance check. Direct generators remain available for advanced control.

## 2.3.3

- Generation capacity now covers loose image, video, and audio jobs as well as composed skills. `TOO_MANY_ACTIVE_RENDERS` means wait for a current generation to finish, then retry the same request identity.

## 2.3.2

- Retrieve uploads as native image previews with original generation URLs. Recover forgotten upload sessions with get_uploads({}) or the cached upload_image panel:recent key. Inspect the actual image; never guess its subject from account metadata.

## 2.3.1

- Link the public documentation hub for setup, uploads, account checks, troubleshooting, REST and self-hosting. Free accounts can connect and use the free first ad; paid generation requires a plan.

## 2.3.0

- Add free, read-only authenticated account/credit readiness. Cached catalogs receive the same check through list_models. Quote before the first generation; an unavailable balance is not zero credits.

## 2.2.1

- Upload results now include exact generation input examples and instructions to continue an existing request with the uploaded references. Upload-only requests remain non-billable.

## 2.2.0

- Add generation request identities: recover the same job after response loss without another charge. Preserve job IDs on uncertain worker acknowledgements and report refunds only when confirmed.

## 2.1.1

- Clarify Claude connector sign-in and conversation activation, Claude Code user scope, and Codex OAuth login. Add a no-generation upload check to setup guidance.

## 2.1.0

- Temporary multi-image upload panels with 24-hour expiry; old connector catalogs can open and read panels through upload_image.
- Upload-only requests activate the agent-media skill. Plugin version bumped so installed clients can receive the new guidance.

## 2.0.0

- The loose surface: nine tools (generate_video, generate_image, generate_audio, quote, list_models, list_characters, get_run_status, upload_image, rate_run). The agent writes the prompt and picks the model instead of calling a fixed recipe.
- generate_video has three modes, chosen by the fields you pass: text, image-to-video (first_frame and optional last_frame) and reference (refs, video_refs, audio_refs addressed as @image1, @video1, @audio1).
- Quality 480p, 720p (default) and 1080p, with the credits per second of each on list_models; seven aspect ratios.
- model "auto" picks from the last 30 days of scored runs, and rate_run feeds those numbers back.

## 1.x

- The fixed skills (make_ugc and friends). Still available over REST and the CLI.
