# Fixture generators

Fixture generation is deliberately separated from parser execution. A generator creates an artifact; the capture process records its provenance, exact byte size, SHA-256, and independent expected outputs.

## macOS-generated cases

Use a dedicated, versioned macOS test host for fixtures that require Apple tooling. Record at minimum:

- macOS product version and build;
- hardware architecture;
- versions of hdiutil, pkgbuild, productbuild, pkgutil, xar, and other tools actually used;
- exact generator commands and relevant environment variables;
- creation timestamp in UTC;
- source files and their hashes;
- final artifact size and SHA-256.

Do not assume that an option such as ditto --hfsCompression produces a particular decmpfs codec. Inspect the resulting metadata and accept a codec-specific fixture only when its algorithm identifier is confirmed. For the LZVN case, the fixture must positively identify LZVN before it can become available.

## Reproducibility

Prefer deterministic source trees, fixed timestamps where supported, stable file ordering, and documented generator versions. Apple-generated disk images and packages may contain nondeterministic metadata; in that case preserve the artifact hash and capture record rather than claiming byte-for-byte reproducibility.

Generators must not execute installer scripts from imported packages. Test payloads should contain inert scripts with recognizable marker contents.
