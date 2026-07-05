# DEVLOG: vivo / iQOO compatibility

This file records behavior implemented in code. Keep it aligned with:

- `userspace/ksud/src/boot_patch.rs`
- `manager/app/src/main/java/com/sakisu/sakisu/ui/util/KsuCli.kt`
- `manager/app/src/main/java/com/sakisu/sakisu/ui/screen/Install.kt`
- `.github/workflows/ddk-lkm.yml`

User-facing background and usage live in `docs/zh/vivo.md` and `docs/vivo.md`.

## Scope

The manager-side switch is intentionally simple: **`去除vr或适配vivo特性`**.

When enabled, SakiSU uses one backend command:

```text
ksud boot-patch-vivo
```

`ksud` decides what the selected image actually needs.

| Image | Action |
|---|---|
| `init_boot.img` or compatible boot ramdisk | Inject `kernelsu.ko` and `ksuinit`; prefer the `_vivo` KMI/LKM variant. |
| `vendor_boot.img` | Remove `vr.ko` and its `modules.*` references only; do not inject KernelSU files. |

Turning the vivo switch off restores the normal SakiSU patch flow.

## Backend Rules

`userspace/ksud/src/boot_patch.rs` keeps vivo handling partition-agnostic:

1. `patch_vivo()` adds `vr.ko` to `remove_module` if it is not already present.
2. `patch_vivo()` then calls the normal `patch()` path.
3. `patch()` loads the ramdisk cpio before loading embedded LKM resources.
4. If the cpio contains `lib/modules/*.ko`, the image is treated as `vendor_boot` and `no_install` is enabled automatically.
5. `remove_vendor_modules()` discovers `lib/modules` and `lib/modules/<version>-gki/` roots dynamically.
6. The cleanup covers `modules.load`, `modules.dep`, `modules.softdep`, and `modules.load.recovery`.

The backend also exposes:

```text
ksud boot-info classify-image <image>
```

It prints one of:

- `vendor_boot`
- `init_boot`
- `unknown`

Manager uses this classification to avoid unnecessary KMI dialogs without duplicating boot-image parser logic.

## Manager Flow

`KsuCli.kt` copies a selected image into cache and calls `boot-info classify-image` when classification is needed.

`Install.kt` uses the result like this:

- SelectFile + vivo ON + classified `vendor_boot`: skip the KMI dialog and run rmvr through `boot-patch-vivo`.
- SelectFile + vivo ON + classified `init_boot` or `unknown`: keep the KMI dialog, because LKM injection may be needed.
- DirectInstall + selected partition `vendor_boot`: skip the KMI dialog.
- Other GKI install paths with no custom `.ko`: keep the KMI dialog.

When vivo mode is enabled and a KMI string is selected, `installBoot()` appends `_vivo` unless the selected KMI already has that suffix. This is harmless for vendor_boot rmvr because the backend skips LKM resource loading on that path.

## Expected User Flow

```text
init_boot.img:
  Install -> SelectFile -> choose init_boot.img
  -> choose androidXX-Y.Z_vivo KMI
  -> boot-patch-vivo injects KernelSU LKM

vendor_boot.img:
  Install -> SelectFile -> choose vendor_boot.img
  -> no KMI dialog when classification succeeds
  -> boot-patch-vivo removes vr.ko and modules.* references only
```

## Signing State

The current signing path must remain compatible with modern Android Gradle Plugin output:

- `kernel/manager/apk_sign.c` requires a trusted v2 certificate.
- If v3 or v3.1 signature blocks are present, their certificates must also be trusted.
- `userspace/ksud/src/apk_sign.rs` no longer rejects APKs merely because v3/v3.1 exists.
- CI prefers repository `KEYSTORE` secrets; if they are absent, it generates an ephemeral same-batch key and passes its certificate size/hash to both LKM and Manager builds.

Do not reintroduce a blanket v2-only signing requirement unless the kernel verifier policy changes again.

## Lessons Kept

- Do not gate rmvr on `partition == "vendor_boot"`; SelectFile may not expose a partition dropdown.
- Do not make the vivo switch mean "rmvr only"; `init_boot` still needs LKM injection.
- Do not hard-code `lib/modules/6.1-gki`; vivo devices also ship 5.10, 5.15, 6.6, 6.12, and other layouts.
- Do not load embedded LKM assets before the vendor_boot decision, because vendor_boot rmvr should not need LKM assets at all.
- Keep the exact `base.apk` match in `kernel/manager/throne_tracker.c`; prefix matching accepts `base.apk.prof` or `base.apk.idsig` and breaks manager detection.
