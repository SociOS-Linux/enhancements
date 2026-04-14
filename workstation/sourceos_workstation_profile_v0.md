- Feature Name: sourceos_workstation_profile_v0

# Summary

Define a reproducible, CLI-first “SourceOS Workstation” profile for CoreOS/Silverblue-derived systems, targeting GNOME (Wayland) with a macOS-familiar interaction grammar, a modern terminal toolchain, and an agent-ready launcher surface (Albert) without forking GNOME core components.

This enhancement standardizes:

- A DevEx CLI tool bundle (fzf/atuin/bat/zoxide/yazi/eza/gum/direnv/etc.) as a manifest (not tribal knowledge).
- A GNOME workstation baseline aligned to macOS muscle-memory via Kinto + gestures via Fusuma.
- An Albert integration model that is “actions-first” (command palette + agent invocation) and delegates file indexing to GNOME’s LocalSearch/Tracker substrate.
- A layered packaging strategy compatible with immutable hosts: SYSTEM (rpm-ostree), USER (Linuxbrew/Homebrew + Nix), TOOLBOX (AUR bridge/container builds).

# Motivation

We want developers to receive a first-class keyboard-first workstation experience on GNOME that:

1) Installs predictably and can be validated (“doctor”) and rolled forward/back.
2) Avoids fragile theming forks and avoids patching GNOME core unnecessarily.
3) Preserves local-first safety boundaries and permission-gated execution surfaces.
4) Works across a spectrum of deployment targets: Fedora CoreOS/Silverblue derivatives, and Fedora-on-Apple hardware images.

Expected outcome: a workstation profile that can be applied as an OS build-time preset and/or post-install “profile apply”, while remaining auditable, reproducible, and policy-checkable.

# Explanation and Examples

## Named concepts

### Workstation Profile
A versioned, installable bundle combining:

- Package sets (SYSTEM/USER/TOOLBOX)
- Configuration overlays (GNOME + shell + terminal)
- Input remapping defaults (Kinto + Fusuma)
- Launcher/action surface integrations (Albert)
- Validation probes (doctor)

### Package Layers

- SYSTEM: minimal host dependencies required for safe base operation.
  - Examples: podman/toolbox, git/ssh, gsettings/dconf tooling, wl-clipboard
- USER: developer toolchain and UX tools, installed into user scope.
  - Prefer Linuxbrew/Homebrew for parity + Nix/direnv for per-repo toolchains.
- TOOLBOX: “dirty build space” for AUR and source builds, isolated from host.
  - Used to bridge AUR on immutable hosts.

### Launcher as Action Bus
Albert is treated as:

- A command palette and action router
- An agent invocation surface

NOT as a duplicate filesystem index.

## Workstation Profile v0 behavior

### Keyboard-first CLI defaults

- Ctrl-R: atuin history search
- Ctrl-T: fzf file chooser (fd-backed)
- Alt-C: fzf directory jump (fd-backed)
- Ctrl-G: lazygit (repo-local)
- Ctrl-P: project/session picker (sesh/tmux)

### Terminal “clipboard abstraction”

Provide a stable `clipcopy` / `clippaste` abstraction that resolves to:

- macOS: pbcopy/pbpaste
- Wayland: wl-copy/wl-paste
- X11 fallback: xclip

### File motion primitives

“Move files around” is a first-class workstream:

- rclone (sync)
- mc (MinIO client; mirror/replicate)
- rsync (local movement)

### Albert integration

Albert should provide:

- App + action search
- “Agent verbs” (e.g. `sourceos:doctor`, `sourceos:apply-profile`, `sourceos:open-project`, `sourceos:k9s`, etc.)

Where file search is needed, it should call into GNOME’s local-search stack.

### GNOME baseline (non-fork policy)

We treat GNOME Shell/Mutter/libadwaita as immovable.
We customize via:

- GNOME Shell extensions (behavioral)
- GSettings/dconf defaults
- Fonts/icons/themes only where compatible with libadwaita constraints

We do not fork Mutter or GNOME Shell to achieve “macOS look.”

## Example: applying the profile (conceptual)

- `sourceos profile apply workstation-v0`
  - installs USER tool bundle
  - configures shell defaults
  - enables GNOME keybindings
  - registers Albert actions

- `sourceos doctor`
  - validates required tools are present
  - validates keybinding collisions
  - validates portal/permissions status

# Drawbacks

- Maintaining a stable workstation profile on GNOME requires discipline against over-theming and against patching GNOME core.
- Packaging parity across rpm-ostree + brew + nix increases complexity.
- AUR bridge adds operational surface area (toolbox/container isolation, signature discipline).

# Rationale and alternatives

## Why this design is best

- Immutable host patterns demand SYSTEM minimalism; developer UX belongs in userland where rollback is cheap.
- GNOME has a stable permission broker (xdg-desktop-portal) that supports agentic boundaries.
- Launcher as action bus provides a single keyboard-first surface without re-building a full desktop environment.

## Alternatives

- Fork GNOME Shell/Mutter to hard-code behaviors.
  - Rejected: update fragility, security surface growth.

- Use KDE Plasma as base.
  - Rejected for v0: larger divergence from GNOME ecosystem; we borrow patterns instead.

- Use NixOS as base.
  - Deferred: Nix is used as a reproducibility substrate, but we avoid forcing NixOS on the host until ops standards are locked.

# Implementation plan (repo topology alignment)

This enhancement is primarily a design contract. Implementation is split across existing upstream repos.

## Design / RFC home

- SociOS-Linux/enhancements
  - This document and follow-ons.

## Typed contracts home

- SourceOS-Linux/sourceos-spec
  - Add JSON Schema types for:
    - WorkstationProfile
    - PackageManifest (layered)
    - DesktopProfile (GNOME)
    - LauncherAction / LauncherProvider (Albert)

## OS build + config homes

- SociOS-Linux/socios-coreos-config
  - Keep close to upstream; avoid large diffs.
  - Only add SYSTEM-level packages strictly required for the workstation (e.g., toolbox/podman/wl-clipboard).

- SociOS-Linux/socios-fedora-builder
  - Build-time overlays for Fedora-on-Apple images.
  - Optional: first-boot profile apply.

- SociOS-Linux/socios-scripts
  - Post-install profile application scripts (userland tool installs, dotfiles, etc.).

- SociOS-Linux/homebrew-socios (or a sibling tap)
  - Brew formulas / taps for any tools missing from upstream brew.

## Launcher integration home

- SociOS-Linux/albert
  - Prefer a small “SourceOS/Prophet actions” plugin (avoid patching core if possible).
  - If upstream plugin repos are archived/unreliable, we own our plugin.

# Testing

## Acceptance checks

1) `doctor` verifies the tool bundle is installed and callable.
2) Keybindings match the workstation defaults without collisions.
3) Albert actions execute correctly and do not require broad filesystem indexing.
4) Fusuma gestures work on supported hardware; Kinto mappings are applied and reversible.

## Regression checks

- GNOME upgrades do not break baseline behavior (extensions pinned/validated).
- Theming changes do not break libadwaita apps.
- rpm-ostree updates preserve SYSTEM invariants.

# Security / trust boundaries

- All agentic actions must be permission-gated via desktop portals where applicable.
- Secrets reside in user keyrings; profile installers must not leak credentials.
- TOOLBOX/AUR is treated as untrusted build surface; outputs should be content-addressed/pinned where possible.

# Open questions

- Where we put the long-lived “profile registry”: inside SourceOS-Linux/sourceos-spec as schema + examples, or in a dedicated SourceOS workstation repo.
- Which GNOME extensions are blessed as part of v0 (pin set + versions).
- How we package Albert actions plugin across rpm/brew/nix.

