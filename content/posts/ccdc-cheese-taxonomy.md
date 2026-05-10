+++
title = "CCDC Cheese: A Taxonomy of Score Gaming and Design Responses"
date = "2026-05-09"
author = "tirefire"
description = "Patterns of score gaming in CCDC family events and the design responses available to address them."
toc = true
+++

The Collegiate Cyber Defense Competition (CCDC) and its regional events exist to develop, under pressure, the skills that translate to operational practice. The scoring engine measures whether services stay up, whether teams respond to business tasks, whether red team gets into systems. Scores summarize how well teams executed the operational role they were assigned.

This is Goodhart's Law in action: when a measure becomes a target, it ceases to be a good measure. Teams optimize for the score, and under sustained optimization pressure the score and the underlying skill diverge. A team can score well by exercising the skill the score was meant to measure, or by gaming the measurement. Cheese is the second path.

This piece catalogs cheese mechanisms observed across CCDC events. Each pattern has a name, a description of what it looks like, the production behavior it corresponds to, the design responses available to address it, and the tradeoffs across those responses. The patterns share a structure: each one allows a team to pass the scoring probe without doing the operational work the probe was supposed to detect. The design responses share a structure too: each one closes the gap between what scoring measures and what the team is actually doing.

A short final section addresses the inverse problem. Some practices look like cheese but are legitimate hardening. The distinction matters and has a test.

## Design response levers

Throughout this piece, the design responses for each pattern draw from a small set of mechanisms organizers have available:

* **Scoring engine probes.** Continuous automated checks the scoring engine performs against scored services. Machine driven and narrow in coverage.
* **Operational verification.** Active testing by orange team or equivalent (humans or human driven scripts simulating real users and admins). Broader in coverage but sampled rather than continuous, and expensive to operate.
* **Injects.** Business tasks delivered to all teams simultaneously, with PDF deliverables submitted through the inject portal. Useful for analysis tasks, policy documents, and evidence of completion. Limited for catching persistent state since teams can briefly enable a capability for the inject and disable it after.
* **Network monitoring.** Passive observation of traffic, particularly egress, to flag patterns inconsistent with declared business function.
* **Red team adversarial exercise.** Penetration of the environment by a skilled adversary, with scoring tied to access obtained.
* **Explicit rules and penalties.** Prohibitions with point penalties or disqualification for specified behaviors.
* **Environment design.** Building dependencies and feedback loops into the environment itself, so that breaking subsystems produces immediate consequence rather than waiting for a probe.

Each lever has different coverage, latency, cost, and false positive characteristics. The patterns in the taxonomy below reference these levers by name.

## What makes a good service check

A service check is a scoring engine probe against a scored service. The properties listed here describe what to aim for when designing one. Not every check will hit every property. Patterns in the taxonomy reference these as design goals.

**It simulates a real use of the service.** The check exercises the service the way a real user, admin, or downstream system would, not a narrow happy path. For services that require authentication, the check logs in and exercises functionality available to that user. A web check that fetches a single anonymous page is weak. A web check that authenticates, submits a form, follows the redirect, and verifies the resulting state is strong.

**It gives blue teams immediate feedback.** When the team does something that breaks the check, the check fails on the next interval. The team learns within minutes. This is how the competition teaches.

**It rewards security improvements, or at least does not penalize them.** Configuring TLS correctly, restricting privilege, applying patches, and other genuine hardening should not break the check. If the check requires plaintext, rejects modern TLS, or expects default credentials, the team is forced to choose between security and points.

**It detects rather than rewards cheese.** Where multiple paths can satisfy the measurement, a good check distinguishes the operational path from the gamed one. If a static replacement passes the check, the check is rewarding cheese rather than detecting it.

**It leaves room for security tradeoffs.** The team should be able to make non obvious defensive decisions and have those reflected in the score based on operational consequence, not on whether the team picked the "expected" approach. Good checks make security design choices feel like real engineering tradeoffs rather than guessing the rubric.

**It measures uptime and SLA honestly.** Check intervals, retry policy, and timeout values reflect actual user expectations, not artificial precision. A service that takes 30 seconds to respond is broken even if the technical SLA window allows it.

**It is reproducible.** The same conditions produce the same result. Teams can predict what will pass and what will fail. Determinism in scoring makes the lessons learnable. Randomness makes for an unfair competition.

**It has a low false positive rate.** Checks that fail when nothing is wrong erode the team's trust in the score and the organizer's ability to investigate when something breaks. Aim for high confidence and tolerate occasional missed detection in exchange.

## Service availability gaming

Patterns in this category target the most direct measurement: whether a scored service is reachable and responsive. The cheese is making the scoring engine see "up" without the service actually being functional or available to real users.

### 1. Selective response based on client identification

**What it looks like.** The team identifies which traffic comes from scoring infrastructure or from red team using signals visible at the local network position, and serves the right responses to each. Identification signals include source IP ranges, HTTP User Agent strings, TCP fingerprints (TTL, window size, MSS), TLS fingerprints (JA3, JA4), HTTP header patterns, request timing, and behavioral signatures. Variants combine signals: "respond fully if from scoring subnet and with scoring User Agent, otherwise serve degraded responses or block."

**Real world analog.** Cloaking: serving different content based on who is asking. Search spam farms do it (crawlers see clean pages, humans see junk). In ops, a server that returns 200 OK to monitoring while real users get errors.

**Design responses.**

* Run scoring checks from multiple source IPs across the scored network, including from inside the team environment, so no single source identifies as the scorer.

  > **Design note: overlapping scoring and red team source ranges.** Source scoring traffic from a network range that also serves as the red team source space, with both scoring and red team source addresses randomly distributed across the range. When IP based identification of either party is structurally impossible, an entire family of cheese patterns becomes unavailable to teams: source based selective response, IP blocking based on observed red team sources, subnet level denial. The cost is operational complexity in scoring infrastructure (the scoring engine must source from many addresses across the range) and in red team coordination. The benefit is structural rather than detective: the patterns stop being viable rather than being caught after the fact.

* Randomize the scoring engine fingerprint across checks: vary User Agent strings, TCP window sizes, TLS client hello values, request ordering, request spacing.
* Score from sources that are also used for legitimate operational traffic, so degrading the scoring sources also degrades real use.
* Operational verification by simulated users from sources different from the scoring infrastructure.

**Tradeoffs.** Multi source scoring catches the pattern at the cost of network complexity and check coordination. Fingerprint randomization is cheap to add but escalates into an arms race. Operational verification is the most reliable signal but is bandwidth limited.

### 2. External infrastructure for client characterization

**What it looks like.** The team uses infrastructure outside the scored environment to gather information about clients that is not visible from inside the network. Common form: the scored service includes a redirect, asset reference (image, script, font), or beacon that contacts a host the team controls outside the competition network. The external host observes source characteristics that differ from what the internal network sees: actual upstream public IP, ISP, AS number, geolocation, browser fingerprint at a different network position. The team uses this to identify red team members tunneling through VPNs whose internal source looks normal but whose true upstream IP is identifiable, then feeds the identification back into selective response decisions.

Variants: external image or script tags in HTML that beacon to a collector, DNS lookups through a domain the team controls, forced redirect to an external authentication or analytics service, TLS connection to an external fingerprinting service.

**Real world analog.** What tracking pixels, ad networks, and bot detection vendors do. Defensible in production with disclosure. In CCDC, a way to identify red team operators across network address translation and reduce red team penalty exposure.

**Design responses.**

* Configure the scoring engine to source from inside the competition network only, refuse to follow redirects to external destinations, and refuse to load external assets. This eliminates the path teams would use to characterize clients via external infrastructure.
* Route all environment internet traffic through a transparent proxy under operations control, providing visibility into any external beacons from scored services. Combine with network policy that scored services may only initiate connections to declared business endpoints.
* Score the egress profile of the scored service. Services that initiate unexpected outbound connections lose points or trigger investigation.
* Run operational verification through a network path that would also trip whatever identification logic the team has built. If verification fails when the scoring path passes, the pattern is exposed.

**Tradeoffs.** Egress monitoring requires network layer inspection and explicit business endpoint declaration, which is operationally complex. Egress scoring needs continuous traffic analysis. Routing operational verification through varied network paths increases scoring infrastructure complexity.

### 3. Stub service implementations

**What it looks like.** A listener bound to the scored port that returns a response shaped like a successful scored check but is not the real service. Variants: a static page returning the exact byte sequence the HTTP probe expects, an SSH banner faked by netcat, an SMTP daemon that accepts EHLO, MAIL, RCPT, DATA but routes nothing, a DNS server that answers only the specific record the scoring engine queries.

**Real world analog.** A chaos testing stub: canned responses while the underlying service is down.

**Design responses.**

* Probe more than the happy path. Vary requests within each protocol so a stub has to implement enough of the protocol to be functionally equivalent to the real service.
* Probe at the protocol layer for behaviors only a real service produces: HTTP cookie handling, SSH key exchange completion, SMTP queue introspection.
* Operational verification by simulated users interacting with the service the way a real user would: send mail that must arrive at a separately verified mailbox, transfer a file with verifiable contents, query a database for content that requires real backend processing.

**Tradeoffs.** Deeper protocol probes raise check complexity and increase false positive rate when services degrade naturally. Operational verification is reliable but bandwidth limited.

### 4. Static replacement of dynamic services

**What it looks like.** A scored service that is supposed to serve dynamic content gets replaced with a static cache of the pages the scoring probe checks. Common form is a wget recursive crawl of the original site served from a static web server. The scored URLs return the expected content. Everything that requires state on the server, sessions, forms, or database queries no longer works.

**Real world analog.** Putting a CDN snapshot in front of a broken application server and calling the incident resolved while sales is still seeing checkout failures.

**Design responses.**

* Probe URLs that require state on the server: form submissions, search results that depend on database content, authenticated pages, dynamically generated reports.
* Operational verification by simulated users interacting with the application's dynamic features, generating content through the application and verifying it is retrievable through a different access path.
* Explicit rule that dynamic services must continue to serve dynamic content.

**Tradeoffs.** Probing dynamic behavior requires the scoring engine to maintain state across requests, which complicates retries and false positive handling. Operational verification is reliable but bandwidth limited.

### 5. Timing service availability to scoring intervals

**What it looks like.** The team observes the scoring engine cadence (typically a few minutes between checks at randomized intervals) and brings services up only during the expected check window, leaving them down or restricted between checks. Variants: scheduled tasks that start services on a schedule matching observed scoring cadence, scripts that bring up services in response to detected probe traffic.

**Real world analog.** A service that runs only when monitoring is checking it. No benign production form exists.

**Design responses.**

* Randomize scoring cadence within bounds, eliminating any predictable interval the team can fit a service window into.
* Run continuous availability checks alongside the formal scored checks, so a service that is only up during the check window fails the continuous check.
* Operational verification at random times outside the scoring cadence.

**Tradeoffs.** Randomized cadence reduces the value of timing data the scoring engine collects. Continuous availability checks increase scoring infrastructure load. Operational verification is bandwidth limited.

## Hardening practices that require comprehensive scoring

Patterns in this category reduce attack surface in ways that may be sensible in production when operational continuity is preserved, but in a competition setting create scoring blind spots when scoring is narrow. The cheese is in how the hardening is done: in ways that pass narrow scoring while breaking workflows the system is supposed to support. Each entry below documents the behavior, then describes what comprehensive scoring should exercise so that operational reality is visible and rewarded.

### 6. Removing operational tooling

**What it looks like.** Removing or restricting standard system commands, shells, and binaries that admins, scheduled tasks, or other services need to operate on the system. Variants: deleted shells, removed binaries (network tools, compilers, scripting interpreters), restricted execution via noexec mount or LSM policies (AppArmor, SELinux), restricted or emptied environment variables that programs depend on.

**Real world analog.** A locked down kiosk image. On a general purpose server, the same restrictions break workflows the system needs. Attack surface reduction is good practice in production when the system's operational requirements are still met.

**What comprehensive scoring should exercise.**

* Operational verification of admin tasks: log in, run package updates, inspect logs, check service status, restart a service, modify a configuration. If the admin workflow fails, the team learns immediately that something they did broke operational reality.
* Injects requiring tooling whose effect must be subsequently verified (install a monitoring tool the scoring engine then checks remains functional).
* Scoring engine probes that exercise scored services through internal mechanisms which depend on common tooling.

**Tradeoffs.** Operational verification of admin workflows is bandwidth limited but the most authentic signal. Injects test at a point in time and miss state that gets enabled briefly. Internal probes raise scoring infrastructure complexity. Without these, broad tool removal passes narrow scoring while breaking workflows real users would notice.

### 7. Removing scheduled task subsystems

**What it looks like.** Disabling or removing the system's scheduled task subsystems (cron, at, systemd timers) on the theory that attackers use them for persistence. Variants: removing the binaries, masking the systemd units, emptying configuration directories, killing the daemons.

**Real world analog.** A server with cron disabled because no application currently uses it. Defensible only when the system genuinely has no scheduled work, since log rotation, package updates, certificate renewal, backup jobs, and other routine maintenance all rely on it.

**What comprehensive scoring should exercise.**

* Environment design that builds in dependencies on scheduled work. A noisy service that fills the disk without log rotation, a service that requires periodic certificate renewal, a database that needs scheduled vacuum. The team feels the consequence of disabling cron immediately rather than waiting for a probe.
* Injects requiring evidence of scheduled work as cumulative output (a final inject asking for backup history, log rotation evidence, or scheduled report runs across the competition window).
* Operational verification of scheduled maintenance: check log rotation timestamps, certificate expiry, scheduled report output across the competition window.

**Tradeoffs.** Built in environment dependencies require design effort upfront but produce continuous, immediate feedback. Injects requiring cumulative evidence work but require careful inject design. Operational verification of scheduled maintenance requires extended observation windows. Without these, broad cron disabling passes narrow scoring while breaking maintenance workflows that real systems would notice.

### 8. Restricted DNS resolution

**What it looks like.** Disabling or restricting DNS resolution from team systems on the theory that attackers need DNS for command and control or data exfiltration. Variants: empty resolver configuration, resolver pointed at non functional servers, firewalled outbound DNS, disabled local resolver caches.

**Real world analog.** A production server with DNS broken. Updates fail, NTP fails, certificate validation fails, anything by hostname fails. A real ops team notices within minutes.

**What comprehensive scoring should exercise.**

* Score services by DNS name rather than IP, so resolution failures fail the check directly.
* Environment design with inter service dependencies that exercise transitive name resolution: services that look up other services by name to function (a web app that connects to its database by hostname, a mail server that does MX lookups, federated trust that requires DNS based discovery).
* Operational verification of DNS dependent workflows (package updates, certificate validation by hostname).

**Tradeoffs.** Name based scoring is straightforward to set up. Transitive dependencies require environment design effort. Operational verification is bandwidth limited. Without these, broad DNS disabling passes narrow probes while breaking workflows that depend on resolution.

### 9. Restricted interactive login surface

**What it looks like.** Restricting who can obtain an interactive shell or session on the system in ways that satisfy a basic authentication probe but break legitimate admin and user workflows. Variants: setting all or most account shells to /bin/false or /sbin/nologin, removing login shells from /etc/shells, locking accounts at the password level, configuring PAM modules (pam_access, pam_listfile, pam_succeed_if, pam_time) to allow scoring's authentication and command execution but deny full session establishment, restricting SSH via ForceCommand to a single command and exit, isolating sshd in a container with no access to the underlying system.

**Real world analog.** A bastion host configured with ForceCommand for git over SSH, or service accounts with /sbin/nologin shells. Both are standard practice when constrained access is the entire intended use. Applied to accounts that legitimately need interactive access (admins, operators, anyone whose role includes logging into the system), the same restrictions become a pattern that breaks operational reality.

**What comprehensive scoring should exercise.**

* Scoring engine probes against SSH and other interactive services that issue a real command after authentication and verify expected output, not just authentication success.
* Operational verification by simulated admins logging in interactively and running expected administrative tasks (rotate a password, inspect logs, install a tool, restart a service).

**Tradeoffs.** Real shell session probes raise check complexity and increase false positive rate when sessions terminate naturally. Operational verification is reliable but bandwidth limited. Without these, restricting interactive access passes basic auth probes while breaking the admin and user workflows real systems require.

### 10. Password rotation gaming

**What it looks like.** The team rotates user passwords on scored services in ways that satisfy a narrow probe but break operational reality. Variants: rapid rotation timed faster than the scoring cadence so probes catch only a brief valid window; convergence to a single shared password across all users for ease of rotation; mass changes that break automation, integrations, or downstream services that depended on the previous credentials.

**Real world analog.** Mass password resets after a confirmed breach are standard practice. Done preemptively and continuously regardless of evidence, or in ways that compromise security further (a single shared password across users defeats the entire purpose of distinct credentials), the same mechanism produces no security gain.

**What comprehensive scoring should exercise.**

* Scoring engine probes that authenticate as individual users with their assigned credentials at intervals. If rotation is rapid enough that scoring credentials become invalid between checks, scoring fails.
* Operational verification by simulated users authenticating with their assigned credentials and performing user level work (read mail, log in, access files). Rotation that breaks user authentication breaks the operational role.
* Explicit rules around password rotation cadence and justification (mass changes beyond a threshold require written justification, single shared password configurations prohibited).
* Password change request (PCR) coordination: when teams change passwords on scored services, they submit the new credentials through the scoring engine's PCR page so the scoring engine can authenticate with current credentials. Rotation without PCR submission breaks scoring authentication.

**Tradeoffs.** Per user authentication probes require the scoring engine to track and use distinct credentials per team and per user. Operational verification of user authentication is bandwidth limited. Explicit rules require enforcement and adjudication. PCR coordination puts the burden on the team to keep the scoring engine current. Missed PCRs cause scoring failures that may look like cheese but are actually operational mistakes.

### 11. Workstation total neutering

**What it looks like.** Stripping a system that exists in the simulation to be a user workstation of all user functionality: removing or disabling the browser, document tools, email client, desktop applications, file share access, and other elements that make it usable as a workstation. Common motivation: the workstation is a soft target for red team and is not directly scored as heavily as servers, so removing user functionality reduces attack surface without obvious penalty.

**Real world analog.** Developers and security engineers run services on their workstations all the time, and turn off applications they do not use. The cheese is total stripping of user functionality from a system whose role in the simulation is to be a user, with no corresponding business case to explain the change.

**What comprehensive scoring should exercise.**

* Operational verification that simulates the user the workstation belongs to: log into the desktop session, open the browser to a specific page, open a development environment and compile something, open a spreadsheet from a file share and edit it, open the email client and send a message that arrives at a verifiable destination.
* Scoring engine probes for the presence of expected user environment elements: installed applications, browser profiles, document folders, configured email client, file share mounts.

**Tradeoffs.** Operational verification of user behavior is the most reliable signal but requires scripts or human interaction simulating users. Environment presence probes can be partially gamed if the team retains the appearance without the substance. Without these, gutting workstations passes basic system level probes while removing the soft targets the simulation depended on.

## Network layer denial

Patterns in this category restrict network traffic flow on the theory that attackers need network access to operate. Like the hardening patterns above, these reductions of attack surface may be sensible in production with proper operational continuity preserved, but in a competition setting can pass narrow scoring while breaking workflows that depend on network connectivity. The design responses below describe what comprehensive scoring should exercise to ensure network reality is visible.

### 12. Blanket egress restriction

**What it looks like.** The team blocks all or most outbound network traffic from team systems on the theory that attackers need outbound channels for command and control or exfiltration. Variants: default deny outbound firewall with narrow allowlist for scoring sources, blocking specific protocols (DNS, HTTP, SSH outbound), blocking specific destinations.

**Real world analog.** An air gapped or network isolated production environment. Defensible only when the systems genuinely have no outbound dependencies, since updates, NTP, DNS, license validation, telemetry, certificate validation, and most other operational needs require some kind of egress.

**What comprehensive scoring should exercise.**

* Score outbound functionality from team systems: NTP synchronization, certificate revocation checks, DNS resolution to external hosts, package update connectivity.
* Environment design with outbound dependencies built in: services that require periodic external authentication, certificate renewal that requires CA contact, package management that requires repository access.
* Operational verification that simulated users can browse the internet, send mail externally, and use cloud services as expected.

**Tradeoffs.** Outbound scoring requires the scoring engine to test connectivity from each team system. Built in dependencies require environment design. Operational verification of internet use is bandwidth limited. Without these, broad egress denial passes narrow scoring while breaking workflows that depend on external connectivity.

### 13. Connection time and rate limits

**What it looks like.** The team applies firewall or service level rules that terminate connections after short time windows or aggressively rate limit sources. Variants: blocking connections that exceed a few seconds duration, terminating long running sessions, dropping requests above a low rate threshold per source.

**Real world analog.** Production systems use connection timeouts and rate limiting routinely (DoS protection, slow loris mitigation, fair queuing). Set aggressively enough to prevent normal operational use, the same controls catch sessions that should last hours and treat normal user request rates as abusive.

**What comprehensive scoring should exercise.**

* Score sessions that require sustained duration: long running SSH sessions executing real commands, file transfers that take time, database queries that return large result sets, video or audio streaming.
* Score request patterns at realistic user rates: bursts of legitimate activity, a user clicking through a web app, a developer making API calls.
* Operational verification of admin workflows that require multi step interactive sessions.

**Tradeoffs.** Sustained session scoring requires the scoring engine to maintain stateful connections, raising complexity. Realistic rate testing requires the scoring engine to send traffic at user like rates rather than uniformly. Without these, aggressive time and rate limits pass single request scoring probes while breaking real user workflows.

## Recovery and rollback gaming

Patterns in this category use system state restoration mechanisms to undo red team intrusions mechanically rather than through detection and remediation. Snapshot revert, image rebuild, and similar mechanisms are legitimate recovery tools after a confirmed incident. The cheese version applies them automatically or on a schedule, eliminating the incident response work the competition is designed to teach.

### 14. Mechanical state rollback

**What it looks like.** The team configures system state to revert to a known snapshot or image without engaging with the incident response work that real recovery requires. Variants: automation triggered by intrusion detection signals (scripts triggered by file integrity monitoring alerts, watchdog services that compare running state against a baseline), scheduled rollback regardless of evidence (cron jobs that revert to a snapshot every hour, image redeployment timed to scoring cadence).

**Real world analog.** Immutable infrastructure with rebuild on anomaly is real in cloud ops, and Kubernetes self healing is appearing in CCDC environments. These work because rebuilds are paired with detection, investigation, and root cause work, and because the operational cost (complexity, expertise, infrastructure) is significant. A script that auto reverts on alert mimics the surface behavior without the engineering depth that makes it function in production. The scheduled variant has no production analog at all, representing a misunderstanding of rollback as defense rather than recovery.

**What comprehensive scoring should exercise.**

* **Explicit point penalty per revert above a free threshold.** A free budget of reverts for legitimate use during the competition. Reverts beyond that incur direct point cost. This is the primary deterrent in environments where the competition controls the underlying infrastructure.
* **Environment design where blue teams own their own infrastructure.** When the team manages its own snapshots and configurations, a revert erases all defensive work the team performed (hardening, patches, custom configurations). Red team can then compromise the freshly reverted state again, producing direct point loss through compromise penalties. The natural consequence makes the deterrent intrinsic rather than rule based.
* **Environment design with continuously accumulating state.** Scored services should accumulate operational state during the competition that mirrors real production usage: emails sent and received in the mail server, customer purchases in the ecommerce database, file updates in the document store, log history that builds over time. When a team reverts, this state is lost, producing immediate visible consequence: an orange team simulated user notices their recent email is missing, a customer record from earlier in the competition has vanished, a file the team modified is back to its original state. The natural consequences mirror what would happen in production, where customers would notice and complain.
* Inject deliverables that require artifacts produced between intrusion and revert (incident report describing what red team did, evidence of investigation, remediation plan). Reverting before producing these loses the deliverable.
* Operational verification of state changes that should persist (configurations the team made, scheduled work output, deliverables in progress). Rollback undoes those.
* Detection of revert patterns: when state returns to identical bytes of a known snapshot multiple times in succession, flag for inspection.

**Tradeoffs.** Explicit revert penalties require accurate revert detection across the environment and an agreed free threshold. Blue team owned infrastructure pushes the deterrent into the environment but requires teams to do the infrastructure work. Continuously accumulating state requires environment design effort to generate state realistically and scoring infrastructure to verify it. Inject deliverables tied to incidents require the inject schedule to be unpredictable. Operational verification of persistent state requires environment design choices about what should remain. Detection of revert patterns requires snapshot fingerprinting infrastructure.

## Inject gaming

Patterns in this category satisfy the formal requirements of an inject (a PDF deliverable submitted through the inject portal) without doing the underlying work. The team produces a document that looks complete but lacks the analysis, environment specific information, or operational understanding the inject was meant to require.

### 15. Boilerplate or generated inject responses

**What it looks like.** The team produces inject deliverables that satisfy the formal requirements (a PDF of approximately the right shape, submitted on time) without doing the underlying work. Variants: copy pasted templates from prior competition years, generic policy boilerplate downloaded from compliance template sites, AI generated documents that look complete but contain generic content or hallucinated specifics, network inventories that do not reflect the actual environment, incident reports that describe generic attacks rather than what the red team actually did.

**Real world analog.** Compliance theater. Organizations that maintain documented policies and procedures for the auditor but never apply them operationally.

**What comprehensive scoring should exercise.**

* Inject grading by humans who can detect generic content versus original work that reflects the team's actual environment. Templates that provide formatting and structure are legitimate efficiency. Templates that provide the substantive content are the cheese form. Grading should distinguish between the two by requiring environment specific content even within templated structure. Subjectivity is partially mitigated by objective rubrics per inject and grading each inject by at least two humans.
* Inject content that requires environment specific information (the network inventory must reflect this team's specific systems and addresses, the incident report must describe what red team actually did at this team's site).
* Cross referencing inject content against environment state (for example, an AUP that requires multi factor authentication while the team's systems do not have it configured should flag the contradiction).
* Follow up injects that build on previous inject responses (an inject asks the team to implement what they wrote in their incident response plan from an earlier inject).
* Injects that require demonstration through application rather than description (do not write a policy on patching, patch the systems and produce evidence).

**Tradeoffs.** Human grading is bandwidth limited. Environment specific content requires injects designed around team specific data, raising inject preparation effort. Cross referencing requires the scoring infrastructure to have visibility into team environment state. Follow up injects require schedule design that allows building. AI detection is unreliable and the gap is closing rapidly.

## Before competition prep gaming

Patterns in this category move work from inside the competition window to before T0, in ways that bypass the analytical and operational work the competition is designed to teach. The competition tests how teams respond to an unfamiliar environment under pressure. Pre work that obviates that response defeats the test.

### 16. Replacement images deployed at T0

**What it looks like.** The team prepares replacement systems before the competition starts (containers, VMs, full OS images with services preconfigured) and deploys them at T0 to swap out the given environment. Variants: containerized service replacements, full VM swaps, kickstart or cloud init configurations that rebuild systems on first boot.

**Real world analog.** Infrastructure as code with declarative deployment is normal production practice. Used to skip the work of analyzing the given environment, that practice defeats what the competition was designed around: figure out what is here, find what is wrong, decide what to fix.

**What comprehensive scoring should exercise.**

* **Heavier SLA penalty schedule early in the competition.** The first hours carry larger SLA violation penalties than later periods. This directly catches teams whose T0 image swap causes service outages during the transition: the moment of swap is exactly when the team is most likely to break things, and it is exactly when the penalty hurts most.
* **Interconnected services.** Scored services depend on each other (the web app needs the database, the mail server needs DNS, the file share needs authentication). If one service goes down during a transition, others fail too. The cost of a botched swap multiplies through the dependency graph.
* **Vulnerabilities at both application and operating system layers.** The given environment includes intentional vulnerabilities at both the OS layer (default credentials, unpatched packages) and the application layer (web app bugs, misconfigurations). Migrating the OS does not fix the application vulnerabilities, so teams that swap the OS still face the application bugs they did not investigate.
* Inject deliverables that require analysis of the given environment (the network inventory inject is impossible to complete with a replacement image without doing the analysis first).
* **Forensic injects that depend on system state.** Injects ask the team to identify and document anomalies on the systems: prepositioned forensic artifacts placed in the environment before the competition (anomalous log entries, unusual files, indicators of prior compromise) and red team activity artifacts generated during the competition (evidence of attacks, persistence indicators, modified configurations). A T0 image swap loses the prepositioned artifacts. A mid competition migration loses the red team evidence. Either way, the forensic injects become impossible to complete.
* Environment design that includes services and configurations that cannot be trivially replicated in advance (custom internal applications, environment specific data, integrations that depend on the given systems).

**Tradeoffs.** Heavier early SLA penalties require schedule design so legitimate environment exploration is not over penalized. Environment design choices (interconnection, multi layer vulnerabilities, non replicable elements) require significant upfront investment but produce structural deterrents that work without inspection. Analysis and forensic injects require coordination between environment design (prepositioned artifacts), red team operations (artifact generation), and inject schedule (asking for documentation at the right times).

### 17. Prepared hardening automation

**What it looks like.** The team prepares scripts before the competition that run at T0 (or shortly after) to harden systems automatically: change all default passwords, disable all non essential services, apply firewall rules, install monitoring tools, and so on. Scripts may be invoked by hand or set to run on boot.

**Real world analog.** Configuration management with idempotent hardening playbooks is standard production practice. Run before any analysis of what is actually present, the same playbooks violate Chesterton's fence: the team removes things they have not investigated and may need.

**What comprehensive scoring should exercise.**

* Inject deliverables that require analysis of the existing environment before changes (the team must produce a network inventory or systems inventory before they are credited for hardening work).
* Continuous scoring of services that blanket hardening would commonly break, so running prepared scripts produces immediate scoring loss.
* Operational verification of workflows that prepared scripts would interfere with (if the script disables a service the system needs, the dependent workflow fails immediately).
* Environment design with non obvious dependencies that prepared scripts would not anticipate (services that depend on each other in unusual ways, custom configurations the team must understand to preserve).
* **Heavier SLA penalty schedule early in the competition.** As with pattern 16, the early window is when T0 hardening scripts are most likely to cause outages, and the penalty schedule makes those outages costly.

**Tradeoffs.** Analysis injects require schedule design that delivers them early enough to be a gate. Continuous scoring of breakable services is the mainline scoring engine work. Operational verification depends on the workflows being non trivial. Environment design with non obvious dependencies is the deepest investment but the most effective. It punishes prepared scripts directly through immediate consequence.

## Coordination gaming

This category covers patterns where the team uses resources outside the formal competition framework: external communication, third party help, sharing between teams, or other coordination that gives the team capabilities not granted by the competition structure.

### 18. External assistance and inter team coordination

**What it looks like.** The team obtains help during the competition that is outside the formally permitted resources: communicating with coaches, alumni, or industry contacts via phone, chat, or in person; using social media or chat platforms to ask for solutions; coordinating with other teams to share approaches or solutions; consulting with experts who are not on the team roster.

**Real world analog.** In production, asking colleagues, vendor support, or online communities for help is normal practice. In a competition specifically designed to test the team's own capability under time pressure with limited resources, the same outside help defeats the test.

**What comprehensive scoring should exercise.**

* Explicit rules defining what communication is permitted (typically: only with white team, orange team, and operations during the competition; no contact with anyone outside the competition framework).
* Physical isolation of the competition space (a controlled room with no personal devices, monitored entry, and on site observation).
* Network isolation of competition systems from the open internet, with permitted resources (documentation sites, package mirrors) accessible through monitored proxies.
* White team observation in the competition area, watching for prohibited communication.
* Logging and inspection of any devices the team brings into the competition area.

**Tradeoffs.** Physical and network isolation requires venue infrastructure that not all events can provide. White team observation is bandwidth limited and intrusive. Device inspection is invasive and slow. Permitted resource lists require maintenance and create the question of where to draw the line (using AI for syntax help versus using AI to write the entire incident report). The harder the isolation is, the less the competition resembles real ops work where consultation is normal. The more open the access, the more advantage accrues to teams with rich support networks outside the room.

## What is not on this list and why

Some practices look like the patterns above but are legitimate hardening rather than cheese. Distinguishing them matters. A practice qualifies as legitimate hardening if it passes both of the following tests:

1. A competent ops team would make this change in a comparable production environment.
2. The change does not leave any of the following broken: desktop and workstation use, admin workflows, incident response capability, forensic recovery capability, business continuity, normal user productivity.

A practice that passes both is hardening. A practice that fails either is cheese (or, in some cases, simple bad practice).

The practices below are commonly confused with the cheese patterns but pass the test. They should not be penalized:

* **Restricting DNS zone transfers (AXFR / IXFR) to specific secondary servers.** Standard BIND security guidance and CIS benchmark for any authoritative DNS server.
* **Disabling DNS recursion on authoritative only servers.** A server whose only role is authoritative answers should not also resolve external names.
* **Disabling SMTP VRFY and EXPN.** Both commands enable user enumeration and have no operational use. Standard hardening.
* **Removing development tools from production servers** (compilers, scripting interpreters not used by production services, network diagnostic utilities). CIS benchmark guidance.
* **Mounting /tmp and /home with noexec.** CIS benchmark recommendation for shared and user writable filesystems.
* **Disabling unused PAM modules.** If the system has no Kerberos KDC, removing pam_krb5 is hardening, not cheese. Same for LDAP modules without a directory, and so on.
* **Disabling NetworkManager on static IP servers.** Servers that do not change network configuration do not need a network configuration daemon.
* **Disabling systemd resolved when the system uses direct /etc/resolv.conf.** Reduces attack surface without breaking name resolution.
* **TLS version floor at 1.2 or 1.3.** Disabling TLS 1.0 and 1.1 is widely accepted security practice. The cheese version would be requiring something the scoring engine does not support, but the floor itself is legitimate.
* **Disabling auditd, syslog, or journald** produces no positive scoring outcome, which is why it is not on the cheese list. It belongs in the bad practice category (self inflicted blindness, broken incident reporting capability), just not the kind of bad practice this taxonomy covers.
* **Migration to a different operating system or software stack** with proper analysis, change management, and operational continuity is legitimate engineering. The cheese version (T0 swap to bypass analysis) is covered by pattern 16. Migration done correctly is not on this list.
* **Kubernetes self healing and immutable infrastructure with proper engineering depth.** Real production patterns increasingly seen in CCDC environments. They work because the operational cost (complexity, expertise, infrastructure) is significant and the rebuild is paired with detection, investigation, and root cause work. The cheese version (a script that mimics self healing without the engineering) is covered by pattern 14. The legitimate version belongs here.

A separate category of context dependent cases requires more judgment:

* **IPv6 disabled.** Legitimate if IPv6 is not part of the environment's role. Cheese if the environment includes IPv6 services.
* **STARTTLS disabled on SMTP.** Legitimate if mail is restricted to authenticated submission on a TLS only port. Cheese if the environment expects opportunistic encryption for legacy clients.
* **Capability stripping on binaries.** Legitimate for binaries that are not used. Cheese if the binary is needed.
* **System wide umask 077.** Legitimate for single tenant servers. Cheese for shared environments.
* **Disabling dbus.** Legitimate on headless servers without GUI or desktop dependencies. Cheese where polkit, NetworkManager, or other dbus consumers are needed.
* **Inbound port blocking as defense against specific exploits.** Legitimate defense in depth (CIS benchmarks recommend it for many service types). Becomes cheese only if used as a complete substitute for addressing the underlying vulnerability and the service remains vulnerable internally. The line is whether the team has done the work to reduce internal exploit risk in addition to blocking external access.

For the context dependent cases, the test is the same: does the change break a workflow the system needs? Apply it case by case.

With comprehensive scoring (operational verification, environment design with real dependencies, scoring engine probes that exercise full functionality), correct hardening is rewarded directly. The team that hardens without breaking anything passes scoring. The team that hardens broadly without analysis fails immediately. Without comprehensive scoring, the cheese patterns and legitimate hardening look indistinguishable from outside, and the practitioners doing the right thing get unfairly grouped with the practitioners gaming the score. The patterns and design responses in this piece are about closing that gap.

## Closing

The patterns in this taxonomy share a structure. Each one allows a team to pass the scoring probe without doing the operational work the probe was supposed to detect. The design responses share a structure too. Each one closes the gap between what scoring measures and what the team is actually doing.

Picking the right design response for each pattern is an engineering decision. The tradeoffs are real and unavoidable: coverage versus complexity, automated versus human verification, prevention versus detection. No response covers every pattern, and most patterns need a combination to address well. Competition designers should choose explicitly rather than letting the choice happen by default.

Comprehensive scoring is expensive. Operational verification needs humans. Environment design needs design effort. Inject content needs preparation. Scoring engine probes that exercise full functionality need engineering investment. Every competition makes its own decisions about how much of this expense to take on, given its resources and goals.

Returning to Goodhart's Law: when a measure becomes a target, it ceases to be a good measure. The patterns above are how that principle plays out in cyber defense competition. The design responses are how organizers push back. Neither side wins permanently. Teams find new patterns; organizers add new responses. The competition stays educational only as long as the responses keep up with the patterns.

The taxonomy here is a snapshot. Patterns will evolve. Responses will follow. The work is ongoing.
