# SC2 Editor Knowledge Base Sources

This subrepository holds the source documents consumed by the `sc2-editor-knowledge`
skill in the parent `sc2-porting-workspace`. Documents are plain Markdown or text so
they can be edited, diffed, and versioned normally.

## Layout

| Directory | Coverage |
|-----------|----------|
| `galaxy/` | Galaxy script language: syntax, native functions, missing natives, error codes, sample libraries. |
| `catalog/` | Catalog data format: XML schema, entry kinds, field overrides, inheritance, merge semantics. |
| `editor/` | Galaxy Editor modules, panels, workflows, AI-assisted modding guides. |
| `document-structure/` | SC2 document files: `DocumentHeader`, `DocumentInfo`, `MapInfo`, `Objects`, `Triggers`, `Attributes`. |
| `triggers/` | Trigger system: ECA structure, GUI vs custom script, libraries, registration, init order. |
| `data-spaces/` | Data space mode, `Base.SC2Data/GameData.xml` includes, modular catalog layout. |
| `bank/` | Bank persistence: file format, signing, encryption, read/write patterns. |
| `mpq/` | MPQ container format for `.SC2Map`/`.SC2Mod`: listfile, attributes, repacking. |
| `runtime-contracts/` | Runtime observer, ready gate, ScriptError scanning, Bank watcher, process gate. |
| `legacy/` | Pre-existing materials from `docs/galaxy/` and `合作指挥官-起义狂潮/docs/银河编辑器/`, kept verbatim for reference. |

## Source attribution

Each top-level directory contains a `SOURCES.md` listing where its content came from.
The `legacy/` directory preserves original pre-existing materials verbatim with their
original license and attribution headers.

## Index

The parent workspace builds a vector index over this directory using
`scripts/kb/kb-build.py`. The index data lives in `../artifacts/kb-index/` and is not
tracked by this repository. A `manifest.json` next to the index records the source hash
and model version so contributors can detect drift and rebuild when needed.

## Contribution

- Prefer editing Markdown in place over creating new files unless a topic is genuinely
  missing.
- Keep section headings flat (H2/H3) so the indexer can chunk by heading.
- Avoid binary assets; reference them by path instead.
- When importing external content, record the source URL, license, and retrieval date
  in the file's frontmatter or top comment.
