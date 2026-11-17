# InfiniScribe agent guidance

InfiniScribe is Infiniforge's standalone document viewer. Its intended scope is broad document viewing, while the application is still incomplete and format coverage varies. It owns its viewer experience and build; it must not require Infinifolder or another Infiniforge app to launch or build.

## Independence and integration

- Build this repository from its own checkout with its pinned library dependencies and documented Android toolchain. Do not depend on sibling application source, build outputs, or workspace-level paths.
- Keep this app's Gradle build independent. Viewer modules inside this repository are appropriate; another application is not a Gradle module or source dependency.
- Use Android SAF and granted content URIs to open documents. Use Infinifolder's storage interfaces when available, while preserving usable local/SAF workflows when it is absent.
- Keep reusable parsing and format support in linked library APIs where appropriate. Keep document-viewer UI, navigation, and app-owned state in this application.
- Preserve URI grants and source identity across handoffs. Do not read another application's private files or persist temporary access links as document identity.

## Project focus

- Treat broad document viewing as the direction of the project, not a promise of complete format coverage. Document implemented formats, partial support, and unsupported behavior accurately.
- Keep viewer modules isolated enough to build and evolve within this app, with shared contracts in this repository or reusable libraries where justified.
- Keep standalone build instructions and pinned library revisions current.

## Ecosystem workflow for agents

1. Read `docs/ECOSYSTEM.md` first to understand the component map and ownership boundaries.
2. Read `docs/APPLICATION.md` for this app's intended role and current status. Before changing a public interface, read the repository's architecture and API documentation listed below and inspect compatibility notes.
3. Check the owning shared library repository before implementing behavior that may already exist elsewhere. Do not conclude a capability is unavailable just because its implementation is outside the current repository.
4. Keep application responsibilities, shared library APIs, and other applications' private data/source separate.
5. Before a cross-repository change, check the exact dependency pin and the relevant interoperability contract. Dependency versions and protocol versions are separate; do not infer protocol incompatibility from a library-version mismatch.
6. Update the local documentation index and relevant API/behavior documentation when public behavior changes. `docs/ECOSYSTEM.md` is generated from the central manifest and must not be edited locally.

## Documentation index

- `README.md` — project purpose, prerequisites, and standalone build instructions.
- `AGENTS.md` — repository-specific development rules and this ecosystem workflow.
- `docs/README.md` — local documentation index.
- `docs/APPLICATION.md` — application purpose, responsibilities, implementation status, planned work, and known limits.
- `docs/ECOSYSTEM.md` — generated ecosystem-wide inventory, boundaries, dependencies, versions, and protocols.
- `docs/architecture.md` — public API and architecture documentation; read before changing interfaces or behavior.
- `docs/storage.md` — public API and architecture documentation; read before changing interfaces or behavior.
- `docs/text-and-document-formats.md` — public API and architecture documentation; read before changing interfaces or behavior.
- `docs/document-and-ebook-readers.md` — public API and architecture documentation; read before changing interfaces or behavior.