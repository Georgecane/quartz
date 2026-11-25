# Oracle records

An oracle is an independent implementation or platform tool used to establish expected behavior. It must be recorded in the case manifest with its platform, version policy, and commands.

## Rules

- Record the exact tool version at capture time. Do not write latest as a version.
- Prefer at least one platform-native oracle and, where feasible, a second independent implementation.
- Compare only capabilities implemented by both tools. Record unsupported fields separately; do not treat missing output as an empty value.
- Preserve disagreements between oracles and QUARTZ. Investigate them before changing expected output.
- Do not derive expected output solely from the parser under test.
- Keep commands read-only when possible. Mount disk images read-only and never execute package scripts.
- Record the host OS build and architecture alongside tool versions.

## Initial oracle set

| Domain | Primary oracle | Secondary comparison |
| --- | --- | --- |
| UDIF / HFS+ | hdiutil and mounted macOS filesystem metadata | dmg2img or an independent HFS+ reader, within supported features |
| Flat PKG / BOM | pkgutil and lsbom | xar and an independent package reader |
| APFS | diskutil and mounted macOS filesystem metadata | libfsapfs or The Sleuth Kit, within supported features |
| Mach-O | Apple otool / vtool | LIEF |
| dyld shared cache | version-matched Apple dyld sources and platform tooling | an independent cache reader such as dylex |

Tool availability and command-line options vary by OS release. The exact commands that were successfully executed must be captured in the case record; this table is a selection guide, not a promise that every command is available on every host.
