# Development Projects
## 2026 EPICS Developers Meeting and Codeathon - Phoebus Track

This directory contains the list of development projects for the Phoebus tools and services during the event week.

---

## Project Index & Sign-up Sheet

| Project ID | Title | Difficulty | Assigned To | Status |
|------------|-------|------------|-------------|--------|
| **Alarm Services** |||||
| [ALARM-KAFKA-001](#alarm-kafka-001-enhanced-kafka-streams-error-handling) | Enhanced Kafka Streams Error Handling | Advanced | | Not Started |
| [ALARM-KAFKA-002](#alarm-kafka-002-resilient-topic-handling-with-retry-logic) | Resilient Topic Handling with Retry Logic | Intermediate | | Not Started |
| [ALARM-REST-001](#alarm-rest-001-alarm-configuration-rest-api) | Alarm Configuration REST API | Intermediate | Anthony Carriveau | Started |
| [ALARM-UI-001](#alarm-ui-001-display-highlow-alarm-limits-in-alarm-tree-tooltip) | Display High/Low Alarm Limits in Alarm Tree Tooltip | Intermediate | | Not Started |
| [ALARM-UI-002](#alarm-ui-002-improve-alarm-log-table-searchquery-ui) | Improve Alarm Log Table Search/Query UI | Intermediate | ISIS | Started |
| [ALARM-TOPICS-001](#alarm-topics-001-centralized-kafka-topic-management-service) | Centralized Kafka Topic Management Service | Intermediate | Georg Weiss (ESS) | In Progress |
| **Phoebus UI & Framework** |||||
| [PHOEBUS-JDK25-001](#phoebus-jdk25-001-utilize-jdk-25-java-design-patterns) | Utilize JDK 25 Java Design Patterns | Advanced | | Not Started |
| [PHOEBUS-UI-001](#phoebus-ui-001-fix-timerangepopover-relative-time-selection) | Fix TimeRangePopover Relative Time Selection | Beginner | | Not Started |
| [PHOEBUS-UI-002](#phoebus-ui-002-data-browser-archive-data-source-management) | Data Browser Archive Data Source Management | Intermediate | Kunal, Sky| Started |
| [PHOEBUS-UI-003](#phoebus-ui-003-fix-pv-resource-leak-in-widgetruntime) | Fix PV Resource Leak in WidgetRuntime | Beginner | Kunal | Started |
| [PHOEBUS-UI-004](#phoebus-ui-004-interactive-graph-widget-for-xyplot) | Interactive Graph Widget for XYPlot | Advanced | | Not Started |
| [PHOEBUS-UI-005](#phoebus-ui-005-chartfx-vs-rt-plot-review) | ChartFX vs RT Plot Review | Intermediate | | Not Started |
| [PHOEBUS-UI-006](#phoebus-ui-006-remove-hardcoded-colors-for-css-consistency) | Remove Hardcoded Colors for Consistency | Intermediate | Urban Bobek (Cosylab) | In Progress |
| [PHOEBUS-DEV-001](#phoebus-dev-001-verify-intellij-idea-setup-instructions) | Verify IntelliJ IDEA Setup Instructions | Beginner | | Not Started |
| [PHOEBUS-LINT-001](#phoebus-lint-001-display-builder-screen-linter) | Display Builder Screen Linter | Beginner | | Not Started |
| **Middle Layer Services** |||||
| [SERVICES-SB4-001](#services-sb4-001-spring-boot-4-migration-planning) | Spring Boot 4 Migration Planning | Advanced | | Started |
| [SERVICES-VERSIONING-001](#services-versioning-001-rest-api-versioning-strategy) | REST API Versioning Strategy | Intermediate | | Started |
| [SERVICES-WEBSOCKET-001](#services-websocket-001-websocket-support-as-a-alternative-to-polling-phoebus-services) | WebSocket Support as Alternative to Polling | Intermediate | | Not Started |
| [SERVICES-OLOG-001](#services-olog-001-elasticsearch-embeddings-index-mapping-and-template-update-for-olog) | Elasticsearch embeddings index mapping and template update for Olog | Intermediate | | Not Started |
| [SERVICES-CFNS-001](#services-cfns-001-multi-threaded-channelfinder-nameserver-with-broadcast-fallback) | Multi-threaded ChannelFinder Nameserver with Broadcast Fallback | Intermediate | | Not Started |
| [SERVICES-CF-DOC-001](#services-cf-doc-001-channel-lifecycle-documentation-and-diagram-for-the-channelfinder-ecosystem) | Channel lifecycle documentation and diagram for the ChannelFinder ecosystem | Beginner | | Not Started |
| [SERVICES-RECSYNC-002](#services-recsync-002-restructure-recsync-repo-and-modernize-python-packaging) | Restructure recsync repo and modernize Python packaging | Beginner | | Not Started |
| [SERVICES-RECSYNC-003](#services-recsync-003-receiver-lifecycle-state-cleanup) | RecCeiver lifecycle/state cleanup | Beginner | | Not Started |
| [SERVICES-RECSYNC-004](#services-recsync-004-direct-pva-rpc-from-reccaster-to-channelfinder) | Direct PVA-RPC from RecCaster to ChannelFinder | Advanced | | Not Started |
| **AI/ML Projects** |||||
| [AI-DISPLAY-001](#ai-display-001-llm-assisted-display-screen-generation) | LLM-Assisted Display Screen Generation | Advanced | | Not Started |
| [AI-DISPLAY-002](#ai-display-002-llm-based-display-screen-cicd-validator) | LLM-Based Display Screen CI/CD Validator | Intermediate | | Not Started |
| [AI-ASSIST-001](#ai-assist-001-llm-powered-phoebus-development-assistant) | LLM-Powered Phoebus Development Assistant | Intermediate | | Not Started |
| [AI-LOG-001](#ai-log-001-intelligent-log-analysis-for-alarm-logs) | Intelligent Log Analysis for Alarm Logs | Intermediate | | Not Started |
| **DevOps & Infrastructure** |||||
| [DEVOPS-ANSIBLE-001](#devops-ansible-001-phoebus-infrastructure-deployment) | Phoebus Infrastructure Deployment | Intermediate | | Not Started |
| [DEVOPS-ANSIBLE-002](#devops-ansible-002-test-and-validation-of-ansible-phoebus-roles) | Test and Validation of ansible-phoebus Roles | Intermediate | | Not Started |
| [DEVOPS-CONTAINER-001](#devops-container-001-improve-dockerfiles-and-compose-for-phoebus-services) | Improve Dockerfiles and Compose for Phoebus Services | Intermediate | | Not Started |
| [DEVOPS-VM-001](#devops-vm-001-epics-services-training-vm) | EPICS Services Training VM | Intermediate | | Not Started |

---

## How to Use This Guide

1. Browse through the project list
2. Check the **Difficulty** level (Beginner, Intermediate, Advanced)
3. Review **Prerequisites** and **Skills Required**
4. Sign up by adding your name to the project
5. Create a GitHub issue in the respective repository
6. Fork the repository and create a feature branch
7. Submit a pull request when ready

---

## Projects

### ALARM-KAFKA-001: Enhanced Kafka Streams Error Handling

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Advanced  
**Skills Required:** Java, Kafka Streams API, Error Handling Patterns  

**Description:**  
Implement robust error handling for Kafka Streams in alarm logger and alarm config logger services using the latest Kafka Streams API patterns. Currently, services can fail silently or terminate when encountering deserialization errors, malformed messages, transient Kafka issues, or missing/deleted topics.

- Add `StreamsUncaughtExceptionHandler` to all KafkaStreams instances in `AlarmMessageLogger`, `AlarmCmdLogger`, and `AlarmConfigLogger` to handle fatal errors and decide on stream continuation vs. shutdown
- Implement custom `DeserializationExceptionHandler` for handling corrupted messages without stopping the entire stream
- Add `ProductionExceptionHandler` for producer-side error handling
- Handle `MissingSourceTopicException` by logging the error, preventing service crash, and allowing retry logic to eventually reconnect when topics are created
- Log structured error information (message offset, partition, topic, error type) for debugging
- Implement graceful degradation: continue processing valid messages despite errors in other messages
- Add metrics/counters for error rates per error type (deserialization failures, missing topics, transient errors)
- Create unit tests with intentionally malformed Kafka messages and missing topic scenarios

**Resources:**
- Kafka Streams error handling: https://kafka.apache.org/41/documentation/streams/developer-guide/config-streams.html#default-deserialization-exception-handler

**Assigned To:** _Available_

---


### ALARM-REST-001: Alarm Configuration REST API

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Intermediate  
**Skills Required:** Java, Spring Boot, REST API Design, Kafka  

**Description:**  
Create a REST API for the alarm configuration service that exposes read-only access to the current alarm tree configuration.

- Add REST controller to alarm-config-logger or create new alarm-config-service module
- Implement GET endpoints:
  - `/api/config/{root}` - Get entire alarm tree for configuration root
- Return the complete alarm config as xml (or json)
- Add OpenAPI/Swagger documentation
- Implement proper HTTP status codes (404 for missing nodes)

**Assigned To:** _Anthony Carriveau_

---

### ALARM-UI-001: Display High/Low Alarm Limits in Alarm Tree Tooltip

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Intermediate  
**Skills Required:** JavaFX, EPICS PVA/PVAccess, Alarm System Architecture  

**Description:**  
Enhance the Phoebus alarm tree to display high and low alarm limits (HIHI, HIGH, LOW, LOLO) in the PV tooltip. This metadata is already available through tools like Probe but would be valuable context directly in the alarm tree view. The challenge is efficiently handling limit updates without reconnecting or making excessive PV reads.

- Add alarm limit fields to alarm tree tooltip display: HIHI, HIGH, LOW, LOLO thresholds
- Implement efficient limit value retrieval:
  - Read alarm limit fields during initial PV connection establishment
  - Use PV metadata subscription to detect limit changes without reconnecting
- Update `AlarmTreeItem` or equivalent model to cache limit values
- Handle cases where limits are not defined (NaN, or not applicable for non-numeric PVs)
- Consider performance implications: avoid reading limits for every PV on every tooltip hover
- Implement lazy loading: fetch limits on first tooltip request, cache for subsequent hovers
- Display limits with appropriate formatting and units in tooltip
- Add user preference to enable/disable limit display in tooltips
- Reference ISIS implementation for listener pattern approach

**Resources:**
- ISIS Computing Group implementation: https://github.com/ISISComputingGroup/cs-studio/blob/master/applications/alarm/alarm-plugins/org.csstudio.alarm.beast/src/org/csstudio/alarm/beast/client/AlarmTreePV.java

**Assigned To:** _Available_

---

### ALARM-UI-002: Improve Alarm Log Table Search/Query UI

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Intermediate  
**Skills Required:** JavaFX, REST APIs, UI/UX Design  

**Description:**  
Redesign the alarm log table search interface to be more intuitive and user-friendly. Current query string is unnecessarily verbose (`pv=*&alarm_severity=*&alarm_message=*&pv_severity=*&pv_message=*&user=*&host=*&command=*&start=7 days&end=now`) and difficult to construct.

- Replace verbose query string with clean search form matching Logbook/Save & Restore UI patterns
- Only include non-default parameters in queries (omit `field=*` wildcards)
- Add intuitive controls: time range picker, severity dropdown, PV name field with autocomplete
- Add saved search templates for common queries
- Implement pagination or virtual scrolling for large result sets

**Resources:**
- `app/alarm/ui/src/main/java/org/phoebus/applications/alarm/ui/table/AlarmTableUI.java`
- Reference Logbook UI: `app/logbook/ui/`

**Assigned To:** _Available_

---

### ALARM-TOPICS-001: Centralized Kafka Topic Management Service

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Intermediate  
**Skills Required:** Java, Kafka Admin API, Refactoring  

**Description:**  
Refactor the alarm server's Kafka topic creation and configuration logic into a centralized utility class or service. Currently, topic management code is scattered across multiple locations (AlarmServerMain, AggregateTopics) with inconsistent patterns and configuration.

- Create a dedicated `KafkaTopicManager` service class in the alarm-server module
- Consolidate all topic creation logic from `AlarmServerMain` and `AggregateTopics` into the new service
- Implement consistent topic configuration (partitions, replication factor, retention, cleanup policy)
- Make topic attributes configurable via Phoebus preferences (replication factor, partitions, retention policy, backup settings)
- Implement topic validation and health checks (verify topic exists, check configuration matches expectations)
- Add logging for all topic operations with structured information
- Add unit tests with embedded Kafka or mock AdminClient
- Update `AlarmServerMain` and `AggregateTopics` to use the new service

**Resources:**
- Kafka AdminClient API: https://kafka.apache.org/41/javadoc/org/apache/kafka/clients/admin/AdminClient.html
- Topic configuration: https://kafka.apache.org/41/documentation.html#topicconfigs

**Assigned To:** Georg Weiss (ESS)  
**Status:** Partially complete - [PR #3715](https://github.com/ControlSystemStudio/phoebus/pull/3715) merged 2026-03-09 (configurable partitions/replication). Centralized service refactor remains.  
**Notes:** Working on allowing customization of Kafka topics partition count and replication factor.

---

### PHOEBUS-JDK25-001: Utilize JDK 25 Java Design Patterns

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Advanced  
**Skills Required:** Java 25, Concurrency, Performance Analysis, Virtual Threads, Pattern Matching  

**Description:**  
Modernize Phoebus to take advantage of JDK 25 language and runtime features, combining a Virtual Threads (Project Loom) integration assessment with modernization of legacy switch statements using modern switch expressions and pattern matching.

**Virtual Threads:**
- Document current threading model in alarm-server, alarm-logger, alarm-config-logger, save-and-restore, scan-server
- Identify blocking operations: Kafka I/O, Elasticsearch writes, Git operations, PV connections, database queries
- Create branch with Virtual Thread executors for:
  - Elasticsearch bulk write operations in `AlarmMessageLogger`
  - Git commit operations in `AlarmConfigLogger`
  - PV connection handling in `AlarmServer` (ServerModel)
  - Database operations in save-and-restore service
  - Scan engine PV processing
- Test with realistic load: 10k+ PVs, 100+ alarms/second, concurrent save-and-restore operations
- Document migration strategy: code changes, JVM requirements, configuration

**Switch Expressions & Pattern Matching:**
- Identify switch statements that can benefit from modern pattern matching
- Convert traditional switch statements to switch expressions where appropriate
- Use pattern matching for instanceof checks in switch cases
- Ensure exhaustiveness checking is leveraged
- Add tests to verify behavior remains unchanged

**Resources:**
- JDK 21 Pattern Matching: https://openjdk.org/jeps/441
- Switch Pattern Matching: https://docs.oracle.com/en/java/javase/21/language/pattern-matching.html

**Assigned To:** _Available_

---

### PHOEBUS-UI-001: Fix TimeRangePopover Relative Time Selection

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Beginner  
**Skills Required:** JavaFX, UI/UX, Time Handling  

**Description:**  
Fix inconsistent behavior in the TimeRangePopover's relative time selection controls, specifically in the TimeRelativeIntervalPane. The current implementation has confusing increment/decrement behavior, especially with months and edge cases when values reach zero.

- Analyze current behavior in `TimeRelativeIntervalPane` for relative time field adjustments
- Fix decrement behavior: when days field is 0 and decremented, it goes to 31 days (should handle more predictably)
- Fix increment behavior: incrementing days from 31 goes to 32 (months conversion logic is broken)
- Consider simplifying by removing "months" field entirely (months are inconsistent time units: 28-31 days)
- Implement consistent rollover behavior for time unit boundaries
- Add validation to prevent invalid intermediate states
- Add unit tests for edge cases and boundary conditions
- Test with Data Browser and other tools that use TimeRangePopover

**Resources:**
- GitHub Issue: https://github.com/ControlSystemStudio/phoebus/issues/3358

**Assigned To:** _Available_

---

### PHOEBUS-UI-002: Data Browser Archive Data Source Management

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Intermediate  
**Skills Required:** JavaFX, Data Browser Architecture, Archive Appliance Integration  

**Description:**  
Add UI functionality to manually add, remove, and modify archive data sources in the Data Browser. Currently, the Data Browser automatically removes inaccessible data sources (good for preventing unnecessary calls), but there's no way to re-add them after an archiver recovers from a temporary outage. This feature existed in legacy CS-Studio and is needed for operational flexibility.
- Add context menu actions in Properties -> Traces -> Archive view:
  - "Add Archive Data Source" - add from configured preferences
  - "Remove Archive Data Source" - remove selected source
  - "Refresh Data Source" - retry connection to disabled source
- Show data source health status in archive view (active, disabled, error with timestamp)??
- Persist manual data source changes in Data Browser configuration files
- Update Data Browser to respect manually added sources even if initially unreachable??

**Resources:**
- Legacy CS-Studio archive preferences for reference

**Assigned To:** _Available_

---

### PHOEBUS-UI-003: Fix PV Resource Leak in WidgetRuntime

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Beginner  
**Skills Required:** Java, PV Access, Resource Management  

**Description:**  
Fix PV resource leak in Display Builder runtime when using non-standard default PV types (loc://, muscade://, tango://). PVs are not properly released when displays close, causing memory leaks.

- Fix PV release mechanism in WidgetRuntime to handle PV name prefix differences
- Ensure PVPool correctly tracks and releases PVs regardless of default datasource configuration
- Test with displays using loc://, ca://, pva://, and custom datasources

**Resources:**
- GitHub PR: https://github.com/ControlSystemStudio/phoebus/pull/3412

**Assigned To:** _Available_

---

### PHOEBUS-UI-004: Interactive Graph Widget for XYPlot

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Advanced  
**Skills Required:** JavaFX, XYPlot Architecture, Mouse Event Handling, PV Access  

**Description:**  
Add interactive editing capability to XYPlot graphs, allowing users to drag points directly on the plot to edit PV values. Implement as new "Edit" mouse mode to avoid interfering with existing graph functionality.

**Resources:**
- GitHub Issue: https://github.com/ControlSystemStudio/phoebus/issues/3167

**Assigned To:** _Available_

---

### PHOEBUS-UI-005: ChartFX vs RT Plot Review

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Intermediate  
**Skills Required:** JavaFX, Plotting, UI Architecture, Evaluation / Technical Assessment  

**Description:**  
Review the JavaFX plotting stack and decide whether ChartFX or the current RT plot is the better long-term fit for Phoebus.

- Compare compatibility, performance, and maintenance cost
- Check support and integration impact

**Resources:**
- ChartFX repo: https://github.com/fair-acc/chart-fx
- Plot library discussion: https://github.com/ControlSystemStudio/phoebus/issues/3362
- Related RT plot work: https://github.com/ControlSystemStudio/phoebus/issues/3167

**Assigned To:** _Available_

---

### PHOEBUS-UI-006: Remove Hardcoded Colors for Consistency

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Intermediate  
**Skills Required:** JavaFX, Java, FXML  

**Description:**  
Refactor hardcoded color values throughout Phoebus to use a central ColorService. Improve code maintainability and consistency by centralizing color management.

- Create or enhance central ColorService for managing application colors
- Ensure consistent color management across all applications
- Add documentation for ColorService usage patterns

**Resources:**
- WidgetColorService: `app/display/model/src/main/java/org/csstudio/display/builder/model/persist/WidgetColorService.java`
- Annunciator hardcoded colors: `app/alarm/ui/src/main/java/org/phoebus/applications/alarm/ui/annunciator/AnnunciatorTable.java#L92`
- Probe hardcoded colors: `app/probe/src/main/java/org/phoebus/applications/probe/view/ProbeController.java#L105`
- Save & Restore FXML styles:
  - `app/save-and-restore/app/src/main/resources/org/phoebus/applications/saveandrestore/ui/configuration/ConfigurationEditor.fxml#L11`
  - `app/save-and-restore/app/src/main/resources/org/phoebus/applications/saveandrestore/ui/snapshot/SnapshotView.fxml#L37`
- Logbook FXML styles: `app/logbook/olog/ui/src/main/resources/org/phoebus/logbook/olog/ui/LogEntryTableView.fxml#L43`
- Queue Server FXML styles: `app/queue-server/src/main/resources/org/phoebus/applications/queueserver/view/ReStatusMonitor.fxml#L13`
- NamedWidgetColors: `app/display/model/src/main/java/org/csstudio/display/builder/model/persist/NamedWidgetColors.java`

**Assigned To:** Urban Bobek (Cosylab)  
**Status:** Partially complete - [PR #3716](https://github.com/ControlSystemStudio/phoebus/pull/3716) merged 2026-03-30 (color service moved to `core/ui`). Hardcoded color cleanup remains.  
**Notes:** Started migrating the widget color service to `core/ui` and compiling a list of all places where colors are either hard-coded or implemented in a non-standard way. This project lays the groundwork for the possibility of having the option to theme Phoebus.

---

### PHOEBUS-DEV-001: Verify IntelliJ IDEA Setup Instructions

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Beginner  
**Skills Required:** IntelliJ IDEA  

**Description:**  
Verify and update the IntelliJ IDEA setup instructions in the README. Test import, build, and run steps to ensure they work with current IntelliJ versions.

- Follow README instructions for importing Phoebus into IntelliJ IDEA
- Verify Maven import and dependency resolution
- Test build process
- Test running Phoebus from IDE
- Update instructions if any steps are outdated or missing

**Resources:**
- Current instructions: https://github.com/ControlSystemStudio/phoebus#developing-with-intellij-idea

**Assigned To:** _Available_

---


### SERVICES-SB4-001: Spring Boot 4 Migration Planning

**Repository:** Multiple (Olog, ChannelFinder, Save & Restore)  
**Difficulty:** Advanced  
**Skills Required:** Java, Spring Boot 3/4, Migration Planning  

**Description:**  
Create comprehensive migration plan and proof-of-concept for upgrading Phoebus middle layer services from Spring Boot 3.x to Spring Boot 4.x, leveraging new features while ensuring backward compatibility.

- Audit current Spring Boot 3.x usage across all services (dependencies, configurations, custom code)
- Document breaking changes and deprecations in Spring Boot 4:
  - Spring Security 7 authorization architecture changes
  - RestTemplate deprecation (migrate to HTTP Interface clients)
  - Configuration property changes
  - Minimum Java version requirements (JDK 21+)
- Create migration checklist with service-specific considerations:
  - Update `pom.xml`/`build.gradle` dependencies
  - Migrate security configurations to Spring Security 7 patterns
  - Replace RestTemplate with declarative HTTP Interface clients
  - Update actuator endpoint configurations
- Implement pilot migration for one service (Save & Restore or Alarm Logger) as proof-of-concept
- Test virtual threads integration with `spring.threads.virtual.enabled=true`
- Leverage Problem Details (RFC 7807) for standardized error responses
- Enable OpenTelemetry observability with `management.tracing.enabled=true`
- Document rollback strategy and deployment considerations

**Resources:**
- Spring Boot 4 roadmap: https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Release-Notes
- Spring Boot 3.4 features: https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.4-Release-Notes
- Spring Security 7: https://docs.spring.io/spring-security/reference/7.0/migration/index.html
- HTTP Interface clients: https://docs.spring.io/spring-framework/reference/integration/rest-clients.html#rest-http-interface

**Assigned To:** _Available_

---

### SERVICES-VERSIONING-001: REST API Versioning Strategy

**Repository:** Multiple (Olog, ChannelFinder, Save & Restore, Alarm Services)  
**Difficulty:** Intermediate  
**Skills Required:** Java, Spring Boot, REST API Design, Versioning Patterns  

**Description:**  
Implement consistent REST API versioning across all Phoebus middle layer services to support backward compatibility, gradual feature rollout, and deprecation policies without breaking existing clients.

- Evaluate versioning strategies and select one approach:
  - URL path versioning: `/api/v1/logs`, `/api/v2/logs` (recommended for clarity)
  - Header-based versioning: `Accept: application/vnd.phoebus.v1+json`
  - Query parameter versioning: `/api/logs?version=1`
- Implement chosen strategy using Spring Boot features:
  - `@RequestMapping` with version prefix
  - Custom `RequestCondition` for header-based versioning
  - Version-specific controller classes or methods
- Create versioning policy document:
  - When to introduce new version (breaking changes only)
  - How long to support deprecated versions (e.g., N-2 policy)
  - Version deprecation headers (`Deprecation`, `Sunset`)
  - Migration guides for each version bump
- Implement version negotiation with default version fallback
- Add OpenAPI/Swagger documentation showing all supported versions
- Update existing endpoints to v1, prepare v2 structure for future changes
- Add integration tests covering multiple API versions simultaneously
- Create client migration examples (Python, Java, curl)

**Resources:**
- REST API Versioning Best Practices: https://restfulapi.net/versioning/
- Spring Boot REST versioning: https://www.baeldung.com/rest-versioning

**Assigned To:** _Available_

---

### SERVICES-WEBSOCKET-001: WebSocket Support as a alternative to Polling Phoebus Services

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Intermediate  
**Skills Required:** Java, WebSocket, JavaFX, REST APIs  

**Description:**  
Add WebSocket support to Phoebus clients for real-time updates from middle layer services, replacing polling-based architecture. Implement intelligent fallback to polling when connecting to older services without WebSocket support, ensuring backward compatibility.

- Implement WebSocket-based updates for improved performance:
  - Olog logbook updates (server already supports WebSocket)
  - Alarm Logger search results updates
  - Save & Restore service updates
  - Replace periodic polling with push-based real-time updates
- Add service capability detection:
  - Attempt WebSocket connection first
  - Auto-detect WebSocket availability (HTTP headers, discovery endpoint, connection attempt)
  - Fall back to polling if service doesn't support WebSocket yet
- Ensure backward compatibility:
  - Clients automatically use WebSocket when available
  - Clients work with older polling-only services

**Resources:**
- Olog WebSocket implementation: https://github.com/Olog/phoebus-olog

**Assigned To:** _Available_

---

### SERVICES-OLOG-001: Elasticsearch embeddings index mapping and template update for Olog

**Repository:** https://github.com/Olog/phoebus-olog  
**Difficulty:** Intermediate  
**Skills Required:** Elasticsearch, Java/Spring Boot, Index Mappings, DevOps  

**Description:**  
Update Olog's Elasticsearch mappings and templates to support vector embeddings for semantic search, while keeping the existing log search behavior working.

- Add a `dense_vector` field (or equivalent mapping) for embeddings
- Review index settings for dimensions, similarity, and indexing strategy
- Check the impact on storage, memory, and search performance
- Update the template and migration plan for existing Olog data
- Validate compatibility with current Olog queries and field names

**Resources:**
- Olog repository: https://github.com/Olog/phoebus-olog
- Elasticsearch dense vector mapping: https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/dense-vector
- Elasticsearch vector performance guidance: https://www.elastic.co/docs/deploy-manage/production-guidance/optimize-performance/approximate-knn-search

**Assigned To:** _Available_

---

### SERVICES-CFNS-001: Multi-threaded ChannelFinder Nameserver with Broadcast Fallback

**Repository:** https://github.com/ChannelFinder/cfNameserver  
**Difficulty:** Intermediate  
**Skills Required:** Java, Multi-threading, EPICS PVAccess, Network Programming  

**Description:**  
Enhance the ChannelFinder Nameserver (currently PVAccess-only) to support concurrent PV name resolution requests using multi-threading, and add intelligent fallback to broadcast name requests when PVs are not found in ChannelFinder database.

- Implement multi-threaded request handling for concurrent PV name lookups
- Implement PVAccess broadcast fallback mechanism:
  - First attempt: Query ChannelFinder service for PV information
  - If PV not found: Fall back to PVAccess broadcast name resolution
  - Cache broadcast results temporarily to reduce network overhead
- Add configuration options for:
  - Thread pool size, Fallback enable/disable toggle, Cache timeout for broadcast results, ChannelFinder query timeout
- Add performance metrics and logging for monitoring
- Add Channel Access protocol support alongside existing PVAccess (nameserver currently only supports PVAccess)

**Resources:**
- cfNameserver repository: https://github.com/ChannelFinder/cfNameserver

**Assigned To:** _Available_

---

### SERVICES-CF-DOC-001: Channel lifecycle documentation and diagram for the ChannelFinder ecosystem

**Repository:** Multi-repository documentation project (ChannelFinder, cfNameserver, recceiver, recsync)  
**Difficulty:** Beginner  
**Skills Required:** Documentation, Architecture, Diagramming, System Design  

**Description:**  
Document the lifecycle of PV/channel state across the ChannelFinder ecosystem, including how channels are created, updated, marked undefined or disabled, and removed or reset during startup/shutdown and restarts. This project captures the output of the ChannelFinder/RecSync lifecycle discussion and turns it into a system diagram and a readable reference.

- Produce a high-level diagram of the ChannelFinder ecosystem lifecycle which covers ChannelFinder, cfNameserver, recceiver, recsync, and recaster interactions
- Document start-up, normal operation, restart, undefined/disabled states, and shutdown transitions
- Clarify how lifecycle and property cleanup should behave across services
- Identify open architectural questions and unresolved edge cases
- Publish the diagram and explanation in the relevant docs or architecture notes

**Resources:**
- ChannelFinder service: https://github.com/ChannelFinder/ChannelFinderService
- cfNameserver: https://github.com/ChannelFinder/cfNameserver
- recceiver: https://github.com/ChannelFinder/recceiver
- recsync: https://github.com/ChannelFinder/recsync

**Assigned To:** _Available_

---

### SERVICES-RECSYNC-002: Restructure recsync repo and modernize Python packaging

**Repository:** https://github.com/ChannelFinder/recsync  
**Difficulty:** Beginner  
**Skills Required:** Python, Pixi, Package Management, Repo Restructuring  

**Description:**  
Complete the restructuring of the `recsync` repository so the architecture matches the current split between the IOC-side `reccaster` and the Python/Twisted server side. This work is about repository layout and packaging modernization, not replacing Twisted itself.

- Split out the Python/Twisted server component from the remaining `recsync` repo structure, following the same pattern as the `reccaster` extraction
- Create a clear project layout for the server package, shared utilities, and any remaining compatibility layers
- Modernize the Python packaging/build setup for the server component using tools such as Pixi where appropriate
- Generate `pixi.lock` for reproducible dependency resolution if the project adopts Pixi
- Define development and production environments in the Python packaging configuration
- Create common dev tasks for:
  - `pytest` or equivalent tests
  - linting/formatting
  - local server startup
- Update documentation with installation and usage instructions for the new repository layout
- Maintain backward compatibility with existing deployment methods where possible
- Track the repo split against the related issue: https://github.com/ChannelFinder/recsync/issues/174

**Note:** Pixi is for packaging and environment management; it does not replace the Twisted server or change the runtime architecture. It may be used as a build/dev tooling layer, but the server remains a Twisted application.

**Resources:**
- recsync repository: https://github.com/ChannelFinder/recsync
- recsync issue: https://github.com/ChannelFinder/recsync/issues/174
- RecCeiver/Twisted server area: https://github.com/ChannelFinder/recsync/tree/master/server
- Pixi documentation: https://pixi.sh/

**Assigned To:** _Available_

---

### SERVICES-RECSYNC-003: RecCeiver lifecycle/state cleanup

**Repository:** https://github.com/ChannelFinder/recceiver  
**Difficulty:** Beginner  
**Skills Required:** Java, Lifecycle Management, State Handling, Testing  

**Description:**  
Group the easy cleanup work in the new Java `RecCeiver` implementation around lifecycle and state management. This is a focused sub-project that covers the startup/shutdown and cleanup issues without expanding into broader architecture redesign.

- Implement the shutdown behavior for marking channels as `undefined` when the service stops
- Ensure state is reset cleanly between restarts to avoid stale or inconsistent runtime state
- Add/extend tests covering start/stop and restart behavior
- Validate that the lifecycle cleanup is idempotent and safe under repeated restarts
- Keep the work scoped to the current `RecCeiver` lifecycle issues rather than broader service redesign

**Related issues:**
- https://github.com/ChannelFinder/recceiver/issues/1
- https://github.com/ChannelFinder/recceiver/issues/2
- https://github.com/ChannelFinder/recceiver/issues/5

**Resources:**
- recceiver repository: https://github.com/ChannelFinder/recceiver
- Issue tracker: https://github.com/ChannelFinder/recceiver/issues

**Assigned To:** _Available_

---

### SERVICES-RECSYNC-004: Direct PVA-RPC from RecCaster to ChannelFinder

**Repository:** https://github.com/ChannelFinder/recsync  
**Difficulty:** Advanced  
**Skills Required:** C++, Java, EPICS PVAccess (PVA-RPC), Network Programming  

**Description:**  
Explore and implement a direct communication path between the EPICS IOC component (`reccaster`) and the `ChannelFinder` service using `pva-rpc`. This approach simplifies the architecture by removing the need for an intermediate `recceiver` (or `recsync` server) process.

- Implement `pva-rpc` support within the `reccaster` component (typically C/C++)
- Define `pva-rpc` service endpoints on the `ChannelFinder` service side to receive and process record data
- Ensure data integrity and efficient transfer of large record sets from the IOC to the directory service
- Support secure communication via PVAccess security mechanisms
- Evaluate performance, scalability, and simplified deployment benefits compared to the traditional `recsync` architecture

**Resources:**
- recsync repository: https://github.com/ChannelFinder/recsync
- ChannelFinder Service: https://github.com/ChannelFinder/ChannelFinderService
- PVAccess (PVA-RPC) documentation: https://epics-base.github.io/pvDataCPP/pva_rpc.html

**Assigned To:** _Available_

---

### PHOEBUS-LINT-001: Display Builder Screen Linter

**Repository:** New phoebus-display-linter or add to Phoebus build tools  
**Difficulty:** Beginner  
**Skills Required:** Python/Java, XML parsing, Display Builder format knowledge  

**Description:**  
Create a command-line linter for Display Builder (.bob) and OPI screens to detect common issues and enforce facility-specific conventions.

- Implement core validation: XML schema, required properties, valid values, PV name format, widget overlaps
- Design flexible configuration system (YAML/JSON) for facility-specific rules: color palettes, naming conventions, fonts, widget limits, embedding depth
- Support both .bob and .opi formats
- Generate CI/CD-friendly reports (console, JSON output, non-zero exit codes)
- Key challenge: Make conventions easily configurable per facility without code changes

**Assigned To:** _Available_

---

### AI-DISPLAY-001: LLM-Assisted Display Screen Generation

**Repository:** https://github.com/ControlSystemStudio/phoebus  
**Difficulty:** Advanced  
**Skills Required:** Python/Java, LLM APIs (OpenAI/Anthropic/local models), Display Builder, Prompt Engineering  

**Description:**  
Explore using Large Language Models (LLMs) to generate Phoebus Display Builder screens (.bob files) from natural language descriptions or existing OPI/EDL files, accelerating screen development and migration from legacy CS-Studio.

- Research Display Builder XML/BOB file format structure:
  - Widget types (Label, TextUpdate, ActionButton, LED, Rectangle, Group, etc.)
  - Widget properties (position, size, PV names, colors, fonts)
  - Layout patterns and parent-child relationships
  - Rules, scripts, and actions
- Create prompt templates for LLM-based screen generation:
  - "Create a motor control screen with position readback, setpoint, start/stop buttons"
  - "Convert this EDM/EDL screen to Display Builder format"
  - "Generate a vacuum system overview with 12 gauge indicators arranged in a grid"
- Implement generation pipeline:
  - LLM API integration (OpenAI GPT-4, Claude, or local Llama models)
  - Prompt engineering with few-shot examples of BOB XML
  - XML validation and widget property verification
  - Post-processing to fix common LLM errors (invalid colors, missing required properties)
- Test with various complexity levels:
  - Simple screens (5-10 widgets)
  - Complex screens (100+ widgets, embedded displays, macros)
- Document findings: success rates, limitations, best practices, cost analysis

**Assigned To:** _Available_

---

### AI-DISPLAY-002: LLM-Based Display Screen CI/CD Validator

**Repository:** New GitHub Actions / CI tools repository  
**Difficulty:** Intermediate  
**Skills Required:** Python, GitHub Actions, LLM APIs, Display Builder, XML parsing  

**Description:**  
Develop an LLM-powered automated validator for Display Builder screens (.bob files) that integrates into CI/CD pipelines to enforce design conventions, detect errors, and ensure quality standards before merging to production.

- Implement validation engine with multiple checks:
  - Static analysis: Schema validation, property ranges, XML correctness
  - Convention enforcement: Naming patterns, layout standards, color palette, fonts
  - LLM semantic analysis: "Does layout make sense?", "Are PV names consistent?", "Are labels clear?"
  - Security: No hardcoded credentials, valid PV patterns
  - Performance: Widget count limits, embedding depth
  - Accessibility: Contrast ratios, text sizes
- Create GitHub Actions workflow:
  - Trigger on PRs with `.bob` file changes
  - Post detailed reports and auto-fix suggestions as PR comments
  - Configurable severity levels (error/warning/info)
- Build configuration system:
  - YAML-based facility-specific rules
  - Custom validation patterns and LLM prompts
  - Exemption mechanisms with justification
- Generate reports:
  - Markdown PR summaries with line numbers
  - JSON output for programmatic use
  - Metrics dashboard showing violation trends
- Support multiple platforms: GitHub Actions, GitLab CI/CD, Jenkins, pre-commit hooks

**Assigned To:** _Available_

---

### AI-ASSIST-001: LLM-Powered Phoebus Development Assistant

**Repository:** New AI tools repository  
**Difficulty:** Intermediate  
**Skills Required:** Python, LLM APIs, RAG (Retrieval Augmented Generation), Vector Databases  

**Description:**  
Build an AI assistant that helps Phoebus developers by answering questions about the codebase, suggesting code patterns, and providing context-aware documentation using Retrieval Augmented Generation (RAG).

- Create knowledge base from Phoebus ecosystem:
  - Index Phoebus source code, JavaDoc, markdown documentation
  - Index Display Builder widget API documentation
  - Index common code patterns (service implementations, widget examples)
  - Index GitHub issues, discussions, and wiki pages
- Explore RAG pipeline:
  - Document chunking and embedding (OpenAI embeddings or open-source alternatives)
  - Semantic search for relevant context retrieval
  - LLM integration (GPT-4, Claude, or local models like CodeLlama)
  - Prompt construction with retrieved context
- Support use cases:
  - "How do I create a custom widget in Display Builder?"
  - "What's the difference between AlarmTree and AlarmTable?"
- Implement features:
  - Citation links back to source code/docs
  - Context window management for long conversations
- Evaluate usefullness:
  - Test on 20-30 common developer questions
  - Measure answer accuracy and relevance
  - Compare to manual documentation search time
  - Collect user feedback on usefulness

**Assigned To:** _Available_

---

### AI-LOG-001: Intelligent Log Analysis for Alarm Logs

**Repository:** New AI/ML analysis tools  
**Difficulty:** Intermediate  
**Skills Required:** Java/Python, Log Parsing, Pattern Recognition  

**Description:**  
Apply natural language processing and pattern recognition to alarm service logs to automatically identify recurring issues, extract failure patterns, and provide actionable insights for control system behavior.

**Assigned To:** _Available_

---

### DEVOPS-ANSIBLE-001: Phoebus Infrastructure Deployment

**Repository:** https://github.com/ControlSystemStudio/ansible-phoebus  
**Difficulty:** Intermediate  
**Skills Required:** Ansible, Linux System Administration, Docker/Podman  

**Description:**  
Create a comprehensive Ansible collection with roles and playbooks for deploying the complete Phoebus tools and services technology stack. The collection should be generic and configurable enough for different facilities to adopt with minimal modifications.

- Design modular Ansible roles for each service: Phoebus client, Olog, ChannelFinder, Save & Restore, Alarm Server, Alarm Logger, Archiver Appliance
- Create roles for infrastructure dependencies: Elasticsearch, PostgreSQL, MongoDB, Kafka (KRaft mode)
- Implement role variables with sensible defaults and facility-specific overrides (ports, paths, credentials, heap sizes)
- Support multiple deployment targets: bare metal, containers (Docker/Podman), systemd services
- Include SSL/TLS certificate configuration and secure credential management (Ansible Vault)
- Add playbooks for common operations: deployment, updates, backups, health checks, scaling
- Create comprehensive documentation with example inventory files and variable configurations
- Package as Ansible Galaxy collection for easy distribution and installation

**Resources:**
- Ansible best practices: https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html
- Ansible Galaxy collections: https://docs.ansible.com/ansible/latest/collections_guide/index.html
- Service repositories: https://github.com/ControlSystemStudio/
- Molecule testing: https://ansible.readthedocs.io/projects/molecule/

**Assigned To:** _Available_

---

### DEVOPS-ANSIBLE-002: Test and Validation of ansible-phoebus Roles

**Repository:** https://github.com/ControlSystemStudio/ansible-phoebus  
**Difficulty:** Intermediate  
**Skills Required:** Ansible, Molecule, CI/CD, Linux System Administration  

**Description:**  
Create the test and validation framework for the `ansible-phoebus` deployment roles so the shared configuration can be trusted across facilities. The goal is to make the roles repeatable, safe to evolve, and easier to adopt in production environments.

- Add Molecule or equivalent tests for the existing Ansible roles
- Validate idempotency and rollback behavior for common deployment flows
- Add linting and syntax checks for roles and playbooks
- Define a minimal CI workflow for PR validation and release testing
- Document expected role inputs, outputs, and failure modes

**Resources:**
- ansible-phoebus repository: https://github.com/ControlSystemStudio/ansible-phoebus
- Molecule testing: https://ansible.readthedocs.io/projects/molecule/
- Ansible best practices: https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html

**Assigned To:** _Available_

---

### DEVOPS-CONTAINER-001: Improve Dockerfiles and Compose for Phoebus Services

**Repository:** Multiple (Phoebus, Olog, ChannelFinder, Save & Restore, Alarm Services)  
**Difficulty:** Intermediate  
**Skills Required:** Docker/Podman, Compose, CI/CD, Service Configuration  

**Description:**  
Review and update the Dockerfiles and Compose files used for Phoebus and the middle-layer services so they are easier to build, run, and maintain across environments.

- Standardize Dockerfiles for Phoebus and the service stack
- Clean up Compose configuration for local development and testing
- Make service startup and dependency wiring more reliable
- Improve image build consistency and tag/version handling
- Document the expected local and CI workflows for container usage
- Update configuration defaults for common deployment scenarios

**Resources:**
- Docker: https://docs.docker.com/
- Docker Compose: https://docs.docker.com/compose/
- Phoebus service repositories: https://github.com/ControlSystemStudio/

**Assigned To:** _Available_

---

### DEVOPS-VM-001: EPICS Services Training VM

**Repository:** New epics-services-training-vm  
**Difficulty:** Intermediate  
**Skills Required:** Vagrant/VirtualBox, Linux, Ansible, EPICS  

**Description:**  
Create a training VM for Phoebus services and tools that complements the existing EPICS training-vm. The VM should provide a pre-configured environment with all Phoebus middle layer services running, suitable for workshops, tutorials, and onboarding new users.

- Build VM based on Vagrant/VirtualBox similar to epics-training-vm structure
- Leverage Ansible roles/playbooks from DEVOPS-ANSIBLE-001 for service deployment
- Include pre-configured services with sample data:
  - Olog (with example log entries)
  - ChannelFinder (with sample PV directory)
  - Save & Restore (with example configurations)
  - Alarm Server & Logger (with example alarm configuration)
  - Archiver Appliance (extract and adapt from existing training VM if present)
- Include Phoebus client with pre-configured service connections
- Add demo IOCs generating test PVs for training exercises
- Create getting-started guide with training exercises:
- Optimize VM size and resource usage for laptop deployment
- Provide scripts for resetting VM to clean state between training sessions
- Document customization for facility-specific training needs

**Resources:**
- EPICS training-vm: https://github.com/epics-training/training-vm
- Ansible roles from DEVOPS-ANSIBLE-001
- Vagrant documentation: https://www.vagrantup.com/docs

**Assigned To:** _Available_  
**Related Work:** Ralph Lange (ITER) and Simon Rose (ESS) worked on splitting the CI setup for the existing training-vm, and Ralph Lange updated oac-tree in preparation for the April 2026 Saclay training.

---

## Getting Help

- Ask in the main room during work hours
- Use the Matrix chat: [#codeathon-oct26:epics-controls.org](https://matrix.to/#/#codeathon-oct26:epics-controls.org)
- Tag mentors: @Kunal Shroff, @Georg Weiss, @Sky Brewer
