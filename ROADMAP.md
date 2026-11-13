# Learning to drive in MetaDrive

## COGS 188 team roadmap

**Status:** Draft for team review, November 13, 2026. Tasks below are planned, not completed.

**Team:** Zahir Ali, Yasir Rizvi, Muhammad Omer.

**Working deadline:** Wednesday, December 2, 2026, 11:59 p.m. Pacific. The existing roadmap records this as the Canvas final-project deadline, while the syllabus places the report PDF in finals week. Confirm which deliverables are due when; until clarified, plan to have the full submission ready on December 2.

## 1. Goal and scope

Train a driving policy from randomly initialized weights in MetaDrive and test this question:

> Does training on varied road layouts and obstacle configurations improve safe completion on unseen scenarios compared with training on a small fixed set?

Use MetaDrive's vehicle-state, navigation, and LiDAR-like observations with continuous steering and acceleration/braking. Camera perception and real-vehicle deployment are outside scope. A successful simulation does not establish real-world driving safety.

The core deliverable is a fair PPO fixed-scenario versus varied-scenario comparison, with reproducible training, held-out evaluation, failure analysis, and a recorded local demonstration. TD3 and SAC are optional extensions, not dependencies for finishing the proposed project.

## 2. Decisions to confirm before long runs

- [ ] Confirm the Canvas deadline and report-PDF deadline with the course staff.
- [ ] Confirm the course concepts the report should connect to, and any rubric requirements.
- [ ] Accept or revise the proposed task split in Section 5; fill the open owner slots.
- [ ] Choose two weekly check-in times and a shared communication channel.
- [ ] Define the fixed training set, varied training set, validation set, and 100-scenario test set, including exact scenario IDs and generation settings.
- [ ] Set the episode horizon, termination rules, reward, observations, traffic settings, and safe-completion checks.
- [ ] Choose the short sweep budget and final per-run budget after profiling.
- [ ] Confirm three independent training seeds per core condition are feasible.
- [ ] Agree on checkpoint selection and the policy for failed or interrupted jobs before final training.

## 3. Experiments and baselines

### Required: scenario-diversity comparison

| Condition | Algorithm | Training scenarios | Target runs | Target budget per run |
| --- | --- | --- | --- | --- |
| Fixed-set baseline | PPO | Small, fixed road/obstacle set | 3 independent seeds | Approximately 1 million environment transitions |
| Varied-set treatment | PPO | Larger, varied road/obstacle set | 3 independent seeds | Same budget as fixed-set baseline |

The proposal treats budgets and repeated-run targets as provisional until the pilot. If resources require a smaller design, decide and record it before final testing. Training should exceed the proposal's 10,000-transition minimum; record actual counts rather than claiming the target was reached.

Hold the network, observations, reward, traffic settings, training-step budget, and PPO settings constant. Scenario diversity should be the intended difference. Document training-set sizes and what varies: road layouts, obstacle placement, or both.

### Required: validation-only learning-rate sweep

Compare PPO learning rates `1e-4`, `3e-4`, and `1e-3` using equal short budgets. Define a common sweep scenario distribution and seed policy before launching. Select the learning rate by validation safe-completion rate, with lower collision rate as the tie-breaker, as proposed. Freeze the selected PPO settings for both core conditions.

Do not choose hyperparameters or checkpoints using the test set. Record each sweep configuration, budget, validation result, and the selection decision.

### Suggested: non-learning reference

If supported by the chosen environment, add a scripted driving policy such as an IDM-based agent, or a simple constant-action policy. Verify compatibility first and describe exactly what it does. Run it on the same test scenarios and metrics. This is a reference floor, not a substitute for the fixed-set PPO baseline.

### Optional: algorithm comparison

Only after the core PPO experiment is on track, compare TD3 with PPO on the varied training set at the same environment-step budget. Add SAC only if time and compute remain. Target three seeds for any extension used to make a comparative claim.

Keep this separate from the scenario-diversity result. A matched step budget measures performance at that sample budget; it does not make runtime or tuning effort equal. Record algorithm settings and wall-clock costs, and label under-tuned or incomplete extensions honestly.

## 4. Compute and setup plan

Use UCSD DSMLP's batch interface as the primary compute route, following the instructor's guidance recorded in the existing roadmap. Do not depend on getting a dedicated A100, V100, or H100. Confirm account access, available resources, GPU selection, and concurrent-job limits before sizing the experiment.

### Setup gate: November 13-16

1. Verify SSH access and launch a small batch job using the [DSMLP batch-job instructions](https://support.ucsd.edu/services?id=kb_article_view&sysparm_article=KB0032269).
2. Pin a working Python, MetaDrive, Stable Baselines3, and PyTorch environment. Record versions and the job image/environment definition.
3. Inspect observation dimensions, finite values, action bounds, reward components, and termination/truncation behavior. Save the effective environment configuration.
4. Run a **50,000-step PPO smoke test**. Save a checkpoint and logs to persistent storage, then load that checkpoint in a fresh process and run evaluation episodes.
5. Measure environment steps per second, CPU/RAM use, and GPU use. Start with a small environment count; compare larger counts only within the job's resource allocation. Use measured throughput to estimate total runtime and storage.
6. Test headless cluster execution and local playback on the proposed M2 Pro Mac with 16 GB RAM. Record a short clip before committing to the demo workflow.
7. Create and freeze the train/validation/test manifests. Keep test results unopened until final evaluation.

Queue independent sweep jobs in parallel only when the allocation allows it. Do not assume six final training jobs can all run at once. Save periodic checkpoints and keep an experiment registry with config path, commit hash, seed, job ID, status, steps completed, and artifact paths.

If access or queues block progress for roughly a day, flag it to the team and course staff. Consider smaller pilots, a reduced pre-agreed budget, or another approved resource. Paid cloud compute is an option only after the team explicitly agrees on the payer, spending cap, and shutdown plan; this document authorizes no spending.

## 5. Proposed task split

The proposal names the responsibilities below, but they still need team confirmation. An open owner slot is not an assignment or commitment.

| Workstream | Proposal's suggested lead | Confirmed owner | Reviewer / backup | Ready when |
| --- | --- | --- | --- | --- |
| MetaDrive setup and scenario configuration | Zahir Ali | TBD | TBD | Environment checks pass; split manifests and config are saved |
| PPO training and cluster execution | Yasir Rizvi | TBD | TBD | Smoke test reloads; sweep and final jobs are reproducible |
| Evaluation, plots, experiment tracking | Muhammad Omer | TBD | TBD | Evaluator produces episode-level metrics and seed summaries |
| Local recording and demo | Zahir Ali | TBD | TBD | Saved policy plays locally; representative clips are recorded |
| Optional TD3/SAC extension | Not assigned | TBD | TBD | Core work is secure; separate matched-budget results exist |
| Report assembly and final submission | All members contribute | TBD | All members | Report, code, artifacts, and submission receipt are checked |

Meet twice weekly after agreeing on times. Flag blockers within 24 hours, record decisions in the repo, and adjust ownership explicitly. Use a small pilot to settle technical disagreements where practical; otherwise use the proposal's majority-decision approach. Every member reviews the final methods and results.

## 6. Schedule to December 2

These are planning windows, not claims that work has already started. Keep background jobs monitored during exams and the holiday period; agree on coverage rather than relying on one person.

| Window (2026) | Work | Exit gate |
| --- | --- | --- |
| Fri Nov 13 - Mon Nov 16 | Setup, batch access, 50k-step smoke test, local playback, throughput profiling, roles and split definitions | Checkpoint reload works; test manifest is frozen; feasible budget is recorded |
| Tue Nov 17 - Fri Nov 20 | Equal-budget learning-rate sweep on validation scenarios; finalize evaluator; write methods outline | Learning rate, all core settings, checkpoint rule, and run matrix are frozen |
| Sat Nov 21 - Wed Nov 25 | Train PPO fixed and varied conditions across independent seeds; monitor jobs and preserve artifacts | Core runs finish, or a documented scope adjustment is agreed before testing |
| Thu Nov 26 - Sun Nov 29 | Evaluate frozen policies on the same 100 unseen scenarios; produce plots, failure analysis, and local demo | Locked results table and reproducible episode records are available |
| Mon Nov 30 - Wed Dec 2 | Finish report, README, reproducibility checks, artifact packaging, and team review | Submit by Dec 2 at 11:59 p.m. Pacific; save confirmation |

Start writing the methods and report outline before training finishes. Do not treat a possible finals-week report deadline as extra time unless course staff confirm it.

## 7. Evaluation protocol

### Freeze before testing

- Save disjoint training, validation, and test scenario manifests. Verify scenario identities and generation settings do not overlap unintentionally.
- Use the same **100 unseen test scenarios** for every final policy. Record traffic/randomness seeds so policy comparisons use matched conditions where possible.
- Define episode horizon, success conditions, collision/departure flags, and evaluation action selection. Use deterministic evaluation where supported, or document and control any stochasticity.
- Select checkpoints using a pre-agreed rule based on training/validation only. Freeze the rule and final model list before reading test results.
- Do not discard low-performing but valid training seeds. Retain failed-job records; distinguish infrastructure failures from genuine learning failures.

### Metrics

For `E` attempted episodes:

| Metric | Definition |
| --- | --- |
| Safe completion rate | `100 * S / E`, where `S` counts arrival before timeout without collision or road departure |
| Collision rate | `100 * C / E`, where `C` counts episodes with a collision |
| Road-departure rate | `100 * D / E`, where `D` counts episodes leaving the permitted road |
| Mean route progress | Mean fraction of the route completed across attempted episodes |
| Mean episode return (secondary) | Mean total reward, interpreted alongside safety and progress metrics |

Episodes end on arrival, collision, road departure, or timeout. Stopping indefinitely is not success. Log simulator flags and define how simultaneous terminal events are counted; metric rates need not sum to 100%.

### Reporting

- Save one row per episode: run ID, training seed, scenario ID, evaluation seed, steps, arrival, collision, departure, timeout, progress, and return.
- Compute each metric per trained policy, then report **mean +/- standard deviation across independent training seeds**. State the actual number of seeds and scenarios. Do not present 300 episodes as 300 independent training runs.
- Plot learning curves against environment steps, marking whether each curve uses training or validation data. Compare final held-out metrics by condition.
- Include representative successes and failures, with a small failure taxonomy grounded in observed episodes. Video is illustration, not the aggregate result.
- Report null or negative results without changing the hypothesis afterward. If a bug requires retesting, document the fix, affected runs, and any test-set exposure.

## 8. Proposed repository structure

This is a target layout, not a claim about files currently in the repo. Keep the existing proposal notebook at its present path unless the team agrees to move it and update links.

```text
COGS188_Final_Project/
  README.md
  ROADMAP.md
  COGS188_Proposal.ipynb
  requirements.txt              # Or another pinned environment definition
  configs/
    env/                       # Observation, reward, traffic, termination settings
    algorithms/                # PPO and any optional TD3/SAC settings
    experiments/               # Sweep and final-run definitions
  scenarios/
    train_fixed.json
    train_varied.json
    validation.json
    test.json                  # Frozen IDs/configs, not tuning feedback
  scripts/
    train.py
    eval.py
    record.py
    plot_results.py
  jobs/                        # DSMLP launch scripts and resource requests
  results/
    run_registry.csv
    episodes/
    summaries/
    plots/
  checkpoints/                 # Small manifests/pointers; large binaries elsewhere
  report/
    figures/
  demos/                       # Short clips or links with scenario/model IDs
```

Pin dependency versions and include exact tested training, evaluation, and recording commands in the README once the scripts exist. Record config paths, seeds, commit hashes, and artifact locations. Do not publish credentials or machine-specific secrets. Use an agreed artifact store or Git LFS if appropriate for large checkpoints/videos, and verify reviewer access before submission.

## 9. Risks and fallback rules

| Risk | Response |
| --- | --- |
| DSMLP access or queue delays | Escalate early; use approved resources or agree on a smaller budget before final testing |
| Simulation throughput is too low | Profile first, tune environment concurrency within limits, and drop optional algorithms before weakening the core comparison |
| Learning curves are flat | Check environment/reward/action handling during the pilot; do not silently restart final runs until a favorable seed appears |
| Checkpoints or logs disappear | Test persistent storage and fresh-process reload during the smoke test; keep run manifests and backups |
| Local demo compatibility fails | Test playback early; preserve a working environment and agree on another recording machine if needed |
| Test leakage | Separate validation from test; freeze model selection; document any accidental exposure |
| Exams, holidays, or uneven workload | Confirm monitoring coverage and backup owners; keep check-ins short and decisions in the repo |
| Deadline ambiguity | Work to Dec 2 and ask course staff to clarify report timing |
| Scope expansion | Finish PPO fixed vs varied first; TD3, SAC, and presentation polish come afterward |

## 10. Definition of done

- [ ] All core runs and any missing/failed runs are accounted for, with actual budgets and seeds recorded.
- [ ] Frozen test evaluation, episode data, summaries, learning curves, and failure examples are saved.
- [ ] Report explains the question, methods, validation selection, results, limitations, and course connections.
- [ ] README includes tested setup and reproduction commands plus artifact locations.
- [ ] Checkpoints can be loaded and the recorded demo is accessible to reviewers.
- [ ] Software and research references are credited; relevant licenses are respected.
- [ ] Every teammate has reviewed the final package.
- [ ] Required course files are submitted and confirmation is saved.

## References and provenance

- [Project repository](https://github.com/zahiraIi/COGS188_Final_Project).
- [Original project proposal](https://github.com/zahiraIi/COGS188_Final_Project/blob/main/COGS188_Proposal.ipynb): research question, PPO comparison, learning rates, metrics, team responsibilities, and provisional run targets.
- [UCSD DSMLP batch-job instructions](https://support.ucsd.edu/services?id=kb_article_view&sysparm_article=KB0032269): primary compute guidance linked by the existing roadmap.
- [MetaDrive documentation](https://metadrive-simulator.readthedocs.io/).
- [Stable Baselines3 documentation](https://stable-baselines3.readthedocs.io/).

Adapted from the November 13 project roadmap for team review. Optional extensions, repository layout, task-owner slots, and exit gates are proposed additions. The working deadline and compute guidance come from that roadmap and still need the team checks listed above. This file makes no claim that training, evaluation, or submission has been completed.
