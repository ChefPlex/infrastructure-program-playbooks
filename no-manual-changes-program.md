# No Manual Changes in Production: Running the Program

A guide for moving an organization from changes made by hand to changes made only through an automated pipeline. The finish line is one sentence: **no person changes production directly, and every change that reaches it went through a pipeline that recorded who asked, who approved, what was tested and what was deployed.**

Most organizations that start this already have pipelines. What is left is the last share of changes still made by hand, usually by people with a good reason for making them that way.

Written from running this kind of program across a large security engineering organization, over about a year.

---

## The misconception this program has to correct first

"No manual changes" gets planned as a tooling migration: build the pipeline, announce it, done. Most of the work is access and incentive.

As long as an engineer can log in and fix something in two minutes, the pipeline is the slower path, and people take the faster one. Lecturing them about discipline will not change that.

Think of a desire path across a lawn. People walk where the walk is shortest, whatever the signs say, so pave the path they use and then fence the lawn. In this program, paving means making the pipeline fast enough for real work, including emergencies, and fencing means removing human write access to production. Do them in that order. If you fence first, people climb the fence.

A working pipeline is the halfway point. The program is finished when the manual path is closed.

---

## 1. Size the problem before choosing a method

Find out how big a problem you are up against before you plan anything. Two sources, and they will disagree.

**Ask.** Survey every team that deploys to production. The questions that earn their place:

| Area | Questions |
|:---|:---|
| **Process** | Is there a written change policy, and where is it? Which environments does it cover? How many changes did the team make last quarter, by type: standard, pre-approved, significant, emergency? What review happens before a change: peer review, manager approval, a CAB (change advisory board)? |
| **People** | How many engineers can push to production? Which roles exist in the change process: requester, reviewer, approver, tester, implementer? Is the person who writes the code ever the same person who deploys it? |
| **Tools** | Which tools handle change requests, source control, CI (continuous integration), CD (continuous delivery), infrastructure provisioning, asset inventory and testing? |
| **Pain** | What would the team change about its own process if it could? |

**Measure.** Then pull the real numbers from deployment logs, cloud audit trails and access logs: how many production changes in the last 90 days came through a pipeline, and how many did not. The survey tells you what people believe happens. The logs tell you what happened. The gap between the two is your first finding, and it is usually the most useful one.

## 2. Define "manual change" before you count anything

Agree the definition before the baseline, or the numbers will move every time someone argues about a borderline case.

| Counts as manual | Does not count as manual |
|:---|:---|
| Logging in to a host and editing a file or restarting a service by hand | A pipeline run, with its approval recorded |
| Clicking through a cloud console to change a setting | A break-glass change that follows the break-glass rules in section 5 |
| A direct update to a production database | An automated job that the pipeline itself deployed |
| Flipping a feature flag or config value in an admin screen with no record | A flag change made through a tool that records who, what and why |
| Copying a hotfix onto a host | |

Write the definition into the charter. Decide the borderline cases once, in writing, and point to that decision when they come up again.

---

## 3. Running it, start to finish

The stages below line up with the PMI (Project Management Institute) process groups, named in parentheses. Use them as a guide, not a script.

### Start (PMI: Initiation)

**Inputs.** The reason the program exists: an audit finding, an outage caused by an unrecorded change, a compliance requirement such as SOX (Sarbanes-Oxley) or SOC 2 (System and Organization Controls 2) change-management controls, or a reliability goal. Add the existing change policy, the pipelines and standards already in place, and the constraints you cannot change.

**Develop the charter.** The charter states the finish line from the top of this guide, the definition from section 2, the baseline from section 1, the scope (which environments, which teams), and the sponsor who will decide when teams disagree. See the [Program Charter Template](https://github.com/ChefPlex/tpm-templates/blob/main/program-charter-template.md).

**Identify stakeholders.** Every team that deploys to production, the on-call rotations, security, compliance, internal audit, the change advisory board if there is one, and whoever owns the pipeline platform. Start the stakeholder register here; it fills in as teams come forward.

### Plan the waves (PMI: Planning)

**Collect requirements once the stakeholders are known.** The real requirements come from the teams that deploy: the job that only runs by hand, the emergency the pipeline cannot handle today. You cannot collect them until you know who to ask.

**Build a requirements traceability matrix.** Each requirement traces to the control it satisfies and the evidence that proves it. For example, "every production change traces to an approved request" traces to the change-management control and to the pipeline's deployment record. Auditors ask for exactly this mapping, and it is much easier to build at the start than to reconstruct at the end.

**Build the WBS and the wave plan.** Group teams into waves. Put a willing team with a simple estate in the first wave, and keep the hardest team out of it. The first wave proves the pattern; the hardest team goes once there is a working example to point to.

**Plan risk.** Identify the risks, rank them by likelihood and impact, then put numbers on the few that warrant it, then decide a response for each. The usual top three: the pipeline is too slow for an emergency; a critical job only one person knows how to run; a team agrees in the meeting and keeps its access afterwards. See the [RAID Log Guide](https://github.com/ChefPlex/tpm-templates/blob/main/raid-log-guide.md).

**Plan communications.** Each team hears about its wave at least four weeks before pre-work starts. See the [Communications Plan Template](https://github.com/ChefPlex/tpm-templates/blob/main/communications-plan-template.md).

### Run each wave (PMI: Execution)

Each team, or group of teams, moves in a wave with three steps.

**Pre-work, about four weeks.** Inventory the team's manual changes from the logs. Build or extend a pipeline for each one. Dry-run each pipeline against a non-production environment. Agree the break-glass procedure. Train the on-call engineers.

**Soft cutover.** The pipeline becomes the default. Manual access still exists, but every manual change is logged and reviewed the next business day: why was it manual, and what pipeline change would have made it unnecessary? This is where the requirements you missed show up.

**Hard cutover.** Human write access to production is removed. Break-glass remains for emergencies. Set the date for this step at the start of the wave, not when the team feels ready, or the step never arrives.

Stagger the waves one to two weeks apart so each wave's pre-work overlaps the next one's. A wave cuts over every week or two while the next ones are still in pre-work. The pipeline platform team is not swamped, and a problem in one wave shows up before the next one cuts over.

### Track it (PMI: Monitoring and Controlling)

Track these weekly, by team:

| Measure | What it tells you |
|:---|:---|
| Share of production changes made through a pipeline | Progress toward the finish line. Taken from logs, never from attestation. |
| Manual changes per week, with the reason for each | Where the pipeline still does not fit the work |
| Break-glass uses, and whether each was reviewed and followed up | Whether the emergency path is being used for emergencies |
| Change failure rate and time to deploy | Whether pipeline-only made delivery worse. If it did, the program will lose support. |

**Run one exception process.** Some changes cannot move by the hard-cutover date. Each exception is written down with an owner, a reason and an expiry date, and goes through the same approval as any other change to the plan. An exception with no expiry date is a permanent manual path.

**Validate scope against the logs.** A team is done when the logs show no manual changes for a set period, such as 30 days. A team saying it is done does not count.

### Hand it over (PMI: Closing)

Hand the controls to a named operations owner. Keep a detective control in place: an alert on any production change that did not come through a pipeline. Retire the old access, shared credentials, jump hosts and scripts once they have shown zero use. Run a lessons-learned session, and close any vendor contracts. See the [Close-Out Report](https://github.com/ChefPlex/tpm-templates/blob/main/program-close-out-report-template.md).

---

## 4. Make the pipeline the fastest path

If the pipeline is slower than the manual path, every deadline creates pressure to go around it.

- **An emergency lane.** A pipeline path for urgent fixes, with a fast approval by an on-call approver. If an urgent fix takes four hours through the pipeline, people will keep their access.
- **Pre-approved standard changes.** Routine, low-risk change types skip the advisory board and go straight through with automated checks.
- **Pipelines for the odd jobs.** The quarterly certificate rotation and the one-off data fix are where manual access survives longest. Build them a pipeline too.

## 5. Break-glass: the concession that keeps the program honest

There will be emergencies the pipeline cannot handle. If the plan does not allow for them, people build quiet workarounds.

- Break-glass access is personal (never a shared account), time-limited, and logged.
- Every use is reviewed within one business day.
- Every use ends with a pipeline change that makes the fix permanent. Otherwise production drifts away from what the pipeline thinks it deployed.

If break-glass use rises, look at the pipeline before you look at the engineers.

---

## Worked example: Wrenfield Software

*Wrenfield Software is fictional. Every number and name below is invented for illustration.*

Wrenfield has 14 teams deploying about 60 services. An internal audit found production changes with no approval record. The sponsor is the VP of Engineering.

**Sizing.** The survey said most teams deployed only through pipelines. The logs said 31% of production changes in the last 90 days were manual, and 70% of those came from four teams. Two of the four did not know their changes counted as manual: they used an admin console that left no record.

**Requirements.** Collecting requirements from the four teams turned up a nightly reconciliation job run by hand by one engineer, and a database fix procedure with no pipeline equivalent. Both went into the wave plan as pre-work items.

**Waves.**

| Wave | Teams | Pre-work starts | Hard cutover |
|:---|:---|:---|:---|
| 1 | 3 teams, simple estates | Week 1 | Week 6 |
| 2 | 4 teams | Week 3 | Week 8 |
| 3 | 4 teams, including two of the heavy four | Week 5 | Week 10 |
| 4 | 3 teams, including the other two | Week 7 | Week 12 |

**What went wrong.** In wave 3, one team's break-glass use tripled after hard cutover. Review showed the emergency lane needed an approver who was only available during business hours. Adding an on-call approver brought break-glass use back down within two weeks.

**Result.** By week 16, pipeline share was 99% across all 14 teams, with two open exceptions, each with an owner and an expiry date.

---

## Where this breaks

**When the pipeline is the bottleneck.** If deployments queue for hours, the program becomes the reason releases are late, and support for it disappears. Fix pipeline speed before the first hard cutover.

**When the sponsor will not back the hard cutover.** Every team will ask for one more month. Without someone who can say no, the soft cutover becomes permanent.

**When the measure comes from attestation.** Teams report themselves done. The logs are the only measure that holds up in front of an auditor.

**When break-glass is a shared account.** Then the log records that someone used it, not who, and the review has nothing to review.

**When "no manual changes" is read as "safe changes".** A pipeline deploys a bad change as faithfully as a good one. The program makes every change recorded and approved. Whether a change was safe still depends on testing, review and rollback.

---

## Related

- [Program Phases Playbook](https://github.com/ChefPlex/tpm-templates/blob/main/program-phases-playbook.md): the general lifecycle this guide applies to one kind of program
- [Program Charter Template](https://github.com/ChefPlex/tpm-templates/blob/main/program-charter-template.md)
- [RAID Log Guide](https://github.com/ChefPlex/tpm-templates/blob/main/raid-log-guide.md)
- [Stakeholder Engagement Playbook](https://github.com/ChefPlex/tpm-templates/blob/main/stakeholder-engagement-playbook.md): for the team that agrees in the meeting and keeps its access
- [Program Swim Lanes Template](https://github.com/ChefPlex/tpm-templates/blob/main/program-swim-lanes-template.md): defining each wave as a lane, with its owner and Definition of Done
- [Program Close-Out Report Template](https://github.com/ChefPlex/tpm-templates/blob/main/program-close-out-report-template.md)

---

*Part of the [ChefPlex TPM portfolio](https://github.com/ChefPlex/tpm-templates/blob/main/PORTFOLIO-GUIDE.md). Maintained by [Eric White](https://www.linkedin.com/in/edwhite). Licensed under [CC BY 4.0](LICENSE).*
