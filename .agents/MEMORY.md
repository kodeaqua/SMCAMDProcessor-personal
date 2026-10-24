# Project memory

Updated: 2026-10-25 (Asia/Jakarta).

## User and environment

- User communicates in Indonesian and wants implementation carried through to concrete results.
- User explicitly requested no workflows; use local builds/tests and `gh` for manual publication.
- Current request explicitly authorizes commit, push, and release after creating/updating agent guidance and memory.
- Repository: `/Users/aqua/Projects/SMCAMDProcessor-personal`; origin: `https://github.com/kodeaqua/SMCAMDProcessor-personal.git`.
- The reported panic came from this same PC. Observed CPU: AMD Ryzen 5 4500, six cores/twelve logical CPUs. Observed kernel: Darwin 25.6.0, macOS build 25G241. Recheck after system changes.
- User uses the `kodeaqua/Lilu` fork. Inspected revision `947dcabf1e4fc295ca1de833a34b44610973691a`, reporting version 1.7.4. Do not silently substitute upstream when checking compatibility.

## Panic investigation and changes

- User supplied `mp_rv_action() entry: 10 of 12 responses` with `AMDRyzenCPUPowerManagement(3.34)` UUID `2DC44CB6-2774-3BA5-84D3-83962C591168` in the backtrace.
- This is an all-CPU rendezvous entry timeout. CPU 4 reported the panic; the excerpt does not identify the stalled CPUs or establish the exact root cause.
- Local `/Library/Logs/DiagnosticReports/Kernel-*.panic` files also contain GPU/AMDSupport/AMD10000Controller TLB timeouts. No sleep was recorded in inspected logs. Their relationship to the rendezvous panic remains unconfirmed.
- Changed `pmAMDRyzen.c`: both `MachineIdle` halt branches now use adjacent `sti; hlt; cli`, restoring XNU's interrupt-disabled return contract, with compiler memory clobbers.
- Changed `AMDRyzenCPUPowerManagement.cpp`: moved P-state decoding/logging out of rendezvous callbacks; snapshots and validates P-state definitions before the barrier; serializes UserClient P-state reset with `rendezvousLock`.
- `CHANGELOG.md` records these as v3.34.2 rendezvous hardening, with runtime verification still required.

## Verification and artifacts

- Release x86_64 kext build succeeded with `kodeaqua/Lilu` headers at the revision above and MacKernelSDK `05094e5e88cec7caedbfb35e8449ed0db94bf95b`.
- Compiled idle object was checked with `otool`: both halt paths contain consecutive STI/HLT/CLI. Kext/dSYM UUIDs match. `git diff --check` passed.
- New kext UUID: `ADFDD46F-EAC2-3340-933A-93B82AE36462`.
- Binary SHA-256: `56b24a217f75462de420fc8f21794a020fce4d79ea0bec9737aff6e99085907e`.
- Versioned release package: `build/AMDRyzenCPUPowerManagement-3.34.2-Release.zip`; staging and manifest: `build/release-v3.34.2/`. Earlier diagnostic package: `build/AMDRyzenCPUPowerManagement-panic-fix.zip`.
- Temporary dependency/build locations: `/tmp/amdp-kodeaqua-Lilu`, `/tmp/amdp-lilu`, `/tmp/amdp-panic-build`. These may disappear; check before reusing.
- Successful build used `HEADER_SEARCH_PATHS=$(PROJECT_DIR)/MacKernelSDK /tmp/amdp-kodeaqua-Lilu/Lilu` as a quoted xcodebuild argument. Default local dependency directories were initially absent; MacKernelSDK is now initialized.
- The new kext has NOT been installed in EFI or loaded into the running kernel. Runtime idle/load/sleep-wake validation remains pending. Do not call all crashes resolved.

## Next steps

- Manual release target: `v3.34.2`, https://github.com/kodeaqua/SMCAMDProcessor-personal/releases/tag/v3.34.2. Verify publication with `gh release view v3.34.2`; publication does not imply EFI installation. Commit messages use `[skip ci]` to avoid triggering existing push workflows.
- On a later authorized deployment, back up active EFI, replace the kext, reboot, confirm the loaded binary, and exercise idle/load transitions and sleep/wake. Keep dSYM for any new panic.
- If crashes recur, obtain full panic and matching loaded-binary symbols; distinguish rendezvous timeout from GPU/TLB timeout before changing unrelated drivers.
