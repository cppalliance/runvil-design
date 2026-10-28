# Ship Delete as a typed tool that removes one name, asks for `recursive` before it takes a tree, and promises no undo

## Summary

I recommend that Runvil ship `Delete` as a typed, one-path tool over `Access::remove`, and not leave deletion to a future shell. Five choices matter most.

First, ship the tool. Ten of thirteen harnesses have none, but their shell and patch routes carry five measured costs: no confinement, an exit flag that hides a failed delete, half-deleted trees after a timeout, no undo, and regex gates that miss spellings. Runvil has no shell, so the model could not delete at all. Second, take a directory with entries only when the call sets `recursive: true`. Third, remove the name and never follow a link. Fourth, promise no undo, read no content and add no stale-file guard, because Cursor's snapshot and its `dev` and `ino` guard serve an undo that Runvil cannot back. Fifth, answer an absent path with a success that opens `Nothing was deleted`, which departs from the standing plan's wording.

The largest risk is that Plan mode checks only the top path of a recursive delete, so a directory named `x.md` takes non-markdown files with it. That is very unlikely, and a test pins it. The fix is a VFS change, `vfs-plan-subtree`, because the Pack cannot see the mode. Confidence is high for the tool, the link rule and the refusal wording, and medium for the rest. The next step is the `delete-real` todo: replace the stub, add `recursive` to `rm.json`, and rewrite its description and the todo's absent-path text.

## Contents

- [The recommendation in brief](#the-recommendation-in-brief)
- [Criteria](#criteria)
- [Options and the do-nothing case](#options-and-the-do-nothing-case)
  - [Ship the typed tool, because the shell route has five measured costs and Runvil has no shell](#ship-the-typed-tool-because-the-shell-route-has-five-measured-costs-and-runvil-has-no-shell)
  - [Take a tree only on request, because Zed's implied recursion hides the one delete with no way back](#take-a-tree-only-on-request-because-zeds-implied-recursion-hides-the-one-delete-with-no-way-back)
  - [Remove the name and never follow the link, because following breaks the reply and the dangling case](#remove-the-name-and-never-follow-the-link-because-following-breaks-the-reply-and-the-dangling-case)
  - [Promise no undo for now, because the only undo that holds needs a VFS journal](#promise-no-undo-for-now-because-the-only-undo-that-holds-needs-a-vfs-journal)
  - [Report an absent path as a success that says nothing was deleted](#report-an-absent-path-as-a-success-that-says-nothing-was-deleted)
  - [Add no stale-file guard, because Cursor's protects an undo snapshot that Runvil does not take](#add-no-stale-file-guard-because-cursors-protects-an-undo-snapshot-that-runvil-does-not-take)
- [Behaviors to implement](#behaviors-to-implement)
  - [Ship `Delete` as a typed tool, and put any future shell `rm` behind the same gate](#ship-delete-as-a-typed-tool-and-put-any-future-shell-rm-behind-the-same-gate)
  - [Refuse a directory that has entries unless the call sets `recursive: true`](#refuse-a-directory-that-has-entries-unless-the-call-sets-recursive-true)
  - [Remove the name at the path and never follow a link](#remove-the-name-at-the-path-and-never-follow-a-link)
  - [Promise no undo, and say so in the description](#promise-no-undo-and-say-so-in-the-description)
  - [Answer an absent path with a success that opens with "Nothing was deleted"](#answer-an-absent-path-with-a-success-that-opens-with-nothing-was-deleted)
  - [Keep the policy's reason and render every refusal in one place](#keep-the-policys-reason-and-render-every-refusal-in-one-place)
  - [Reply with one line that names what went: kind, path and size](#reply-with-one-line-that-names-what-went-kind-path-and-size)
  - [Report what remains after a failed recursive delete](#report-what-remains-after-a-failed-recursive-delete)
  - [Read no content: one `stat` and one `remove` per call](#read-no-content-one-stat-and-one-remove-per-call)
  - [Claim the canonical path and add no lock of the Pack's own](#claim-the-canonical-path-and-add-no-lock-of-the-packs-own)
  - [Refuse an empty, whitespace-only or root path before any I/O](#refuse-an-empty-whitespace-only-or-root-path-before-any-io)
  - [Take one path per call](#take-one-path-per-call)
  - [Add no stale-file guard until `Stat` carries identity](#add-no-stale-file-guard-until-stat-carries-identity)
  - [Write the description from the behavior](#write-the-description-from-the-behavior)
- [Where Runvil should differ from Cursor](#where-runvil-should-differ-from-cursor)
- [Mapping to Runvil](#mapping-to-runvil)
- [Risks and open limits](#risks-and-open-limits)
- [Sources](#sources)

## The recommendation in brief

Table A gives one row per behavior, in the order of Behaviors to implement. The confidence word rates the evidence, and the reason is under each behavior.

| Behavior | Recommendation | Confidence |
|---|---|---|
| Typed tool | Ship `Delete` over `Access::remove`. Put any future shell `rm` behind the same gate | High |
| Directories | Refuse a directory with entries unless `recursive: true`. An empty directory goes without it | Medium |
| Links | Remove the name, never follow the leaf, and say a link went | High |
| No undo | Say the delete is permanent. Take no snapshot | Medium |
| Absent path | Reply with a success that opens `Nothing was deleted` | Medium |
| Refusals | Keep the policy's reason. Render every refusal with the shared `refusal` function | High |
| Reply line | `Deleted /a.txt (34 bytes).` with a kind word for other types | Medium |
| Recursive count | After a failure, count what remains, with a cap of 10,000 entries | Low |
| Read nothing | One `stat` and one `remove`. No content read | High |
| Canonical claim | Pass the path to `Access::remove`. Add no Pack lock | High |
| Empty and root paths | Refuse empty and whitespace-only paths in the Pack. The VFS refuses the root | Medium |
| One path | One string `path` and one boolean `recursive`. No arrays. Any other key is refused by name | High |
| No guard | Compare no identity or fingerprint until `Stat` carries identity | Medium |
| Description | Rewrite `rm.json`: permanent, link, `recursive`, absent path | High |

*Table A. The recommendation for each behavior, with its confidence. Sources: the behaviors below and the options in Tables 1 to 6.*

## Criteria

Six criteria judge every option and every behavior below. Each keeps one name through the report.

- **File survival.** A delete removes what the model named and nothing else, and the model knows what cannot come back. Two failures count. A wrong target goes (a link's target, a tree under a path the model took for a file). Or a right target goes with a promise of recovery that does not hold.
- **Checkable target.** What a gate or a user approves is a path and an effect, stated before the call runs. A shell command string fails this, because what `rm` removes depends on the working directory, glob expansion and variables.
- **Loud failure.** The model gets the outcome in one call, and no outcome is a silent wrong result. A refusal that names the path, the cause and the next step passes. A success sentence over a different outcome than the one asked for fails.
- **Replay.** The same call against the same files gives the same text on every host, so a recorded run replays. Anything that depends on the host's error wording, a clock or a race fails.
- **Bounded cost.** Work per call has a stated bound, and the model is told when a bound applies. A delete should cost a constant number of filesystem calls whatever the file size.
- **VFS fit.** The behavior works on Runvil's `Access` as it stands, the gap is named when it does not, and each policy is decided once in the layer that owns it: mode gates and claims in the VFS, message wording in the Pack.

When two criteria conflict, the order is File survival, then Checkable target, then Loud failure, then Replay, then Bounded cost. VFS fit is a constraint rather than a preference. File survival ranks first because Delete is the one file tool whose damage no later call can repair. Checkable target ranks second because it is the reason a typed tool can exist at all: it is the property the shell route lacks.

### Confidence and likelihood use two separate scales

Every recommendation carries a confidence word: high, medium, or low. Confidence rates the evidence behind the recommendation. Likelihood rates an event and uses a fixed ladder: very unlikely, unlikely, roughly even chance, likely, very likely, almost certain. The two never share a sentence.

Evidence carries tier labels. `(bundle)` and `(daemon)` mean read out of Cursor 3.27.12's shipped agent bundle and its unminified daemon twin. `(probe)` means measured by running something. `(source)` means read from a pinned commit of another harness or of Runvil, tests included. `(web)` means a public document. Each claim says whether it was observed, assumed, or judged, and a judgment is marked as mine. The evidence report is [tool-delete.md](tool-delete.md), cited by section name.

## Options and the do-nothing case

Six decisions have credible alternatives. Each gets a table with the do-nothing option named, the recommended option marked, and a verdict. The remaining choices (one path per call, the canonical claim, the reply wording) have no serious rival and appear only under Behaviors to implement. Confidence for each decision is in the brief table and under its behavior.

### Ship the typed tool, because the shell route has five measured costs and Runvil has no shell

Ten of the thirteen harnesses in [tool-delete.md](tool-delete.md) (section The short version) ship no delete tool. Five delete through a shell, Copilot deletes through a patch directive, and four accept either. So doing nothing is the common design, and it must be weighed as a real option (Table 1). In Runvil it means something different. The standing plan makes the shell a separate Pack and a non-goal of `plugin-std`, and Runvil has no patch tool, so with no `Delete` the model cannot delete anything. It has to ask a human.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| **Typed `Delete` tool (recommended)** | The Pack validates, takes one `stat`, calls `Access::remove`, words the reply | The target is a path the gate checks before the call runs. Mode policy, claims and the event sink apply with no Pack code. One approval covers one thing | A destructive primitive to design and test. A name the model must learn. No fallback for what it refuses | Recommended |
| Do nothing: no tool and no shell | Nobody. The model asks the human | No destructive surface. Nothing to get wrong. File survival is met trivially | The model cannot clean up its own output, remove an empty directory it created, or undo a bad write by removing the file | Right for a run that only reads or only writes notes. A per-run choice, not the default |
| Shell only (a later Pack, most likely over bashkit) | A command parser and a gate | One mechanism that composes: `rm -rf node_modules` in one call | The gate has to infer intent from a string. The five costs below. Whether bashkit's `rm` reaches `Access::remove` is not known | Set aside until that Pack exists and its `rm` is shown to use `Access` |
| Patch directive (Copilot, Codex CLI, OpenCode, Cline, Aider) | A patch parser, which Runvil lacks | A deletion reviewed beside the edit. Copilot's reaches the OS trash | A seventh tool outside the six. One approval for the whole patch. Apply is not transactional in Codex CLI, OpenCode or Aider | Set aside |
| Operator grant only (the zeroclaw shape) | The host decides whether the tool is offered | Deletion is a deployment decision, with no model-facing default | Needs no new mechanism: `tools.offer` by id already exists | Adopt on top of the tool, not instead of it |

*Table 1. Five ways to give a model deletion in Runvil. Sources: [tool-delete.md](tool-delete.md), Decision 1 and the Claude Code, Copilot, pi, zeroclaw and Codex CLI sections; the standing plan's non-goals.*

The evidence for the tool is measured on the routes it replaces. On the shell route, pi deleted `../outside/secret.txt` with exit 0 and Cline deleted an absolute path outside its working directory `(probe)`. Pi returned `isError: false` for `rm missing.txt; echo continuing` `(probe)`. A pi `rm -rf` of 20,000 files under a 0.4 s timeout left 14,845 entries, and the model saw only the timeout text `(probe)`. Pi's shipped example gate blocks `rm -rf` and allows `rm -fr`, `find -delete`, `git clean -fdx`, `unlink` and every PowerShell command `(probe)`. Zeroclaw's `find` check refuses `-exec` and not `-delete` `(source)`. Rig's own library tool beats its shell route on footprint, confinement and preview `(source)`. These are five costs, and a typed path avoids all five because the gate can judge it before the call runs. That is the Checkable target criterion.

The evidence for doing nothing is also real, and it is narrower than the headcount suggests. Claude Code deletes through `Bash`, and its classifier and sandbox rules are not in the extracted sources, so this report cannot show that Claude Code pays the five costs `(source)`. Copilot's patch route has the best recovery of any harness, because VS Code sends the file to the OS trash when `files.enableTrash` is on and the file system has a trash, and its chat edit session can restore the text `(source)`. That recovery belongs to the editor and not to the patch format, and Runvil has neither. Cursor itself ships both a tool and a shell classifier, with the tool as the precise path `(bundle)`. No source measures how often a model needs to delete. The case for the tool rests on the Checkable target argument and on Runvil having no other route, and not on how often deletion is wanted.

Two Runvil facts decide it. First, `Access::remove` already runs `gate`, then a claim, then the event sink, so the tool inherits `Mode::Ask` and `Mode::Plan` with no Pack code `(source)`. A shell `rm` inherits them only if the shell routes through `Access`, which this report did not verify for bashkit. Second, the `std/rm` definition already exists in the Pack, so the choice is about what the Pack does with a call. A prompt that never deletes can still leave `std/rm` out with `tools.offer("std/read")` and its siblings, as the plan describes `(source)`. The operator-grant row therefore costs nothing and is worth documenting. Whether a bare `tools.offer("std")` includes `Delete` is not stated in the plan's text and is an open limit.

The shell route lost for now, and it can win later. If the future shell Pack makes `rm` call `Access::remove`, the shell inherits the same gate and four of the five costs shrink to the exit-flag and command-parsing questions. If it spawns a real process, it carries all five. Until that Pack exists the typed tool is the only route, and the tool should be written so the shell Pack can reuse its refusals. The patch route lost for three reasons. Runvil has no patch tool. One approval for a patch covers "apply these edits" and not "delete this file", since OpenCode asks one `edit` permission listing every path `(source)`. And the patch routes read every target first, so Codex CLI cannot delete a binary file `(source)` and Cline reports every read error as `File not found` `(probe)`. Confidence in shipping the tool is high, because the five costs are measured on the routes it replaces and Runvil has no other route.

### Take a tree only on request, because Zed's implied recursion hides the one delete with no way back

Cursor refuses every directory, Zed removes them all recursively with no flag, and Runvil's `Access::remove(path, recursive)` already has the flag (Table 2). The do-nothing option here is to ship without any recursion.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| Files only, every directory refused, empty ones too (Cursor, rig, Codex CLI) | The Pack stats and refuses | One inode per call. No partial failure | With no shell, a directory can never be removed, not even an empty one left after its files go. A tree of N entries takes N calls | Set aside |
| Recursive by default (Zed) | The backend | One call covers the common case | `{path: "src"}` looks as harmless as `{path: "src/old.rs"}`. The prompt shows a path and nothing about a tree. Partial failure must be reported, and Zed's reply does not | Set aside |
| **`recursive: true` to take a tree; an empty directory goes without it (recommended)** | The model sets the flag; the backend removes; the claim covers the subtree | The dangerous case sits in the arguments, where an approver looks. It is `Access::remove`'s own shape and its tested behavior | A second parameter, so `rm.json` stops being Cursor's text verbatim. A model that omits the flag gets a refusal and one more call | Recommended |
| Do nothing: files and empty directories only, no flag in version one | The model, one call per entry | The smallest surface. The structured path can never delete a tree, as in Codex CLI's `recursive: false` | The N-call cost above, and a risk (my judgment) that a model loops on a generated directory | The fallback if the Plan-mode gap in Risks and open limits is judged too large |

*Table 2. Four ways to treat a directory. Sources: [tool-delete.md](tool-delete.md), Decision 2 and the Zed and Codex CLI sections; `memory-tests.rs` in the VFS crate.*

The cost of refusing every directory is specific to Runvil. In Cursor a refused directory sends the model to the shell, which is the route the tool was meant to avoid `(source)`. Runvil has no shell, so the refusal is final. The VFS backend already separates the cases: an empty directory goes with `remove(path, false)`, a directory with entries answers `DirectoryNotEmpty`, and `remove(path, true)` takes the subtree `(source)`. Zed's implied recursion fails the Checkable target criterion in the argument itself, and its pre-delete buffer pass blanks buffers over files that still exist after a cancel `(source)`. Codex CLI's flat refusal gets the safety and loses the capability. The explicit flag keeps both, at the cost of one parameter. Confidence is medium and not high, because no source measures how models use a `recursive` flag on a delete tool, and the departure from Cursor's one-parameter schema gives up some of the plan's "no new instructions" aim.

### Remove the name and never follow the link, because following breaks the reply and the dangling case

Decision 9 in [tool-delete.md](tool-delete.md) asks whether a delete acts on a link or on its target. The harnesses disagree, and rig disagrees with itself (Table 3).

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| **Remove the name, never follow the leaf (recommended)** | The backend (`resolve_no_follow`, `symlink_metadata`) | `delete link` does what a user expects. A dangling link is removable. The target survives | A model that believed it deleted the file sees it still there. The reply must say a link went | Recommended |
| Follow the leaf, remove the target (rig `Local`; Aider, read and not probed) | The Pack or backend canonicalizes the leaf | Deleting through an alias works | A dangling link or junction can never be removed. The target vanishes and the link dangles. The reply echoes the typed name, so `Deleted link.` names a thing that did not go. A link inside a confined base can steer the call outside it | Set aside |
| Follow to classify, remove the name (Cursor) | The gate and the executor, on different objects | Nothing the first lacks | The gate judges the target while the unlink removes the link. The undo snapshot describes a file that still exists. A dangling link is refused | Set aside |
| Do nothing: refuse every link | The Pack stats and refuses | No link question at all | Generated and dangling links can never be removed. No harness does this | Set aside |

*Table 3. Four ways to treat a symbolic link at the leaf. Sources: [tool-delete.md](tool-delete.md), Decision 9 and Q8; rig probe of a junction; Runvil `real.rs`.*

The evidence is a probe and a source read. Rig's `Local` workspace removed the target through a junction and refused to delete a dangling one, while its remote script's `rm -f` removes the link `(probe)`. Cursor's undo record described a target that was never deleted `(bundle)`. Runvil's real backend already removes a link as a link, and its trait documents that a removal "removes the link and never the target" `(source)`. `Access::stat` also uses `symlink_metadata`, so the Pack sees `FileType::Symlink` and can say so `(source)`. The first option costs nothing to build. The only work is the reply wording and a test that keeps a snapshot or a canonicalizing step from creeping in. Confidence is high, because a probe and Cursor's own mismatch show the cost of both rivals.

### Promise no undo for now, because the only undo that holds needs a VFS journal

Decision 6 asks what undo records and whether the tool promises recoverability (Table 4). The do-nothing option, no undo and no promise, is the first row, and it is the recommendation for now.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| **No undo in the tool, no promise (recommended now)** | The Pack states permanence in the description and the reply | Honesty. No read. Nothing to get half right | Every accidental delete stays deleted | Recommended until a journal exists |
| Snapshot in the Pack, bounded, text only (Cursor) | The Pack reads up to 50 MiB first | Most accidents involve a text file | Binary and oversized files have no undo and the model is not told which case it is in. The snapshot loses the BOM and CRLF. The read is wasted when an earlier entry exists. The evidence report estimates 150 to 250 MiB of transient memory for a 50 MiB file | Set aside |
| Journal in the VFS, so every delete is reversible | The VFS | The only version where undo is a property rather than a hope | A VFS feature: a journal or a copy-on-write store | The right long-term answer, and a separate decision |
| Typed record beside the reply, no restore (rig, Cline, Codex CLI) | The Pack reads, the host keeps the record | The host can show what was destroyed, preview a dry run and gate on the record | A read first (16 MiB in rig, 512 MiB in Codex CLI). Nothing restores from it. `Access::remove` returns a bool, so there is no channel for it | Take the shape with the journal |
| Lean on the host or git (Copilot's trash, Aider's commits) | The editor or git | The best recovery of any harness studied | Only where that layer exists. `RealBackend` has neither | A host choice, not the tool's job |

*Table 4. Five ways to handle recovery after a delete. Sources: [tool-delete.md](tool-delete.md), Decision 6 and Q5, Q7 and Q10; Runvil `operations.rs`.*

The reasoning rests on one distinction that three harnesses make: recording what was destroyed is different from promising to restore it. Rig's `FilesChanged`, Cline's `PatchFileChange` and Codex CLI's `AppliedPatchDelta` record and do not restore `(source)`. Cursor's snapshot looks like undo and is not for the files people most want back, since binary files, files over 50 MiB and files read with an error all delete with a success sentence identical to the text case `(daemon)`. Runvil has no restore path in `Access` at all, so a snapshot would have no consumer `(source)`. The recommendation is medium confidence, because it depends on whether the VFS gets a journal, which is a larger decision than this tool, and the evidence report reaches the same level.

### Report an absent path as a success that says nothing was deleted

Cursor, Zed, rig and the patch routes fail on a missing file (Aider's is fatal at parse time and a warning at apply time), and Runvil's own `Access::remove` returns `Ok(false)` (Table 5). The standing plan's `delete-real` todo says to map `Ok(false)` "to the not-found result", which is the do-nothing reading: an error.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| Error, as Cursor, Zed, rig, the patch routes and the plan's wording | The Pack maps `Ok(false)` to a failure | The model learns its picture of the tree was wrong | A retry after a partial failure is an error though the wanted state holds. The loser of a race sees an error for doing the right thing | Set aside |
| **Success whose text leads with "Nothing was deleted" (recommended)** | The Pack maps `Ok(false)` to a normal reply | Idempotent, retry safe and race safe. It is `Access::remove`'s contract, and OpenCode's core rewrite returns `existed: false` the same way | A mistyped path can read as done if the model skims. The wording has to prevent that | Recommended |

*Table 5. Two ways to answer a delete of a path that is not there. Sources: [tool-delete.md](tool-delete.md), Decision 4; `Access::remove` in the VFS crate.*

The evidence for the recommendation is thinner than the evidence report's high rating suggests. One harness, OpenCode, returns success with `existed: false`, and that code is its V2 core and `dev` branch, which the legacy tool in the session loop does not use `(source)`. The other support is Runvil's own contract. The evidence against is every typed and patch route that fails. The rest of the case is the second criterion: a replayed or retried delete should not turn a reached goal into an error. The wording carries the risk, which is why the behavior below fixes the first words of the reply. Confidence is medium. A plan edit follows from it, because the plan's todo text says the opposite.

### Add no stale-file guard, because Cursor's protects an undo snapshot that Runvil does not take

Cursor compares `dev` and `ino` from two `stat` calls around its undo read `(daemon)`. Runvil's `Stat` has no identity field `(source)`. Table 6 lists the four ways to guard. The do-nothing option wins here because the guard's job disappears with the snapshot.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| **No guard (recommended)** | Nobody. The claim system covers races inside the VFS | Simplicity. Nothing that looks like a guard and is not | Out of process replacement on the real backend between `stat` and `remove` can make the reply's size describe the earlier file | Recommended while there is no snapshot |
| Identity (`dev`, `ino`) on `Stat`, as Cursor | A new optional `Stat` field, filled by the real backend | Catches replacement on real filesystems. Probed to work on NTFS | Memory and virtual backends have no identity. A fake identity that always compares equal looks like a guard and is not. The window between the second `stat` and the unlink stays open | Set aside until a record or snapshot needs it |
| Content fingerprint (size plus a hash of the bytes read) | The Pack | Works on any backend that can read | Needs the read that the read-nothing behavior removes | Set aside with the snapshot |
| Atomic remove against a verified identity | The backend | Closes the window | A large backend request that most operating systems do not offer portably | Set aside |

*Table 6. Four ways to guard against a file changing before the delete. Sources: [tool-delete.md](tool-delete.md), Decision 5 and Q9; inode probes; Runvil `claims.rs`.*

Cursor is the only harness of the thirteen with an identity check at all, and its purpose is to ensure the snapshot it read matches the file it unlinks `(daemon)`. Without a snapshot there is nothing to protect, and the evidence report's own rule applies: skip the check when you did not read the file `(source)`. In Runvil, a write by another live scope between a read of the path and the remove surfaces as `VfsError::Conflict`, because claims from another live scope always conflict `(source)`. The check that remains open is out-of-process change on the real backend, and a delete of a path the model has just replaced is bounded: without `recursive`, the call removes one name or one empty directory. Confidence is medium. This choice is the one most likely to be reopened if a record or snapshot is added.

## Behaviors to implement

Fourteen behaviors, ordered by how much damage a wrong choice does. Each opens with the recommendation, then the reasoning, the evidence, the concrete failure without it, a test, and a confidence word.

### Ship `Delete` as a typed tool, and put any future shell `rm` behind the same gate

**Recommendation.** Implement `std/rm` (wire name `Delete`) as a typed tool over `Access::remove`. When a shell Pack arrives, make its `rm` call `Access::remove` as well, or have it refuse `rm` on paths the VFS governs.

**Why.** A path is a checkable target and a command string is not. The tool also meets VFS fit for free: `Access::remove` already runs the mode policy, takes a claim and fires the event sink, so Ask mode, Plan mode, canonical claims and an audit event arrive with no Pack code. Two routes of unequal strictness leave the weaker one as the real gate, which is why the shell clause matters.

**Evidence.** The costs of the shell and patch routes are measured: pi and Cline deleted outside the project `(probe)`, pi's exit flag hid a failed `rm` `(probe)`, a timed-out pi `rm -rf` left 14,845 of 20,000 entries unreported `(probe)`, and pi's example gate missed `rm -fr`, `find -delete`, `git clean -fdx`, `unlink` and all PowerShell `(probe)`. Pi's sibling example guards `write` and `edit` and not `bash`, so `rm .env` passes while a write to `.env` is blocked `(source)`. Cursor ships the tool and a shell classifier together, and its classifier prompt calls recursive force deletion of a non-regenerable tree "an irreversible high-impact mechanism" `(bundle)`. Rig's library `delete` beats rig's shell route on footprint, confinement and preview `(source)`. Ten of thirteen harnesses ship no tool, and Claude Code's classifier is not in the extracted sources, so the counter-evidence is unmeasured `(source)`.

**Without it.** With no tool and no shell, the model cannot remove a file it created and the run stops for a human. With a shell and no shared gate, `rm` succeeds on a path that `Delete` refuses in Plan mode.

**Test.** Through `Pack::call`, `Delete {path: "/a.txt"}` on a seeded file removes it, and a following `Read` answers not found. In `Mode::Ask` the same call is refused with text containing "needs user approval" and the file is still present. In `Mode::Plan`, `/data.bin` is refused and `/notes.md` is removed. When a shell Pack exists, add the same three cases through its `rm`.

**Confidence.** High, because the five costs are measured on the routes it replaces and Runvil has no other route.

### Refuse a directory that has entries unless the call sets `recursive: true`

**Recommendation.** Add an optional boolean `recursive`, false by default. A file, a link or an empty directory goes without it. A directory with entries answers a refusal that names `recursive` and says it removes everything under the directory. With `recursive: true`, call `Access::remove(path, true)`.

**Why.** File survival ranks first, and a tree is the one target no later call can repair. Putting the flag in the arguments makes the dangerous case visible to whatever approves the call (Checkable target). An empty directory goes without the flag because Runvil has no shell, so a flat refusal would leave a model unable to remove a directory it emptied.

**Evidence.** Cursor refuses every directory, empty ones included, with `Path is not a file (directory): <path>` `(bundle)`. Rig and the patch routes of Codex CLI, Cline and OpenCode refuse them too, and Aider's `unlink()` raises `(source)`. Zed removes a tree with no flag, and the model is told only in the doc comment and never in the arguments `(source)`. Codex CLI keeps its structured path to one file with `recursive: false` `(source)`. In Runvil, an empty directory removes with `remove(path, false)`, a directory with entries answers `DirectoryNotEmpty` and stays intact, and `remove(path, true)` takes the subtree `(source)`. Cursor's own classifier prompt draws the same line: removing an empty directory is "ordinary cleanup" `(bundle)`.

**Without it.** A flat refusal leaves directories permanent. Implied recursion lets `{path: "src"}` take the source tree under an approval prompt that shows one path.

**Test.** Seed `/d/a.txt`. `Delete {path: "/d"}` answers text containing `recursive`, and `/d/a.txt` still reads. `Delete {path: "/d", recursive: true}` removes both. An empty `/e` goes with no flag. `recursive: "yes"` fails the argument check as a wrong type.

**Confidence.** Medium, because no source measures how models use a recursive flag on a delete tool and the flag departs from Cursor's schema.

### Remove the name at the path and never follow a link

**Recommendation.** Stat the path without following the leaf. Remove the name through `Access::remove`. When the type is `Symlink`, say a link was removed and that its target was not touched. Never canonicalize the leaf and never read through it.

**Why.** File survival and Loud failure both turn on the reply naming the object that went. A delete that follows the leaf removes a file the model did not name, and a reply that echoes the typed name hides it.

**Evidence.** Cursor classifies with `stat`, which follows, and unlinks the name, so its undo record described a file that still existed, and its gate judged the target while the unlink removed the link `(bundle)`. A dangling link is refused with `Permission denied: <path>` `(daemon)`. Rig's `Local` workspace follows the leaf: `delete link` removed the target, left the link dangling and replied `Deleted link.`, and a dangling link or junction could not be deleted `(probe)`. Rig's remote script removes the link instead, so rig disagrees with itself `(source)`. Runvil's real backend removes a link as a link and its trait says so, and `Access::stat` returns `FileType::Symlink` for the leaf `(source)`.

**Without it.** The model deletes `latest.md`, a link to `v3.md`, and `v3.md` is gone while the reply says `Deleted latest.md`. Or a dangling link stays for the life of the tree.

**Test.** On the real backend in a temporary directory, make `target.txt` and a link to it. `Delete {path: "/link"}` removes the link, leaves `target.txt` with its bytes, and the reply says a link went. Repeat with a link to a missing path and expect success. Gate the file-link case on `#[cfg(unix)]`, since rig's probe could not create a file symlink on Windows (os error 1314) and used a junction.

**Confidence.** High, because the junction probe and Cursor's mismatch show the cost of both rivals, and the backend already behaves this way.

### Promise no undo, and say so in the description

**Recommendation.** State in the description that the delete is permanent and moves nothing to a trash. Take no snapshot. Do not mention undo, restore or recovery anywhere the model reads.

**Why.** File survival has a second failure: a recovery promise that does not hold. Runvil's `Access` has no journal and no restore, so any undo the Pack offered would live only in the Pack and cover some files. A snapshot that covers some files without saying which is worse than none. Nothing but bytes could come back in any case: mode bits, owner, timestamps and extended attributes go with the inode, and `Access` has no call that restores them.

**Evidence.** Cursor's description never says the delete is permanent, and a model reading "the file cannot be deleted" as one of three graceful failures has no reason to think there is no undo `(bundle)`. Its snapshot skips files over 50 MiB (52,428,800 bytes), skips any file that fails to decode as text, drops the BOM and rewrites CRLF to LF, and the success text is the same in every case `(daemon)`. Zed's reply `Deleted {path}` does not say that ignored files, binary files, files of 6 GiB or more and empty directories have no undo, and its trash call `trash_file` is never made `(source)`. Copilot's trash and chat edit session are the editor's `(source)`. Aider's `/undo` refuses when the file is dirty or was pushed `(source)`. Zeroclaw's prompt only suggests `trash` `(source)`.

**Without it.** The model deletes a file it should have copied, believing a restore exists. Or a later change adds a text-only snapshot and the model comes to rely on it for files it does not cover.

**Test.** A spec test that `rm.json`'s description contains the word "permanent" and does not contain "undo", "restore" or "recover". An integration test that after a successful `Delete`, `Read` of the path answers not found and no sibling path appears in a `glob` of the root.

**Confidence.** Medium, because it holds only until the VFS gets a journal, and that decision is outside this tool.

### Answer an absent path with a success that opens with "Nothing was deleted"

**Recommendation.** Call `stat` first. When it finds no path, or when `remove` returns `Ok(false)` for a path that vanished since the `stat`, return a normal reply, not an error: `Nothing was deleted: {path} does not exist.` Answer an absent path before `remove` runs, so no mode check and no approval request happens for a path that is not there. Do not begin the reply with the word `Deleted`.

**Why.** The first words carry the outcome, so a skimming model cannot read an absent path as a finished delete. Nothing changes when nothing is there, so no prompt should appear for it. Replay and retry both want idempotence: a second delete of the same path should give the same text as the first absent answer, and a delete that already did its job should not turn into an error. `Access::remove` already makes that promise, and the plan's mapping to a not-found result would undo it.

**Evidence.** Cursor fails with `Error deleting file : File not found: <path>` while its description promises to fail gracefully, and the loser of a two-agent race sees a raw `ENOENT` with the prefix doubled `(bundle)`. Zed answers `Couldn't delete {path} because that path isn't in this project.` for a missing path, a broken link and an ignored directory alike `(source)`. Rig answers `{resolved}: no such file` `(source)`. Aider's missing target is fatal at parse time and a warning at apply time `(source)`. Cline's `fs.rm(..., {force: true})` turns a vanished file into a success `(source)`. OpenCode's core `FileMutation.remove` succeeds with `existed: false` under a keyed mutex on the canonical path `(source)`. The `Access::remove` documentation says removing the same path twice succeeds `(source)`.

**Without it.** A retry after a partial failure reports an error although the wanted state holds, and the model spends turns repairing a tree that is already right. A reply that began with `Deleted` over an absent path would be the silent wrong result the criteria forbid.

**Test.** On an empty VFS, `Delete {path: "/gone.txt"}` returns a non-error reply equal to `Nothing was deleted: /gone.txt does not exist.` Delete a seeded `/a.txt`, then delete it again, and expect the first reply to start `Deleted` and the second to equal `Nothing was deleted: /a.txt does not exist.` No absent-path reply may start with `Deleted`. In `Mode::Ask`, `Delete {path: "/gone.txt"}` of an absent path answers `Nothing was deleted: /gone.txt does not exist.`, and the same call on a present path answers the approval refusal.

**Confidence.** Medium, because one non-shipping codepath and Runvil's own contract support it and the typed and patch routes elsewhere all fail.

### Keep the policy's reason and render every refusal in one place

**Recommendation.** Turn each `VfsError` into one sentence with the shared `refusal` function, `refusal(tool, given_path, &VfsError) -> String` in `plugin-std`, and write no renderer of Delete's own. The function carries the variant's own `reason` for `PermissionDenied` and `ApprovalRequired`, and its own `detail` for `OutsideRoot`. It drops the VFS text's leading label for `ReadOnly` and `RootProtected`, writes `I/O failure on {path} ({kind}): {message}` for `Io`, and ends in an arm that uses `Display`, so a variant added later still reads as text. It names the next step where one exists, such as setting `recursive`. It adds no prefix around a sentence that is already complete. For `Conflict` it writes `{path} is also used by another call in this round (a read, write, glob, grep, list or delete that covers it), so it cannot be removed in this round. Send this call again in your next message.` A failure message names no other tool. The Plan and Ask next-step text lives in the VFS policy reason and not in this Pack: the todo `vfs-mode-text` changes `ModePolicy` in `lib.rs` so that the Plan reason ends `which end in .md (case-sensitive)` and the Ask reason ends `and retrying will not help`.

**Why.** Loud failure needs the model to know the cause. A model told only that permission was denied cannot tell a mode refusal from a read-only mount. One rendering function, shared by all six tools, is what keeps Cursor's doubled prefix from reappearing when real approval lands and a second layer starts wrapping messages, and it gives one sentence per variant across the set. A prompt can offer a subset of the tools, so a failure message that named another tool could send the model to one it does not have. Replay needs a limit: `VfsError::Io` carries the operating system's wording, which varies by host, so name the `kind` before the message.

**Evidence.** Cursor's serializer keeps `path` and `isReadonly` for a permission refusal and drops `clientVisibleError`, so `Blocked by cursor ignore`, `Blocked by permissions configuration` and the dangling-link sentence all reach the model as `Permission denied: <path>` `(bundle)`. Four of the seven result cases (not found, not a file, busy and the generic error) render as `Error deleting file : <message>`, with an empty slot before the colon, and the empty-path case doubles it: `Error deleting file : Error deleting file : path is empty or only whitespace` `(bundle)`. An empty rejection reason renders `File deletion rejected: File deletion rejected` `(bundle)`. Pi's hook passes a block reason to the model verbatim, and a probe of its shipped example returned the reason intact `(probe)`. Runvil's `PermissionDenied` and `ApprovalRequired` each carry a `reason`, and `Mode::Ask` words it today as `Delete on /a.txt needs user approval: the Ask mode refuses all mutations` `(source)`.

**Without it.** The model retries a Plan-mode denial as if it were transient, or asks the user to change file permissions for a mode refusal.

**Test.** A table-driven test over `DirectoryNotEmpty`, `PermissionDenied`, `ApprovalRequired`, `ReadOnly`, `RootProtected`, `OutsideRoot`, `Conflict` and `Io`, asserting each rendered string against a literal, that each carries its variant's reason, that none contains `Error deleting file`, and that none names `Read`, `Write`, `StrReplace`, `Glob` or `Grep`. The `Conflict` case asserts the sentence above byte for byte. Run it end to end through `Pack::call`. Cursor's lesson is that a test on the thrown text passes while the real tool emits something else.

**Confidence.** High, because the loss of reasons and the doubled prefixes are observed in the shipped build and Runvil's variants already carry what Cursor drops.

### Reply with one line that names what went: kind, path and size

**Recommendation.** Reply `Deleted /a.txt (34 bytes).` for a file, with `(1 byte)` and `(0 bytes)` in their singular and zero forms. Reply `Deleted /e (empty directory).` for an empty directory, `Deleted /l (link). Its target was not touched.` for a link, and `Deleted /p (special file).` for a pipe, socket or device. Take the size from the one `stat` made before the remove. Build the plural with the shared `plural(n, noun)` helper in `plugin-std`. A directory removed with `recursive: true` replies `Deleted /old and everything under it.` and gives no size. The path in every reply and message is the argument as the model wrote it. Return the reply as trusted text, because it holds only the caller's path, a size and fixed words.

**Why.** Loud failure: the size lets the model catch a wrong target, since a 4 GB file where a note was expected shows in the reply. Replay: `stat` size is the same on every run. Bounded cost: one line. A directory's `size` field says nothing about what it holds, so the reply gives none.

**Evidence.** Cursor replies `Successfully deleted file: <path> (<n> bytes)` with the count from the first `stat`, and has no singular, so one byte reads `(1 bytes)` `(bundle)`. Zed replies `Deleted {path}`, rig `Deleted a.txt.`, Codex CLI and OpenCode `D <path>` under a `Success. Updated the following files:` header, Cline `gone.txt: [deleted]`, Aider `Deleting {path}` `(source)`. Only Cursor reports a size. Rig's reply echoes the typed name over a followed link, which is why the link wording above matters `(probe)`.

**Without it.** A bare `Deleted` gives the model nothing to check. A size taken after the remove would be wrong or absent.

**Test.** A byte-exact table: a 34-byte file gives `Deleted /a.txt (34 bytes).`, a 1-byte file `(1 byte)`, a 0-byte file `(0 bytes)`, an empty directory the directory line, a link the link line, and a directory removed with `recursive: true` the `and everything under it` line. Delete a 34-byte `/a.txt` with the argument `a.txt` and expect `Deleted a.txt (34 bytes).` Run each twice and compare the strings byte for byte.

**Confidence.** Medium, because the size is Cursor's idea alone and its value to the model is judged and not measured.

### Report what remains after a failed recursive delete

**Recommendation.** A successful `remove(path, true)` replies `Deleted /old and everything under it.` and counts nothing. After a failed `remove(path, true)`, count the entries under the directory with a walk of `Access::list`, and say what is left: `Deleting /old failed: <reason>. 5 entries remain.` (`1 entry remains.` for one), where `<reason>` is the shared `refusal` function's sentence for the error. The cap is 10,000 entries, counted as the files, links and directories under the target, with the target not counted. Past the cap the walk stops and the reply says `Deleting /old failed: <reason>. At least 10000 entries remain.` Walk with `Access::list`, which lists every entry, links and hidden paths included. Never walk with `Access::glob`.

**Why.** File survival and Loud failure: after a failed tree delete the model must know what state it is in. Replay: a count is deterministic. Bounded cost: a successful delete pays for no walk, and after a failure the cap keeps the walk from growing with the tree. `Access::list` returns each entry with its stat in one call per directory, so the walk costs about one call per directory `(source)`. A count before the delete would buy only a number in the success text, and the count after a failure is the part that tells the model what state it is in. The memory backend removes a subtree under one tree lock, so it cannot fail halfway, but the real backend's `remove_dir_all` can, and the VFS maps the failure to `Io` with no count `(source)`. A walk through `Access::glob` would undercount, because a glob keeps only files or only directories and Glob's ignore rules hide paths from its listing, while `remove(path, true)` removes every entry.

**Evidence.** Pi's `rm -rf` of 20,000 files left 14,845 entries under a 0.4 s timeout and 14,725 after an abort, and the model saw only the timeout or abort text `(probe)`. Zed's `Deleting {path}: {e}` does not say what was already removed `(source)`. Zeroclaw's purge counts a file before it deletes it and never names what went `(source)`. Only the patch routes list what they did `(source)`. No harness reports a count of a recursive delete, so there is no precedent for the cap.

**Without it.** After a failed tree delete the model may believe nothing happened, or that everything did, and build on the wrong state.

**Test.** Seed 12 files under `/old` in the memory VFS and expect the reply `Deleted /old and everything under it.` On the real backend under `#[cfg(unix)]`, make one nested file undeletable and expect a failure reply whose remaining count is above zero. Seed one undeletable file directly under `/old` and no other entry, and expect `1 entry remains.` Seed 10,001 entries under `/old`, make the delete fail, and expect `At least 10000 entries remain.`

**Confidence.** Low, because the hazard is measured on pi and Zed but no harness reports a count, and the mechanism is untested in Runvil.

### Read no content: one `stat` and one `remove` per call

**Recommendation.** Make at most one `stat`, for the kind and size, and one `remove`. Read no bytes, decode nothing and hash nothing.

**Why.** Bounded cost: the work stays constant whatever the file size. The encoding, line-ending and binary questions that a content read raises (the evidence report's Q3, Q4 and Q5) do not arise. Nothing on the delete path can fail because a file would not decode.

**Evidence.** Cursor reads up to 50 MiB (52,428,800 bytes) before every delete, solely for undo, and throws the snapshot away when the agent already touched the file `(daemon)`. The evidence report estimates 150 to 250 MiB of transient memory for a 50 MiB file `(source)`. Rig reads up to 16 MiB, and over a remote transport that read crosses the wire `(source)`. Codex CLI reads up to 512 MiB twice `(source)`. Cline and OpenCode read the whole file only to prove it exists or to draw a diff, and Cline reports a directory or a permission error as `File not found` `(probe)`. A path, a kind and a size from `stat` are all the decision needs `(source)`.

**Without it.** Deleting a 2 GiB log reads 2 GiB first. A read error on a file the model may legitimately delete aborts or mislabels the delete.

**Test.** Run `Delete` on a large seeded file through a backend that logs operations, as the VFS crate's test support does, and assert the log holds no `Read` and exactly one `Delete`. Delete a file that is not valid UTF-8 and expect success.

**Confidence.** High, because the cost is measured across four harnesses and nothing in the recommended design needs the content.

### Claim the canonical path and add no lock of the Pack's own

**Recommendation.** Hand the path to `Access::remove` as the model gave it. `gate` canonicalizes before it claims, so the claim key is the canonical path. Add no lock in the Pack keyed on the argument string. A sibling `Glob`, `Grep` or list that covers the path, or a sibling write under a directory removed with `recursive`, conflicts in either order until the run's scope ends. The refusal sentence names the path the VFS reports.

**Why.** File survival and Replay both need two spellings of one file to serialize. A lock keyed on the raw string fails at its only job. The VFS claim covers more than a per-turn lock can, because it spans scopes and a recursive delete claims the whole subtree.

**Evidence.** Cursor's lock is keyed on the raw argument, so `C:\a\b.txt` and `C:\a\.\b.txt` take different locks, and its scope is one agent turn `(bundle)`. Zed and rig take no lock, and Zed runs the calls of one message concurrently `(source)`. OpenCode's core rewrite removes under a keyed mutex on the canonical path `(source)`. Runvil canonicalizes before it consults the policy "so a denied operation never registers a claim and every claim key is the canonical path", and a claim holds for its scope's whole life `(source)`.

**Without it.** Two spellings of one path race, and only a stale-file guard would catch it. A second lock in the Pack adds a second ordering to reason about next to the claims.

**Test.** Within one scope, read `/a/b.txt` and then delete `/a/./b.txt`, and expect success. From a second live scope that has read `/a/b.txt`, delete `/a/./b.txt` and expect the `Conflict` sentence of the refusal behavior. Confirm the exact conflict rule against the claims ledger when writing the test.

**Confidence.** High, because the behavior already exists in `Access` and the Cursor alternative is a documented bug.

### Refuse an empty, whitespace-only or root path before any I/O

**Recommendation.** In the Pack, refuse a `path` that is empty after trimming, with the one blank-path helper that the four tools with a required `path` share: ``std_Delete: `path` is empty or only whitespace, and the call passed "{value}"``. Leave the root and anything above it to the VFS, which answers `RootProtected` or an invalid-path error.

**Why.** Loud failure: a dropped argument arrives as an empty string, and in the VFS a whitespace-only string is a legal file name. Without a Pack check, a blank path would reach the backend and come back as a missing file, or as a delete of a file named with spaces. Each policy sits in the layer that owns it: text rules in the Pack, root protection in the VFS.

**Evidence.** Cursor's first statement is `path.trim() === ""`, which refuses before any I/O `(daemon)`. Zed has no guard I could find for `.`, the empty string or the root's name, and its project panel guards the worktree root where the tool does not (an inference, not run) `(source)`. Rig refuses the root as a directory `(source)`. Runvil's `canonicalize` refuses an empty path as `invalid path "": the path is empty` and does not refuse `"   "` `(source)`. Both backends answer `RootProtected` for `/`, and a VFS test pins it `(source)`.

**Without it.** A model that drops its argument gets a `Nothing was deleted` reply over a blank name and may read it as a tidy no-op.

**Test.** `Delete {path: ""}` refuses with ``std_Delete: `path` is empty or only whitespace, and the call passed ""``, and `{path: "   "}` with the same sentence ending `passed "   "`. `Delete {path: "/"}` answers the root-protected sentence, and so does `{path: "/", recursive: true}`. Check that `{path: "."}` resolves to the same root refusal.

**Confidence.** Medium, because the root rule is already tested in the VFS and the whitespace rule is my judgment from Cursor's trim.

### Take one path per call

**Recommendation.** Keep the schema to one string `path` and the one boolean `recursive`. Accept no array and expand no glob. Refuse any other parameter, naming it, with the shared `refuse_unknown` helper in `check.rs`: ``std_Delete: `{name}` is not a parameter of this tool. The parameters are `path` and `recursive`.``

**Why.** File survival and Checkable target: each approval is about one thing, and the reply never has to carry partial success. Pagination and chunking (the evidence report's Q1) do not arise: one path produces one line. The shared argument check passes a key the schema does not declare, so the body refuses it. A misspelled `recursive` would otherwise be ignored, and the refusal that followed would tell the model to set a flag it believes it set.

**Evidence.** Cursor, Zed and rig take one path `(source)`. The patch routes take many and have partial failure: Codex CLI, OpenCode and Aider verify every target and then apply in order with no rollback, and OpenCode's source carries a TODO for multi-file transactions `(source)`. Copilot's parser fails the whole patch before any edit applies when one file is missing, which is atomic for that one check only `(source)`. A prompt that says "delete these five things" is skimmed, which is the evidence report's judgment and not a measurement `(source)`.

**Without it.** A batch whose third path fails leaves two deleted and needs an output shape for partial success.

**Test.** Assert the schema's `properties` are exactly `path` and `recursive`. An array passed as `path` fails the argument check as a wrong type. `Delete {path: "/a", force: true}` answers ``std_Delete: `force` is not a parameter of this tool. The parameters are `path` and `recursive`.`` Two deletes in one turn each produce their own reply and their own event.

**Confidence.** High, because every batch route studied has partial failure and the model can call the tool five times.

### Add no stale-file guard until `Stat` carries identity

**Recommendation.** Compare nothing between the `stat` and the `remove`. Document that the claim system covers races inside the VFS, and that an out-of-process replacement on the real backend can make the reply's size describe the earlier file.

**Why.** Cursor's guard exists so the snapshot it read matches the file it unlinks. The read-nothing behavior removes the read and the no-undo behavior removes the snapshot, so nothing is left to protect. A guard on a field some backends leave empty is worse than none, because it looks like protection.

**Evidence.** Cursor compares `dev` and `ino` from two `stat` calls and treats a failed second `stat` as a mismatch, and its own report says the window between the second `stat` and the unlink stays open `(daemon)`. On NTFS the pair is stable and changes on recreate `(probe)`. No other harness compares identity: rig has `write_if` and no `remove_if`, Cline's `force: true` hides a vanished file, and Aider checks `exists()` and then calls `unlink()` `(source)`. `Stat` has no identity field, and a write from another live scope already conflicts with the remove `(source)`.

**Without it.** If this behavior is skipped and a guard is added now, a comparison of two `None` identities passes every time and reads as protection.

**Test.** From a second live scope, write the path after the first scope has read it, and expect the delete to answer `Conflict` from the claims and not from any Pack check. Assert the Pack source holds no identity or fingerprint comparison.

**Confidence.** Medium, because the choice follows from leaving out the snapshot and reopens if a record is added.

### Write the description from the behavior

**Recommendation.** Replace Cursor's text in `rm.json`. Say the delete is permanent, that a link goes without its target, that a directory with entries needs `recursive`, and that an absent path is not an error. Say that Delete removes any path the call names, `.git` and ignored paths included, and that a recursive delete removes the ignored and hidden entries under the directory. Keep "absolute" in the `path` text. Drop "fail gracefully". Record each edit in `EDITS.md`, which now says the file is verbatim.

**Why.** Loud failure starts before the call: the description is the one place the model learns the rules. Every file tool accepts a relative path (joined onto `/`) and advertises the absolute form, because Glob and Grep print it. Ignore rules and `.git` hide entries from Glob and Grep listings only, and no privacy rule exists in version one, so the description says what Delete removes.

**Evidence.** Cursor's text says `The absolute path of the file to delete` while its resolver takes either form, and promises to fail gracefully on a missing file while the real path is a thrown error flagged `isError` `(bundle)`. It omits that directories are refused and that the delete is permanent `(bundle)`. Zed's doc comment says "and the directory's contents, recursively" `(source)`. Rig's description reads `Delete a file, not with \`rm\` in \`shell\`. Directories are refused.` `(source)`. The proposed wording is in Mapping to Runvil.

**Without it.** Models send directories to a tool that refuses them, assume an undo, and expect an error for an absent path that the tool now answers with success.

**Test.** A spec test over `rm.json`: the description contains "permanent", "recursive", "link" and ".git" and does not contain "gracefully"; the `path` text starts "The absolute path"; `required` is exactly `["path"]`; `recursive` has type boolean. The existing spec tests already refuse descriptions that promise behavior the build lacks.

**Confidence.** High, because each omission and each false promise in Cursor's text is documented, and the fix costs only wording.

## Where Runvil should differ from Cursor

Table 7 lists where the Runvil tool should depart from Cursor's `Delete`. Rows 1, 2, 13 and 14 are machinery that exists in Cursor only because of its JavaScript tool wrapper or its proto layer, and they are dropped and not reproduced. On the doubled prefix, the parent spec's author wrote "I would fix it and note the divergence, since nothing depends on the malformed string", as quoted in [tool-delete.md](tool-delete.md), Decision 3.

| # | What Cursor does | What Runvil should do | Why |
|---|---|---|---|
| 1 | Throws a finished sentence, serializes the error into a proto record that drops the path, then renders it again, so four of seven cases read `Error deleting file : <message>` | Render each refusal once, with the shared `refusal` function and no prefix over a finished sentence | There is no serialization round trip to lose fields. Loud failure |
| 2 | Drops the gate's reason, so every rule reads `Permission denied: <path>` | Carry the `reason` of `PermissionDenied` and `ApprovalRequired` into the reply | The model needs the cause to act. Loud failure |
| 3 | Fails on an absent path with `Error deleting file : File not found: <path>`, though its description says to fail gracefully | Succeed with `Nothing was deleted: ...` | Idempotence and retry. `Access::remove` returns `Ok(false)` |
| 4 | Refuses every directory, empty ones too | Remove an empty directory freely, refuse one with entries unless `recursive: true` | Runvil has no shell to fall back to |
| 5 | Classifies with `stat` (follows the link) and unlinks the name. A dangling link is refused | Stat without following, remove the name, and remove a dangling link | The reply names what went, and the backend already behaves so |
| 6 | Reads up to 50 MiB for a text-only undo snapshot that strips the BOM and rewrites CRLF to LF | Read nothing, and promise no undo | The snapshot has silent holes and no consumer. Bounded cost |
| 7 | Compares `dev` and `ino` around the read | Compare nothing | The guard protects the snapshot, and `Stat` has no identity field |
| 8 | Locks on the raw argument string for one turn | Claim the canonical path through `Access` | Two spellings serialize, and claims span scopes |
| 9 | Writes `(1 bytes)` and has no singular | Write `(1 byte)` | A plain wording fix. The byte count itself stays |
| 10 | Reports a fifo, socket or device as `Path is not a file (other)` and refuses it | Remove the name and call it a special file | `unlink` removes only the entry, and no shell exists for the fallback. A judgment: Cursor and rig both refuse |
| 11 | Resolves any absolute path, and its out-of-workspace verdict falls through `onNeedsApproval: async () => null` | Let `canonicalize` and the rooted backend refuse a path outside the root | Containment is in the VFS. Cursor's resolver has none |
| 12 | Asks for approval before the `stat`, so a prompt can appear for a path that does not exist | Call `stat` first and answer an absent path with `Nothing was deleted` before `remove` runs, so no mode check or approval request happens for a path that is not there | The mode policy runs inside `Access::remove`, which an absent path never reaches |
| 13 | Ships two surfaces by model branch: `delete_file` with `target_file` and an `explanation` that is never read, and `Delete` with `path`; a one-line description for two Codex branches | One name `Delete`, one schema, one description | Version machinery for a proto layer and per-model naming. Runvil serves one surface |
| 14 | Converts the size from `BigInt` to `Number` and forces `isError` in a wrapper | Report a `u64` size and let the Pack's error result carry the flag | JavaScript-only machinery |
| 15 | `ApplyPatch` refuses `*** Delete File:` and names `delete_file`, even on branches where the tool is `Delete` | Not applicable: Runvil has no patch tool. If one is added, build the tool name in the message from the registered name | A hard-coded name points at a tool the model cannot call |

*Table 7. Fifteen departures from Cursor's `Delete`. Sources: [tool-delete.md](tool-delete.md), sections The short version, Decisions 2 to 9 and The contract.*

Confidence for each row is the confidence of the behavior it names.

## Mapping to Runvil

Table 8 maps each behavior to the `Access` method that covers it. Vinnie owns every row. The next step is a todo id from the standing plan in `runvil/vibe/2026-10-27-2-plugin-std-file-tools.md`, or an id marked new, which is a proposal. Rows 15 to 17 are gaps that no behavior closes by itself.

| # | Behavior | `Access` method | Status | What the gap needs | Next step |
|---|---|---|---|---|---|
| 1 | Typed tool | `remove`, `stat` | Covered | The Pack replaces the stub. A future shell must call `Access` too | `delete-real`, and `shell` for the clause |
| 2 | Recursive flag | `remove(path, true)` | Partial | The schema gains `recursive`. `Policy::check` takes one path, so Plan mode judges only the top path of a tree (row 16) | `delete-real` |
| 3 | Remove the name | `stat`, `remove` | Covered | `Access` has no `read_link`, so the reply says the target was untouched and cannot name it | `delete-real` |
| 4 | No undo | none | Covered by omission | Description wording only. A journal is a separate VFS feature | `delete-real` |
| 5 | Absent path | `remove` returns `Ok(false)`, `stat` returns `NotFound` | Covered | The plan's todo text says to map `Ok(false)` to the not-found result and must change | `delete-real` |
| 6 | Refusals | the `VfsError` variants, `gate` | Partial | The shared `refusal` function. The Plan and Ask next-step text in the VFS policy reason in `lib.rs`. Real approval and the operator flow later | `refusal-text` and `vfs-mode-text` (new); real approval waits on `approval` |
| 7 | One-line reply | `stat` | Covered | A `FileType` to kind-word table and the shared `plural` helper | `delete-real` |
| 8 | Remaining count | `list` | Partial | After a failed delete, a bounded walk of `Access::list` with a cap of 10,000 entries. Walk with `list`, which lists every entry, links and hidden paths included, and never with `glob` | `delete-real` |
| 9 | Read nothing | `stat`, `remove` | Covered | None | `delete-real` |
| 10 | Canonical claim | `gate`, `claim_write`, `claim_subtree` | Covered | None. Add no Pack lock | `delete-real` |
| 11 | Empty and root paths | `canonicalize`, `RootProtected` | Partial | `"   "` is a legal name, so the Pack trims and refuses with the shared blank-path helper | `delete-real` |
| 12 | One path | the schema | Covered | `rm.json` gains `recursive` and nothing else. The shared `refuse_unknown` helper in `check.rs` refuses any other key | `delete-real`, with the helper from `read-real` |
| 13 | No guard | none | Covered by omission | `Stat` has no identity field. Reopen if a record or snapshot is added | `delete-real` |
| 14 | Description | `rm.json` | Missing | Rewrite the text, add `recursive`, update `EDITS.md` | `delete-real` |
| 15 | One event per tree | `fire(Op::Delete, path)` | Partial | A recursive removal fires once for the top path. A host that audits per file needs a VFS change, which the plan does not list | `delete-real` documents it |
| 16 | Plan mode over a tree | the mode policy | Missing | In `Access::remove`, when `recursive` is set, gate `Op::Delete` on every entry under the path before the claim, so Plan refuses `/x.md/a.bin` and names it. The Pack cannot refuse `recursive` under Plan, because it cannot see the mode. No plan todo covers it | `vfs-plan-subtree` (new), with `delete-real` pinning today's behavior until it lands |
| 17 | Shell gate | none | Missing | A shell `rm` must reach `Access::remove` | `shell` |

*Table 8. Mapping from behavior to Runvil. Sources: Runvil `operations.rs`, `real.rs`, `memory.rs`, `stat.rs`, `error.rs`, `lib.rs` and `claims.rs` in the VFS crate; the `plugin-std` crate and the standing plan.*

### Tests to add to the crate

Each test below maps to a behavior's Test line. Unit tests sit in a `delete-tests.rs` sibling and integration tests in `tests/it/`, following the crate's layout.

- A byte-exact reply table for the file, singular, zero-byte, empty directory, link and `recursive` tree cases, and a relative argument echoed as written, run twice for replay.
- An absent-path test: first reply starts `Deleted`, second equals `Nothing was deleted: /a.txt does not exist.`, no absent reply starts with `Deleted`, and in `Mode::Ask` an absent path answers the same sentence while a present path answers the approval refusal.
- A directory test: refusal names `recursive` and leaves the contents readable. Then the flag removes the tree and the reply is `Deleted /d and everything under it.`
- A failure-count test: the remaining count after a failed recursive delete, `1 entry remains.` for one, and `At least 10000 entries remain.` past the cap.
- A link test on the real backend: link gone, target intact, dangling link removable. Gate the file-link case on `#[cfg(unix)]`.
- A logging-backend test that a delete makes one `Delete` and no `Read`, including a file that is not valid UTF-8.
- A table over `DirectoryNotEmpty`, `PermissionDenied`, `ApprovalRequired`, `ReadOnly`, `RootProtected`, `OutsideRoot`, `Conflict` and `Io`, run through `Pack::call`, with the assertion that no message names another tool.
- Mode tests: Ask refuses with the policy's text, Plan refuses `/data.bin` and removes `/notes.md`.
- A test that pins today's behavior for a recursive delete of a directory named `x.md` under Plan mode, so the change that `vfs-plan-subtree` makes shows when it lands. After it lands, the same call refuses `/x.md/a.bin` and names it.
- A claim test: a second live scope that has read `/a/b.txt` makes `Delete` of `/a/./b.txt` answer the `Conflict` sentence.
- Empty, whitespace-only and root path tests, including the root with `recursive: true`.
- An unknown-parameter test: `force` is refused by name and the sentence lists `path` and `recursive`.
- Spec tests over `rm.json`: `required` is `["path"]`, `recursive` is boolean, the description has the required words and lacks "gracefully", and the `path` text says "absolute".

### Schema and description fixes to make

These are listed and not made here. The crate and its schemas were not changed. `schemas/rm.json` and `schemas/EDITS.md` currently say Cursor's text is kept verbatim.

```json
{
  "wire_name": "Delete",
  "description": "Deletes the file, link or empty directory at `path`. The delete is permanent: nothing is moved to a trash. A link is removed and the file it points to is not touched. A directory that has entries is refused unless `recursive` is true, which removes the directory and everything under it. A path that does not exist is not an error: the reply says nothing was deleted. Delete removes any path the call names, `.git` and ignored paths included, and a recursive delete removes the ignored and hidden entries under the directory.",
  "parameters": {
    "type": "object",
    "properties": {
      "path": { "type": "string", "description": "The absolute path of the file, link or directory to delete." },
      "recursive": { "type": "boolean", "description": "Set true to delete a directory that has entries, together with everything under it. Default false." }
    },
    "required": ["path"]
  }
}
```

Four plan edits follow. First, change the `delete-real` todo text to say that `Ok(false)` becomes a success reply that says nothing was deleted, that a directory with entries is refused unless `recursive` is true, that a failed recursive delete counts what remains with `Access::list`, and that the description says the delete is permanent. Second, record in the plan's Decision Record that `rm.json` is no longer verbatim, so the next Cursor re-snapshot treats `Delete` as a deliberate departure. Third, add the todo ids this report needs: `refusal-text` (the shared `refusal` function, before `read-real`), `vfs-plan-subtree` (the recursive gate in `Access::remove`) and `vfs-mode-text` (the Plan and Ask next-step text in the VFS policy reason). The last two are independent and can land at any time. They change the VFS, which the plan's scope lines rule out, so each needs its own entry in the Decision Record. Fourth, order `delete-real` after `replace-real`, with no wait on `glob-real`, because the count walks with `list`. Confidence in these plan edits is high, because they only make the plan text match the behaviors above. The new todo ids are proposals, confidence medium.

## Risks and open limits

Table 9 lists the risks. Likelihood uses the fixed ladder and rates the event. The confidence column rates the evidence behind that likelihood, and it is a separate judgment.

| # | Risk | Likelihood | Confidence in the evidence | What would change the recommendation |
|---|---|---|---|---|
| 1 | A recursive delete under Plan mode takes non-markdown files, because the policy checks only the top path and a directory named `x.md` passes | Very unlikely | High. I read the single `gate` call in `Access::remove` and the one-path `Policy::check`. I did not run it | Plan mode serving as a boundary for untrusted prompts, or one observed case. Then build `vfs-plan-subtree` (medium, because the choice between gating each entry in `Access::remove` and deleting bottom-up is a design call), and ship Table 2's fallback until it lands |
| 2 | Models set `recursive: true` on every directory call out of `rm -r` habit, so the flag stops being a signal | Roughly even chance | Low. No source measures a recursive flag on a delete tool | Transcripts showing the flag set on the first call with no earlier refusal. Then rely on Ask-mode approval for any recursive call |
| 3 | A skimming model reads `Nothing was deleted: ...` as a finished delete | Unlikely | Low. No transcript evidence either way | A transcript where a model reports a delete after that reply. Then return an error result for an absent path and keep the retry case in the wording |
| 4 | The future shell Pack spawns a process or routes `rm` around `Access` | Roughly even chance | Low. I did not read bashkit | Any shell `rm` that bypasses `Access::remove`. Then all five measured costs apply, the shell needs an explicit grant, and the typed-tool behavior's shell clause becomes a hard requirement |
| 5 | A user loses a file the model should not have deleted, because the tool promises no undo | Roughly even chance | Low. No usage data exists, and the chance is for the early weeks of use | The first loss. Then build the journal and take the typed-record shape from Table 4 |
| 6 | The real backend's `remove_dir_all` fails partway on Windows, for example on an open handle | Unlikely | Low. Cursor's Windows probe used Node, where open files unlinked cleanly, and Rust's standard library was not probed | A probe showing a partial failure. Then the remaining count after a failed delete matters more and moves from low to medium |
| 7 | A host that audits per file sees one `Delete` event for a whole tree | Almost certain | High. The code fires once per `remove` | A host that needs per-file records. That needs a VFS change outside the plan |

*Table 9. Risks, with likelihood and confidence kept apart. Sources: Runvil `operations.rs`, `lib.rs` and `observe.rs`; [tool-delete.md](tool-delete.md), Q8, Q9 and the Zed and pi sections.*

### What the evidence does not settle

Six limits remain, and each could move a recommendation. First, no source measures how often a model needs to delete or how it uses a `recursive` flag, so the case for the tool rests on the Checkable target argument and on Runvil having no other route. Second, Claude Code deletes through `Bash` behind a classifier and sandbox whose rules are not in the extracted sources, so the strongest counter-example to the five costs is unmeasured. Third, the plan does not say whether a bare `tools.offer("std")` includes `Delete`. If it does, the operator-grant option in Table 1 needs a deliberate step in each prompt that does not want deletion. Fourth, a grep for the word `structured` in `runvil-plugin/src` found nothing, and the stub tests assert that no tool reports structured output. Whether a Pack can attach a typed record to a reply is not checked here, and it matters only when a journal exists. Fifth, whether Cursor prompts the user before an out-of-workspace delete cannot be read from its bundle, because its gate falls through and the approval provider lies outside it. That limit does not affect Runvil, whose `canonicalize` and rooted backend refuse such a path, and it is recorded so nobody reads Cursor's gate as a model. Sixth, a `Conflict` that the Pack maps to a `ToolError` reaches the model as failure text and does not end the run (`scheduler/tool_call.rs`, read, not run). No two-call round has been run.

Three things would change the main call. If the shell Pack routes `rm` through `Access` and first-run transcripts show runs that never delete, doing nothing becomes the cheaper option and the tool should be offered by grant only. If the VFS gets a journal, the no-undo behavior reverses and the typed record becomes the reply's second channel. If Plan mode becomes a boundary for untrusted prompts, the recursive flag waits for `vfs-plan-subtree`. The recommended default stands on today's evidence: ship the tool, and treat the flag, the absent-path wording and the no-undo promise as the three choices most likely to be overturned.

## Sources

Evidence reports, by file name, all in this folder:

- [tool-delete.md](tool-delete.md), the primary ground: Cursor's `Delete` read from the 3.27.12 bundle and daemon, enriched with twelve harnesses and edited for prose, final on 2026-10-28. Cited by section name.
- [file-tools-index.md](file-tools-index.md), for the shared facts and the parent spec's corrections, including that Delete's `permissionDenied` keeps its path and loses the gate's reason.

Sources behind those reports and used here, each with its tier. Cursor offsets are those the evidence report gives.

| Source | Tier | Used for |
|---|---|---|
| Cursor 3.27.12, `cursor-agent-exec/dist/main.js`, 9,814,287 bytes: `DELETE` registration at 4193302, schemas at 4191981, result oneof at 5959184, `ApplyPatch` refusal at 4199381, Auto-review prompt at 4705128, `recordFileState` at 3405284 | `(bundle)` | Contract, strings, serializer, the classifier prompt |
| Cursor 3.27.12, `cursor-agent-host/dist/agent-host-daemon/dist/bin/daemon.cjs`, 20,213,626 bytes: `LocalDeleteExecutor`, `readText`, `resolvePath`, `shellArgs`, `parseCommandName` | `(daemon)` | Executor order, the 50 MiB ceiling, the guard, the `pi_bash` gate input |
| Inode and unlink probes on Windows 10.0.26300, Node 24.19 | `(probe)` | `dev` and `ino` stability, `EPERM` on a directory, `EBUSY` rarity |
| pi, `git@github.com:earendil-works/pi.git` at `a92283056` (2026-10-27): `bash.ts`, `powershell.ts`, `docs/security.md`, `examples/extensions/permission-gate.ts`; probes of its tool and gate | `(source)` `(probe)` | Exit flag, timeout, gate misses, confinement |
| rig, `git@github.com:0xPlaygrounds/rig.git` at `a83b774d0` (2026-10-25): `crates/rig-tools/src/delete.rs`, `workspace.rs`, `fs.rs`, `remote.rs`, `overlay.rs`; I re-read `delete.rs` lines 65 and 74 to 75; probe of a copy of `Local` | `(source)` `(probe)` | Typed record, leaf-link behavior, registration choice |
| Zed `37faef81f7` (2026-10-27): `crates/agent/src/tools/delete_path_tool.rs`, 25,600 bytes; `worktree.rs`, `project.rs`, `action_log.rs`, `buffer_store.rs`; I re-checked that the tool file never calls `trash_file` | `(source)` | Implied recursion, buffer pass, gating ladder |
| VS Code repository at `b208bc25` (2026-10-25): `extensions/copilot/.../applyPatch/parser.ts:239-248`, `chatEditingSession.ts:596-672`, `bulkFileEdits.ts:249-253`; I re-read line 252 | `(source)` | Copilot's directive, trash and edit session |
| Claude Code 2.1.311 win32-x64 native package: `sdk-tools.d.ts`, 171,918 bytes | `(source)` | No delete tool; the `Bash` schema |
| zeroclaw `2d85371f3` (2026-09-18): `policy.rs`, `shell.rs`, `data_management.rs` | `(source)` | Allowlist, `find -delete` miss, purge |
| Codex CLI `bea47c06df` (2026-10-28): `codex-rs/apply-patch/src/lib.rs`, `exec_policy.rs` | `(source)` | Directive, verification read, forced-`rm` rule |
| Gemini CLI `v0.66.0`, `970413207732`; OpenCode `v1.19.5`, `810c0eba1a73`, `dev` at `fa1260d6ca`; Cline `v4.1.26`, `a459f1d`; Aider `5dc9490bb3`; Goose `ee8500d5e060`, tag `v1.56.0` | `(source)` `(probe)` | Routes, permissions, `FileMutation.remove`, Cline and Aider probes |
| Runvil VFS crate, `crates/runvil-internal/vfs/src/`: `handle/operations.rs`, `handle/access.rs`, `handle/claims.rs`, `real.rs`, `memory.rs`, `memory-tests.rs`, `error.rs`, `stat.rs`, `lib.rs`, `observe.rs`, `traits/backend.rs`, `traits/policy.rs` | `(source)` | `Access::remove`, gate order, claims, backends, error variants, `ModePolicy` |
| Runvil `crates/plugin-std/`: `schemas/rm.json`, `schemas/EDITS.md`, `src/pack-tests.rs`, `tests/it/tools.rs` | `(source)` | The stub, the verbatim claim, the test layout |
| Runvil `vibe/2026-10-27-2-plugin-std-file-tools.md` | `(source)` | Todo ids, non-goals, the `delete-real` wording |

All Runvil files were read from the working tree. No Runvil code or schema was changed.

*2026-10-27 - Claude Sonnet 5.5*
