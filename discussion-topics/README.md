# Core Developers Discussion Topics
## Monday, October 19th, 2026

This document outlines the technical discussion topics for the Phoebus Tools and Services core developers session.

---

Completed topics from earlier events are in the [archive](../archive/README.md).

---

## Discussion Format

- **Duration:** 
- **Participants:** Developers and maintainers
- **Goal:** Strategic planning, architecture discussions, and technical decisions

---

## Technical Discussion Topics

### 1. Architecture & Framework

#### 1.1 Phoebus Framework Evolution
- [ ] Plugin system improvements and standardization
- [ ] Resource management and memory optimization
- [ ] Cross-platform compatibility challenges (Windows, Linux, macOS)
  - Windows is used in control rooms and beamlines (Fermilab, SNS, PSI, Saclay): UNC/network-drive paths ([#3837](https://github.com/ControlSystemStudio/phoebus/issues/3837), [PR #3861](https://github.com/ControlSystemStudio/phoebus/pull/3861))
  - ARM / aarch64 support: JavaFX availability and docs build ([#3059](https://github.com/ControlSystemStudio/phoebus/issues/3059), [#3249](https://github.com/ControlSystemStudio/phoebus/issues/3249))
  - macOS and Debian build/runtime issues ([#3042](https://github.com/ControlSystemStudio/phoebus/issues/3042), [#3003](https://github.com/ControlSystemStudio/phoebus/issues/3003), [#3691](https://github.com/ControlSystemStudio/phoebus/issues/3691))
- [ ] Launcher and startup contract
  - Formalize launcher behavior so facility-specific startups are not broken by changes ([#3828](https://github.com/ControlSystemStudio/phoebus/issues/3828) `-clean` with `-app`/`-resource`)
  - [ ] Window positioning from BOB x/y properties ([#3871](https://github.com/ControlSystemStudio/phoebus/issues/3871))
  - Deployment-only / multi-user product ([#3515](https://github.com/ControlSystemStudio/phoebus/issues/3515), [#410](https://github.com/ControlSystemStudio/phoebus/issues/410))

#### 1.2 UI/UX Framework Modernization
- [ ] CSS styling and theming improvements
- [ ] Responsive design patterns for different screen sizes
- [ ] Dark mode and accessibility enhancements
- [ ] Custom controls and widget library expansion
- [ ] Central color/font service and generalized color definitions ([#3185](https://github.com/ControlSystemStudio/phoebus/issues/3185))
- [ ] Performance with many windows/tabs and high update rates ([#3169](https://github.com/ControlSystemStudio/phoebus/issues/3169), RepresentationUpdateThrottle findings from DLS)
  - Partially addressed: update throttle singleton merged in [PR #3792](https://github.com/ControlSystemStudio/phoebus/pull/3792) (2026-05-11); issue still open

#### 1.3 Build System & Dependencies
- [ ] Dependency management and version conflicts
  - Duplicate library versions in the product build; messy dependency tree
  - Large transitive dependencies ([#3123](https://github.com/ControlSystemStudio/phoebus/issues/3123))
- [ ] Modularization strategy (Java modules/JPMS)
- [ ] CI/CD pipeline improvements

#### 1.4 Java Platform Modernization
- [ ] **JDK 21 / 25 Follow-up and Stabilization**
  - Review edge cases and production regressions after the major dependency refresh
  - Confirm JavaFX compatibility, performance, and threading assumptions after the upgrade
- [ ] **Dependency Upgrade Cadence**
  - Keep major framework upgrades smaller and regular instead of large, disruptive "big bang" migrations
  - Coordinate JDK, Spring Boot, Kafka, and Elasticsearch upgrades with release planning
- [ ] **Migration Strategy**
  - Impact on JavaFX compatibility and performance
  - Virtual Threads integration with existing Phoebus threading model
  - Timeline and rollout plan for JDK 21 adoption
  - Release sequencing between Phoebus 6 and Phoebus 7 branches
- [ ] **Adopting newer JDK language features**
  - Bulk modernization (switch patterns, records) using SonarCloud/IntelliJ reports ([#3708](https://github.com/ControlSystemStudio/phoebus/issues/3708))
  - Garbage collector behavior on JDK 25 (generational mode): memory/CPU trade-offs and operator experience

#### 1.5 Display Builder & Operator UX
- [ ] **Navigation and macros**
  - Navigation patterns across facilities (breadcrumbs, landing pages, macro-heavy hierarchies)
  - Diagnostics for where a macro is set; consistent relative path handling ([#3803](https://github.com/ControlSystemStudio/phoebus/issues/3803), [#3873](https://github.com/ControlSystemStudio/phoebus/issues/3873))
- [ ] **Actions and rules**
  - Generic action framework (open data browser, Save & Restore, layouts) with backward compatibility
  - Formulas in Write PV action ([#3941](https://github.com/ControlSystemStudio/phoebus/issues/3941)), rules using other widgets' properties ([#3862](https://github.com/ControlSystemStudio/phoebus/issues/3862))
- [ ] **Widget behavior and defaults**
  - Per-widget update rate (currently a global throttle)
  - TextInput/TextEntry autocomplete (3rd most used widget at BNL)
  - Array widget scrollbars and spacing ([#3945](https://github.com/ControlSystemStudio/phoebus/issues/3945), [#1412](https://github.com/ControlSystemStudio/phoebus/issues/1412))
  - Machine-readable widget property defaults ([#3815](https://github.com/ControlSystemStudio/phoebus/issues/3815), [#3658](https://github.com/ControlSystemStudio/phoebus/issues/3658))
- [ ] **Display quality tooling**
  - Display linter/analyzer and automated GUI testing of displays ([#1965](https://github.com/ControlSystemStudio/phoebus/issues/1965))

#### 1.6 Data Browser & Plotting
- [ ] Alarm limits in Data Browser ([#3889](https://github.com/ControlSystemStudio/phoebus/issues/3889)) and PV disconnect events
- [ ] Optimized (last-sample) retrieval as default from Phoebus 6; more downsampling methods ([#3724](https://github.com/ControlSystemStudio/phoebus/issues/3724))
- [ ] Archive data source management and independence from Archiver Appliance ([#3432](https://github.com/ControlSystemStudio/phoebus/issues/3432))
- [ ] Plot library direction: ChartFX vs the current RT plot (RT plot needs a new owner) ([#3362](https://github.com/ControlSystemStudio/phoebus/issues/3362), [#3167](https://github.com/ControlSystemStudio/phoebus/issues/3167))
- [ ] Waterfall plots and new vtypes ([#3549](https://github.com/ControlSystemStudio/phoebus/issues/3549))
- [ ] Alternative time-series backends (Parquet, InfluxDB) ([#3363](https://github.com/ControlSystemStudio/phoebus/issues/3363))

---

### 2. Middle Layer Services

#### 2.1 Service Architecture
- [ ] REST API standardization across services
- [ ] Authentication and authorization strategy (OAuth2, JWT, LDAP)
- [ ] Service discovery and registration
- [ ] Load balancing and high availability

#### 2.2 Spring Boot 4 Migration
- [ ] Spring Security 7: Enhanced OAuth2 resource server, authorization architecture improvements
- [ ] REST API Versioning: URL-based vs header-based strategies, deprecation policies
- [ ] Virtual Threads Support: Integration with Project Loom for improved concurrency
- [ ] Native Image: GraalVM native compilation for faster startup and lower memory footprint
- [ ] Observability: OpenTelemetry integration, metrics, distributed tracing across services
- [ ] Problem Details: Standardized error responses across all REST APIs
- [ ] HTTP Interface Clients: Declarative HTTP clients replacing RestTemplate/WebClient
- [ ] Docker Compose Support: Simplified local development and testing environments

#### 2.3 Archiver Service
- [ ] Archive Appliance performance tuning
- [ ] Data retrieval optimization strategies
- [ ] Storage backend options
- [ ] **Data Browser and metadata integration**
  - Surface alarm limit metadata (HIHI, HIGH, LOW, LOLO) in Data Browser results
  - Default optimized bin behavior to "last sample" when appropriate
  - Evaluate NTAggregate / VStatistics improvements for statistical summaries
- [ ] **Operational metadata and disconnect visibility**
  - Better representation of disconnects and alarm metadata in archived data views
- [ ] **Parquet storage backend**
  - FRIB reports ~50% storage reduction; retest with compression, waveforms, and retrieval performance
  - Promote from demo release to a supported option
- [ ] **Platform and deployment**
  - Move from Tomcat to a Spring Boot style service and manage preferences accordingly
  - Replace JCA with CorePVA; JDK/Tomcat compatibility
  - Containerization / Kubernetes, cluster sizing and monitoring lessons (ESS, FRIB, NSLS-II surveys)
- [ ] **Retrieval API**
  - Pagination and streaming endpoints; alarm limit fields returning latest value regardless of time ([#3889](https://github.com/ControlSystemStudio/phoebus/issues/3889), [#3922](https://github.com/ControlSystemStudio/phoebus/issues/3922))

#### 2.4 Olog Integration
- [ ] Search and indexing improvements (Elasticsearch integration)
- [ ] Template system enhancements
- [ ] Integration with other services (ChannelFinder, Alarm)
- [ ] Notification mechanisms
- [ ] **Search language and AI-assisted queries**
  - Search is growing into a DSL; options: query object, LLM natural-language to Elasticsearch translation, result summarization
  - Same needs exist in ChannelFinder and alarm log search
- [ ] **Sunset of legacy Olog**
  - Decide timeline, data migration tool, and affected sites; keep legacy client in a separate repo meanwhile ([#2499](https://github.com/ControlSystemStudio/phoebus/issues/2499), [#3116](https://github.com/ControlSystemStudio/phoebus/issues/3116))
- [ ] OpenAPI/Swagger documentation for Olog (draft PR in progress)

#### 2.5 ChannelFinder Service
- [ ] Property and tag management
- [ ] Scalability for large installations (10M+ channels)
- [ ] Integration with PVAccess and ChannelAccess Nameserver
- [ ] **PVAccess name resolution and CF NameServer**
  - Shared behavior for search/backoff/timeout logic across name services
  - Consider moving common NameService logic into CorePVA / pvAccess libraries
  - Decide how fallback and broadcast behavior should be configured in production deployments
- [ ] **API and service modernization**
  - Batch operations and REST API cleanup
  - Backward compatibility for existing clients while preparing v1 API improvements
  - RESTful design (properties as PATCH on a channel, owner reserved for authentication, map vs array property format)
- [ ] **RecSync restructuring**
  - New Java RecCeiver (testing at 2M to 4M channels, volunteers from ALS, ESS, ISIS)
  - RecCaster as a standalone EPICS module with a configurable RecCeiver list instead of UDP beacon discovery
  - Protocol compatibility between RecCaster and RecCeiver versions; possible rename of the repo
- [ ] **Service architecture options**
  - Keep separate, merge ChannelFinder with CFNameServer, merge all, or merge RecCeiver with ChannelFinder/NameServer
- [ ] Read-only / unauthenticated ChannelFinder client mode ([#2070](https://github.com/ControlSystemStudio/phoebus/issues/2070)) and a possible MCP endpoint for AI tooling

#### 2.6 Alarm Services
- [ ] Alarm configuration management
  - Batch add/edit of PVs, import/export, switching between many configurations (ESS runs 40+)
  - Interactive alarm editing proposal ([#3920](https://github.com/ControlSystemStudio/phoebus/issues/3920))
  - Offline mode and REST API for configuration history
  - Dynamic/gated alarms for operating modes (ISIS, FRIB, Fermilab PIP-II, CEA): use cases first, then a possible EPICS module
- [ ] Notification systems (email, SMS, messaging platforms)
  - Annunciator: until-acknowledged option, translations ([#3944](https://github.com/ControlSystemStudio/phoebus/issues/3944), [#3844](https://github.com/ControlSystemStudio/phoebus/issues/3844), [#2968](https://github.com/ControlSystemStudio/phoebus/issues/2968))
- [ ] Alarm history and analytics
  - [ ] "Disable until" handling in the logger ([#3032](https://github.com/ControlSystemStudio/phoebus/issues/3032)); table usability ([#3943](https://github.com/ControlSystemStudio/phoebus/issues/3943))
- [ ] Kafka operations
  - Run Kafka 4 without Zookeeper; update docs and configuration ([#3441](https://github.com/ControlSystemStudio/phoebus/issues/3441))
  - Topic auto-creation vs explicit creation, import leaving stale topics, Elasticsearch high-water-mark stalls in the logger
  - Alarm config logger repo layout for multiple alarm servers ([#3015](https://github.com/ControlSystemStudio/phoebus/issues/3015))
- [ ] Authorization and audit
  - Per-user permissions, per-alarm permissions ([#1336](https://github.com/ControlSystemStudio/phoebus/issues/1336)), and security concerns ([#3702](https://github.com/ControlSystemStudio/phoebus/issues/3702))

#### 2.7 Save & Restore
- [ ] Access security implications of "restore from service" ([#3357](https://github.com/ControlSystemStudio/phoebus/issues/3357))
- [ ] Credentials handling and embedded LDAP / hashed passwords ([#3045](https://github.com/ControlSystemStudio/phoebus/issues/3045), [#3378](https://github.com/ControlSystemStudio/phoebus/issues/3378))
- [ ] Compare current values to stored snapshots
- [ ] Binary values in snapshots ([#2755](https://github.com/ControlSystemStudio/phoebus/issues/2755))

#### 2.8 Service Health, Testing & Realtime Updates
- [ ] Health endpoint reporting improvements (start with Alarm Logger)
- [ ] WebSocket updates with polling fallback for older services
- [ ] End-to-end and integration tests for Elasticsearch/Jackson bindings (mocks missed errors during the upgrade)
- [ ] OpenAPI standard for all services

---

### 3. Data Access & Performance

#### 3.1 EPICS Core Java (PVA)
- [ ] Connection management and reconnection strategies
- [ ] Compatibility with EPICS 7 features
- [ ] **PVAccess name resolution and search algorithm compatibility**
  - Align ChannelFinder and other name services with the pvxs/PVAccess search behavior
  - Standardize backoff and timeout policies across implementations
  - Evaluate QSERV2 / pvxs integration and configuration exposure for host/port settings
- [ ] **Ownership and maintenance of CorePVA**
  - Kay Kasemir's retirement: CorePVA, Secure PVAccess, RT plot library, DBWR, PVWS need owners
  - SPVA versioning and keychain layout ([#3865](https://github.com/ControlSystemStudio/phoebus/issues/3865))
- [ ] **Known protocol issues**
  - Search period and excessive search logging ([#3705](https://github.com/ControlSystemStudio/phoebus/issues/3705), [#3654](https://github.com/ControlSystemStudio/phoebus/issues/3654))
  - CA/PVA array decoding differences ([#3536](https://github.com/ControlSystemStudio/phoebus/issues/3536), [#3791](https://github.com/ControlSystemStudio/phoebus/issues/3791)) and failures on missing fields ([#3248](https://github.com/ControlSystemStudio/phoebus/issues/3248))
  - Blocked TCP connections caused by IOC behavior: threading approach after the mitigation release

#### 3.2 PV Access Layer
- [ ] Name service interoperability and fallback behavior
- [ ] Production readiness of alternative name server implementations
- [ ] Service discovery and protocol compatibility across EPICS deployments

---

### 4. Development Practices & Community

#### 4.1 Code Quality & Standards
- [ ] Coding standards and style guides
- [ ] Static analysis tools integration
- [ ] Unit testing and test coverage goals
- [ ] Documentation standards
- [ ] Java code conventions ([#894](https://github.com/ControlSystemStudio/phoebus/issues/894)) and IDE guidance (IntelliJ first; drop or de-emphasize Eclipse configuration)

#### 4.2 Release Management
- [ ] Versioning strategy (semantic versioning)
  - Tooling to detect breaking public API changes; relax major-version bumps tied to JDK changes
- [ ] Release cycle and timing ([#1380](https://github.com/ControlSystemStudio/phoebus/issues/1380))
  - Release tagging/versioning process and Maven Central publishing
- [ ] LTS (Long Term Support) versions
- [ ] Backwards compatibility policies
- [ ] Migration guides and upgrade paths

#### 4.3 Community Engagement
- [ ] Codeathon planning and follow-up from previous sessions
- [ ] Registration and travel logistics for in-person events and site access
- [ ] Project review and onboarding for new contributors after codeathon work
- [ ] Coordination across facilities for shared roadmap themes and architecture changes
- [ ] **Maintainer and testing capacity**
  - Owners for areas left by Kay Kasemir's retirement (CorePVA, RT plot, Secure PVA, DBWR/PVWS, Scan Server)
  - Volunteers to test large upgrade PRs and pre-release builds; keep the open PR/issue backlog falling
  - Triage of 240+ open issues: close stale, label for codeathon

#### 4.4 Documentation Strategy
- [ ] **Documentation Structure by User Profile** ([#3558](https://github.com/ControlSystemStudio/phoebus/issues/3558))
  - Reorganize docs by user type: Operator, Display Designer, Sysadmin, Contributor?
  - Consider [Diátaxis](https://diataxis.fr/) framework (Tutorials, Guides, References, Explanations)
  - Move CONTRIBUTING.md and ARCHITECTURE.md to repository root
- [ ] **Documentation Standardization Across Repositories**
  - Consistent documentation structure across Phoebus, Olog, ChannelFinder, Archive Appliance, etc.
  - Standardize section organization (Getting Started, Installation, Configuration, API Reference, etc.)
  - Common tooling: Sphinx vs MkDocs vs other static site generators
  - Auto-generated documentation consistency (JavaDoc, OpenAPI/Swagger, Sphinx autodoc)
  - Shared documentation templates and style guides
  - Cross-repository linking strategy
  - Hosting strategy (ReadTheDocs vs GitHub Pages vs self-hosted)

---

### 5. Security & Deployment

#### 5.1 Security Considerations
- [ ] Secure communication channels (TLS/SSL)
- [ ] Credential management (avoiding hardcoded passwords)
- [ ] Access control and permissions model
- [ ] Dependency vulnerabilities and AI-driven vulnerability reports (including the JCA server implementation used in tests) ([#2300](https://github.com/ControlSystemStudio/phoebus/issues/2300))
- [ ] Certificate/hostname verification policy for service clients (configurable permissiveness)
- [ ] Access control for web clients (PVWS, PVA gateways)

#### 5.2 Deployment Strategies
- [ ] Containerization (Docker, Kubernetes)
  - PVWS and services in containers with host networking for CA/PVA
- [ ] Configuration management
  - `phoebus-ansible` repository and merging middleware services; training VM / reference stack for workshops
- [ ] Monitoring and alerting
- [ ] Disaster recovery and backup strategies

---

### 6. Roadmap & Future Features

#### 6.1 Roadmap 
- [ ] Timeline for major releases


#### 6.2 Future Features 
- [ ] Web-based interfaces vs native applications
  - Web toolkits working group, DLS Daedalus / cs-web-lib, PVWS merge, and long-term UI technology (JavaFX vs alternatives)
- [ ] Mobile support considerations
- [ ] AI/ML integration opportunities
  - LLM-assisted display generation and validation, development assistants, natural-language logbook search, MCP endpoints, Osprey
- [ ] Next-generation control system requirements
- [ ] Bluesky Queue Server / OAC-tree integration and experiment-facing workflows

---

### 7. Technical Debt & Refactoring
- [ ] Identification of major technical debt areas
- [ ] Deprecated API removal strategy
- [ ] Legacy code migration plans
- [ ] Resource leaks (local PVs, display close/reload) ([#3729](https://github.com/ControlSystemStudio/phoebus/issues/3729), [#3582](https://github.com/ControlSystemStudio/phoebus/issues/3582))
- [ ] Central lifecycle management (Cleaner instance) ([#2962](https://github.com/ControlSystemStudio/phoebus/issues/2962))
- [ ] Python scripting: performance and exception handling vs the UI thread

---

## Meeting Notes

Discussion on alarm services:

- Current philosophy and work flow of adding/updated alarm configs is clunky and rigid.
- Proposal for Phoebus: support batch operations (add PVs) in the alarm config UI.
    - Selection of PVs in (for instance) channel finder UI, add to alarm config from context menu.
    - Batch operation to add PVs in the alarm tree UI not a feature needed by everybody.
- General concern: no user level access control to operations.
- Feature request: context menu item to "disable until", easier than launching the config dialog.
- Feature request: "disable until" to use a preference to avoid the need to specify end date/time.
- Actions on alarm config: who did what and why... Integration with Olog?
- Update UI to indicate that Kafka producer messages cannot be sent (read-only client). New preference?
- Proposal: add REST API on top of alarm config logger, e.g. get alarm config at specific date.
- Support an "offline mode" alarm tree where operations act on the configuration (e.g. in memory).
  - Load alarm config file and render tree UI.
  - Make changes in UI/memory
  - Save changes from memory to config file.
- Actions not allowed (e.g. no producer messages) must provide feedback or be disabled.
- REST API for Kafka producer messages for ack, disable, config change?
  - This would be a new service replacing (?) producer messages from alarm UIs.

---

## References

- [Phoebus Documentation](https://github.com/ControlSystemStudio/phoebus/tree/master/docs)
- [EPICS Website](https://epics-controls.org/)
- [Previous Codeathon Reports](https://epics.anl.gov/meetings/)

