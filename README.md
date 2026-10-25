# Dependency policy

The packages that the automatic dependency updates of the zairakai group must not update yet, and why.

The update tools (`@zairakai/js-dev-tools` for npm, `zairakai/laravel-dev-tools` for Composer) read these files at each
run, so a rule changes without a release of the tools and without a cascade through the packages.

## Files

| File | Ecosystem | Read by |
| :--- | :-------- | :------ |
| `node.json` | npm | `update-dependencies` of js-dev-tools |
| `php.json` | Composer | `update-dependencies` of laravel-dev-tools |

Another file (bash, docker...) can be added for another tool.

## Format

```json
{
  "entries": [
    {
      "package": "typescript",
      "reason": "Why the package is held back.",
      "since": "2026-10-11",
      "unblock": { "version": "7.0.0", "peers": ["typescript-eslint"] }
    }
  ]
}
```

- `package`: the package that is not updated, whatever the kind of update (weekly update, fix of an advisory).
- `reason`: one sentence, the real cause.
- `since`: the day the entry was added.
- `unblock` (optional): the condition that lifts the block. The block is lifted when the latest release of every package
  of `peers` accepts `version` in its peer range. Without `unblock`, the entry is lifted by hand.

## Lifting a block

A scheduled job of this repository checks the `unblock` condition of every entry of `node.json`. When it is met, the job
opens a merge request that removes the entry, merges it when its pipeline succeeds and sends a notification. The update
tools take the package at their next run. Entries without `unblock`, and `php.json`, are removed by hand.

## Rules

- A project can still list a package of its own in `reject` of its `.dependency-update.json`.
- If this file cannot be read, the update tools stop instead of updating blindly.
- Commits follow the handbook (`feat(policy): #ticket subject`), signed.
