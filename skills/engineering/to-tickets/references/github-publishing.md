# Publishing tickets to GitHub

Use this when the tracker is GitHub Issues: a spec as the parent issue and one sub-issue per ticket. It also covers handing tickets to the Hermes factory.

## Before creating anything

- **Duplicates.** Search open and closed issues for each ticket's distinctive symbols and phrases: `gh issue list -R $R --state all --search "<term>"`. If one already covers the work, extend it instead of filing a second.
- **Drift.** If the tickets rest on a scan of an older SHA, list what changed since then on the paths they touch: `git --git-dir=$BARE log --oneline <scan-sha>..origin/main -- <paths>`. If that's empty, the evidence still holds. Put both SHAs in the spec's evidence line. Also skim the new commits for plans that now own part of the work.
- **Labels.** Run `gh label list -R $R --limit 200 --json name --jq '.[].name'` and choose area labels from that output. `gh issue create` fails if a label doesn't exist.
- **What `ready-for-agent` sets off.** Check timers, dispatch scripts and workflows that read the label, plus the repo's dispatch doc for what the label means there. If it starts automatic pickup, publish only after the user explicitly approves.

## Create in order, idempotently

1. Create the parent first: `gh issue create -R $R --title "$T" --body-file spec.md --label ...`. The command prints the URL, and the number is `${URL##*/}`.
2. Write each ticket body with a `#PARENT` placeholder. Fill it in with `sed`, then create the tickets in dependency order, blockers first.
3. Siblings that mention each other (for example, a rebase note about a shared file) can't use numbers that don't exist yet. Refer to them by title at creation, then run `gh issue edit N --body-file` once every number exists.
4. Before each create, look for the exact title: `--search "\"$T\" in:title"`, then `jq 'select(.title=="$T")'`. Search matches loosely, so keep the exact match. A rerun after a partial failure then reuses what exists instead of duplicating it. Save each number to a file as you go.

## Native sub-issue links

```bash
node_id() { gh api graphql -f query="query{repository(owner:\"$OWNER\",name:\"$NAME\"){issue(number:$1){id}}}" --jq .data.repository.issue.id; }
gh api graphql -f query="mutation{addSubIssue(input:{issueId:\"$(node_id $PARENT)\",subIssueId:\"$(node_id $CHILD)\"}){subIssue{number}}}"
```

Write blocking edges in each ticket's "Blocked by" section. Use a native dependency link only after confirming it works on that repo.

## Read back before reporting

Query the parent's `subIssues` with state and labels, and grep each ticket body for the parent reference and sibling numbers. Report from that readback, not from what the create commands printed.

```bash
gh api graphql -f query='query{repository(owner:"O",name:"N"){issue(number:P){title labels(first:5){nodes{name}} subIssues(first:20){nodes{number title state labels(first:5){nodes{name}}}}}}}'
```

## Ticket bodies for an implementation pipeline

An agent builder reads the ticket cold, so add two sections to the issue template:

- **Scope.** List what's in and what's out. For each out item, name the plan or ticket that owns it, so the builder doesn't take it on.
- **Verify.** Give the commands that prove the change, plus any manual check. If the pipeline can't run that check (a simulator, a device), say where it runs and that the PR has to report whether it was done.

For a behavior-preserving refactor, say "Nothing a user sees changes" so the builder treats a visible change as a bug. For an illegal-states ticket, write the criterion as "X doesn't compile". When a ticket replaces loose fields with a union, have the PR list every field combination the code writes today and the state each one maps to, before the type changes.

## Hermes factory handoff

- **The GitHub issue is the ticket.** A builder card names one issue, and its PR closes that issue. Don't offer Kanban cards instead of issues. The coordinator creates the cards from issues.
- **Publishing isn't dispatching.** Builders start only when the coordinator gets an intake request, through the profile's existing intake path. After publishing, tell the user which tickets are published and which are dispatched. Offer one coordinator request that covers the independent tickets, with one builder per issue.
- **Spec header.** The parent issue opens by saying that it's the spec, that its sub-issues are the tickets, where it was requested, and that sending the tickets to builders is a separate request.
