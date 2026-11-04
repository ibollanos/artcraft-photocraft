---
name: artcraft-photocraft
description: Edit, retouch, composite, or export photos in PhotoCraft. Use when the user names PhotoCraft or asks for a photo-editing task it can perform.
---

# PhotoCraft

PhotoCraft targets 1:1 parity with Adobe Photoshop 2026 (27.11.0), and its command ids, labels and menu paths follow that version's menus.

If neither `photocraft-live` nor `photocraft-headless` is in your tool list, stop and ask the user to register one. Do not substitute shell image tools.

## Choose the transport

Use `photocraft-live` when a PhotoCraft window is open and the user wants to watch, or when the task needs UI state or screenshots.

Use `photocraft-headless` for unattended work, batch jobs, or when no window is open.

When unsure, call `session_list` on `photocraft-live`. If it fails, the app is not running, so use headless. Pick one transport for the task and do not mix them on one document.

## Plan in Photoshop terms

Write the task as an ordered Photoshop workflow first: menu path, tool, values, and the expected result.

Search for the Photoshop method if you do not know it. Keep each step at command level, not UI clicks.

## Resolve the plan to MCP calls

For each step, call `command_list` on the chosen server with a filter on the Photoshop label or menu path. Match the row by `menu` and `label`, then read its `params` string.

Never invent a command id or a parameter. If no row matches, say the step is unsupported and offer the closest alternative.

Check `enabled` and `disabledReason` before running a command.

## Execute and verify

Duplicate the document before experimental edits, with `image.duplicate` or `history.newDocument`.

Use `command_batch` for independent steps and `command_run` when the next step depends on the result. A failed batch keeps the steps that already succeeded, so inspect the document before retrying.

After each group of edits, call `render_preview` and look at the PNG. Use `doc_inspect` when the layer tree or selection matters.

Previews are bounded, so judge composition and colour rather than one-pixel detail. Undo with `edit.undo` when the result is wrong.

Ask before flattening, deleting layers, or overwriting a file. Otherwise save with `doc_save` and export with `doc_export`, using paths inside the automation roots.

## Parallel work

You may split independent work across sub-agents, such as exploring variants or reading references. Keep one writer per document so edits do not collide.

## Fallback to the source

Treat the source as a last resort, after `command_list` and the MCP docs have failed to answer a question. Locate the checkout with a command substitution rather than assuming a path:

```sh
repo="$(find "$HOME/development" -maxdepth 4 -type d -name photocraft -not -path '*/target/*' -print -quit)"
test -n "$repo" && git -C "$repo" rev-parse --is-inside-work-tree
```

Check that the tree is clean, then pull:

```sh
git -C "$repo" status --short
git -C "$repo" pull --ff-only
```

Pull only when `status --short` prints nothing. If it prints changes, read the checkout as it is.

Search `docs/` and the command modules for the missing behavior, then return to the MCP plan. Do not edit the checkout to finish a task.
