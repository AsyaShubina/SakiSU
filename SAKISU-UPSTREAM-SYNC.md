# SakiSU Upstream Sync Plan

This branch is rebuilt from the latest ReSukiSU `origin/main`.

Target branch:

```text
sync/resukisu-main-20260705
```

## Goal

Replay SakiSU-specific work on top of the newest ReSukiSU mainline in a clean,
reviewable order. Do not hard-merge the old SakiSU branch history.

## Replay Order

1. Branding and package migration
   - Project name: `SakiSU`
   - Android package namespace: `com.sakisu.sakisu`
   - GitHub repository links: `XingChenRS/SakiSU`
   - Keep ReSukiSU as upstream credit only.

2. vivo/iQOO support
   - Manager switch text remains: `去除vr或适配vivo特性`
   - `vendor_boot` path removes `vr.ko` only and must not inject KernelSU LKM.
   - `init_boot`/compatible boot ramdisk path keeps normal LKM injection and
     prefers `_vivo` KMI/LKM variants.
   - `boot-patch-vivo` must auto-detect image kind from ramdisk content; do not
     depend on a UI partition value.
   - `vr.ko` cleanup must cover `lib/modules` and `lib/modules/<version>-gki`
     roots, including `modules.load`, `modules.dep`, `modules.softdep`, and
     `modules.load.recovery`.
   - Keep `boot-info classify-image` for Manager-side KMI dialog decisions.
   - Avoid loading embedded LKM assets before the vendor_boot auto-detect path
     decides they are needed.

3. Signing and manager trust policy
   - Do not return to forced v2-only APK signing.
   - Kernel and ksud signature checks should accept a trusted v2 certificate;
     if v3 or v3.1 blocks are present, their certificates must match too.
   - Keep exact `base.apk` tracking; do not accept `base.apk.prof`,
     `base.apk.idsig`, or sibling artifacts as manager APKs.

4. CI behavior
   - Push-triggered builds remain the primary validation path.
   - Long-lived keystore secrets are preferred when present.
   - Missing keystore secrets fall back to a self-consistent ephemeral key.
   - Crowdin must skip cleanly when Crowdin secrets are absent.
   - Preserve normal and vivo LKM build artifacts.

5. Documentation
   - Root `README.md` should exist.
   - `docs/README.md`, `docs/zh/README.md`, `docs/vivo.md`, and
     `docs/zh/vivo.md` must describe current code behavior.
   - `DEVLOG-VIVO.md` records implementation details and must stay aligned with
     `boot_patch.rs`, `KsuCli.kt`, and `Install.kt`.

## Verification Gates

- `git diff --check`
- Markdown local link and image reference check
- `cargo fmt --manifest-path userspace/ksud/Cargo.toml -- --check`
- `cargo fmt --manifest-path userspace/ksuinit/Cargo.toml -- --check`
- GitHub Actions for the sync branch, then dev/main after merge:
  - Build Manager
  - Build SU
  - Clippy check
  - Rustfmt check
  - ClangFormat check
  - ShellCheck
  - Crowdin Action

