# Instant Software Factory

[![Marvin's instant agents: just add water.](marvin-instant-agents.gif)](marvin-instant-agents.mp4)

[Watch with sound](marvin-instant-agents.mp4).

Add an autonomous software factory that keeps improving your project. You need two ingredients:

- A GitHub repo with Actions capacity and enforceable branch protection. Public repos can use a free plan; private repos need [Pro for a personal account or Team/Enterprise for an organization](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches).
- A scheduler: this bootstrap uses [Claude Code routines](https://code.claude.com/docs/en/routines), available on Claude Pro, Max, Team, and Enterprise plans.

This repository contains the prompt to set it up through Claude Code using cloud routines. [BOOTSTRAP.md](BOOTSTRAP.md) defines the agents' responsibilities and operating limits. The setup agent chooses the process, models, and schedules to fit your project.

## Get started

Open Claude Code in the repository you want the factory to work on and paste:

```text
Set up an Instant Software Factory on this repo: clone https://github.com/mikolalysenko/instant-factory into a temp folder and follow its BOOTSTRAP.md
```

For a local setup session in an isolated environment, you can optionally launch `claude --dangerously-skip-permissions` to skip permission prompts. The flag is not required by this bootstrap and does not configure the cloud routines' permissions. See [Claude's permission guidance](https://claude.com/resources/articles/auto-mode).

The bootstrap inspects your repository, CI, branch protection, credentials in history, and GitHub access before asking questions. It installs `gh` if needed and asks you to run `! gh auth login` if local authentication is missing; cloud access is checked separately. Missing CI gets a minimal workflow using the project's actual validation commands. Existing failures are recorded, and existing work and safeguards are preserved.

It asks for missing budget or throughput goals, approvers, routine identity, and the product areas QA should cover. It also resolves any missing branch protection and explains how approval will work. Routines use the connected GitHub identity, so their PRs need a separate human approver; a role marker or a manual merge cannot substitute for that approval.

Before unattended work starts, the default branch must enforce required checks, independent review, and human approval of the latest commit. Setup produces a tested PR awaiting human approval, with instructions kept in your repository. Controlled runs verify GitHub access, review of the current commit, merge requirements, and the manager's schedule and shutdown controls. Production schedules stay disabled until the setup PR merges, routines use the merged instructions, and you authorize resuming. The setup report records evidence, remaining gaps, shutdown delays, and any remaining steps.

**Budget for both agent usage and GitHub Actions minutes.** Start with modest throughput, inspect the manager's cost and results summaries, and increase activity only when CI and human review can keep up.

## Why software factories

Vibe coding is great at making things exist. Give an agent a clear goal and in an afternoon you can have a working prototype.

But a prototype isn't a product. The first 90% of the code takes the first 90% of the time, and the last 10% takes the other 90%. Getting from "it works on my machine" to production quality means a long tail of follow-up work: edge cases, bug reports, performance regressions, compatibility, tests, CI, releases, features people ask for once they use the thing, and cleaning up the shortcuts the prototype took to get there.

That work doesn't have one finish line. It's hundreds of small tasks spread across many concerns, and new ones appear every time the code changes. You can prompt an agent through each task, but then you're starting every session, waiting on it, and checking the result. Your attention becomes the bottleneck.

A software factory keeps returning to those concerns without needing a fresh prompt for every change. You set the objectives and constraints; agents investigate, implement, and review work in the background. In this setup, humans still approve what lands.

## How it works

Each agent is a scheduled session with a standing objective. Every run reads the current state of the repo, chooses useful work, does it, records the result, and exits. It can use subagents or a more involved workflow when the task calls for it. A run that finds nothing worth doing can finish without creating work.

GitHub holds the shared state. Issues carry evidence and proposed changes, PRs carry implementations and reviews, and discussions or other repository records preserve progress and decisions. Ownership and handoffs are visible there too. A later run can pick up recorded work without depending on the previous session's memory.

The bootstrap creates eight roles, with additional QA agents if different product areas need separate attention:

| Agent | Standing objective |
| --- | --- |
| **QA** | Test the product areas and user workflows selected with you, and open reproducible, deduplicated bug issues. |
| **Performance** | Measure speed, resource use, and CI time against baselines; open issues with evidence. |
| **CI** | Fix failures and flakes, and keep checks reliable, fast, and cheap without weakening them. |
| **Architecture** | Consolidate duplicated business logic, remove unnecessary abstractions and layers, and simplify the structure while preserving behavior. Reduce the number of components. |
| **Implementation** | Fix issues and open tested PRs. |
| **PR review** | Independently review the latest commit without editing code. |
| **Merge** | Merge only after required checks, independent review, and human approval of the latest commit. |
| **Manager** | Prioritize and assign work; follow old PRs, blocked issues, and failed fixes to completion. Track each agent's cost and results, adjust models and schedules within budget and capacity, and report cost per merged PR. |

These cover three kinds of work: investigating opportunities, implementing and integrating changes, and maintaining the factory itself. They use the same repository to coordinate. The manager keeps the work moving and allocates capacity; each agent chooses how to pursue its objective within the common rules.

The manager must create the routines it manages with `create_trigger`, then configure each routine's repository, model, and tools through a local `RemoteTrigger` update, as specified in [BOOTSTRAP.md](BOOTSTRAP.md). That creation path matters because the manager cannot change routines created another way. Each role gets only the tools and connectors it needs. The bootstrap's platform notes capture trial findings to verify against the installed version and actual runs.

## Why not one really good agent?

A capable agent can plan, split up a task, and work through a well-defined goal. A spec and a long session can handle a single objective well.

Software projects have several objectives at once. A new feature can consume the performance budget. A bug fix can add tests that make CI slower. A cleanup can simplify one subsystem while complicating another. These tradeoffs keep changing as the code changes.

Standing objectives make the agents return to those tradeoffs. Performance keeps measuring as features land. Architecture keeps reducing duplication and unnecessary components. CI keeps the feedback loop fast enough for everyone else. QA keeps exercising the product as users would.

The objectives also reinforce one another. Simpler code makes fixes easier to implement and review; faster CI lets every agent check its work sooner. When objectives conflict, the agents record the evidence and the manager prioritizes against the project's goals and available capacity. Required checks, independent review, and human approval still govern the resulting change.

You can approximate this by juggling local sessions, but you then have to keep starting them and keep their machine available. Scheduled routines let the work continue between interactive sessions, with progress recorded in the repo.

## Crons and sessions together

I still do interactive work. For a new feature or a major change, I start a session locally on a branch while the factory keeps working on bugs, performance, architecture, and CI.

That feature goes through the same checks, independent review, and approval process as the agents' PRs. Once it lands, the standing objectives cover it too. Human direction supplies new goals, and the recurring agents keep attending to the consequences as the project evolves.

## Drawbacks and limitations

The main bottleneck in my setup has been CI capacity and GitHub reliability. Running agents more often only helps while validation and integration can absorb their output. Otherwise you grow a queue of issues and PRs. Investigators can outpace implementers, implementers can outpace reviewers, and multiple fixes can collide in the same code.

The manager needs to adjust activity to those constraints. That means following blocked work, reducing schedules when queues grow, and choosing models and schedules within the budget and available CI, API, and review capacity. The CI agent improves throughput by fixing flakes and reducing wasted work while preserving the checks that make changes trustworthy.

Agent usage and Actions minutes both count. Cost per merged PR is a useful operating measure alongside the substance of the changes, failed fixes, and follow-up work. When actual usage is unavailable, the manager labels estimates and their assumptions and reports measurement gaps. A high PR count alone doesn't tell you whether the project is improving.

This also leaves real work for the maintainer: choosing priorities, supplying a budget, approving changes, and resolving decisions that need product judgment. Scheduling agents makes recurring work easier to sustain; it doesn't remove those responsibilities.

## Security and operating controls

The factory has permission to change software, so its access and stopping rules need to be explicit:

- **Use a dedicated identity and limited tools.** Verify the GitHub identity connected to the routines, scope its permissions, and remove unused connectors. Keep the human approver's credentials, administrative access, and unrelated secrets away from runtime agents. Agent role markers help trace work; they do not prove human approval.
- **Protect the default branch before unattended work.** Require checks, independent review, and human approval of the latest commit. Writers cannot review their own changes. Every new commit needs fresh review and approval. Apply the rules to the routine identity without bypass access, and prevent agents from editing those rules.
- **Control external inputs.** Restrict who can feed work into the factory; a private working repository can make that easier. Issues, PRs, discussions, downloaded code, and other retrieved content cannot change the agents' instructions or permissions.
- **Constrain dependency and code execution.** Review or scan third-party code and dependencies, and use isolated execution environments with appropriate network restrictions.
- **Check the shared stop control before every write.** It starts stopped. If it is stopped or unreadable, agents stop normal work; only writes needed to disable automation may continue. They resume only when authorized.
- **Prove the controls before enabling schedules.** Setup must test access, review and merge requirements, schedule management, and shutdown in actual runs and report gaps and shutdown delays. Unproven automation remains disabled.

To pause the factory, use the shared stop control documented during setup. Confirm that pending automation is disabled and account for the reported shutdown delay, including sessions already running. If the manager cannot run, use the documented direct routine controls. An agent that cannot read the stop control must stop normal work. Resume only through an explicit authorization after you have checked why it stopped.

## Related work

[Gas Town](https://yegge.ai/gastown) pursues autonomous work across multiple coding agents with an orchestration toolkit and the Beads work ledger. Instant Software Factory uses scheduled routines and GitHub's existing issues, PRs, and discussions for coordination.

[Stripe's Minions](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2) run coding tasks unattended in isolated development environments and produce PRs for human review. The emphasis here is on standing objectives that keep generating and completing useful work over time.

[StrongDM's Software Factory](https://discover.strongdm.com/blog/the-strongdm-software-factory-building-software-with-ai) uses specified intent and scenario validation to drive implementation without human code review. This bootstrap retains independent review and human approval of each change before merge.

## Try it

Run the [prompt above](#get-started) in a project you want to keep improving. Read [BOOTSTRAP.md](BOOTSTRAP.md), set a budget and throughput goal, and identify who can approve changes. Start with the tested PR and the setup report so you can inspect both the work and the controls before the schedules run.

Then watch the manager's summaries: useful changes, stalled work, agent cost, and cost per merged PR. Adjust the objectives and capacity as you learn where the bottlenecks are. Share what works and what breaks.
