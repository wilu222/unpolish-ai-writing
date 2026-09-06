# Notes (not part of the skill)

Background for humans. **Do not load this file when running unpolish.** Install copies only `SKILL.md` and `references/`.

## AI writing tells 

Most of the AI writing tells covered in the skill are already covered by at least one other popular agent skill. However, some tells are new/unique to this repo at the time of writing:

### Metaphorical land / landed

Claude output overuses *land* / *landed* for shipping, arriving, or a decision sticking (“commits landed,” “subdomains land in X”). Directional only. Keep literal aircraft, birds, and ground.

- Chris Richardson, [LinkedIn](https://www.linkedin.com/posts/pojos_is-it-just-me-or-has-land-become-claude-activity-7484768755642830848-pKAB/)

### Verbless noun fragments

Verbless slogan lines (“Less busywork. More impact.”) are a common AI-copy tell. The skill restores a subject and verb instead of leaving bare fragments.

- Andy Chadwick, [LinkedIn](https://www.linkedin.com/posts/andy-chadwick_one-thing-im-getting-increasingly-sick-of-activity-7491523021518753792-7pnX/)


-----

## Imperfections

### Why they are sparse

Informal “mistake” counts mix slang, style, and grammar. Older published-social figures (2011–2013) put residual misspellings in a low range, roughly two to six words per thousand. We use that only as **directional** guidance — those papers predate today’s autocorrect, and they are not a rate we convert. λ ≈ words / 600 (≈ 0.17%) is a conservative editorial choice so a typo does not land in every paragraph (the usual naive-prompt result). Many imperfections are punctuation or capitalization, not misspellings.

- Brandwatch / mycleveragency (2013), via [PCMag](https://www.pcmag.com/news/infographic-twitter-named-most-illiterate-social-network)
- Baeza-Yates & Rello, [ICWSM 2011](https://doi.org/10.1609/icwsm.v5i4.14085)

### Dropped apostrophes vs wrong-key typos

A 2012 UK **SMS** corpus (not published posts) found people often left out apostrophes (`dont`, `im`) and rarely hit the wrong key. Directional only: dropped apostrophe is the heaviest type, keyboard slip the lightest. SMS is a different register, and we do not take their mix as weights.

- Tagg, Baron & Rayson, *[“i didn’t spel that wrong did i. Oops”](https://eprints.lancs.ac.uk/id/eprint/60484/)*
