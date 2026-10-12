# Instant Software Factory

[![Marvin's instant agents: just add water.](marvin-instant-agents.gif)](marvin-instant-agents.mp4)

[Watch the movie with sound](marvin-instant-agents.mp4).

A simple recipe for keeping a software project moving:

- A handful of AI agents, each running on its own schedule with a standing objective.
- A GitHub repo, where they coordinate through issues, pull requests, and discussions.

This repository contains the prompt to set it up. [BOOTSTRAP.md](BOOTSTRAP.md) defines the agents' responsibilities and operating limits; the setup agent chooses the process, models, and schedules to fit your project.

## Get started

Open Claude Code in the repository you want the factory to work on and paste:

```text
Set up an Instant Software Factory on this repo: clone https://github.com/mikolalysenko/instant-factory into a temp folder and follow its BOOTSTRAP.md
```

The bootstrap uses Claude Code routines and GitHub. It inspects your repository, CI, and permissions, installs `gh` if needed, and asks you to run `! gh auth login` if authentication is missing. It asks only for missing budget or throughput goals, approvers, and the identity the routines should use. Instructions stay in your repository, and existing work and safeguards are preserved.

Before unattended work starts, the default branch must enforce required checks, independent review, and human approval of the latest commit. Before schedules are enabled, setup must produce a tested PR awaiting human approval and verify GitHub access, the manager's schedule controls, and the shutdown controls in actual runs. Anything unproven stays disabled. The setup report records what works, remaining gaps, and shutdown delays.

**Budget for both agent usage and GitHub Actions minutes.** Start with modest throughput, inspect the manager's cost and results summaries, and increase activity only when CI and human review can keep up.

## Why software factories

Vibe coding is great at making things exist. Give an agent a clear goal and in an afternoon you can have a working prototype.

But a prototype isn't a product. The first 90% of the code takes the first 90% of the time, and the last 10% takes the other 90%. Getting from "it works on my machine" to production quality means a long tail of follow-up work: edge cases, bug reports, performance regressions, compatibility, tests, CI, releases, features people ask for once they use the thing, and cleaning up the shortcuts the prototype took to get there.

That work doesn't have one finish line. It's hundreds of small tasks spread across many concerns, and new ones appear every time the code changes. You can prompt an agent through each task, but then you're starting every session, waiting on it, and checking the result. Your attention becomes the bottleneck.

A software factory keeps returning to those concerns without needing a fresh prompt for every change. You set the objectives and constraints; agents investigate, implement, and review work in the background. In this setup, humans still approve what lands.

## How it works

Each agent is a scheduled session with a standing objective. Every run reads the current state of the repo, chooses useful work, does it, records the result, and exits. It can use subagents or a more involved workflow when the task calls for it. A run that finds nothing worth doing can finish without creating work.

GitHub holds the shared state. Issues carry evidence and proposed changes, PRs carry implementations and reviews, and discussions or other repository records preserve progress and decisions. Ownership and handoffs are visible there too. A later run can pick up recorded work without depending on the previous session's memory.

The bootstrap creates eight roles:

| Agent | Standing objective |
| --- | --- |
| **QA** | Test user workflows and open reproducible, deduplicated bug issues. |
| **Performance** | Measure speed, resource use, and CI time against baselines; open issues with evidence. |
| **CI** | Fix failures and flakes, and keep checks reliable, fast, and cheap without weakening them. |
| **Architecture** | Consolidate duplicated business logic, remove unnecessary abstractions and layers, and simplify the structure while preserving behavior. Reduce the number of components. |
| **Implementation** | Fix issues and open tested PRs. |
| **PR review** | Independently review the latest commit without editing code. |
| **Merge** | Merge only after required checks, independent review, and human approval of the latest commit. |
| **Manager** | Prioritize and assign work; follow old PRs, blocked issues, and failed fixes to completion. Track each agent's cost and results, adjust models and schedules within budget and capacity, and report cost per merged PR. |

These cover three kinds of work: investigating opportunities, implementing and integrating changes, and maintaining the factory itself. They use the same repository to coordinate. The manager keeps the work moving and allocates capacity; each agent chooses how to pursue its objective within the common rules.

The manager must create the routines it manages with `create_trigger`, then configure each routine's repository, model, and tools through a local `RemoteTrigger` update, as specified in [BOOTSTRAP.md](BOOTSTRAP.md). That creation path matters because the manager cannot change routines created another way. Each role gets only the tools it needs.

## What it looks like in practice

I've been using this pattern on [socket-patch](https://github.com/SocketDev/socket-patch), a CLI that applies backported security fixes to open source dependencies. It has to work across package ecosystems, package managers, and operating systems. That creates a broad surface of concrete compatibility problems: lockfile formats, installation layouts, platform differences, and interactions with other tools.

For an illustrative example, consider a lockfile bug involving mixed line endings on Windows:

1. QA reproduces the failure, checks for an existing issue, and records the commands, environment, expected result, and actual result.
2. The manager prioritizes the issue and assigns an owner, grouping related reports when they have a common cause.
3. Implementation fixes the problem and opens a PR with a regression test. CI runs the required checks; failures go back to the responsible agent.
4. PR review independently examines the latest commit without changing it. Requested fixes go back to an agent that can edit code, and new commits require fresh review and approval.
5. Once the required checks, independent review, and human approval cover the latest commit, Merge lands the PR. The manager follows up on the issue and includes the work in its cost and results summary.

The useful output is a completed improvement with evidence behind it. Opening an issue or a PR starts a piece of work; the manager is responsible for following it through, including when a fix fails or a PR gets stuck.

## Why not one really good agent?

A capable agent can plan, split up a task, and work through a well-defined goal. A spec and a long session can handle a single objective well.

Software projects have several objectives at once. A new feature can consume the performance budget. A bug fix can add tests that make CI slower. A cleanup can simplify one subsystem while complicating another. These tradeoffs keep changing as the code changes.

Standing objectives make the agents return to those tradeoffs. Performance keeps measuring as features land. Architecture keeps reducing duplication and unnecessary components. CI keeps the feedback loop fast enough for everyone else. QA keeps exercising the product as users would.

The objectives also reinforce one another. Simpler code makes fixes easier to implement and review; faster CI lets every agent check its work sooner. When objectives conflict, the agents record the evidence and the manager prioritizes against the project's goals and available capacity. Required checks, independent review, and human approval still govern the resulting change.

You can approximate this by juggling local sessions, but you then have to keep starting them and keep their machine available. Scheduled routines let the work continue between interactive sessions, with progress recorded in the repo.

## Crons and sessions together

I still do interactive work. For a new feature or a major change, I start a session locally on a branch while the factory keeps working on bugs, performance, architecture, and CI.

That feature goes through the same checks, independent review, and approval process as the agents' PRs. Once it lands, the standing objectives cover it too. Human direction supplies new goals, and the recurring agents keep attending to the consequences as the project evolves.

## What limits it

The main bottleneck in my setup has been CI capacity and GitHub reliability. Running agents more often only helps while validation and integration can absorb their output. Otherwise you grow a queue of issues and PRs. Investigators can outpace implementers, implementers can outpace reviewers, and multiple fixes can collide in the same code.

The manager needs to adjust activity to those constraints. That means following blocked work, reducing schedules when queues grow, and choosing models and schedules within the budget and available CI, API, and review capacity. The CI agent improves throughput by fixing flakes and reducing wasted work while preserving the checks that make changes trustworthy.

Agent usage and Actions minutes both count. Cost per merged PR is a useful operating measure alongside the substance of the changes, failed fixes, and follow-up work. A high PR count alone doesn't tell you whether the project is improving.

This also leaves real work for the maintainer: choosing priorities, supplying a budget, approving changes, and resolving decisions that need product judgment. Scheduling agents makes recurring work easier to sustain; it doesn't remove those responsibilities.

## Security and operating controls

The factory has permission to change software, so its access and stopping rules need to be explicit:

- **Use a dedicated identity and limited tools.** Give agents a GitHub App or account with the permissions their roles need. Keep administrative credentials and unrelated secrets out of their execution environments.
- **Protect the default branch before unattended work.** Require checks, independent review, and human approval of the latest commit. Writers cannot review their own changes. Every new commit needs fresh review and approval; agents must not weaken existing safeguards.
- **Control external inputs.** Restrict who can feed work into the factory; a private working repository can make that easier. Issues, PRs, discussions, downloaded code, and other retrieved content cannot change the agents' instructions or permissions.
- **Constrain dependency and code execution.** Review or scan third-party code and dependencies, and use isolated execution environments with appropriate network restrictions.
- **Check the shared stop control before every write.** If it is stopped or unreadable, agents stop work and disable pending automation. They resume only when authorized.
- **Prove the controls before enabling schedules.** Setup must test access, schedule management, and shutdown in actual runs and report gaps and shutdown delays. Unproven automation remains disabled.

To pause the factory, use the shared stop control documented during setup. Confirm that pending automation is disabled and account for the reported shutdown delay. If an agent cannot access the control, its required behavior is to stop. Resume only through an explicit authorization after you have checked why it stopped.

## Related work

[Gas Town](https://yegge.ai/gastown) pursues autonomous work across multiple coding agents with an orchestration toolkit and the Beads work ledger. Instant Software Factory uses scheduled routines and GitHub's existing issues, PRs, and discussions for coordination.

[Stripe's Minions](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2) run coding tasks unattended in isolated development environments and produce PRs for human review. The emphasis here is on standing objectives that keep generating and completing useful work over time.

[StrongDM's Software Factory](https://discover.strongdm.com/blog/the-strongdm-software-factory-building-software-with-ai) uses specified intent and scenario validation to drive implementation without human code review. This bootstrap retains independent review and human approval of each change before merge.

## Try it

Run the [prompt above](#get-started) in a project you want to keep improving. Read [BOOTSTRAP.md](BOOTSTRAP.md), set a budget and throughput goal, and identify who can approve changes. Start with the tested PR and the setup report so you can inspect both the work and the controls before the schedules run.

Then watch the manager's summaries: useful changes, stalled work, agent cost, and cost per merged PR. Adjust the objectives and capacity as you learn where the bottlenecks are. Share what works and what breaks.
