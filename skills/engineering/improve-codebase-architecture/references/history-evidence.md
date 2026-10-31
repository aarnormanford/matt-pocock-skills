# History evidence: hot spots, co-change and checklist scoring

Use this when a scan needs git history as evidence: finding hot spots, proving shotgun surgery (one change forcing edits in files that don't import each other), or scoring a repo against an outside checklist of code smells.

## Work from a snapshot

Read files from a scratch export, not from a pipeline's checkout. Run history commands against the bare repo, read-only.

```bash
SHA=$(git --git-dir=$BARE rev-parse origin/main)
mkdir -p $SCRATCH/tree && git --git-dir=$BARE archive $SHA | tar -x -C $SCRATCH/tree
echo $SHA > $SCRATCH/SHA     # every finding cites this SHA
```

## Date windows: use `--since-as-filter`

`git log --since=N.days` stops walking at the first commit older than the cutoff, because it assumes commit dates only go forward. One commit with a wrong date (a pipeline commit dated 2000, a rebased import) silently cuts the history short, and no error appears. `--since-as-filter` (git 2.37+) checks every commit instead.

Check for the problem with `git log --first-parent --format=%ad --date=short $SHA | sort | uniq -c`. A year far outside the expected range is the tell.

## Hot spots

```bash
git --git-dir=$BARE log --since-as-filter=30.days --format= --name-only $SHA \
  | grep -v '^$' | sort | uniq -c | sort -rn | head -40
```

Filter out generated files, lockfiles and docs before reading the list.

## Co-change (shotgun surgery)

Evidence that holds up has three parts:

1. **Pairs that change in the same commit.** Run `git log --since-as-filter=N.days --format=@%h --name-only $SHA`, then group files by commit and count pairs.
2. **No import between them.** If one file imports the other, a shared change is normal coupling, not shotgun surgery. Grep each file for the other's module path.
3. **The trend over time.** Bucket each file's commits by week (`--format=%ad --date=short -- <file>`). If the count is falling toward zero, a migration or wave is ending, and the finding belongs in Further Notes rather than in work.

Also check for existing guards, such as parity tests or allowlist checks, before calling co-change unprotected.

## Scoring an outside checklist

- Map every item to evidence that names a file and line, or to history output. Record a verdict for each: held, partial or unproven.
- Items like shotgun surgery need history, so a file read alone can't convict them.
- Read compound items in full. "Errors / validation not at the boundary" is two checks, and passing one doesn't settle the other.
- Use the checklist's exact wording, from the source image or post, not a paraphrase. Paraphrases drop halves of compound items.
- Before proposing work for an item, check whether the repo's own plans already own it. Leave out items that don't hold rather than stretching them to fill the list.
