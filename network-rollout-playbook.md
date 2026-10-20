# Network Rollout Playbook

**How to roll a network change across many sites, from a handful of campuses to several hundred stores: what to order first, how to make one site plan repeatable, and how to cut each site over without the business noticing.**

A network rollout is closer to a construction schedule than a software release. The configuration for the new equipment is written once, tested once and copied to every site. What takes the time is everything around it: the circuit the carrier delivers on its own calendar, the wiring closet nobody has opened in ten years, the power circuit too small for the new switches, and the site manager who was never told the network would be down on Thursday night.

This playbook is for the technical program manager (TPM) who owns that schedule. It assumes the network engineers know how to design and configure the equipment. The TPM makes sure every site is ready on the day its window opens, and that everyone affected was told before it opens.

Written from rollouts across retail store networks, multi-campus office networks and colocation circuit turn-ups.

---

## The short version

1. **Plan in the right order.** The schedule comes after the requirements, because it is built from them.
2. **Write the non-functional specifications first.** Capacity, scaling and disaster recovery (DR) decide the site design. They are not a section to fill in afterward.
3. **Circuits are the critical path.** Order them by priority tier, from two carriers where the design needs it, and ask where the cables physically run.
4. **Build one site plan and copy it.** Then survey every site to find the ones that do not fit the copy, before the cut-over finds them for you.
5. **Give every site the same pre-work window and stagger the starts,** so cut-overs land on a steady weekly cadence that the field team, the carriers and the site managers can plan around.
6. **Cut each site over in small changes,** each with its own back-out, and test the business functions afterward rather than the link light.
7. **Publish status on a clock during every cut-over,** at times announced in advance, whether or not anything has changed.
8. **The program closes when the old circuits stop billing** and network operations owns the result.

---

## Step 1: Plan in the right order

Each stage produces something the next one needs: no scope without a charter, no site design without the requirements, and no schedule without the circuit lead times. The PMI (Project Management Institute) process groups are in parentheses for anyone who works from them.

| Stage | What comes out of it | What it feeds |
|:---|:---|:---|
| **Start** (Initiating) | Charter: sites in scope, capital and expense budget, sponsor, what is out. The first names on the stakeholder list | Requirements, scope |
| **Plan** (Planning) | Requirements, the site standard (Step 2), circuit and hardware orders with lead times (Step 3), one work package per site, the risk list | The wave schedule |
| **Run** (Executing) | Site pre-work and cut-overs (Steps 4 to 7) | Tracking, close-out |
| **Track** (Monitoring and Controlling) | Change requests and re-baselines, in one place | An updated schedule and budget |
| **Close** (Closing) | Old circuits cancelled, handover to network operations, close-out report (Step 9) | Operations |

### Unknown vs Actionable

**Unknown plan:** a project plan whose first task is "Schedule site cut-overs," with requirements gathering running in parallel.

**Actionable plan:** the wave schedule has the approved site design and the circuit order dates as predecessors. The site design has the non-functional specifications as its predecessor, and they have the signed requirements as theirs. When requirements slip, the plan shows the cut-overs slipping with them, on the day it happens.

The unknown plan looks faster for the first month. Then the cut-over dates arrive and every slip since the start shows up at once.

## Step 2: Write the non-functional specifications before the site design

**The non-functional specifications describe how much the network has to carry, how it grows, and what happens when part of it fails.** They are written before the site design, because they are what the site design has to meet. A rollout is usually driven by an application (a new point-of-sale system, a centralized ordering system, a move of users to virtual desktops), and the application owner rarely knows what it needs from the network until someone asks.

| Specification | The question | Example of a usable answer |
|:---|:---|:---|
| **Capacity** | What does each site send and receive at its busiest hour, with the new application running? | Measured in a performance test of the application over a link the same size as a real site's, at peak volume |
| **Scaling** | What is the rule for sizing a site? | "Two access ports per register and workstation, plus 25 percent spare per closet" |
| **Growth** | Where will the load be halfway through the equipment's life? | Store bandwidth sized to year three on the measured trend, with the next circuit tier priced now |
| **Disaster recovery** | Where does each site's traffic go when a data center fails, and how fast? | Each site reaches the DR data center over its second path within the application's recovery time objective (RTO) |
| **Availability** | Which sites need two paths, and which can live with one plus a backup? | Data centers and call centers on two carriers; stores on one carrier with a backup link from the other |

**Test the application over a real-sized link before the site design is signed.** A performance test in the data center, on a fast local network, tells you nothing about a store at the end of a small circuit. The capacity number that matters comes from the busiest hour at the smallest site.

The [Data Center and Storage Build-Out Playbook](data-center-build-out-playbook.md) covers recovery targets and growth sizing in more depth (its Steps 1 and 4). The same rules apply to the network that connects the sites.

## Step 3: Circuits are the critical path, so order them by tier

**The carrier's delivery date sets the schedule for every site that needs a new circuit, and nobody on the program controls it.** Order circuits as soon as the site list and the capacity specification are stable. Do not wait for the detailed design.

Sort sites into priority tiers and order in tier order:

| Tier | Typical sites | Why this order |
|:---|:---|:---|
| **1** | Data centers and colocation sites | Every other site's traffic goes through them, so their circuits are predecessors for everything |
| **2** | Large campuses, call centers, distribution centers | High user counts and business hours that leave few cut-over windows |
| **3** | Stores, branches and small offices | High volume, standard design, schedulable in waves |

### Two carriers do not guarantee two paths

Using two carriers is a common way to remove a single point of failure and, at volume, to get better pricing from both. It does not prove the paths are separate. National Institute of Standards and Technology (NIST) Special Publication (SP) 800-53 Rev. 5 control CP-8(2) asks organizations to "Obtain alternate telecommunications services to reduce the likelihood of sharing a single point of failure with primary telecommunications services." Its discussion explains why that takes work: "In certain circumstances, telecommunications service providers or services may share the same physical lines, which increases the vulnerability of a single failure point. It is important to have provider transparency for the actual physical transmission capability for telecommunication services." ([source](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r5.pdf), as of October 2026)

**Ask both carriers for the physical route into each tier 1 and tier 2 building,** including the building entrance and the last mile. Two contracts that arrive through the same conduit fail together.

### Put the one-time and recurring cost on one line

A circuit program has a capital cost (equipment, installation) and a recurring cost (the monthly circuit charges). Leadership approves it on both. Show them together, with the old circuit charges that go away, and give the net change per year.

> **Template A: Circuit order tracker** (one row per circuit)
>
> | Site | Tier | Carrier | Type and size | Order date | Carrier committed date | Delivered and tested | Physical route confirmed (Y/N) | Old circuit | Old circuit cancel date |
> |:---|:---|:---|:---|:---|:---|:---|:---|:---|:---|
> | | | | | | | | | | |
>
> **Rule:** the carrier's committed date is a risk, never a commitment. Any circuit without a committed date four weeks before its site's pre-work starts goes on the risk register.

The old circuit's cancel date is part of the plan from the first day. Old and new circuits run side by side until the new one carries production, and every month of overlap is paid twice. Check each contract's notice period and early termination terms before the cancel dates are set.

## Step 4: Survey every site, and look for the one that is different

**Every rollout has a standard site design, and every rollout has sites that do not match it.** The survey's job is to find them while there is still time to plan for them.

What the survey covers, per site:

- **The closets.** The main distribution frame (MDF) and each intermediate distribution frame (IDF): rack space, ventilation, and whether the door locks.
- **Power.** The circuit feeding the closet against the draw of the new equipment. Newer switches, especially ones that power phones and access points over the network cable, can draw much more than what they replace.
- **Wiring.** Condition and labeling of the cabling in the closets. Old, unlabeled cabling turns a two-hour swap into a night of tracing.
- **The existing configuration.** What is actually configured on the old equipment, compared to the standard. Hand-edited exceptions found here have to be carried forward or retired on purpose.
- **Access.** Who has keys, what hours the building is open, and whether a landlord, a mall or a second tenant controls the closet.
- **Local contacts.** The site manager, and someone on site during the cut-over who can describe what they see.

**Classify each site from its survey:** standard, standard with listed exceptions, or custom. Standard sites go into the waves. Custom sites get their own plan and usually go late, after the method has been proven on the easy ones.

The site that is different is usually one of these: the oldest site, where the wiring and power have never been touched; a leased site where someone else owns the closet; a call center that cannot lose service during business hours; a site that was acquired and never brought to standard. Ask the field engineers which sites they expect trouble from. They usually have a list.

> **Template B: Site survey record**
>
> | Item | Finding | Exception to standard? | Owner of the fix | Fix by |
> |:---|:---|:---|:---|:---|
> | Site, address, tier | | | | |
> | MDF / IDF count and locations | | | | |
> | Rack space and ventilation per closet | | | | |
> | Closet power: circuit capacity vs planned draw | | | | |
> | Wiring condition and labeling | | | | |
> | Existing equipment and configuration exceptions | | | | |
> | Circuit(s) in place and on order (Template A) | | | | |
> | Building access, hours, who controls the closet | | | | |
> | Site manager and on-site contact for cut-over night | | | | |
> | Business hours, peak days and blackout dates | | | | |
> | **Classification: Standard / Standard with exceptions / Custom** | | | | |

## Step 5: Give every site the same pre-work window, and stagger the starts

**Every site gets the same fixed pre-work window before its cut-over, and the windows start a week apart.** With a four-week window and one cut-over a week, about four sites are in pre-work at any time, each at a different stage. The field team works on several sites at once, and each cut-over arrives on a known day. No cut-over is rushed because its pre-work started late, and carriers, facilities and site managers can see weeks ahead when they are needed.

An illustrative four-week pre-work window:

| When | Pre-work |
|:---|:---|
| **Cut-over minus 28 days** | Survey confirmed, exceptions assigned, circuit delivery date confirmed |
| **Minus 21 days** | Equipment configured from the standard and staged in the lab; site exceptions added and reviewed |
| **Minus 14 days** | Power and wiring work in the closets complete; new circuit delivered and tested |
| **Minus 7 days** | Change approved; site manager and business owners notified; on-site contact confirmed |
| **Minus 2 days** | Go / no-go review: every item above confirmed, or the site moves to a later week |
| **Cut-over** | Executed under the change (Step 6), with status on a clock (Step 7) |

**Run the first one or two waves as pilots,** on standard sites close to the core team. A pilot is a test of the procedure as much as of the equipment. Adjust the pre-work window after the pilot if it was too short, before the cadence is set for everyone else.

### When the plan changes, re-baseline it in the open

Surveys add work. A site counted as four switches turns out to have six, or the closet needs new power before anything can be installed. Engineers get pulled onto higher-priority work with no return date. **Each of those is a change request with a new baseline,** approved through the same integrated change control the charter set up, and reported with the reason.

### Unknown vs Actionable

**Unknown status line:** "Rollout is 60 percent complete. Some sites delayed."

**Actionable status line:** "24 of 40 sites cut over; plan was 26. Two sites moved from week 14 to week 17: their closets need new power circuits, ordered on the 3rd, facility date the 28th. Change request 4 re-baselines the finish from October 31 to November 14 for those two sites only. All other weeks unchanged."

## Step 6: Cut each site over in small changes

**Split a large cut-over into several smaller changes, each with its own window and its own back-out.** A core or campus cut-over done in one night puts every risk in one window.

A sequence that works for core and campus equipment:

1. **Rack, stack and update.** Install the new equipment beside the old and bring its software to the approved version. No production traffic.
2. **Bring it online.** Connect the new equipment to the network and confirm it is managed and monitored. Production still runs on the old equipment.
3. **Connect to production.** Move routing or uplinks to the new equipment, with the old equipment still cabled and ready to take traffic back.
4. **Physical transition.** Move the remaining connections over, then remove the old equipment.

Store and branch cut-overs are usually small enough for one window. Use the same discipline inside it: new equipment staged and tested before the old is touched, and a back-out that fits in the window.

**Treat rule migration as its own workstream.** When the change moves access control lists (ACLs) or firewall rules from old equipment to new, count the rules, give the migration an owner, and test it before the cut-over. A few thousand rules carried over by hand will contain mistakes, and the mistake shows up as an application that stops working for one site.

NIST SP 800-53 Rev. 5 control CM-3(2): "Test, validate, and document changes to the system before finalizing the implementation of the changes."

**Test the business functions afterward, from the site.** A link light and a successful ping prove the network is up. They do not prove a register can take a payment, a call center agent can take a call or a printer can print a label. Agree the list of business tests with each site type during planning (Step 2's specifications are the place for them), and have the on-site contact run them before the change is closed.

> **Template C: Site cut-over checklist** (one per site, per change)
>
> **Before the window**
> - [ ] Pre-work items from Step 5 all confirmed at the go / no-go review
> - [ ] Change approved; method of procedure (MOP) written, with step timings from the pilot
> - [ ] Current configuration of the old equipment backed up and stored off the device
> - [ ] New circuit tested end to end; old circuit still in place
> - [ ] Rule migration (if any) tested; rule count matches the source
> - [ ] Site manager, help desk and business owners notified, with the status times (Step 7)
>
> **In the window**
> - [ ] Cut-over steps run in order, each checked against its expected result
> - [ ] **Go / no-go point:** time, and the criteria that must be true to continue
> - [ ] **Back-out trigger:** the result, or the clock time, that forces a back-out
> - [ ] Business tests run from the site by the on-site contact
>
> **After the window**
> - [ ] New equipment in monitoring, with alerts confirmed (Step 8)
> - [ ] Baseline configuration recorded
> - [ ] Help desk watching for site calls for the next business day
> - [ ] Change closed with anything that surprised the team, so the next site's MOP improves

## Step 7: Publish status on a clock during the cut-over

**Announce the status update times before the cut-over starts, and send an update at each one, even when nothing has changed.** A cut-over that goes quiet for four hours reads as a cut-over in trouble, and people start calling the engineers doing it, which slows it down. With a published clock, business owners and the help desk know when the next word is coming and stop asking in between. If an update is missed, the program lead follows up.

For a weekend or overnight cut-over, four or five updates usually cover it: start, a midpoint, the go / no-go point, completion, and one the next morning after business tests. Keep each update the same shape so a reader can find what changed in a few seconds.

> **Template D: Timed status update**
>
> ```text
> Subject: Update - [site / change] - [time] - [ON TRACK / DELAYED / BACKING OUT / COMPLETE]
>
> Status:        [one line]
> Done since last update:
> In progress:
> Issues:        [none, or each issue with an owner]
> Business impact right now:
> Next update:   [time, or "final"]
> Contact:       [cut-over lead, and how to reach them]
> ```
>
> **Rule:** send it at the announced time. A late update is worse than an update that says "no change."

Who receives it is part of the communication plan from Step 1. The [Communications Plan Template](https://github.com/ChefPlex/tpm-templates/blob/main/communications-plan-template.md) covers the rest.

## Step 8: Monitor before, during and after

**Put monitoring in place before the rollout starts, so there is a baseline to compare against.** Without it, the first week after a cut-over turns into an argument about whether the network was always this slow.

- **Before:** monitoring on the old network for at least a few weeks, so normal utilization, latency and error rates are known per site.
- **During:** each new device added to monitoring as part of its cut-over (Template C), with alerts tested.
- **After:** the same measures compared to the baseline, site by site, for the first weeks.

Record each site's new configuration as its baseline. NIST SP 800-53 Rev. 5 control CM-2 asks organizations to review and update the baseline configuration "When system components are installed or upgraded." A rollout replaces components at every site, so every cut-over is that moment.

Measures worth reporting each week:

| Measure | What done looks like |
|:---|:---|
| Sites cut over, against the wave plan | All, on the re-baselined dates |
| Circuits delivered, against the carriers' committed dates | All tier 1 and tier 2 on time; late tier 3 circuits moved to later waves, not forced |
| Incidents caused by the program | The headline number, reported with each cause |
| Sites in pre-work with an open item at minus 7 days | Zero; each one is a site likely to move |
| Old circuits still billing after their cancel date | Zero at close |

## Step 9: Close the program

**The rollout is finished when the old circuits are cancelled, the closets are cleaned up, and network operations owns the new network as normal work.** Expect several weeks of work after the last cut-over.

- **Warranty and finish work.** Agree a warranty period with the installers, and schedule the wiring cleanup that cut-over nights skip. Old cable left in a closet becomes the next survey's problem.
- **Close the procurements.** Cancel every old circuit on its planned date and confirm the last bill. End support contracts on retired equipment.
- **Hand over.** Network operations gets the baselines, the as-built drawings, the site exceptions and the monitoring. Remaining minor items move to their queue as operational work, by name.
- **Close-out report.** Approved budget against actual, capital and expense separately; sites delivered against plan; incidents; change requests and why. The [Program Close-Out Report Template](https://github.com/ChefPlex/tpm-templates/blob/main/program-close-out-report-template.md) has the structure.

---

## Worked example: Ostrander Farm Supply (fictional)

*Ostrander Farm Supply is an invented company. All names and numbers below are made up for illustration.*

**The situation.** Ostrander runs 412 stores, three regional offices, a call center and two data centers, one of them in a colocation facility. The company is rolling out a store ordering system that sends product images and stock counts to the data centers all day. Store links were sized for card payments and email. The network program has to be ready before the application reaches each store.

**Step 1 and 2: outputs and specifications.** The charter sets a $2.35 million capital and $570,000 expense budget and puts store point-of-sale changes out of scope. A performance test of the ordering application over a link the size of a small store's shows the link saturating during the midday stock sync. The capacity specification is written from that busiest hour, and the scaling rule is two ports per register and workstation plus 25 percent spare.

**Step 3: circuits.** Tier 1 (both data centers) and tier 2 (the regional offices and the call center) get circuits from two carriers. Stores get a primary circuit from one carrier and a broadband backup from the other. Asked for physical routes, one carrier's tier 1 circuit and the other's turn out to enter the colocation building through the same conduit. The second carrier re-routes through the building's other entrance, at extra cost and a three-week delay. The business case shows recurring circuit cost rising by $86,000 a year against $131,000 a year in old circuits that go away.

**Step 4: surveys.** Of 412 stores, 351 are standard, 47 are standard with exceptions (mostly closet power too small for the new switches), and 14 are custom. Seven of the custom sites are in shopping centers where the landlord controls the closet and requires two weeks' notice for access. The field engineers name the oldest store, which has no labeling at all in its closet; it is scheduled last in its region with an extra night.

**Step 5: cadence.** After two pilot weeks of five stores each, stores move in waves of 20 a week, each store with a four-week pre-work window. The first pilot finds that the lab-staged configuration is missing the backup link's settings; the standard is fixed before wave 1. In month 3 the surveys of the regional offices find 40 percent more switches than the inventory showed. Change request 2 re-baselines the regional office finish by five weeks and leaves the store waves unchanged.

**Step 6 and 7: cut-overs.** Each regional office core goes over in three changes on three weekends: bring online, connect to production, physical transition. The call center moves outside business hours, and its business tests are a real inbound call, an agent login and a callback. During the largest office's cut-over, updates go out at 8:00, 12:00, 16:00 and 20:00 on Saturday. The 16:00 update reports a backed-out uplink and the new plan.

**Step 8 and 9: monitoring and close.** The monitoring baseline taken two months earlier settles a complaint about one store's speed: its utilization had been at the limit before the cut-over. At close, 14 old store circuits are still billing past their cancel dates because their cancellation notices went to the wrong carrier account. They are cancelled and the overcharge is credited. Wiring cleanup at the regional offices runs for six weeks after the last cut-over, inside the installers' warranty period.

---

## Where this breaks

- **The capacity test is run in the wrong place.** An application tested on the data center's local network will pass. The rollout then sizes store links from a number that has never seen a store.
- **The carrier date is treated as a commitment.** It is the item most often late. Plan waves around delivered circuits, and move a store to a later week rather than cut it over onto a circuit that has not been tested.
- **The survey is skipped for "standard" sites.** The standard is a description of what the sites should look like. Some of them do not, and the cut-over night is the expensive place to find out.
- **Pre-work windows shrink to protect the cadence.** When the schedule tightens, the temptation is to cut the four weeks to two. The sites that get the shorter window are the ones that fail. Move the site instead.
- **Two carriers are reported as two paths.** Until the physical routes are confirmed, the diversity is an assumption.
- **The business tests are a ping.** The network is up and the registers cannot take payments.
- **The tail never closes.** Old circuits billing for months after their sites moved, closets full of disconnected cable, and minor items with no owner.
- **This method is heavy for a small change.** Replacing one switch in one closet does not need tiers, waves and timed updates. Use the templates where the number of sites, or the importance of one site, justifies them.

---

## Sources

All accessed as of October 2026.

- NIST SP 800-53 Rev. 5, *Security and Privacy Controls for Information Systems and Organizations* (CP-8 telecommunications services, CM-2 baseline configuration, CM-3 configuration change control): https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r5.pdf
- Project Management Institute, *A Guide to the Project Management Body of Knowledge (PMBOK Guide)*, fifth edition (2013), for the process group names in Step 1.

NIST SP 800-53 is written for federal systems. Private organizations are not bound by it; it is cited here as a tested, public description of what change control and alternate telecommunications should cover.

## Related

- [Data Center and Storage Build-Out Playbook](data-center-build-out-playbook.md): recovery targets, growth sizing and long-lead procurement for the sites the network connects
- [End-of-Support Migration Playbook](end-of-support-migration-playbook.md): when the rollout is driven by network equipment reaching end of support
- [The RAID Log](https://github.com/ChefPlex/tpm-templates/blob/main/raid-log-guide.md): carrier dates and site exceptions as dependencies and risks
- [Program Charter Template](https://github.com/ChefPlex/tpm-templates/blob/main/program-charter-template.md): the first output in the Step 1 chain
- [Communications Plan Template](https://github.com/ChefPlex/tpm-templates/blob/main/communications-plan-template.md): who hears about each cut-over, and when
- [Vendor and SOW Management](https://github.com/ChefPlex/tpm-templates/blob/main/vendor-and-sow-management.md): installer and carrier statements of work with acceptance criteria
- [Program Close-Out Report Template](https://github.com/ChefPlex/tpm-templates/blob/main/program-close-out-report-template.md): the Step 9 report

---

*Part of the [ChefPlex TPM portfolio](https://github.com/ChefPlex/tpm-templates/blob/main/PORTFOLIO-GUIDE.md). Maintained by [Eric White](https://www.linkedin.com/in/edwhite). Licensed under [CC BY 4.0](LICENSE).*
