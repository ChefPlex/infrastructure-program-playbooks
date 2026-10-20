# End-of-Support Migration Playbook

**How to move a server and operating system estate off a version the vendor is retiring, against a
deadline you did not set: find out what you have, decide what happens to each system, and finish
before the date arrives.**

The vendor picks the date. You pick everything else. An end-of-support program looks like an upgrade
project from the outside, and the upgrades are the easy part. The hard parts are finding every system
that runs the old version, finding out who owns each one, finding out what depends on it, and getting
a decision made about each system early enough to act on it.

This playbook is for the Technical Program Manager (TPM) who has been handed that program. It assumes
an estate of hundreds of servers, more than one IT group, and a steering committee that wants a
number every month. These are patterns that generalize. The specifics of any one estate will not.

---

## The short version

- **Plan backward from the date.** The date does not move. Scope does.
- **The inventory will grow while you look at it.** Report it as a discovery curve, not as a series
  of corrections.
- **Triage before you remediate.** Every system goes to one of four outcomes: decommission, upgrade
  in place, replace or migrate, or pay for extended support.
- **Decommission is the cheapest outcome** and usually the biggest bucket nobody planned for.
- **Ownership is the long pole.** Finding who owns a server and what depends on it takes longer than
  upgrading it.
- **The cost of waiting is the business case.** Put the price of extended support beside the program
  budget, on the same page.
- **Nothing touches production without a test environment first**, a written method of procedure,
  and a change window.

---

## What "end of support" means, and what the standards expect

Vendors publish a lifecycle for each product version. The phases differ by vendor, but the shape is
usually the same: a period of full support, a period of security fixes only, and then nothing.

One worked illustration, using one vendor's published terms. Microsoft's
[Fixed Lifecycle Policy](https://learn.microsoft.com/en-us/lifecycle/policies/fixed) (as of October
2026) provides "A minimum of five years Mainstream Support", followed for some products by a period
of Extended Support. After that, security updates for some products continue only through a paid
Extended Security Update (ESU) program. Microsoft describes it plainly in its
[Windows Server ESU overview](https://learn.microsoft.com/en-us/windows-server/get-started/extended-security-updates-overview)
(as of October 2026): "The Extended Security Update (ESU) program is a last resort option for
customers who need to run certain legacy Microsoft products past the end of support." Its
[ESU FAQ](https://learn.microsoft.com/en-us/lifecycle/faq/extended-security-updates) (as of October
2026) adds: "They are not intended as a long-term solution, but rather as a temporary bridge to stay
secure while one migrates to a newer, supported platform."

Read that as the vendor telling you the extension has an end too. On the same overview page, the ESU
window for one recent server generation was listed as three years, and it ended on October 13, 2026. Your
program should treat any extension as a bridge with a published far bank, never as a place to stay.

Three public sources set the expectation your security and audit teams will hold you to:

| Source | What it says | Why it matters to the program |
|:---|:---|:---|
| [NIST SP 800-53 Rev. 5](https://doi.org/10.6028/NIST.SP.800-53r5), control SA-22 "Unsupported System Components" (National Institute of Standards and Technology, Special Publication; as of October 2026) | "Replace system components when support for the components is no longer available from the developer, vendor, or manufacturer", or provide alternative support | Replacement is the default. Anything you keep needs a named alternative source of support. |
| [NIST SP 800-40 Rev. 4](https://doi.org/10.6028/NIST.SP.800-40r4), Guide to Enterprise Patch Management Planning (April 2022; as of October 2026) | At end of life, "a patch for it will never be released because the software is at end-of-life", and "Unpatchable assets need to be included in risk response planning" | A system past its date is a standing risk, not a backlog item. It needs a recorded response. |
| [CISA Bad Practices](https://www.cisa.gov/stopransomware/bad-practices) (Cybersecurity and Infrastructure Security Agency; as of October 2026) | "Use of unsupported (or end-of-life) software in service of Critical Infrastructure and National Critical Functions is dangerous and significantly elevates risk to national security, national economic security, and national public health and safety." It adds: "This dangerous practice is especially egregious in technologies accessible from the Internet." | Written for critical infrastructure, and the logic travels: internet-facing systems on unsupported versions go first in the queue. |

SA-22 also says what "support" covers, which is useful when someone claims a system is "supported"
because a contractor still answers the phone: "Support for system components includes software
patches, firmware updates, replacement parts, and maintenance contracts."

---

## Step 1: Plan backward from the date you did not set

**Start from the end-of-support date and work backward.** Most projects plan forward from a start
date and land wherever the work lands. This one cannot, because the finish line is published and
fixed.

Work backward in this order:

1. **The date.** Look it up on the vendor's lifecycle page and write the exact date, time zone
   included, into the charter. Vendors publish these precisely and people misremember them.
2. **The freeze before it.** Most organizations freeze changes around quarter close, year end or a
   peak season. Subtract those windows. The real deadline is usually weeks earlier than the vendor's.
3. **The cut-over capacity.** Count how many change windows you have between now and the freeze, and
   how many systems can safely go in each one. That number is your ceiling.
4. **The decision deadline.** Every system needs a triage decision (Step 3) before it can be
   scheduled. Set a date by which every system must have one, and make it early.

⚠️ **The date does not move, so scope has to.** When the plan does not fit, the levers are fewer
systems (decommission more), more capacity (more engineers, more windows), or a paid extension for
the residual. Moving the date is not on the list, and saying so early saves a meeting later.

---

## Step 2: Discovery, and why the number keeps going up

**The inventory grows while you look at it.** The first count comes from whatever inventory already
exists. The second comes after someone runs a scan. The third comes after application teams start
answering questions and remember the server under a desk, the appliance with an embedded operating
system, and the virtual machine that was cloned for a test two years ago.

That is a discovery curve, and it is normal. Each count is more complete than the last. The mistake
is reporting each new number as if the previous one was wrong.

NIST says the same thing about inventories in general. From
[SP 800-40 Rev. 4](https://doi.org/10.6028/NIST.SP.800-40r4): "While a comprehensive inventory
of all assets is ideal, it may be impossible to achieve, given the highly dynamic nature of assets
and software." And: "Without constant updates, inventories will quickly become outdated and provide
increasingly inaccurate and incomplete information for patching efforts."

### Use more than one source, because none of them is complete

| Source | What it finds | What it misses |
|:---|:---|:---|
| **Vendor assessment toolkit** (a discovery scanner the vendor or a partner provides) | Everything reachable on the network with credentials | Systems on isolated segments, systems it has no credentials for, and who owns any of it |
| **Configuration management database (CMDB) and asset inventory** | What someone registered, with an owner field | Anything never registered, and owners who have left |
| **Virtualization and cloud consoles** | Every virtual machine, including powered-off ones | Physical servers and appliances |
| **Monitoring, backup and patching tools** | Systems someone cared about enough to monitor or back up | Systems nobody cared about, which are the ones you need |
| **Application teams** | What depends on what, and which systems are really retired | Anything they have forgotten |

**Reconcile them into one list with a source column.** A system found by the scanner and missing
from the CMDB is a finding in itself: it probably has no owner.

### Report the curve as a funnel

Report every system by the stage it is in, so growth at the top reads as discovery rather than
slippage:

| Stage | Meaning |
|:---|:---|
| **Identified** | Found, on the list, nothing else known yet |
| **Analysis underway** | Owner and dependencies being established |
| **Decommission planning** | Decided: it goes away |
| **Upgrade planning** | Decided: upgrade in place or replace, scheduled or being scheduled |
| **Complete** | Off the old version, verified |

**Unknown status line:**

> Server count revised from 400 to 520. Completion now 12 percent, down from 15 percent.

**Actionable status line:**

> Discovery added 120 systems this month (scanner run on two segments not previously covered).
> 520 identified. 140 in analysis, 90 decommission planning, 210 upgrade planning, 60 complete.
> We expect discovery to slow after the remaining two segments are scanned next month.

The first reads like the program is going backward. The second says the same facts and tells the
committee what to expect next.

---

## Step 3: Triage every system before anyone upgrades anything

**Every system gets one of four outcomes, and it gets one before work starts.** Upgrading the systems
you already know about is the tempting first move, and it spends your best engineers on the easy
cases while the decisions pile up.

| Outcome | When it fits | NIST SP 800-40 risk response |
|:---|:---|:---|
| **Decommission** | Nobody uses it, or its function has moved elsewhere | Avoid |
| **Upgrade in place** | The application supports the new version and the hardware can run it | Mitigate |
| **Replace or migrate** | New build on the new version, or move the workload to another platform or a service | Mitigate, or Transfer if a provider takes over patching |
| **Extended support** | It cannot move before the date, and the business accepts the cost | Accept, with compensating controls |

The risk-response column comes from [SP 800-40 Rev. 4](https://doi.org/10.6028/NIST.SP.800-40r4)
section 2.1, which lists accept, mitigate, transfer and avoid. Its description of avoid reads almost
like a decommission plan: "Ensure that the risk does not occur by eliminating the attack surface,
such as by uninstalling the vulnerable software, decommissioning assets with the vulnerabilities".

### Decommission is the biggest bucket nobody planned for

**Expect a large share of the estate to need nothing more than switching off.** Old estates collect
servers that outlived their purpose: a project that ended, an application that moved, a test box
nobody remembered. Nobody budgets for them, because nobody knew they were there.

Decommission is the cheapest outcome by a wide margin. It needs an owner to agree, a period where the
server is shut down but not deleted (so anyone who screams can be heard), and a backup kept for the
retention period. **Run a decommission track from week one**, separate from the upgrade track, so it
does not wait behind the hard cases.

⚠️ **Shut down, wait, then delete.** A server that is powered off for a few weeks and generates no
complaints is safe to retire. A server deleted on the strength of "I think nobody uses it" is how a
quarterly batch job fails in a way nobody can explain.

---

## Step 4: Ownership is the long pole

**Finding who owns a server takes longer than upgrading it.** An upgrade is a few hours of engineering
in a change window. Finding the owner can take weeks: the CMDB names someone who left, their manager
says the team was reorganized, the new team says it was never theirs.

Treat ownership discovery as its own workstream:

- **Assign an owner to every system, by name.** A team name is not an owner. "Unknown" is a status,
  and it should be visible in the funnel.
- **Set a deadline for claiming.** Publish the list of unowned systems with a date. Anything still
  unclaimed after the date goes to the decommission track with a shutdown notice. That turns silence
  into a decision.
- **Escalate unowned systems that are clearly in use.** If the server has traffic and nobody claims
  it, that is a management problem, and the steering committee is the place to solve it.
- **Watch the boundary between groups.** When two IT organizations share an estate, systems on the
  boundary get claimed by neither. Agree in writing which group decides for which systems.

⚠️ **The unowned systems are often the oldest and most central.** They are the hardest to find an
owner for and the riskiest to touch. Plan for them to be the last ones finished, and say so.

---

## Step 5: Map the dependencies, including the ones that call in

**A server's dependencies run in both directions.** What it calls is usually documented. What calls
it usually is not.

Questions to answer for each system before it is scheduled:

- **What runs on it?** Applications, services, scheduled tasks, scripts.
- **What does it call?** Databases, file shares, other servers, external services.
- **What calls it?** Applications that connect to it by name or address, users who map a drive to
  it, jobs on other servers that copy files to it.
- **What authenticates against it?** If it is a directory server, a certificate server or anything
  that issues or checks credentials, every application that logs in through it is a dependency. See
  the [Directory Services Upgrade Playbook](directory-services-upgrade-playbook.md) for that case.
- **What is hard-coded?** Hostnames and addresses in configuration files and connection strings.
  These break silently when a replacement server gets a new name.
- **What runs once a quarter or once a year?** Month-end, quarter-end and year-end jobs do not show up
  in a week of traffic monitoring.

**Network traffic data is the best single check** on what the application team tells you. Collect it
for at least a full month before a cut-over, longer if the system supports a quarterly process.

---

## Step 6: Put the cost of waiting beside the program budget

**The business case for the program is the cost of not doing it.** Leadership hears "upgrade the
servers" as maintenance. They hear "extended support will cost this much every year, and the program
costs this much once" as a decision.

Extended support pricing is usually per system, per year, and often has a minimum quantity. As one
published example, Microsoft's [ESU FAQ](https://learn.microsoft.com/en-us/lifecycle/faq/extended-security-updates)
(as of October 2026) prices on-premises coverage as "Year 1: 100% of full license price annually",
and the same for years two and three, with a minimum core count per virtual machine. The same page closes the
obvious loophole: "If customers miss a year of ESUs coverage, they may buy coverage for previous
years at the same time they buy coverage for a current period." Waiting does not make the earlier
years free.

**If a vendor offers a custom or negotiated support agreement, ask whether it carries a minimum
number of systems, and get the answer in writing.** A minimum changes the arithmetic: covering 30
stragglers can cost the same as covering a hundred, which makes the last few systems far more
expensive per system than the first few. Read the actual terms; they change between product
generations.

**Unknown business case:**

> The operating system goes out of support next year. We need to upgrade approximately 500 servers.
> Requesting budget for the migration program.

**Actionable business case:**

> Vendor support for 500 servers ends on a fixed date in 14 months. After that date there are no
> security fixes. Extended support is quoted at $1.6 million per year, rising if we fall below the
> vendor's minimum count. The migration program is estimated at $3.2 million, one time. Two years of
> extended support costs as much as the whole program and leaves us where we started. We expect a
> third of the estate to be decommissioned rather than upgraded, which is where most of the savings
> are.

The numbers in the actionable version are illustrative. The structure is the point: one-time cost, annual
cost of waiting, and the comparison on one line.

---

## Step 7: Staff it, and run the program apparatus

**Your own engineers are already running the estate.** An end-of-support program asks them to do a
second job on top of the first. Plan for added capacity from the start rather than discovering in
month four that the upgrade rate is set by whoever has spare evenings.

- **Contract engineers for the volume work.** Temporary engineers who build, upgrade and migrate
  servers to a standard procedure. Bring them in once the triage is far enough along to give them a
  queue. See [Vendor and SOW Management](https://github.com/ChefPlex/tpm-templates/blob/main/vendor-and-sow-management.md)
  for writing the statement of work (SOW) so you pay for systems completed, not hours attended.
- **Your own engineers for the hard cases.** The unowned, the central, the oddly configured. These
  need people with history.
- **Application owners for testing and sign-off.** Their time is the scarcest and the least under
  your control. Schedule it early.

Run the full apparatus, because a fixed external date gives no room to recover from a lost decision:

| Artifact | What it holds for this program |
|:---|:---|
| **Charter** | The date, the scope, the four triage outcomes, who decides for which systems. See the [Program Charter Template](https://github.com/ChefPlex/tpm-templates/blob/main/program-charter-template.md). |
| **RAID log** (risks, assumptions, issues, dependencies) | Unowned systems, applications that cannot move, vendor dependencies. See the [RAID Log Guide](https://github.com/ChefPlex/tpm-templates/blob/main/raid-log-guide.md). |
| **Decision log** | Every extended-support purchase and every decommission over an objection, with who decided. |
| **Communication plan** | Who hears about which outage, and when. Application owners hear before users do. |
| **Budget sheet** | Program cost to date against plan, and the extended-support exposure for whatever remains. |

**Use the steering committee for decisions, not updates.** A chief information officer (CIO) level
committee is the right place to settle which group owns a boundary system, to approve an extended
support purchase, and to force an owner to claim a server. Bring it those decisions with a
recommendation. Status can go in writing.

---

## Step 8: Test first, then cut over under change control

**Build a separate test environment before production is touched.** Upgrade a copy of a
representative system, run the application against it, and let the owner test it. The first
upgrade of each kind of system is where the surprises are. Have them in the lab.

Every production cut-over then follows the same discipline:

- **Formal change control.** A change request, reviewed and approved, for every cut-over.
- **A written method of procedure (MOP).** The step-by-step plan for the change: who does what, in
  what order, how to check it worked, and how to back out.
- **After hours.** Schedule cut-overs in agreed windows when the business can absorb an outage.
- **A named go or no-go decision.** Someone with authority checks the pre-conditions and says go.
- **Owner sign-off after.** The application owner confirms it works before the change is closed.

```text
METHOD OF PROCEDURE (MOP) - one per cut-over
------------------------------------------------------------
System / change request:
Window (date, start, end, time zone):
Implementer:                 Backup implementer:
Application owner on call:   Go / no-go decision by:
Pre-checks (backup taken and verified, snapshot, monitoring paused, users notified):
Freeze point (if data moves): when the old system stops taking changes, and who announces it
Steps (numbered, each with owner and expected result):
Verification (what the owner tests, and what "working" looks like):
Back-out trigger (the condition that means we roll back):
Back-out steps (numbered, with time needed):
Post-change (monitoring resumed, CMDB updated, funnel updated, owner sign-off):
------------------------------------------------------------
```

For a complex cut-over, add a clock time to each step and a second person who checks it before the next step starts.

⚠️ **Write the back-out time into the window.** If backing out takes two hours, the decision to back
out has to be made two hours before the window closes. Teams that skip this finish a failed change at
7 a.m. with the business arriving.

---

## Step 9: Report the funnel and the cost of what remains

**A single percentage hides the only things leadership can act on.** Report three things every
month: the funnel, the rate, and the exposure.

- **The funnel.** Counts by stage (Step 2), with the change since last month.
- **The rate.** Systems completed per month, against the rate needed to finish before the freeze.
- **The exposure.** What extended support would cost for everything not yet complete if the date
  arrived today.

The exposure line is what keeps the business case alive. It falls as the program works, and it gives
leadership a number that is worth paying attention to. See the
[Program Status Reporting Framework](https://github.com/ChefPlex/program-reporting-frameworks/blob/main/program-status-reporting-framework.md)
for the rest of the report.

---

## Step 10: Decide what happens to the residual before the date, not on it

**Something will be left.** An application whose vendor never certified the new version. A system
that runs a process nobody can stop. An appliance with an embedded operating system you cannot touch.

Decide each one explicitly, before the date:

- **Pay for extended support**, for a named period, with a named owner and a plan to exit.
- **Isolate it.** SA-22 names this mitigation: "The increased risk of using unsupported system
  components can be mitigated, for example, by prohibiting the connection of such components to
  public or uncontrolled networks, or implementing other forms of isolation."
- **Accept it on the record**, with the risk owner's name, a review date, and the reason.

Record each one in an exception register. An unsupported system with a decision, an owner and a
review date is a managed risk. One without them is a finding waiting for an auditor.

---

## Templates

### Per-system triage record

```text
SYSTEM TRIAGE RECORD
------------------------------------------------------------
System name / identifier:
Found by (scanner / CMDB / console / team):
Environment (production / test / development):
Internet-facing (yes / no):
Owner (person, by name):            Owning group:
Business function:
Runs (applications, services, scheduled jobs):
Calls (databases, shares, services):
Called by / authenticates through it:
Periodic jobs (monthly / quarterly / yearly):
Funnel stage: Identified / Analysis underway / Decommission planning / Upgrade planning / Complete
Outcome: Decommission / Upgrade in place / Replace or migrate / Extended support
Reason for outcome:
Decision by and date:
Target window:
Extended support needed (yes / no) and annual cost:
Exception register entry (if kept past the date):
------------------------------------------------------------
```

### Monthly funnel status block

```text
END-OF-SUPPORT PROGRAM STATUS - [month]
------------------------------------------------------------
Vendor end-of-support date:         Change freeze starts:
Change windows remaining:           Systems per window (planned):

Funnel                    This month   Last month   Change
Identified                ____         ____         ____
Analysis underway         ____         ____         ____
Decommission planning     ____         ____         ____
Upgrade planning          ____         ____         ____
Complete                  ____         ____         ____
Total in scope            ____         ____         ____

Discovery note (where new systems came from, whether it is slowing):
Completion rate (per month):  actual ____   needed to finish ____
Unowned systems:              ____ (claim deadline: ____)
Extended-support exposure if the date were today: $____ per year
Decisions needed from steering committee:
1.
2.
------------------------------------------------------------
```

---

## Worked example: Larkspur Freight

*Fictional company, invented numbers.*

Larkspur Freight runs its logistics systems on a server operating system whose vendor support ends in
14 months. Two IT groups share the estate: corporate IT and the operations technology group that
runs the warehouses. A CIO steering committee meets monthly.

**Month 1.** The CMDB lists 400 servers on the old version. The TPM writes the end date into the
charter, subtracts the year-end change freeze, and finds 11 months of usable windows. She sets a
triage deadline at month 5.

**Month 2.** The vendor's assessment scanner runs across the corporate network and finds 90 servers
missing from the CMDB. The virtualization console adds 30 powered-off virtual machines. The status
report shows 520 identified and explains where the 120 came from.

**Month 3.** Operations technology scans the warehouse segments and adds 80 more. Total: 600. The
report says discovery should now flatten, because every segment has been scanned once.

**Month 4.** Ownership is the bottleneck. 150 servers have no claimant. The TPM publishes the list
with a 30-day claim deadline, and the steering committee agrees that anything unclaimed goes to the
decommission track with a shutdown notice.

**Month 5.** Triage is complete:

| Outcome | Systems |
|:---|:---|
| Decommission | 200 |
| Upgrade in place | 250 |
| Replace or migrate | 120 |
| Extended support (residual) | 30 |
| **Total** | **600** |

A third of the estate goes away. That was not in the original budget at all, and it cuts the upgrade
workload by the same third.

**Month 6.** The business case is refreshed for the committee. Extended support for all 600 would
cost $1.8 million per year. The program budget is $3.6 million. Covering only the 30 residual
servers would cost $300,000 per year, because the vendor's minimum count means 30 servers are billed
as if they were 100. The committee approves four contract engineers for six months.

**Months 7 to 13.** The test environment catches a reporting application that fails on the new
version. The fix goes to the vendor of that application, and those 12 servers join the residual
list, bringing it to 42. Still under the vendor minimum, so the annual cost does not change. Every
cut-over runs after hours under a MOP. The funnel report shows the completion rate, and the
extended-support exposure falls from $1.8 million toward the residual figure.

**Month 14.** 558 servers are complete or decommissioned. 42 are on extended support, each with an
owner, an isolation plan and a review date in the exception register.

What the TPM would do differently: run the warehouse scans in month 2 instead of month 3, and start
the decommission track the day the first unowned list was published.

---

## Where this breaks

- **When the vendor date is not the real date.** A security policy, a cyber insurance requirement or
  a contract can require you off earlier than the vendor. Check before you plan backward from the
  wrong date.
- **When "extended support" becomes the plan.** It is priced to be temporary and it ends. The vendor
  overview cited above lists a three-year window with a fixed end. A program that buys the first year
  and stops working will be buying the second year under worse conditions.
- **When the funnel is gamed.** Moving systems into "decommission planning" without an owner's
  agreement makes the numbers look better and stalls at shutdown. Count a system as decommissioned
  only when it is off.
- **When the application, not the operating system, is the blocker.** If a third-party application is
  not certified on the new version, no amount of server work helps. Find these in triage and escalate
  them to the application's vendor early.
- **When the two IT groups report different numbers.** Agree one master list and one owner of it in
  month one. Two lists produce two status reports and a committee that trusts neither.
- **When the cut-over is clean and the dependency is not.** A server can upgrade perfectly and still
  break the quarterly job that called it by its old name. That is why the dependency map includes
  what calls in, and why owner sign-off is part of every change.
- **When discovery never stops.** If the count is still rising sharply in the last third of the
  program, scope is out of control. Freeze the scope with the committee and handle late finds as
  exceptions with their own plan.

---

## Related

- [Directory Services Upgrade Playbook](directory-services-upgrade-playbook.md)
- [Vulnerability Remediation Runbook](https://github.com/ChefPlex/security-program-playbooks/blob/main/vulnerability-remediation-runbook.md)
- [Encryption Program Playbook](https://github.com/ChefPlex/security-program-playbooks/blob/main/encryption-program-playbook.md)
- [RAID Log Guide](https://github.com/ChefPlex/tpm-templates/blob/main/raid-log-guide.md)
- [Program Charter Template](https://github.com/ChefPlex/tpm-templates/blob/main/program-charter-template.md)
- [Vendor and SOW Management](https://github.com/ChefPlex/tpm-templates/blob/main/vendor-and-sow-management.md)
- [Rescuing Work That Is Off Track](https://github.com/ChefPlex/tpm-templates/blob/main/rescuing-work-that-is-off-track.md)
- [Program Status Reporting Framework](https://github.com/ChefPlex/program-reporting-frameworks/blob/main/program-status-reporting-framework.md)

---

*Part of the [ChefPlex TPM portfolio](https://github.com/ChefPlex/tpm-templates/blob/main/PORTFOLIO-GUIDE.md). Maintained by [Eric White](https://www.linkedin.com/in/edwhite). Licensed under [CC BY 4.0](LICENSE).*
