# Release Checklist (v1.0 and beyond)

CopyCat releases are cut by tagging `vX.Y.Z`, which triggers
`.github/workflows/release.yml`. This checklist gates a release.

## 1. Quality gates (CI must be green)

- [ ] `cargo fmt --all --check`
- [ ] `cargo clippy --all-targets -- -D warnings`
- [ ] `cargo test --workspace` on Linux, macOS, Windows
- [ ] `cargo test -p copycat-core --features sftp` (Linux)
- [ ] `copycat-gui-core` tests + `cargo check -p copycat-gui` (Linux w/ webkit)

## 2. Security review

- [ ] `cargo audit` clean (no unpatched advisories) — enforced in CI.
- [ ] Credential handling spot-check: no secrets in logs or the audit file
      (endpoints redacted via `copycat_core::audit::redact_endpoint`), passwords
      only in memory / `COPYCAT_PASSWORD`, never a flag.
- [ ] API defaults: binds `127.0.0.1`; exposing requires token or TLS.
- [ ] SFTP host-key verification against `known_hosts` is on by default.
- [ ] Run `/security-review` (or equivalent) over the release diff.

## 3. i18n readiness

CopyCat is English-only for v1 but is structured so UI strings are isolated:

- [ ] CLI/user-facing strings live in the CLI/GUI layers (not `copycat-core`),
      so a future `fluent`/`gettext` layer can wrap them without touching the
      engine.
- [ ] No locale-dependent parsing of machine data (checksums, sizes, config are
      locale-independent).
- [ ] GUI strings are in the frontend (`gui/ui`), ready for a resource bundle.

> Full localization (message catalogs, RTL, pluralization) is a post-1.0 goal;
> this check confirms nothing blocks it architecturally.

## 4. Version bump & changelog

- [ ] Bump `version` in `cli/Cargo.toml` (`[workspace.package]`) and
      `gui/Cargo.toml`, and `packaging/*` where pinned.
- [ ] Move `CHANGELOG.md` `[Unreleased]` items under the new version + date.
- [ ] Commit, then tag: `git tag vX.Y.Z && git push --tags`.

## 5. Publish (automated from the tag)

- [ ] GitHub Release with binaries + completions + man page.
- [ ] Homebrew, Winget, Chocolatey, apt(.deb), dnf(.rpm), npm — manifests under
      `packaging/` with placeholders substituted from built artifacts.
- [ ] macOS notarization + Windows Authenticode signing (release secrets).

## 6. Post-release

- [ ] Smoke-test install from each channel (`copycat --version`).
- [ ] `copycat install-gui` resolves the GUI on each platform.
