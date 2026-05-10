+++
title = "CCDC Cheese: A Taxonomy of Score Gaming and Design Responses"
date = "2026-05-10"
author = "tirefire"
description = "Patterns of score gaming in CCDC family events and the design responses available to address them."
toc = true
+++

Collegiate Cyber Defense Competition (CCDC) gives college students hands on practice in cyber defense. Each team gets a fictional small business network with services like email and file shares, and has to harden it, defend it against a red team, keep uptime for simulated customers, and perform business level functions like developing policies for the network. The competition scores all of this through service uptime, inject responses, red team penalties, and incident reports.

Despite all these opportunities, Goodhart's Law still applies: when a measure becomes a target, it stops being a good measure. Teams under pressure take the easiest path, optimizing for whatever changes net the most points with the least effort.

This document catalogs those shortcuts, how scoring should take them into account, and what mechanisms exist to detect, prevent, or disincentivize the behaviors rather than reward them. Each cheese pattern includes a description of how it appears, a real world analog, and competition design responses with their tradeoffs. A final section covers things that look like cheese but are not, and explains the distinction.

## Design response levers

Each pattern below references one or more of these mechanisms.

### Scoring engine checks

Continuous automated checks performed remotely over the network against scored services. They are machine driven and narrow in coverage, since they can only test what they can reach over the network.

### Orange team checks

Hands on keyboard testing by humans who simulate real users and admins. They are sampled rather than continuous and expensive to operate, but they have broader coverage than scoring engine checks because they can test things that are not reachable over the network.

### White team injects

Business tasks delivered to all teams simultaneously through the inject portal. Teams submit PDF deliverables during a defined window. Injects are good for analysis tasks and policy documents, but they are less effective for catching persistent state since teams can enable a capability for an inject and disable it afterward.

### Network monitoring

Passive observation of network traffic, particularly egress, to flag patterns inconsistent with the declared business function.

### Red team

Penetration by a skilled adversary, with scoring tied to the access they obtain.

### Explicit rules and penalties

Prohibitions written into the team packet, enforced through point penalties or disqualification.

### Environment design

Dependencies and feedback loops built into the environment itself, so that breaking subsystems produces an immediate consequence rather than waiting for a probe to catch it.

---

Each lever varies in coverage, latency, cost, and false positive rate. Scoring engine checks and environment design enforce through technology and should generally be preferred to explicit rules and penalties, which depend on policy enforcement and human adjudication.

## What makes a good service check

A service check is a scoring engine check against a scored service. The properties below describe what a good one does, and patterns later in this document reference them as design goals.

### Simulates real use

The check exercises the service the way real users would, including authentication where the service requires it. A web check that fetches a single anonymous page is weak. A web check that authenticates, submits a form, follows the redirect, and verifies the resulting state is strong.

### Gives blue teams immediate feedback

When the team breaks the check, the check fails on the next interval and the team learns within minutes. That is how the competition teaches.

### Rewards security improvements

Genuine hardening should not break the check. A check that requires plaintext or expects default credentials forces a choice between security and points, which is the wrong choice to ask teams to make.

### Detects rather than rewards cheese

Where multiple paths satisfy the measurement, a good check distinguishes the operational path from the gamed one. If a static replacement passes the check, the check is rewarding cheese instead of detecting it.

### Leaves room for security tradeoffs

Defensive decisions should be reflected in the score based on operational consequence, not on whether the team picked the expected approach. Good checks make security feel like real engineering rather than guessing the rubric.

### Measures uptime honestly

Check intervals and timeout values should reflect actual user expectations. A service that takes 30 seconds to respond is broken even if the technical SLA window allows it.

### Is reproducible

The same conditions produce the same result, so teams can predict what will pass and fail. Determinism makes the lessons learnable.

### Has a low false positive rate

Checks that fail when nothing is wrong erode the team's trust in the score, so aim for high confidence and tolerate occasional missed detection in exchange.

## Service availability gaming

Patterns in this category target the most direct measurement: whether a scored service is reachable and responsive. The cheese is making the scoring engine see "up" without the service actually being functional or available to real users.

### 1. Selective response based on client identification

**What it looks like:** The team identifies which traffic comes from scoring infrastructure or from red team using signals visible at the local network position, and then serves the right response to each. Signals include source IP ranges, HTTP User Agent strings, TCP fingerprints (TTL and window size), TLS fingerprints (JA3, JA4), HTTP header patterns, request timing, and behavioral signatures. Variants combine signals so that the team responds fully if traffic comes from a scoring subnet and includes a scoring User Agent, and blocks or degrades anything else.

**Real world analog:** Cloaking, where a server serves different content based on who is asking. Search spam farms do this so that crawlers see clean pages while humans see junk. In ops, the equivalent is a server that returns 200 OK to monitoring while real users get errors.

**Design responses:**

* Run scoring checks from multiple source IPs across the scored network, including from inside the team environment, so that no single source identifies as the scorer.

  > **Design note: overlapping scoring and red team source ranges.** Source scoring traffic from a network range that also serves as the red team source space, with both scoring and red team source addresses randomly distributed across the range. IP based identification of either party becomes structurally impossible, and an entire family of cheese patterns becomes unavailable: any pattern that depends on identifying scoring or red team by source IP. The cost is operational complexity in scoring infrastructure (the scoring engine must source from many addresses across the range) and in red team coordination. The benefit is structural rather than detective, since the patterns stop being viable rather than being caught after the fact.

* Randomize the scoring engine fingerprint across checks by varying User Agent strings, TCP window sizes, TLS client hello values, request ordering, and request spacing.
* Score from sources that are also used for legitimate operational traffic, so that degrading the scoring sources also degrades real use.
* Run orange team checks from sources different from the scoring infrastructure.

**Tradeoffs:** Multi source scoring costs network complexity and check coordination. Fingerprint randomization is cheap to add but escalates into an arms race that the scorer must keep up with. Orange team checks are the most reliable signal but they are bandwidth limited.

### 2. External infrastructure for client characterization

**What it looks like:** The team uses infrastructure outside the scored environment to gather information about clients that is not visible from inside the network. The common form is a scored service that includes a redirect, asset reference (image, script, font), or beacon that contacts a host the team controls outside the competition network. The external host observes source characteristics that differ from what the internal network sees, including actual upstream public IP, ISP, AS number, geolocation, and browser fingerprint at a different network position. The team uses this to identify red team members tunneling through VPNs whose internal source looks normal but whose true upstream IP is identifiable, and then feeds the identification back into selective response decisions.

Variants include external image or script tags in HTML that beacon to a collector, DNS lookups through a domain the team controls, forced redirect to an external authentication or analytics service, and TLS connection to an external fingerprinting service.

**Real world analog:** What tracking pixels and bot detection vendors do. The technique is defensible in production with disclosure. In CCDC, it becomes a way to identify red team operators across network address translation and reduce red team penalty exposure.

**Design responses:**

* Configure the scoring engine to source from inside the competition network only, refuse to follow redirects to external destinations, and refuse to load external assets. The path teams would use to characterize clients via external infrastructure disappears.
* Route all environment internet traffic through a transparent proxy under operations control so that any external beacons from scored services become visible. Combine this with a network policy that scored services may only initiate connections to declared business endpoints.
* Score the egress profile of the scored service. Services that initiate unexpected outbound connections lose points or trigger investigation.
* Run orange team checks through a network path that would also trip whatever identification logic the team has built. If checks fail when the scoring path passes, the pattern is exposed.

**Tradeoffs:** Egress monitoring requires network layer inspection and explicit business endpoint declaration, which is operationally complex. Egress scoring needs continuous traffic analysis. Routing orange team checks through varied network paths increases scoring infrastructure complexity.

### 3. Stub service implementations

**What it looks like:** A listener bound to the scored port that returns a response shaped like a successful scored check but is not the real service. Variants include a static page returning the exact byte sequence the HTTP probe expects, an SSH banner faked by netcat, an SMTP daemon that accepts EHLO, MAIL, RCPT, DATA but routes nothing, and a DNS server that answers only the specific record the scoring engine queries.

**Real world analog:** A chaos testing stub: canned responses while the underlying service is down.

**Design responses:**

* Probe more than the happy path. Vary requests within each protocol so that a stub has to implement enough of the protocol to be functionally equivalent to the real service.
* Probe at the protocol layer for behaviors only a real service produces, like HTTP cookie handling, SSH key exchange completion, or SMTP queue introspection.
* Run orange team checks that exercise the service the way real users would: send mail that must arrive at a separately verified mailbox, transfer a file with verifiable contents, or query a database for content that requires real backend processing.

**Tradeoffs:** Deeper protocol probes raise check complexity and increase the false positive rate when services degrade naturally. Orange team checks are reliable but bandwidth limited.

### 4. Static replacement of dynamic services

**What it looks like:** A scored service that is supposed to serve dynamic content gets replaced with a static cache of the pages the scoring probe checks. The common form is a wget recursive crawl of the original site served from a static web server. The scored URLs return the expected content, while everything that requires state on the server, sessions, forms, or database queries no longer works.

**Real world analog:** Putting a CDN snapshot in front of a broken application server and calling the incident resolved while sales is still seeing checkout failures.

**Design responses:**

* Probe URLs that require state on the server, including form submissions, search results that depend on database content, authenticated pages, and dynamically generated reports.
* Run orange team checks of the application's dynamic features, generating content through the application and verifying it is retrievable through a different access path.
* Add an explicit rule that dynamic services must continue to serve dynamic content.

**Tradeoffs:** Probing dynamic behavior requires the scoring engine to maintain state across requests, which complicates retries and false positive handling. Orange team checks are reliable but bandwidth limited.

### 5. Timing service availability to scoring intervals

**What it looks like:** The team observes the scoring engine cadence (typically a few minutes between checks at randomized intervals) and brings services up only during the expected check window, leaving them down or restricted between checks. Variants include scheduled tasks that start services on a schedule matching observed scoring cadence, and scripts that bring up services in response to detected probe traffic.

**Real world analog:** A service that runs only when monitoring is checking it. No benign production form exists.

**Design responses:**

* Randomize scoring cadence within bounds, eliminating any predictable interval the team can fit a service window into.
* Run continuous availability checks alongside the formal scored checks, so that a service that is only up during the check window fails the continuous check.
* Run orange team checks at random times outside the scoring cadence.

**Tradeoffs:** Randomized cadence reduces the value of timing data the scoring engine collects. Continuous availability checks increase scoring infrastructure load. Orange team checks are bandwidth limited.

## Hardening practices that require comprehensive scoring

Patterns in this category reduce attack surface, which is sensible practice in production when operational continuity is preserved. In a competition with narrow scoring, the same actions create blind spots. The cheese is in how the hardening is done: in ways that pass narrow scoring while breaking workflows the system is supposed to support. Each entry below documents the behavior, then describes what comprehensive scoring should exercise so that operational reality is visible and rewarded.

### 6. Removing operational tooling

**What it looks like:** Removing or restricting standard system commands and binaries that admins, scheduled tasks, or other services need to operate on the system. On Linux, variants include deleted shells, removed binaries (network tools, compilers, scripting interpreters), restricted execution via noexec mount or LSM policies (AppArmor, SELinux), and restricted or emptied environment variables that programs depend on. On Windows, variants include AppLocker or Windows Defender Application Control rules that block cmd.exe, powershell.exe, or other administrative tools, ConstrainedLanguageMode for PowerShell, and removed or renamed binaries in System32.

**Real world analog:** A locked down kiosk image. On a general purpose server, the same restrictions break workflows the system needs. Attack surface reduction is good practice in production when the system's operational requirements are still met.

**What comprehensive scoring should exercise:**

* Orange team checks of admin tasks: log in, run package updates, inspect logs, check service status, restart a service, modify a configuration. If the admin workflow fails, the team learns immediately that something they did broke operational reality.
* White team injects requiring tooling whose effect must be subsequently verified, such as installing a monitoring tool the scoring engine then checks remains functional.
* Scoring engine checks that exercise scored services through internal mechanisms which depend on common tooling.

**Tradeoffs:** Orange team checks of admin workflows are bandwidth limited but the most authentic signal. White team injects test at a point in time and miss state that gets enabled briefly. Internal probes raise scoring infrastructure complexity.

### 7. Removing scheduled task subsystems

**What it looks like:** Disabling or removing the system's scheduled task subsystems on the theory that attackers use them for persistence. On Linux, this means cron, at, and systemd timers, with variants like removing the binaries, masking the systemd units, emptying configuration directories, and killing the daemons. On Windows, this means Windows Task Scheduler, with variants like disabling the Schedule service, removing scheduled tasks via schtasks or Task Scheduler MMC, and Group Policy that prevents task creation.

**Real world analog:** A server with cron disabled because no application currently uses it. Defensible only when the system genuinely has no scheduled work, since log rotation, package updates, certificate renewal, backup jobs, and other routine maintenance all rely on it.

**What comprehensive scoring should exercise:**

* Environment design that builds in dependencies on scheduled work, like a noisy service that fills the disk without log rotation, a service that requires periodic certificate renewal, or a database that needs scheduled vacuum. The team feels the consequence of disabling cron immediately rather than waiting for a probe.
* White team injects requiring evidence of scheduled work as cumulative output, such as a final inject asking for backup history, log rotation evidence, or scheduled report runs across the competition window.
* Orange team checks of scheduled maintenance, including log rotation timestamps, certificate expiry, and scheduled report output across the competition window.

**Tradeoffs:** Built in environment dependencies require design effort upfront but produce continuous, immediate feedback. White team injects requiring cumulative evidence work but require careful inject design. Orange team checks of scheduled maintenance require extended observation windows.

### 8. Restricted DNS resolution

**What it looks like:** Disabling or restricting DNS resolution from team systems on the theory that attackers need DNS for command and control or data exfiltration. On Linux, variants include empty /etc/resolv.conf, resolver pointed at non functional servers, firewalled outbound DNS, and disabled systemd resolved or nscd. On Windows, variants include empty DNS server list in network adapter settings, hosts file overrides that misroute lookups, disabled DNS Client (DnsCache) service, and Group Policy that breaks DNS configuration on domain joined systems.

**Real world analog:** A production server with DNS broken. Updates fail, NTP fails, certificate validation fails, and anything by hostname fails. A real ops team notices within minutes.

**What comprehensive scoring should exercise:**

* Score services by DNS name rather than IP, so that resolution failures fail the check directly.
* Environment design with inter service dependencies that exercise transitive name resolution, like a web app that connects to its database by hostname, a mail server that does MX lookups, or federated trust that requires DNS based discovery.
* Orange team checks of DNS dependent workflows, including package updates and certificate validation by hostname.

**Tradeoffs:** Name based scoring is straightforward to set up. Transitive dependencies require environment design effort. Orange team checks are bandwidth limited.

### 9. Restricted interactive login surface

**What it looks like:** Restricting who can obtain an interactive shell or session on the system in ways that satisfy a basic authentication probe but break legitimate admin and user workflows. On Linux, variants include setting all or most account shells to /bin/false or /sbin/nologin, removing login shells from /etc/shells, locking accounts at the password level, configuring PAM modules (pam_access, pam_listfile, pam_succeed_if, pam_time) to allow scoring's authentication and command execution but deny full session establishment, restricting SSH via ForceCommand to a single command and exit, and isolating sshd in a container with no access to the underlying system. On Windows, variants include "Deny log on locally" or "Deny log on through Remote Desktop Services" Group Policy assigned broadly, removing accounts from the Remote Desktop Users group while keeping them active for service authentication, AppLocker rules that block cmd.exe and powershell.exe at login, and restrictive Logon Workstations settings on user accounts.

**Real world analog:** A bastion host configured with ForceCommand for git over SSH, or service accounts with /sbin/nologin shells. Both are standard practice when constrained access is the entire intended use. The Windows equivalent is a service account configured with "Deny log on locally" so it can authenticate to a service without ever being able to log in interactively. Applied to accounts that legitimately need interactive access (admins, operators, anyone whose role includes logging into the system), the same restrictions become a pattern that breaks operational reality.

**What comprehensive scoring should exercise:**

* Scoring engine checks against SSH and other interactive services that issue a real command after authentication and verify expected output, not just authentication success.
* Orange team checks where admins log in interactively and run expected administrative tasks like rotating a password, inspecting logs, installing a tool, or restarting a service.

**Tradeoffs:** Real shell session probes raise check complexity and increase the false positive rate when sessions terminate naturally. Orange team checks are reliable but bandwidth limited.

### 10. Password rotation gaming

**What it looks like:** The team rotates user passwords on scored services in ways that satisfy a narrow probe but break operational reality. Variants include rapid rotation timed faster than the scoring cadence so that probes catch only a brief valid window, convergence to a single shared password across all users for ease of rotation, and mass changes that break automation, integrations, or downstream services that depended on the previous credentials.

**Real world analog:** Mass password resets after a confirmed breach are standard practice. Done preemptively and continuously regardless of evidence, or in ways that compromise security further (a single shared password across users defeats the entire purpose of distinct credentials), the same mechanism produces no security gain.

**What comprehensive scoring should exercise:**

* Scoring engine checks that authenticate as individual users with their assigned credentials at intervals. If rotation is rapid enough that scoring credentials become invalid between checks, scoring fails.
* Orange team checks where users authenticate with their assigned credentials and perform user level work like reading mail, logging in, or accessing files. Rotation that breaks user authentication breaks the operational role.
* Explicit rules around password rotation cadence and justification, such as requiring written justification for mass changes beyond a threshold and prohibiting single shared password configurations.
* Password change request (PCR) coordination, where teams that change passwords on scored services submit the new credentials through the scoring engine's PCR page so that the scoring engine can authenticate with current credentials. Rotation without PCR submission breaks scoring authentication.

**Tradeoffs:** Per user authentication probes require the scoring engine to track and use distinct credentials per team and per user. Orange team checks of user authentication are bandwidth limited. Explicit rules require enforcement and adjudication. PCR coordination puts the burden on the team to keep the scoring engine current, and missed PCRs cause scoring failures that may look like cheese but are actually operational mistakes.

### 11. Workstation total neutering

**What it looks like:** Stripping a system that exists in the simulation to be a user workstation of all user functionality, including removing or disabling the browser, document tools, email client, desktop applications, file share access, and other elements that make it usable as a workstation. The common motivation is that the workstation is a soft target for red team and is not directly scored as heavily as servers, so removing user functionality reduces attack surface without obvious penalty.

**Real world analog:** Developers and security engineers run services on their workstations all the time, and turn off applications they do not use. The cheese is total stripping of user functionality from a system whose role in the simulation is to be a user, with no corresponding business case to explain the change.

**What comprehensive scoring should exercise:**

* Orange team checks that simulate the user the workstation belongs to: log into the desktop session, open the browser to a specific page, open a development environment and compile something, open a spreadsheet from a file share and edit it, open the email client and send a message that arrives at a verifiable destination.
* Scoring engine checks for the presence of expected user environment elements: installed applications, browser profiles, document folders, configured email client, file share mounts.

**Tradeoffs:** Orange team checks of user behavior are the most reliable signal but require scripts or human interaction simulating users. Environment presence probes can be partially gamed if the team retains the appearance without the substance.

## Network layer denial

Patterns in this category restrict network traffic flow on the theory that attackers need network access. Like the hardening patterns above, these reductions may be sensible in production with operational continuity preserved. In competition, the same restrictions can pass narrow scoring while breaking workflows that depend on network connectivity.

### 12. Blanket egress restriction

**What it looks like:** The team blocks all or most outbound network traffic from team systems on the theory that attackers need outbound channels for command and control or exfiltration. Variants include default deny outbound firewall with narrow allowlist for scoring sources, blocking specific outbound protocols (DNS, HTTP, SSH on Linux; SMB, WinRM, RDP, LDAP on Windows), and blocking specific destinations.

**Real world analog:** An air gapped or network isolated production environment. Defensible only when the systems genuinely have no outbound dependencies, since updates, NTP, DNS, license validation, telemetry, certificate validation, and most other operational needs require some kind of egress.

**What comprehensive scoring should exercise:**

* Score outbound functionality from team systems, including NTP synchronization, certificate revocation checks, DNS resolution to external hosts, and patch connectivity (apt or yum repositories on Linux, Windows Update on Windows).
* Environment design with outbound dependencies built in, like services that require periodic external authentication, certificate renewal that requires CA contact, or package management that requires repository access.
* Orange team checks that users can browse the internet, send mail externally, and otherwise reach the cloud services they are expected to reach.

**Tradeoffs:** Outbound scoring requires the scoring engine to test connectivity from each team system. Built in dependencies require environment design. Orange team checks of internet use are bandwidth limited.

### 13. Connection time and rate limits

**What it looks like:** The team applies firewall or service level rules that terminate connections after short time windows or aggressively rate limit sources. Variants include blocking connections that exceed a few seconds duration, terminating long running sessions, and dropping requests above a low rate threshold per source.

**Real world analog:** Production systems use connection timeouts and rate limiting routinely (DoS protection, slow loris mitigation, fair queuing). Set aggressively enough to prevent normal operational use, the same controls catch sessions that should last hours and treat normal user request rates as abusive.

**What comprehensive scoring should exercise:**

* Score sessions that require sustained duration, like long running SSH sessions executing real commands, file transfers that take time, database queries that return large result sets, and video or audio streaming.
* Score request patterns at realistic user rates, including bursts of legitimate activity, a user clicking through a web app, or a developer making API calls.
* Orange team checks of admin workflows that require multi step interactive sessions.

**Tradeoffs:** Sustained session scoring requires the scoring engine to maintain stateful connections, raising complexity. Realistic rate testing requires the scoring engine to send traffic at user like rates rather than uniformly.

## Recovery and rollback gaming

Patterns in this category use system state restoration to undo red team intrusions mechanically, bypassing detection and remediation. Snapshot revert and image rebuild are legitimate recovery tools after a confirmed incident. Applied automatically or on a schedule, they eliminate the incident response work the competition is designed to teach.

### 14. Mechanical state rollback

**What it looks like:** The team configures system state to revert to a known snapshot or image without engaging with the incident response work that real recovery requires. Variants include automation triggered by intrusion detection signals (scripts triggered by file integrity monitoring alerts, watchdog services that compare running state against a baseline) and scheduled rollback regardless of evidence (cron jobs or Windows scheduled tasks that revert to a snapshot every hour, image redeployment timed to scoring cadence).

**Real world analog:** Immutable infrastructure with rebuild on anomaly is real in cloud ops, and Kubernetes self healing is appearing in CCDC environments. These approaches work because rebuilds are paired with detection and root cause work, and because the operational cost of running the platform is significant. A script that auto reverts on alert mimics the surface behavior without the engineering depth that makes it function in production. The scheduled variant has no production analog at all, representing a misunderstanding of rollback as defense rather than recovery.

**What comprehensive scoring should exercise:**

* Explicit point penalty per revert above a free threshold. The team has a free budget of reverts for legitimate use during the competition, and reverts beyond that incur direct point cost. This is the primary deterrent in environments where the competition controls the underlying infrastructure.
* Environment design where blue teams own their own infrastructure. When the team manages its own snapshots and configurations, a revert erases all defensive work the team performed (hardening, patches, custom configurations). Red team can then compromise the freshly reverted state again, producing direct point loss through compromise penalties. The natural consequence makes the deterrent intrinsic rather than rule based.
* Environment design with continuously accumulating state. Scored services should accumulate operational state during the competition that mirrors real production usage, including emails sent and received in the mail server, customer purchases in the ecommerce database, file updates in the document store, and log history that builds over time. When a team reverts, this state is lost. An orange team simulated user notices their recent email is missing, a customer record from earlier in the competition has vanished, a file the team modified is back to its original state. The natural consequences mirror what would happen in production, where customers would notice and complain.
* Inject deliverables that require artifacts produced between intrusion and revert (incident report describing what red team did, evidence of investigation, remediation plan). Reverting before producing these loses the deliverable.
* Orange team checks of state changes that should persist (configurations the team made, scheduled work output, deliverables in progress). Rollback undoes those.
* Detection of revert patterns, where state returns to identical bytes of a known snapshot multiple times in succession and gets flagged for inspection.

**Tradeoffs:** Explicit revert penalties require accurate revert detection across the environment and an agreed free threshold. Blue team owned infrastructure pushes the deterrent into the environment but requires teams to do the infrastructure work. Continuously accumulating state requires environment design effort to generate state realistically and scoring infrastructure to verify it. Inject deliverables tied to incidents require the inject schedule to be unpredictable. Orange team checks of persistent state require environment design choices about what should remain. Detection of revert patterns requires snapshot fingerprinting infrastructure.

## Inject gaming

Patterns in this category satisfy the formal requirements of an inject (a PDF deliverable submitted through the inject portal) without doing the underlying work. The team produces a document that looks complete but lacks the analysis, environment specific information, or operational understanding the inject was meant to require.

### 15. Boilerplate or generated inject responses

**What it looks like:** The team produces inject deliverables that satisfy the formal requirements (a PDF of approximately the right shape, submitted on time) without doing the underlying work. Variants include copy pasted templates from prior competition years, generic policy boilerplate downloaded from compliance template sites, AI generated documents that look complete but contain generic content or hallucinated specifics, network inventories that do not reflect the actual environment, and incident reports that describe generic attacks rather than what the red team actually did.

**Real world analog:** Compliance theater. Organizations that maintain documented policies and procedures for the auditor but never apply them operationally.

**What comprehensive scoring should exercise:**

* Inject grading by humans who can detect generic content versus original work that reflects the team's actual environment. Templates that provide formatting and structure are legitimate efficiency, while templates that provide the substantive content are the cheese form. Grading should distinguish between the two by requiring environment specific content even within templated structure. Subjectivity is partially mitigated by objective rubrics per inject and by grading each inject by at least two humans.
* Inject content that requires environment specific information, like a network inventory that must reflect this team's specific systems and addresses, or an incident report that must describe what red team actually did at this team's site.
* Cross referencing inject content against environment state. For example, an AUP that requires multi factor authentication while the team's systems do not have it configured should flag the contradiction.
* Follow up injects that build on previous inject responses, like an inject that asks the team to implement what they wrote in their incident response plan from an earlier inject.
* Injects that require demonstration through application rather than description. Do not ask the team to write a policy on patching, ask them to patch the systems and produce evidence.

**Tradeoffs:** Human grading is bandwidth limited. Environment specific content requires injects designed around team specific data, raising inject preparation effort. Cross referencing requires the scoring infrastructure to have visibility into team environment state. Follow up injects require schedule design that allows building. AI detection is unreliable and the gap is closing rapidly.

## Before competition prep gaming

Patterns in this category move work from inside the competition window to before T0, in ways that bypass the analytical and operational work the competition is designed to teach. The competition tests how teams respond to an unfamiliar environment under pressure, and pre work that obviates that response defeats the test.

### 16. Replacement images deployed at T0

**What it looks like:** The team prepares replacement systems before the competition starts (containers, VMs, full OS images with services preconfigured) and deploys them at T0 to swap out the given environment. Variants include containerized service replacements, full VM swaps, and kickstart or cloud init configurations that rebuild systems on first boot.

**Real world analog:** Infrastructure as code with declarative deployment is normal production practice. Used to skip the work of analyzing the given environment, that practice defeats what the competition was designed around: figure out what is here, find what is wrong, decide what to fix.

**What comprehensive scoring should exercise:**

* Heavier SLA penalty schedule early in the competition. The first hours carry larger SLA violation penalties than later periods, which directly catches teams whose T0 image swap causes service outages during the transition. The moment of swap is when the team is most likely to break things, and it is when the penalty hurts most.
* Interconnected services. Scored services depend on each other. The web app needs the database, the mail server needs DNS, the file share needs authentication. If one service goes down during a transition, others fail too, and the cost of a botched swap multiplies through the dependency graph.
* Vulnerabilities at both application and operating system layers. The given environment includes intentional vulnerabilities at both the OS layer (default credentials, unpatched packages) and the application layer (web app bugs, misconfigurations). Migrating the OS does not fix the application vulnerabilities, so teams that swap the OS still face the application bugs they did not investigate.
* Inject deliverables that require analysis of the given environment, like a network inventory inject that is impossible to complete with a replacement image without doing the analysis first.
* Forensic injects that depend on system state. Injects ask the team to identify and document anomalies on the systems, including prepositioned forensic artifacts placed in the environment before the competition (anomalous log entries, unusual files, indicators of prior compromise) and red team activity artifacts generated during the competition (evidence of attacks, persistence indicators, modified configurations). A T0 image swap loses the prepositioned artifacts, and a mid competition migration loses the red team evidence. Either way, the forensic injects become impossible to complete.
* Environment design that includes services and configurations that cannot be trivially replicated in advance, like custom internal applications, environment specific data, or integrations that depend on the given systems.

**Tradeoffs:** Heavier early SLA penalties require schedule design so that legitimate environment exploration is not over penalized. Environment design choices (interconnection, multi layer vulnerabilities, non replicable elements) require significant upfront investment but produce structural deterrents that work without inspection. Analysis and forensic injects require coordination between environment design (prepositioned artifacts), red team operations (artifact generation), and inject schedule (asking for documentation at the right times).

### 17. Prepared hardening automation

**What it looks like:** The team prepares scripts before the competition that run at T0 (or shortly after) to harden systems automatically by changing all default passwords, disabling all non essential services, applying firewall rules, installing monitoring tools, and so on. Scripts may be invoked by hand or set to run on boot.

**Real world analog:** Configuration management with idempotent hardening playbooks is standard production practice. Run before any analysis of what is actually present, the same playbooks violate Chesterton's fence: the team removes things they have not investigated and may need.

**What comprehensive scoring should exercise:**

* Inject deliverables that require analysis of the existing environment before changes. The team must produce a network inventory or systems inventory before they are credited for hardening work.
* Continuous scoring of services that blanket hardening would commonly break, so that running prepared scripts produces immediate scoring loss.
* Orange team checks of workflows that prepared scripts would interfere with. If the script disables a service the system needs, the dependent workflow fails immediately.
* Environment design with non obvious dependencies that prepared scripts would not anticipate, like services that depend on each other in unusual ways or custom configurations the team must understand to preserve.
* Heavier SLA penalty schedule early in the competition. As with pattern 16, the early window is when T0 hardening scripts are most likely to cause outages, and the penalty schedule makes those outages costly.

**Tradeoffs:** Analysis injects require schedule design that delivers them early enough to be a gate. Continuous scoring of breakable services is the mainline scoring engine work. Orange team checks depend on the workflows being non trivial. Environment design with non obvious dependencies is the deepest investment but the most effective, since it punishes prepared scripts directly through immediate consequence.

## Coordination gaming

This category covers patterns where the team uses resources outside the formal competition framework, like external communication, third party help, sharing between teams, or other coordination that gives the team capabilities not granted by the competition structure.

### 18. External assistance and inter team coordination

**What it looks like:** The team obtains help during the competition that is outside the formally permitted resources, including communicating with coaches, alumni, or industry contacts via phone, chat, or in person; using social media or chat platforms to ask for solutions; coordinating with other teams to share approaches or solutions; and consulting with experts who are not on the team roster.

**Real world analog:** In production, asking colleagues, vendor support, or online communities for help is normal practice. In a competition specifically designed to test the team's own capability under time pressure with limited resources, the same outside help defeats the test.

**What comprehensive scoring should exercise:**

* Explicit rules defining what communication is permitted. Typically only with white team, orange team, and operations during the competition, and no contact with anyone outside the competition framework.
* Physical isolation of the competition space, like a controlled room with no personal devices, monitored entry, and on site observation.
* Network isolation of competition systems from the open internet, with permitted resources (documentation sites, package mirrors) accessible through monitored proxies.
* White team observation in the competition area, watching for prohibited communication.
* Logging and inspection of any devices the team brings into the competition area.

**Tradeoffs:** Physical and network isolation requires venue infrastructure that not all events can provide. White team observation is bandwidth limited and intrusive. Device inspection is invasive and slow. Permitted resource lists require maintenance and create the question of where to draw the line, like using AI for syntax help versus using AI to write the entire incident report. The harder the isolation is, the less the competition resembles real ops work where consultation is normal. The more open the access, the more advantage accrues to teams with rich support networks outside the room.

## What is not on this list and why

Some practices look like the patterns above but are legitimate hardening rather than cheese. Distinguishing them matters. A practice qualifies as legitimate hardening if it passes both of the following tests:

1. A competent ops team would make this change in a comparable production environment.
2. The change does not leave any of the following broken: desktop and workstation use, admin workflows, incident response capability, forensic recovery capability, business continuity, normal user productivity.

A practice that passes both is hardening. A practice that fails either is cheese, or in some cases simple bad practice.

The practices below are commonly confused with the cheese patterns but pass the test, and should not be penalized.

* Restricting DNS zone transfers (AXFR / IXFR) to specific secondary servers. Standard BIND security guidance and CIS benchmark for any authoritative DNS server.
* Disabling DNS recursion on authoritative only servers. A server whose only role is authoritative answers should not also resolve external names.
* Disabling SMTP VRFY and EXPN. Both commands enable user enumeration and have no operational use, so disabling them is standard hardening.
* Removing development tools from production servers (compilers, scripting interpreters not used by production services, network diagnostic utilities). CIS benchmark guidance.
* Mounting /tmp and /home with noexec. CIS benchmark recommendation for shared and user writable filesystems.
* Disabling unused PAM modules. If the system has no Kerberos KDC, removing pam_krb5 is hardening rather than cheese, and the same applies to LDAP modules without a directory and so on.
* Disabling NetworkManager on static IP servers. Servers that do not change network configuration do not need a network configuration daemon.
* Disabling systemd resolved when the system uses direct /etc/resolv.conf. Reduces attack surface without breaking name resolution.
* Disabling Print Spooler on Windows servers that do not print. CIS benchmark and post PrintNightmare standard practice.
* Removing or disabling SMBv1 on Windows. Widely accepted security practice for any system that does not need to communicate with legacy clients.
* Disabling LLMNR and NetBIOS over TCP/IP on Windows. CIS benchmark recommendation, since both protocols are commonly abused for credential theft and rarely needed in modern environments.
* Disabling unused Windows services (Remote Registry, Telnet client and server, Internet Connection Sharing, and similar) on systems that do not require them. CIS and DISA STIG guidance.
* Restricting NTLM authentication to NTLMv2 only and disabling LM and NTLMv1. Standard hardening for any Windows environment.
* TLS version floor at 1.2 or 1.3. Disabling TLS 1.0 and 1.1 is widely accepted security practice. The cheese version would be requiring something the scoring engine does not support, but the floor itself is legitimate.
* Disabling auditd, syslog, or journald produces no positive scoring outcome, which is why it is not on the cheese list. It belongs in the bad practice category (self inflicted blindness, broken incident reporting capability), just not the kind of bad practice this taxonomy covers.
* Migration to a different operating system or software stack with proper analysis, change management, and operational continuity is legitimate engineering. The cheese version (T0 swap to bypass analysis) is covered by pattern 16, and migration done correctly is not on this list.
* Kubernetes self healing and immutable infrastructure with proper engineering depth. Real production patterns increasingly seen in CCDC environments. They work because the operational cost is significant and the rebuild is paired with detection and root cause work. The cheese version (a script that mimics self healing without the engineering) is covered by pattern 14, and the legitimate version belongs here.

A separate category of context dependent cases requires more judgment.

* IPv6 disabled. Legitimate if IPv6 is not part of the environment's role, cheese if the environment includes IPv6 services.
* STARTTLS disabled on SMTP. Legitimate if mail is restricted to authenticated submission on a TLS only port, cheese if the environment expects opportunistic encryption for legacy clients.
* Capability stripping on binaries. Legitimate for binaries that are not used, cheese if the binary is needed.
* System wide umask 077. Legitimate for single tenant servers, cheese for shared environments.
* Disabling dbus. Legitimate on headless servers without GUI or desktop dependencies, cheese where polkit, NetworkManager, or other dbus consumers are needed.
* Inbound port blocking as defense against specific exploits. Legitimate defense in depth, since CIS benchmarks recommend it for many service types. It becomes cheese only if used as a complete substitute for addressing the underlying vulnerability and the service remains vulnerable internally. The line is whether the team has done the work to reduce internal exploit risk in addition to blocking external access.

For the context dependent cases, the test is the same: does the change break a workflow the system needs? Apply it case by case.

With comprehensive scoring (orange team checks, environment design with real dependencies, scoring engine checks that exercise full functionality), correct hardening is rewarded directly. The team that hardens without breaking anything passes scoring, and the team that hardens broadly without analysis fails immediately. Without comprehensive scoring, the cheese patterns and legitimate hardening look indistinguishable from outside, and the practitioners doing the right thing get unfairly grouped with the practitioners gaming the score.

## Closing

Each pattern lets a team pass the scoring probe without doing the operational work the probe was supposed to detect, and each design response closes the gap between what scoring measures and what the team is actually doing.

Picking the right design response for each pattern is engineering. The tradeoffs are real and unavoidable, like coverage against complexity, automated against human verification, and prevention against detection. No response covers every pattern, and most patterns need a combination of responses to address well.

Comprehensive scoring is expensive. Orange team checks need humans, environment design needs design effort, inject content needs preparation, and scoring engine checks that exercise full functionality need engineering investment. Every competition makes its own decisions about how much of this expense to take on, given its resources and goals.

Returning to Goodhart's Law: when a measure becomes a target, it ceases to be a good measure. The patterns above are how that principle plays out in cyber defense competition, and the design responses are how organizers push back. Neither side wins permanently. Teams find new patterns and organizers add new responses, and the competition stays educational only as long as the responses keep up.

The taxonomy here is a snapshot. Patterns will evolve, responses will follow, and the work is ongoing.
