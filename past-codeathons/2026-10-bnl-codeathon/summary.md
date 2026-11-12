# Summary of decisions and actions: 2026 October Codeathon (BNL)

A short companion to the full [minutes](minutes.md), for review at the follow-up meeting. Numbers in the **Topic** column link to the topic in the minutes. GitHub status was checked on 12 Nov 2026.

For the meeting, skim section 0, then work through sections 1 to 4. Section 5 lists quick factual questions to settle by email.

## 0. Already done since the codeathon

| Item | PR | Topic |
|---|---|---|
| Opt-in `-force_start` when the `-settings` file is missing | [#3968](https://github.com/ControlSystemStudio/phoebus/pull/3968) (LordIdra), merged 26 Oct | [12](minutes.md#topic-12-phoebus-launch-when-the--settings-file-is-missing) |
| Save-and-restore restore policy, sync or async (JCA deadlock workaround) | [#3997](https://github.com/ControlSystemStudio/phoebus/pull/3997) (georgweiss), merged 2 Nov | [8](minutes.md#topic-8-save-and-restore-facility-workflows-ui-client-vs-service-restore-restore-ordering-and-deadlocks) |
| Case-insensitive filter in the snapshot view | [#3989](https://github.com/ControlSystemStudio/phoebus/pull/3989) (abrahamwolk), merged | [8](minutes.md#topic-8-save-and-restore-facility-workflows-ui-client-vs-service-restore-restore-ordering-and-deadlocks) |
| Kafka client libraries updated to 4.3.1 | [#3985](https://github.com/ControlSystemStudio/phoebus/pull/3985) (georgweiss), merged 26 Oct | [3](minutes.md#topic-3-alarm-services), [16](minutes.md#topic-16-afternoon-working-session-oct-21) |
| Docker setup: Kafka updated, ZooKeeper removed | [#3977](https://github.com/ControlSystemStudio/phoebus/pull/3977) (roycpro), merged 22 Oct | [3](minutes.md#topic-3-alarm-services) |
| Translation fixes | [#3982](https://github.com/ControlSystemStudio/phoebus/pull/3982) (lcaouen), merged 24 Oct | [16](minutes.md#topic-16-afternoon-working-session-oct-21) |
| Scroll-bar flicker fix re-applied for JavaFX 25 | [#3972](https://github.com/ControlSystemStudio/phoebus/pull/3972) (rjwills28), merged 22 Oct | [17](minutes.md#topic-17-morning-working-session-oct-22) |
| PVWS on Spring Boot 4 and Java 25 | [pvws#11](https://github.com/ControlSystemStudio/pvws/pull/11) (shroffk), merged 22 Oct | [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security), [15](minutes.md#topic-15-afternoon-working-session-oct-20) |
| PVWS Spring Boot version aligned with the ORNL ("legacy") code base | [pvws#12](https://github.com/ControlSystemStudio/pvws/pull/12) (georgweiss), merged 5 Nov | [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security), [16](minutes.md#topic-16-afternoon-working-session-oct-21) |
| PVWS Docker setup updated for Java 25 | [pvws#13](https://github.com/ControlSystemStudio/pvws/pull/13) (georgweiss), merged 2 Nov | [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security), [15](minutes.md#topic-15-afternoon-working-session-oct-20) |
| RecCeiver start-up and shutdown cleanup | [recceiver#7](https://github.com/ChannelFinder/recceiver/pull/7) (roycpro), merged 2 Nov | [7](minutes.md#topic-7-recsync-recceiver-reccaster-and-the-channelfinder-ecosystem) |

## 1. Key decisions (confirm)

| # | Decision | Topic |
|---|---|---|
| D1 | Archiver Parquet: keep STS and MTS in protobuf; use Parquet only at the last ETL step into the final tier. | [1](minutes.md#topic-1-archiver-appliance) |
| D2 | Extend JSON batch support to more archiver BPLs, including `archivePV`. | [1](minutes.md#topic-1-archiver-appliance) |
| D3 | LTTB in the archiver as a retrieval-only post-processor, scalar case first. PR open: [epicsarchiverap#563](https://github.com/archiver-appliance/epicsarchiverap/pull/563). | [15](minutes.md#topic-15-afternoon-working-session-oct-20) |
| D4 | PVWS: the Spring Boot version is the maintained one. It is renamed "PVWS", moves to Spring Boot 4 and JDK 25 (JDK 21 dropped), and its container uses a standard Java base image. **Mostly done** (see section 0). | [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security), [15](minutes.md#topic-15-afternoon-working-session-oct-20) |
| D5 | Alarm tree: add edit and run modes, based on Percy's prototype, before any tree-table rework. Merge only after FRIB and Oak Ridge agree. **In progress:** [#3991](https://github.com/ControlSystemStudio/phoebus/pull/3991), [#4007](https://github.com/ControlSystemStudio/phoebus/pull/4007) and related PRs by LordIdra are open. The original proposal ([#3920](https://github.com/ControlSystemStudio/phoebus/issues/3920)) was closed. | [3](minutes.md#topic-3-alarm-services) |
| D6 | Mode-dependent alarms are three problems: static configuration swaps (exist today), Save & Restore for slow changes, and IOC logic for fast changes. | [3](minutes.md#topic-3-alarm-services) |
| D7 | The alarm server must log a lost Kafka connection or a deleted topic as an error. Health notifications are wanted. | [16](minutes.md#topic-16-afternoon-working-session-oct-21) |
| D8 | Olog: deploy the AI search together and document it. Move the Python client and the migration tool into the Olog organization. The legacy Olog needs a sunset plan. | [6](minutes.md#topic-6-olog-ai-search-python-client-and-the-legacy-olog), [15](minutes.md#topic-15-afternoon-working-session-oct-20) |
| D9 | Design the middle-layer service interfaces with agentic (MCP) use in mind, and invite the Osprey developer to give a talk. | [5](minutes.md#topic-5-agentic-frameworks-osprey-and-the-middle-layer-services) |
| D10 | RecSync: add an *undefined* channel state; add unicast address lists to RecCaster with DNS re-resolution; versions are major = protocol, minor = implementation; a CA name server is needed. | [7](minutes.md#topic-7-recsync-recceiver-reccaster-and-the-channelfinder-ecosystem) |
| D11 | Save-and-restore: add a server switch to disable service restore. The restore mode becomes an administrator preference (default: client). Service restore requires authentication. Ordered restore uses a sequencer or staged configurations. Add column hiding. | [8](minutes.md#topic-8-save-and-restore-facility-workflows-ui-client-vs-service-restore-restore-ordering-and-deadlocks) |
| D12 | Tabs widget: add a new *Default Tab* property; *Active Tab* becomes runtime state only. This deliberately breaks compatibility. | [9](minutes.md#topic-9-display-builder-tabs-widget-active-tab-vs-default-tab) |
| D13 | Colors: define one central, named color set first, then add opt-in alarm sensitivity to individual widgets. | [10](minutes.md#topic-10-alarm-sensitive-widgets-and-alarm-colors) |
| D14 | Truncated text ([#3928](https://github.com/ControlSystemStudio/phoebus/pull/3928)): the feature is wanted but needs fixes. An ellipsis must never be written to a PV. | [11](minutes.md#topic-11-showing-truncated-values-in-text-entry-and-text-update-pr-3928) |
| D15 | Missing `-settings` file: opt-in `-force_start`, with the default behaviour unchanged. **Done** ([#3968](https://github.com/ControlSystemStudio/phoebus/pull/3968)). | [12](minutes.md#topic-12-phoebus-launch-when-the--settings-file-is-missing) |
| D16 | Gateways: release the new CA gateway as CA Gateway version 3. Discourage bridging complex types from CA to PVA. | [13](minutes.md#topic-13-gateways-pva-gateway-a-new-ca-gateway-and-capva-bridging) |
| D17 | VType `DescriptionProvider`: release it in a pre-release of epicsCoreJava first; the official release follows testing. | [14](minutes.md#topic-14-vtypes-separating-description-from-display-epicscorejava-descriptionprovider) |
| D18 | Use sequential default ports (8080, 8081, …) for the services. PRs are open for ChannelFinder, Phoebus, Olog and PVWS (see section 3). | [18](minutes.md#topic-18-afternoon-working-session-and-wrap-up-oct-22) |
| D19 | CI flags formatting problems but never rewrites code. | [17](minutes.md#topic-17-morning-working-session-oct-22) |
| D20 | A blocking fix for the annunciator race condition ([#3966](https://github.com/ControlSystemStudio/phoebus/issues/3966)) is acceptable. | [18](minutes.md#topic-18-afternoon-working-session-and-wrap-up-oct-22) |

## 2. Actions without an owner (assign or drop)

| # | Action | Topic |
|---|---|---|
| U1 | Open a ticket for the consolidation problem when hold is greater than 11 (suggested by Anthony, FRIB). | [1](minutes.md#topic-1-archiver-appliance) |
| U2 | Batch `archivePV` and other BPLs (a good starter project). | [1](minutes.md#topic-1-archiver-appliance) |
| U3 | Announce the Spring Boot PVWS as the maintained version (ORNL or the collaboration). | [15](minutes.md#topic-15-afternoon-working-session-oct-20) |
| U4 | PVWS authentication (JWT plus user and password, behind an enable property). | [15](minutes.md#topic-15-afternoon-working-session-oct-20) |
| U5 | Verify Kafka reconnection in the alarm services and clients. The library upgrade is done ([#3985](https://github.com/ControlSystemStudio/phoebus/pull/3985)); the reconnection tests are not. | [3](minutes.md#topic-3-alarm-services), [16](minutes.md#topic-16-afternoon-working-session-oct-21) |
| U6 | Document how to handle alarm configuration changes at different rates. | [3](minutes.md#topic-3-alarm-services) |
| U7 | Health and connection-loss notifications for the alarm server. | [16](minutes.md#topic-16-afternoon-working-session-oct-21) |
| U8 | Investigate ghost alarms ([#3978](https://github.com/ControlSystemStudio/phoebus/issues/3978), reported by Loic). | [17](minutes.md#topic-17-morning-working-session-oct-22) |
| U9 | Olog MCP server or Osprey logbook adapter. | [5](minutes.md#topic-5-agentic-frameworks-osprey-and-the-middle-layer-services) |
| U10 | Document the ChannelFinder ecosystem: plugins, info tags and allow-list (the volunteer's name was not captured). | [7](minutes.md#topic-7-recsync-recceiver-reccaster-and-the-channelfinder-ecosystem) |
| U11 | Document that QSRV2 sets both environment variables, and which PVXS release fixed it. | [7](minutes.md#topic-7-recsync-recceiver-reccaster-and-the-channelfinder-ecosystem) |
| U12 | Set an initial RecSync version and align the other components with it. | [7](minutes.md#topic-7-recsync-recceiver-reccaster-and-the-channelfinder-ecosystem) |
| U13 | Upgrade RecCeiver to JDK 25 (the volunteer's name was not captured). Possibly covered by [recceiver#8](https://github.com/ChannelFinder/recceiver/pull/8) "Spring4" (shroffk, open). | [17](minutes.md#topic-17-morning-working-session-oct-22) |
| U14 | Column hiding in the save-and-restore, ChannelFinder and alarm-log tables. | [8](minutes.md#topic-8-save-and-restore-facility-workflows-ui-client-vs-service-restore-restore-ordering-and-deadlocks) |
| U15 | Open an issue or PR for the *Default Tab* property. | [9](minutes.md#topic-9-display-builder-tabs-widget-active-tab-vs-default-tab) |
| U16 | Once colors are consolidated, open an issue for alarm-sensitive widgets. | [10](minutes.md#topic-10-alarm-sensitive-widgets-and-alarm-colors) |
| U17 | Find an owner for the Oak Ridge CA/PVA bridge library, or discourage its use. | [13](minutes.md#topic-13-gateways-pva-gateway-a-new-ca-gateway-and-capva-bridging) |
| U18 | Test PVWS-based tools against the VType pre-release. | [14](minutes.md#topic-14-vtypes-separating-description-from-display-epicscorejava-descriptionprovider) |
| U19 | Review the JDK 25 documentation PR and remove the remaining JDK 21 references. | [15](minutes.md#topic-15-afternoon-working-session-oct-20) |
| U20 | Review the ISIS UI pull requests. | [15](minutes.md#topic-15-afternoon-working-session-oct-20) |
| U21 | Clean up the service Docker files and add compose files for the full stack and for the alarm services. | [16](minutes.md#topic-16-afternoon-working-session-oct-21) |
| U22 | Share the April DBWR vs cs-web-lib comparison. | [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security) |

## 3. Actions with owners (quick status check)

| Owner | Action | Topic | Status (12 Nov) |
|---|---|---|---|
| Anthony Carriveau (FRIB) | PR adding a Parquet migration guide to the archiver docs | [1](minutes.md#topic-1-archiver-appliance) | |
| FRIB | Test Parquet in the ETL stage | [1](minutes.md#topic-1-archiver-appliance) | |
| Murali Shankar | Store the timestamp user tag in the archiver | [1](minutes.md#topic-1-archiver-appliance) | [epicsarchiverap#559](https://github.com/archiver-appliance/epicsarchiverap/pull/559) open |
| caraxlr | LTTB retrieval post-processor for scalars | [15](minutes.md#topic-15-afternoon-working-session-oct-20) | [epicsarchiverap#563](https://github.com/archiver-appliance/epicsarchiverap/pull/563) open |
| Murali Shankar | Update the Archiver Appliance in the Ansible role to 2.2.1 | [18](minutes.md#topic-18-afternoon-working-session-and-wrap-up-oct-22) | |
| Murali Shankar | Help implement the CA name server | [7](minutes.md#topic-7-recsync-recceiver-reccaster-and-the-channelfinder-ecosystem) | |
| ORNL, DLS, Berkeley, CEA (Rémi) | Test the Spring Boot PVWS and report | [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security) | |
| Loic Caouen (CEA) | Test PVWS against a soft IOC over CA and PVA | [16](minutes.md#topic-16-afternoon-working-session-oct-21) | |
| Loic Caouen (CEA) | Get permission to publish the alarm-web code | [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security) | |
| Loic Caouen (CEA) | List where colors are hard-coded or configurable | [10](minutes.md#topic-10-alarm-sensitive-widgets-and-alarm-colors) | |
| Loic Caouen (CEA) | Sequential default ports | [18](minutes.md#topic-18-afternoon-working-session-and-wrap-up-oct-22) | open: [CF#238](https://github.com/ChannelFinder/ChannelFinderService/pull/238), [#4006](https://github.com/ControlSystemStudio/phoebus/pull/4006), [olog#277](https://github.com/Olog/phoebus-olog/pull/277), [pvws#15](https://github.com/ControlSystemStudio/pvws/pull/15) |
| Martin Gaughran (DLS) | Share the PVWS stress tests and the display library link | [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security) | |
| Ralph Lange (ITER) | Draft PVWS security requirements | [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security) | |
| Ralph Lange (ITER) | Check whether the alarm handler can support ITER's filtering | [3](minutes.md#topic-3-alarm-services) | |
| Ralph Lange (ITER) | Three-chained-gateway reconnect test | [13](minutes.md#topic-13-gateways-pva-gateway-a-new-ca-gateway-and-capva-bridging) | |
| Ralph Lange (ITER) | Introduce Ankit to ITER's new CS-Studio owner | [15](minutes.md#topic-15-afternoon-working-session-oct-20) | |
| Percy (ISIS), Georg Weiss (ESS) | Alarm tree edit and run modes | [3](minutes.md#topic-3-alarm-services) | [#3991](https://github.com/ControlSystemStudio/phoebus/pull/3991), [#4007](https://github.com/ControlSystemStudio/phoebus/pull/4007) and related PRs open |
| Georg Weiss (ESS) | Kafka 4 client update for the alarm services | [16](minutes.md#topic-16-afternoon-working-session-oct-21) | [#3985](https://github.com/ControlSystemStudio/phoebus/pull/3985) merged |
| Georg Weiss (ESS) | Kafka loss and topic-deletion tests; missing alarm-logger messages | [16](minutes.md#topic-16-afternoon-working-session-oct-21) | |
| Georg Weiss (ESS) | Ask ESS stakeholders about the restore-mode default and the quick filter | [8](minutes.md#topic-8-save-and-restore-facility-workflows-ui-client-vs-service-restore-restore-ordering-and-deadlocks) | |
| Jure Varlec (Cosylab) | Restore-deadlock reproducer and issue | [8](minutes.md#topic-8-save-and-restore-facility-workflows-ui-client-vs-service-restore-restore-ordering-and-deadlocks) | related: [#3983](https://github.com/ControlSystemStudio/phoebus/issues/3983) open |
| CEA | Share its alarm-disabling scripts | [3](minutes.md#topic-3-alarm-services) | |
| Kunal Shroff | GitHub discussion on alarm security expectations | [3](minutes.md#topic-3-alarm-services) | |
| Kunal Shroff, NSLS-II student, ESS | Deploy and document the Olog AI search | [6](minutes.md#topic-6-olog-ai-search-python-client-and-the-legacy-olog) | |
| Ankit (NSLS-II), ESS | Olog Python client | [6](minutes.md#topic-6-olog-ai-search-python-client-and-the-legacy-olog) | |
| NSLS-II, FRIB | Timeline for retiring the legacy Olog | [6](minutes.md#topic-6-olog-ai-search-python-client-and-the-legacy-olog) | |
| Kunal Shroff; Seth Nemesure | Move the Olog migration tool and test it with e-log | [15](minutes.md#topic-15-afternoon-working-session-oct-20) | |
| Lucy (BNL) | Invite the Osprey developer; share slides | [5](minutes.md#topic-5-agentic-frameworks-osprey-and-the-middle-layer-services) | |
| Kunal Shroff | Java RecCeiver quick start with a full configuration | [7](minutes.md#topic-7-recsync-recceiver-reccaster-and-the-channelfinder-ecosystem) | |
| Tynan Ford | RecCaster unicast mode | [7](minutes.md#topic-7-recsync-recceiver-reccaster-and-the-channelfinder-ecosystem) | [reccaster#24](https://github.com/ChannelFinder/reccaster/issues/24) open |
| Tynan Ford | PVWS CORS and Docker PRs | [17](minutes.md#topic-17-morning-working-session-oct-22) | CORS: [pvws#14](https://github.com/ControlSystemStudio/pvws/pull/14) open (opened by georgweiss) |
| Michael Davidsaver | Explore an EPICS module that publishes an IOC's PV interface | [4](minutes.md#topic-4-auto-generating-engineering-screens-side-discussion) | |
| Kunal Shroff | Merge DescriptionProvider into a pre-release; NT `descriptor` ticket | [14](minutes.md#topic-14-vtypes-separating-description-from-display-epicscorejava-descriptionprovider) | [epicsCoreJava#146](https://github.com/epics-base/epicsCoreJava/pull/146) open |
| Oak Ridge | Test DBWR against the VType pre-release | [14](minutes.md#topic-14-vtypes-separating-description-from-display-epicscorejava-descriptionprovider) | |
| Oak Ridge | Second review of the Oracle UTC PR | [16](minutes.md#topic-16-afternoon-working-session-oct-21) | [#3905](https://github.com/ControlSystemStudio/phoebus/pull/3905) open |
| Oak Ridge | Contact for the ScanEngine | [16](minutes.md#topic-16-afternoon-working-session-oct-21) | |
| CEA demo participants | Edge cases and screenshots on PR #3928 | [11](minutes.md#topic-11-showing-truncated-values-in-text-entry-and-text-update-pr-3928) | [#3928](https://github.com/ControlSystemStudio/phoebus/pull/3928) open |
| Kunal Shroff | Ansible role and training VM | [18](minutes.md#topic-18-afternoon-working-session-and-wrap-up-oct-22) | |
| Andrew | Wiki page for the codeathon results | [18](minutes.md#topic-18-afternoon-working-session-and-wrap-up-oct-22) | |

## 4. Not decided (agree who drives each and by when)

| # | Question | Topic |
|---|---|---|
| N1 | DBWR or Daedalus as the community web runtime, and who maintains DBWR. | [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security) |
| N2 | Whether to merge PVInfo with the CEA alarm web. | [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security) |
| N3 | How to authorize alarm acknowledgements and configuration changes. | [3](minutes.md#topic-3-alarm-services) |
| N4 | How to support multiple alarm hierarchies over the same PVs. | [3](minutes.md#topic-3-alarm-services) |
| N5 | Whether to add partial restore of composite snapshots. | [8](minutes.md#topic-8-save-and-restore-facility-workflows-ui-client-vs-service-restore-restore-ordering-and-deadlocks) |
| N6 | Whether service restore should stay available to Phoebus clients. | [8](minutes.md#topic-8-save-and-restore-facility-workflows-ui-client-vs-service-restore-restore-ordering-and-deadlocks) |
| N7 | The end-of-support date for the legacy Olog. | [6](minutes.md#topic-6-olog-ai-search-python-client-and-the-legacy-olog) |
| N8 | prek or pre-commit ([#3980](https://github.com/ControlSystemStudio/phoebus/issues/3980)). | [17](minutes.md#topic-17-morning-working-session-oct-22) |
| N9 | Whether to support a limited CA→PVA bridge for legacy systems. | [13](minutes.md#topic-13-gateways-pva-gateway-a-new-ca-gateway-and-capva-bridging) |

## 5. Quick facts to confirm (by email)

- Is the display library DLS mentioned Weiss? Is the DLS project in the screen-generation discussion ibek? [2](minutes.md#topic-2-web-toolkits-pvws-dbwr-pvinfo-and-web-security), [4](minutes.md#topic-4-auto-generating-engineering-screens-side-discussion)
- Was the third site that relies on the `-settings` launch failure ITER? [12](minutes.md#topic-12-phoebus-launch-when-the--settings-file-is-missing)
- Which site hosts Mistral AI on site? Has NSLS-II moved to KRaft? Which facility is considering the ScanEngine? [6](minutes.md#topic-6-olog-ai-search-python-client-and-the-legacy-olog), [3](minutes.md#topic-3-alarm-services), [16](minutes.md#topic-16-afternoon-working-session-oct-21)
- The link to the GitHub discussion "Color alarm sensitive widgets", and what the Oct 20 color discussion concluded. [10](minutes.md#topic-10-alarm-sensitive-widgets-and-alarm-colors)
- The timeline and testers for CA Gateway v3. Performance targets for the Java RecCeiver. [13](minutes.md#topic-13-gateways-pva-gateway-a-new-ca-gateway-and-capva-bridging), [7](minutes.md#topic-7-recsync-recceiver-reccaster-and-the-channelfinder-ecosystem)
