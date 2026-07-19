# SC2 MPQ Container Format

`.SC2Map` and `.SC2Mod` files are MPQ (Mo'PaQ) archives. The workspace uses the
MPQEditor toolchain (via the `sc2-mpq` skill) to inspect, unpack, and repack them.

## Archive structure

An MPQ archive contains:

- A list of files (the **listfile**, stored as `(listfile)` internally).
- Optional `(attributes)` and `(signature)` blocks for integrity.
- The actual file payloads, possibly compressed (zlib/bzip2/lzma) and/or encrypted.

The editor writes a listfile on save; if a third-party tool removes it, the
archive becomes harder to enumerate but the file payloads are still present.

## Critical files inside SC2 archives

See `document-structure/overview.md` for the file list. The most-modified files
during porting are:

- `DocumentHeader` (binary, dependency list — DO NOT cross-copy).
- `DocumentInfo` (XML, dependency mirror).
- `Base.SC2Data/GameData.xml` and `GameData/*.xml` (Catalog data).
- `TriggerLibs/*.galaxy` and `TriggerLibs/*_h.galaxy` (Galaxy libraries).
- `MapScript.galaxy` (compiled trigger script; auto-regenerated on save).

## Unpacking and repacking

The `sc2-mpq` skill wraps `MPQEditor.exe` with three Python helpers:

- `extract_mpq.py` — unpack an archive to a directory.
- `pack_mpq.py` — pack a directory back into an MPQ archive.
- `verify_mpq.py` — verify listfile, attributes, and signature.

Standard flow:

```
.SC2Map --extract--> dir/ --edit--> dir/ --pack--> .SC2Map
```

When repacking:

- Preserve the original listfile; otherwise the editor may refuse to open the
  archive.
- Preserve `(attributes)` if present; some editors verify it on load.
- Use MPQEditor's `w` (write) mode carefully: deleting a file from the archive
  invalidates the listfile unless MPQEditor is told to rebuild it.

## Live sync

The legacy `mpq-tooling` source (registered in `config/workspace.json`) provides
live-sync utilities that watch a directory and push changes into an open MPQ
archive. This is faster than full unpack/pack cycles for iterative editing.

## Pitfalls

- `DocumentHeader` is binary; copying it between maps silently changes the
  effective dependencies. Always preserve the target map's `DocumentHeader`.
- Some editor versions write the same file to both `Base.SC2Data/GameData.xml`
  and `Base.SC2Data/GameData/<file>.xml`; the engine reads both, but the
  explicit `<Includes>` in `GameData.xml` is authoritative.
- MPQ archives have a 4 GB hard limit; large custom asset packs (e.g. imported
  videos) can hit this.
- File names inside MPQ are case-sensitive on some platforms; the editor
  consistently uses PascalCase for system files (`MapInfo`, `Objects`, etc.).

## See also

- `document-structure/overview.md`
- `data-spaces/usage-guide.md`
