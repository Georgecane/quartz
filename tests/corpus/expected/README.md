# Expected-output contract

Each available corpus case records three independent expected-output documents. Paths are relative to QUARTZ_CORPUS_ROOT and are declared by the case manifest.

## inventory.json

A normalized enumeration of objects discovered by the relevant parser layer. It should include stable identifiers where the format provides them, paths, object kinds, parent relationships, and volume or archive membership. Preserve original path bytes or a lossless representation when the format's naming rules cannot be represented as valid UTF-8.

## metadata.json

Format-specific structural metadata: image geometry, filesystem/container identifiers, volume roles, snapshot identifiers, package metadata, Mach-O load commands, cache mappings, architecture, platform, checksums, and verification results as applicable. Unknown and unsupported values must be represented explicitly; they must not be replaced with fabricated defaults.

## content.json

A content manifest for extracted logical streams. Each entry should include its path or stable object identity, object kind, logical size, SHA-256, and applicable fork, xattr, symlink, sparse-extent, or hard-link information. Resource forks and alternate streams must be hashed separately from the data fork. Hard links must preserve shared identity rather than being represented as unrelated copies.

## Comparison rules

- Normalize only fields that are genuinely host-dependent, and document each normalization.
- Keep raw oracle output separately when practical so normalization can be reviewed.
- A missing value is not equivalent to an empty value or zero.
- Do not discard metadata just because one oracle does not expose it.
- Content hashes must be calculated independently from the QUARTZ extraction result.
- Expected files for a planned case are intended output paths only; they must not be treated as captured evidence until the case becomes available.
