# QUARTZ Test Corpus

This directory defines the versioned fixture corpus used to validate QUARTZ image, filesystem, package, archive, executable, and shared-cache parsers.

The corpus is test data and test metadata. It is not part of the runtime and must not define parser behavior. Parser output is compared with independently recorded expectations and, where available, an external oracle.

## Status

- **planned**: the test case is specified, but its artifact and expected outputs have not been captured and verified.
- **available**: the artifact, its SHA-256 and size, and all expected-output files have been recorded. Only this status is eligible for normal corpus execution.
- **quarantined**: a previously available case is temporarily excluded. A quarantine reason is mandatory; the case must not be silently deleted or treated as passing.

The manifests under manifests/planned are specifications only. They do not claim that the corresponding artifacts already exist.

## Artifact storage

Large or restricted Apple artifacts must not be committed to this repository by default. Set QUARTZ_CORPUS_ROOT to a local directory containing the artifact and expected-output files. Every path inside a manifest is relative to that root, must remain inside it, and is case-sensitive.

An available case must record the artifact's exact byte size and lowercase SHA-256. Hashes must be calculated from the actual bytes after acquisition or generation. Never use placeholder, guessed, or copied hashes. Record the original source, product version/build, generator/tool versions, and acquisition details where known.

Before redistributing any artifact, verify that redistribution is permitted. A public URL does not itself grant redistribution rights. Keep source provenance even when the artifact is transformed into a smaller fixture.

## Case execution contract

A corpus runner should process an available case in these distinct phases:

1. **Resolve** the manifest and safe relative artifact path.
2. **Identify** the outer input format without trusting its filename.
3. **Inspect** structural metadata without extracting the complete artifact.
4. **Verify** checksums, signatures, references, bounds, and format-specific invariants where supported.
5. **Ingest** selected data through bounded streaming interfaces.
6. **Compare** normalized metadata and content hashes with expected outputs and independent oracle records.
7. **Report** unsupported features, incomplete data, mismatches, and resource usage explicitly.

A parser returning success is not sufficient for a passing case. A case passes only when all required assertions pass. Unsupported encrypted or version-specific structures must be reported as unsupported or incomplete, never converted into fabricated metadata.

## Resource limits

Limits are per-case test-harness ceilings, not universal product limits. They must be chosen for the actual fixture and may not be relaxed automatically after a failure. Test input size, decompressed output, entry count, nesting depth, memory, and expansion ratio independently.

## Expected outputs

Each available case records three independent expected-output documents:

- **Inventory:** normalized enumeration of objects, paths, kinds, stable identifiers, parent relationships, and archive or volume membership.
- **Metadata:** format-specific structural metadata such as image geometry, filesystem/container identifiers, volume roles, snapshot identifiers, package metadata, Mach-O load commands, cache mappings, architecture, platform, checksums, and verification results.
- **Content manifest:** logical size and SHA-256 for extracted streams, plus applicable resource forks, xattrs, symlink targets, sparse extents, and hard-link relationships.

Expected outputs must be captured from a known-good fixture and independently checked. Do not generate expected output from the QUARTZ parser being tested. A missing value is not equivalent to an empty value or zero.

## Oracle records

Oracle tool names, operating-system versions, commands, and observed versions must be recorded. An oracle is a comparison source, not an unquestionable authority. Preserve disagreements between independent tools and investigate them rather than silently choosing one.

Initial oracle candidates:

| Domain | Primary oracle | Secondary comparison |
| --- | --- | --- |
| UDIF / HFS+ | hdiutil and mounted macOS filesystem metadata | dmg2img or an independent HFS+ reader, within supported features |
| Flat PKG / BOM | pkgutil and lsbom | xar and an independent package reader |
| APFS | diskutil and mounted macOS filesystem metadata | libfsapfs or The Sleuth Kit, within supported features |
| Mach-O | Apple otool / vtool | LIEF |
| dyld shared cache | version-matched Apple dyld sources and platform tooling | an independent cache reader such as dylex |

Tool availability and command-line options vary by OS release. The exact commands that were successfully executed must be captured in the case record; this table is a selection guide, not a promise that every command is available on every host.

## Adding a case

1. Add a manifest that validates against schema/case-v1.schema.json.
2. Keep it planned until the real artifact has been acquired or generated and inspected.
3. Record the artifact path, byte size, and SHA-256 from the actual file.
4. Capture expected outputs independently; record oracle versions and commands.
5. Set status to available only after artifact and expected-output validation succeeds.
6. Add malformed variants as separate cases with their own provenance and expectations. Never mutate the original fixture in place.

The corpus format is versioned independently from QUARTZ crate APIs. Breaking manifest changes require a new schema version and migration policy.
