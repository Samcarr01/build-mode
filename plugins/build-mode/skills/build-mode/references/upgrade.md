# Upgrade

An existing project is running an old copy of the scaffold. The doc pack is copied into each repo rather than linked, so improvements to this skill never reach projects already under way. Do this when a project predates a change to `/checkpoint`, or when the user says the loop has stopped behaving as it used to.

**The risk here is not getting it wrong, it is destroying working state.** They are mid-build. Treat every file as guilty until proven to be pure process.

**Automatic upgrade.** SKILL.md starts an upgrade without being asked when the stamp on the first line of `CLAUDE.md` is missing or older than its setup version (Check the folder, step 5). Same order as below, with these changes:

- **Only if all of these hold:** no unsaved work in the folder (step 1); the layout matches what this skill expects (step 2); only the pure-process files, the stamped `CLAUDE.md` lines and, on a live project, the one roadmap task in step 6 would change; no file it would replace in `.claude/skills/`, `.claude/agents/` or `.claude/rules/` looks hand-edited (has lines no template has); and the new `settings.json` would not allow anything the current one does not. If any of those fails, keep the project's version and ask the user in one line instead.
- **Never during Unblock or Undo.** Something is broken; wait for a calm session.
- **Restore point first, always** (step 1).
- **No yes needed.** Skip step 3's "get a yes". Skip step 2's question about the Claude Code side unless `docs/TOOLING.md` does not say whether Claude Code has a browser tool.
- **Report it** in the usual three plain lines, as step 7 does, then carry on with what the user asked for.

This runs against a project with real work in it, so the order below is not negotiable. Survey, then restore point, then plan, then write. Never open by writing a file.

**1. Make a restore point before touching anything.**

```bash
cd "/sessions/<session>/mnt/<project>" && git status --short && git log --oneline -3
```

If the tree is dirty, stop and say so: uncommitted work means an undo would take their own changes with it. Ask them to commit or stash first, or to tell you to go ahead knowing that. If it is clean, note the current SHA in chat before you start. That one line is what makes every step below reversible.

**2. Survey what is actually there, and never write a file whose job is already done.**

This is the guardrail that matters. The failure mode is not that the layout differs, it is that you look for `docs/ROADMAP.md`, do not find it, and helpfully create one - beside the real 400-line roadmap sitting at the repo root. Now there are two, and the project has quietly lost its single source of truth.

So before creating anything, look for it anywhere: `ROADMAP.md`, `PROGRESS.md`, `LEARNINGS.md`, `ARCHITECTURE.md`, `DESIGN.md`, `TOOLING.md` and prompt files at any depth, not just where this skill would have put them. If you find something doing that job, **use that path**. If the layout does not match what you expect and you cannot tell which file plays which role, **stop and ask**. Do not create, do not migrate, do not tidy.

A project that grew its own sensible shape is not wrong. It is the one you are visiting.

Read, do not assume:

- `.claude/skills/*/SKILL.md` - which of the three exist, and does `/checkpoint` already build, click and push, or is it the old notes-only version?
- `.claude/agents/` - do the two subagents exist, and does `CLAUDE.md` still point at `requesting-code-review` instead of the project reviewer?
- `.claude/rules/`, `.claude/settings.json`, `.mcp.json`
- `docs/TOOLING.md` - **this is the record of what is connected.** Accounts, project refs, which MCPs each side has, what is deliberately not set up. Projects from before 2.3 have no **Hosting** section and assume Vercel; if the project runs anywhere else, that gap is why Sync has been reading the wrong tools.
- `docs/ROADMAP.md` and `PROGRESS.md` - where the build actually is, so nothing you say contradicts it

Then check the live picture against it, because the file may be months stale: `ListConnectors` here, and ask them in one line whether the Claude Code side has changed since - particularly whether a browser tool exists, because that decides which `/checkpoint` you can generate.

**3. Say what you will change, and get a yes.** Three short lists: files being replaced, files being merged, files not being touched. They are halfway through a build and the fear is that this eats their work, so showing the blast radius before you act is most of the job.

**4. Replace only the pure-process files.** The three skills in `.claude/skills/`, the two subagents in `.claude/agents/`, `.claude/rules/design.md`, `.claude/settings.json`. These describe how the project is worked on and contain nothing discovered, so a fresh copy is safe. Preserve any project-specific `allow` rules already in `settings.json` rather than flattening it back to the default. When adding the subagents to a project that had none, fill the reviewer's data-isolation line from `docs/ARCHITECTURE.md`, and tell the user Claude Code needs one restart to see them.

   `.claude/` cannot be written over the device bridge, so the new files travel as `setup-claude.sh`, exactly as Kickoff delivers them. The script copies each file, checks every file it installed matches, and only then writes `<!-- build-mode setup: <version> -->` as the first line of `CLAUDE.md`, replacing any older stamp. If the check fails, the old stamp stays and the next session tries again. This holds for an upgrade the user asked for as much as the automatic one. Make the first line of the next prompt `Run bash setup-claude.sh, check it reports every file matching, then delete it.`, after the fresh-branch line in branch mode, and ask Claude Code to note the upgrade in that task's PROGRESS entry. The user still just types `/next`.

   `CLAUDE.md` itself is not pure process. Swap only the lines a release changed, such as the plain-talk bullets in "The person you are working with", and leave every project fact alone.

**5. Merge `docs/TOOLING.md`, never replace it.** It looks like a process file and is not. Accounts, project refs, the **Not available** list and the two-sides inventory are all findings, some of which cost a conversation to establish and cannot be recovered from a template. Add the new sections, correct anything the survey proved wrong, and leave every real value alone. If it has no **Hosting** section, write one from what is actually there (the host's MCP, `CLAUDE.md`'s Deploying section, a one-line question to the user) rather than from the template. If the live site does not show its commit yet, add that as a small roadmap task rather than doing it inside the upgrade.

**6. Never touch the content files.** `ROADMAP.md`, `PROGRESS.md`, `LEARNINGS.md`, `ARCHITECTURE.md`, `DESIGN.md`. These are the project's memory and history. If one is missing a section the current templates have - commit SHAs in PROGRESS entries, the `[@]` status key in ROADMAP, the Layout, Hierarchy, Copy and Screens sections in DESIGN.md - add that section by hand, going forward only. For DESIGN.md that means filling the new sections with real values for this project, not pasting the template; it is the fastest way to lift an older project's screens. Do not backfill, reformat or tidy. A tidied history is a lost history.

   One exception: on a project that is already live, add one task to the top of `docs/ROADMAP.md`, if it is not there already: write the customer A/B test if the project has none and run it, scan every past save for passwords with `gitleaks git --redact --no-banner`, and do the backup practice from `references/branch-mode.md`. The rules in this release protect future work; this task checks the work already done.

**7. Report in the three plain lines**, then hand back. Usually:

```
**What changed:** `/checkpoint` now runs a password scan and an add-on check before it saves, and asks the code reviewer before it commits.
**Anything risky?** No. Your roadmap, progress log and design are untouched.
**What you need to do:** Nothing. Type /next as usual.

---

Restore point is `a4f9c21` if anything looks wrong.
```

**If the project has no `docs/` and no `.claude/`**, this is not an upgrade. It is an existing codebase adopting the system for the first time: read the code to write `ARCHITECTURE.md` from what is actually built, ask where they think they are up to for `ROADMAP.md`, and generate the rest fresh. Say which of it you inferred rather than knew.
