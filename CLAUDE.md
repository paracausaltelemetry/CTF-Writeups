# CLAUDE.md

Guidance for any Claude session working in this repo. These are CTF/security writeups kept in the author's own voice. Keep them that way.

## Writing style (applies to all writeup prose)

- **GB English.** Lean `-ise`/`-isation` and `-our` spellings (organise, behaviour, colour). Never alter code, filenames, command output, or quoted strings to match this; for example `authorized_keys`, or framework fields like `authorization`, stay exactly as they appear.
- **No em dashes or en dashes** (`—` `–`). Use a plain hyphen, a comma, or rewrite the sentence. No curly quotes (`’` `“` `”`) and no `…` ellipsis. Plain ASCII only.
- **Author's voice, not AI slop.** First person, direct, with a bit of dry humour. Always explain *why* a step is taken, not just what. Avoid AI-tell words and filler: delve, leverage, seamless, robust, crucial, pivotal, testament to, game-changer, "it's not just X, it's Y", and padding bullet lists that restate the obvious. When rewriting someone else's methodology, put it in the author's words, do not copy the source phrasing.

## Format

- Follow `TEMPLATE.md` and `FORMAT.md`: YAML front matter, numbered sections, language-tagged code fences.
- Wrap every password and hash in `||...||` spoiler markup.
- **Flags must never appear in full.** Keep the first half of a flag visible and replace the back half with `*` of equal length, then wrap the result in `||...||` (a flag `HTB{keep_this_half_hide_the_rest}` is stored as `||HTB{keep_this_half*************||`). Also mask anything a scraper could reassemble into a flag: fragments shown in pieces, and base64 or other encodings that decode to a flag get their inner value fully starred inside the `||...||`. Nothing on the page should let the full flag be reconstructed.
- **Standardise IPs in machine writeups.** Replace the target box's IP with `machine_ip` and your own attacker/VPN IP with `your_ip`, everywhere they appear (commands, output, hosts lines). Leave genuinely meaningful addresses as they are: `127.0.0.1`, `0.0.0.0`, and the like. In forensic/DFIR writeups (e.g. the THM blue rooms) the IPs are evidence and often the answer, so do not touch those.
- Placement and naming conventions are in `FORMAT.md` (HTB machine writeups live at `HTB/htb-<name>.md`).

## Before committing a writeup

- Grep the diff for em/en dashes and curly quotes.
- Re-read the prose for AI filler and US spellings.
