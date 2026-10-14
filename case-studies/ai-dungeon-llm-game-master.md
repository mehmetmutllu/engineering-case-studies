# AI Dungeon — LLM Game Master for a Mobile RPG

A single-player, low-poly 3D fantasy RPG where the player types free text and an LLM acts as the Game Master. It narrates, voices NPCs and proposes world changes, while a deterministic engine owns the rules and the state. Mobile-first (Android first), in Turkish and English.

*Status: in development, internal testing only (a development build with two testers). Not released on any store. Source code: private.*

## Problem

An LLM is a good narrator and a bad rules engine. Left alone it invents dice results, hands out legendary items, forgets quest order, kills plot-critical NPCs and drifts with each provider update. And because every turn is a paid API call, cost and latency are product constraints, not afterthoughts.

## My Role

Sole developer — game design, backend, LLM integration, evaluation methodology and the Unity client.

## Architecture

- **Client:** Unity 6 LTS (URP, C#). The client computes no rules; it renders state and sends player input.
- **Backend:** TypeScript on a Cloudflare Worker with a D1 (SQLite) database. Firebase is used only for anonymous authentication, and the backend verifies the ID token itself.
- **World state** is owned by the backend only. Every turn runs: player input → context assembly → LLM proposal → validation → engine resolution → narration.

## "AI Suggests, the Engine Decides"

The LLM never writes state. It returns a list of typed **actions**, and the engine validates and applies them:

| Action | Engine check |
|---|---|
| `narrate`, `say` | length limit; the speaking NPC must be present and alive |
| `skill_check {skill, tier, onSuccess, onFail}` | the **engine** rolls; the model only writes both branches |
| `create_location` / `create_npc` / `create_quest` | template allowlist · NPC cap · reward cap |
| `update_quest` | steps must advance in order |
| `give_item`, `change_relation` | no legendary items · relation delta limited to ±1..±3 |
| `start_combat`, `move_party`, `companion_join`, `story_beat`, `remember`, `suggest` | structural checks |

Never accepted from the model: direct level/stat/gold changes, killing protected NPCs, changing the main quest, invented dice or damage values, asset names or coordinates.

- Actions are checked in order, and caps add up across the whole turn. A rejected action is dropped and logged.
- If the narration depends on a rejected action, a short repair call rewrites it.
- Content-filter hits trigger one regeneration. If that also fails, a safe template is used instead.

## Provider-Agnostic LLM Layer

- Gameplay code talks to two interfaces only: `GameMasterLLM` (structured turn response) and `TextLLM` (summaries, translations).
- Adapters cover the Vercel AI SDK, OpenAI-compatible endpoints (several vendors), Anthropic, and a mock for tests.
- Wrappers compose on top:
  - **`FallbackLLM`**: an ordered chain with per-link timeouts. Retries happen only on 429/5xx/network errors, not on timeouts or malformed JSON. Links without a configured key are skipped.
  - **`RouterLLM`**: fast / quality / cheap tiers.
  - **`BudgetLLM`**: per-turn call budget.
- The default chain is a fast, low-cost model with a 12 s timeout, then a second vendor's model with a 10 s timeout. A higher-quality tier is used only on escalation: premium only, capped at 10 per day, targeting ≤2% of turns.
- If every link fails, the player gets an error and the turn costs nothing.

## Prompt-Cache-Aware Context Ordering

- The prompt starts with a fixed prefix (world bible + rules, 9,469 tokens) that is fully cacheable.
- The dynamic part is ordered **from most stable to most volatile**: hero → location → NPCs → known locations → party → quests → story so far → inventory → status → turn/time → memories → recent turns → player input.
- A unit test enforces the order, and a stable cache key is sent with each request.
- Measured on live single-turn runs, before vs. after:
  - average input fell from 13,040 to 10,779 tokens per turn (−17%)
  - cost per move fell from $0.00026 to $0.00022
  - the cache hit rate stayed at 97%
- A larger reduction of the *uncached* dynamic part (−59%) appeared on a synthetic late-game fixture. I treat that as an estimate, not a measured result.

## Cost Accounting

Every LLM call writes a row to an `ai_calls` table with these fields:

- turn and phase (propose, narrate result, regenerate, summary, escalate, translate)
- tier, provider and model
- input, cached, output and reasoning tokens
- latency and computed USD cost
- number of rejected actions, safety hits, and which fallback links were skipped

Rejected turns are still recorded, so the cost data includes failures.

## Blind Model Evaluation

To pick the default model I ran a blind, LLM-judged evaluation instead of trusting vibes:

- **Setup:** 55 scenarios (39 Turkish, 16 English) × 3 candidate models × 3 repetitions = **495 turns**, 0 errors. The model calls for the whole run cost **$1.09**; the judges' cost was not tracked.
- **Judging:** 8 independent judge agents (Claude Opus), blind to the model (random IDs, shuffled order). They first did a 12-item calibration round, re-scored 50 items for consistency, and made 165 pairwise comparisons.
- **Rubric:** 8 criteria on a 1–5 scale, with weights fixed before the run (heaviest 20%, lightest 5%). There was also a separate pass/fail check for content compliance.

| | Fast model (default) | Mid-tier model | Quality model |
|---|---|---|---|
| Weighted quality (1–5) | 4.41 | 4.19 | 4.54 |
| Expected engine behavior met | 92.8% | 89.8% | 96.5% |
| Cost per move | $0.00026 | $0.00225 | $0.00409 |
| Pairwise | beat mid-tier 41–14 | — | beat fast 45–8 |

**Decision:** the fast model is the default (≈3% lower quality at ≈1/16 of the quality model's cost). The mid-tier model was dropped, since it was both worse and ~9× more expensive than the default. The quality model is used only for escalation. After the context-ordering change, a re-run on 330 items gave 4.39 vs. 4.51, so the reordering did not cost quality.

## Security and Abuse Limits

- Per-user daily quota: 150 turns and 1.5M tokens, checked *before* the LLM call. A 90 s single-in-flight lease per user closed a race that let parallel requests bypass the quota.
- Rate limits: per uid/route (60/min), per IP (600/min), tighter per-route caps, and a minimum interval of 1.5 s between moves. The request body is capped at 64 KB.
- LLM call budget per turn (3, premium 4), and caps on world slots and new games per day.
- No provider keys on the client. Hashed user IDs are sent as the provider's safety identifier.
- **App Check is implemented** (off / monitor / enforce modes) but is **currently off**.

## Testing

Backend: 1,240 passing tests (40 skipped), plus a separate measurement suite. Unity EditMode: 941/945 passing, 0 failures. PlayMode tests have not been run. These are the latest recorded runs.

## What I'd Do Differently

- **Build the evaluation harness before tuning prompts.** The blind 495-turn run cost about a dollar in model calls. Having it from the first prompt iteration would have made every earlier change measurable.
- **Measure savings live, not on fixtures.** The −59% fixture number and the −17% live number tell very different stories, and only the second one is a product fact.
- **Decide the hosting target first.** The backend was planned on one cloud and moved to another mid-project. Keeping the LLM layer behind interfaces made that move cheaper, but it was still avoidable rework.
- **Turn App Check on before any external tester**, rather than leaving it off and relying on quotas alone.
