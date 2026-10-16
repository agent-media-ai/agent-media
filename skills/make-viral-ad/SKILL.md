---
name: 'make-viral-ad'
description: 'Make a scroll-stopping crazy character video with agent-media in one call: a real-looking adult with ONE absurd trait (two heads, four arms, a unibrow, a mini me), a named character sheet saved as a reusable star, and a 10 second vertical clip with sound. Use when the user asks for a viral ad, a crazy or weird character, an AI influencer, or a recurring character for TikTok, Reels or Shorts.'
allowed-tools: ['mcp__agent-media__list_crazy_traits', 'mcp__agent-media__create_crazy_character', 'mcp__agent-media__get_run_status', 'mcp__agent-media__list_my_stars', 'mcp__agent-media__crazy_scene']
x-skill-slug: 'make-viral-ad'
x-skill-version: '2.7.1'
---
# Make a crazy character video

The format: a photoreal, ordinary adult with exactly ONE absurd physical trait, filmed like a friend's phone video in a real, slightly trashy place. The trait is the hook. The character is a recurring star: it is saved with its named character sheet, and every later clip reuses it.

agent-media does the whole recipe server-side in ONE call (one take, the named landscape sheet, the saved star and a 10 second clip with sound). Do not rebuild it by hand with `generate_image` and `generate_video`.

## Who can use it

Accounts with a plan or credits. A new character with its first clip costs 725 credits ($7.25); a new clip of a saved star costs 600 credits ($6.00). If the account cannot pay, say so plainly, point to the billing link the tool returns, and stop.

Free accounts cannot cast from an agent: the tools answer `VIRAL_CAST_NEEDS_PLAN` and nothing is charged. A new account's one free crazy character clip, if it is still unused, is made in the agent-media app: https://app.agent-media.ai/start/character. Tell the user that, once, and do not try to make the clip another way.

## The flow (one step, price first)

1. **See the menu.** Call `list_crazy_traits`. It lists every trait with what it looks like and whether it is castable, the catalog whos, the places with their moves, the prices, and whether this account can cast now.
2. **Pick WHO.** Either a catalog who id in `who` (for example `casino_granny`), or a NEW adult described in a few plain words in `who` (for example "a 60-year-old lifeguard") together with `who_gender`, `who_age` (21 to 90), and optionally `outfit`. Original people only: no celebrities, real people, look-alikes, franchise characters or brands. Describe positively ("bald", not "no hair").
3. **Pick ONE trait** in `trait`, a castable one from `list_crazy_traits` (now: two_heads, three_heads, four_arms, giant_beehive, pink_mohawk, bowl_cut, giant_quiff, ten_foot_hair, handlebar_mustache, unibrow, giant_glasses, mini_me, giant_white_moustache). Never stack two traits.
4. **Place and move** are optional: `place`, `move`. Left out, they are picked to suit the who. `move` needs `place` and must be one of that place's moves.
5. **Quote.** Call `create_crazy_character` WITHOUT `confirm`. It returns the server's price (725 credits) and creates and charges nothing. Show the price and wait for a clear yes.
6. **Cast.** Call `create_crazy_character` again with the SAME input plus `confirm: true` and a fresh UUID in `request_id`. The quoted price is sent along, so a price that moved in between is refused (`PRICE_CHANGED`) before any charge. The 725 credits are charged once, and refunded if the job fails.
7. **Wait.** Call `get_run_status` with the returned job id (`wait: true`) and keep calling until it is completed or failed. A whole character takes a while: say so once and do not spam status. Hand over the video link only when you hold it. The status also shows the character's name, the take photo and, once the sheet is saved, the sheet and the star's `character_id`.
8. **More clips.** `crazy_scene` with `character_id` (from the status or `list_my_stars`) makes a new 10 second clip of the same star, 600 credits, with the same gate: first without `confirm` for the price, then with `confirm: true` after a yes.

## Format rules

- ONE absurd trait on an otherwise real-looking adult; the goal is a clip that opens close (waist or knees up), never from far away (the prompts aim for it, the render does not guarantee it); a mundane, minimal outfit inside platform rules, a different outfit per clip; any caption you write is short, deadpan and carries an AI label (for example "AI"); never write an em dash.
- The trait must read as EXTREME at phone size; a mildly odd person is worthless.
- One face per video, no split screens or panels. No swapping a character onto someone else's footage.
- Every clip carries a burned-in "AI · agent-media.ai" mark. Do not promise an unmarked clip.

## Errors

| Code | What to do |
|---|---|
| `VIRAL_CAST_NEEDS_PLAN`, `VIRAL_SCENE_NEEDS_PLAN` | This needs a plan or credits (https://app.agent-media.ai/billing). A new account's one free crazy character clip, if it is still unused, is made in the agent-media app instead: https://app.agent-media.ai/start/character |
| `INSUFFICIENT_CREDITS` | Not enough credits. Tell the user they can add credits or a plan at https://app.agent-media.ai/billing; do not retry until they have. |
| `PRICE_CHANGED` | The price changed since the quote. Call again WITHOUT confirm, show the new price, and confirm only after the user agrees. |
| `VIRAL_TRAIT_OFF` | That trait cannot be cast right now. Call list_crazy_traits and pick a trait marked castable. |
| `VIRAL_COMBO_REFUSED` | That who, trait, place and move do not go together. Leave place and move out, or pick them from list_crazy_traits. |
| `VIRAL_WHO_REFUSED`, `VIRAL_WHO_WORDING`, `LIKENESS_REFUSED` | Rewrite the who: an original adult in plain, positive words, nothing sexual, no real people or brands. |
| `MODERATION_UNAVAILABLE` | The description could not be checked right now. Nothing was charged. Try again shortly, or use a catalog who. |
| `CASTING_DAILY_LIMIT` | The daily casting limit for this account is reached. Try again tomorrow (the day turns over at midnight UTC). |
| `CASTING_BUSY`, `TOO_MANY_ACTIVE_RENDERS` | Too many renders are running on this account. Wait for one to finish (get_run_status), then try again. |
| `STAR_NOT_FOUND`, `STAR_NOT_VIRAL`, `STAR_SHEET_MISSING`, `STAR_TRAIT_RETIRED` | That character cannot shoot a new scene. Call list_my_stars for the characters this account can use. |
| `VIRAL_CHARACTER_GENERATION_DISABLED`, `VIRAL_PAID_CASTING_DISABLED` | Crazy characters are paused right now. Nothing was charged. Tell the user and try again later; do not make the clip another way. |
| `IDEMPOTENCY_CONFLICT` | That request_id was already used for a different request. Use a new request_id for a new character. |
| `SUBMISSION_UNCONFIRMED` | The answer was lost after a confirmed call. Call again with the SAME input, confirm:true and the SAME request_id to recover the job without a second charge. |

## Same thing over REST

`POST /v2/quote/viral-character` (the price, nothing charged) and `POST /v2/viral-character` with `stage: "cast"`, a catalog `archetype_id` OR a `custom_who` object (`description`, `gender`, `age`, optional `outfit`), `feature_id`, optional `scene_id` and `motion_id`, `expected_credits` (the quoted price) and an `Idempotency-Key` header. A new scene is `stage: "scene"` with `character_id`. `GET /v2/viral-character/stars` lists the saved stars.
