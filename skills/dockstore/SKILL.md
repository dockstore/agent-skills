---
name: dockstore
description: Find and inspect bioinformatics tools, workflows and notebooks (CWL, WDL, Nextflow, Galaxy, Jupyter) registered on Dockstore using its MCP server. Use when the user asks to search Dockstore, find a pipeline for a bioinformatics task, or read a workflow's descriptor, versions, test parameters or Dockerfile.
---

# Dockstore

Dockstore is a registry of bioinformatics tools and workflows, served through the GA4GH Tool Registry Service (TRS) API. The `dockstore` MCP server exposes it read-only; all tools are prefixed `mcp__dockstore__` (or similar, depending on client).

## Workflow

1. **Discover** with `search_entries` when you know what the user wants but not which entry. Use `query` for keywords and narrow with facets: `type`, `language`, `author`, `input_format`, `output_format`, `input_data`, `output_data`, `operation`, `subject_area`. Facets are ANDed. Keep `limit` small.
2. **Inspect** the entry with `get_entry` (TRS id such as `#workflow/github.com/org/repo/name`, or `quay.io/org/repo` for tools). It returns metadata, a description (often a README, truncated by default) and versions.
3. **Pick a version** and call `get_version` with `<entry id>:<version name>` to see its files and `descriptor_path`.
4. **Read files** with `get_file` (`version_id` plus `path` from `get_version`): primary descriptor, imports, test parameter files, Dockerfile.

## Query syntax

`query` and facet arguments use Lucene syntax, case-insensitive: `"variant calling"` for phrases, `AND`/`OR`/`NOT`, `-term`, `GATK*` wildcards, `haplotyp~` fuzzy. Multiple words match in any order unless joined by `OR`. Escape `+ - = && || > < ! ( ) { } [ ] ^ " ~ * ? : \` with a backslash to match literally. Regular expressions are not supported.

## TRS-style tools (alternative path)

These mirror the TRS API and are useful for enumerating or filtering by registry fields rather than keyword search:

- `list_tools`: paged listing filtered by `name`, `toolname`, `organization`, `registry`, `alias`, `tool_class`, `descriptor_type`, `checker`. `offset` is a **page number**, not an item index; follow `next_offset`. Use `limit: 1` to count matches via `total`.
- `get_tool`: one entry with a page of versions. Pass `summary: true` for monorepo workflows, which can have 1000+ versions; keep paging with `next_version_offset` until unset (pages may be short or empty before the end).
- `get_tool_version`: one version; pass `files` (a descriptor type: `CWL`, `WDL`, `NFL`, `GALAXY`, `SMK`, `JUPYTER`, `SERVICE`) to list its files.
- `get_tool_descriptor_by_path`: fetch the primary descriptor (omit `relative_path`) or any listed file. Note `version_id` here is the bare version name (e.g. `master`), unlike `get_version`.
- `get_trs_info`: which Dockstore instance the server targets and its TRS version. `local_only: true` checks the server is up without calling Dockstore.

## Tips

- Prefer `search_entries` -> `get_entry` -> `get_version` -> `get_file` for most tasks.
- Large READMEs and files are truncated (`description_limit`, `content_limit`); pass `null` only when you need the whole thing.
- Run `get_entry` and `get_version`/`get_file` calls in parallel when independent.
- When recommending a workflow, cite its TRS id, chosen version, language, and a link: `https://dockstore.org/workflows/<path>` (for `#workflow/github.com/org/repo/name` use `github.com/org/repo/name`).
- Pin recommendations to a specific version rather than a moving branch where possible, and say so if only branches exist.
- The server is read-only: it cannot register, publish or modify entries. For that, point the user to the Dockstore website or CLI.
- To test against the QA instance, use `https://qa.dockstore.org/mcp` instead of the production URL in `.mcp.json`. Check with `get_trs_info` which instance you are on.
