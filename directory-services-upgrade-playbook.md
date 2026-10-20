# Directory Services Upgrade and Consolidation Playbook

**How to upgrade and consolidate Active Directory (AD) and LDAP (Lightweight Directory Access
Protocol) across several domains without breaking authentication: what to find first, what to
decide on purpose, and how to make a quiet delivery happen.**

A directory program has a strange success criterion. If it goes well, nobody notices. Every login,
every application that checks a password, every printer that looks up an email address keeps
working, and the only evidence the program happened is that the old domain controllers are gone
before the vendor's deadline. That outcome is not luck. It is made, mostly in the first third of the
program, by finding every dependency before anything is touched.

These are patterns drawn from several programs of very different sizes. The specifics of any one
estate will not generalize. The shape will.

---

## The short version

1. **This is a coexistence program.** You are retiring one generation of domain controllers (DCs)
   while another generation, out of scope, keeps running undisturbed. That constraint is the
   program, not a footnote to it.
2. **The functional level is a decision, not a milestone.** It is capped by the oldest DC you keep,
   it is generally irreversible, and the right answer is often one level lower than the highest
   available. Write the decision down.
3. **The real risk is the LDAP binders.** Applications, appliances and scripts that authenticate
   straight against a DC, often by hard-coded name or address, with a service account nobody owns.
   Find them before you retire anything.
4. **Build a test environment before production is touched,** and make it reproduce the binders,
   not just the domain.
5. **Every DC added or retired is a change** with a written method of procedure (MOP), done after
   hours, under formal change control.
6. **Service account sprawl will surface.** Record it. It is where access governance starts, and
   the directory program is the cheapest moment to see it.

---

## Step 1: Write the scope as a coexistence statement

**The first document is not a project plan. It is a statement of what changes and what must not.**
A directory upgrade touches the one system every other system depends on, and the part of the
estate you are *not* changing is usually the part that can hurt you.

A typical mixed estate has at least three generations in it:

| Generation | What happens to it | Why it matters |
|:---|:---|:---|
| **Retiring** | Replaced and demoted | The vendor deadline is about these |
| **Kept, out of scope** | Left running, untouched | Caps your functional level and limits what you can change at forest scope |
| **New** | Promoted into the existing domains | Has its own minimum functional level, so it can force decisions too |

The kept generation is out of scope for *work* but in scope for *risk*. A forest schema update, a
DNS (Domain Name System) change or a Group Policy change made for the new DCs reaches the old ones
too. Microsoft's upgrade guidance says the schema preparation step runs "only once in the forest for
each newer version of Windows Server", which is another way of saying it touches everything in the
forest, kept generation included.

**Weak scope statement:**

> Upgrade all domain controllers to Windows Server 2022 and raise the functional level.

**Better scope statement:**

> Retire the 18 domain controllers running the retiring release across the forest before the
> vendor end-of-support date. Promote 10 replacements on the new release. The 6 domain controllers
> at the distribution sites are out of scope and must keep authenticating without change. Collapse
> the three departmental domains into their parent domains. The forest functional level target is a
> separate, recorded decision (see Step 3).

The better version names the boundary, gives the reader numbers to check against, and refuses to
smuggle the functional level in as a task.

> **Template: coexistence scope statement**
>
> - **Retiring:** [count] DCs on [release], in domains [list], by [date, and who set it]
> - **New:** [count] DCs on [release], in domains [list]
> - **Kept, out of scope:** [count] DCs on [release], owned by [team], must not be disturbed because [reason]
> - **Domains collapsing:** [source domain] into [target domain], for each
> - **Explicitly not in this program:** [for example, merging the two DNS namespaces, LDAP signing enforcement]
> - **Functional level:** decided in [decision record link], not assumed here

---

## Step 2: Inventory the controllers by generation, then the roles they hold

**Count the DCs, then find out what each one actually does.** A DC is rarely just a DC. It may hold
one of the Flexible Single Master Operations (FSMO) roles, be a global catalog, be the DNS server
half the network points at, or be the address some appliance was configured with years ago.

Microsoft's upgrade guidance makes FSMO connectivity a prerequisite: the first new DC needs the
schema master and the infrastructure master reachable, and in Microsoft's words, "Any DC
installation also requires connectivity to the RID master." So the inventory must say where the
roles live before the first change is scheduled.

> **Template: domain controller inventory record**
>
> | Field | Value |
> |:---|:---|
> | DC name | |
> | Domain and DNS namespace | |
> | Site | |
> | Operating system release | |
> | Generation | Retiring / Kept / New |
> | FSMO roles held | |
> | Global catalog | Yes / No |
> | DNS server for | [subnets, DHCP scopes, other servers' resolver settings] |
> | Known clients configured by name or IP | [from Step 4] |
> | SYSVOL replication engine | DFSR / FRS |
> | Change ticket for retirement or promotion | |

**Check the SYSVOL replication engine early.** Microsoft states "Windows Server 2016 is the last
Windows Server release that supports the File Replication Service (FRS)." If an old domain still
replicates SYSVOL with FRS, a DC on any later release cannot replicate it, so the domain has to move
to DFSR (Distributed File System Replication) first. The same page says domains at the 2016 level
"must use DFSR as the engine to replicate SYSVOL." That migration is its own small project with its
own change window.

---

## Step 3: Decide the functional level, and write down why

**The functional level is the setting that tells the DCs which features they can use together.** It
is easy to treat as the finish line (*raise to the latest level*). Treat it instead as a design
decision with a cost on each side.

### What the sources actually say

| Fact | Source wording |
|:---|:---|
| The default advice is to go as high as you can | "set the domain and forest functional levels to the highest value that your environment can support" |
| It is capped by your oldest DC | "Do not raise the functional level if the domain has or will have a domain controller that is of an earlier version than the version that is cited for that level." |
| It is generally one-way | "Generally, the changes to the domain and forest functional levels are irreversible." |
| It does not change what applications see | "Additionally, applications are unaffected by changes to the domain functional levels or to the forest functional levels." |
| New DCs set a floor too | "Windows Server 2019 or later requires a Windows Server 2008 forest functional level as a minimum." |

Microsoft's current interoperability matrix (as of October 2026) lists three levels and which DC
releases can run at each:

| DC release | 2025 level | 2016 level | 2012 R2 level |
|:---|:---|:---|:---|
| Windows Server 2025 | Supported | Supported | Not supported |
| Windows Server 2022 | Not supported | Supported | Supported |
| Windows Server 2019 | Not supported | Supported | Supported |
| Windows Server 2016 | Not supported | Supported | Supported |
| Windows Server 2012 R2 | Not supported | Not supported | Supported |

Two consequences for a coexistence program:

- **Your environment includes the DCs you are keeping.** Microsoft's "highest value that your
  environment can support" is the right rule, read honestly. If the kept generation runs an older
  release, the ceiling is that release's level, and the only way past it is to pull the kept
  generation into scope.
- **The squeeze runs both ways.** The newest release in the matrix cannot run at the oldest level:
  a Windows Server 2025 DC needs at least the 2016 level. Choosing the release for the new DCs and
  choosing the functional level are one decision, not two.

### Why one level lower is often the right answer

The first jump usually pays for itself: it gets the forest onto a level the vendor still lists,
retires the oldest constraints and is reachable without touching anything out of scope. The next
step often is not. It means retiring the kept generation, migrating whatever runs on it and
re-testing every binder, in exchange for features the organization may not use.

**So the useful question is not *how high can we go*. It is *what does the next level buy, and what
does it cost to get there*.** If the answer is a feature list nobody has asked for and a second
program's worth of disruption, stop one level short and record that you did.

⚠️ **Microsoft's own pages disagree on this point,** which is a reason to cite the right one. The
upgrade guide says, "No new forest or domain functional levels have been added since Windows Server
2016." The functional levels page, meanwhile, lists a Windows Server 2025 level. Use the
functional levels page as the authority and date your citation.

> **Template: functional level decision record** (use the [ADR template](https://github.com/ChefPlex/tpm-templates/blob/main/adr-template.md))
>
> - **Current forest and domain levels:** [level], [level]
> - **Highest level the kept generation allows:** [level], because [oldest kept DC release]
> - **Level chosen:** [level]
> - **What the next level up would buy:** [features, with who asked for them]
> - **What it would cost:** [DCs pulled into scope, applications re-tested, months]
> - **Why we stopped here:** [one or two sentences]
> - **When to revisit:** [the event that changes the answer, for example the kept generation's own end of support, or the first DC on a release that needs a higher level]
> - **Who approved:** [name, date]
> - **Reversibility:** generally irreversible once raised; rollback only by forest recovery or the narrow exceptions Microsoft documents

---

## Step 4: Find the LDAP binders

**This is the program.** Everything else is careful sequencing. The thing that breaks authentication
during a DC retirement is almost never the directory itself. It is something outside the directory
that talks to one specific DC and nobody wrote down.

### Who binds, and why they break

- **Applications** that check a password against the directory with an LDAP bind instead of using
  the platform's single sign-on. Often configured once, with a DC hostname or IP address in a
  config file, and never touched again.
- **Appliances and devices.** Printers and scanners looking up email addresses, badge and door
  systems, network gear authenticating administrators, phone systems. They have a vendor, a
  configuration screen and frequently no owner inside the organization.
- **Scripts and scheduled jobs** that run monthly or quarterly, so they do not show up in a short
  observation window.
- **Hard-coded service accounts** behind all of the above, often with passwords set to never
  expire, because changing them would mean finding every place the password was typed.

Retiring a DC breaks a binder in two ways: the address it was pointed at disappears, or the new DCs
are configured more strictly than the old ones and reject the way it binds.

### Use the directory's own logs to find them

Microsoft documents events for exactly this. Event 2887 is a daily summary count of insecure binds.
Event 2889, once diagnostic logging is raised, records each one: "The log entry displays the IP
address of the client and the identity that the client tried to use to authenticate." That gives you
a list of addresses and accounts to chase.

**Know the limit of that list.** Microsoft's KB4520412 describes the trigger for event 2889 as:
"Triggered when a client does not use signing for binds on sessions on port 389." Clients that
bind correctly, or over LDAPS (LDAP over TLS, Transport Layer Security, on port 636), do not appear. They can still be pointed at a
DC you are about to retire. **Event 2889 finds the insecure binders. It does not find all of them.**
Complete the list with DNS query logs, firewall or flow logs to the DCs and a search of
configuration repositories for DC names and addresses.

### Why the insecure ones matter beyond this program

The LDAP standard is plain about simple binds without encryption. RFC 4513 says: "LDAP
implementations SHOULD NOT by default support authentication methods using clear text passwords and
other unprotected authentication credentials unless the data on the session is protected using TLS
or other data confidentiality and data integrity protection."

It also names a quieter trap. A simple bind with an empty password is an anonymous bind, and the RFC
warns that "users intending to perform Name/Password Authentication may inadvertently provide an
empty password and thus cause poorly implemented clients to request Unauthenticated access." **A
device that appears to authenticate may be binding anonymously.** It works until a DC stops allowing
anonymous access, and then it looks like the upgrade broke it.

**Weak action item:**

> Check which apps use LDAP.

**Better action item:**

> Owner: directory lead. By [date], produce the LDAP consumer list from 30 days of event 2889 on
> every DC, plus DNS and flow logs for clients that bind securely. Each entry gets an owner, the DC
> it points at, and a remediation before its DC enters a change window. Unowned entries go to the
> program manager weekly.

> **Template: LDAP consumer record**
>
> | Field | Value |
> |:---|:---|
> | Consumer | [application, device type, script] |
> | Found by | Event 2889 / DNS logs / flow logs / config search / owner told us |
> | Client address(es) | |
> | Account used to bind | |
> | Bind type | Signed / unsigned SASL / simple over 389 / simple over LDAPS / anonymous |
> | Points at | Specific DC name or IP / domain name / load-balanced name |
> | Business owner | [a person, or none found] |
> | Technical contact | |
> | Usage pattern | Continuous / daily / month-end / quarterly / annual |
> | Remediation | Repoint to domain name / move to signed or LDAPS bind / replace account / retire consumer |
> | Must be done before | [DC retirement change ticket] |
> | Verified after change | [date, by whom, how] |

---

## Step 5: Record the service accounts you trip over

**Every binder has an account behind it, and many of those accounts are where the real exposure
is:** stale, set to never expire, over-privileged and owned by nobody. This is non-human identity
discovery, done at program scale because the program forced it.

Do not let it take over the directory program. Do write it down. Microsoft's guidance on
on-premises service accounts puts it this way: "Finding on-premises service accounts is key to
ensuring their security. Doing so can be difficult for non-MSA accounts." (MSA is a managed service
account.) It recommends documenting an owner, a purpose, permission scopes, a risk profile, an
expected lifetime with periodic attestation, and where the password is stored. It also sets a
standard many older estates are far from: "Each service should have its own service account for
auditing and security purposes."

For each account found, record those fields and hand the list to whoever owns access governance.
Where a service can use a group managed service account (gMSA), Microsoft recommends it. **The
directory program finds the accounts. Fixing them is usually a follow-on.**

---

## Step 6: Consolidate domains in the order that keeps access working

Consolidation is where two halves of an organization, often on separate DNS namespaces, and a few
departmental domains that grew up on their own, are brought into fewer domains. It is the part of
the program with the most moving pieces and the least tolerance for a missed step.

### The sequence that holds

1. **Establish coexistence first.** Name resolution in both directions between the namespaces, and
   the trust relationships the migration needs, working and tested before any account moves.
2. **Inventory what each departmental domain owns:** users, groups, computers, service accounts,
   Group Policy, file shares and the binders from Step 4.
3. **Move in waves,** by department, not by object type. A department moved together can be tested
   together.
4. **Keep access working through the move.** A moved account gets a new identifier. Microsoft's
   definition of the SID (security identifier) history attribute: "Whenever an object is moved from
   one domain to another, a new SID is created and that new SID becomes the objectSID." And: "The
   previous SID is added to the sIDHistory property." Every file share, group and application that
   granted access to the old SID is now pointing at an identifier the account no longer carries as
   its primary one. **Decide, before the first wave, how those permissions keep working during the
   transition**: confirm in your migration tool's own documentation whether it carries SID history
   and what that requires of the trust. Then plan when that history gets cleaned up and the
   permissions re-granted to the new SID, because an old SID left in place is a standing exposure.
5. **Decommission the departmental domain** only when nothing has authenticated against it for an
   agreed period, and its DCs are retired the same way as any other (Step 8).

⚠️ **Do not combine a namespace merge with a DC retirement unless you must.** Each is a sound
program. Together they double the number of things that can be wrong in one change window, and
when something breaks, nobody can say which half caused it.

---

## Step 7: Build the test environment before production is touched

**A directory test environment is worth building even when it feels like overhead,** because the
production directory has no undo button. Functional level changes are generally irreversible, and
a forced demotion has consequences that last (Step 8).

What it has to reproduce:

- **The mix of releases,** including the kept generation, so the functional level decision and the
  new DC promotion are rehearsed against the real constraint.
- **The domain and namespace layout,** including trusts and name resolution between the halves.
- **A sample of real binders.** At least one of each type from the consumer list: an application,
  an appliance or its emulator, a scheduled script. A test domain with no binders proves the
  directory works, which nobody doubted.
- **The MOP itself.** Every step of the per-DC procedure in Step 8 is run in the test environment
  first, and the timings go into the change plan.

---

## Step 8: Run every DC change under change control, with a written MOP

**Each DC promoted or retired is a separate change,** reviewed, scheduled after hours and executed
from a written method of procedure. It is slow on purpose. A program of thirty quiet changes is
much easier to defend than one of three dramatic ones.

Microsoft's recommended path suits this: "The recommended way to upgrade a domain is to use a clean
OS install to promote new servers to DCs that run a newer version of Windows Server and demote the
older DCs as needed." Add first, move the work over, then retire. Raise the functional level last,
after the retiring DCs are gone, which is the order Microsoft's guide follows: "After the server is
demoted and removed, you can raise the forest functional and domain functional levels to the latest
version of Windows Server."

**Never force a demotion to save a window.** Microsoft's warning is direct: "Forced demotion leaves
orphaned metadata in Active Directory on the remaining domain controllers in the forest." The same
page says the cleanup must then be done manually and immediately. If a DC cannot demote cleanly, stop and find out why.

> **Template: per-DC cut-over checklist (retirement)**
>
> **Before the window**
> - [ ] Change approved; MOP rehearsed in the test environment; timings recorded
> - [ ] Replacement DC promoted, replicating cleanly, and serving DNS
> - [ ] FSMO roles (if any) moved and confirmed with `Get-ADDomain` and `Get-ADForest`
> - [ ] Every LDAP consumer pointing at this DC remediated and verified (Step 4 records closed)
> - [ ] DHCP scopes, static resolver settings and appliance configs no longer reference this DC
> - [ ] Event 2889 and DNS logs show no remaining clients for [agreed period]
> - [ ] Backup of the system state taken; rollback steps written
> - [ ] Application and device owners notified of the window ([communications plan](https://github.com/ChefPlex/tpm-templates/blob/main/communications-plan-template.md))
>
> **In the window**
> - [ ] Demote cleanly (no forced removal)
> - [ ] Confirm replication health across remaining DCs
> - [ ] Run the smoke tests: user sign-in at each affected site, one test per binder type
>
> **After the window**
> - [ ] Watch authentication failures and helpdesk volume for [24 to 72 hours]
> - [ ] Remove DNS records and monitoring for the retired DC
> - [ ] Close the change with what was found, including anything that surprised you

---

## Step 9: Measure it, close it, and name what comes next

**Measure observed state, not reported state.** The useful numbers:

| Measure | Where it comes from | What done looks like |
|:---|:---|:---|
| Retiring DCs remaining | DC inventory | Zero, before the vendor date |
| LDAP consumers without a remediation | Consumer list | Zero for each DC before its window |
| Insecure binds per day | Event 2887 summaries | Falling, with every remaining source named |
| Authentication incidents caused by the program | Incident and helpdesk records | The headline number; ideally zero |
| Departmental domains remaining | Domain inventory | Zero |

Use the [RAID log](https://github.com/ChefPlex/tpm-templates/blob/main/raid-log-guide.md) for the
risks and dependencies. The unowned binders and the kept-generation constraint belong in it from
week one.

### Close with the follow-ons stated, not implied

The program usually ends with the directory current and quieter, and with two pieces of work it
uncovered but did not own:

- **LDAP signing and channel binding enforcement.** Microsoft's KB4520412 says the March 2020
  updates, and updates in the foreseeable future, "will not change LDAP signing or LDAP channel
  binding default policies or their registry equivalent on new or existing Active Directory domain
  controllers." Nobody enforces it for you. The consumer list from Step 4 is most of the
  preparation, and Microsoft's own sequencing is to fix the clients first: "After no such events are
  observed for an extended period, we recommend that you configure the server to reject such binds."
- **Service account governance,** from the Step 5 records.

**Hand both over by name, with the evidence.** A clean delivery is a weak argument for funding the
next program unless the uncovered work is written down where a decision-maker will see it.

---

## Worked example: Calder Valley Freight (fictional)

**Calder Valley Freight** is an invented logistics company. All names and numbers are made up and
rounded.

**The estate.** One forest, two halves on separate DNS namespaces: the corporate domain on
`calder.example` and the operations domain on `calderlogistics.example`. Three departmental domains
(finance, engineering and a legacy acquisition) sit under them. Thirty DCs in total.

**The deadline.** Eighteen DCs run a release whose vendor support ends in nine months (Windows
Server 2012 R2 in this example, kept alive on paid Extended Security Updates that end
October 13, 2026). That date is fixed and set outside the company.

**The constraint.** Six DCs at the distribution sites run a newer release that is still supported
but older than the newest (Windows Server 2019 in this example). They support a warehouse system owned by another team,
and that team has a freeze through peak season. They are out of scope and must not be disturbed.

**Scope.** Retire 18, promote 10 on Windows Server 2022, keep 6, collapse the three departmental
domains into their parents. Merging the two namespaces was raised and explicitly deferred.

**The functional level decision.** The original plan said *raise to the 2025 level*, the highest
on the matrix. Only Windows Server 2025 DCs can run at 2025, so reaching it would mean building the
new DCs on 2025 and replacing the kept distribution-site DCs against the other team's freeze. The
18 retiring DCs held the forest at the 2012 R2 level. Once they were gone, the 2016 level was
reachable without touching the distribution sites, and it is the floor a Windows Server 2025 DC
needs to join. Nobody could name a
2025-level feature the business needed. The program recorded 2016 as the target, with the revisit
trigger written as: *when the distribution-site DCs are replaced, or reach their own end of
support*. The choice of Windows Server 2022 for the new DCs came out of the same decision: at the
2016 level, 2022 and 2025 both work, and the team already had a hardened 2022 build.

**The binders.** Thirty days of event 2889 across all DCs produced about 140 distinct client
addresses. Roughly 40 were applications, 70 were devices (printers, scanners, badge readers, two
phone systems) and 30 could not be identified at first. DNS and flow logs added about 25 clients that
bound securely and so never appeared in 2889. Twelve consumers had a retiring DC's IP address
hard-coded. Four devices turned out to be binding anonymously with an empty password.

**The accounts.** About 300 accounts looked like service accounts. Around 90 had passwords set to
never expire and about 60 had no identifiable owner. They were recorded with owner, purpose and risk
and handed to the security team as a follow-on. Twelve that blocked a DC retirement were fixed
inside the program.

**Execution.** A test environment with both namespaces, a 2019 DC, a sample printer, two
applications and the month-end job was built in the first six weeks. Every promotion and retirement
was a separate after-hours change with a rehearsed MOP. SYSVOL in the acquisition domain was still on
FRS and was migrated to DFSR first, in its own window. One DC failed to demote cleanly; the window
was abandoned, the cause (a stale replication partner) was fixed and it was retired the following
week rather than forced.

**Result.** All 18 retired three weeks before the vendor date. No authentication outage was traced
to the program. The departmental domains were decommissioned after 60 days with no sign-ins. The
close-out named LDAP signing enforcement and service account governance as funded follow-ons, with
the consumer list and account register attached.

---

## Where this breaks

- **The observation window is too short.** Thirty days of logs misses the quarterly report and the
  annual audit job. Ask owners about calendar-driven work, and keep logging on after the inventory
  closes.
- **Event 2889 is treated as the complete list.** It shows insecure binds on port 389 only. Signed
  and LDAPS binders pointed at a specific DC break just as hard.
- **The kept generation gets touched anyway.** A forest schema update, a Group Policy change or a
  DNS change aimed at the new DCs reaches the old ones. Test against the kept release, and tell its
  owner before every forest-wide change.
- **Someone raises the functional level early.** It is a single command and generally irreversible.
  Restrict who can do it and make it a scheduled change tied to the decision record.
- **The test environment has no binders.** A clean test of the directory proves little. The breakage
  lives in the consumers.
- **Ownerless devices stall the plan.** A device nobody owns cannot be remediated by anyone with
  authority. Escalate these weekly; the decision to replace, repoint or retire them is a management
  call, and it is on the critical path.
- **A window is saved by forcing a demotion.** The orphaned metadata costs more than the window.
- **The clean delivery hides the work.** When nothing breaks, the people who fund the next program
  may conclude it was easy. The close-out has to show what was found and fixed.

---

## Sources

All accessed and current as of October 2026.

- Microsoft Learn, [Active Directory Domain Services functional levels](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/active-directory-functional-levels): interoperability matrix, recommendation to use the highest supportable level, FRS and DFSR notes.
- Microsoft Learn, [Upgrade domain controllers to a newer version of Windows Server](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/upgrade-domain-controllers): recommended upgrade method, FSMO prerequisites, minimum forest functional levels for new DCs, adprep, functional level rollback limits.
- Microsoft Learn, [Raise Active Directory domain and forest functional levels](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/raise-active-directory-domain-forest-functional-levels): irreversibility, the caution against raising past the oldest DC, effect on applications.
- Microsoft Learn, [How to enable LDAP signing in Windows Server](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/enable-ldap-signing-in-windows-server): events 2886 to 2889 and the advice on when to reject insecure binds.
- Microsoft Support, [2020, 2023, and 2024 LDAP channel binding and LDAP signing requirements for Windows (KB4520412)](https://support.microsoft.com/topic/2020-2023-and-2024-ldap-channel-binding-and-ldap-signing-requirements-for-windows-kb4520412-ef185fb8-00f7-167d-744c-f299a66fc00a): event 2889 trigger, channel binding settings, default policies unchanged.
- Microsoft Learn, [Demoting domain controllers and domains](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/demoting-domain-controllers-and-domains--level-200-): forced demotion and orphaned metadata.
- Microsoft Learn, [Secure on-premises service accounts](https://learn.microsoft.com/en-us/entra/architecture/service-accounts-on-premises): finding and documenting service accounts, gMSA recommendation.
- Microsoft Learn, [SID-History attribute](https://learn.microsoft.com/en-us/windows/win32/adschema/a-sidhistory): what happens to an object's SID when it moves between domains.
- IETF, [RFC 4513: LDAP Authentication Methods and Security Mechanisms](https://www.rfc-editor.org/rfc/rfc4513): clear text password guidance and unauthenticated simple binds.

---

## Related

- [End-of-Support Migration Playbook](end-of-support-migration-playbook.md): the server and operating system version of the same coexistence problem
- [Encryption Program Playbook](https://github.com/ChefPlex/security-program-playbooks/blob/main/encryption-program-playbook.md): ownership discovery and the concentration trap at estate scale
- [Security Program Intake and Kickoff](https://github.com/ChefPlex/security-program-playbooks/blob/main/security-program-intake-kickoff.md): for the follow-on LDAP hardening and service account work
- [Cryptography Fundamentals for TPMs](https://github.com/ChefPlex/learning-notes/blob/main/cryptography-fundamentals-for-tpms.md): background on TLS, signing and channel binding
- [The RAID Log](https://github.com/ChefPlex/tpm-templates/blob/main/raid-log-guide.md)
- [ADR Template](https://github.com/ChefPlex/tpm-templates/blob/main/adr-template.md): for the functional level decision
- [Communications Plan Template](https://github.com/ChefPlex/tpm-templates/blob/main/communications-plan-template.md): for change window notices to application and device owners

---

*Part of the [ChefPlex TPM portfolio](https://github.com/ChefPlex/tpm-templates/blob/main/PORTFOLIO-GUIDE.md). Maintained by [Eric White](https://www.linkedin.com/in/edwhite). Licensed under [CC BY 4.0](LICENSE).*
