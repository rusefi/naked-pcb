# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A photo archive of "naked" (opened-up) automotive electronics PCBs: aftermarket ECUs (MaxxEcu, MoTeC, Link, Haltech, Syvecs, EcuMaster, Fueltech, ...), PDMs, wideband controllers, igniters, and similar boards. There is no source code, no build system, and no tests. All work here is adding/organizing photos and editing Markdown.

`AGENTS.md` (used by Codex) just points to this file. Keep shared guidance here, not there.

## Layout and conventions

- One top-level folder per device, named after the product (e.g. `MaxxEcu-Race`, `motec m800 evo x`, `HTG GCU`). Folder names are inconsistent in casing and may contain spaces; follow the existing name when adding to a folder rather than renaming it.
- Each folder holds the photos plus a lowercase `readme.md` (not `README.md`). The root `README.md` is a one-liner and is not an index of the folders.
- A folder's `readme.md` must reference **every** image file in that folder (JPG, PNG, etc.) using standard Markdown image syntax `![description](filename)` (rule from `.junie/guidelines.md`). When adding photos to a folder, add matching image lines to its `readme.md`; when creating a new device folder, create its `readme.md` with a `# Device Name` heading and one image line per photo.
- Image references are relative to the folder, and file names are case-sensitive on GitHub (`IMG_1552.JPG` vs `.jpg`). Match the actual extension exactly.
- Some readmes also carry notes: identified ICs with datasheet links (`LinkECU/readme.md`), an approximate year, or an external link to a teardown write-up (`cantcu`, `t113-mmi`). Keep such notes short and next to the relevant photos.
- Photos are committed as-is from phones/cameras; don't rename or re-encode existing images.

## Useful checks

Find image files not referenced by their folder's readme (the main consistency rule of this repo):

```bash
for d in */; do
  r="$d/readme.md"
  for f in "$d"*.{jpg,JPG,jpeg,png,PNG}; do
    [ -e "$f" ] || continue
    b=$(basename "$f")
    { [ -f "$r" ] && grep -qF "($b)" "$r"; } || echo "unreferenced: $f"
  done
done
```

List device folders with no `readme.md` at all:

```bash
for d in */; do [ -f "$d/readme.md" ] || echo "$d"; done
```
