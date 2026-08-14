## ADDED Requirements

### Requirement: Evidence-based project and component classification
The full-package Skill SHALL inspect any optional project preselection, the enabled-project catalog, original filename, relative path when available, file extension and magic, and archive member names to produce ranked project/component candidates with confidence and human-readable evidence. An optional project preselection SHALL be treated as evidence rather than as final authority. A single uploaded archive MAY map to multiple components, and an unrecognized file MUST remain explicitly unclassified rather than be silently assigned.

#### Scenario: No project was selected in the UI
- **WHEN** component files are uploaded without a project preselection
- **THEN** the classifier ranks matching enabled projects and the human selects or enters the authoritative project in the mandatory clarification

#### Scenario: Protocol-stack archive contains several components
- **WHEN** an uploaded archive contains CUCP, CUUP, and DU payload filenames
- **THEN** the classifier proposes all three component mappings for that one source file and records the matching archive-member evidence

#### Scenario: Filename and archive contents disagree
- **WHEN** filename rules suggest one component but archive-member rules strongly identify another
- **THEN** both candidates and their evidence are surfaced for human choice rather than silently resolving the conflict

#### Scenario: RAR component archive is uploaded
- **WHEN** a valid RAR archive is uploaded and a supported RAR inspection backend is available
- **THEN** its member names participate in classification under the same safety and size limits as ZIP and TAR archives

### Requirement: Mandatory packaging clarification gate
Every full-package build SHALL use the existing clarification request/resolution protocol before any package bytes are built or uploaded. This gate SHALL be server-enforced, SHALL NOT depend on the model choosing to ask, and SHALL override a user preference that disables ordinary optional clarification.

#### Scenario: Packaging is requested with clarification disabled
- **WHEN** a user whose ordinary clarification preference is disabled uploads component files for packaging
- **THEN** the Configuration Manager still emits a clarification card and waits for the human response before continuing

#### Scenario: Model attempts to build without asking
- **WHEN** the model calls a package-build tool without a valid confirmed plan
- **THEN** the tool rejects the call and neither constructs nor uploads a package

### Requirement: Confirmation covers project and every uploaded file
The mandatory clarification SHALL include the target project and a separate mapping question for every uploaded file. Each file question SHALL expose the classifier’s candidate component mapping, including multi-component mappings where applicable, and SHALL allow the human to explicitly exclude the file. No file SHALL be omitted from the confirmation payload.

#### Scenario: Thirteen files are uploaded
- **WHEN** a turn uploads thirteen files
- **THEN** the clarification contains one target-project confirmation and thirteen file-to-component confirmation questions, plus any package metadata questions required by configuration

#### Scenario: Human excludes an alternative build artifact
- **WHEN** the human selects “不纳入整包” for an uploaded file
- **THEN** that exclusion is recorded as an explicit confirmed mapping decision and the file is not copied into the package

### Requirement: Confirmation completeness and binding
The system SHALL require an answer for the target project and every file mapping. A confirmed plan SHALL be bound to the selected project, session, uploader, file paths, sizes, and SHA-256 hashes; adding, replacing, or modifying any input SHALL invalidate the plan and require a new clarification.

#### Scenario: A mapping answer is missing
- **WHEN** the clarification resolution omits the answer for any uploaded file
- **THEN** packaging remains blocked and the system reports which confirmation is missing

#### Scenario: Input changes after confirmation
- **WHEN** a confirmed file is replaced or its hash no longer matches
- **THEN** the build tool rejects the stale plan and requires the complete confirmation flow again

#### Scenario: Clarification is cancelled or times out
- **WHEN** the human cancels the clarification or does not answer before the mandatory timeout
- **THEN** the run ends without building or uploading any package
