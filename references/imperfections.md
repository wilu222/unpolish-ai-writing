# Imperfections (read only when the user asked for imperfections)

Sparse hash-drawn imperfections. **λ ≈ max(0.05, word_count / 600)**. Soft skip is only Poisson P(k=0). Use **SHA-256 only** (no wall-clock, no djb2). When the user says “reroll” / “vary it,” invent an internal salt (`reroll1`, their phrase, …); otherwise `salt` is empty.

Never-touch: names, numbers, quotes, titles, URLs, slur-adjacent.

## Hash recipe

1. **Normalize** the stripped rewrite: lowercase, collapse whitespace runs to a single space, keep the full string → `text`
2. **Seeds:** `count_seed = "count|" + text + "|" + salt`; per slot `s` (0-based): `slot_seed = "slot|" + str(s) + "|" + text + "|" + salt`
3. **Digest words:** `digest = SHA-256(UTF-8 bytes of seed)` (32 bytes); `W0…W5` = uint32 big-endian from `digest[0:4]`, `[4:8]`, … `[20:24]`
4. **Poisson count:** `u = W0_count / 2^32` from `count_seed`; `k` = smallest `m ≥ 0` with `cdf_poisson(m; λ) ≥ u`
5. For each slot `s` in `0 .. k-1`: hash `slot_seed` → words; `r = W0 % 100` → type band; walk if needed (except missing-end-punct → none); apply W1–W5 per type. Choose all targets against the immutable pre-imperfections draft; avoid overlaps; apply high-to-low offset. Report `imperfection: none` or `imperfection: <type> @ …`

```mermaid
flowchart TD
  draft[Stripped draft] --> normalize[Normalize full text]
  salt[Optional internal salt] --> countSeed
  normalize --> countSeed["Seed: count pipe text pipe salt"]
  countSeed --> countHash[SHA-256]
  countHash --> countRoll["W0 / 2^32"]
  countRoll --> poisson["Poisson count lambda = max(0.05, words / 600)"]
  poisson --> slots[Create k slots]
  normalize --> slotSeed["Seed: slot pipe index pipe text pipe salt"]
  slots --> slotSeed
  slotSeed --> slotHash[SHA-256]
  slotHash --> typeRoll["W0: imperfection type"]
  slotHash --> locationRoll["W1: sentence or region"]
  slotHash --> candidateRoll["W2: eligible candidate"]
  slotHash --> subtypeRoll["W3: wrong-word pool or keyboard subtype"]
  slotHash --> mutationRoll["W4 W5: character and neighbor"]
```

## Types (weighted CDF)

`r = W0 % 100` from the slot digest, then walk bands (except missing-end-punct). If a type can’t apply, try the next band; if none apply → that slot is none.

| `r` | Weight | Type | How to apply |
| --- | --- | --- | --- |
| 0–27 | 28 | Misspelling / wrong word | Two pools below; `W3 % 10` and `W2`. Report id: `wrong_word`. |
| 28–51 | 24 | Dropped apostrophe | `W1` → sentence; `W2` → eligible contraction (`don't`→`dont`, `I'm`→`Im`, `that's`→`thats`). Prefer these over ambiguous `it's`→`its`. |
| 52–69 | 18 | Missing end punctuation | Eligible = paragraph-final sentences ending in `.` or `?`. `W1 %` over that pool; remove the mark. Empty (or all claimed) → `imperfection: none` — **do not walk**. |
| 70–83 | 14 | Uncapitalized sentence start | `W1` → eligible sentence; lowercase first letter (never standalone `I`). |
| 84–95 | 12 | Extra space | `W1` → sentence; `W2` → which single space becomes two (`I  think`). |
| 96–99 | 4 | Keyboard slip | `W3` picks subtype; `W2`/`W4`/`W5` pick word/char/neighbor. |

Word use by type:

| Type | W0 | W1 | W2 | W3 | W4 / W5 |
| --- | --- | --- | --- | --- | --- |
| Misspelling / wrong word | type band | (optional region) | pool index | pool prefer `W3%10` | — |
| Dropped apostrophe | type band | sentence | contraction | — | — |
| Missing punctuation | type band | paragraph-final index | — | — | — |
| Lowercase start | type band | sentence | — | — | — |
| Extra space | type band | sentence | which space | — | — |
| Keyboard slip | type band | sentence | word | subtype `W3%2` | char / neighbor |

## Misspelling / wrong word — two pools

Do **not** invent a misspelling of a word that isn’t in the draft. Scan into two pools, then:

1. `p = W3 % 10` — if `p == 0` (~10%) prefer **Pool B**; else prefer **Pool A**
2. If Pool A is preferred and empty → walk to the next imperfection type (**do not** fall back to Pool B). If Pool B is preferred and empty → may fall back to Pool A; if that is also empty → walk bands
3. `idx = W2 % pool.length` — one canonical form only (one row per source type that appears; do **not** give each occurrence of `to` its own vote)
4. Apply that slip once (first occurrence in the chosen region)

**Pool A — wrong word / mix-up** (`W3 % 10 ≠ 0`, ~90%)

| Source (must already appear) | Slip |
| --- | --- |
| could have / could've | could of |
| should have / should've | should of |
| would have / would've | would of |
| might have / might've | might of |
| must have / must've | must of |
| a lot | alot |
| in fact | infact |
| as well | aswell |
| lose (verb: fail to keep / be defeated) | loose |
| losing (same meaning) | loosing |
| than (comparison only: bigger than, rather than) | then |
| their | there |
| there | their |
| they're | their |
| your | you're |
| you're | your |
| its | it's |
| it's | its |
| to | too |
| too | to |
| affect | effect |
| effect | affect |
| of | off |
| off | of |

**Pool B — conventional nonword misspellings** (`W3 % 10 == 0`, ~10%)

| Source (must already appear) | Slip |
| --- | --- |
| definitely | definately |
| separate / separately | seperate / seperately |
| necessary | neccessary |
| receive / received | recieve / recieved |
| believe / believed | beleive / beleived |
| occurred | occured |
| weird | wierd |
| argument | arguement |
| beginning | begining |
| tomorrow | tommorrow |

Dropped-apostrophe already covers `Im` / `dont` / `thats` (prefer those contractions over wrong-word `it's`↔`its` when both could apply).

## Sentence pick

- Split on `.` `?` `!` into sentences; trim empties
- Eligible = body sentences (skip mostly-quote or URL-heavy sentences)
- **Missing end punctuation:** eligible = paragraph-final sentences ending in `.` or `?` (blank line or end of draft). `W1 %` over that pool only. Empty or all claimed → that slot is `imperfection: none` (no walk)
- Index = `W1 % eligible_count`
- When count ≥ 2, skip a sentence already claimed by an earlier slot; if none left for this type, walk bands (except missing-end-punct → none)

## Keyboard slip

Two subtypes (from `W3 % 2`):

| `W3 % 2` | Subtype | Example |
| --- | --- | --- |
| 0 | Adjacent-letter **transposition** | `the`→`teh`, `and`→`adn` |
| 1 | QWERTY-neighbor **substitution** (same-row only) | `should`→`shuild` |

Shared picks:

1. Eligible words in the chosen sentence: length ≥ 3, not never-touch → `W2 % count`
2. Interior character index → `W4 % (len - 2)`, applied at position `1 + that`
3. Neighbor / direction → `W5`

For transposition: swap the chosen interior letter with the next letter (if at last interior, swap with previous).

For substitution: replace the chosen letter with a **same-row horizontal** QWERTY neighbor; neighbor index = `W5 % neighbor_count`.

| Key | Neighbors (left/right only) |
| --- | --- |
| a | s |
| b | v n |
| c | x v |
| d | s f |
| e | w r |
| f | d g |
| g | f h |
| h | g j |
| i | u o |
| j | h k |
| k | j l |
| l | k |
| m | n |
| n | b m |
| o | i p |
| p | o |
| q | w |
| r | e t |
| s | a d |
| t | r y |
| u | y i |
| v | c b |
| w | q e |
| x | z c |
| y | t u |
| z | x |

## Collision handling

1. Choose **all** slot targets against the immutable pre-imperfections draft (character offsets or stable sentence+token indices).
2. Reject a target that overlaps an earlier slot’s span; re-walk type bands for that slot if needed (except missing-end-punct → none).
3. Apply surviving edits from **highest offset to lowest**.

## Checked fixtures

UTF-8 SHA-256, big-endian words. Recompute to verify.

**Fixture 1 — often zero at λ≈1**

- `text` = `fixture a short forum draft about a roommate and dishes`
- `count_seed` = `count|fixture a short forum draft about a roommate and dishes|`
- digest prefix `39894103…` → `W0 = 965296387` → `u ≈ 0.22475`
- At λ = 1.0 → **k = 0**. At λ = 1.67 → **k = 1**.

**Fixture 2 — more than one imperfection at long λ**

- `text` = `fixture b longer draft` + 500 spaced `x` tokens (seed shape `count|fixture b longer draft x x … x|`)
- digest prefix `a5472dc4…` → `W0 = 2772905412` → `u ≈ 0.64562`
- At λ = 1.0 → **k = 1**. At λ = 1.67 → **k = 2**.

**Fixture 3 — type band + Pool A prefer words (same seed; band label updated)**

- `slot_seed` = `slot|1|i definitely could have handled that argument better than i did|`
- digest prefix `b28e8162…` → `W0 % 100 = 82` → uncapitalized-start band (70–83)
- `W3 % 10 = 3` → prefer **Pool A** when the type is wrong_word. If Pool A empty, walk — do **not** fall back to Pool B.

**Fixture 4 — type band check (same seed; band label updated)**

- `slot_seed` = `slot|0|fixture a short forum draft about a roommate and dishes|`
- digest prefix `2b20b7b6…` → `W0 % 100 = 70` → uncapitalized-start band (70–83)
