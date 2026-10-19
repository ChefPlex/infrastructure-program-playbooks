# infrastructure-program-playbooks

Playbooks for Technical Program Managers (TPMs) running infrastructure and platform programs: data
center and site build-outs, server and operating system migrations, directory services, and network
rollouts.

Infrastructure programs fail in a different place from software programs. The code is rarely the
problem. The problem is the dependency nobody listed, the system you are keeping that the change
must not disturb, and the vendor deadline you did not set. These playbooks are about that part.

Companion to [security-program-playbooks](https://github.com/ChefPlex/security-program-playbooks).

---

## What Is Here

### Infrastructure

| Playbook | What It Is | Status |
|:---|:---|:---|
| [End-of-Support Migration Playbook](end-of-support-migration-playbook.md) | Moving a server and operating system estate off a version the vendor is retiring, against a date you did not set. Planning backward from the date, the discovery curve and how to report it, four-way triage (decommission, upgrade, replace, extended support), why ownership is the long pole, and putting the cost of waiting beside the program budget | Available |
| [Directory Services Upgrade and Consolidation](directory-services-upgrade-playbook.md) | Upgrading and consolidating Active Directory (AD) and LDAP (Lightweight Directory Access Protocol) across domains without breaking authentication. Scoping it as a coexistence program, treating the functional level as a recorded decision, finding the LDAP binders before anything is retired, and consolidating domains in the order that keeps access working | Available |
| [Data Center and Storage Build-Out](data-center-build-out-playbook.md) | Standing up or modernizing capacity across a primary and a disaster recovery (DR) site. Recovery targets before hardware, long-lead procurement as the critical path, the floor planned on paper, primary and DR built as a pair, consolidation onto fewer and denser machines, a move weekend run on a published clock, and why DR is real only once a failover test passes | Available |
| [Network Rollout Playbook](network-rollout-playbook.md) | Rolling a network change across many sites, from a few campuses to hundreds of stores. Running the program as a sequence of outputs, capacity measured at a real site's size, circuits ordered by tier, a fixed pre-work window per site on a weekly cadence, cut-overs in small changes, status published on a clock, and the site that is always different | Available |

---

*Part of the [ChefPlex TPM portfolio](https://github.com/ChefPlex/tpm-templates/blob/main/PORTFOLIO-GUIDE.md). Maintained by [Eric White](https://www.linkedin.com/in/edwhite). Licensed under [CC BY 4.0](LICENSE).*
