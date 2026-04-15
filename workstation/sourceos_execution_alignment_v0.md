# SourceOS Execution Alignment for Workstation Profile v0

This note binds the workstation profile work to the broader SourceOS execution fabric so the desktop/DevEx layer does not drift from the system’s audit, provenance, and zero-trust model.

## Why this note exists

The workstation profile is not just a UI/theme/package exercise.
It must remain compatible with the SourceOS execution model built around:

- Run Capsule
- Agent Contract
- Vercel gateway / Porter core split
- Quilt-backed auditability and immutable artifact records
- graph-backed reasoning and provenance
- local-first/offline-capable operation
- zero-trust capability boundaries

## Alignment rules

### 1. Desktop affordances are not a second control plane

Albert, GNOME shortcuts, Kinto mappings, Fusuma gestures, and shell tools are treated as presentation and invocation surfaces.

They are allowed to:
- trigger actions
- display status
- route users into workflows

They are not allowed to redefine:
- capability policy
- audit requirements
- provenance rules
- execution authorization semantics

Those remain owned by the canonical execution contracts.

### 2. Workstation profile must map cleanly onto Run Capsule / Agent Contract

Any user-facing “agent action” surfaced through launcher, shell, or desktop must eventually resolve to execution semantics that can be represented by:

- an Agent Contract (what tools/policies/profiles exist)
- a Run Capsule (what actually happened)

This means workstation work should prefer declarative action descriptions and avoid opaque one-off scripts with no audit mapping.

### 3. Package/install layers must preserve trust boundaries

The workstation stack is layered:

- SYSTEM: minimal immutable host dependencies
- USER: CLI/dev tooling and profile config
- TOOLBOX: isolated AUR/source build surface

This layering exists partly for operability, but also for security.
Risky or fast-moving dependencies should not silently collapse into the host trust boundary.

### 4. Local-first behavior must remain first-class

Homebase/local caching and offline continuity imply that workstation tooling should:

- function acceptably when partially disconnected
- queue/reconcile where possible
- avoid assuming always-on central services for basic operator workflows

### 5. Real-time UX must connect to real-time execution telemetry

The SourceOS execution fabric still needs stronger streaming support (SSE/WebSockets) for live event visibility.
The workstation profile should therefore reserve room for:

- live status surfaces
- event streams in launcher/CLI/UI
- incremental progress feedback

without inventing a separate event model.

## Immediate design consequences

1. The workstation profile RFC should stay in `SociOS-Linux/enhancements`.
2. Typed forms for `RunCapsule`, `AgentContract`, `WorkstationProfile`, and related launcher/profile objects should be formalized in `SourceOS-Linux/sourceos-spec` once fields stabilize.
3. Build/config/install repos should consume these contracts, not redefine them ad hoc.

## Repo topology implication

### `SociOS-Linux`
Implementation and build/config/install surfaces:
- `enhancements`
- `socios-coreos-config`
- `socios-fedora-builder`
- `socios-fedora-usb`
- `socios-scripts`
- `homebrew-socios`
- `albert` and adjacent desktop/launcher repos

### `SourceOS-Linux`
Canonical contracts/specification surfaces:
- `sourceos-spec`

## Open questions

- Which workstation actions are purely local UX helpers versus auditable agent actions?
- Which launcher actions need explicit typed capability references?
- How much of the workstation profile belongs in contracts versus implementation overlays?
- What is the minimal schema set needed before we start wiring build/install repos directly to typed workstation manifests?
