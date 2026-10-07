# Core Developers Discussion Topics
## Monday, October 19th, 2026

This document outlines the technical discussion topics for the Phoebus Tools and Services core developers session.

---

## Discussion Format

- **Duration:** 
- **Participants:** Developers and maintainers
- **Goal:** Strategic planning, architecture discussions, and technical decisions

---

## Technical Discussion Topics

### 1. Architecture & Framework

#### 1.1 Phoebus Framework Evolution
- [ ] Future of the RCP (Rich Client Platform) architecture
- [ ] Plugin system improvements and standardization
- [ ] Resource management and memory optimization
- [ ] Cross-platform compatibility challenges (Windows, Linux, macOS)

#### 1.2 UI/UX Framework Modernization
- [ ] JavaFX version strategy and roadmap
- [ ] CSS styling and theming improvements
- [ ] Responsive design patterns for different screen sizes
- [ ] Dark mode and accessibility enhancements
- [ ] Custom controls and widget library expansion

#### 1.3 Build System & Dependencies
- [ ] Dependency management and version conflicts
- [ ] Modularization strategy (Java modules/JPMS)
- [ ] CI/CD pipeline improvements

#### 1.4 Java Platform Modernization
- [ ] **JDK 21 LTS Adoption**
  - Virtual Threads: improve scalability
  - Sequenced Collections: ???
  - Pattern Matching for switch
  - Record Patterns: ???
- [ ] **JDK 25 (Future) Considerations**
  - Structured Concurrency: Simplified multi-threaded
  - Stream Gatherers
- [ ] **Migration Strategy**
  - Impact on JavaFX compatibility and performance
  - Virtual Threads integration with existing Phoebus threading model
  - Timeline and rollout plan for JDK 21 adoption

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

#### 2.4 Olog Integration
- [ ] Search and indexing improvements (Elasticsearch integration)
- [ ] Template system enhancements
- [ ] Integration with other services (ChannelFinder, Alarm)
- [ ] Notification mechanisms

#### 2.5 ChannelFinder Service
- [ ] Property and tag management
- [ ] Scalability for large installations (10M+ channels)
- [ ] Integration with PVAccess and ChannelAccess Nameserver

#### 2.6 Alarm Services
- [ ] Alarm configuration management
- [ ] Notification systems (email, SMS, messaging platforms)
- [ ] Alarm history and analytics

---

### 3. Data Access & Performance

#### 3.1 EPICS Core Java (PVA)
- [ ] Connection management and reconnection strategies
- [ ] Compatibility with EPICS 7 features

#### 3.2 PV Access Layer
- [ ] ???

---

### 4. Development Practices & Community

#### 4.1 Code Quality & Standards
- [ ] Coding standards and style guides
- [ ] Static analysis tools integration
- [ ] Unit testing and test coverage goals
- [ ] Documentation standards

#### 4.2 Release Management
- [ ] Versioning strategy (semantic versioning)
- [ ] Release cycle and timing
- [ ] LTS (Long Term Support) versions
- [ ] Backwards compatibility policies
- [ ] Migration guides and upgrade paths

#### 4.3 Community Engagement
- [ ] ???

#### 4.4 Documentation Strategy
- [ ] **Documentation Structure by User Profile** ([#3558](https://github.com/ControlSystemStudio/phoebus/issues/3558))
  - Reorganize docs by user type: Operator, Display Designer, Sysadmin, Contributor?
  - Consider [Diátaxis](https://diataxis.fr/) framework (Tutorials, Guides, References, Explanations)
  - Move CONTRIBUTING.md and ARCHITECTURE.md to repository root
- [ ] **Multi-version Documentation** ([#3556](https://github.com/ControlSystemStudio/phoebus/issues/3556))
  - Build docs for all tagged releases on ReadTheDocs
  - Default to `stable` (latest release) instead of `latest` (master)
  - Enable version selector for older releases
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

#### 5.2 Deployment Strategies
- [ ] Containerization (Docker, Kubernetes)
- [ ] Configuration management
- [ ] Monitoring and alerting
- [ ] Disaster recovery and backup strategies

---

### 6. Roadmap & Future Features

#### 6.1 Roadmap 
- [ ] Timeline for major releases


#### 6.2 Future Features 
- [ ] Web-based interfaces vs native applications
- [ ] Mobile support considerations
- [ ] AI/ML integration opportunities
- [ ] Next-generation control system requirements

---

### 7. Technical Debt & Refactoring
- [ ] Identification of major technical debt areas
- [ ] Deprecated API removal strategy
- [ ] Legacy code migration plans

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
