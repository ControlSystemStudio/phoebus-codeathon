# 2026 February Codeathon (Diamond Light Source)

## Summary

- Event: February 2026 EPICS Codeathon hosted by Diamond Light Source
- Results: [2026 Codeathon Project Results](https://github.com/epics-base/epics-base/wiki/2026-Codeathon-Project-Results)
- Minutes: [Collaboration meeting minutes](https://docs.google.com/document/d/13bLL-JE9knMuv3FnUysZkpgDQgCkbXEk7beRkDKO-go/edit?usp=sharing)
- Focus here: Phoebus and the middle layer services

## Completed Topics

Resolved between February and October 2026 (moved from [discussion-topics](../../discussion-topics/README.md)).

| Topic | Outcome | Reference |
|-------|---------|-----------|
| JDK 25 upgrade and dependency refresh | Done, including Spring Boot. Stabilization follow-up remains open in section 1.4. | [Project Results](https://github.com/epics-base/epics-base/wiki/2026-Codeathon-Project-Results) |
| JavaFX version strategy and roadmap | Discussed and closed | |
| Standalone window mode | Merged 2026-07-09 | [PR #3859](https://github.com/ControlSystemStudio/phoebus/pull/3859), [#3543](https://github.com/ControlSystemStudio/phoebus/issues/3543) |
| Alarm logger "enabled" date deserialization | Merged 2025-03-20 | [PR #3330](https://github.com/ControlSystemStudio/phoebus/pull/3330), [#3304](https://github.com/ControlSystemStudio/phoebus/issues/3304) |
| Save & Restore tree table UI | Merged 2026-07-10 | [PR #3872](https://github.com/ControlSystemStudio/phoebus/pull/3872) |
| Save & Restore snapshot delete / unique IDs | Merged 2025-10-17 | [PR #3593](https://github.com/ControlSystemStudio/phoebus/pull/3593), [#3587](https://github.com/ControlSystemStudio/phoebus/issues/3587) |
| Multi-version documentation | Issue closed; proof of concept site | [#3556](https://github.com/ControlSystemStudio/phoebus/issues/3556), https://phoebus-test.readthedocs.io |

## Completed Tasks

Moved from [documentation-tasks](../../documentation-tasks/README.md) and [projects](../../projects/README.md).

| Task | Outcome | Reference |
|------|---------|-----------|
| ALARM-KAFKA-002: Resilient topic handling with retry logic (Loic Caouen, CEA) | Tested and merged; also removed State topic from `delete_alarm_topics.sh` | [Project Results](https://github.com/epics-base/epics-base/wiki/2026-Codeathon-Project-Results) |
| PHOEBUS-UI-005: Default email address preferences (Martin Gaughran, DLS) | Merged 2026-03-02 | [PR #3717](https://github.com/ControlSystemStudio/phoebus/pull/3717) |
| SERVICES-HEALTH-001: Health endpoints for Phoebus services (Kunal Shroff, BNL) | Merged for Phoebus and Olog | [PR #3714](https://github.com/ControlSystemStudio/phoebus/pull/3714), [Olog PR #256](https://github.com/Olog/phoebus-olog/pull/256) |
| Core-PVA DBE_MASK support (Sky Brewer, ESS) | Merged 2026-03-02 | [PR #3710](https://github.com/ControlSystemStudio/phoebus/pull/3710) |
| Data Browser enhancements from CSNS: smoothing, waveform overlap (Georg Weiss, ESS) | Merged 2026-02-27 | [PR #3462](https://github.com/ControlSystemStudio/phoebus/pull/3462) |
| SonarCloud for Phoebus and Olog; recsync-rs linting; ChannelFinderService Swagger UI fix (Sky Brewer) | Merged | [ChannelFinderService PR #203](https://github.com/ChannelFinder/ChannelFinderService/pull/203), [recsync-rs PR #7](https://github.com/ChannelFinder/recsync-rs/pull/7) |
| Migrate Olog documentation to new structure (Remi Nicole, Georg Weiss) | Merged | https://olog.readthedocs.io/en/latest/ |
| Improve Olog user/operator documentation (DOC) | Merged 2026-03-02 | [Olog PR #257](https://github.com/Olog/phoebus-olog/pull/257) |
| Sphinx template for Phoebus projects (Remi Nicole, CEA) | Template and instructions published | [documentation-tasks/template](../../documentation-tasks/template) |
| Common user roles and wanted articles for ChannelFinder, Olog, Alarm, Archiver Appliance | List produced | [Google Doc](https://docs.google.com/document/d/10oipk7mUnPIMryraRGQpPgFCsdGvfIuPCNlf4ueU-sc/edit?usp=sharing) |
