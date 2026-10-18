# infrastructure-program-playbooks

Playbooks for Technical Program Managers (TPMs) running infrastructure and platform programs: data
center and site build-outs, server and operating system migrations, directory services, network
rollouts, and cloud adoption.

Infrastructure programs fail in a different place from software programs. The code is rarely the
problem. The problem is the dependency nobody listed, the system you are keeping that the change
must not disturb, and the vendor deadline you did not set. These playbooks are about that part.

Companion to [security-program-playbooks](https://github.com/ChefPlex/security-program-playbooks).

---

## What Is Here

⏳ **In progress. This repository is private until at least three playbooks below are finished.**

### Infrastructure

| Playbook | What It Is | Status |
|:---|:---|:---|
| End-of-Support Migration Playbook | Moving a server and operating system estate off a version the vendor is retiring: the discovery curve, dependency analysis, coexistence with the versions you keep, and the cost of waiting | Planned |
| Directory Services Upgrade and Consolidation | Upgrading and consolidating Active Directory and LDAP (Lightweight Directory Access Protocol) across domains without breaking authentication | Planned |
| Data Center and Site Build-Out | Standing up a new site from contract to production: long-lead procurement, the critical path through power and network, and acceptance | Planned |
| Network Rollout Playbook | Rolling a network change across many sites: waves, cut-over windows, rollback, and the site that is always different | Planned |

### Cloud and Platform

| Playbook | What It Is | Status |
|:---|:---|:---|
| Cloud Migration and Platform Adoption | Moving workloads to one or more clouds and getting teams to adopt a shared platform, measured by use rather than by announcement | Planned |

---

*Part of the [ChefPlex TPM portfolio](https://github.com/ChefPlex/tpm-templates/blob/main/PORTFOLIO-GUIDE.md). Maintained by [Eric White](https://www.linkedin.com/in/edwhite). Licensed under [CC BY 4.0](LICENSE).*
