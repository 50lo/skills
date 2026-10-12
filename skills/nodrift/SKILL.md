---
name: nodrift
description: Track which Markdown documents describe which code or document sections, detect changes since their last review, and update affected documentation. Use when setting up documentation freshness checks, checking no-drift.lock, or maintaining docs after tracked code changes.
---

# NoDrift

This folder is the complete skill package:

```text
nodrift/
  SKILL.md
  scripts/nodrift.py
  agents/openai.yaml
```

Keep documentation current using the bundled [scripts/nodrift.py](scripts/nodrift.py).
The script records reviewed targets in `no-drift.lock`; the agent reads the code
and documentation, decides whether the prose needs changes, and records the review.
A fresh check proves only that tracked targets match their saved snapshots.

## Run the bundled script

Resolve `scripts/nodrift.py` relative to this skill's actual location, then use
that absolute path while running commands from the project being checked.
Do not change into the skill directory to run checks or assume it is on PATH.

Set `NODRIFT` to that resolved absolute script path. The examples below use it;
replace `/absolute/path/to/nodrift` with the actual skill directory:

```sh
NODRIFT=/absolute/path/to/nodrift/scripts/nodrift.py
python3 "$NODRIFT" --help
python3 "$NODRIFT" status
python3 "$NODRIFT" check
```

Requires Python 3.12 or newer on macOS or Linux; no third-party packages.
For Python targets, use the same Python minor version that created the bindings.
On a parser mismatch, use that interpreter or review the targets and documentation
before acknowledging a refresh with the new interpreter.

## Create tracking

Read the document and the implementation it describes before recording a new
binding. Bring the document up to date first; a new binding establishes the
reviewed snapshot immediately.

```sh
python3 "$NODRIFT" link docs/auth.md src/auth.go
python3 "$NODRIFT" link docs/api.md 'src/client.py#Client.connect'
python3 "$NODRIFT" link docs/overview.md 'docs/auth.md#Token Validation'
```

Repeat `link DOC TARGET` to bind a document to multiple targets. Whole files work
in any language. Python supports functions, classes, and methods; Markdown
supports headings and their subsections. Quote targets containing `#` or spaces.
A method binding does not track its imports, globals, or other dependencies;
add separate bindings for dependencies that matter to the document.

Alternatively, write references in the document's prose or inline code:

```text
Implementation: @./../src/auth.go.
Client method: @./../src/client.py#Client.connect.
Details: @./guide.md#token-validation.
```

Then run `link DOC` to discover those references. Inline paths are relative to
the document's directory; command-line paths are relative to the working directory.
Use normalized heading fragments, such as `token-validation`, in inline references.
References inside fenced or indented code examples are ignored. Use explicit
`link DOC TARGET` for paths with spaces or punctuation delimiters.

Keep `no-drift.lock` with the project's code and documentation and include it in
the task's changes. In Git projects it lives at the nearest repository root,
including worktrees. Outside Git it uses the nearest ancestor lockfile, or creates
one in the working directory. Docs must be `.md` files; all paths must stay inside
that root. Only recorded bindings are checked; an empty check means no coverage.

## Maintain documentation during coding

1. Run `check` to establish the current state before changing tracked code.
   Note existing stale bindings so they are not mistaken for new changes.
2. Use `refs FILE` to find documents affected by a file you change. Run `check`
   again after the edits; it includes uncommitted changes.
3. For each affected stale binding, read the current target and its document.
   Compare the documented behavior to the code and edit the prose if needed.
   If the target is missing, renamed, or ambiguous, resolve the binding rather
   than treating the error as proof that the prose is obsolete.
4. After reviewing, acknowledge only the bindings actually reviewed:

   ```sh
   python3 "$NODRIFT" link docs/api.md 'src/client.py#Client.connect' --ack
   ```

   Use `link DOC -a` only after reviewing every existing binding for that document
   and any newly discovered inline references. This command refreshes them all.
   A review may conclude that no prose edits are needed. Editing a document alone
   does not clear a stale binding. Never acknowledge merely to make a check pass.
5. Run a final full `check`. Report documents reviewed, edits made, and any remaining
   stale or unresolved bindings. If unrelated pre-existing drift remains outside
   the task, report it rather than silently refreshing it.

## Other commands and failure handling

```sh
python3 "$NODRIFT" refs src/auth.go
python3 "$NODRIFT" refs 'src/client.py#Client.connect'
python3 "$NODRIFT" check --changed src
python3 "$NODRIFT" unlink docs/auth.md src/auth.go
```

- `status` lists saved bindings without checking freshness. `refs FILE` includes
  whole-file and symbol bindings; `refs FILE#SYMBOL` matches only that target.
- `--changed PATH` is a supplied filter, not automatic Git diff detection. It
  selects affected documents and checks all their bindings.
- Exit 0 means success, 1 means stale or unresolved bindings during `check`, and
  2 means a usage, lockfile, or command error. Missing docs/targets and invalid
  Python can appear as `STALE`; read the reason before taking action.
- Python fingerprints ignore formatting and comments but include docstrings and
  decorators. Other whole files and selected Markdown sections track every byte.
- `check` does not discover new inline references. `link DOC` does and retains
  existing bindings even if their references were removed. To remove tracking,
  remove the inline reference if present and run `unlink DOC TARGET`.
- Serialize modifying commands. Failed links leave the lockfile unchanged;
  fix the reported issue rather than editing saved fingerprints by hand.

Do not add installation scripts, hooks, or CI configuration unless the user
separately requests that integration.
