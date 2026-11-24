<p align="center">
  <img src="assets/quartz-logo.webp" alt="QUARTZ logo" width="360">
</p>

# QUARTZ

**An Apple user-space environment emulator for non-Apple hardware.**

QUARTZ is a Rust-based project exploring how Apple software can run on hardware that was not manufactured by Apple. Its goal is to provide a coherent Apple user-space environment in which original Apple executables, frameworks, dynamic libraries, and system components can be loaded and executed without booting an entire guest operating system.

QUARTZ is designed around the preservation and execution of original Apple binaries, not the wholesale reimplementation or rewriting of Apple's frameworks.

> **Project status:** Early-stage architecture and development. The components and capabilities described below are design goals unless explicitly implemented and documented in the source tree.

## Vision

Apple applications are more than executable files. They rely on Mach-O loading semantics, `dyld`, system frameworks, Darwin interfaces, Mach IPC, runtime services, and platform-specific environment information.

QUARTZ aims to bring these pieces together into a focused user-space execution environment. Instead of starting a complete macOS, iOS, or iPadOS guest installation, QUARTZ builds the environment required by a target application and provides the runtime boundary needed to execute it.

The central design principle is:

**Preserve Apple's original user-space binaries and frameworks wherever possible. Emulate the environment they expect, rather than replacing the frameworks themselves.**

## Design goals

- **User-space execution:** Run target applications without booting a complete guest operating system.
- **Original Apple components:** Load original Mach-O executables, frameworks, dynamic libraries, and system components.
- **Cross-architecture support:** Design for both ARM64 and x86_64 guest binaries.
- **Apple platform profiles:** Model the target environment for macOS, iOS, and iPadOS, including configurable device profiles where applicable.
- **Recovery-based environment construction:** Import system components from supported Apple recovery and installer sources, IPSW images, extracted filesystems, and standalone dyld shared caches.
- **Dependency-aware assembly:** Resolve application dependencies against a consistent platform, OS version, and build.
- **Host independence:** Separate the guest Apple environment from the host machine and host operating system.
- **Explicit compatibility reporting:** Report missing, unsupported, or incompatible components instead of silently substituting unrelated versions.

## Architecture

QUARTZ is organized around several cooperating subsystems.

### Environment Builder

The Environment Builder turns Apple system sources into a structured guest user-space environment.

Its main components are:

- **Image Importer:** Detects and imports Recovery images, IPSW packages, extracted filesystems, and standalone shared caches.
- **Shared Cache Manager:** Parses and manages `dyld_shared_cache` data, including image indexes, mappings, symbols, fixups, and associated subcaches.
- **Environment Assembler:** Resolves dependencies, selects compatible system components, and constructs a version-aware guest filesystem and environment manifest.

### Runtime

The runtime is intended to provide the execution boundary between original Apple user-space code and the host.

Planned areas include:

- Mach-O loading and image initialization
- `dyld` behavior and dependency resolution
- Guest virtual memory and address-space management
- Darwin ABI and system-call interfaces
- Mach ports, messages, and IPC
- XPC and required user-space service contracts
- Threading, synchronization, and process state
- Hardware and device-profile queries
- Host integration and resource mediation

These interfaces are not intended to become replacements for Apple's frameworks. They exist to provide the operating-system and runtime contracts those original frameworks expect.

## Supported input sources

The Environment Builder is designed to accept four source categories:

| Source | Purpose |
| --- | --- |
| **Recovery** | Apple recovery and macOS installer-related artifacts |
| **IPSW** | Apple restore packages and their system payloads |
| **Filesystem** | Extracted Apple system filesystems |
| **Shared cache** | Standalone `dyld_shared_cache` files and associated subcaches |

For online acquisition, QUARTZ prioritizes discoverable official Apple sources. Local imports remain supported independently of the download mechanism.

An installer, a recovery environment, and an IPSW are distinct artifact types. QUARTZ must identify what each artifact actually contains rather than treating them as interchangeable formats.

## Architecture support

The design targets both:

- **ARM64**
- **x86_64**

Guest architecture is distinct from host architecture. Cross-architecture execution may require instruction interpretation or translation, while Apple frameworks and libraries remain original guest binaries.

A binary or shared cache for one architecture must never be treated as though it were a native binary or cache for another architecture.

## Environment consistency

A guest environment must preserve the relationship between its platform, OS version, build, architecture, frameworks, and shared caches.

The initial design favors build-consistent environments. Components from different OS builds must not be mixed silently. Any future cross-build compatibility must be established by explicit analysis and validation.

Each assembled environment is expected to carry a manifest describing its source artifacts, platform, version, build, architecture, components, and compatibility state.

## Non-goals

QUARTZ is not intended to be:

- A conventional virtual machine that boots a complete Apple operating system.
- A full macOS, iOS, or iPadOS reimplementation.
- A project that rewrites every Apple framework into a host-native equivalent.
- A mechanism that converts ARM64 system libraries into x86_64 libraries or vice versa.

The focus is a user-space execution environment with a sufficiently accurate Apple runtime and system interface.

## Development principles

1. Keep image acquisition, image parsing, cache management, environment assembly, and runtime execution as separate layers.
2. Preserve source files and their provenance; do not modify original Apple binaries as a general compatibility strategy.
3. Treat platform, OS version, build, and architecture as first-class compatibility dimensions.
4. Make parsing defensive and validate all offsets, lengths, mappings, and references.
5. Prefer explicit capability and compatibility reports over implicit fallbacks.
6. Keep host-specific implementation details outside the guest environment model.

## Project status

QUARTZ is at an early stage. The architecture is being defined before the runtime and import pipeline are implemented. Features described as goals or planned components should not be interpreted as working functionality.

Implementation status, supported artifact formats, and tested platform/build combinations will be documented here as they become verifiable.

## Contributing

Contributions, design discussions, and technical reviews are welcome. Before proposing a major architectural change, please open an issue describing the problem, the proposed approach, and its impact on the project’s core design principles.

## License

A license has not yet been declared. Until a license is added to this repository, no open-source license should be assumed.
