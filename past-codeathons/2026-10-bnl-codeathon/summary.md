# Summary: 2026 October Codeathon (BNL)

EPICS Developers Meeting and Codeathon, hosted by the Electron-Ion Collider (EIC) project at BNL, 19–23 October 2026. This page covers the Phoebus tools and services track.

- **Part 1, Briefing:** a short overview of the codeathon, to present.
- **Part 2, Decisions and actions:** the list to review at the follow-up meeting.

Full notes are in the [minutes](minutes.md). See also the [project results on the wiki](https://github.com/epics-base/epics-base/wiki/2026%E2%80%9010-Codeathon-Project-Results) and the [event page](https://github.com/epics-base/epics-base/wiki/2026-EPICS-Developers-Meeting-and-Codeathon-hosted-by-EIC). GitHub status was checked on 12 Nov 2026.

## Contents

- [Part 1: Briefing](#part-1)
  - [A week of hands-on work, not presentations](#b-week)
  - [Five things to take home](#b-takeaways)
  - [Platform modernization](#b-platform)
  - [Web: one PVWS going forward](#b-web)
  - [Alarms](#b-alarms)
  - [Archiver Appliance](#b-archiver)
  - [ChannelFinder, RecSync and PV name resolution](#b-cf)
  - [Olog, AI and agents](#b-olog)
  - [Save-and-restore and Display Builder](#b-sar)
  - [Where help is needed](#b-help)
- [Part 2: Decisions and actions](#part-2)
  - [0. Already done since the codeathon](#done)
  - [1. Key decisions](#decisions)
  - [2. Actions without an owner](#unowned)
  - [3. Actions with owners](#owned)
  - [4. Not decided](#undecided)
  - [5. Quick facts to confirm](#facts)

## <a id="part-1"></a>Part 1: Briefing

### <a id="b-week"></a>A week of hands-on work, not presentations

| 37 | 14 | 7 | 5 |
|:---:|:---:|:---:|:---:|
| attendees | organizations | countries | days |

- **Monday:** a joint session to present the discussion topics, the major projects and the mentors.
- **Tuesday to Thursday:** two parallel tracks, EPICS Base and Phoebus. Discussions were mostly in the morning, project work in the afternoon.
- **Friday:** a common wrap-up.
- **Lunches:** working lunches on security, DevOps and CI/CD, and AI/ML.

---

### <a id="b-takeaways"></a>Five things to take home

1. **Platform:** Phoebus and its services are moving to **Java 25 and Spring Boot 4**.
2. **Web:** there is now **one maintained PVWS**.
3. **Services:** facilities **shared their workflows** for alarms, the archiver, save-and-restore and ChannelFinder, and agreed on a common direction.
4. **Review is the bottleneck.** Phoebus merges quickly: 8 of the 11 PRs opened during the week were merged by 9 November. EPICS Base has 17 of its 18 new PRs still open.
5. **Practices are converging:** default ports, CI checks and contributor guidance.

---

### <a id="b-platform"></a>Platform modernization

- **Java 25 and Spring Boot 4** across the services. PVWS has already moved; the other services follow.
- **Kafka 4.3.1** client libraries in the alarm services; **ZooKeeper removed** from the Docker setup. Both are done.
- **Smaller, regular dependency upgrades** instead of occasional large jumps.
- **Sequential default ports** (8080, 8081, …) so that all the services can run on one host, as on a training VM. PRs are open in four repositories.
- **CI flags formatting problems but never rewrites code.** Whether to use prek or pre-commit is still open.

**More in the minutes:** [Oct 21 working session](minutes.md#topic-16) · [Oct 22 morning session](minutes.md#topic-17) · [Default ports](minutes.md#topic-18)

---

### <a id="b-web"></a>Web: one PVWS going forward

- There were two forks of PVWS. **The Spring Boot version is the maintained one**, and it is now simply called "PVWS".
- **Done:** Spring Boot 4 and Java 25, synced with the ORNL code base, Docker image updated.
- **Next:**
  - security requirements (ITER drafting);
  - authentication;
  - CORS (PR open);
  - test reports from ORNL, DLS, Berkeley and CEA.
- **Still open:** which web runtime the community backs, DBWR or Daedalus, and who maintains DBWR.

**More in the minutes:** [Web toolkits and PVWS](minutes.md#topic-2) · [Oct 20 working session](minutes.md#topic-15)

---

### <a id="b-alarms"></a>Alarms

- **Edit and run modes for the alarm tree** are in progress (ISIS and ESS; several PRs are open). They will be merged only after FRIB and Oak Ridge agree.
- **"Mode-dependent alarms" are three separate problems:**
  - static configuration swaps, which already work today;
  - Save & Restore for slow changes;
  - IOC logic for fast changes.
- The alarm server must **log a lost Kafka connection or a deleted topic as an error**, and should send health notifications.
- **Still open:** who may acknowledge alarms or change the configuration, and how to support multiple alarm hierarchies over the same PVs.

**More in the minutes:** [Alarm services](minutes.md#topic-3) · [Oct 21 working session](minutes.md#topic-16)

---

### <a id="b-archiver"></a>Archiver Appliance

- **Parquet storage:**
  - FRIB measured about **50% less storage** on 800k PVs; the saving depends on how noisy the data is.
  - ZSTD compresses best and does not slow down retrieval.
  - Parquet is used only for the **final storage tier**; the short- and medium-term stores stay in protobuf.
- **PRs submitted:**
  - user tags for all DBR types;
  - LTTB downsampling, which returns real samples;
  - faster queries for disconnected PVs;
  - cluster split-brain handling.
- **Next:**
  - batch versions of more management calls, starting with `archivePV` (a good starter project);
  - a Parquet migration guide.

**More in the minutes:** [Archiver Appliance](minutes.md#topic-1) · [LTTB discussion](minutes.md#topic-15)

---

### <a id="b-cf"></a>ChannelFinder, RecSync and PV name resolution

- **RecSync:**
  - a new *undefined* channel state;
  - unicast address lists for RecCaster;
  - a clear versioning rule: major = protocol, minor = implementation.
- **RecCeiver** start-up and shutdown cleanup is merged. **PVInfo 3.0.0** is released.
- **ChannelFinder REST:** new endpoints designed for batch operations and for property and tag metadata.
- **Direction:** use the ChannelFinder directory and a CA name server for scalable PV name resolution. So far this is an architecture discussion; nothing has been built yet.

**More in the minutes:** [RecSync and ChannelFinder](minutes.md#topic-7)

---

### <a id="b-olog"></a>Olog, AI and agents

- **Olog AI search:** deploy it together and document it.
- **Python client and migration tool** move into the Olog GitHub organization.
- **The legacy Olog** needs a retirement plan (NSLS-II and FRIB).
- **Agentic use (MCP):**
  - design the middle-layer service APIs with agents in mind;
  - ideas include an Olog MCP server and an Osprey logbook adapter;
  - invite the Osprey developer to give a talk.

**More in the minutes:** [Olog](minutes.md#topic-6) · [Agentic frameworks](minutes.md#topic-5)

---

### <a id="b-sar"></a>Save-and-restore and Display Builder

- **Save-and-restore:**
  - restore policy (sync or async) and a case-insensitive filter, both merged;
  - the restore mode becomes an administrator setting, and service restore requires authentication.
- **Colors:** first one central, named color set; then opt-in alarm sensitivity for individual widgets.
- **Tabs widget:** a new *Default Tab* property; *Active Tab* becomes runtime state only.
- **Launch and fixes:** opt-in `-force_start` and a JavaFX 25 scroll-bar fix, both merged.
- **Display tooling:** phoebusgen v4, and phoebuslint, which found thousands of issues in Berkeley's displays.

**More in the minutes:** [Save-and-restore](minutes.md#topic-8) · [Tabs widget](minutes.md#topic-9) · [Alarm colors](minutes.md#topic-10) · [Missing -settings file](minutes.md#topic-12)

---

### <a id="b-help"></a>Where help is needed

- **[11 changes are already merged](#done)** since the event: PVWS, Kafka, save-and-restore, `-force_start` and RecCeiver.
- **[22 actions have no owner](#unowned)**, for example:
  - batch `archivePV`;
  - PVWS authentication;
  - Kafka reconnection tests;
  - ChannelFinder documentation;
  - column hiding in tables;
  - Docker Compose files for the full stack.
- **[9 questions are still open](#undecided)**, for example:
  - DBWR or Daedalus;
  - alarm authorization;
  - the end-of-support date for the legacy Olog;
  - prek or pre-commit.
- **[PRs need reviewers](#owned)**, for example:
  - alarm-tree edit mode;
  - default ports;
  - PVWS CORS;
  - truncated text values;
  - the Oracle UTC fix.

## <a id="part-2"></a>Part 2: Decisions and actions

For the follow-up meeting, skim section 0, then work through sections 1 to 4. Section 5 lists quick factual questions to settle by email. The numbers in the **Topic** column link to the topic in the [minutes](minutes.md).

### <a id="done"></a>0. Already done since the codeathon

| Item | PR | Topic |
|---|---|---|
| Opt-in `-force_start` when the `-settings` file is missing | [#3968](https://github.com/ControlSystemStudio/phoebus/pull/3968) (LordIdra), merged 26 Oct | [12](minutes.md#topic-12) |
| Save-and-restore restore policy, sync or async (JCA deadlock workaround) | [#3997](https://github.com/ControlSystemStudio/phoebus/pull/3997) (georgweiss), merged 2 Nov | [8](minutes.md#topic-8) |
| Case-insensitive filter in the snapshot view | [#3989](https://github.com/ControlSystemStudio/phoebus/pull/3989) (abrahamwolk), merged | [8](minutes.md#topic-8) |
| Kafka client libraries updated to 4.3.1 | [#3985](https://github.com/ControlSystemStudio/phoebus/pull/3985) (georgweiss), merged 26 Oct | [3](minutes.md#topic-3), [16](minutes.md#topic-16) |
| Docker setup: Kafka updated, ZooKeeper removed | [#3977](https://github.com/ControlSystemStudio/phoebus/pull/3977) (roycpro), merged 22 Oct | [3](minutes.md#topic-3) |
| Translation fixes | [#3982](https://github.com/ControlSystemStudio/phoebus/pull/3982) (lcaouen), merged 24 Oct | [16](minutes.md#topic-16) |
| Scroll-bar flicker fix re-applied for JavaFX 25 | [#3972](https://github.com/ControlSystemStudio/phoebus/pull/3972) (rjwills28), merged 22 Oct | [17](minutes.md#topic-17) |
| PVWS on Spring Boot 4 and Java 25 | [pvws#11](https://github.com/ControlSystemStudio/pvws/pull/11) (shroffk), merged 22 Oct | [2](minutes.md#topic-2), [15](minutes.md#topic-15) |
| PVWS Spring Boot version aligned with the ORNL ("legacy") code base | [pvws#12](https://github.com/ControlSystemStudio/pvws/pull/12) (georgweiss), merged 5 Nov | [2](minutes.md#topic-2), [16](minutes.md#topic-16) |
| PVWS Docker setup updated for Java 25 | [pvws#13](https://github.com/ControlSystemStudio/pvws/pull/13) (georgweiss), merged 2 Nov | [2](minutes.md#topic-2), [15](minutes.md#topic-15) |
| RecCeiver start-up and shutdown cleanup | [recceiver#7](https://github.com/ChannelFinder/recceiver/pull/7) (roycpro), merged 2 Nov | [7](minutes.md#topic-7) |

### <a id="decisions"></a>1. Key decisions (confirm)

| # | Decision | Topic |
|---|---|---|
| D1 | Archiver Parquet: keep STS and MTS in protobuf; use Parquet only at the last ETL step into the final tier. | [1](minutes.md#topic-1) |
| D2 | Extend JSON batch support to more archiver BPLs, including `archivePV`. | [1](minutes.md#topic-1) |
| D3 | LTTB in the archiver as a retrieval-only post-processor, scalar case first. PR open: [epicsarchiverap#563](https://github.com/archiver-appliance/epicsarchiverap/pull/563). | [15](minutes.md#topic-15) |
| D4 | PVWS: the Spring Boot version is the maintained one. It is renamed "PVWS", moves to Spring Boot 4 and JDK 25 (JDK 21 dropped), and its container uses a standard Java base image. **Mostly done** (see section 0). | [2](minutes.md#topic-2), [15](minutes.md#topic-15) |
| D5 | Alarm tree: add edit and run modes, based on Percy's prototype, before any tree-table rework. Merge only after FRIB and Oak Ridge agree. **In progress:** [#3991](https://github.com/ControlSystemStudio/phoebus/pull/3991), [#4007](https://github.com/ControlSystemStudio/phoebus/pull/4007) and related PRs by LordIdra are open. The original proposal ([#3920](https://github.com/ControlSystemStudio/phoebus/issues/3920)) was closed. | [3](minutes.md#topic-3) |
| D6 | Mode-dependent alarms are three problems: static configuration swaps (exist today), Save & Restore for slow changes, and IOC logic for fast changes. | [3](minutes.md#topic-3) |
| D7 | The alarm server must log a lost Kafka connection or a deleted topic as an error. Health notifications are wanted. | [16](minutes.md#topic-16) |
| D8 | Olog: deploy the AI search together and document it. Move the Python client and the migration tool into the Olog organization. The legacy Olog needs a sunset plan. | [6](minutes.md#topic-6), [15](minutes.md#topic-15) |
| D9 | Design the middle-layer service interfaces with agentic (MCP) use in mind, and invite the Osprey developer to give a talk. | [5](minutes.md#topic-5) |
| D10 | RecSync: add an *undefined* channel state; add unicast address lists to RecCaster with DNS re-resolution; versions are major = protocol, minor = implementation; a CA name server is needed. | [7](minutes.md#topic-7) |
| D11 | Save-and-restore: add a server switch to disable service restore. The restore mode becomes an administrator preference (default: client). Service restore requires authentication. Ordered restore uses a sequencer or staged configurations. Add column hiding. | [8](minutes.md#topic-8) |
| D12 | Tabs widget: add a new *Default Tab* property; *Active Tab* becomes runtime state only. This deliberately breaks compatibility. | [9](minutes.md#topic-9) |
| D13 | Colors: define one central, named color set first, then add opt-in alarm sensitivity to individual widgets. | [10](minutes.md#topic-10) |
| D14 | Truncated text ([#3928](https://github.com/ControlSystemStudio/phoebus/pull/3928)): the feature is wanted but needs fixes. An ellipsis must never be written to a PV. | [11](minutes.md#topic-11) |
| D15 | Missing `-settings` file: opt-in `-force_start`, with the default behaviour unchanged. **Done** ([#3968](https://github.com/ControlSystemStudio/phoebus/pull/3968)). | [12](minutes.md#topic-12) |
| D16 | Gateways: release the new CA gateway as CA Gateway version 3. Discourage bridging complex types from CA to PVA. | [13](minutes.md#topic-13) |
| D17 | VType `DescriptionProvider`: release it in a pre-release of epicsCoreJava first; the official release follows testing. | [14](minutes.md#topic-14) |
| D18 | Use sequential default ports (8080, 8081, …) for the services. PRs are open for ChannelFinder, Phoebus, Olog and PVWS (see section 3). | [18](minutes.md#topic-18) |
| D19 | CI flags formatting problems but never rewrites code. | [17](minutes.md#topic-17) |
| D20 | A blocking fix for the annunciator race condition ([#3966](https://github.com/ControlSystemStudio/phoebus/issues/3966)) is acceptable. | [18](minutes.md#topic-18) |

### <a id="unowned"></a>2. Actions without an owner (assign or drop)

| # | Action | Topic |
|---|---|---|
| U1 | Open a ticket for the consolidation problem when hold is greater than 11 (suggested by Anthony, FRIB). | [1](minutes.md#topic-1) |
| U2 | Batch `archivePV` and other BPLs (a good starter project). | [1](minutes.md#topic-1) |
| U3 | Announce the Spring Boot PVWS as the maintained version (ORNL or the collaboration). | [15](minutes.md#topic-15) |
| U4 | PVWS authentication (JWT plus user and password, behind an enable property). | [15](minutes.md#topic-15) |
| U5 | Verify Kafka reconnection in the alarm services and clients. The library upgrade is done ([#3985](https://github.com/ControlSystemStudio/phoebus/pull/3985)); the reconnection tests are not. | [3](minutes.md#topic-3), [16](minutes.md#topic-16) |
| U6 | Document how to handle alarm configuration changes at different rates. | [3](minutes.md#topic-3) |
| U7 | Health and connection-loss notifications for the alarm server. | [16](minutes.md#topic-16) |
| U8 | Investigate ghost alarms ([#3978](https://github.com/ControlSystemStudio/phoebus/issues/3978), reported by Loic). | [17](minutes.md#topic-17) |
| U9 | Olog MCP server or Osprey logbook adapter. | [5](minutes.md#topic-5) |
| U10 | Document the ChannelFinder ecosystem: plugins, info tags and allow-list (the volunteer's name was not captured). | [7](minutes.md#topic-7) |
| U11 | Document that QSRV2 sets both environment variables, and which PVXS release fixed it. | [7](minutes.md#topic-7) |
| U12 | Set an initial RecSync version and align the other components with it. | [7](minutes.md#topic-7) |
| U13 | Upgrade RecCeiver to JDK 25 (the volunteer's name was not captured). Possibly covered by [recceiver#8](https://github.com/ChannelFinder/recceiver/pull/8) "Spring4" (shroffk, open). | [17](minutes.md#topic-17) |
| U14 | Column hiding in the save-and-restore, ChannelFinder and alarm-log tables. | [8](minutes.md#topic-8) |
| U15 | Open an issue or PR for the *Default Tab* property. | [9](minutes.md#topic-9) |
| U16 | Once colors are consolidated, open an issue for alarm-sensitive widgets. | [10](minutes.md#topic-10) |
| U17 | Find an owner for the Oak Ridge CA/PVA bridge library, or discourage its use. | [13](minutes.md#topic-13) |
| U18 | Test PVWS-based tools against the VType pre-release. | [14](minutes.md#topic-14) |
| U19 | Review the JDK 25 documentation PR and remove the remaining JDK 21 references. | [15](minutes.md#topic-15) |
| U20 | Review the ISIS UI pull requests. | [15](minutes.md#topic-15) |
| U21 | Clean up the service Docker files and add compose files for the full stack and for the alarm services. | [16](minutes.md#topic-16) |
| U22 | Share the April DBWR vs cs-web-lib comparison. | [2](minutes.md#topic-2) |

### <a id="owned"></a>3. Actions with owners (quick status check)

| Owner | Action | Topic | Status (12 Nov) |
|---|---|---|---|
| Anthony Carriveau (FRIB) | PR adding a Parquet migration guide to the archiver docs | [1](minutes.md#topic-1) | |
| FRIB | Test Parquet in the ETL stage | [1](minutes.md#topic-1) | |
| Murali Shankar | Store the timestamp user tag in the archiver | [1](minutes.md#topic-1) | [epicsarchiverap#559](https://github.com/archiver-appliance/epicsarchiverap/pull/559) open |
| caraxlr | LTTB retrieval post-processor for scalars | [15](minutes.md#topic-15) | [epicsarchiverap#563](https://github.com/archiver-appliance/epicsarchiverap/pull/563) open |
| Murali Shankar | Update the Archiver Appliance in the Ansible role to 2.2.1 | [18](minutes.md#topic-18) | |
| Murali Shankar | Help implement the CA name server | [7](minutes.md#topic-7) | |
| ORNL, DLS, Berkeley, CEA (Rémi) | Test the Spring Boot PVWS and report | [2](minutes.md#topic-2) | |
| Loic Caouen (CEA) | Test PVWS against a soft IOC over CA and PVA | [16](minutes.md#topic-16) | |
| Loic Caouen (CEA) | Get permission to publish the alarm-web code | [2](minutes.md#topic-2) | |
| Loic Caouen (CEA) | List where colors are hard-coded or configurable | [10](minutes.md#topic-10) | |
| Loic Caouen (CEA) | Sequential default ports | [18](minutes.md#topic-18) | open: [CF#238](https://github.com/ChannelFinder/ChannelFinderService/pull/238), [#4006](https://github.com/ControlSystemStudio/phoebus/pull/4006), [olog#277](https://github.com/Olog/phoebus-olog/pull/277), [pvws#15](https://github.com/ControlSystemStudio/pvws/pull/15) |
| Martin Gaughran (DLS) | Share the PVWS stress tests and the display library link | [2](minutes.md#topic-2) | |
| Ralph Lange (ITER) | Draft PVWS security requirements | [2](minutes.md#topic-2) | |
| Ralph Lange (ITER) | Check whether the alarm handler can support ITER's filtering | [3](minutes.md#topic-3) | |
| Ralph Lange (ITER) | Three-chained-gateway reconnect test | [13](minutes.md#topic-13) | |
| Ralph Lange (ITER) | Introduce Ankit to ITER's new CS-Studio owner | [15](minutes.md#topic-15) | |
| Percy (ISIS), Georg Weiss (ESS) | Alarm tree edit and run modes | [3](minutes.md#topic-3) | [#3991](https://github.com/ControlSystemStudio/phoebus/pull/3991), [#4007](https://github.com/ControlSystemStudio/phoebus/pull/4007) and related PRs open |
| Georg Weiss (ESS) | Kafka 4 client update for the alarm services | [16](minutes.md#topic-16) | [#3985](https://github.com/ControlSystemStudio/phoebus/pull/3985) merged |
| Georg Weiss (ESS) | Kafka loss and topic-deletion tests; missing alarm-logger messages | [16](minutes.md#topic-16) | |
| Georg Weiss (ESS) | Ask ESS stakeholders about the restore-mode default and the quick filter | [8](minutes.md#topic-8) | |
| Jure Varlec (Cosylab) | Restore-deadlock reproducer and issue | [8](minutes.md#topic-8) | related: [#3983](https://github.com/ControlSystemStudio/phoebus/issues/3983) open |
| CEA | Share its alarm-disabling scripts | [3](minutes.md#topic-3) | |
| Kunal Shroff | GitHub discussion on alarm security expectations | [3](minutes.md#topic-3) | |
| Kunal Shroff, NSLS-II student, ESS | Deploy and document the Olog AI search | [6](minutes.md#topic-6) | |
| Ankit (NSLS-II), ESS | Olog Python client | [6](minutes.md#topic-6) | |
| NSLS-II, FRIB | Timeline for retiring the legacy Olog | [6](minutes.md#topic-6) | |
| Kunal Shroff; Seth Nemesure | Move the Olog migration tool and test it with e-log | [15](minutes.md#topic-15) | |
| Lucy (BNL) | Invite the Osprey developer; share slides | [5](minutes.md#topic-5) | |
| Kunal Shroff | Java RecCeiver quick start with a full configuration | [7](minutes.md#topic-7) | |
| Tynan Ford | RecCaster unicast mode | [7](minutes.md#topic-7) | [reccaster#24](https://github.com/ChannelFinder/reccaster/issues/24) open |
| Tynan Ford | PVWS CORS and Docker PRs | [17](minutes.md#topic-17) | CORS: [pvws#14](https://github.com/ControlSystemStudio/pvws/pull/14) open (opened by georgweiss) |
| Michael Davidsaver | Explore an EPICS module that publishes an IOC's PV interface | [4](minutes.md#topic-4) | |
| Kunal Shroff | Merge DescriptionProvider into a pre-release; NT `descriptor` ticket | [14](minutes.md#topic-14) | [epicsCoreJava#146](https://github.com/epics-base/epicsCoreJava/pull/146) open |
| Oak Ridge | Test DBWR against the VType pre-release | [14](minutes.md#topic-14) | |
| Oak Ridge | Second review of the Oracle UTC PR | [16](minutes.md#topic-16) | [#3905](https://github.com/ControlSystemStudio/phoebus/pull/3905) open |
| Oak Ridge | Contact for the ScanEngine | [16](minutes.md#topic-16) | |
| CEA demo participants | Edge cases and screenshots on PR #3928 | [11](minutes.md#topic-11) | [#3928](https://github.com/ControlSystemStudio/phoebus/pull/3928) open |
| Kunal Shroff | Ansible role and training VM | [18](minutes.md#topic-18) | |
| Andrew | Wiki page for the codeathon results | [18](minutes.md#topic-18) | |

### <a id="undecided"></a>4. Not decided (agree who drives each and by when)

| # | Question | Topic |
|---|---|---|
| N1 | DBWR or Daedalus as the community web runtime, and who maintains DBWR. | [2](minutes.md#topic-2) |
| N2 | Whether to merge PVInfo with the CEA alarm web. | [2](minutes.md#topic-2) |
| N3 | How to authorize alarm acknowledgements and configuration changes. | [3](minutes.md#topic-3) |
| N4 | How to support multiple alarm hierarchies over the same PVs. | [3](minutes.md#topic-3) |
| N5 | Whether to add partial restore of composite snapshots. | [8](minutes.md#topic-8) |
| N6 | Whether service restore should stay available to Phoebus clients. | [8](minutes.md#topic-8) |
| N7 | The end-of-support date for the legacy Olog. | [6](minutes.md#topic-6) |
| N8 | prek or pre-commit ([#3980](https://github.com/ControlSystemStudio/phoebus/issues/3980)). | [17](minutes.md#topic-17) |
| N9 | Whether to support a limited CA→PVA bridge for legacy systems. | [13](minutes.md#topic-13) |

### <a id="facts"></a>5. Quick facts to confirm (by email)

- Is the display library DLS mentioned Weiss? Is the DLS project in the screen-generation discussion ibek? [2](minutes.md#topic-2), [4](minutes.md#topic-4)
- Was the third site that relies on the `-settings` launch failure ITER? [12](minutes.md#topic-12)
- Which site hosts Mistral AI on site? Has NSLS-II moved to KRaft? Which facility is considering the ScanEngine? [6](minutes.md#topic-6), [3](minutes.md#topic-3), [16](minutes.md#topic-16)
- The link to the GitHub discussion "Color alarm sensitive widgets", and what the Oct 20 color discussion concluded. [10](minutes.md#topic-10)
- The timeline and testers for CA Gateway v3. Performance targets for the Java RecCeiver. [13](minutes.md#topic-13), [7](minutes.md#topic-7)
