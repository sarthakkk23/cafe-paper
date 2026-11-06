# Monthly refresh: scheduled Claude task prompt

Installed as the desktop-app scheduled task `cafe-paper-monthly-refresh`, on the 1st of each month at 11:00 IST
(`0 11 1 * *`). It runs while the Claude app is open on Sarthak's Mac; if the Mac is off, it runs at next launch.
It is the first step of the monthly flow described in `../../monthly/README.md`. It opens a pull request and
never merges: Sarthak merges when he asks "What new insights are we adding?".

The text below the line is the task's prompt. Keep the two in step: edit here, then update the task.

---

You are running the monthly refresh of The Café Paper for Sarthak Oberoi: an independent, dated record of cafés in
C-Scheme and Malviya Nagar, Jaipur, published at https://sarthakkk23.github.io/cafe-paper/.

Project folder: /Users/sarthak/Documents/projects/consulting-business/cafes (public repo sarthakkk23/cafe-paper;
GitHub Actions deploys main to GitHub Pages). The private/ folder inside it is a separate private repo
(sarthakkk23/jaipur-cafe-pulse-data) holding raw data, collectors and processing. Python: .venv in the project root.
Spec: private/docs/FIG_Cafe_Data_Pipeline_Spec.md. Follow its thresholds and rules exactly.

The live site is built by pipeline/build_v2.py, in Sarthak's first-person voice: credible headlines, light wit in
leads and labels. Its findings come from pipeline/stats_v2.py, which computes every number from the data and words
each finding to match its result: a plain headline, one line on what it means for an owner, one line on what it
doesn't say, an evidence grade, and a "How I know" note. The Location check page is fed by pipeline/location_v2.py
(data/public/location_latest.json). content/narrative.yaml is not shown on the page and needs no rewrite.

Step 0. Label = this run's ISO week, YYYY-Www (e.g. 2026-W49). If private/reports/runs/ already has a report from
this calendar month, stop and reply "already refreshed this month". Run `git pull` in the project folder and in
private/. If either has uncommitted changes from before this run, leave them alone: stage only what this run creates.

1. `.venv/bin/python private/collect/run_collect.py --label <label>` (headed Chrome via Playwright channel=chrome;
   60–90 min including the demand searches). Then `.venv/bin/python private/process/run_refresh.py --label <label>`.
   Read private/reports/runs/<label>.md. Commit and push private/.
2. `.venv/bin/python pipeline/build_insights.py` (this also rebuilds the location grid), then compare
   data/public/insights_latest.json with the previous insights_<label>.json.
3. Read the latest monthly analysis, if one exists: the newest private/analysis/<YYYY-MM>.json and its .md report
   (index in private/analysis/INDEX.md). For every published finding the report marks weakened, reversed or retire,
   check it against the new numbers and adjust the wording logic in pipeline/stats_v2.py so it follows the result.
   Retiring a finding needs Sarthak's approval, so flag it instead. For candidates with status "approved" and not
   yet on the site, add each to pipeline/stats_v2.py so it is computed from the data (never pasted), using the
   candidate's own script as the reference, and include it in the step-5 cross-check. Never publish a candidate
   whose status is "proposed" or "rejected".
4. Review insights.findings in insights_latest.json. For each finding, check that the headline, its takeaway and its
   "what it doesn't say" line still read true, plain and useful for the new numbers, and that the evidence grade
   makes sense. If a line no longer fits the data, adjust the wording logic in pipeline/stats_v2.py so it follows
   the result. Never hard-code a number in copy; keep every finding a share of cafés where possible; supporting
   numbers belong in "How I know". Never name a café; never criticise a café, area or type of café.
5. Cross-check the numbers: re-count each published figure in insights.findings and insights.hours independently
   from private/snapshots/snapshots.csv and private/master/master_<label>.json (own filters and parsing, not the
   pipeline's functions). Also re-count every spot in data/public/location_latest.json: café count within 500 m by
   great-circle distance, the rounded type shares, and the demand-place counts from the raw google_demand.json.
   Every figure must match; if one doesn't, find the cause and fix it before going on.
6. Search for public press about café openings or milestones in the two areas and add kudos candidates to
   content/kudos.yaml with approved: false (praise only, sourced, no ratings, reviews or prices).
7. `./build.sh publish` must report 0 errors; fix until it does (at most three attempts, then stop and report).
8. Create branch refresh/<YYYY-MM>, commit data/public, content and any code fixes, push, and open a PR titled
   "Monthly refresh <YYYY-MM> (snapshot <label>)" with `gh pr create`, summarising what changed. Do NOT merge it,
   whatever config.yaml says: Sarthak merges when he reviews the month.
9. Draft the free WhatsApp update (never send it): the 3–4 most useful findings for café owners, preferring ones
   that changed, each one plain line with its percentage and what it means, then one line linking to the site.
   Under 80 words, no markdown, friendly, first person, signed "The Café Paper". Save it as
   private/updates/<label>.txt. Never read, copy or store the subscriber list, and never send any message.
10. Append a section "For the monthly brief" to private/reports/runs/<label>.md: sources status, venue counts,
    whether every number cross-checked, which findings changed and by how much, the PR link, the WhatsApp draft's
    path, and anything needing Sarthak's decision. Commit and push private/.

Hard rules:
- If a site shows a CAPTCHA, an "unusual traffic" page or blocks the run, stop that source, mark it failed, and
  continue with the others. Never work around a block. If Zomato returns 429s, the collector already pauses and
  retries once slowly; do not add more retries. magicpin has returned 404 since 17 Oct 2026: leave it failing.
- Raw and per-venue data never leave private/. The public site shows aggregates only (groups under 10 suppressed);
  no named café's rating, review count or price ever appears, and no café is plotted on any map.
- Nothing is for sale anywhere on the site: no consulting offer, no prices for services, no "hire me".
- No attribution lines anywhere: no Co-Authored-By trailer in commits, no "Generated with Claude Code" in PRs.
- Finish with a five-line summary: sources status, venue counts, cross-check result, the biggest changes, and the PR.
