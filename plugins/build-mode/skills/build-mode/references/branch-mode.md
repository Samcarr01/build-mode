# Branch mode - going live

Before launch, every task commits straight to `main`. That is right while nobody is using the thing: fewer moving parts, and a bad push breaks nothing that matters. The day the build has a real user, a client relying on it, or the real domain pointed at it, every push to `main` is a release. From then on the project runs in **branch mode**: every task on its own branch, a preview of it to check, and the user's merge as the one moment it goes live.

What that buys: a pushed branch cannot break production, so `/checkpoint` pushes every task without a decision to make; Sync checks the change on a preview before any real user sees it; and an unmerged pull request is the cheapest undo there is.

## The live line

Planned at kickoff, not discovered afterwards. Waiting for the user to say "it's live" means the first real users can arrive while tasks still go straight to `main`.

**Set it in the roadmap step.** Make the call and show the other two options in the same line, so a yes is all it needs:

- **Live from day one** when the build replaces something people already use, such as an existing site or system. The Go live tasks follow the first deploy in Phase 0, and branch mode starts at M1.
- **Live after the first useful milestone** when the user or their client wants to start early. More feedback sooner, but branch mode, and any preview costs, run for longer.
- **Live at the end** for internal tools and personal builds, and whenever nothing points elsewhere.

It sits at the top of the roadmap beside Worked if:

```
Live line: after M2 (client starts using it)
Now: Building - commits go straight to main
```

- **The Go live tasks are the last tasks of the milestone before the line**, each with a normal ID (`M2-T6 Go live: preview wiring`), so the switch is finished before anyone real arrives. Every milestone after the line carries `(live)` in its heading.
- **Before the line, Claude Code flags and never decides.** The Deploying section of `CLAUDE.md` carries one extra rule: `Before the live line (<milestone>): if a task would make something real early - point the real domain, send to a real address, take a real payment, invite a real user - stop and say so before doing it.` The roadmap belongs to Cowork, and the call is the user's.
- **Sync watches it.** Two tasks from the line, say so in the Sync note and the handover. If Sync sees real use before the line (the domain pointed, real sign-ups, a client already relying on it), offer to move the line forward in one line.
- **Moving the line is a Replan.** Mark the old Go live tasks `~~cut~~ (moved)` and add them again with new IDs in the new milestone, because IDs never move or get reused. Update the `Live line` row.
- **When the Go live tasks are done,** the `Now:` line becomes `Live - branch mode since <date>`.
- **A project from before this has no live line.** Offer one in a single line at its next Sync. Until it has one, "it's live" or "we've launched" still triggers the Go live tasks straight away.

## Going live, once per project

The last four tasks of the milestone before the live line, in this order. Previews do not get switched on until steps 1 and 2 are done and live.

1. **Find out what a preview will touch (Cowork).** Read the Hosting section of `docs/TOOLING.md` and the database row of `docs/ARCHITECTURE.md`, then answer two questions and write the answers in the Hosting section.
   - **Which database does a preview use?**
     - *Database on the host, wired with reference variables* (Railway Postgres with `${{Postgres.DATABASE_URL}}`). A Railway PR environment replicates the base environment's services, variables included, so the preview gets its own copy of the database service and the reference resolves to it. Check every service with the host's MCP (`list-variables` on Railway): a pasted connection string, rather than a reference, is copied as it is and points the preview at the live database. Each pasted value becomes an `[@]` task for the user to swap before anything else.
     - *Database outside the host* (Supabase alongside Railway, or behind Vercel). Previews read the same variables as production unless told otherwise, so they talk to the production database. Either give previews their own database (a second free project, set as Preview-scoped variables on Vercel, or in a `staging` environment that Railway's PR environments inherit from instead of production), or accept it, write that down, and keep migrations out of the build.
   - **Do migrations run on deploy?** If the build or pre-deploy command migrates and a preview uses the production database, the preview changes the live schema before anyone has merged. That combination is never allowed; fix one side of it first.

2. **Block outside actions outside production (Claude Code, the last task committed to `main`).** Previews copy production's keys on most hosts, so a test click in a preview can email a real customer, post to a real page or take a real payment. The code that reaches outside the app (the approved-action helper named in `CLAUDE.md`, where the project has one) and every scheduled job refuse to act unless the host says this is production, and log "skipped: preview" instead. Railway: `RAILWAY_ENVIRONMENT_NAME` is `production`. Vercel: `VERCEL_ENV` is `production`. Any other host: find its environment-name variable in its docs and write it in `docs/TOOLING.md`. The host sets these per environment, so they are never copied into a preview. This also covers `JOBS_ENABLED` from the Automation recipe, which a Railway PR environment would otherwise copy from production.

   The same task replaces the Deploying section of `CLAUDE.md` with:

   ```
   Branch mode, live since <date>. Never commit to main.
   Every task gets its own branch from an up-to-date main, named after the task ID.
   Pushing a branch never deploys: push it at /checkpoint, then open a pull request.
   Merging to main deploys<, and runs migrations against the live database>. Only the user merges.
   Previews: <one per pull request on <host> | none, so Sync checks production right after the merge>.
   Outside actions are blocked outside production by <helper name>.
   ```

   Done when: a test shows the helper skips when the variable says anything other than production, and the Deploying section matches what the host does. A build with nothing outbound only needs the Deploying section.

3. **Switch previews on (`[@]` for the user).**
   - **Vercel:** already on; every branch gets a preview.
   - **Netlify:** builds a preview per pull request.
   - **Railway:** Project Settings, Environments, Enable PR Environments. Also Enable Focused PR Environments when the project has more than one service, so a change to the web app does not redeploy the worker. If step 1 made a `staging` environment, set it as the base PR environments inherit from. Each open pull request runs a copy of the services and is billed by usage until it is merged or closed, so pull requests do not sit open.
   - **Any other host:** check its docs. With no previews, branch mode still works: the merge is still the single moment it goes live, and Sync checks production straight after it.

4. **The first pull request is the proof (Cowork, in its Sync).** Check with the host's MCP that the preview has the database step 1 said it would, and note whether it came up empty or holding data. Write the answer in `docs/LEARNINGS.md`. If the preview points at the live database, the user closes the pull request and nothing else gets built until it is fixed.

## Every task in branch mode

- **Start line.** The prompt opens with: `Start on a fresh branch: git switch main && git pull && git switch -c <task-id-in-lowercase>-<short-slug>` (for example `m2-t3-invoice-list`).
- **Pull-request line.** In Before you finish, after `/checkpoint`: `Then open a pull request with gh pr create --base main, titled "<ID>: <task name>", with the PROGRESS entry as the body. Do not merge it.` The pull request link goes in the PROGRESS entry.
- **One open pull request at a time.** If the last one is still open, the handover says merge or close it first. Two open branches both append to `docs/PROGRESS.md` and collide.
- **Sync checks the preview, not production.** Find the preview from the host's MCP or the link the host posts on the pull request (Vercel and Railway both comment one). `/version` on the preview must match the branch's latest commit, and the rest of Sync (opening it, the design score, the findability check) runs against the preview URL. A fix goes on the same branch, so the same pull request and preview update.
- **Merging is the user's.** Sync ends with one line: "Merge it" and the pull request link, or the fix prompt. If the pull request contains a migration, that line says so, because merging runs it against the live database. They merge on GitHub with **Create a merge commit**, which keeps the SHAs in PROGRESS valid; a squash merge would replace them. Claude Code and Cowork never merge.
- **After the merge,** the next Sync checks `/version` on production against the head of `main` before anything else gets built.

## Undo in branch mode

- **Not merged yet:** close the pull request and delete the branch. Nothing live changed.
- **Merged:** the **Revert** button on the pull request in GitHub opens a revert pull request with its own preview. Check it, then the user merges it.
- The database rule in Undo still holds: code goes back, data does not. A merged migration is its own careful task.

## What does not change

- **The `.claude/` files.** `/checkpoint` already pushes a branch without asking, because a branch push does not deploy, and it reads the Deploying section to know that. The `ask` rule on `git push` in `settings.json` stays as the last guard: Claude Code still shows its push prompt, and the user approves it knowing a branch push is safe.
- **Commit every task, SHA in PROGRESS.** Same as before; the branch just carries them until the merge.
- **One task, one branch.** Worktrees and parallel branches stay off.
