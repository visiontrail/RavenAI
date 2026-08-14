## ADDED Requirements

### Requirement: Configuration Manager identity
The system SHALL present the former `package_search` Agent as “配置管理员” in Chinese and “Configuration Manager” in English across the conversation selector, prompts, status text, accessibility labels, and completion summaries while preserving the stable internal `package_search` key for compatibility.

#### Scenario: User opens the Agent selector
- **WHEN** the conversation Agent selector is rendered in Chinese
- **THEN** the package-search entry is labelled “配置管理员” and no user-facing “检索重构包” label remains

### Requirement: Configuration Manager Skills loading
The Configuration Manager SHALL be a supported Skills host and SHALL materialize enabled built-in, Agent-level, and selected-project Skills into its isolated workspace using the same project setting source used by Project Expert and Log Analysis. Project Skills SHALL override an equally named Agent or built-in Skill, and Agent Skills SHALL override an equally named built-in Skill.

#### Scenario: Configuration Manager starts with enabled Skills
- **WHEN** a Configuration Manager run starts for a project that has enabled Skills
- **THEN** all applicable Skills are materialized, exposed through the SDK Skill tool, advertised in the prompt, and reported in trace metadata

#### Scenario: Project overrides the built-in packager
- **WHEN** a project enables a Skill with the same name as the built-in full-package Skill
- **THEN** the project Skill and its JSON rules are loaded in place of the lower-precedence definition for that run only

### Requirement: Repository-optional project binding
The Configuration Manager SHALL allow selection of any enabled project on which `package_search` is enabled, including projects without a Git repository, because package construction is bound to the project catalog and package repository rather than to source-code availability.

#### Scenario: Project has no Git URL
- **WHEN** an enabled repository-less project enables the Configuration Manager Agent
- **THEN** it appears in the project selector and a Configuration Manager workspace can be created without attempting a Git clone

#### Scenario: Packaging starts without a project preselection
- **WHEN** a new Configuration Manager turn includes component files but no project selection
- **THEN** the system creates an unbound workspace, ranks enabled project candidates from the Skill catalog, and defers authoritative project binding until the mandatory clarification is answered

### Requirement: Multi-file component attachments
The Configuration Manager conversation SHALL accept one or more component files in a single turn, preserve their original filenames, store them in the isolated session workspace with collision-safe names, and retain a hash-bound manifest for follow-up turns.

#### Scenario: User uploads multiple component files
- **WHEN** the user selects multiple component or archive files and sends a Configuration Manager message
- **THEN** every file is uploaded, persisted in the session workspace, listed in the user message UI, and included in the packaging manifest

#### Scenario: Upload exceeds a safety limit
- **WHEN** an individual or aggregate component upload exceeds configured size or disk-reserve limits
- **THEN** the request is rejected without starting an Agent run or leaving partial staged files
