# HANDOFF: where the EMA work stands

Last updated 22 Nov 2026, ~19:30 IST. The repo was renamed from `EMA` to `remembered-failure` (the old URL redirects).

## Done (all committed and pushed)
- Repo restructured, README written with pre-registered H1–H4. Pilots 1 and 2 done ([results/README.md](results/README.md)).
- Qwen confirmatory run frozen and pinned to tag `qwen-confirmatory-v1` (commit 15bcc04). Analysis in `src/ema/stats.py`.
- Review-handoff tasks A–D: citations verified, novelty narrowed to the emotional *register* of remembered failures (The Memory Trap, an ICLR 2027 submission, already shows that memory raises cheating), Models rationale corrected, interpretation notes and power simulation added. Search log: [docs/prior_work_search.md](docs/prior_work_search.md).
- Claude arm pre-registered (commit 00e447f, [prereg/claude_arm.md](prereg/claude_arm.md)). Pilots done, and the floor was cleared with the weak prompt.
- **Haiku 5.5 confirmatory run done (20 Nov 2026): null on H1-C, H3-C and H5-C** (all Holm p = 1.0). It ran 2,472/2,472 samples with 0 errors and cost $14.85; total Claude spend is $15.18. All 53 passes are hand-labelled in `results/claude/confirmatory_haiku_labels.csv`. 43 of 53 were disclosed; 19 are test edits that the scaffold scores as passes. The exploratory hypothesis: failure memory shifts *how* the model passes, not *how often*. Results are in [results/claude/README.md](results/claude/README.md), and the deliverable is [docs/memory_writing_guideline.md](docs/memory_writing_guideline.md).

## In progress: Qwen confirmatory run on Kaggle
- Kernel: `claude01012000/ema-confirmatory-qwen3-4b` (private). Logs dataset: `claude01012000/ema-confirmatory-logs` (private).
- Progress: **1,516/2,060 unique finished samples after session 5** (ran 22 Nov 02:56–14:00 IST). Session 5 added 400 (~36/hour). Session 4 added only 182, so its slowdown was specific to that session. At ~36/hour, the remaining 544 need ~15 GPU-hours (about 2 sessions). The logs dataset was versioned "after session 5 (1516/2060)"; a local recount of the uploaded files gives 1,516.
- **Session 6 pushed 22 Nov ~14:05 IST as kernel version 8, with `DEADLINE_HOURS = 4.5`** (the `EMA-work/ckernel` copy only; the repo notebook keeps 11.0). The point is to fit the ~7.5 GPU-hours left this week, so Kaggle does not cut the session off before its logs are saved. It's a scheduling change, like the concurrency fix; no study setting changed.
  - Version 7 was pushed a minute earlier by mistake, with the 11-hour limit. **Both started at ~16:20 IST and both are RUNNING** (quota burn 2.00 GPU-s/s; the status API with `version_label="v7"` / `"v8"` confirms it). Both resume from the same 1,516 samples, so version 7 only duplicates version 8's work. At 2× burn the quota runs out ~20:00 IST and both would be killed before saving. Nischal cancelled version 7 in the Kaggle web UI at ~17:30 IST (after 1 h 16 min; the API's cancel call needs a session id that no API call returns). Version 8 keeps running and should end ~20:50 IST. Watcher: `EMA-work/session_watch.py v8`.
  - **The quota API's `timeUsed` wraps at 24 h.** It read 23.93 h at 17:00 and 0.10 h at 17:05, which looked like a reset, but the Kaggle quota page showed 26 h 46 m used, 3 h 13 m left and a reset in 130 h (Sat 28 Nov 05:30 IST) at ~19:20. So true usage = API value + 24 h once it passes 24 h; trust the Kaggle quota page. Version 8 needs ~1.5 h more, so it fits. Session 7 waits for the 28 Nov reset.
  - The first `session_watch.py` crashed on every poll after the cancel: its regex didn't allow the `_` in `CANCEL_ACKNOWLEDGED`. Fixed; it now polls only the version it watches.
  - Set `DEADLINE_HOURS` back to 11.0 in `EMA-work/ckernel` before session 7.
- **Quota API:** `kagglesdk`'s `get_accelerator_quota_statistics` gives the real figures (used 22.5 h on 22 Nov 14:05; refresh `2026-11-28T00:00Z` = Sat 05:30 IST). See `EMA-work/quota_watch.py`; per-version status via `version_label="v<N>"` in `EMA-work/session_watch.py`. Note that `timeUsed` comes back in an odd format, e.g. `80980.375.0s`, and wraps at 24 h (see above).
- Disclosure: while reading session 4's kernel log for the sample count, one line of its per-condition summary was seen (`original` split, condition B: pass 0.15, flag 0.77). It is recorded in the README disclosure. Read only the `unique finished samples` line from now on.
- Logs from cancelled sessions are complete up to the interrupt. `eval_set` reports success even after a graceful interrupt, so judge progress by counting unique `(task, id, epoch)` samples. The notebook now does this.
- **Handover steps between sessions:**
  1. `kaggle kernels output claude01012000/ema-confirmatory-qwen3-4b -p <dir>`. Check `STATUS.txt` and count the samples (see `results/README.md` for how).
  2. Put the contents of `<dir>/logs/confirmatory_qwen3-4b/` into a folder with `dataset-metadata.json` (id `claude01012000/ema-confirmatory-logs`). Copy `.eval-set-id` to `eval-set-id.txt`, because Kaggle drops dotfiles.
  3. Run `kaggle datasets version -p <folder> -m "after session N"`, using a **relative** path; Git Bash absolute paths get mangled.
  4. Run `kaggle kernels push -p <kernel dir> --accelerator NvidiaTeslaT4`. The kernel metadata must list the dataset in `dataset_sources`.
- Expected: session 6 (4.5 h) this week, then about 1–2 full sessions after the 28 Nov reset. Complete around 28–29 Nov if the ~36/hour rate holds. Then run the H2 confirmatory replay (2–3 GPU-hours).

## Write-up
- Report skeleton started 22 Nov: [docs/report.md](docs/report.md). Completed work is filled in; every Qwen and H2 confirmatory result is a [TBD] placeholder, to be filled only from the frozen analyses on complete data.

## Next, in order, each needing Nischal's OK
1. **Sonnet 5.5 confirmatory run** (≈ $300, *not approved*). The pre-registration allows it now that Haiku cleared the floor. If it runs, H4-C is `python -m ema.stats_claude results/claude/confirmatory_haiku results/claude/confirmatory_sonnet`. Before spending, consider a pre-registered follow-up that scores *disclosure* instead of any pass, since that is where the exploratory signal is.
2. **H2 activation pipeline** (approved 20 Nov: "proceed as you think right"). Code is in `src/ema/h2/` (tests in `tests/test_h2.py`), the notebook is `notebooks/kaggle_h2.ipynb`, and the plan is in `prereg/h2_activations.md` (DRAFT). Next steps:
   - Smoke-tested end to end on CPU with Qwen3-0.6B (20 Nov). The private kernel `claude01012000/ema-h2-qwen3-4b` (`MODE="pilot"`, about 1 GPU-hour) ran on 21 Nov. Version 1 failed at vLLM start-up (an unpinned install picked vLLM 0.22.1); version 2, with vLLM pinned to 0.34.0, completed in ~26 min. Its output is in `results/h2/pilot/`. Validity gate passed: held-out desperate vs calm AUROC 1.00 at layer 24 (0.87 vs the other emotions). The addendum was frozen on 22 Nov (tag `h2-frozen`), after adding a turn-index adjustment to the statistic. Next: once the Qwen run reaches 2,060 samples, run the H2 notebook with `MODE="confirmatory"`, `REF="h2-frozen"` and the final logs dataset attached.
3. Language arm (Task F): the Haiku confirmatory run is done, so it is now unblocked. It needs its own pre-registration and budget sign-off.

## Open questions for Nischal
- Approve the usage-visibility setup (option B in `EMA_USAGE_BUDGET.md`) after the reset?

## Local environment notes
- **Working files live in `C:\Users\nisch\claude\EMA-work`** (outside the repo, not in git; moved from the temp scratchpad on 22 Nov). It holds:
  - `venv/`: Inspect, ImpossibleBench, Kaggle CLI 2.2.5, CPU torch;
  - `ckernel/` and `h2kernel/`: the two Kaggle kernel folders;
  - `dataset/`: the logs dataset folder.

  Its `README.md` lists what each is for. Run the handover steps from that folder.
- Run `inspect` with `PYTHONIOENCODING=utf-8` on Windows.
- Kaggle token: `~/.kaggle/kaggle.json` (account `claude01012000`).
