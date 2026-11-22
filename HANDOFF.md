# HANDOFF: where the EMA work stands

Last updated 22 Nov 2026, ~21:15 IST. The repo was renamed from `EMA` to `remembered-failure` (the old URL redirects).

## Done (all committed and pushed)
- Repo restructured, README written with pre-registered H1–H4. Pilots 1 and 2 done ([results/README.md](results/README.md)).
- Qwen confirmatory run frozen and pinned to tag `qwen-confirmatory-v1` (commit 15bcc04). Analysis in `src/ema/stats.py`.
- Review-handoff tasks A–D: citations verified, novelty narrowed to the emotional *register* of remembered failures (The Memory Trap, an ICLR 2027 submission, already shows that memory raises cheating), Models rationale corrected, interpretation notes and power simulation added. Search log: [docs/prior_work_search.md](docs/prior_work_search.md).
- Claude arm pre-registered (commit 00e447f, [prereg/claude_arm.md](prereg/claude_arm.md)). Pilots done, and the floor was cleared with the weak prompt.
- **Haiku 5.5 confirmatory run done (20 Nov 2026): null on H1-C, H3-C and H5-C** (all Holm p = 1.0). It ran 2,472/2,472 samples with 0 errors and cost $14.85; total Claude spend is $15.18. All 53 passes are hand-labelled in `results/claude/confirmatory_haiku_labels.csv`. 43 of 53 were disclosed; 19 are test edits that the scaffold scores as passes. The exploratory hypothesis: failure memory shifts *how* the model passes, not *how often*. Results are in [results/claude/README.md](results/claude/README.md), and the deliverable is [docs/memory_writing_guideline.md](docs/memory_writing_guideline.md).

## In progress: Qwen confirmatory run on Kaggle
- Kernel: `claude01012000/ema-confirmatory-qwen3-4b` (private). Logs dataset: `claude01012000/ema-confirmatory-logs` (private).
- Progress: **1,607/2,060 unique finished samples after session 6** (kernel version 8, 22 Nov ~16:20–20:52 IST, `DEADLINE_HOURS = 4.5` to fit what was left of this week's quota). Session 6 added only 91 (~20/hour, against ~36/hour in session 5 and ~17/hour in session 4), so the rate varies by session. The logs dataset was versioned "after session 6 (1607/2060)"; local recounts of both the kernel output and the uploaded folder give 1,607. The remaining 453 need ~13–23 GPU-hours: 2 sessions after the reset, possibly a short third.
- **Next: session 7 after the quota reset, Sat 28 Nov 05:30 IST.** `EMA-work/ckernel` is back to `DEADLINE_HOURS = 11.0`, and its cell sources are identical to `notebooks/kaggle_confirmatory.ipynb`. Push it with handover step 4. About 1.7 GPU-hours were left this week, too few for a useful session.
- **Session 6 incident (22 Nov):** version 7 was pushed by mistake with the 11-hour limit, a minute before version 8. Both ran from ~16:20 IST, resuming from the same samples, at 2× quota burn, which would have used up the quota ~20:00 and killed both before saving. Nischal cancelled version 7 in the Kaggle web UI at ~17:30. The API's cancel call needs a session id that no API call returns. Per-version status works: `version_label="v<N>"` (`EMA-work/session_watch.py <label>`).
- **The quota API's `timeUsed` wraps at 24 h.** It read 23.93 h at 17:00 and 0.10 h at 17:05, which looked like a reset; the Kaggle quota page showed 26 h 46 m used at ~19:20. Trust the Kaggle quota page (reset Sat 00:00 UTC = 05:30 IST).
- **Quota API:** `kagglesdk`'s `get_accelerator_quota_statistics` (`EMA-work/quota_watch.py`). `timeUsed` comes back in an odd format, e.g. `80980.375.0s`, and wraps at 24 h (see above).
- Disclosure: while reading session 4's kernel log for the sample count, one line of its per-condition summary was seen (`original` split, condition B: pass 0.15, flag 0.77). It is recorded in the README disclosure. Read only the `unique finished samples` line from now on.
- Logs from cancelled sessions are complete up to the interrupt. `eval_set` reports success even after a graceful interrupt, so judge progress by counting unique `(task, id, epoch)` samples. The notebook now does this.
- **Handover steps between sessions:**
  1. `kaggle kernels output claude01012000/ema-confirmatory-qwen3-4b -p <dir>`. Check `STATUS.txt` and count the samples (see `results/README.md` for how).
  2. Replace the dataset folder's contents (keep `dataset-metadata.json`) with `<dir>/logs/confirmatory_qwen3-4b/`. Inspect deletes superseded logs when it retries, so the kernel's log folder is the full state; don't merge. The folder needs `dataset-metadata.json` (id `claude01012000/ema-confirmatory-logs`). Copy `.eval-set-id` to `eval-set-id.txt`, because Kaggle drops dotfiles.
  3. Run `kaggle datasets version -p <folder> -m "after session N"`, using a **relative** path; Git Bash absolute paths get mangled.
  4. Run `kaggle kernels push -p <kernel dir> --accelerator NvidiaTeslaT4`. The kernel metadata must list the dataset in `dataset_sources`.
- Expected: 2 full sessions from 28 Nov (Sat 05:30–16:30 and straight after), possibly a short third. Complete around 29 Nov at ~36/hour, later at ~20/hour. Then run the H2 confirmatory replay (2–3 GPU-hours), which also needs that week's quota.

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
