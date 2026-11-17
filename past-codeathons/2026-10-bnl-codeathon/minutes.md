# Meeting minutes: 2026 October Codeathon (BNL)

Minutes of the 2026 EPICS Developers Meeting and Codeathon, BNL, Oct 19 to 23, 2026. They were compiled from the automatic transcripts of the Zoom recordings and from notes taken in the room on Oct 20. Times are local (US Eastern) and approximate. Names that were unclear in the recording are marked as uncertain.

**Status:** draft, under review. Covers Oct 20 to 22. Friday Oct 23 was not recorded. A briefing and the decisions, actions and open items are in the [summary](summary.md).

## When things were discussed

| Day | Scheduled discussion sessions | Discussions and notes from working time |
|---|---|---|
| Tue 20 Oct | [1](#topic-1) Archiver Appliance; [2](#topic-2) Web toolkits and PVWS | [15](#topic-15) Afternoon session |
| Wed 21 Oct | [3](#topic-3) Alarm services; [4](#topic-4) Screen auto-generation (coffee-break discussion); [5](#topic-5) Agentic frameworks; [6](#topic-6) Olog | [9](#topic-9) Tabs widget; [10](#topic-10) Alarm-sensitive widgets; [16](#topic-16) Afternoon session |
| Thu 22 Oct | [7](#topic-7) RecSync and ChannelFinder; [8](#topic-8) Save-and-restore | [11](#topic-11) Text truncation; [12](#topic-12) Missing `-settings` file; [13](#topic-13) Gateways; [14](#topic-14) VType description; [17](#topic-17) Morning session; [18](#topic-18) Afternoon session and wrap-up |

## Contents

- [Part 1: Discussion topics](#part-1)
  - [Topic 1: Archiver Appliance](#topic-1)
  - [Topic 2: Web toolkits, PVWS, DBWR, PVInfo, and web security](#topic-2)
  - [Topic 3: Alarm services](#topic-3)
  - [Topic 4: Auto-generating engineering screens (side discussion)](#topic-4)
  - [Topic 5: Agentic frameworks (Osprey) and the middle-layer services](#topic-5)
  - [Topic 6: Olog: AI search, Python client and the legacy Olog](#topic-6)
  - [Topic 7: RecSync, RecCeiver, RecCaster and the ChannelFinder ecosystem](#topic-7)
  - [Topic 8: Save-and-restore: facility workflows, UI, client vs service restore, restore ordering and deadlocks](#topic-8)
  - [Topic 9: Display Builder Tabs widget: Active Tab vs Default Tab](#topic-9)
  - [Topic 10: Alarm-sensitive widgets and alarm colors](#topic-10)
  - [Topic 11: Showing truncated values in Text Entry and Text Update (PR #3928)](#topic-11)
  - [Topic 12: Phoebus launch when the `-settings` file is missing](#topic-12)
  - [Topic 13: Gateways: PVA gateway, a new CA gateway, and CA/PVA bridging](#topic-13)
  - [Topic 14: VTypes: separating "description" from "display" (epicsCoreJava DescriptionProvider)](#topic-14)
- [Part 2: Codeathon working-session notes](#part-2)
  - [Topic 15: Afternoon working session (Oct 20)](#topic-15)
  - [Topic 16: Afternoon working session (Oct 21)](#topic-16)
  - [Topic 17: Morning working session (Oct 22)](#topic-17)
  - [Topic 18: Afternoon working session and wrap-up (Oct 22)](#topic-18)

## <a id="part-1"></a>Part 1: Discussion topics

Discussions of the services, areas and design decisions covered at the codeathon. Topics 1 to 8 were scheduled sessions from the [agenda](../../discussion-topics/README.md). Topics 9 to 14 came up during working time, usually because a pull request, issue or design question needed a decision from the group.

### <a id="topic-1"></a>Topic 1: Archiver Appliance

Tue 20 Oct 2026, morning session ending at 10:52. Agenda 2.3 Archiver Service, and 6.2 AI/ML for the AI-readiness part. The session had three parts:
- 1.1 Roadmap, deployment and waveform data
- 1.2 Parquet storage, FRIB results
- 1.3 Batch operations and AI-ready data

It was followed by a coffee break.

#### Discussion summary

##### 1.1 Roadmap, deployment and waveform data (until 10:23)

Murali Shankar presented the Archiver Appliance roadmap. This part was not recorded and comes from notes taken in the room. The main item is dynamic reassignment of PVs to appliances using Kubernetes cluster features ([reassign guide](https://epicsarchiver.readthedocs.io/en/latest/sysadmin/guides/reassign.html)). This avoids moving data between files, or moving files, and deals with state held by each appliance. Long-term storage holds many files, for example one file per PV per year, which slows things down. Packaging them was suggested: Parquet for compression and tar for bundling. When the archiver restarts after downtime, it causes broadcast storms that put load on the IOCs, although this may depend on the IOC host environment. For IOCs stopped on purpose, sites pause archiving. ESS uses ChannelFinder to automate both archiving PVs and pausing or resuming them.

A recap followed. Archiver work is focused on storage problems in large clusters and on making clustering work better in containers and Kubernetes. Murali recommended standalone single instances where possible. Otherwise, sites should plan for a Kubernetes-style deployment rather than installing each node by hand. The meeting agreed that the PV-disconnect and name-resolution problem goes beyond the archiver and should be discussed together with ChannelFinder and name resolution.

APS asked about best practice for waveform PVs. APS limits the size of waveforms it archives and lowers the sampling rate when storage use is high. ESS does not enforce an archiving policy, because stakeholders always say they need everything. Instead, ESS uses an EPICS module based on the LTTB (Largest-Triangle-Three-Buckets) algorithm. The module publishes a reduced copy of the waveform PV with fewer points but the same overall shape, and the copy can be archived instead of the full PV. Participants also noted that storage costs more than the disks themselves; network load and staff time add to it.

APS reported that retrieving one year of waveform data crashed the archiver web client while plotting; the Data Browser was not involved. The reply was that retrieval streams the data and should not run out of memory. Memory problems usually come from clients such as MATLAB or Pandas loading the raw data into memory, and decades of data had been retrieved without problems in early tests. Finally, the meeting raised what it means for archived data to be AI-ready, whether to add an MCP server on top of archiver retrieval, and invited advice from AI/ML experts. This was deferred until after the Parquet discussion (see 1.3).

##### 1.2 Parquet storage, FRIB results (10:23 to 10:37)

Cyril Roy (FRIB) presented FRIB's Parquet tests. In the first test, all partitions for one PV were stored as Parquet. A header is added every time data is written to a file, so short-term storage (STS) files written about every 10 seconds became too large. Related archiver issue: [#510](https://github.com/archiver-appliance/epicsarchiverap/issues/510), "Parquet file combination tuning". Later tests used Parquet only for long-term storage (LTS) and gave good compression. Compression depends on the data pattern rather than the file size: repetitive data compresses well, and noisy or random data much less. Overall savings are about 50%. ZSTD gave the best compression, followed by Snappy and then uncompressed, and the choice of codec has little effect on retrieval time. Retrieval from Parquet was noticeably slower than from the original protobuf files: one query took about 5 minutes with protobuf and about 15 minutes with Parquet.

Anthony Carriveau (FRIB) described the production migration. FRIB has limited storage. In one network of about 800,000 PVs, FRIB moved the noisiest and most data-heavy PVs: about 65,000 PVs with LTS data from 2024 and earlier. A utility written by Cyril measured about 60% compression per file, and the conversion took a little under a day. FRIB added an isolated "real long-term storage" (RLTS) partition with no ETL; files are moved there from LTS by hand. Out of a total footprint of about 36 TB, about 20 TB was freed.

The migration ran into several problems. Many of the selected PVs were paused or deleted, so their data first had to be consolidated into LTS. A medium long-term storage (MLTS) partition, set up earlier at FRIB by Eric, had gather=1 and hold=13. With hold greater than 11 (more than 12 months), consolidation left files behind; FRIB had to reduce it to 11 during a downtime before consolidation could finish. After the RLTS partition was added, retrieval immediately began reading the Parquet files in RLTS instead of LTS, which the team confirmed using `lsof`. As a result, the bitwise-identical checks had read from RLTS both times. The reason is still unknown. The last step is to delete the original protobuf files. Anthony described the overall process as fairly simple and successful.

Murali and Sky noted that Parquet settings can be configured for each PV, so storing waveforms is likely to be one of the first uses of Parquet. When asked whether all tiers should use Parquet, the meeting advised keeping STS and MTS in protobuf for now and applying Parquet only at the last ETL step into the final tier. For a standard STS, MTS, and LTS setup, the final tier is LTS. Sites with an extra tier beyond LTS, such as an "extra long-term storage" (XLTS) tier or FRIB's RLTS partition, may keep LTS in protobuf and use Parquet only in that extra tier. This is because of the header overhead and because compression works better with larger blocks of data. FRIB plans to do the same and will test Parquet in the ETL stage thoroughly before relying on it. Participants stressed that FRIB's experience should be documented so that other facilities do not have to rediscover these problems.

##### 1.3 Batch operations and AI-ready data (10:37 to 10:52)

**Batch business process logic (BPL).** Murali proposed an easy entry project for new archiver contributors: make the BPL batch-friendly. Many BPLs still handle one PV at a time, which does not scale to millions of PVs. Pause and resume now accept batches; for example, 100,000 PVs can be paused in one call. Murali implemented this using the same REST endpoint: if the request content type is `application/json`, the body is treated as a list of PVs. ESS uses ChannelFinder to archive and pause PVs; Berkeley also uses ChannelFinder for archiving, but not for pausing. Requests to `archivePV` are still sent one PV at a time, so a batch archive call would let the whole list be sent at once.

One participant asked whether HTTP/2 or HTTP/3 multiplexing would be a better way to scale requests in 2026. The reply was that Tomcat still starts a thread for each request. A batch call lets the server divide PVs by appliance instance, process them internally, combine the results, and return one response. The counterargument was that clients still need to split very large lists into batches, so the extra complexity may not save much. ESS and Berkeley, the heaviest users, considered it a small but useful improvement. Murali's two existing implementations should make it easy to extend batch support to other BPLs.

**AI-ready archived data.** A common pattern is to copy the entire long-term store into an AI system and parse the files there, because AI analysis needs whole systems of data rather than one PV at a time. Others asked why the existing retrieval API is not enough. Murali asked Lucy and Leve for input on what metadata should be stored. Lucy (BNL) described analysis problems on a multiplexing (pulse-to-pulse modulated, PPM) machine. For example, a cycle switches from gold ions to helium within seconds. The cycle number and user or species are not visible from the timestamp, and magnets and BPMs are logged at different rates. As a result, correlating a magnet ramp with BPM readings is very difficult.

Michael Davidsaver's response was that the timestamp user tag was designed for exactly this problem. SLAC, TRIUMF, and Los Alamos have similar multi-beam or multi-mode needs. PVAccess can filter monitors by user tag, and an archiver that stores the tag would allow filtering afterwards. Bob Dalesio stressed that timestamps from the hardware timing system carry meaning and that data analysis follows "garbage in, garbage out". The archiver does not store the user tag yet; an issue is open, and Murali intends to implement it. Based on the points from Lucy, Michael, and Bob, the group described a complete solution: IOCs provide correct tags, FPGAs or the hardware timing system provide accurate timestamps, the archiver stores the user tag, and a client-side tool rebuilds events. The archiver is not a DAQ and does not build events. For event-driven data such as quench or injection snapshots with pre- and post-trigger data, an event-logging system such as the "Machine Learning Data Platform" is more suitable. This platform manages the lifecycle and provenance of each event.

#### Key findings

*Roadmap and waveforms (1.1)*
- The roadmap includes dynamic PV reassignment based on Kubernetes, and it is documented.
- ESS automates pausing and resuming archiving through ChannelFinder.
- ESS uses an LTTB-based EPICS module that reduces waveform data while preserving its shape; the reduced PV is the one archived.
- Archiver retrieval streams data, so out-of-memory problems usually come from the client.

*Parquet (1.2)*
- Parquet saves about 50% of storage overall; the result depends on whether the data is noisy or repetitive.
- ZSTD gives the best compression, followed by Snappy and then uncompressed. The codec has little effect on retrieval time.
- Compression depends on the data pattern, not the file size.
- Retrieving from Parquet is slower than from protobuf; in one example it took about 15 minutes instead of 5.
- FRIB production migration: about 65,000 of 800,000 PVs, using LTS data from 2024 and earlier; about 60% compression per file; under a day to convert; about 20 TB freed out of 36 TB.
- Parquet can be configured for each PV, so waveforms are a likely early use (Murali, Sky).

*Batch and AI (1.3)*
- Pause and resume BPLs already accept JSON batches, for example 100,000 PVs in one call.
- ESS uses ChannelFinder to archive and pause PVs; Berkeley uses it for archiving. `archivePV` still accepts only one PV per request.
- The archiver does not store the timestamp user tag yet, and an issue is open.
- PVAccess supports filtering monitors by user tag.
- Multiplexing and multi-beam facilities, including BNL, SLAC, TRIUMF, and Los Alamos, share the problem of separating data by cycle or beam.

#### Best practices highlighted

*Deployment and operations (1.1)*
- **Deployment (Murali):** use standalone single instances first. If that is not possible, plan for a Kubernetes-style cluster instead of installing each node by hand.
- **Intentionally stopped IOCs:** pause archiving, and automate pause and resume through ChannelFinder as ESS does.
- **Waveforms:** archive a reduced PV that keeps the waveform shape, using LTTB as ESS does, instead of the full waveform. Limiting size or sampling rate is the APS alternative.
- **Large retrievals:** stream data from the archiver; avoid clients that load all raw data into memory, such as MATLAB or Pandas.

*Parquet (1.2)*
- Keep STS and MTS in protobuf. Use Parquet only at the last ETL step into the final tier: usually LTS, or an extra tier beyond LTS where a site has one.
- Do not use Parquet for STS: every write adds a header, and compression works better on larger blocks of data.
- Use the ZSTD codec. The choice of codec has little effect on retrieval time.
- Start with noisy, data-heavy PVs and older data. Savings depend on the data pattern.
- Before migrating, consolidate paused or deleted PVs into LTS, and keep hold at 11 or less (12 months or less) so consolidation can finish.
- When migrating existing data, consider FRIB's approach: put converted files in a separate partition with no ETL, such as RLTS.
- When validating, use `lsof` to check which files retrieval actually reads, and confirm that the data is bitwise identical.
- Delete protobuf files only after their data has been consolidated and verified.
- Test retrieval performance before moving frequently accessed data, because Parquet retrieval can be slower.

*Batch and AI-ready data (1.3)*
- Store as much meaning as possible when archiving ("garbage in, garbage out"); context cannot be reconstructed afterwards.
- Use timestamps from the hardware timing system where possible.
- Use the user tag to label cycles, beams, and modes.
- Treat the archiver as a store for continuous data, not a DAQ. Use an event logger for triggered pre- and post-event data.
- Prefer batch calls, split by appliance instance on the server, over many individual HTTP requests.

#### Conclusions / agreements
- **Agreed:** PV disconnects and name resolution are a broader problem and will be discussed with ChannelFinder and name resolution.
- **Current Parquet recommendation:** keep STS and MTS in protobuf, and use Parquet only at the last ETL step into the final tier, usually LTS.
- FRIB intends to follow this approach and test Parquet in the ETL stage thoroughly.
- The Parquet migration lessons should be documented for other facilities.
- Batch support is worth extending to other BPLs. ESS and Berkeley called it a small but useful improvement, and Murali's existing implementations reduce the effort.
- For AI or event analysis, the whole chain is needed: IOC tags, accurate timestamps, user tags in the archiver, and client-side event building.

#### Issues / problems

*Roadmap and waveforms (1.1)*
- Having a large number of files in long-term storage slows performance.
- Restarting the archiver causes broadcast storms and load on IOCs.
- Waveform storage keeps growing, and sites have no common policy for it.
- Retrieving one year of waveform data crashed the archiver web client while plotting. The cause is unknown and may be specific to one site.

*Parquet (1.2)*
- Parquet adds a header on every write, which inflates STS files. Related: [#510](https://github.com/archiver-appliance/epicsarchiverap/issues/510).
- When hold is greater than 11 (more than 12 months), consolidation leaves files unconsolidated.
- After RLTS was added, retrieval read from RLTS instead of LTS. The cause is still unknown.
- Paused or deleted PVs need consolidation before migration.
- There is a risk of deleting protobuf files before their data has been consolidated.

*Batch and AI (1.3)*
- BPLs that handle one PV at a time do not scale to millions of PVs.
- On PPM or multiplexed machines, archived data cannot be separated by cycle, user, or species.
- Magnets and BPMs are logged at different rates, and magnet setpoints and readbacks lag each other, which makes correlation difficult.
- It is unclear whether batch calls reduce overall complexity, because clients still need to split very large requests.

#### Project ideas
- Package long-term storage files using Parquet and tar.
- Write best-practice guidance on archiving waveforms, including LTTB-style shape-preserving reduction.
- Use per-PV Parquet configuration for waveform storage.
- Investigate the web-client crash during long waveform retrievals and, if confirmed, file an issue against the archiver web client.
- Add a Parquet migration guide to the archiver sysadmin documentation.
- Fix consolidation when hold is greater than 11.
- Find out why retrieval selects RLTS instead of LTS.
- **Starter project:** add JSON batch support to other BPLs, following the pause and resume implementation.
- Add a batch `archivePV` call so ChannelFinder-driven archiving, as used by ESS and Berkeley, can submit whole lists of PVs.
- Store the timestamp user tag in the archiver and support filtering on it.
- Build a client-side event builder for archived data.
- Define what AI-ready archived data means and consider an MCP server for archiver retrieval. Related agenda item: 6.2.

#### Commitments / actions
- **Anthony Carriveau (FRIB):** open a PR to the archiver documentation with a Parquet migration guide and lessons learned. Anthony agreed: "Yeah, absolutely".
- **Owner to be confirmed (Anthony suggested it):** create a ticket for the consolidation problem when hold is greater than 11 or 12.
- **FRIB:** test Parquet in the ETL stage thoroughly before relying on it.
- **Murali Shankar:** implement storage of the timestamp user tag in the archiver. Others are welcome to work on the open issue.
- **Volunteer needed:** batch processing for `archivePV` and other BPLs.

#### Follow-up after the codeathon
Status checked on 12 Nov 2026.
- [epicsarchiverap#559](https://github.com/archiver-appliance/epicsarchiverap/pull/559) "Add support for PVAccess userTags." (slacmshankar, opened 22 Oct, open).
- [epicsarchiverap#563](https://github.com/archiver-appliance/epicsarchiverap/pull/563) "Lttb postprocessor for scalar pvs" (caraxlr, opened 23 Oct, open). See Topic 15.

#### Linked passages
- LTTB as an archiver retrieval post-processor: Topic 15 (afternoon working session, 14:33 to 14:39).

### <a id="topic-2"></a>Topic 2: Web toolkits, PVWS, DBWR, PVInfo, and web security

Tue 20 Oct 2026, 11:32 to 12:20. Agenda 6.2 (web vs native), 5.1 (access control for web clients), 2.1 (Tomcat and Spring Boot alignment), and 3.1 (DBWR and PVWS ownership after Kay).

#### Discussion summary

**DLS web stack (Martin Gaughran, remote).** DLS has two layers: cs-web-lib, a React widget library that uses PVWS to connect to EPICS, and Daedalus, a web equivalent of Phoebus built on cs-web-lib, where current development effort is focused. Planned work for about March 2027 includes:
- Quick screens: drag widgets from displays into custom panels that can be shared, for lightweight editing by beamline staff.
- Alarm displays: show alarm status and integrate with the alarm handler, but less fully than in Phoebus.

From 2027 onward, DLS plans authentication and RBAC by running separate PVWS instances for different permission levels, and a DLS-specific "baton" system for beamline control alongside data acquisition. A full Data Browser is longer term. An editor has been postponed; DLS uses the Phoebus editor for now and is considering working with a new display library (name unclear in the recording; possibly **Weiss**, weiss-controls, not yet confirmed) that has an editor and Phoebus-like widgets but does not use the Phoebus file format. Daedalus can already load and run `.bob` screens, but it is not yet a direct replacement for DBWR. DLS asked interested sites to tell them what they would need. For Diamond-II, DLS will use both a web UI and Phoebus depending on the use case; dropping Phoebus is undecided and would not happen before about 2030.

**PVWS forks.** There are two nearly identical forks: the ORNL version, which builds a WAR for Tomcat, and the ControlSystemStudio Spring Boot version, which uses embedded Tomcat and can still produce a WAR. Kay's recent changes were brought into the Spring Boot version, so it is almost up to date. Participants disliked the Tomcat deployment model, but not Tomcat itself. NSLS-II has already moved from the Tomcat version to the Spring Boot version without problems. The meeting agreed to focus effort on the Spring Boot version, with follow-up tests by the main users. ORNL (SNS), DLS, Berkeley, and CEA will test it. The collaboration will check that it is in sync with the ORNL fork and create a container image, a simple Dockerfile of about five lines, during the codeathon. Results are due at the next monthly developers meeting. Nobody proposed GraphQL or a new WebSocket solution.

**DBWR.** DBWR, which renders `.bob` files as web pages, is heavily used at ORNL and used at NSLS-II for accelerator overview screens. Moving DBWR to Spring Boot should be easy and could be done during the codeathon. The bigger question is whether DBWR is the right direction if DLS, and possibly ITER, invest in a richer React-based framework. The comparison that matters is Daedalus vs DBWR. This is mostly a question of resources, expertise, and finding a maintainer, especially for ORNL. No decision was made. An older comparison of DBWR and cs-web-lib from April exists and is now out of date.

**Facility strategies.** Ralph Lange said ITER's CS-Studio owner is leaving and ITER still runs a 15-year-old Eclipse-based CS-Studio. A team of four will evaluate four options before the end of the year:
- Phoebus as it is today.
- Daedalus, the next-generation web option.
- A commercial product, which would mean removing EPICS.
- Building a new system themselves.

ITER will contact DLS. The meeting noted that PVWS depends on CorePVA and other Phoebus libraries, so a web direction still relies on Phoebus code. Kunal Shroff said EIC plans to treat desktop Phoebus and web tools as complementary and invest in both. ITER currently relies heavily on WebOPI.

**PVInfo (ALS) and CEA alarm web.** PVInfo is a React application used daily by physics staff and operators for quick troubleshooting. It shows live and archived values, links to the archiver's built-in viewer, and lists Olog entries mentioning a PV. The community version uses the Phoebus Olog API, and it connects to ChannelFinder, PVWS, and the Phoebus alarm services. Kunal Shroff had tried to deploy it at NSLS-II but did not finish before he left. NSLS-II PV names are not parsed correctly by the archiver viewer, which could be fixed in PVInfo's link or in the archiver. PVInfo is being upgraded to a new Material UI/React version, and testers are wanted.

CEA's web alarm application, developed by Loic Caouen, is almost finished. It provides a tree, a table, an area matrix panel, and an admin view for editing the tree by drag and drop. Admin access requires login with a JWT token and the right role. Its API talks to Kafka directly and is separate from the alarm server. The developer would prefer to merge it into the alarm server so Kafka does not have to be exposed. The code is not public; CEA must first get permission, and only one person works on it. Whether to merge PVInfo with the CEA alarm web was discussed but not decided. PVInfo is read-only, while exposing Kafka would allow acknowledgements.

**PVWS security.** Today NSLS-II only redirects to single sign-on in front of the read-only web runtime, and Berkeley and CEA run PVWS read-only. Writing through PVWS needs authentication, authorization, and auditing. Auditing is hard because writes go to IOCs from the PVWS server, so the user identity is easy to fake. The direction is to use standard web and Spring Boot security rather than build a custom solution; CEA's JWT-based alarm web is one example. First, collect the security requirements; then compare candidate solutions.

#### Key findings
- DLS: cs-web-lib is the widget library on PVWS; Daedalus is the Phoebus equivalent. Quick screens and alarm displays are planned for about March 2027, followed by RBAC through separate PVWS instances.
- Daedalus can load and run `.bob` screens but is not a direct replacement for DBWR yet.
- The two PVWS forks are nearly identical; NSLS-II moved to the Spring Boot version without problems.
- Diamond-II will use both web tools and Phoebus.
- ITER is choosing between Phoebus, Daedalus, a commercial product, and building its own; the decision is due by the end of 2026.
- PVWS depends on CorePVA and other Phoebus libraries.
- PVInfo is used daily by operators and integrates Olog, ChannelFinder, PVWS, alarms, and the archiver viewer.
- CEA's alarm web is almost finished, with JWT and role-based admin. Its API uses Kafka directly, and the code is not yet public.
- Current PVWS deployments are read-only or protected only by an SSO redirect.

#### Best practices highlighted
- Prefer Spring Boot with embedded Tomcat over deploying WAR files to Tomcat. Spring Boot can still build a WAR if needed.
- Do not expose Kafka to web clients; put alarm actions behind an authenticated API.
- For web security, use standard frameworks such as Spring Security or JWT rather than building a custom solution.
- Coordinate on shared web toolkits, because they change quickly and are expensive to maintain.

#### Conclusions / agreements
- **Agreed:** focus PVWS effort on the Spring Boot version; needs follow-up (site tests, sync with ORNL fork, container image).
- No interest in GraphQL or a new WebSocket solution.
- Follow up on PVWS at the next monthly meeting.
- **Not decided:** DBWR vs Daedalus as the community web runtime, and whether to merge PVInfo with the CEA alarm web.

#### Issues / problems
- Effort is split across two PVWS forks.
- DBWR has no maintainer since Kay retired, and its future is unclear.
- PVWS has no authentication, authorization, or audit for writes, and user identity is easy to fake.
- The CEA alarm web API exposes Kafka, which is a security concern, and the code is not public.
- The archiver viewer does not parse some NSLS-II PV names.
- DLS does not know what other sites need to adopt Daedalus.

#### Project ideas
- PVWS Spring Boot container image (about five-line Dockerfile) and a check that it is in sync with the ORNL fork. Related project: DEVOPS-CONTAINER-001.
- Move DBWR to Spring Boot.
- PVWS security requirements document, followed by a review of solutions.
- Docker Compose file for Phoebus middle-layer services (archiver, alarms, ChannelFinder, PVInfo). Related project: DEVOPS-CONTAINER-001.
- Test deployments of PVInfo and checks of its documentation, plus testing the Material UI/React upgrade.
- Fix PV-name parsing for archiver viewer links, in PVInfo or in the archiver.
- Add PVWS stress tests to release testing, not CI.

#### Commitments / actions
- **ORNL/SNS:** test the Spring Boot PVWS WAR in the existing deployment pipeline. Report at the next monthly meeting.
- **DLS:** test the Spring Boot PVWS by swapping the container image on Kubernetes.
- **Berkeley:** test the Spring Boot PVWS; Berkeley currently uses the Tomcat version.
- **CEA:** ask Rémi to test the Spring Boot PVWS.
- **Collaboration (volunteer needed):** check that the Spring Boot PVWS is in sync with the ORNL fork, and build a container image during the codeathon.
- **Martin Gaughran (DLS):** find the PVWS stress-test scripts and results and share them.
- **Martin Gaughran (DLS):** email links for the new display library (possibly Weiss) and related projects. Link not yet received.
- **Ralph Lange (ITER), asked to lead:** write a rough document of PVWS security concerns and requirements this week.
- **Person not identified:** share the older DBWR vs cs-web-lib comparison from April.
- **Loic Caouen (CEA):** get permission to publish the alarm-web code.

#### Follow-up after the codeathon
Status checked on 12 Nov 2026.
- [pvws#11](https://github.com/ControlSystemStudio/pvws/pull/11) "Spring4" (shroffk) was merged on 22 Oct: Spring Boot 4 and Java 25.
- [pvws#12](https://github.com/ControlSystemStudio/pvws/pull/12) "Aligning with minor changes in the 'legacy' pvws code base" (georgweiss) was merged on 5 Nov. This brings the Spring Boot version in line with the ORNL fork.
- [pvws#13](https://github.com/ControlSystemStudio/pvws/pull/13) "Docker stuff updated for Java 25" (georgweiss) was merged on 2 Nov. A Dockerfile already existed ([pvws#6](https://github.com/ControlSystemStudio/pvws/pull/6), 2025).
- [pvws#14](https://github.com/ControlSystemStudio/pvws/pull/14) "Allow cors requests" (georgweiss, open). See Topic 17.

#### Open questions
- Is the new display library DLS mentioned Weiss (weiss-controls/weiss)? Awaiting link from DLS.
- Who will maintain DBWR, and should the community runtime be DBWR or Daedalus?
- Who shared the April DBWR vs cs-web-lib comparison?

#### Linked passages
- The legacy CS-Studio Olog plugin and the PVWS follow-up work after lunch: Topic 15.
- Alarm REST API and the alarm architecture: related to the alarm discussion (agenda 2.6) and ALARM-REST-001.

### <a id="topic-3"></a>Topic 3: Alarm services

Wed 21 Oct 2026, 09:10 to 10:31 and 10:54 to 10:56. Agenda 2.6 Alarm Services. Chaired by Kunal Shroff. The Olog session that was also planned for this morning was postponed, and after the break the meeting moved straight on to agentic frameworks. A side conversation during the break about auto-generating engineering screens is recorded separately as Topic 4.

#### Discussion summary

The session took six sub-topics in turn: Kafka resilience, configuration management, authorization, multiple configurations, mode-dependent (dynamic) alarms, and alarm floods with analytics.

**1. Kafka upgrade and reconnection (09:11 to 09:17; 10:26 to 10:31).**
- The alarm services now run on Spring Boot 4 and a newer Kafka client. Speakers were unsure whether it is Kafka 3 or 4.
- The newer Kafka error-handling features are not used yet. If Kafka goes down or the network blips, the alarm server only reports that it is disconnected. Someone then has to restart the alarm server, the alarm config logger and the alarm logger by hand.
- Newer Kafka offers configurable reconnect retries and back-off, so the three services could recover on their own. Kunal called this a crucial reliability improvement.
- Part of this work may already have been done, but it was never verified, closed or documented. Verifying it was described as an easy project:
  - check that all three services and the alarm clients (tree, table, panel) use the current Kafka;
  - start the alarm server with Kafka, stop and restart Kafka repeatedly, and observe recovery.
- Nobody volunteered in the session. Kunal said a volunteer would be found at the coffee break.
- **KRaft:** Oak Ridge has migrated to KRaft (no ZooKeeper), and it was one of the most painless migrations. Whether NSLS-II has also done so was unclear.
  - Sites still on ZooKeeper were advised to switch, as Kafka itself recommends.
  - For a single-node Kafka, which is what the site asking uses, dropping ZooKeeper has no impact, because ZooKeeper only manages clusters.
- **Versions:** after the session, participants found that the Kafka client libraries pulled in by Spring Boot are 3.9, while at least one site runs a Kafka 4 server. Library names start with the Scala version, and Apache and Confluent Kafka are numbered differently, so care is needed when comparing versions.

**2. Alarm configuration management (09:17 to 09:36).**

*NSLS-II and EIC practice:*
- Almost all configuration is manual, one PV at a time; multi-select add of PVs was added recently.
- The alarm config logger version-controls the whole configuration on every change. It is a useful backup: if the alarm server is corrupted, the configuration can be restored to any point in time.
- What is missing is scripting support. There is no REST API, so programmatic changes mean publishing configuration messages to Kafka directly, which is convoluted.

*Why the tooling is like this:* it deliberately follows Kay's philosophy that the alarm server is not a place to dump a million PVs. Alarms should be added one at a time with expert input: descriptions, guidance and actions. Many facilities struggle with this.

*ESS proposal: separate configuration from alarm state.*
- Configuration would move to its own service, with a relational database and a REST API.
- The alarm server would only monitor PVs, compute the alarm state of the tree and send alarm messages.
- Phoebus configuration tools would use the new API. Clients would still subscribe to Kafka for alarm state.
- A "configuration changed" message on Kafka would tell services to fetch the new configuration.

*Acknowledged and disabled states:* users asked to keep these across configuration imports; today an import resets them. A second site reported the same complaint. Kunal asked FRIB, which re-imports XML often, but FRIB was not aware of the behaviour.

*ISIS input and the edit/run-mode proposal:*
- Percy (ISIS) said ISIS is migrating from Vista. It starts with an incomplete configuration and needs operators to fix it quickly, because adding every alarm carefully is not practical there.
- Percy has prototyped a friendlier editor outside Phoebus ([#3920](https://github.com/ControlSystemStudio/phoebus/issues/3920)). It uses drag-and-drop and a properties panel instead of dialog boxes.
- In his view the UI is the easy part. The real question is the back end: a relational database, Kafka, or something else.
- Kunal proposed one alarm tree application with two modes, as the Display Builder has an editor and a runtime for the same file:
  - **Edit mode:** context menu for configuration (add, move, remove, properties panel).
  - **Run mode:** operator actions such as guidance, how long an alarm has been active, and related displays.
- Separating the modes reduces cognitive load. It is also a low-risk first step towards a separate configuration service, because it shows how the two sides interact.
- Percy agreed and thought it could be done fairly quickly, since most of the functionality exists. Georg Weiss agreed to work with him.
- Kunal asked FRIB and Oak Ridge, whose alarm workflows are the most formal, whether this would break anything. ITER would be the third such site, but Ralph was not present. FRIB said its pipeline is script-based and would not be affected; Oak Ridge did not answer.

*Tree table:* converting the alarm tree into a tree table was also suggested. Several people worried it would be too busy next to the existing alarm table. The narrow version gained support: move the counts of active and disabled alarms, now shown in the node label, into a separate column, without adding many more columns. The edit and run modes will be merged first.

**3. Authorization and security (09:36 to 09:50; 10:28).**

*Open requests:*
- **Per-node ownership** ([#1336](https://github.com/ControlSystemStudio/phoebus/issues/1336), an old request from FRIB): only the owning group, for example the RF engineers, can change its part of the tree. Nobody said they urgently needed this, so it was noted but deprioritized.
- **Blocking configuration changes altogether.** One site already does this by configuring the network so producer messages are not allowed.
- **Commands configured in the alarm tree** can be executed on the alarm server itself, automatically or from the context menu, which is a security risk ([#3702](https://github.com/ControlSystemStudio/phoebus/issues/3702)).

*CEA (Loic Caouen):*
- Needs three roles: read-only, acknowledge (operators) and administrator (configuration).
- None of this is enforced today. Where the alarm server and Phoebus are on different machines, Kafka is open to the whole network, which is a security problem.
- CEA does not let anyone change configuration from Phoebus. The first project generates an XML file and imports it. New projects use CEA's own API, which takes JSON in the same format as the Kafka messages.
- A future project will use CI to scan IOC database files for PVs with alarm limits and generate the configuration.
- If the configuration is split into its own service, CEA's API would call that service's API.

*Other site practices:*
- FRIB manages its XML through a Git pull-request workflow, but nothing stops someone editing from Phoebus.
- One site blocks Kafka producer messages from most networks, so configuration can only be changed from a few machines in a restricted zone.

*Discussion:*
- Splitting configuration from runtime would allow an operator-only Phoebus with configuration disabled.
- Even after a split, someone still has to decide who may acknowledge, because acknowledgement is not part of the configuration.
- Clients send acknowledgements straight to Kafka. Controlling them therefore means routing client actions through an alarm server API instead of Kafka. Kafka's own access controls were mentioned but would be complicated to set up.
- CEA would like such an API added to the alarm server.
- Kunal suggested generating alarm configuration from IOC info tags, as Berkeley does for archiving. Others warned that relying on alarm limits misses hardware and communication failures (INVALID alarms), for example a failed PLC or vacuum-controller link.
- No path forward was agreed.

**4. Multiple alarm configurations (09:50 to 09:55).**
- NSLS-II has an operations configuration (clean, used in the control room) and an engineering configuration (noisy). Each beamline also has its own configuration and a separate, standalone alarm deployment, roughly 30 to 40 in total.
- CEA has about 50 configurations on a single Kafka, one to three per project.
- Running one alarm server per configuration works well.
- The Phoebus client shows one topic at a time. ESS operators open several windows; CEA would sometimes like to see several topics in one view. There was no strong demand for change.

**5. Mode-dependent (dynamic) alarms (09:55 to 10:12).**
- Edmund Blomley (KIT) presented this at the last two EPICS meetings. His view was that the IOC is the right place to solve it. His options were:
  - an extra control PV per PV that rewrites the alarm limits, which multiplies the number of PVs;
  - logic inside the alarm service.
- Kunal rejected putting logic in the alarm server: processing belongs in the IOC, not in services or tools.
- How often modes change varies by facility. At NSLS-II, alarm settings have been stable since commissioning; elsewhere modes change hourly.
- Examples:
  - a shared beamline where pulses alternate between end stations (for example 120 Hz at SLAC);
  - Oak Ridge's planned second target station, which will take every sixteenth pulse;
  - low-energy first injection at NSLS-II, where most alarms can be ignored.

*Three approaches, by speed of change:*
- **Pulse to pulse:** must be done in the IOC, driven by the timing system. A reliable, facility-wide mode broadcast would be ideal, including soft IOCs with no timing hardware, but no such mechanism exists today.
- **Hourly to daily:** restore alarm limits from Save & Restore snapshots, triggered through its REST API.
  - Kunal liked this because it avoids changing every IOC and doubling the PV count; EIC would otherwise go from 20 to 40 million PVs.
  - Scaling to about 200,000 PVs would need stress testing. MASAR at NSLS-II was built for thousands of save sets, mainly for orbit setpoints.
  - The writes are not instantaneous.
- **Rare changes (months):** swap static configurations, which already works.

*Already available:* masking subtrees with PV-enabled filters. Berkeley switches a mode PV (mbbo) to enable and disable sets of alarms. CEA has a script that disables parts of the tree depending on the active gas path.

*Outcome:* the first step is to collect use cases and existing scripts. Kunal summarised that the problem had been broken into three problems with three solutions. If the configuration must change faster than people can change it, the answer is at the IOC level, not in the alarm server.

**6. Alarm floods, multiple hierarchies and analytics (10:12 to 10:23).**

*ITER (Ralph Lange):*
- ITER receives systems as in-kind contributions, and none of the alarm configurations delivered so far was usable.
- The control room must have fewer than 140 alarms. Only 8 of 170 systems have been delivered, and they already define about 30,000 alarms.
- Email notifications sent about 6,000 emails a minute over a weekend and brought down the Microsoft Exchange server.
- ITER needs cause-and-effect filtering: when a pump fails, show the pump, not the 300 pressure sensors downstream. These dependencies cut across the alarm tree's structure.
- Ralph's task at the codeathon is to find out whether the alarm handler can support this.

*What exists and what is missing:*
- The alarm tree can already group a pump and its sensors under one node. The intended behaviour is a single email from the top node listing everything below it; Kay worked on this, but it is not fully tuned.
- What is missing is support for several independent hierarchies over the same PVs, for example by network, power or location.
- Percy said ISIS needs the same kind of "virtual hierarchy" to switch off everything related to one of its two targets.
- Darren (ESS) showed alarm masking in a graphical application at a recent Phoebus meeting. It goes in the same direction but is still limited to one hierarchy.
- Karen White's (ORNL) rules on reasonable alarm counts were cited, along with the question of who actually consumes alarms from millions of PVs.
- No solution was proposed. One remark: "the situation is hopeless, but not serious".

*Analytics:* Kunal proposed analysing the alarm logger's Elasticsearch data.
- Elasticsearch now supports vector embeddings. Kunal described dense vectors as mainly for search and sparse vectors as useful for anomaly detection.
- With good timestamps, logs could also reveal correlations and cascades (2,000 alarms that started from a few).
- This needs long alarm-log histories.
- Several facilities run the alarm logger, mainly for auditing who acknowledged what and when. They said they would help build analysis tools.

**7. Kafka "topic does not exist" (10:54 to 10:56).**
- With the newer Kafka client, a missing topic raises a specific exception. The client stays connected to the broker but keeps retrying a topic it can never join.
- A utility class now creates topics with the correct settings, which addresses the many issues about Kafka configuration disappearing.
- A preference could either create the missing topic or raise a notification.
- The speaker wanted to clean up the related code first, and it was left for an afternoon discussion.

#### Key findings
- The alarm services do not use the newer Kafka reconnect features. A Kafka interruption needs a manual restart of the alarm server, config logger and logger.
- Oak Ridge moved to KRaft painlessly. For single-node Kafka, dropping ZooKeeper has no impact.
- There is no REST or scripting API for alarm configuration; scripting means sending Kafka messages.
- Importing a configuration resets acknowledged and disabled states.
- At some sites Kafka is open to the whole network, and clients send acknowledgements straight to Kafka, so neither can be access-controlled.
- One alarm server per configuration works well at scale: NSLS-II about 30 to 40 (one per beamline plus accelerator), CEA about 50.
- PV-enabled filters for masking subtrees exist and are used at Berkeley.
- ITER: 8 of 170 systems already define about 30,000 alarms against a target of fewer than 140; email notifications brought down the mail server.
- Alarm logs are used mainly for auditing today.

#### Best practices highlighted
- Use KRaft instead of ZooKeeper; for single-node Kafka it is a drop-in change.
- Use the alarm config logger as a versioned backup, so the configuration can be restored to any point in time.
- Add alarms thoughtfully, with descriptions, guidance and actions, rather than in bulk.
- Run one alarm server instance per alarm configuration.
- Put mode-dependent logic in the IOC, not in the alarm server or tools.
- Choose the approach for mode-dependent alarms by how fast the mode changes.
- Group dependent alarms under a top node so that one summary notification goes out instead of hundreds.
- Do not rely only on alarm limits; hardware and communication failures (INVALID) also need alarms.
- Until access control exists, restrict Kafka producers by network if configuration changes must be controlled.
- Keep long alarm-log histories with good timestamps to allow later analysis.

#### Conclusions / agreements
- **Agreed:** add edit and run modes to the alarm tree, based on Percy's prototype. This splits the tooling only; it is not a new configuration service or API. It will be merged only after FRIB and Oak Ridge give the green light.
- Do the edit and run modes before any tree-table rework. If a tree table is added, move the alarm counts into a separate column first.
- A separate configuration service with a database and REST API is a larger, longer-term project.
- Mode-dependent alarm configuration was broken into three problems with three solutions:
  - static configuration swaps, which already work;
  - Save & Restore snapshots for slow changes, to be explored;
  - IOC-level logic for fast changes, for which nothing exists yet.
- Upgrade the Kafka libraries and verify that disconnects and reconnects are handled.
- **Not decided:**
  - how to authorize acknowledgements and configuration changes;
  - how to support multiple alarm hierarchies.

#### Issues / problems
- Alarm services need manual restarts after Kafka interruptions.
- Configuration is manual and cumbersome, with no batch or scripting interface.
- Acknowledged and disabled states are lost on import.
- There is no authorization for configuration changes or acknowledgements, and Kafka is open to the network.
- Commands configured in the alarm tree can execute on the alarm server, which is a security risk.
- Configuration controls outside Phoebus, such as FRIB's Git workflow, do not stop edits from Phoebus.
- Relying on alarm limits misses hardware and communication failures.
- Alarm floods and email storms (ITER); there is no support for multiple cause-and-effect hierarchies.
- In-kind contributions arrive with unusable alarm configurations (ITER).
- Kafka client and server version numbers are confusing (Scala prefix; Apache vs Confluent).
- A missing Kafka topic leaves clients retrying forever.

#### Project ideas
- Verify and finish Kafka reconnect and retry handling in the alarm server, config logger, logger and clients. Related: ALARM-KAFKA-001/002.
- Alarm tree edit and run modes with an inline properties panel ([#3920](https://github.com/ControlSystemStudio/phoebus/issues/3920)). Related: ALARM-UI-001/002.
- Move the alarm counts from the node label into a separate column.
- Keep acknowledged and disabled states across configuration imports.
- A separate alarm configuration service with a database and REST API. Related: ALARM-REST-001.
- An alarm server API for client actions such as acknowledge, so Kafka does not have to be exposed.
- Save & Restore integration to switch alarm limits by operating mode.
- Generate alarm configuration from IOC info tags or from CI checks of database files.
- Alarm-log analytics: anomaly and cascade detection using Elasticsearch vectors.
- A preference to auto-create a missing Kafka topic or raise a notification. Related: ALARM-TOPICS-001.

#### Commitments / actions
- **Percy (ISIS) and Georg Weiss (ESS):** add edit and run modes to the alarm tree, based on Percy's prototype.
- **FRIB and Oak Ridge:** give the green light before the edit and run-mode change is merged.
- **Unassigned (volunteer to be found):** upgrade to the latest Kafka libraries and verify disconnect and reconnect handling in the alarm services and clients.
- **Unassigned:** document how alarm configuration changes at different rates can be handled, including how Save & Restore could be used.
- **CEA:** share its alarm-disabling scripts and guidance as input for requirements.
- **Several facilities (not named):** use alarm-logger data to help build tools for improving alarm configurations.
- **Ralph Lange (ITER):** find out whether the alarm handler can support ITER's hierarchical filtering.
- **Kunal Shroff:** open a GitHub discussion collecting alarm security expectations (who can acknowledge, disable or change configuration), with links to related discussions.

#### Follow-up after the codeathon
Status checked on 12 Nov 2026.
- [#3920](https://github.com/ControlSystemStudio/phoebus/issues/3920) "Interactive alarm editing proposal" was closed as not planned on 28 Oct. The work continues as a series of open PRs by LordIdra (ISIS):
  - [#3991](https://github.com/ControlSystemStudio/phoebus/pull/3991) "Add editing mode to alarm tree";
  - [#4007](https://github.com/ControlSystemStudio/phoebus/pull/4007) "Move alarm config menu out of dialog box and into 'properties' panel";
  - [#4013](https://github.com/ControlSystemStudio/phoebus/pull/4013) "Overhaul disable alarms dialog", [#4010](https://github.com/ControlSystemStudio/phoebus/pull/4010) "Enabling filter UI improvements", [#3988](https://github.com/ControlSystemStudio/phoebus/pull/3988) "Alarm tree search" and [#4018](https://github.com/ControlSystemStudio/phoebus/pull/4018) "Alarm tree filter".
- [#3985](https://github.com/ControlSystemStudio/phoebus/pull/3985) "Switching to 4.3.1 for the Kafka client libs" (georgweiss) was merged on 26 Oct.
- [#3977](https://github.com/ControlSystemStudio/phoebus/pull/3977) "docker: update kafka and remove zookeeper" (roycpro) was merged on 22 Oct.

#### Open questions
- How should acknowledgements and configuration changes be authorized: an alarm server API, Kafka access controls, or something else?
- How can the same PVs belong to several alarm hierarchies (network, power, location)?
- Can Save & Restore handle mode switching for about 200,000 PVs?
- Is a facility-wide mode broadcast to all IOCs, including soft IOCs, feasible?
- Has NSLS-II migrated to KRaft?

#### Linked passages
- CEA alarm web and its API: Topic 2.
- Kafka topic-creation code cleanup was deferred to the afternoon.
- Screen auto-generation side conversation during the break: Topic 4.

### <a id="topic-4"></a>Topic 4: Auto-generating engineering screens (side discussion)

Wed 21 Oct 2026, 10:32 to 10:45. This was an informal conversation during the coffee break, not an agenda session. It relates to agenda 1.6 (Display Builder) and 6.2 (future features). The participants included NSLS-II beamline staff and Michael Davidsaver. Individual speakers are not always identifiable, and parts of the recording are fragmentary.

#### Discussion summary

**Single source of truth for IOCs and screens.**
- At NSLS-II, PhoebusGen is used as part of automated IOC deployment.
  - IOC configuration files and the database templates in the IOC source repositories generate the substitution files needed to start an IOC.
  - The aim is to generate the engineering screens from the same source.
- This is driven by broken screens: missing macros, and old CS-Studio macros in file paths that no longer work.
- The participant wants to fix as many as possible automatically. NSLS-II has about 30 beamlines and probably several thousand screens.
- Work focuses on beamlines for now, but the same stack is used for the accelerator.
- PV data has also been pushed into ChannelFinder, replacing what used to be done with MATLAB and EDM.

**Publishing an IOC's PV interface.**
- There is currently no easy way to find out which PVs an IOC provides, other than dumping them to a file with `dbl`.
- PVAccess can serve structured data. An IOC init hook could build a structure listing all PVs and their record types by parsing the PV names.
- Michael Davidsaver proposed an EPICS module, related to RecCaster, that is compiled into the IOC. Without any change to the startup script, it would serve one PVAccess PV describing the IOC's whole interface, available over the network.
- A filter could hide internal PVs, such as soft-channel records, and show only meaningful ones such as asyn parameters.
- A related idea from DLS was mentioned (name unclear in the recording; possibly ibek from epics-containers).

**Grouping matters.**
- Generating a screen straight from the PV list gives a "wall of text entries" in template order. That is fine while developing an IOC but poor for users.
- Two lightweight fixes were discussed:
  - add info tags that group PVs logically;
  - split database templates into logical files, with a top-level template that includes them.
- With either, a generator can put each group in its own group container. Split templates are also easier to read.
- The near-term goal is not full screens but launcher buttons that open existing screens: for example, one button per areaDetector detector that opens the standard detector screen, plus the correct list of plugins for each detector.

**Existing generators at other sites.**
- One site generates screens from EPICS template files. Templates are grouped logically, each group becomes a screen section, and an overall screen embeds as many instances as needed.
- Another project generates screens from CSV files that map FPGA registers to PVs, together with substitution files.
- PhoebusGen can be combined with a PSI project that parses database templates into Python structures. Together they generate Phoebus screens from a database template. Substitution files are handled by generating a screen per template and then embedding the instances, either all at once or through a drop-down selector.
- One participant wrote a converter that turns the PandABox web UI layout into a Phoebus screen layout.
- Another benefit: if the screen format ever changes, only the generator needs updating, not thousands of screens.

#### Key findings
- There is no standard way to discover an IOC's PV interface over the network.
- Generating screens from flat PV lists gives poor screens unless the PVs are grouped.
- Several sites already generate screens (PhoebusGen with the PSI template parser, CSV register maps, PandABox layouts).

#### Best practices highlighted
- Generate IOC configuration and engineering screens from a single source of truth.
- Group PVs with info tags or split templates so generators can lay out screens sensibly.
- Prefer generated launcher buttons to existing, well-designed screens over fully generated screens, where such screens exist.
- Generated screens make a future change of screen format cheap.

#### Project ideas
- An EPICS module, related to RecCaster, that publishes an IOC's PV interface (names, types, groups) as one PVAccess PV, built by an init hook (Michael Davidsaver).
- Auto-fix legacy macro and path problems in existing screens.
- Share and combine screen generators: PhoebusGen with the PSI template parser, and template-based and CSV-based generators.

#### Commitments / actions
- **Michael Davidsaver:** explore writing the EPICS module that publishes an IOC's PV interface ("we're going to see if we can write an EPICS module").

#### Open questions
- Which DLS project was meant (possibly ibek)?

#### Linked passages
- Display Builder topics later in the codeathon, if any.

### <a id="topic-5"></a>Topic 5: Agentic frameworks (Osprey) and the middle-layer services

Wed 21 Oct 2026, 11:03 to 11:33. Agenda 6.2 (AI/ML). Kunal Shroff chaired, and Lucy (BNL) gave an overview of Osprey.

#### Discussion summary

**Osprey overview (Lucy, BNL).**
- BNL is part of MODE, a newly announced Genesis Mission Phase 2 project. Its precursor was the SEED project, and its core is the use of Osprey at the participating labs.
- The project is led from Berkeley. Osprey itself is developed at ALS by Thorsten Hellert.
- Osprey is an LLM agent framework that connects to EPICS, the archiver, logbooks and other systems through a tool tier of MCP servers.
- A table in the report shows which services each lab has connected: ALS, APS, JLab, SLAC, Oak Ridge and Fermilab.
- BNL's roadblock has been that Osprey was built around Claude Code, which BNL is not allowed to use.

*Typical uses:*
- PV discovery. For example, Berkeley does semantic channel finding with natural-language requests combined with the archiver.
- Data analysis on archived data, such as writing Python to plot data or running SVD. One example found a 60 Hz error in a quadrupole power supply signal.
- "ARIEL", an agentic retrieval interface for e-logs.
- Details are in the IPAC'26 proceedings.

*Two approaches to acting on the machine:*
- **A: the agent writes code.** For example, at JLab Osprey wrote a PID loop on live PVs to reduce noise.
- **B: the agent runs existing, trusted applications or scripts, or suggests a configuration for a person to load.** For example, at SLAC it suggests Badger optimizer configurations.
- B is more conservative because the tools are already known to be safe. A raises the same trust question as an AI-written pull request: can you read and understand the code?

**Digital twins.** This work is at an early stage and labs take different approaches.
- SLAC uses LUME: a simulated accelerator (Elegant, Bmad, ...) publishes PVs with the same shape as the real ones, so the middle layer is identical for both.
- Fermilab has self-documented APIs with MCP servers for agents.
- BNL's current (Phase 1) project publishes a simulated orbit next to the real BPM data, filling in places with no BPMs. A sandbox mode suggests magnet settings for a desired orbit, which a person can then load into the real machine.
- ISIS has a similar online model. Its digital twin also generates OPI screens with virtual PVs, so users can interact with the loop before writing to the control system.
- Terminology was clarified:
  - a *virtual accelerator* is the simulation;
  - a *digital shadow* only reads live values and publishes derived ones, such as a tune throughout the ramp;
  - a *digital twin* closes the loop and can write back to the control system.
- Writing is hard at BNL, where many magnets may not be written by such tools.
- Control systems have never planned for a mirrored set of PVs, for example with a "virtual" suffix, served from a different source.

**APS experience.**
- APS is evaluating Osprey against using Claude directly, to see what Osprey actually adds.
- APS uses Claude Code "skills" from a marketplace (archiver, PV info, CLI tools) rather than MCP servers, and may add MCP later.
- APS is still gathering requirements: a web UI with user authentication, access to different models through Argo, and showing plots.

**How the Phoebus services fit in.**
- Kunal asked how much effort it took to connect the archiver, and how ChannelFinder (with live PV status) or the alarm logger (for anomaly detection) could be added.
- The archiver MCP is a separate service that talks to the Archiver Appliance; it is not built into the appliance.
- The Osprey documentation describes its MCP servers (built with FastMCP) and its ChannelFinder tool. Kunal noted that AI might eventually help with the alarm configuration problem discussed that morning.
- Osprey's logbook integration already supports several logbooks (ALS, JLab, Oak Ridge) through a facility adapter that normalizes entries. Supporting Phoebus Olog may therefore only need an adapter. Otherwise, the community could build its own MCP server following the same architecture.
- Kunal asked for a volunteer to build an Osprey-compatible MCP server for Phoebus Olog. Nobody volunteered.

**Direction.**
- Kunal proposed that the community keep Osprey in mind as it evolves the middle-layer services, so their interfaces suit agentic use.
- Lucy agreed: the project is funded for three years, and EIC should be able to reuse what the labs build rather than go its own way.
- The code is on GitHub; each site needs its own LLM API key. Sites can set Osprey up themselves, and BNL also meets the Osprey team weekly.
- A BNL colleague had some success with a local open-source LLM (Ollama).
- BNL's main obstacle has been the lack of a working EPICS system to test on. It now has an archiver deployment, and the Phoebus services (archiver, ChannelFinder) can be used for mock tests.
- Lucy suggested inviting the Osprey developer to give a talk at a developers meeting before the community starts anything big. Questions from the community will be collected for it.

#### Key findings
- Osprey is being adopted at ALS, APS, JLab, SLAC, Oak Ridge and Fermilab, mainly for PV discovery and analysis of archived data.
- Osprey connects to services through MCP servers. The archiver MCP is a separate service, not part of the Archiver Appliance.
- Osprey's logbook integration supports several logbooks through facility adapters.
- APS uses Claude Code skills rather than MCP.
- BNL was blocked by Osprey's dependence on Claude, and by having no EPICS test system.

#### Best practices highlighted
- Prefer agents that run existing, validated tools or suggest configurations for people to approve (approach B) over agents that write new control code.
- Make simulated accelerators publish PVs with the same structure as the real machine, so that tools work unchanged.
- Design middle-layer service interfaces with agentic use (MCP) in mind.
- Learn from the Osprey developers before starting large work.

#### Conclusions / agreements
- Keep the Osprey framework in mind as the middle-layer services evolve. Lucy (BNL) supported this so that EIC can reuse the work.
- Invite the Osprey developer to present at a developers meeting, and collect the community's questions first.

#### Issues / problems
- Policy restrictions on specific LLM providers block adoption at some labs (BNL).
- No volunteer was found for an Olog MCP server.
- Control systems have no convention for mirrored "virtual" PVs.
- Write access for agents and digital twins is restricted (BNL magnets).

#### Project ideas
- An MCP server or Osprey logbook adapter for Phoebus Olog. Related: SERVICES-OLOG-001 and AI-LOG-001.
- MCP access to ChannelFinder (including live PV status) and to the alarm logger for anomaly detection.
- A test bed using the Phoebus services (archiver, ChannelFinder) to try Osprey at a site without a production EPICS system.
- Local LLMs (Ollama) for sites that cannot use cloud models.

#### Commitments / actions
- **Lucy (BNL) and the chair:** collect community questions and invite the Osprey developer to give a talk at a developers meeting.
- **Unassigned (no volunteer):** build an Osprey-compatible MCP server or logbook adapter for Phoebus Olog.
- **Lucy:** share the Osprey GitHub link (posted in the meeting chat) and her slides.

#### Open questions
- Can Osprey's existing logbook adapter support Phoebus Olog, or is a new MCP server needed?

#### Linked passages
- AI-ready archived data and MCP for archiver retrieval: Topic 1.3.
- Olog AI search: Topic 6.
- Alarm-log analytics: Topic 3.

### <a id="topic-6"></a>Topic 6: Olog: AI search, Python client and the legacy Olog

Wed 21 Oct 2026, 11:33 to 12:07. Agenda 2.4 (Olog Integration) and 6.2. Kunal Shroff chaired; the session was moved from earlier in the morning. Lunch followed.

#### Discussion summary

**Olog usage.** A show of hands found that roughly 2 to 10 of those present use the old Olog or the new Phoebus Olog.

**Search improvements.**
- Georg (ESS) implemented a generic keyword search. Typed keywords are matched against title, description and author, and results are ranked by score. This does not use embeddings.
- An NSLS-II student implemented AI search for the Phoebus Olog using Elasticsearch embeddings and Spring AI.
  - **Hybrid search:** title and description are embedded, and all other fields are treated as metadata. The standard filters, such as logbook, still apply.
  - The LLM builds the Elasticsearch query from the natural-language request. An optional "analysis" step then reads the top 20 hits and points out patterns.
  - Embeddings are stored in a separate index linked by entry ID, so the original entry, including attachments, is shown.
  - There is a one-time ingestion of existing entries. After that, new and updated entries are embedded automatically.
  - Through Spring AI the model is set in a configuration file and can be swapped: NSLS-II uses Azure, and Ollama, Hugging Face and others are supported. One site (not identified) uses Mistral AI hosted on site, which should work the same way.
  - Costs are tokens for building the embeddings (Stuart Wilkins, NSLS-II group leader, told the student not to worry about them) and a modest amount of storage.
  - A local model needs much more than a standard Olog VM. The student had to use the mentor's larger machine. A local test with about 50,000 entries is feasible on a big server, but not on a laptop.
  - Documentation exists in the student's repository but needs improving.
  - The student first tried MCP, but found it slow and error-prone. Giving the LLM clear, narrow instructions was faster and more reliable.
  - The feature is shown in the Olog web client. The Phoebus client could use the same REST API, since nothing happens in the client itself.
  - Entries with no text (only an attachment or title) are a limitation. Image analysis is possible with Spring AI but was not explored.
  - The PR is ready and waiting for NSLS-II to merge it.
- ESS is keen to try the AI search on its large live Olog data.
- Kunal offered to work on installation documentation, and on how to swap LLMs, with a focus on local LLMs to avoid data-privacy concerns.
- CEA asked whether the same approach could search ChannelFinder, and suggested looking at PVInfo, which also uses Elasticsearch. The architecture looked similar, so it is probably possible.

**Python client.**
- The old Olog had a Python client used in Bluesky and other Python workflows. The new Olog has no up-to-date, maintained Python client.
- The NSLS-II student is fixing an outdated one written by someone else.
- Kunal asked for it to be moved into the Olog GitHub organization so that several people can maintain it.
- Ankit (NSLS-II) is interested in working on it, and ESS agreed to help by testing it.

**Sunsetting the legacy Olog.**
- NSLS-II still uses the old Olog and has mentioned needing about three more years. Kunal called this too long, given limited resources and how clunky the old service is.
- Kunal asked FRIB (Cyril and Anthony) to work out their timelines. FRIB also uses the legacy Olog, in an early version ("the one from Eric").
- No end-of-support date was set.

#### Key findings
- AI search for Phoebus Olog (hybrid embeddings with Spring AI) is implemented and its PR is ready.
- The LLM is swappable by configuration (Azure, Ollama, Hugging Face, Mistral). Embedding cost is mostly a one-time ingestion.
- Narrow, well-instructed LLM use worked better than MCP for this search.
- The new Olog has no maintained Python client.
- NSLS-II still relies on the legacy Olog.

#### Best practices highlighted
- Embed only title and description, keep other fields as metadata, and link embeddings to the original entry by ID.
- Use a provider-agnostic layer (Spring AI) so sites can choose cloud or local LLMs.
- Prefer local LLMs where data must stay on site.
- Put shared client libraries in the project's GitHub organization so they can be maintained together.

#### Conclusions / agreements
- Work together on deploying the Olog AI search and on its installation documentation, including how to swap LLMs.
- Move the Olog Python client into the Olog GitHub organization.
- The legacy Olog needs a sunset plan; three more years is considered too long.

#### Issues / problems
- The AI search PR is waiting to be merged at NSLS-II.
- Local embedding and LLM inference needs significant hardware.
- Cloud LLMs (Azure) send data off site.
- Entries without text cannot be searched semantically, and attachments are not embedded.
- There is no agreed end-of-support date for the legacy Olog.

#### Project ideas
- Deploy and document the Olog AI search, including local LLM setups. Related: SERVICES-OLOG-001 and AI-LOG-001.
- Add AI search to the Phoebus Olog client.
- Apply the same approach to ChannelFinder and PVInfo.
- Maintain a Python client for the new Olog.
- Embed attachments and analyse images.

#### Commitments / actions
- **Kunal Shroff with the NSLS-II student, and ESS:** deploy the Olog AI search, give feedback and produce installation documentation.
- **NSLS-II student:** send the link to the Python client repository. Kunal asked for it to be moved to the Olog organization.
- **Ankit (NSLS-II) and ESS:** help update and test the Olog Python client.
- **NSLS-II and FRIB:** clarify how long the legacy Olog must be supported, so a sunset date can be decided.

#### Open questions
- When should support for the legacy Olog end?
- Which site uses Mistral AI on site?

#### Linked passages
- Olog data migration tool: Topic 15.
- Osprey logbook integration (ARIEL) and MCP: Topic 5.

### <a id="topic-7"></a>Topic 7: RecSync, RecCeiver, RecCaster and the ChannelFinder ecosystem

Thu 22 Oct 2026, 09:33 to 10:23. Agenda: ChannelFinder / RecSync. Kunal Shroff presented and chaired. Michael Davidsaver, the original author of RecSync, Ralph Lange (ITER), Tynan Ford and Murali Shankar contributed. The recording starts in the middle of the introduction.

#### Discussion summary

**How the ecosystem has changed.**
- RecSync used to be one repository holding both the server (RecCeiver) and the EPICS IOC module (RecCaster). Over the last six months they were split:
  - RecCaster is now a standalone repository in the ChannelFinder organization, built like any other EPICS module.
  - The RecSync repository will become just the protocol definition.
- The old Python RecCeiver is built on Twisted. Michael Davidsaver said it was only ever meant to be a working prototype.
- For EIC, which needs to scale to tens of millions of PVs, Kunal wrote a new Java RecCeiver.
  - It implements the same protocol and uses JDK 25 virtual threads, so it can hold hundreds of thousands of mostly idle connections without bloat.
  - It was tested with up to 2 million PVs and performed significantly better than the old CF store.
  - Tynan has deployed it, but not at production scale.
  - It is available as a Docker image, and a single properties file holds its configuration.
- The speed-up comes mainly from ChannelFinder, not from the language. ChannelFinder now has batch and update endpoints.
  - The old CF store pulled all of an IOC's channels into memory, merged them and wrote them back, and made many individual calls.
  - The new RecCeiver lets ChannelFinder do this work, so the network is now the bottleneck.
  - Kunal: "I don't believe this is a language problem." Even the Python RecCeiver would improve a lot if its REST calls were rewritten.
- **Rust.** There is enthusiasm for a Rust RecCeiver, but none exists yet. The architecture is modular, so a better implementation could be swapped in later. Kunal would prefer that effort goes into maintaining and improving one implementation.
- **Running both receivers.** The two RecCeivers cannot usefully run at the same time, because an IOC connects to only one.

**Documentation and getting started.**
- The quick start must show a complete minimal configuration, including ChannelFinder credentials and location. ChannelFinder's getting-started documentation has historically been "very expert-friendly".
- Michael asked for a documented list of common info tag names and value syntax, as suggested defaults and best practice. Examples are archiving tags and the property allow-list.
- ChannelFinder is designed to be the centre of an ecosystem, so on its own it is not very useful. Its ecosystem setup has never been documented in one place.
- Proposals:
  - a section in the ChannelFinder documentation listing the available plugins, such as the archiver, with links from the RecCeiver and RecCaster docs;
  - a training VM running the whole pipeline, where putting an info tag in a database file results in the PV being archived.

**Channel status: adding an "undefined" state.**
- Today a channel is *active* while RecCeiver holds the IOC connection, and *inactive* once it is lost. The issue is how RecCeiver going down should be shown.
- RecSync's strength is that ChannelFinder knows as soon as an IOC is gone, even when no client is connected.
  - Without clients, a CA client would only see a disconnect through beacons and an echo, roughly 30 seconds in the worst case.
- **Proposal:** add a third state, *undefined*, for when the RecCeiver daemon shuts down (clean-on-stop). This shows that status is unknown, rather than that the IOCs are down.
  - On start-up, channels are *inactive* until IOCs reconnect.
  - ChannelFinder itself knows nothing about RecCeiver, and Kunal prefers not to couple them more tightly.
- ESS disabled clean-on-start because it is very slow with the old CF store. It wants it to stay configurable.

**Containers, unicast and dynamic DNS.**
- Docker images exist for most pieces. One is still missing: an IOC with RecCaster built in. With that, the whole pipeline could run in containers.
- Moving away from UDP makes containers much easier: no host networking, and PV Access name resolution can go through the CF name server over unicast.
- The last UDP dependency is RecCaster waiting for the RecCeiver beacon.
  - **Plan:** an optional, configurable list of RecCeiver addresses or host names in RecCaster, like CA/PVA address lists, keeping UDP available.
  - Today RecCaster pushes its data once per connection.
- **Michael Davidsaver's point.** Container users want DNS names that resolve to several IP addresses, and mappings that change at runtime.
  - In Kubernetes, ChannelFinder and RecCeiver may change IP address at any time.
  - Clients must periodically redo name lookups rather than resolve once at start-up. Most C APIs do not expose the DNS TTL.
  - The JVM has a DNS cache TTL setting.
  - This affects RecCaster (C/C++) and the Java RecCeiver alike.
- **Agreed approach:** start with configurable lists and periodic re-resolution, and capture the Kubernetes concerns in the issue.

**Monitoring.**
- Adding Spring Boot Actuator to RecCeiver gives basic health monitoring (uptime, memory, CPU), as was done for the other middle-layer services at the last codeathon. Kunal called it a good first task.
- Michael asked for protocol-level metrics too, which matter more for troubleshooting: number of connected clients, last beacon sent, last message received.
  - At sites with thousands of IOCs, searching logs is impractical.

**CF name server.**
- The CF name server supports only PV Access. A Channel Access name server is wanted, ideally one server speaking both protocols.
  - It needs only a small part of each protocol: name resolution, plus TCP name lookup for PVA.
  - Kunal hopes EIC will be PVA-only, but CA support is needed in practice.
- The CF name server relies on the environment variable that QSRV defines. QSRV2 sets both variables, so tools can read the common one and nothing needs to change. QSRV1 exposed the port number and QSRV2 originally did not. This has since been fixed in a recent PVXS release. It needs documenting, because Fermilab had been confused by it.
- **Choosing a Java CA library.** Cosylab originally wrote JCA but was never asked to maintain it, so its status is unclear. Cosylab replaced it with Jackie in save-and-restore because of a bug (see Topic 8). Jackie, by Sebastian Marsching (aquenos), is client-only. Kay Kasemir had looked at using it. A name server does not need a full server implementation.

**Versioning.**
- Now that RecCeiver and RecCaster are separate, they will have separate versions. The contract between them is the protocol.
- **Agreed:** derive versions from the RecSync protocol version. The major version is the protocol version, and the minor version is specific to each implementation.
  - The protocol header already has a version field: the first byte must be 0, and clients reject anything else.
  - In about 15 years no protocol version change has been needed. The protocol does not support IPv6.
- PVA structures do not matter to ChannelFinder. Name resolution works on PV names only, with the same semantics as CA.

**Gateways (raised by Ralph Lange).**
- Clients that look up IOC addresses in the CF name server would bypass gateways.
- At a firewall boundary they could not connect anyway. The gateway should use the name server on its upstream side, where the IOCs are, while downstream clients use only the gateway.

#### Key findings
- The Java RecCeiver (JDK 25, virtual threads) handled 2 million PVs and is a drop-in replacement for the Python one.
- RecSync performance depends on ChannelFinder's batch and update endpoints, not on the language.
- RecSync gives near-immediate notice when an IOC goes away, even with no clients connected.
- The RecSync protocol has been stable for about 15 years and already carries a version byte.
- QSRV2 sets both environment variables, so the CF name server needs no change.

#### Best practices highlighted
- Choose one RecCeiver implementation for each IOC network.
- Turn RecCaster info tags into a documented set of suggested names, and document the property allow-list.
- Make quick starts complete enough for a non-expert to get running.
- In container environments, prefer unicast address lists and re-resolve DNS names periodically.
- Gateways should use the CF name server upstream. Clients downstream should use only the gateway.

#### Conclusions / agreements
- Recommend that sites try the Java RecCeiver. Kunal asked for advice on what performance targets to aim for.
- Add an *undefined* channel state for when RecCeiver shuts down, and keep clean-on-start configurable.
- Add optional unicast address lists to RecCaster and the RecCeivers, with DNS re-resolution taken into account.
- Version RecSync components from the protocol version: major = protocol, minor = implementation.
- A CA name server is needed in addition to PVA. Ideally one server would speak both.

#### Issues / problems
- The RecCeiver and RecCaster documentation lacks a complete quick start, links to the properties file, and info tag conventions.
- The old CF store is very slow at clean-on-start and clean-on-stop, so ESS disabled clean-on-start.
- Unicast from RecCaster to RecCeiver is not supported.
- Containers and Kubernetes change IP addresses at runtime, and few EPICS components re-resolve names.
- When something in the RecSync ecosystem fails, it is not obvious why.

#### Project ideas
- A Docker image of an IOC with RecCaster, to complete a containerized pipeline.
- Unicast and host-name lists in RecCaster, plus periodic DNS re-resolution.
- Spring Boot Actuator health checks in the Java RecCeiver, then protocol metrics: clients, beacons, last messages.
- A Channel Access name server, or a combined CA/PVA CF name server.
- The *undefined* channel state on RecCeiver shutdown.
- ChannelFinder ecosystem documentation: plugins, info tags and allow-list, plus a training VM showing the full pipeline.

#### Commitments / actions
- **Kunal Shroff:** add a full configuration example, including ChannelFinder credentials and location, to the Java RecCeiver quick start. Link the properties file from the README.
- **Tynan Ford:** RecCaster unicast mode, covering the issue write-up and implementation. Kunal asked Tynan to help formulate the issue and later confirmed Tynan as the volunteer. Follow-up: [ChannelFinder/reccaster#24](https://github.com/ChannelFinder/reccaster/issues/24) "RecCaster unicast mode" (tynanford, open).
- **Murali Shankar:** help implement the Channel Access name server.
- **Volunteer, name not captured:** document the ChannelFinder ecosystem, info tags and allow-list. Kunal asked "Who's volunteering for that? Thank you."
- **Unassigned:** document that QSRV2 sets both environment variables, and the PVXS release that fixed the port number.
- **Unassigned:** set an initial RecSync version and align the other components with it.

#### Follow-up after the codeathon
Status checked on 12 Nov 2026.
- [reccaster#24](https://github.com/ChannelFinder/reccaster/issues/24) "RecCaster unicast mode" (tynanford, open).
- [recceiver#7](https://github.com/ChannelFinder/recceiver/pull/7) "update startup/shutdown cleanup" (roycpro) was merged on 2 Nov.
- [recceiver#8](https://github.com/ChannelFinder/recceiver/pull/8) "Spring4" (shroffk, opened 28 Oct, open).

#### Open questions
- What performance targets should the Java RecCeiver aim for?
- How should RecSync components behave when ChannelFinder or RecCeiver moves in Kubernetes?
- Should one CF name server speak both CA and PVA?

#### Linked passages
- Michael Davidsaver's IOC PV-interface module and RecCaster: Topic 4.
- Archiver Appliance and ChannelFinder-driven archiving: Topic 1.
- PVInfo: Topic 2.
- Gateways: Topic 13.

### <a id="topic-8"></a>Topic 8: Save-and-restore: facility workflows, UI, client vs service restore, restore ordering and deadlocks

Thu 22 Oct 2026, 13:37 to 15:04. Agenda: Save-and-restore ("collect usage workflows from facilities"; restore from client vs service). Kunal Shroff chaired. Georg (ESS, maintainer) and Jure Varlec (Cosylab) led much of it. Others came from ISIS/UK, CEA and EIC.

#### Discussion summary

**How facilities use it**
- **ESS** is probably the largest installation: several hundred configurations and thousands of snapshots. ESS is still commissioning, and users are still learning how to organize snapshots.
- Kunal recalled the same pattern at NSLS-II. Configurations and snapshots grow fast during commissioning, then plateau.
- **Cosylab customers.** Several have it delivered. One uses it heavily, mainly for **large waveforms** such as power-supply ramps.
  - Comparing 10,000-point waveforms in a table is impractical.
  - Cosylab built that customer a separate Python tool. It compares a PV's live value, any snapshot and any archived value, with site-specific features such as scaling to compare different power supplies.
  - **Lesson:** "sometimes it might make sense to just make a separate site-specific tool" rather than grow save-and-restore.
- NSLS-II had a simpler built-in waveform comparator: a pop-up from the snapshot table showing the snapshot and live waveforms.

**Restoring part of a composite snapshot (ESS request)**
- ESS users want to restore only some of the snapshots inside a composite snapshot.
  - In the aggregated table they cannot see which snapshot each value comes from.
  - Expanding composite snapshots in the tree view was added the previous week ([phoebus#3963](https://github.com/ControlSystemStudio/phoebus/pull/3963)).
- **Proposal (Georg):** select a subset in the tree, right-click, and restore. Behind the scenes a temporary in-memory "virtual" composite snapshot would be built, probably not saved.
- **Concerns:**
  - It works against the separate goal of simplifying a UI that is already complex.
  - It cannot be restored from the service, which does not know about the virtual snapshot.
  - It is "an awful lot of work to address a problem of not properly creating configurations". Georg agreed.
- ESS also wants a **quick filter**. Georg still has to agree its exact behaviour with stakeholders: making the view more compact or highlighting matches.

**Hiding table columns (UK request)**
- UK facilities find the table too dense and want to hide columns, ideally with the choice saved.
- JavaFX tables support column hiding, but not with **nested columns**, which the save-and-restore table uses. The same request exists for the alarm log and ChannelFinder tables.
- **Saving the choice is harder:**
  - Many control rooms use **shared operator accounts**, so whoever hides a column is not the next person to look at the table.
  - EIC is the exception: everyone logs on to one MCR account, but changes are mapped back to the individual.
- Alarm table: Kay Kasemir had previously argued that some alarm-table columns should not be hideable, because hiding them could confuse. This led to a short debate about user responsibility versus preventing user error.
- Ideas:
  - a "show all columns" reset;
  - named column layouts that users can choose, similar to Phoebus layouts.

**Restore from client vs from service: security and UI**
- An issue asks for an option to remove "restore from service".
- **Security:**
  - A restore from the service effectively bypasses Channel Access security, since all puts come from one machine.
  - A user whose network setup blocks a client-side restore could still restore through the service.
- **Why service restore exists:**
  - to restore from outside the controls network;
  - to support REST-based scripting, such as triggering a restore from Python;
  - to drive OPI buttons.
- **Mitigation:** save-and-restore already has authentication, and a restore through the service requires an authenticated, authorized user. Auditing is then **split**: the CA put log no longer shows who did it, so the save-and-restore log has to be checked too.
- **UI confusion:** operators don't understand the choice. Cosylab: "I can't imagine a reason why you would sometimes want one and sometimes the other as a user."
  - Counter-view: the choice depends on where Phoebus runs, for example inside the network or behind a read-only gateway. That makes it a client-side setting, not a per-restore choice.
- **Where restore buttons can work:**
  - An OPI button with client-side restore can't work in DBWR or other web runtimes, since there is no Phoebus behind them. Only a service restore would work there.
  - Today an OPI button opens the snapshot view and the user presses Restore. A dedicated "retrieve and restore" action could combine the two, but needs refactoring.
- **Kunal's use case:** a drop-down at the top of the alarm tree to switch the whole alarm configuration "mode". It would restore saved sets of HIHI/HIGH/LOW/LOLO limits through save-and-restore. A "take snapshot" button on overview screens is similar.
- Georg has turned down requests for an OPI widget that shows the snapshot tree. "There should be no widget. It should just be a button action. And if you can't fit it in a button action, then please don't put it in an OPI screen."

**Put ordering and parallel restore**
- Some devices accept a setpoint only in the right mode, so the mode PV must be written first. Parallel restore can't guarantee this, and dependencies between PVs won't be modelled.
  - A participant proposed options to **disable parallel restore**: one global and one per configuration.
  - Current versions always restore in parallel, and in most cases this is not a problem.
- **Counter-arguments:**
  - Writing in order doesn't guarantee the hardware is ready.
  - EPICS doesn't guarantee that CA puts arrive in the order sent.
  - Proper sequencing needs put-with-callback, readbacks, timeouts and tolerances, and IOC support. At that point save-and-restore becomes a sequencer.
- Few facilities use put-with-callback. Kay's scan engine is the main user, and core-pv supports it.
- ESS has a dedicated sequencer application because it judged that changing save-and-restore was not feasible.
- **Recommendation:** use a sequencer (such as ESS's, or the scan engine), or split the configuration into sequential sub-configurations: first all the mode PVs, then the setpoints.
- Kunal suggested an OPI pattern, recalling an ITER screen: a restore button per stage, LEDs that turn green when a stage completes, and text explaining each step.

**Deadlocks during client-side restore (Jure Varlec, Cosylab)**
- A Cosylab customer saw the client **hang during restore**: it does some writes and then stops.
- Steps so far:
  - Updating JCA moved the deadlock elsewhere.
  - Switching save-and-restore to **Jackie** through core-pv avoided it. Jackie is not currently included in the save-and-restore build.
  - A further race affects Jackie and PVA, and would affect JCA. It is triggered by the slow restore of large waveforms.
  - A Claude-generated patch exposed yet another deadlock, so they patched restore to run **serially**.
- They have a Claude-generated threading report but did not trust it enough to post it. They reproduced the problem easily with generated databases of **scalar** PVs in a soft IOC.
- Cosylab has **no budget** for deeper threading work.
- **Kunal:**
  - JCA is very old and hard to fix: it has its own linked lists from before Java collections existed. "This is going to be the third [time] where I have brought up the idea of a replacement for JCA", but no facility will take the risk.
  - He and Abraham Wolk would like to explore moving PV I/O to **virtual threads**, for example one per PV.
- Georg: client restore submits writes to a thread pool as fast as possible, with the pool size as the limit and the default EPICS timeout on writes. ESS has not seen the deadlock.
- **Agreed:** a reproducer (database plus scripts) is more valuable than the AI report.

#### Key findings
- Large waveforms in snapshots are hard to compare in a table. Site-specific comparison tools have been more successful.
- Service restore bypasses CA access security, but is protected by save-and-restore authentication and authorization. Auditing is split between the two.
- Parallel client restore can deadlock or race under load with some CA libraries. Serial restore avoids it.
- Put order cannot guarantee correct device sequencing.

#### Best practices highlighted
- Organize configurations so that subsets can be restored on their own, instead of relying on partial restores of composites.
- For ordered restores, split configurations into sequential stages, or use a sequencer.
- Keep OPI integration to button actions, with no snapshot-tree widgets in displays.
- For specialized waveform comparison, build a site-specific tool.
- Report concurrency bugs with a minimal reproducer rather than an AI-generated analysis.

#### Conclusions / agreements
1. **Server setting to disable service restore** (read-only mode, as PVWS has). "Complete consensus."
2. **Restore mode as an administrator preference, not a user choice.**
   - The values are client, server or both; the default is client.
   - The UI choice is shown only when both are enabled, which is useful for developers.
3. **Service restore requires authentication.**
   - OPI buttons and the alarm-tree mode switch should prompt for credentials and trigger a service restore when it is enabled.
   - For the first iteration, OPI-button restore supports the server only; this also works in DBWR.
4. **Column hiding.** Enable it, and persist it as a first step, in the save-and-restore, ChannelFinder and alarm-log history tables. The alarm tree table is left out for now. Suggested as a codeathon project.
5. **Ordered restore.** Recommend sequential sub-configurations or a sequencer, rather than adding ordering to save-and-restore.
6. **Partial restore of composites.** No agreement. It was considered against UI simplification and a workaround for poor configuration organization. Georg will discuss it with ESS stakeholders.

#### Issues / problems
- The client UI exposes the client vs service choice, which operators don't understand.
- Service restore lets users bypass network-level write restrictions.
- Nested table columns block JavaFX's built-in column hiding.
- Client restore deadlocks or races with JCA, and possibly with Jackie and PVA, under load.

#### Project ideas
- Server-side switch to disable service restore.
- Restore-mode preference (client/server/both), hiding the UI choice when only one is allowed.
- A combined "retrieve and restore" action for OPI buttons, plus a "take snapshot" action.
- Alarm-tree drop-down to switch alarm-limit configurations through save-and-restore.
- Column hiding and persistence in the save-and-restore, ChannelFinder and alarm-log tables; possibly named column layouts.
- Quick filter in the snapshot view.
- Move PV I/O in core-pv to virtual threads; long-term replacement of JCA.

#### Commitments / actions
- **Georg (ESS):** talk to ESS stakeholders about the restore-mode default and about the quick filter.
- **Jure Varlec (Cosylab):** provide a reproducer (database and scripts) for the restore deadlock, and open an issue. Aimed for the same day.
- **Georg:** look at Jure's AI threading report alongside the reproducer.
- **Unassigned (codeathon project):** column hiding in the save-and-restore, ChannelFinder and alarm-log tables.

#### Follow-up after the codeathon
- [phoebus#3997](https://github.com/ControlSystemStudio/phoebus/pull/3997) "Add preference to choose snapshot restore policy: sync or async" (georgweiss) was merged on 2 Nov 2026.
  - It adds `org.phoebus.saveandrestore.util/restorePolicy=ASYNCHRONOUS|SYNCHRONOUS` (default asynchronous), as a workaround for JCA deadlocks.
- [phoebus#3983](https://github.com/ControlSystemStudio/phoebus/issues/3983) "jackie: array PV values can contain corrupted data" (Jure Varlec, open).
  - Jackie's `ValueConverter` wraps a reused buffer, so snapshots of large waveforms may contain invalid data.
- [phoebus#3989](https://github.com/ControlSystemStudio/phoebus/pull/3989) made the snapshot-view filter case-insensitive (abrahamwolk, merged).

#### Open questions
- Should service restore remain available to Phoebus clients at all, or only to scripts and the REST API?
- How should failed writes be reported when a restore is triggered from an OPI button?
- Is the partial restore of composites worth its UI complexity?

#### Linked passages
- Alarm limits and configuration management: Topic 3.
- JCA vs Jackie: Topic 7.
- The deadlock discussion continued informally at about 16:40, including a problem with the Jackie/PVA wrapper.

### <a id="topic-9"></a>Topic 9: Display Builder Tabs widget: Active Tab vs Default Tab

Wed 21 Oct 2026, 14:08 to 14:12. Agenda: Display Builder. This was a short design review during the afternoon working session (Topic 16); Kunal Shroff asked for a "UI philosophical discussion".

#### Discussion summary
- In the Tabs widget, the *Active Tab* property also acts as the default tab at runtime. Clicking a tab in the editor changes the property, so saving after editing another tab silently changes which tab opens. Participants found this unintuitive and annoying.
- **Proposal (Kunal):** break the link, even though it was originally kept for backward compatibility.
  - *Active Tab* becomes runtime state and is no longer an editable property.
  - A new *Default Tab* property, set explicitly by the user, decides which tab opens. It defaults to the first tab.
  - The room broadly agreed.

#### Conclusions / agreements
- Separate the two meanings: *Active Tab* becomes runtime state only, and a new, user-set *Default Tab* property decides which tab opens. This deliberately breaks backward compatibility.

#### Issues / problems
- Selecting a tab in the editor silently changes which tab opens by default at runtime.

#### Project ideas
- Add a *Default Tab* property to the Display Builder Tabs widget.

#### Commitments / actions
- **Unassigned:** open an issue or PR to add a *Default Tab* property and stop exposing *Active Tab* as editable.

#### Linked passages
- Afternoon working session: Topic 16.

### <a id="topic-10"></a>Topic 10: Alarm-sensitive widgets and alarm colors

Wed 21 Oct 2026, 14:31 to 14:50 and 16:22. Agenda: Display Builder and alarm visualization. This discussion and poll took place during the afternoon working session (Topic 16). Kunal Shroff chaired.

#### Discussion summary
- **Colors.**
  - Boolean widget colors have nothing to do with alarms.
  - The alarm colors (major, minor, and the acknowledged variants) are configurable in some applications and hard-coded in others.
  - Loic (CEA) is listing where colors are configurable and where they are hard-coded, so they can be consolidated. This continues a long color discussion held on Oct 20, which was not recorded.
- **Proposed order of work:**
  1. Define a central set of well-defined, named, configurable colors as the single authority.
  2. Then add an optional "alarm sensitive" checkbox to widgets, so they use the alarm color built into the widget rather than rules.
- **Expected requests.** NSLS-II is likely to ask for alarm-sensitive shapes (fill color), because many of its vacuum screens show gauges as shapes.
- **Poll: alarm sensitivity beyond the border.**
  - **LEDs:** everyone with red/green LEDs makes them alarm sensitive using rules, because it is not supported out of the box.
  - **Other widget types used this way:** text update (background), shapes, polygons, action buttons, progress bars, tanks, thermometers, and the two symbol widgets.
  - **ITER symbol widgets:** ITER provides the symbol library and the alarm functionality exists, but Ralph Lange did not know how widely it is used.
  - **Labels and buttons** would cover more than 90 % of cases.
  - **Scripts:** some sites use scripts. The EDM to Phoebus auto-converter always generates scripts for this.
- **Implementation.** This must be done for each widget, not in a common base class, because the property that should follow the alarm differs between widgets.
- **Acknowledgement state.** Alarm awareness can go beyond severity and status, for example showing whether an alarm has been acknowledged. That information is available through the alarm data source.
- **Follow-up.** At 16:22 Kunal opened a GitHub discussion titled "Color alarm sensitive widgets" and asked everyone to add what had been discussed.

#### Key findings
- Alarm sensitivity beyond the border is used everywhere. It is done with rules or scripts because widgets do not support it, and auto-converted EDM screens rely on scripts.
- The alarm colors are not defined in one place.

#### Conclusions / agreements
- First consolidate the color definitions into one configurable, named set, then add opt-in alarm sensitivity to individual widgets.
- Once the colors are settled, open an issue listing the widgets that need alarm visualization beyond the border.

#### Issues / problems
- Alarm colors are hard-coded in some places and configurable in others; there is no single authority.
- LEDs and other widgets do not support alarm-sensitive fill or background out of the box, so every site uses rules or scripts.

#### Project ideas
- A central, named, configurable color authority for Phoebus.
- Opt-in alarm sensitivity, beyond the border, for individual widgets.
- Show alarm acknowledgement state in widgets through the alarm data source.

#### Commitments / actions
- **Loic (CEA):** finish the list of places where colors are hard-coded or configurable.
- **Kunal Shroff:** started the GitHub discussion "Color alarm sensitive widgets". **All participants:** add the poll results and use cases to it.
- **Unassigned:** after the colors are consolidated, open an issue for opt-in alarm sensitivity in LED, text update, shape, polygon, action button, progress bar, tank, thermometer and symbol widgets.
- **Ralph Lange (ITER), maybe:** find out how much ITER uses alarm sensitivity in its symbol widgets.

#### Open questions
- What was concluded in the Oct 20 color and translation discussion? It was not recorded.
- The GitHub discussion "Color alarm sensitive widgets": no link was recorded.

#### Linked passages
- Afternoon working session (Loic's color list): Topic 16.
- Alarm services: Topic 3.
- Auto-generating screens: Topic 4.

### <a id="topic-11"></a>Topic 11: Showing truncated values in Text Entry and Text Update (PR #3928)

Thu 22 Oct 2026, 11:25 to 11:29 and 12:04 to 12:15. Design review of [phoebus#3928](https://github.com/ControlSystemStudio/phoebus/pull/3928) "Feat: truncate overflowing content of TextEntry and TextUpdate, append ellipsis" (gabriel-desmarchelier, CEA). Kunal Shroff led the review. Participants from CEA demonstrated the PR.

#### Discussion summary
- **The problem.** A long number or string in a Text Update or Text Entry widget is silently cut off, so operators cannot tell that the value is incomplete. Kunal: "which is not good".
- **The PR's approach.**
  - It truncates the displayed text and appends an ellipsis.
  - It uses a focus property listener, so the full text is shown while the widget is being edited. Being event-driven, this was welcomed.
  - In the demo, it kept two values: the original text, and the ellipsized text calculated from font and size. The original is used while editing, and it is what gets written to the PV.
  - Whether the ellipsis goes at the start or the end is configured in the widget.
- **Editing workflows.**
  - When a user selects all and retypes, truncation does not matter.
  - When a user wants to change only the last 2 or 3 characters, the editing behaviour matters.
- **Performance.**
  - Text Entry and Text Update are the most heavily used widgets; a display can have thousands of them. There was concern about thousands of focus listeners.
  - The response was that the extra work happens only while editing, so it should be fine.
- **JavaFX native ellipsis.**
  - Someone asked why the native JavaFX ellipsis was not used.
  - Concern: if the displayed text is actually replaced by an ellipsis, it could be written back to the PV. The PR's approach avoids this.
  - Font sensitivity was a known problem with BOY. Manually sizing widgets was never consistent across operating systems, NX and fonts.
- **Demo results.**
  - Several cases did not show the ellipsis correctly. The cause may be borders or margins in the width calculation.
  - All the broken cases were left-aligned; right-aligned widgets worked as expected.

#### Key findings
- Truncation is calculated from font and size and is unaffected by display zoom.
- Left-aligned widgets show inconsistent ellipsis behaviour. Right-aligned widgets are consistent.

#### Conclusions / agreements
- The feature is wanted. Kunal: "this is very useful", but "we need to work on this" before merging.
- Keep the full original value separate from the displayed text, so an ellipsis can never be written to a PV.
- Participants volunteered to test it. It was considered better for someone other than the issue's reporter to test.

#### Issues / problems
- The ellipsis is wrong for left-aligned widgets, possibly because of border or margin calculations.
- The behaviour when partially editing a truncated value is unclear.

#### Commitments / actions
- **Demo participants:** take screenshots of the inconsistent cases and add comments on the edge cases to PR #3928.

#### Open questions
- Is the custom calculation or the JavaFX native ellipsis better in the long run?

#### Linked passages
- None.

### <a id="topic-12"></a>Topic 12: Phoebus launch when the `-settings` file is missing

Thu 22 Oct 2026, 12:15 to 12:21, revisited at 13:36. Kunal Shroff chaired. The request came from ISIS, in [phoebus#3960](https://github.com/ControlSystemStudio/phoebus/issues/3960) "Start and log error (rather than refusing to start) if -settings flag points to nonexistent file" (LordIdra).

#### Discussion summary
- **The request (ISIS).** One Phoebus product is deployed in several places, each with its own `settings.ini` to override defaults.
  - Some places need only the defaults, but Phoebus refuses to start if `-settings` points to a missing file.
  - So ISIS has to create empty `settings.ini` files. They asked for a logged error instead of a refusal to start.
- **The other side.** Other sites rely on the failure to start.
  - Default settings are "not very useful": everything points to localhost.
  - Refusing to start makes you fix the problem. It also prevents operators from running Phoebus with an unknown configuration while assuming it is correct.
  - ESS, Berkeley and probably ITER rely on this.
- **A preference won't work.** The behaviour concerns loading preferences, so it cannot itself be a preference. It has to be a program argument.
- **After lunch.** Kunal reported that the topic, and preference management generally, came up again over lunch with Ralph Lange and others. Everyone agreed it is "a complicated and messy thing". Kunal said he liked the force-start option.

#### Conclusions / agreements
- Add a new, more permissive program argument, for example a "force start" option. With it, Phoebus logs an error and starts when the settings file is missing.
- Keep the current default: refuse to start. The new argument is opt-in, so no existing deployment breaks.
- Record on the issue that other sites rely on the failure to start.

#### Commitments / actions
- **A participant, probably the ISIS representative:** update the issue with the agreed approach, including that ESS, Berkeley and probably ITER rely on the failure.

#### Follow-up after the codeathon
- [phoebus#3968](https://github.com/ControlSystemStudio/phoebus/pull/3968) (LordIdra, ISIS) was **merged on 26 Oct 2026** and implements the agreement.
  - It adds a new `-force_start` program argument, which must come before `-settings`.
  - With it, a settings file that fails to load is logged as SEVERE and Phoebus still starts.
  - Without it, behaviour is unchanged: launch fails.

#### Open questions
- Was the third site that relies on the failure ITER? The name was unclear in the recording.

### <a id="topic-13"></a>Topic 13: Gateways: PVA gateway, a new CA gateway, and CA/PVA bridging

Thu 22 Oct 2026, 16:08 to 16:22, plus an informal thread at 16:37 to 16:39. Kunal Shroff raised the topic and asked the EPICS core developers in the next room to join. The main speaker is Ralph Lange (ITER), developer of a new CA gateway.

#### Discussion summary

**PVA gateway status**
- There are two implementations, both with the binary name `pvagw`.
  - The old `pva2pva` is deprecated. It does not support access security.
  - The new one is in **P4P**. It "works pretty well" and supports access security.
- SLAC (Greg White) stress-tested a PVA gateway and presented exhaustive performance tests at conferences.
- The new configuration defines upstream (client-side) and downstream (server-side) interfaces separately, and which servers see which clients.
  - What used to need several gateways can now run in one, including the "reverse gateway" and the paired gateways that sites ran to see gateway status PVs on both sides.

**CA ↔ PVA bridge: dropped for complex types**
- Ralph had considered a bridge with Channel Access on one side and PV Access on the other, as a first step for sites migrating to PVA.
- He was talked out of it that week:
  - It works for scalars and simple arrays.
  - A PVA `descriptor` can't be filled reliably from CA. Kay advised not to fill it.
  - **NTNDArray images** would need a dozen CA subscriptions per PVA structure. The gateway can't keep the PVA promise that a structure is consistent; that only works on the IOC.
- An Oak Ridge library doing CA-on-one-side / PVA-on-the-other is looking for ownership. Suggestion: discourage it and push sites towards native PVA (NT types) instead.
- Counter-view: one lab hopes to use such a bridge for legacy systems it does not want to upgrade, while it moves to PVA-only. This works if scope is narrowly limited to scalars, simple arrays and NT structures, with images excluded by configuration, e.g. regular expressions.

**A new CA gateway (version 3?)**
- The existing CA gateway uses an old library that produces "three pages" in a vulnerability report.
- Ralph's rewrite:
  - The server side uses **RSRV**, the hardened IOC server.
  - It keeps DBR types in its cache, and its CA clients run upstream.
  - It is roughly **10× faster**, multi-threaded, and free of the existing gateway's bugs.
  - It copes with image streams that the old gateway could not handle.
- It currently lives in a branch on his clone of the CA Gateway. Open question: release it as **CA Gateway version 3**, or as a new project? Those present favoured version 3.
- There are "enthusiastic testers". It started as a weekend and evening project and now has requirements from ITER, so there is no timeline.

**Reconnection storms through chained gateways**
- One case: a C++ client with 150k PVs went through three chained CA gateways to about 50 IOCs. When the IOCs were off long enough and then switched back on, it took **three hours** to reconnect.
- Ralph will build a three-gateway test to reproduce it. All unresolved channels must first reach the longest search back-off period, so a test takes over an hour, with about 45 minutes of waiting.

#### Key findings
- Use the P4P-based PVA gateway; `pva2pva` is deprecated.
- A gateway can't bridge complex PVA structures (images, descriptors) from CA consistently.
- A RSRV-based CA gateway rewrite exists. It is about 10× faster and handles image streams.

#### Best practices highlighted
- Prefer native PVA, with NT types produced on the IOC, over bridging complex types from CA.
- If a CA→PVA bridge is used, limit it explicitly to scalars and simple arrays.

#### Conclusions / agreements
- Those present supported releasing the new gateway as CA Gateway version 3.
- The CA→PVA bridge is out of Ralph's requirements for now.

#### Commitments / actions
- **Ralph Lange:** build a three-chained-gateway reconnect test.
- **Unassigned:** find an owner for the Oak Ridge CA/PVA bridge library, or decide to discourage it.

#### Open questions
- What is the timeline for CA Gateway v3, and who will test it?
- Should a limited CA→PVA bridge be supported for legacy systems?

#### Linked passages
- Gateways and the CF name server: Topic 7.

### <a id="topic-14"></a>Topic 14: VTypes: separating "description" from "display" (epicsCoreJava DescriptionProvider)

Thu 22 Oct 2026, 14:51 to 14:52 and 16:22 to 16:33, plus a brief return at 16:49. Kunal Shroff led. Georg (ESS) and EPICS core developers joined, including the speaker from Topic 13. The proposal is Abraham Wolk's (ESS): [epicsCoreJava#146](https://github.com/epics-base/epicsCoreJava/pull/146) "Add interface `DescriptionProvider`" (open). It builds on [epicsCoreJava#140](https://github.com/epics-base/epicsCoreJava/issues/140) "How to incorporate descriptions in VTypes".

#### Discussion summary
- **The problem.**
  - In epicsCoreJava VTypes, the description (DESC) sits inside the `Display` interface together with limits, units and format. Display was designed with PV Access NT structures in mind.
  - Types like strings and booleans have a description but no numeric display information. They end up filling Display with NaNs and empty values: "it's clunky".
  - Kunal noted it is again a case where CA and PVA behave differently.
- **The proposal.** Add a separate `Description` interface through a `DescriptionProvider`.
  - Scalars (int, float, …) would implement both Description and Display.
  - Strings would implement only Description.
- **PVA's view.**
  - In NT types only `value` is mandatory. Everything else, including `display` and the optional `descriptor`, is optional, because the same spec is used for writing, where only the value is sent.
  - The optional elements are strongly encouraged when reading.
- **Root cause.** The VTypes copied the NT structures closely and so inherited "display" everywhere.
- **Unresolved puzzle.** The NT spec has both a top-level `descriptor` and a `description` inside `display`. No one in the room could say why there are two or how they differ.
- **Impact on clients.**
  - Every client must be checked: Phoebus, Display Builder Web Runtime, PVWS-based web tools, and other Qt/PVA display managers that might be interested.
  - The object will still expose the same methods, so most tools should keep working.
  - The archiver does not archive the description unless explicitly asked.
- Kunal later stressed caution: "most of our types have very religiously matched the NT structures", so be confident before breaking that.

#### Conclusions / agreements
- Proceed cautiously with Abraham's proposal.
- Kunal will merge it into a pre-release branch or pre-release of epicsCoreJava. A new official VType release follows only after everyone has tested. The matching Phoebus PR depends on that release.

#### Commitments / actions
- **Kunal Shroff:** merge #146 into a pre-release.
- **Kunal Shroff:** open a ticket asking how `descriptor` differs from `display.description` in the NT types, and why both exist.
- **Oak Ridge volunteer:** test Display Builder Web Runtime against the pre-release.
- **Unassigned:** test PVWS-based tools against the pre-release.

#### Open questions
- What is the difference between `descriptor` and `display.description` in the NT types?
- How do the C++ and Python PVA clients and the Qt display managers represent description? Kunal suggested looking at their implementations.

#### Linked passages
- DBWR and the web runtimes: Topic 2.

## <a id="part-2"></a>Part 2: Codeathon working-session notes

Notes from hands-on project time: project check-ins, pull-request triage, quick questions to the room, smaller agreements and logistics.

### <a id="topic-15"></a>Topic 15: Afternoon working session (Oct 20)

Tue 20 Oct 2026, 12:21 to 14:40. No formal agenda item; this is hands-on codeathon work with short coordination discussions.

#### Discussion summary

**Legacy CS-Studio Olog plugin and ITER (12:21 to 12:29).** NSLS-II (Ankit), where some users still run the old Eclipse-based CS-Studio, needs an Olog client plugin that talks to the new Phoebus Olog REST API. The strong recommendation was to move off the old CS-Studio rather than build this. If it is built anyway, the site should work with ITER, which still actively builds the Eclipse CS-Studio, so both can share the plugin and reuse ITER's build system. The build system (Maven wrapped around another build system) was the hardest part of the old CS-Studio and is the reason Phoebus was created. Mapping the new REST API into the plugin is a known, small amount of work. Ralph Lange was asked to introduce Ankit (NSLS-II) to the ITER person taking over CS-Studio. Ideally the plugin can be dropped into the existing product without a full rebuild; at worst a new product zip must be made.

Shift-management integration was also discussed. ITER (Nadine) integrated its logbook with the ITER shift system, and NSLS-II adds the shift ID to every accelerator log entry. In the old CS-Studio this was done in the client's log-entry dialog, so web-client entries did not get shift data. Phoebus Olog supports pluggable **log-entry processors** in the service, so the service can fetch the shift ID and add the metadata itself. This is cleaner, works for every client, and keeps shift details out of the user's entry dialog.

**PVWS Spring Boot follow-up (13:40 to 13:58, 14:23 to 14:27).** Work started on the PVWS container image and on bringing the latest ORNL changes into the Spring Boot version. The group discussed whether to bundle a Java runtime with the Spring Boot PVWS. The leaning was not to package the JRE in the application itself; the Docker image should build on a standard Java base image. Some sites (ESS, and one site for Windows and Mac clients) already build packages with a bundled JRE using `jpackage`. Another site had problems with `jpackage` on Linux about three years ago. One participant has a working Debian package setup and a README in the Phoebus repo; others were interested.

Adding authentication to PVWS was suggested as a step-by-step effort (a to-do, with no firm commitment), starting in the Spring Boot version: first a rudimentary JWT token plus simple user and password authentication, with a property to enable or disable it for compatibility. Kunal Shroff upgraded the PVWS Spring Boot version to Spring Boot 4 and Java 25, and Georg Weiss did JDK 25 work on the Docker side; and the project will be renamed to just "PVWS". After the upgrade it builds, runs, and passes its unit tests. JDK 21 support is dropped because the integration tests fail on JDK 21 with Spring Boot 4. This does not affect current deployments, but sites are asked to test it over the next few weeks. The group felt the change should be announced so people stop using the old version. It was not settled whether ORNL or the collaboration should make the announcement.

**JDK 25 documentation and developer setup (13:46 to 14:08).** JDK 25 is now the minimum, but the documentation, Dockerfiles, and compose files still mention JDK 21 in many places. A documentation PR needs reviewers to find what is left. Seth Nemesure's VS Code setup PR was changed to drop the `.vscode` folder: the launch file goes into `.gitignore` and the instructions go into the documentation. This addresses Kay's and Georg's concern and avoids requests to commit IntelliJ or Eclipse settings too. Seth merged the JDK 25 branch into his branch to avoid conflicts, and another participant verified the PR. A set of UI pull requests from ISIS needs reviewers, starting with the table font-size PR.

**Olog data migration tool (14:11 to 14:15).** A small, pluggable tool moves log entries from an existing logbook service into Phoebus Olog. It currently lives in a personal repository. It was agreed to move it into the Olog organization so facilities can maintain it together. Ankit will use it, and Seth and Kunal plan to test it for migrating from other logbooks such as e-log.

**LTTB in the archiver (14:33 to 14:39).** LTTB (largest-triangle-three-buckets) was first added as a Data Browser formula function; Kay and Sky suggested adding it to the archiver instead. It visually represents data well with far fewer points, for example a waveform with about 10% of its points. In the archiver it would be a retrieval post-processor only, not applied before storage, because it is not a high-fidelity representation. Archiver post-processors take the samples in a bin and can return any type, so the input and output types need not match. Two use cases were identified: LTTB across the scalar samples in a bin, and LTTB applied to each waveform in a bin. Agreed to implement the scalar case first.

#### Key findings
- Phoebus Olog log-entry processors let the service add shift data automatically, for all clients.
- ITER still builds the Eclipse CS-Studio and can share its build system.
- The Spring Boot PVWS now runs on Spring Boot 4 and Java 25, builds, and passes its unit tests; JDK 21 is no longer supported.
- JDK 21 references remain throughout the Phoebus documentation, Dockerfiles, and compose files.
- LTTB can cut a waveform to about 10% of its points while keeping its visual shape.

#### Best practices highlighted
- Add shift and other metadata in the Olog service through log-entry processors, not in a client dialog.
- Keep the logbook and shift service separate.
- Do not commit IDE-specific folders such as `.vscode`; put setup instructions in the documentation instead.
- Make new PVWS authentication optional through a property, for backward compatibility.
- Apply lossy reduction such as LTTB only at retrieval, never before storage.

#### Conclusions / agreements
- Recommendation: move off the old Eclipse CS-Studio. If an Olog plugin is built, build it together with ITER.
- Do not bundle the JRE in the Spring Boot PVWS application; base the container on a standard Java image.
- Suggested approach (no firm commitment): add PVWS authentication step by step in the Spring Boot version, JWT token plus simple user and password first, behind an enable property.
- Rename the Spring Boot PVWS project to "PVWS".
- The Spring Boot PVWS drops JDK 21 support; sites to test over the coming weeks.
- Move the Olog data migration tool into the Olog organization.
- LTTB in the archiver: retrieval-only post-processor, scalar case first, per-waveform later.

#### Issues / problems
- The old CS-Studio build system is very hard to work with.
- With the old CS-Studio approach, entries made through the web client did not get shift data.
- `jpackage` had problems on Linux at one site.
- JDK 25 migration was split across several branches (`jdk-25` and others), which caused confusion.
- ISIS UI pull requests are waiting for reviewers.

#### Project ideas
- Olog client plugin for the legacy Eclipse CS-Studio against the Phoebus Olog REST API, built with ITER.
- PVWS container image on a standard Java base image. Related project: DEVOPS-CONTAINER-001.
- Share the Debian packaging setup and `jpackage` experience across sites.
- Rudimentary JWT plus user and password authentication for the Spring Boot PVWS.
- Olog data migration tool, including an e-log importer.
- LTTB retrieval post-processor in the Archiver Appliance.

#### Commitments / actions
- **Ralph Lange (ITER):** introduce Ankit to the ITER person taking over CS-Studio.
- **Seth Nemesure:** update the VS Code PR (drop `.vscode`, add `.gitignore` entry and documentation), merging the JDK 25 branch. Done the same afternoon.
- **Unassigned:** review the JDK 25 documentation PR and find remaining JDK 21 references.
- **Unassigned:** review the ISIS UI pull requests, starting with the table font-size PR.
- **Kunal Shroff:** upgrade the Spring Boot PVWS to Spring Boot 4 and Java 25. Done; tests pass. **Georg Weiss:** JDK 25 updates for the Docker setup.
- **To do, no firm commitment:** add rudimentary JWT and user/password authentication to the Spring Boot PVWS after the upgrade.
- **Unassigned:** announce the Spring Boot PVWS as the community-maintained version (ORNL or the collaboration).
- **Sites:** test the Spring Boot PVWS on JDK 25 over the next few weeks.
- **Kunal Shroff:** move the Olog data migration tool into the Olog organization. Possibly already done; to confirm.
- **Seth Nemesure and Kunal Shroff:** test the migration tool for moving from other logbooks such as e-log. Ankit to verify it still works.
- **Unassigned:** implement LTTB as an archiver retrieval post-processor for scalars.

#### Follow-up after the codeathon
Status checked on 12 Nov 2026.
- PVWS: [pvws#11](https://github.com/ControlSystemStudio/pvws/pull/11) (Spring Boot 4 and Java 25, shroffk) was merged on 22 Oct, and [pvws#13](https://github.com/ControlSystemStudio/pvws/pull/13) (Docker for Java 25, georgweiss) on 2 Nov.
- LTTB: [epicsarchiverap#563](https://github.com/archiver-appliance/epicsarchiverap/pull/563) "Lttb postprocessor for scalar pvs" (caraxlr, open).

#### Open questions
- Should LTTB also be applied to each individual waveform in a bin?
- Who announces the Spring Boot PVWS as the maintained version?

#### Linked passages
- PVWS decisions and security: Topic 2.
- LTTB and waveform reduction: Topic 1.1 (waveform best practices).
- Legacy Olog sunset: Topic 6.

### <a id="topic-16"></a>Topic 16: Afternoon working session (Oct 21)

Wed 21 Oct 2026, 13:05 to 16:28. This was project time with short group discussions. Two design discussions from this session are written up separately as Topic 9 (Tabs widget) and Topic 10 (alarm-sensitive widgets). Kunal Shroff chaired.

#### 16.1 Project check-in and pull-request triage (13:09 to 13:21)

**Discussion summary**

Kunal confirmed that no more discussion was planned for the day and that the afternoon was for projects. He asked who still needed one.

- **Alarm services and Kafka.** Georg (ESS) is updating the alarm services to the Kafka Streams 4 client API. Some APIs used by the alarm services were removed in version 4, so testing of the alarm services should wait until he finishes.
- **PVWS.**
  - Two PRs are open: Kunal's upgrade to JDK 25 and Spring Boot 4, and a PR catching up with changes made earlier by Kay Kasemir. They may conflict.
  - Agreed order: merge Kay's changes first, then Kunal's.
  - The unit tests already start several IOCs and check connection and disconnection. A real deployment test against a soft IOC, over both Channel Access and PV Access, is still wanted. Loic (CEA) took this on.
- **Open PRs.**
  - Kunal wants to cut the number of open PRs significantly over the next two days.
  - He will contact Percy (ISIS) and Diamond the next morning about their PRs. Berkeley has one PR open and CEA has six or seven.
  - People were asked to form small groups to find what is blocking PRs and close them.
- **Loic (CEA)'s plan for the afternoon:**
  - a list of where colors are defined in the code (generated with AI, now being checked);
  - translation fixes, at least for the menus;
  - PVWS testing.
- **PVWS security (JWT)** was pushed to later.
- **Docker files.**
  - The Docker files for the services were reviewed: the save-and-restore one is up to date and some others are obsolete.
  - The suggestion was to clean them up and provide a compose file that deploys the full stack, plus one with just the three alarm services.
- **Alarm annunciator.** A race condition was reported. This is ISIS issue [#3966](https://github.com/ControlSystemStudio/phoebus/issues/3966), "MediaPlayer race condition in audio annunciator". ISIS has since submitted a fix.

**Commitments / actions**
- **Georg (ESS):** finish the Kafka Streams 4 client update for the alarm services.
- **Kunal Shroff:** merge Kay Kasemir's PVWS changes, then the JDK 25 / Spring Boot 4 PR.
- **Loic (CEA):** deploy PVWS and test it against a real soft IOC over CA and PVA.
- **Kunal Shroff:** contact Percy (ISIS) and Diamond about their open PRs.
- **All participants:** review and close open PRs in small groups during the codeathon.
- **Unassigned:** clean up the service Docker files and provide compose files for the full stack and for the alarm services alone.

#### 16.2 Alarm server behavior when Kafka disconnects (13:57 to 14:00, 14:21 to 14:28, 16:28)

**Discussion summary**
- Tests were planned for how the alarm server and alarm logger behave when Kafka goes away. Kafka 3 to 4 brought significant changes. Georg was investigating why some messages were not logged by the alarm logger, which looked like a Kafka issue. It had been assumed fixed by the move to Kafka 3, but never confirmed.
- **Agreed tests:**
  1. The Kafka server goes away and comes back. Does the alarm server reconnect cleanly?
  2. A topic is deleted while the alarm server is running.
- **Options when a topic disappears:**
  - recreate the topics automatically with a Kafka utility; or
  - raise an error and notify someone.
- **How sites manage configuration:**
  - Some sites (Berkeley, FRIB) re-import the alarm configuration regularly. They delete the topics first, inside a Docker setup that deletes all topics on start-up, then import from the master copy.
  - Operators may make changes during shifts. Those changes are wiped on the next import, and operators accept this.
  - If the alarm server is shut down before the topics are deleted, there is no problem.
- **Agreement:** at minimum, the alarm server should log this situation properly and flag it as an error.
- **Health notifications for the alarm server.**
  - The other services use Spring Boot Actuator for health and metrics. The alarm server is not a Spring Boot application, so adding this would mean converting it.
  - Alternatively, exceptions such as losing connection to its topics could trigger a notification, for example an email using the existing mechanism. It would be optional, so sites without email can leave it unconfigured.
- At the end of the day Kunal asked who else was interested in the connect/disconnect behavior.

**Conclusions / agreements**
- The alarm server must log a lost connection or a deleted topic clearly, as an error.
- Health notifications for the alarm server are wanted.

**Commitments / actions**
- **Georg (ESS), with others:** test Kafka server loss and recovery, and topic deletion, against the alarm server and alarm logger. Investigate the alarm-logger messages that go missing.
- **Unassigned:** health and connection-loss notifications for the alarm server, either through Spring Boot Actuator after a conversion or through optional email notifications.

#### 16.3 PR review: Oracle timestamps in the RDB archive (14:05 to 14:06)

- Kunal asked Oak Ridge to review phoebus PR [#3905](https://github.com/ControlSystemStudio/phoebus/pull/3905), "Archive: Support for UTC-related Oracle issues", by Kay Kasemir.
- The PR works around Oracle using the local time zone instead of UTC by making UTC the default for ingestion. The code looked self-evident, but nobody present had a setup to test it.
- **Action, Oak Ridge:** give a second review so that #3905 can be merged.

#### 16.4 Oak Ridge polls: ScanEngine and Archive RDB (15:59 to 16:26)

- **ScanEngine.** Oak Ridge asked whether any other facility uses it. One facility is considering it.
  - Kunal: it is not being abandoned, but no other facility runs a deployment, so others cannot test it.
  - Oak Ridge is needed to keep checking that it works. It depends on many core modules, and upgrades to those (such as the move to PVA 4) must not break it.
- **Archive RDB (relational database archiver).** Apparently only Oak Ridge uses it.
  - Oak Ridge is no longer considering the Cassandra-based archiver and is now looking at the Archiver Appliance.
  - Murali Shankar said a relational data source does not fit the appliance, because the storage mechanism is conceptually different.
  - Oak Ridge should consider moving away from the RDB engine, although migrating its large data volume will take a long time.

**Commitments / actions**
- **Oak Ridge:** act as the contact for the ScanEngine. Keep checking that it works as core modules change, and report problems.

#### Issues / problems (session-wide)
- The removal of Kafka Streams 4 APIs blocks testing of the alarm services until Georg's update is done.
- There are many open PRs, including from ISIS, Diamond, Berkeley and CEA.
- A race condition in the alarm annunciator (#3966, reported by ISIS). A fix has been submitted.
- Some messages are missing from the alarm logger, possibly because of Kafka.
- The alarm server has no health checks or notifications.
- The ScanEngine and the RDB archive have only one user site, which makes them hard to test.

#### Project ideas
- Clean up the Docker files and provide compose files for the full stack and for the alarm services. Related: DEPLOY projects.
- Health checks and notifications for the alarm server.

#### Follow-up after the codeathon
Status checked on 12 Nov 2026.
- Kafka client: [phoebus#3985](https://github.com/ControlSystemStudio/phoebus/pull/3985) "Switching to 4.3.1 for the Kafka client libs" (georgweiss) was merged on 26 Oct.
- Translations: [phoebus#3982](https://github.com/ControlSystemStudio/phoebus/pull/3982) "Phoebus is not fully translated" (lcaouen) was merged on 24 Oct.
- PVWS: Kay's changes ([pvws#12](https://github.com/ControlSystemStudio/pvws/pull/12), merged 5 Nov) and the Spring Boot 4 / JDK 25 upgrade ([pvws#11](https://github.com/ControlSystemStudio/pvws/pull/11), merged 22 Oct) are both in.
- Oracle timestamps: [phoebus#3905](https://github.com/ControlSystemStudio/phoebus/pull/3905) is still open.

#### Open questions
- Which facility is considering the ScanEngine?

#### Linked passages
- Kafka and alarm configuration management: Topic 3.
- PVWS Spring Boot and security: Topics 2 and 15.
- Archiver Appliance roadmap: Topic 1.
- Tabs widget: Topic 9. Alarm-sensitive widgets and colors: Topic 10.

### <a id="topic-17"></a>Topic 17: Morning working session (Oct 22)

Thu 22 Oct 2026, 10:23 to 12:04. This was hands-on working time, with short discussions in between. Several conversations ran in parallel, so only the identifiable threads are recorded here.

Two design discussions from this session are separate topics:
- PR #3928, truncated text with an ellipsis: Topic 11.
- Phoebus launch when the `-settings` file is missing: Topic 12.

#### 17.1 Alarm server: "ghost" configuration after deleting topics

**Discussion**
- After alarm topics are deleted and a configuration is re-imported, entries that are no longer in the configuration come back. An issue was opened during the session.
- Loic Caouen (CEA) removed an entry completely from the XML and deployed it. It looked fine at first, but the node reappeared "somewhat randomly" about a day later.
- Kunal saw the same at EIC.
  - The alarm server's import code explicitly deletes the root and all messages in the topic before importing.
  - Despite that, he could never get a reliable full replacement, so EIC's Ansible deployment now deletes everything explicitly first.
  - Another participant does the same.
- **Suspected cause:** something is persisted or cached on the Kafka side.
- **Kunal's view:** import-based reconfiguration "is just not the right way to solve this problem". It is the only mechanism available, but it is not reliable.

**Follow-up**
- [phoebus#3978](https://github.com/ControlSystemStudio/phoebus/issues/3978) "Ghost alarms when deleting topics" (Loic Caouen, open).

**Open question**
- How should complete reconfiguration of an alarm tree be supported in future? Kunal asked the sites that recently improved the alarm server whether they had seen this.

#### 17.2 PVWS: CORS headers for browser clients

**Discussion**
- Tynan Ford needed CORS headers on PVWS for PVInfo, a separate browser web application that connects to the PVWS WebSocket and HTTP endpoints. He added his own `web.xml` in his Docker setup.
- **Suggestion:** this is useful to others, so it belongs upstream.
  - ChannelFinder already has an `application.properties` setting for CORS.
  - PVWS should get a similar property once it moves to Spring Boot (see Topic 2).
- Tynan runs PVWS from his own Docker setup, not as a WAR on Tomcat. Kunal asked him to contribute it.

**Commitment**
- **Tynan Ford:** pull requests for the Docker setup and for CORS.

**Follow-up**
- [pvws#14](https://github.com/ControlSystemStudio/pvws/pull/14) "Allow cors requests" (open). The PR was opened by georgweiss on 2 Nov.

#### 17.3 Alarm client "disconnected" indicator

**Discussion**
- The alarm client shows "disconnected" whenever it receives no message from Kafka for about 10 seconds. The alarm server sends a periodic heartbeat for exactly this purpose.
- This confuses users. The client may be connected to Kafka while the alarm server is not running, or the server may be fine but no message has arrived. The indicator cannot distinguish "Kafka unreachable" from "no alarm server".

**Project idea**
- Distinguish "connected to Kafka, but no alarm server heartbeat" from "no connection to Kafka".

#### 17.4 Pre-commit vs prek

**Discussion**
- One task of this codeathon is to document how contributors run pre-commit checks locally. Many recent pull requests failed CI on formatting.
- A participant (described by Kunal as "his argument") proposed switching to prek, a Rust reimplementation that uses the same pre-commit configuration but is faster and more parallel.
- Someone asked why CI does not just fix the formatting itself. Kunal: contributors should take responsibility for changes to their own files, and "our CI pipeline just flag[s] issues, but [doesn't] touch anything".
- Both tools read the same configuration, so contributors can use either. ESS uses the current pre-commit and it works fine.

**Best practice**
- CI should report formatting problems, not rewrite committed files.

**Follow-up**
- [phoebus#3980](https://github.com/ControlSystemStudio/phoebus/issues/3980) "Recommend https://prek.j178.dev/ over pre-commit" (shroffk, open).
- Related, already merged: [phoebus#3971](https://github.com/ControlSystemStudio/phoebus/pull/3971) "Add Pre-commit Checks to PR Template and Docs".

#### 17.5 Other items

- **Coding guidelines.** While reviewing a newcomer's widget PR, Kunal suggested adding coding guidelines to the repository, so that anything unclear to new contributors gets written down.
- **Scroll-bar flicker fix.** Kunal asked for a volunteer to check that the scroll-bar flicker fix, re-applied now that JavaFX 25 is in use, works. It was merged the same day as [phoebus#3972](https://github.com/ControlSystemStudio/phoebus/pull/3972) (rjwills28).
- **RecCeiver JDK 25.** Kunal asked who was upgrading the RecCeiver to JDK 25. Someone volunteered.
- **Oracle timestamp issue.** The Oracle timestamp issue from Topic 16 was assigned to a participant.
- **Plan for the rest of the day.** Pull requests and code until lunch, then save-and-restore and the JDK upgrade after lunch.

#### Commitments / actions
- **Tynan Ford:** Docker and CORS pull requests for PVWS. CORS is now in [pvws#14](https://github.com/ControlSystemStudio/pvws/pull/14), opened by georgweiss.
- **Volunteer, name not captured:** RecCeiver JDK 25 upgrade.
- **Assignee, name not captured:** Oracle timestamp issue.
- **Loic Caouen (issue reporter), investigator unassigned:** investigate the ghost alarm configuration; see #3978.

#### Open questions
- Should Phoebus recommend prek, or stay with pre-commit? (#3980)

#### Linked passages
- PVWS Spring Boot migration: Topic 2.
- Oracle timestamp issue: Topic 16.
- Alarm services: Topic 3.

### <a id="topic-18"></a>Topic 18: Afternoon working session and wrap-up (Oct 22)

Thu 22 Oct 2026, 14:33 to 14:52 and 15:30 to 16:54. This was working time with short discussions in between. The gateway and VType-description discussions from this period are Topics 13 and 14.

#### 18.1 Ansible roles and the training VM

**Discussion**
- Kunal showed a reworked Ansible role for the training VM. A choice of variables decides whether to install Phoebus itself or only the services.
- The EIC role is now very thin. It depends on the public role and only copies EIC-specific resources such as preferences. Kunal: "it's become really simple".
- Not yet tested on all platforms. The training VM currently runs on x86-64.
- Plan: go through the open issues one by one and fix some in the role itself. Produce release bundles from it, the same way binaries are produced.

**Archiver Appliance version in the role**
- Murali Shankar had sent changes, as about 20 separate branches, updating the Archiver Appliance in the role from 2.0.7 to 2.0.10.
- Murali Shankar recommended going straight to **2.2.1**, which he runs in production. He releases to production about every couple of months; long-unreleased versions had accumulated "so many fixes".
- Kunal: Phoebus should learn from this and release more often.

**Ansible deployment instructions PR**
- The reviewers decided to keep sentences a reviewer had flagged as unnecessary, because forgetting that step has caused unintended consequences. The PR was approved and merged.

#### 18.2 Default ports for the services

**Discussion**
- All the middle-layer services default to 8080.
  - That assumed each runs on its own machine.
  - For training, or a single VM running everything, every property file has to be edited.
- **Proposal:** sequential defaults, 8080, 8081, 8082, … per service.
- The proposer's motivation is many installs and a recent training in France, where building the VM was the hard part.
- Kunal noted the services live in several repositories (Phoebus, Olog, ChannelFinder, RecCeiver…), so no single place can guarantee a unique set. The Archiver Appliance now uses a single Tomcat on 8080.
- Kunal: no one in the room objects, so go ahead. "No guarantees that the rest of the facilities don't get upset."

**Agreement**
- Change the default ports to a sequential scheme through pull requests. Other facilities can comment on the PRs.

**Follow-up**
- Sequential default ports, all opened by lcaouen and still open on 12 Nov 2026: [ChannelFinderService#238](https://github.com/ChannelFinder/ChannelFinderService/pull/238), [phoebus#4006](https://github.com/ControlSystemStudio/phoebus/pull/4006), [phoebus-olog#277](https://github.com/Olog/phoebus-olog/pull/277) and [pvws#15](https://github.com/ControlSystemStudio/pvws/pull/15).

#### 18.3 Audio annunciator race condition (#3966)

**Discussion**
- The audio annunciator player is asynchronous by design, which does not suit ISIS's use: a fixed period between messages.
- The only fix seen is a blocking `CompletableFuture`. This would not affect normal use.
- The site using the annunciator most (described as "the happiest users") does not see the problem. It plays a message about every 30 seconds rather than single beeps.

**Conclusion**
- A blocking fix is acceptable. See [phoebus#3966](https://github.com/ControlSystemStudio/phoebus/issues/3966) and Topic 16.

#### 18.4 Documentation for a new application

**Discussion**
- A contributor asked where to document their application.
- Each application has a doc folder at its top level, with an entry point currently in reStructuredText and images in an `images` subfolder.
- Phoebus is slowly moving to Markdown, and both formats are supported for now. The advice was to create a doc folder.

#### 18.5 Wrap-up and Friday plan

- **Collecting results.** Andrew will put together one wiki page for the developers' meeting and codeathon results.
- **Review of decisions.** Kunal asked everyone to review the design decisions discussed during the week and add them as bullet points on that page: "I have a feeling I've forgotten half of that". An hour to an hour and a half was set aside on Friday morning for this.
- **Friday.** No more scheduled discussions. The morning is for finishing pull requests, testing and collecting results. Gaps in the discussion list can be raised Friday morning. Rooms are available until about 3 to 4 PM.
- **Monthly meeting.** It was mentioned that this is usually held on the second Thursday of each month.

#### Commitments / actions
- **Murali Shankar:** update the Archiver Appliance in the Ansible role, aiming for 2.2.1 instead of 2.0.10. Kunal is to follow up on the separate branches.
- **Kunal Shroff:** continue the shared Ansible role and training VM, and work through its issues.
- **Proposer (Loic Caouen, CEA):** pull requests for sequential default service ports.
- **Andrew:** wiki page for developers' meeting and codeathon results.
- **All participants:** review the week's decisions and add bullet points to the wiki page.

#### Open questions
- What is the agreed port for each service? The proposer was to decide the order.

#### Linked passages
- Annunciator: Topic 16.
- Archiver Appliance: Topic 1.
- Training VM and RecSync pipeline: Topic 7.
