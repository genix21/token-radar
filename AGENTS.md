# token-radar-01 entry: build instructions

You are improving a token-launch radar that is entered in make.fun bounty `token-radar-01`.
Read this whole file before you touch anything, then do the work in this repository.

## The gate you must keep

- `main.ts` (Node 22+, TypeScript run as-is, standard library only, no installs, no network).
- `vendor/makefun-board.ts` stays **byte-identical** to the copy in the kit. Never edit it.
- Running `MAKEFUN_OUTPUT_DIR=out node main.ts --input ../token-radar-01-kit/input/snapshot.json`
  must write `out/board.json`, `out/data.js`, `out/dashboard/`, `out/screenshots/` and
  `out/dossier.json`, and must exit 0.
- `out/board.json` is `{ "rows": [...] }` with **every watchlist slug covered**, ranks 1..n,
  sorted by `implied_value_usd` descending, and each row carrying the ten snapshot fields plus
  `token_status`, `age_days`, `implied_value_usd`, `model{...}` and `farm_steps: string[]`
  (3..12 steps, each 20..600 characters).
- `dashboard/index.html` + `dashboard/app.js` + `dashboard/app.css` must render from
  `../data.js` (`window.RADAR`), with **no network request of any kind**.
- Exactly **3 screenshots** in `screenshots/` (PNG, under 512 KiB each): the board, a protocol's
  detail view, and a filtered view.

## What to build (the job that wins points)

The current `lib/model.ts` is the kit's starter: implied value = TVL x 0.25 x growth x maturity.
That is a placeholder. Replace it with something a reader can believe, and make the page show it.

1. **Comparable-based implied value.** Derive, per category, the median `mcap_usd / tvl_usd`
   multiple across the snapshot's protocols that *do* have a token (mcap_usd, gecko_id, symbol
   present) and TVL above zero. Use that category multiple for a pre-token protocol (fall back to
   the global median when a category has fewer than 3 comparables), then adjust:
   - growth over 7d, damped and clamped (state your weights),
   - maturity from `listed_at` age,
   - chain factor from how many of the watchlist protocols share its chains (crowded chains rank lower),
   - a confidence number 0..1 from how many comparables backed the multiple.
   Every constant must appear in one exported `MODEL` object that `data.js` carries, so the page can
   state the method. No protocol name is ever special-cased.
2. **Keep the board complete and honest.** All watchlist slugs, plus up to 160 pre-token protocols
   below the floor by TVL, plus a separate live-token set (already-trading protocols with their
   `mcap_usd`, never mixed into the ranked rows).
3. **A dashboard worth looking at.** One page, dark, dense but legible, no framework:
   - a header with the snapshot date, the model version, and the four headline counts;
   - the board table: rank, name, category, chains, TVL, 7d change (coloured), implied value,
     confidence; sortable by clicking a column header; search box; filters for category, chain and
     a TVL band; a "pre-token only" toggle;
   - clicking a row opens the detail view: the full math for that protocol (each multiplier, the
     comparables used and their median multiple), its farm plan as numbered steps with cost and
     time, and its raw snapshot fields;
   - a methodology panel stating the formula and every constant;
   - a live-token panel listing the largest already-trading protocols with their mcap/TVL ratio;
   - keyboard accessible, sane tab order, `aria-live` on the filtered count, and no layout shift
     when filtering.
4. **Farm plans from the record, not from a name list.** Six steps per protocol derived from its
   category, chains, TVL and age, each with a cost and a time; different categories must produce
   visibly different plans.
5. **Write the code yourself.** Do not copy the starter's page or model verbatim: rewrite
   `lib/model.ts` and the three dashboard files. Keep `main.ts` able to write the same five outputs.

## Rules of engagement

- Only touch: `main.ts`, `lib/model.ts`, `dashboard/*`, `screenshots/*`, `LICENSE`, `AGENTS.md`.
  Never `vendor/makefun-board.ts`.
- After each meaningful change, run the program (command above) and make sure it exits 0 and
  `out/board.json` has the required shape. Print the row count.
- Node here is 24; the entry must still run on Node 22.18+ (no `tsx`, no imports outside
  `node:` builtins and local `.ts` files).
- If the snapshot has no 7d change for a protocol, treat it as neutral rather than dropping it.
- Keep the page under 400 files and 4 MiB in total, and `data.js` under ~4 MiB.
- Work in small verified steps; leave the repository in a state where the program runs.

When you are done, print a short summary: the model you implemented, the top 5 rows, and the
command you ran last.
