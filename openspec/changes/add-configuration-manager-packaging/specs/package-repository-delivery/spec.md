## ADDED Requirements

### Requirement: Atomic repository publication
After artifact validation, the package builder SHALL atomically copy the whole package into the existing Raven reconstruction-package storage and persist a package record containing project code, version, patch flag, components, size, SHA-256, build manifest, and confirmation reference. A failed copy or metadata write SHALL not leave a discoverable partial package.

#### Scenario: Publication succeeds
- **WHEN** a validated whole package is published
- **THEN** it appears in the existing reconstruction-package list and package-search tools under the confirmed project

#### Scenario: Metadata persistence fails
- **WHEN** the repository metadata cannot be saved
- **THEN** the newly copied artifact is rolled back or quarantined and is not returned as a successful package

### Requirement: Conversation download delivery
The Configuration Manager SHALL append a stable, clickable download link for the published artifact to the terminal conversation answer and SHALL include structured package artifact data in the terminal SSE event, independent of whether the model follows its preferred final-answer schema.

#### Scenario: Build completes normally
- **WHEN** repository publication returns a package ID
- **THEN** the conversation shows the package name, size, SHA-256 summary, repository ID, and a clickable `/raven/api/download/{id}` link

#### Scenario: Model answer omits the link
- **WHEN** the model’s final text does not mention the package artifact
- **THEN** the service deterministically adds the download section from the persisted build result before saving and streaming the assistant message

### Requirement: Download compatibility
The generated link SHALL use the existing Raven package download route and authorization policy, preserve non-ASCII filenames, and return the exact bytes whose SHA-256 is stored in repository metadata.

#### Scenario: User clicks the conversation link
- **WHEN** the user activates the whole-package download link from the chat window
- **THEN** the browser downloads the published `.tgz` with the recorded filename and matching SHA-256
