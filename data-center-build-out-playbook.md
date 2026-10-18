# Data Center and Storage Build-Out Playbook

**How to stand up or modernize data center capacity across a primary and a disaster recovery site: what sets the schedule, what to settle on paper before anything is racked, and how to prove the second site actually works.**

This playbook covers four kinds of work that usually arrive tangled together: storage and network fabric modernization across a primary and a disaster recovery (DR) site, rack build-out and floor layout, moves between colocation facilities, and folding a new application or web property into both sites. It is written for the technical program manager (TPM) who has to get all four done without an outage, and it assumes you are not the electrical engineer, the storage architect or the facilities lead. You are the person who makes sure their work lands in the right order.

**The bottom line:** infrastructure build-outs rarely fail on technology. They fail on a purchase order placed six weeks too late, a rack that will not fit the power it was promised, a DR site nobody has ever failed over to, and a migration with no way back. Every step below is about one of those.

These are patterns that generalize. The specifics of any one facility will not.

---

## Step 1: Write down what "recovered" means before you buy anything

**The first deliverable is not a design. It is a number for how long each system can be down and how much data it can lose.** Every later decision, from how far apart the sites are to what kind of storage replication you buy, follows from those two numbers. A program that starts with hardware selection is choosing an answer before anyone has asked the question.

The National Institute of Standards and Technology (NIST) Special Publication (SP) 800-34 Rev. 1 gives the standard definitions ([source](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-34r1.pdf), as of October 2026):

| Term | Definition (verbatim, NIST SP 800-34 Rev. 1) |
|:---|:---|
| **Recovery Time Objective (RTO)** | "RTO defines the maximum amount of time that a system resource can remain unavailable before there is an unacceptable impact on other system resources, supported mission/business processes, and the MTD." |
| **Recovery Point Objective (RPO)** | "The RPO represents the point in time, prior to a disruption or system outage, to which mission/business process data must be recovered (given the most recent backup copy of the data) after an outage." |
| **Maximum Tolerable Downtime (MTD)** | The total outage time the business owner will accept for the process, all impacts included. The RTO sits inside it: "Because the RTO must ensure that the MTD is not exceeded, the RTO must normally be shorter than the MTD." |

**Get these from the business owner, in writing, per system.** Not per site, and not one number for everything. A payments system and an internal wiki do not share a recovery target, and averaging them produces a DR design that is overbuilt for one and underbuilt for the other.

### Choose the facility class to match the target, not the brochure

Facility redundancy is usually described with the Uptime Institute Tier classification. Its public page is careful about what the Tiers are: "The data center Tier definitions define criteria, but not specific technology options or design choices to meet the Tier." It also warns against reading the levels as a ranking: "This progression does not mean that a Tier IV data center is better than a Tier II" ([Uptime Institute, Tier Classification System](https://uptimeinstitute.com/tiers), as of October 2026).

The level that matters most in practice is concurrent maintainability, which the same page defines this way: "A Tier III data center is concurrently maintainable with redundant components as a key differentiator, with redundant distribution paths to serve the critical environment." In plain terms: can the facility service its power and cooling without taking your equipment down? **Ask every colocation provider that question directly, and ask whether their rating is certified or self-described.** The full Tier standard is a paid document. Buy it if the contract will reference it.

## Step 2: Long-lead procurement is the critical path

**On almost every build-out, the schedule is set by the things you have to order, not the things you have to do.** Installing a storage array takes days. Getting it delivered, getting the circuit that feeds it, and getting the network carrier to light the link between sites can take months, and none of that clock starts until a purchase order exists.

| Long-lead item | Why it bites | What the TPM does |
|:---|:---|:---|
| **Storage arrays, switches, servers** | Configured-to-order hardware ships on the vendor's schedule, and a single missing part holds the whole order | Get a committed ship date in writing per line item, not a quote validity date |
| **Power to the cage or rack** | New circuits, power distribution units (PDUs) and breaker work are the facility's job, on the facility's calendar | Order power with the space, not after the design is final |
| **Network circuits and inter-site links** | Carrier provisioning is outside everyone's control and is the item most often late | Order first. Treat the carrier's date as a risk, never a commitment |
| **Cross-connects** | Cheap, small and forgotten, then they block go-live | List every one in the plan with an owner |
| **Contract signature** | Nothing above starts until legal and procurement finish | Put contract review on the plan as a task with a date |

### Weak vs better

**Weak plan line**

> Procure storage hardware. Owner: Infrastructure. Due: Q2.

**Better plan line**

> **P-04: DR storage array, order placed.** Owner: one named storage lead. PO by March 1 to hold a committed ship date of April 15 (vendor quote attached). Install cannot start until P-07 (DR rack power, two feeds) is energized. **If the PO slips, go-live slips day for day; there is no parallel path.**

The better line names the dependency that the weak one hides: the hardware is useless until the power it plugs into exists, and that is a different team's work on a different contract. Log every long-lead item as a dependency with a likelihood and an impact, the way the [RAID log guide](https://github.com/ChefPlex/tpm-templates/blob/main/raid-log-guide.md) treats them.

## Step 3: Plan the floor on paper before anything is racked

**Every rack gets drawn before it gets built.** A rack elevation is a drawing of each rack, slot by slot, showing what goes where. It is the cheapest place to find out that the plan does not work, and the most expensive thing to skip. Moving a loaded array three racks over because the power was on the wrong side is a weekend nobody budgeted for.

What the paper plan has to settle, rack by rack:

- **Power.** The expected draw of everything in the rack against the circuit it is on, with headroom. Two feeds from two independent sources (often called A and B) so a single feed can fail or be serviced. Check that each dual-corded device is actually plugged into both.
- **Cooling.** Where the hot exhaust goes. Equipment that blows the wrong way, or a gap with no blanking panel, defeats the room's cooling design.
- **Cable paths.** Where power and data run, kept separate, with enough slack and labels at both ends. NIST SP 800-53 Rev. 5 control PE-9 (from the physical and environmental protection, PE, family) states the requirement simply: "Protect power equipment and power cabling for the system from damage and destruction." ([source](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r5.pdf), as of October 2026).
- **Weight.** Fully loaded storage and battery equipment is heavy. Check floor loading with the facility before delivery day, not on it.
- **Growth space.** Empty slots, spare power and spare ports reserved on purpose for the next eighteen months (Step 4).
- **Hazards.** Know where water can come from. PE-15 asks for "master shutoff or isolation valves that are accessible, working properly, and known to key personnel" in the same publication.

The detailed facility standards are paid documents and worth buying for the engineers who design to them. ANSI/TIA-942 (ANSI is the American National Standards Institute) is the Telecommunications Industry Association (TIA) data center standard; TIA's own page says it "specifies the minimum requirements for data centers and covers all physical infrastructure including, but not limited to, site location, architectural, electrical, mechanical, fire safety, telecommunication, security and other requirements" ([TIA](https://tiaonline.org/products-and-services/tia942certification/ansi-tia-942-standard/), as of October 2026). For cooling, the American Society of Heating, Refrigerating and Air-Conditioning Engineers (ASHRAE) publishes data center guidance through its datacom resources ([ASHRAE](https://www.ashrae.org/technical-resources/bookstore/datacom-series), as of October 2026). **Your job is to make sure someone qualified has checked the plan against them, not to interpret them yourself.**

> **Template A: Rack elevation and site readiness checklist** (one per rack, filled in before delivery)
>
> | Item | Value | Confirmed by | Date |
> |:---|:---|:---|:---|
> | Site / cage / rack ID | | | |
> | Devices, top to bottom, with slot positions (attach drawing) | | | |
> | Planned power draw in kilowatts (kW) vs circuit capacity (kW), each feed | | | |
> | Feed A source and breaker / Feed B source and breaker | | | |
> | Every dual-corded device on both feeds? (Y/N) | | | |
> | Airflow direction matches the aisle layout? Blanking panels in? | | | |
> | Loaded weight vs floor rating, confirmed with facility | | | |
> | Data cable paths and power cable paths separated and labeled | | | |
> | Cross-connects required (list, with order numbers) | | | |
> | Reserved growth: slots / kW / ports | | | |
> | Facility access list includes install crew for install dates | | | |
> | Asset tags recorded in inventory (Step 9) | | | |
> | **Ready to install: signed by** | | | |

## Step 4: Specify for growth, not for today

**Capacity planning means buying for where the load will be when the equipment is halfway through its life, not where it is on the day you order.** A storage array specified to today's usage is full in its second year, and the expansion you need then is the one the frame, the rack and the power circuit were never sized for.

Ask four questions for every component:

1. **What is the growth rate, and where did it come from?** A measured trend from the last few years beats a guess, and a guess beats nothing. Write down which one you used.
2. **What is the expansion unit?** Some things grow by adding drives to an existing frame. Others need a new frame, a new rack and new power. Know which before you sign.
3. **What runs out first?** Usually not disk. It is power per rack, switch ports or floor space.
4. **What is the scaling rule for repeated sites?** When the same design rolls out to many locations, the specification needs a rule (for example, "one port per device plus 25 percent spare") rather than a hand-sized list per site.

### Weak vs better

**Weak specification:** "DR array sized to match current production capacity."

**Better specification:** "DR array sized to hold production capacity at month 36 on the measured 30 percent annual growth trend, with the frame, rack power and switch ports for one further expansion shelf reserved now. Expansion beyond that is a new frame and is out of this program's scope, recorded as such."

The weak version fails twice: it is undersized by the time it is installed, and DR that is smaller than production is not DR for the systems that do not fit.

## Step 5: Build primary and DR as a pair

**Primary and DR are one design delivered at two addresses.** Programs that build primary first and "do DR next" end up with two sites that drifted apart: different firmware, different layouts, different capacity, and a replication setup that was bolted on rather than designed in.

NIST SP 800-53 Rev. 5 frames the requirements for the second site in the contingency planning (CP) control family ([source](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r5.pdf), as of October 2026). Four of them are worth reading as program requirements:

| Control | Verbatim text | What it means for the plan |
|:---|:---|:---|
| CP-7(1) | "Identify an alternate processing site that is sufficiently separated from the primary processing site to reduce susceptibility to the same threats." | Two cages in the same building, or on the same power grid and flood plain, are not two sites |
| CP-7(2) | "Identify potential accessibility problems to alternate processing sites in the event of an area-wide disruption or disaster and outlines explicit mitigation actions." | If the people who would run the failover cannot get there, the plan needs remote hands or a different site |
| CP-7(3) | "Develop alternate processing site agreements that contain priority-of-service provisions in accordance with availability requirements (including recovery time objectives)." | The colocation and carrier contracts must support the RTO from Step 1, in writing |
| CP-6 | "Ensure that the alternate storage site provides controls equivalent to that of the primary site" | DR is not the place for weaker access control or security |

### Decide how warm the DR site is

NIST SP 800-34 Rev. 1 describes a range from cold sites (space and power only) through warm sites to hot sites. For a hot site it says: "Hot sites should have the most recent version of backed-up data loaded, requiring only updating with data since the last backup." **The RTO and RPO from Step 1 pick the temperature.** A short RTO with a cold site is a plan that cannot be met, and the gap should be stated to the business owner now rather than discovered during an outage.

Practical rules for building the pair:

- **Same design, same firmware, same naming.** Differences between sites are where failovers break.
- **Replication is designed with the storage, not after it.** The bandwidth between sites, the replication mode and the RPO it can actually hold are one decision.
- **DR capacity matches production capacity for everything in scope.** If DR is intentionally smaller, write down which systems do not fail over.

## Step 6: Data migration is its own workstream

**Moving data onto new storage is a separate program inside the program**, with its own owner, its own plan and its own schedule. It is the step most often treated as "and then we copy the data over," and the step most likely to cause the outage everyone was trying to avoid.

What makes it different:

- **It runs on the application owners' calendars.** Each cut-over needs a window the business agrees to, and those windows are scarce near peak trading or seasonal periods.
- **It goes in waves.** Start with low-risk systems to prove the method, then move up. The first wave is a test of the procedure as much as a move.
- **Every cut-over has a rollback.** A rollback is the written, rehearsed way back to the old storage, with a decision point and a person who calls it. Keep the old storage intact and untouched until the new one has run in production long enough to trust.
- **Validation is defined before the move.** NIST SP 800-34 Rev. 1 gives a good example of what validation looks like for a recovered database: "compare a database audit log to the recovered database to make sure all transactions were properly updated." Agree the equivalent check for each system before the window opens.

> **Template B: Cut-over and rollback plan** (one per migration window)
>
> | Field | Entry |
> |:---|:---|
> | Systems in this window | |
> | Window start / end (with business owner sign-off) | |
> | Cut-over lead (one person) | |
> | Rollback decision owner (one person) | |
> | Pre-checks: backup verified, replication in sync, change approved | |
> | Cut-over steps with owner and expected duration for each | |
> | **Go / no-go point:** time, and the criteria that must be true | |
> | **Rollback trigger:** what result, or what clock time, forces rollback | |
> | Rollback steps with owner and expected duration for each | |
> | Validation checks after cut-over, and who signs each | |
> | Old storage retained until (date), then released by (name) | |
> | Communications: who is told at start, at go / no-go, at close | |
>
> **Rule:** the rollback steps must fit inside the window with the cut-over steps. If they do not, the window is too short.

## Step 7: Acceptance testing before production

**New equipment is not ready because it powered on.** Acceptance testing is the step between "installed" and "carrying production," and it is where you find the failed component, the miscabled path or the half-configured redundancy while it is still cheap.

### Weak vs better

**Weak acceptance:** "Array installed and passing vendor health check."

**Better acceptance:** "Array accepted when: (1) it has run under a test load for an agreed burn-in period with no hardware faults; (2) each redundant component has been failed on purpose (one power feed pulled, one controller failed, one network path disconnected) with no loss of service; (3) performance under the test load meets the specified baseline; (4) monitoring and alerting fire for each induced failure; (5) the asset is in inventory and the support contract shows it as covered. Accepted by one named storage lead."

The better version tests the redundancy you paid for. Two power supplies prove nothing until one of them has been unplugged while the system kept running. Write acceptance criteria into the hardware contract where you can, so the vendor's install service is not finished until they are met. The [vendor and statement of work (SOW) guide](https://github.com/ChefPlex/tpm-templates/blob/main/vendor-and-sow-management.md) covers how to make acceptance a defined act by a named person.

## Step 8: DR is only real once a failover test passes

**A DR site that has never been failed over to is a hypothesis with a lease.** Replication dashboards showing green tell you data is copying. They do not tell you that applications start, that users can reach them, or that the people on call know the procedure.

NIST SP 800-34 Rev. 1 is explicit about what a full test includes: "The full-scale functional exercise should include a system failover to the alternate location." It also separates three activities that programs tend to merge: "Testing validates recovery capabilities, whereas training prepares recovery personnel for plan activation and exercising the plan identifies planning gaps." And SP 800-53 Rev. 5 explains why testing at the DR site itself matters (CP-4(2)): "Conditions at the alternate processing site may be significantly different than the conditions at the primary site."

What a real DR test covers:

- **Failover** of the in-scope systems to the DR site, under a declared scenario.
- **Measured time** against the RTO and **measured data loss** against the RPO. Measured, with a clock, not estimated afterwards.
- **User access** from where users actually are, including name resolution and network paths.
- **Failback.** SP 800-34 lists "Restoration of normal operations" among what a test covers. Coming home is its own procedure, and it is usually the less rehearsed half.
- **A written result.** SP 800-34: "Results of the test are documented in an After Action Report, and Lessons Learned are developed for updating information in the ISCP." (ISCP is the Information System Contingency Plan.)

Plan for the case where coming home is not possible. SP 800-53 Rev. 5 CP-7(6) asks organizations to "Plan and prepare for circumstances that preclude returning to the primary processing site." If DR cannot run production for months, that is a finding, not a footnote.

> **Template C: DR test record** (one per test)
>
> | Field | Entry |
> |:---|:---|
> | Test date, type (tabletop / partial / full failover) | |
> | Scenario declared | |
> | Systems in scope, with their RTO and RPO | |
> | Test lead / business observer | |
> | Failover start time / systems available time / **measured recovery time** | |
> | Last replicated transaction before failover / **measured data loss** | |
> | Met RTO? Met RPO? (per system, Y/N) | |
> | User access verified from (locations) | |
> | Failback start / complete / issues | |
> | Failures found, each with an owner and a due date | |
> | Plan updates required | |
> | Next test date | |
> | **Result: PASS / PASS WITH FINDINGS / FAIL, signed by** | |
>
> **Rule:** DR is reported as available only for systems with a PASS on a full failover test inside the last test cycle. Everything else is reported as "replicated, not proven."

## Step 9: Colocation moves are logistics programs

**Moving equipment between facilities is closer to running a house move with a fixed-date lease than to an engineering project.** The engineering is mostly done. What remains is inventory, sequencing, shipping, access and people, and each one can stop the move on the day.

### Inventory first, and reconcile it physically

You cannot sequence what you have not counted. NIST SP 800-53 Rev. 5 CM-8 (configuration management family) asks for an inventory that "Accurately reflects the system" and "Includes all components within the system." For a move, that means a walk of every rack with the asset list in hand, checking serial numbers and cable connections against the record. **The inventory in the tool and the inventory in the rack are never the same**, and the differences are the devices nobody planned for.

### The rest of the move plan

- **Sequencing.** Group equipment by the applications it supports, not by rack. Move whole application stacks together, or arrange temporary links so split stacks still work.
- **Live databases and storage.** Where possible, replicate to the destination and cut over (Step 6) rather than shipping the only copy in a truck. If equipment must physically move with data on it, have a verified backup that stays behind.
- **Shipping.** Specialist movers, insurance that matches the value, packing for heavy and fragile equipment, and a named person who rides with or tracks the load.
- **Remote hands.** Facility staff who will rack, cable or power-cycle equipment for you. Agree the scope, rates and response times in advance, and give them written, labeled instructions.
- **The contract's access rules.** Who is on the access list, how far ahead visits must be booked, loading dock hours, and what paperwork equipment needs to leave. SP 800-53 PE-16 requires organizations to control components entering and leaving a facility and to "Maintain records of the system components." Expect both facilities to enforce their own version of this, and book it early.
- **The lease clock.** The old contract's end date is fixed. Work back from it, with time for the move to slip at least once.

## Step 10: Fold a new application or web property into both sites

**Adding a new system, or an acquired web property, to existing data centers is a small build-out with the same failure modes.** The common mistake is to land it in primary and treat DR as a follow-up ticket that never closes.

Before it goes live, settle:

- **Its RTO and RPO** (Step 1), agreed by its business owner.
- **Where it runs and where it fails over to,** with capacity reserved in both sites (Step 4).
- **Network and name resolution** at both sites, including how traffic moves to DR when it has to.
- **Replication and backup** set up and verified before the first production data lands.
- **A failover test** (Step 8), included in the onboarding, not deferred.
- **Ownership,** recorded: who is called when it fails at 2 a.m., at each site.

**The definition of done for onboarding is a passed failover test**, the same as for the build-out itself.

## Step 11: Manage the contracts that hold the sites up

**Colocation and hardware contracts decide what you can do on the day something goes wrong.** Read them for operations, not only for price. The [vendor and SOW guide](https://github.com/ChefPlex/tpm-templates/blob/main/vendor-and-sow-management.md) covers the general method; these are the clauses specific to data centers.

| Contract | Clauses a TPM should read |
|:---|:---|
| **Colocation** | Power commitment per rack and the charge for exceeding it · cross-connect fees and lead times · remote hands scope, rates and response times · access list rules and notice periods · service levels for power and cooling, and what remedy applies · expansion rights for adjacent space · exit terms and decommission obligations |
| **Hardware** | Committed delivery date and remedy for a miss · installation services and their acceptance criteria (Step 7) · when the support term starts (ship date or install date) · parts replacement response times at each site · end-of-support dates for the models you are buying |
| **Carrier** | Provisioning date commitment · service levels for the inter-site link · diverse routing, if the DR design depends on it |

**Check support start dates carefully.** If support starts at shipment and installation is delayed by power or circuits, you are paying for coverage on equipment sitting in a crate.

## Step 12: Decommission the old equipment and close the old contracts

**The program is not finished when the new sites are live. It is finished when the old equipment is gone, the data on it is unrecoverable and the old contracts are closed.**

NIST SP 800-88 Rev. 2 covers media sanitization, which it describes this way: "Media sanitization refers to a process that renders access to target data on the media infeasible for a given level of effort" ([NIST CSRC](https://csrc.nist.gov/pubs/sp/800/88/r2/final), as of October 2026). SP 800-53 Rev. 5 MP-6 (media protection family) requires organizations to "Sanitize [Assignment: organization-defined system media] prior to disposal, release out of organizational control, or release for reuse." Plan the sanitization method with your security team, keep a record per device, and do not let old arrays leave a facility on a trade-in until that record exists.

Then close the paper: cancel circuits, end support on retired hardware, return space, and update the inventory. Retired equipment that is still billed is a common and quiet cost.

---

## Where this breaks

- **The recovery targets were never agreed.** If the business never signed off on RTO and RPO, every DR design is defensible and none is accountable. The program ends with a second site and no way to say whether it is good enough.
- **Long-lead items are treated as tasks.** A plan that lists "order hardware" as a two-day task, rather than as a dependency with a vendor's ship date, will look on track until the week the equipment does not arrive.
- **The DR test is scoped down until it passes.** Testing only the systems that are easy to fail over, or failing over without failing back, produces a PASS that means nothing. Report what was not tested in the same line as what was.
- **Replication status is reported as DR readiness.** Green replication is a necessary condition, not proof. Use the Template C rule.
- **Migration rollback exists only on paper.** A rollback that was never rehearsed, or that does not fit in the window, is not a rollback. It is a hope with a heading.
- **The paper plan is not kept current.** Rack elevations that are not updated after install become the wrong map, and the next person to add equipment trusts it.
- **Capacity planning uses a growth number nobody can trace.** If the growth rate came from a guess, say so. A precise-looking specification built on a guess is worse than a rough one that is labeled as rough.
- **This method is heavy for small changes.** Adding a few servers to an existing rack with known power does not need twelve steps. Use the templates where the risk is, and skip them where it is not.

---

## Worked example: Bramblewood Home Goods (fictional)

*Bramblewood Home Goods is an invented company. All names and numbers below are made up and rounded for illustration.*

**The situation.** Bramblewood, a mid-sized retailer, runs its systems from one colocation cage with an aging storage array and a "DR site" that is a second, smaller cage across town holding nightly backups. Leadership approves a program to modernize storage and fabric at both sites, move the DR cage to a facility in a different region, and bring a recently acquired online store into both.

**Step 1: targets.** The TPM runs sessions with business owners for 30 systems. Order processing and the online store get a 1-hour RTO and a 5-minute RPO. Finance reporting gets 24 hours and 24 hours. Twelve internal tools get 72 hours and 24 hours. The existing "DR" cage, holding nightly backups, could never have met a 5-minute RPO, and the gap is written into the business case.

**Step 2: long-lead.** The carrier quotes 90 days for the inter-region link, and the storage vendor quotes 60 days to ship. Both are ordered in week 2, before the detailed design is final. The new DR facility needs 45 days to deliver a powered cage. The carrier link becomes the critical path and is logged as the program's top dependency.

**Step 3 and 4: paper plan and growth.** The team draws 8 racks per site. Storage is sized to month 36 on a 30 percent annual growth trend, which puts the arrays at about twice today's usage. The paper plan shows that the primary cage's existing circuits support only 6 of the 8 racks at the planned draw. The extra power is ordered in week 4, three months before it would have been discovered on install day.

**Step 5: the pair.** Both sites get the same array model, firmware and naming. Synchronous replication cannot be supported over the inter-region distance, so the team designs asynchronous replication and measures in testing that it holds a 5-minute RPO at expected load. The colocation contract for the DR site includes remote hands with a 1-hour response, because the on-call team is a 4-hour drive away (CP-7(2)).

**Step 6: migration.** Thirty systems move to the new primary array in five waves over ten weekends. Wave 1 is the twelve internal tools. In wave 2, validation finds a reporting job pointed at a hard-coded path on the old array. The team rolls back inside the window, fixes the path, and moves the system the following weekend. The old array stays live, read-only, for 30 days after the last wave.

**Step 7: acceptance.** During burn-in, pulling Feed B on one DR rack drops a switch that was plugged into Feed A twice. It is re-cabled and the test is repeated before acceptance is signed.

**Step 8: DR test.** The first full failover test runs in month 7. Order processing recovers in 50 minutes with about 3 minutes of data loss: PASS. The online store fails over but customers cannot reach it, because its name records point only at primary: FAIL, with an owner and a 30-day fix. Failback takes six hours, longer than planned, and the runbook is rewritten. The steering committee report shows 28 of 30 systems as "DR proven" and 2 as "replicated, not proven" until the retest passes in month 8.

**Step 9: the move.** The old DR cage holds 40 devices according to the inventory. The physical walk finds 46. Four of the extra six are retired and two support a finance system nobody had listed. The finance system is added to wave 4. The move finishes three weeks before the old lease ends.

**Step 10 to 12: onboarding and close.** The acquired online store is onboarded at both sites, and its done criterion is the month 8 retest. The old arrays are sanitized, with a certificate per device, before trade-in. Two old circuits still billing after the move are cancelled at close.

**What made it work:** the recovery targets came first, the carrier link was ordered before the design was finished, the power shortfall was found on paper, and DR was reported as proven only system by system.

---

## Sources

All accessed as of October 2026.

- NIST SP 800-34 Rev. 1, *Contingency Planning Guide for Federal Information Systems*: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-34r1.pdf
- NIST SP 800-53 Rev. 5, *Security and Privacy Controls for Information Systems and Organizations* (CP, PE, CM and MP control families): https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r5.pdf
- NIST SP 800-88 Rev. 2, *Guidelines for Media Sanitization*: https://csrc.nist.gov/pubs/sp/800/88/r2/final
- Uptime Institute, Tier Classification System (public page; the full Tier standard is a paid document): https://uptimeinstitute.com/tiers
- TIA, ANSI/TIA-942 standard (public page; the standard itself is a paid document): https://tiaonline.org/products-and-services/tia942certification/ansi-tia-942-standard/
- ASHRAE datacom resources (public page; the guidance itself is sold or licensed): https://www.ashrae.org/technical-resources/bookstore/datacom-series

These are federal guidelines and industry standards. Private organizations are not bound by the NIST publications; they are cited here as a tested, public model for what recovery planning should cover.

## Related

- [Vendor and SOW Management](https://github.com/ChefPlex/tpm-templates/blob/main/vendor-and-sow-management.md): writing hardware and colocation statements of work that can be accepted or rejected
- [The RAID Log](https://github.com/ChefPlex/tpm-templates/blob/main/raid-log-guide.md): tracking long-lead items as dependencies with likelihood and impact
- [Program Phases Playbook](https://github.com/ChefPlex/tpm-templates/blob/main/program-phases-playbook.md): where a build-out sits in the program lifecycle
- [Steering Committee Deck Structure](https://github.com/ChefPlex/program-reporting-frameworks/blob/main/steering-committee-deck-structure.md): reporting DR readiness and critical-path risk upward
- [Cyber Incident Playbook](https://github.com/ChefPlex/tpm-templates/blob/main/incident-management/cyber-incident-playbook.md): when a real event forces the failover you tested
- [End-of-Support Migration Playbook](end-of-support-migration-playbook.md): moving the server estate that runs on this infrastructure
- [Directory Services Upgrade Playbook](directory-services-upgrade-playbook.md): the authentication dependency every failover test touches

---

*Part of the [ChefPlex TPM portfolio](https://github.com/ChefPlex/tpm-templates/blob/main/PORTFOLIO-GUIDE.md). Maintained by [Eric White](https://www.linkedin.com/in/edwhite). Licensed under [CC BY 4.0](LICENSE).*
