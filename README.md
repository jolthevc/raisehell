# RAISE HELL
### A little fire for your day.

**An editorial brand built on a simple proposition: you get one shot at being alive. Do not sleepwalk through it.**

RAISE HELL publishes compact, exceptionally written, emotionally potent messages, primarily as plain white type on black Instagram posts. Its handle direction is **@raisehelldaily** (availability not verified). Its mark is one burning match. Its product is not advice volume or decorative design. It is the moment a few words cut through somebody's day.

The writing is raw, sometimes profane, irreverent, confident, masculine-leaning, and emotionally intelligent. It celebrates ambition, courage, love, self-respect, humor, and being alive. It is **not** a red-pill, grievance, misogyny, contempt, or black-pill account. The reader should leave feeling more alive and more capable, not bitter.

## What this repository is
The editorial source of truth for RAISE HELL, plus a lightweight [editorial console](index.html) for GitHub Pages. The console calls the existing n8n Generation webhook and Zernio; it does not require a separate application server. See [SETUP.md](SETUP.md) for activation and limitations.

## Start here, in order
1. [AGENTS.md](AGENTS.md) for AI operating instructions, required reading, requests, outputs, and source precedence.
2. [Brand identity](brand/identity.md), [manifesto](brand/manifesto.md), and [audience and promise](brand/audience-and-promise.md).
3. [Editorial philosophy](literature/editorial-philosophy.md) and [voice and tone](literature/voice-and-tone.md).
4. [Emotional engineering](literature/emotional-engineering.md), [content territories](literature/content-territories.md), and [post archetypes](literature/post-archetypes.md).
5. [Literary techniques](literature/literary-techniques.md), [hooks and endings](literature/hooks-and-endings.md), and [language and style](literature/language-and-style.md).
6. [Anti-patterns](standards/anti-patterns.md), [quality rubric](standards/quality-rubric.md), [editorial balance](standards/editorial-balance.md), and [Gold Standards](standards/gold-standards.md).
7. For execution: [ideation](workflows/ideation.md), [drafting and revision](workflows/drafting-and-revision.md), and [final review](workflows/final-review.md).

## Editorial contract
- A powerful human truth comes first. Profanity, punchiness, and technique are supporting instruments.
- Write as a perceptive person speaking to another person, not a coach shouting slogans.
- Preserve range: defiance and discipline are essential; so are tenderness, humor, family, love, mortality, regret, gratitude, and wonder.
- Use simple language with sophisticated underlying thought. Keep the machinery invisible.
- A post can be one line or several short paragraphs. Form follows the thought.
- No weak or derivative work gets published just because a quota exists.
- Publication requires human approval. Drafts are not approved pieces; examples in manuals are not proven hits.
- [Gold Standards](standards/gold-standards.md) starts empty. Only actual published posts may be added by the owner, including through an expressly enabled, owner-configured performance threshold in the console. No synthetic winners.

## Brand snapshot
| Dimension | Direction |
|---|---|
| Name | RAISE HELL |
| Tagline | A little fire for your day. |
| Profile symbol | A single burning match |
| Social format | Minimal black background, high-contrast white type |
| Primary platform | Instagram |
| Distribution | Shareable standalone text posts |
| Personality | Raw, witty, bold, fiercely alive, unsentimental but caring |
| Philosophy | Raise Hell is not being angry at the world. It is refusing to sleepwalk through your one shot at being alive. |
| Creative aspiration | Make someone feel understood, then make them feel capable of something bigger. |

## How to ask an AI to work here
- **Generate:** "Read AGENTS.md and the required doctrine. Give me 12 genuinely distinct RAISE HELL candidates, with strong emotional variety, then self-edit and shortlist the best 4."
- **Revise:** "Read the editorial standards. Diagnose this post and give me 3 materially different improvements without polishing its humanity away."
- **Review:** "Evaluate these posts using the quality rubric and anti-pattern vetoes. Do not equate automated scores with actual engagement."
- **Learn:** "Review these owner-confirmed performance results and suggest evidence-based lessons. Do not edit Gold Standards unless explicitly asked."

## Status
**v1.1: Literary operating system and lightweight HTML console.** The root `index.html` is the deployable one-file console. See [SETUP.md](SETUP.md). The console is not a background automation service; integrations require live account verification.
