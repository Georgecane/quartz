<p align="center">
  <img src="assets/quartz-logo.webp" alt="quartz logo" width="360">
</p>

# quartz

**An Apple user-space execution environment for non-Apple hardware.**

**quartz** is a Rust-based project for running original Apple user-space software on hardware that was not manufactured by Apple. It constructs the Apple user-space environment an application expects, supplies the corresponding Apple libraries, frameworks, system resources, and runtime services, and translates target CPU instructions to the host instruction-set architecture when required.

**quartz** does not boot a complete Apple operating system and is not a conventional virtual machine. It does not aim to statically or dynamically analyze the target application. Its purpose is to provide an execution environment in which the original application and original Apple components can run.

> **Project status:** Early-stage architecture and development. The components and capabilities described below are design goals unless explicitly implemented and documented in the source tree.

## Vision

An Apple application depends on more than its executable file. It expects a compatible user-space environment: Apple frameworks and dynamic libraries, resources, dynamic-linking behavior, Darwin ABI contracts, runtime facilities, IPC mechanisms, and system services.

**quartz** aims to assemble and provide that environment without booting a complete macOS, iOS, or iPadOS installation. It preserves the original Apple application and system components, translates CPU instructions across host and target ISAs when necessary, and supplies the interfaces and resources those components expect.

The central design principle is:

**Run original Apple software with the original Apple libraries, frameworks, and resources inside a compatible user-space environment. Translate execution across CPU architectures where needed; do not replace Apple's software with rewritten substitutes.**

## What quartz is

- **A user-space execution environment:** It provides the runtime context and system interfaces needed by Apple user-space applications.
- **An ISA translation system:** It translates or interprets target CPU instructions for the host ISA when the host and target architectures differ. This is execution-time compatibility, not offline conversion of an application's binary into a different architecture.
- **An Apple component provider:** It makes the appropriate original Apple frameworks, dynamic libraries, shared-cache images, resources, and other system components available to applications.
- **A runtime compatibility layer:** It provides the loading, linking, process, memory, threading, IPC, and service contracts expected by the target software.
- **A platform-aware environment builder:** It assembles a coherent set of components for a selected Apple platform, OS version, build, and architecture.

## What quartz is not

**quartz** is not:

- A virtual machine that boots a complete Apple guest operating system.
- A full reimplementation of macOS, iOS, or iPadOS.
- A project for rewriting Apple's frameworks as host-native replacements.
- A static-analysis or dynamic-analysis tool for the application being executed.
- An offline binary converter that rewrites an ARM64 application into an x86_64 application or vice versa.

The runtime must still read the executable-format metadata required to map an image, link its dependencies, and start execution. That operational loading work is not application analysis and is not the project's goal.

## Execution model

The intended execution path is:

1. **Select an Apple user-space environment.** Choose the platform, OS version, build, and target architecture required by the application.
2. **Assemble the required Apple components.** Make the matching frameworks, dynamic libraries, shared-cache images, resources, and runtime services available through the environment's filesystem and loading interfaces.
3. **Load the original application.** Map its original executable image and resolve the dependencies it declares against the selected Apple components.
4. **Translate CPU instructions when needed.** Execute the target ISA through an interpreter or dynamic translation backend appropriate to the host ISA. The original application and Apple libraries remain the software being executed.
5. **Provide the expected runtime contracts.** Mediate the required Darwin interfaces, process and memory behavior, IPC, threading, and user-space service interactions.

This model is intended to make the application see a coherent Apple user-space environment. It does not imply that every kernel or system service can be replaced by a simple library shim; the required runtime and service contracts must be implemented according to the behavior the software actually needs.

## Design goals

- **Original Apple software:** Execute original Mach-O applications, frameworks, dynamic libraries, and system components.
- **Cross-architecture execution:** Support ARM64 and x86_64 target software, with instruction translation when required by the host architecture.
- **Original system resources:** Provide the frameworks, libraries, assets, configuration, and other resources expected by the selected Apple environment.
- **Apple platform profiles:** Support environment profiles for macOS, iOS, and iPadOS, including configurable device profiles where applicable.
- **Environment construction:** Import Apple components from recovery and installer artifacts, IPSW images, extracted filesystems, and standalone dyld shared caches.
- **Dependency resolution:** Resolve image dependencies against a coherent platform, OS version, build, and architecture.
- **Runtime compatibility:** Implement the execution, loading, ABI, memory, IPC, threading, and service boundaries required by the target software.
- **Host independence:** Keep the Apple user-space environment separate from host-specific implementation details.
- **Explicit compatibility reporting:** Report missing, unsupported, or incompatible components instead of silently substituting unrelated versions.

## Architecture

**quartz** is organized around environment construction and runtime execution.

### Environment Builder

The Environment Builder assembles the Apple user-space components required for an execution environment.

Its main components are:

- **Image Importer:** Detects and imports Recovery images, IPSW packages, installer artifacts, extracted filesystems, and standalone shared caches while preserving source provenance and platform metadata.
- **Shared Cache Manager:** Reads and exposes dyld_shared_cache images, mappings, subcaches, symbols, and fixup information so that original Apple libraries can be made available to the runtime.
- **Environment Assembler:** Selects compatible system components, resolves the dependencies declared by executable images, and constructs a version-aware Apple user-space filesystem view and environment manifest.

These components inspect container and executable-format structures only as required to import resources, identify components, construct the environment, and support loading. They are not intended to perform static or dynamic analysis of the target application's behavior.

### Runtime

The runtime provides the execution environment between original Apple user-space code and the host.

Planned areas include:

- **ISA execution and translation:** Interpret or dynamically translate target CPU instructions for the host ISA.
- **Mach-O image loading:** Map original executable images and perform the loading operations required to begin execution.
- **Dynamic linking:** Provide the necessary dyld loading, image lookup, and dependency-resolution behavior.
- **Address-space management:** Provide process address-space layout, memory mapping, protections, and related behavior expected by the target software.
- **Darwin ABI and system interfaces:** Implement the required user-space and kernel-facing contracts.
- **Mach IPC:** Provide the Mach port and message behavior required by applications and frameworks.
- **XPC and runtime services:** Supply the user-space service contracts needed by the selected environment.
- **Process and threading facilities:** Provide process state, threads, synchronization, and runtime primitives.
- **Apple environment resources:** Expose the selected frameworks, dynamic libraries, shared-cache images, resources, and platform information.
- **Host integration:** Mediate host CPU, memory, graphics, storage, input, and other resources where required.

The runtime executes original Apple code and provides the environment it expects. It is not intended to replace Apple's frameworks with newly implemented host-native equivalents.

## Supported input sources

The Environment Builder is designed to accept four source categories:

| Source | Purpose |
| --- | --- |
| **Recovery** | Apple recovery environments and macOS installer-related artifacts |
| **IPSW** | Apple restore packages and their system payloads |
| **Filesystem** | Extracted Apple system filesystems |
| **Shared cache** | Standalone dyld_shared_cache files and associated subcaches |

Online acquisition should prioritize discoverable official Apple sources. Local imports remain supported independently of the download mechanism.

An installer, a recovery environment, and an IPSW are distinct artifact types. **quartz** must identify what each artifact actually contains rather than treating them as interchangeable formats.

## Architecture support

The design targets both:

- **ARM64**
- **x86_64**

The target ISA and host ISA are separate execution properties. When they differ, the runtime must translate or interpret instructions so that the original target-architecture code can execute on the host. This does not convert the original binary or its Apple libraries into a different-architecture binary.

The selected environment must provide components matching the target software's architecture and platform requirements. A component for one architecture must not be silently substituted for a component built for another.

## Environment consistency

An execution environment must preserve the relationship between its Apple platform, OS version, build, target architecture, frameworks, dynamic libraries, and shared caches.

The initial design favors build-consistent environments. Components from different OS builds must not be mixed silently. Any future cross-build compatibility must be established explicitly.

Each assembled environment is expected to carry a manifest describing its source artifacts, platform, version, build, architecture, available components, and compatibility state.

## Development principles

1. Keep acquisition, container parsing, component import, shared-cache access, environment assembly, ISA translation, and runtime execution as separate layers.
2. Preserve original Apple binaries and their provenance. Do not rewrite them as a general compatibility strategy.
3. Use executable-format metadata only for the operational tasks of importing, mapping, loading, and linking software; application static or dynamic analysis is outside the project's purpose.
4. Treat platform, OS version, build, and architecture as first-class environment compatibility dimensions.
5. Make parsing defensive and validate offsets, lengths, mappings, and references before using imported data.
6. Prefer explicit capability and compatibility reports over implicit fallbacks.
7. Keep host-specific implementation details outside the Apple user-space environment model.

## Project status

**quartz** is at an early stage. Its architecture is being defined before the environment importer and runtime are fully implemented. Features described as goals or planned components should not be interpreted as working functionality.

Implementation status, supported artifact formats, and tested platform/build combinations will be documented here as they become verifiable.

## Contributing

Contributions, design discussions, and technical reviews are welcome. Before proposing a major architectural change, please open an issue describing the problem, the proposed approach, and its impact on the project's core design principles.

## License

**quartz** is licensed under the **GNU General Public License v3.0 (GPL-3.0)**. See the [LICENSE](LICENSE) file for the full license text.
