---
title: "Worked Example: Your First Contribution"
icon: "🎓"
created: 2026-04-25
updated: 2026-04-25
status: stub
---

# Worked Example: Your First Contribution

> **Stub.** A real, end-to-end walkthrough of finding, fixing, testing, and submitting a small bug fix to the engine. The intent is "show, don't tell" — a contributor should be able to follow along on their own clone and watch the same workflow play out.
>
> Below is the skeleton. The actual content needs a real bug — picked from the historical issue tracker or invented — that's small enough to walk through in a single page but representative enough to teach the workflow.

## The bug we're fixing

> _TODO: pick a real, small, reproducible bug. Good candidates are things like:_
> - _A property default that's wrong on a built-in component (test fixture: spawn the component, assert default value)._
> - _A missing nullcheck causing a `NullReferenceException` in a specific code path (test fixture: trigger the code path, assert no exception)._
> - _A typo in a user-facing console message (test: snapshot string output)._
> - _An off-by-one in a math helper (test: assert outputs at boundaries)._
>
> _Should be: small enough to fix in <50 lines of changes, has a clear test, touches one project, doesn't require native rebuild._

> _Briefly describe the bug. Include the symptom from a user's perspective, then the actual broken behaviour._

## Step 1: Reproduce locally

> _TODO: walk through opening sbox-dev, hitting the bug, recording exact steps. Include screenshots if possible (the wiki currently has none — see the bigger-picture screenshot question)._

## Step 2: Find the offending code

> _TODO: show the search process. "I started by grepping for the error message." or "I knew it was in the physics system, so I opened `engine/Sandbox.Engine/Systems/Physics/`." This is the most-skipped step in tutorials and the most useful for a learner — how does an experienced contributor navigate the codebase?_

> _Once located, paste the offending function with context. Highlight the actual buggy line._

## Step 3: Understand why it's broken

> _TODO: explain the root cause. Was it a wrong default? A logic error? A missing check? Sometimes "the bug" you see is a symptom of a different underlying issue — explain how to think about that._

## Step 4: Write the fix

> _TODO: paste the corrected code. Show the diff inline if possible (`- old line` / `+ new line`)._
>
> _Discuss design decisions: why this fix and not another. If there were two valid fixes, mention the alternative and why the chosen one is better._

## Step 5: Write the test

> _TODO: this is the part most contributors skip and shouldn't. Show:_
>
> - _Where the test goes (which `Sandbox.Test*` project)_
> - _The new test method_
> - _How it would have failed against the old code_
> - _How to run just this test (`dotnet test --filter ...`)_

## Step 6: Run the suite

> _TODO: `dotnet test engine\Sandbox-Engine.slnx`. Show what success looks like, what failure looks like, what to do if an unrelated test breaks (someone else's regression — note it but don't try to fix it in this PR)._

## Step 7: Format and commit

> _TODO: `dotnet format`, then commit with a message like "Fix Rigidbody.Mass default initialisation" with a body explaining the issue ref and the fix. Reference the issue number._

## Step 8: Open the PR

> _TODO: walk through the PR description: what was broken, how it was fixed, what test was added. Show what a model PR description looks like — short, specific, references the issue, doesn't oversell._

## Step 9: Respond to review

> _TODO: if the reviewer asks for changes, what does that look like? Push more commits to the same branch (don't open a new PR). Address each comment with a reply. Squash if asked._

## Step 10: Merged

> _TODO: what happens after merge? Where do you see the change live? When does it land in the next build the community uses?_

---

## What this walkthrough should NOT be

- A made-up bug. Use a real one from history or pick a plausible specimen by inspecting the engine.
- A tutorial on writing tests in general. Reference [the unit tests page](../code/code-basics/unit-tests.md) for that.
- A complete style guide. Reference [Building from Source](contributing.md) for conventions.

## Stub maintainer notes

The walkthrough is most useful when the bug is *real*, the fix is *unobvious-but-not-clever*, and the test is *one a learner could plausibly write themselves on the second try*. If those three properties hold, the page teaches the workflow without bogging down in a specific subsystem's deep-knowledge prerequisites.

If filling this in: pick the bug first, then everything else follows from it. Don't pick the structure first and try to fit a bug into it — the prose will read like AI template again, which is precisely what this whole rewrite is trying to escape.
