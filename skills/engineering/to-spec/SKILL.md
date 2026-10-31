---
name: to-spec
description: "Turn the current conversation into a spec and publish it to the project issue tracker: no interview, just synthesis of what you've already discussed."
disable-model-invocation: true
---

This skill takes the current conversation context and codebase understanding and produces a spec. Do NOT interview the user; just synthesize what you already know.

The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

## Process

0. Confirm the source material is actually in your context. If the request points at findings, decisions, or a discussion you can't see, don't synthesize a spec from nothing. Search session history first, then check what the gateway actually delivered (load the `hermes-agent` skill's troubleshooting reference, section "Reconstructing what a chat session received"). If it's still missing, say so plainly and offer two paths: the user pastes it, or you regenerate it from the repo. Recommend regenerating codebase findings, because they go stale and a pasted summary can't be verified against files.
   - When the user says the context is attached ("full context included"), look for the attachment before saying it's missing. Chat gateways deliver documents and photos as separate messages after the text, so they can land a turn later. Check the profile's `cache/documents/` and `cache/images/` and the gateway log after the text's timestamp.
   - A screenshot or list of generic criteria (code-smell indicators, a style checklist) is a rubric, not findings. Findings name files in a specific repo.
   - Resolve the repo yourself before asking. In a factory workroom it's the repo the factory builds from: the coordinator's role file names the base clone its workspaces come from, and GitHub's main is the reference for freshness. Ask only when more than one repo fits. Asking about something the workspace already answers costs the user a turn and reads as not knowing the setup.
   - Re-verify prior findings against current main before writing them into a spec, because health findings go stale. Record the baseline SHA they were scored at and check that it's an ancestor of the current head (`git merge-base --is-ancestor`). Re-check each finding at the file and line level, then report which held, which changed and which moved from unproven to proven.
   - Read code from a throwaway snapshot, never from a pipeline's working checkout: `git --git-dir=<bare> archive <sha> | tar -x -C <scratch>/tree`. Another worker may hold that checkout's lock and branch.

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain glossary vocabulary throughout the spec, and respect any ADRs in the area you're touching.
   - For cleanup or health work, read the repo's existing remediation plans, large-file or ownership registers, and deepening docs first. If a finding already has an owning plan step, put it under Out of Scope and name that owner. Spec only the gap. Two plans for one file give the work two owners, and they'll conflict.
   - Drop findings that turn out not to hold, and don't stretch weak ones to cover a whole checklist. Record evidence that's real but fading, such as co-change from a migration that's ending, in Further Notes instead of slicing it into work.

2. Sketch out the seams at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point you can. The fewer seams across the codebase, the better - the ideal number is one.

Check with the user that these seams match their expectations.

Before a seam goes into Testing Decisions, check that its harness exists and runs something:
- Find the test files the gate's command would pick up. A glob gate like `node --test $(find src -name '*.test.mjs')` over zero files still exits green, so a passing gate proves nothing.
- If the proof needs a device or simulator, check that the implementer's host can produce it (the iOS Simulator needs macOS with Xcode). Also check whether the scenario needs a dev build. If the pipeline can't run it, the spec says where it runs and that the PR must report how it was done.
- Say plainly when the proof is weaker than a rerunnable test. A one-time simulator pass shows the change worked when the PR merged, but nothing reruns it on the next change. For an illegal-states change, the type checker is the lasting guard.

3. Write the spec using the template below, then publish it to the project issue tracker. Apply the `ready-for-agent` triage label - no need for additional triage.
   - Before publishing, check whether `ready-for-agent` on that tracker starts automatic pickup by an implementation pipeline (a factory, agent board, or bot). If it does, or you can't tell, draft the spec in chat and publish only when the user explicitly says so. A planning request doesn't authorize implementation, and the label can turn the spec into work without anyone deciding to start it.
   - If `/to-tickets` will break the spec into sub-issues, leave `ready-for-agent` off the spec. Only the leaf tickets get it, because a labeled parent invites a builder to take on the whole spec as one job. Open the body by saying it's the spec, that its sub-issues are the tickets, and that sending them to builders is a separate request.

<spec-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any further notes about the feature.

</spec-template>
