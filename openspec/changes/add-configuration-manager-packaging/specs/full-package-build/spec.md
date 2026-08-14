## ADDED Requirements

### Requirement: JSON-driven package catalog
The full-package Skill SHALL load project, component, recognition, extraction, output-name, file-attribute, version, and package-name rules from a versioned JSON catalog. Adding a project or component SHALL not require changing classifier or builder code when the existing rule vocabulary is sufficient.

#### Scenario: Administrator adds a component in JSON
- **WHEN** a valid component definition using supported rule types is added through an overriding Agent or project Skill
- **THEN** the Configuration Manager can classify, confirm, and package that component on its next run without application restart or source changes

#### Scenario: Catalog is invalid
- **WHEN** required project or component fields are missing, duplicated, or unsafe
- **THEN** the Skill reports a configuration error before clarification or package construction

### Requirement: Confirmed-plan-only deterministic build
The package builder SHALL consume only a complete hash-bound confirmed plan, safely materialize each confirmed component according to its JSON rule, generate `si.ini`, and create a `.tgz` whole-package artifact with deterministic component ordering and collision-free output names.

#### Scenario: One archive supplies multiple components
- **WHEN** a confirmed source archive maps to CUCP, CUUP, and DU
- **THEN** the archive is extracted once and the configured payload for each component is copied and renamed into the whole-package workspace

#### Scenario: Direct-include component is selected
- **WHEN** a confirmed component is configured for direct inclusion
- **THEN** its original archive bytes are copied under the configured whole-package filename without unpacking and repacking the component

### Requirement: Safe multi-format archive handling
The builder SHALL support ZIP, TAR, TAR.GZ/TGZ, 7z, and RAR inputs through the existing workspace archive backends, reject traversal paths and unsafe links, enforce extracted-byte and file-count limits, and clean temporary extraction output after completion or failure.

#### Scenario: Archive contains path traversal
- **WHEN** an archive member would escape the extraction workspace
- **THEN** the build fails before writing outside the workspace and no repository record is created

#### Scenario: Primary RAR backend fails
- **WHEN** the primary RAR reader cannot decode a valid archive
- **THEN** configured `unar` and `bsdtar` fallbacks are attempted under the same validation limits

### Requirement: Manifest and artifact validation
The builder SHALL generate package metadata including project, package version, patch flag, component names and versions, input hashes, output hash, and confirmation reference. Before publication it SHALL reopen the artifact and verify `si.ini`, declared file count, configured filenames, and non-empty payloads.

#### Scenario: Output is complete
- **WHEN** all confirmed components are successfully materialized
- **THEN** artifact validation passes and the produced manifest and `si.ini` describe exactly the included files

#### Scenario: Component payload cannot be found
- **WHEN** an extraction rule cannot find the confirmed component payload
- **THEN** the build fails with the source file and component name, leaves no partial artifact in the repository, and preserves diagnostic evidence for the conversation

### Requirement: LX10 fixture compatibility
The default catalog SHALL cover the current RavenClient Lingxi-10 component model and SHALL classify the non-metadata files under `Temp/LX10-V1.0.0.3`, including multi-component protocol-stack archives and explicit exclusion of alternative/unusable artifacts.

#### Scenario: LX10 fixture is packaged
- **WHEN** the LX10 fixture files are classified, every mapping is human-confirmed, and version `1.0.0.3` is selected
- **THEN** a valid Lingxi-10 `.tgz` whole package is produced with the confirmed subset and correct `si.ini` attributes
