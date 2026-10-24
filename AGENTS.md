# Repository guidance

## Start here

- Read `.agents/MEMORY.md` for local project context, investigation results, and outstanding verification. Treat dated observations as evidence to recheck, not proof of current runtime state.
- Communicate with the user in concise Indonesian unless requested otherwise.
- Complete authorized implementation and local verification. Preserve unrelated working-tree changes.
- Use local build/test commands. Do not use GitHub Actions or add/modify workflows for this user's tasks. Use `gh` for requested GitHub operations and manual releases.
- Commit, push, and release when the current task authorizes them; do not infer permanent publication authorization from a previous release.

## Project map

- `AMDRyzenCPUPowerManagement/`: C/C++ kernel extension, MSR access, CPU idle/P-state control, timers, Super I/O, and GPU telemetry.
- `SMCAMDProcessor/`: VirtualSMC sensor plugin.
- `AMD Power Gadget/`: Swift/SwiftUI app.
- `APGLaunchHelper/`: launch helper; `AMDPowerGadgetTests/`: app tests.
- `SMCAMDProcessor.xcodeproj`: Xcode project and shared schemes.
- `Config/Version.xcconfig`: shared marketing/build version; inspect target-specific module versions before changing release identifiers.
- `docs/ARCHITECTURE.md`, `docs/BOOT_ARGS.md`, `docs/PRIVILEGE_AND_SECURITY.md`, and `docs/TROUBLESHOOTING.md`: focused technical references. Confirm claims against implementation.

## Dependencies and local builds

- This user's installed Lilu is from `https://github.com/kodeaqua/Lilu`, not an assumed upstream build. Record the exact header/source revision used for compatibility checks.
- Initialize the pinned `MacKernelSDK` submodule with `git submodule update --init MacKernelSDK` when needed.
- Default dependency locations are `Lilu.kext/Contents/Resources`, `VirtualSMC.kext/Contents/Resources`, and `MacKernelSDK/`. Use local search-path overrides if dependencies are elsewhere; do not commit temporary absolute paths.
- Build only relevant targets. Example with dependencies in their default locations:

```sh
xcodebuild -project SMCAMDProcessor.xcodeproj \
  -scheme AMDRyzenCPUPowerManagement -configuration Release \
  ARCHS=x86_64 ONLY_ACTIVE_ARCH=NO -derivedDataPath ./build \
  CODE_SIGNING_ALLOWED=NO build
```

- Other schemes: `SMCAMDProcessor`, `AMD Power Gadget`, `APGLaunchHelper`. Quote names containing spaces.
- For relevant app changes, run local tests:

```sh
xcodebuild test -project SMCAMDProcessor.xcodeproj \
  -scheme 'AMD Power Gadget' -configuration Debug \
  -destination 'platform=macOS' -only-testing:AMDPowerGadgetTests \
  -derivedDataPath ./build-test CODE_SIGNING_ALLOWED=NO
```

- Do not run a later `clean` against shared DerivedData if earlier products are needed for packaging.
- Check `git diff --check`. For kernel interrupt/assembly changes, inspect compiled instructions when relevant. Match kext/dSYM UUIDs before distributing artifacts.

## Kernel constraints

- Keep all-CPU rendezvous callbacks bounded and free of logging, allocation, blocking locks, and diagnostic formatting. Validate inputs before entering barriers; decode shared state afterward.
- Preserve XNU's interrupt-state contract in `MachineIdle`: adjacent `sti; hlt; cli`, with compiler memory clobbers, including fallback branches.
- Review lock order, sleep/wake lifecycle, timer callbacks, and UserClient concurrency together. Keep per-CPU state indexed and bounded correctly.
- Respect per-CPU-profile capability gates and privilege validation. Do not enable unvalidated MSR/CPPC writes or weaken write authorization to work around failures.
- A kext in a panic backtrace identifies an involved path, not necessarily the root cause. A reporting CPU need not be the stalled CPU.
- Separate successful compilation, static verification, installation, and runtime validation in reports. Do not claim a panic fixed solely because a build passes.
- EFI changes, kext loading/unloading, and reboot are separate from source/build work. Perform them only within explicit user authorization for those operations.

## Memory and artifacts

- Update `.agents/MEMORY.md` when durable environment facts, decisions, or verification results change. Record unresolved questions and the next verification step.
- Keep secrets and unrelated personal information out of instructions, memory, logs, and release packages.
- `AGENTS.md` and `.agents/` currently match repository ignore rules. If the user asks to commit this guidance, explicitly include only the intended files rather than unignoring private agent data broadly.
- Build output belongs under ignored `build/` or a temporary directory. Preserve dSYMs and a manifest with dependency revisions, UUIDs, hashes, and validation status when packaging diagnostic kext builds.
- Manual release notes must state behavior changes, checks performed, and remaining runtime limitations. Do not describe a diagnostic hardening build as hardware-validated without evidence.
