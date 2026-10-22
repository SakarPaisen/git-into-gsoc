# 8. Keep track of your progress

A few notes help you remember what you learned and choose a next step. Start whenever useful; you do not need to finish the earlier steps first.

```bash
git-into-gsoc progress summary
```

The summary is a readable list. It shows the ten phases with a check mark on the ones you finished and a `(next)` label on the first one you have not, then the organizations you shortlisted, the repositories and issues you track, and how many weekly notes you saved. On a fresh installation it starts with `0/10 phases done`. It is a personal checklist, not an application status or a measure of readiness.

For a script or an assistant that reads structured text, add `--output json`:

```bash
git-into-gsoc progress summary --output json
```

That prints the same information as JSON, including `completion` as `0/10` and `next_phase` as the first phase you have not marked done.

## Save something you want to revisit

These commands write to a file on your computer. Replace the example organization and repository with your own choices when appropriate:

```bash
git-into-gsoc progress shortlist_org --value "SymPy"
git-into-gsoc progress track_repo --value sympy/sympy
git-into-gsoc progress track_issue --value https://github.com/sympy/sympy/issues/12345
git-into-gsoc progress note --value "Read the setup guide. Next: run the project locally."
git-into-gsoc progress complete_phase --phase track
```

The tool saves the text you provide; it does not assign the issue to you. Re-run `git-into-gsoc issue <link>` before you start work on a tracked issue, because availability changes daily during GSoC season.

Valid phase names are `learn`, `discover`, `evaluate`, `orient`, `community`, `find_issues`, `contribute`, `iterate`, `proposal`, and `track`. Guide chapters do not map to one command each: some have none and some use several. The [journey map](../journey.md) lists every phase with its commands and the guide page that explains it, so use it to find the command for the phase you want to mark. Mark `track` once keeping a weekly note is a habit. Mark a phase complete only when its goal is useful and satisfied for you. You can revisit any phase.

## Keep a weekly note

```bash
git-into-gsoc log --goal "Reproduce #12345" --done "Built sympy, found the failing test" \
  --links https://github.com/sympy/sympy/issues/12345 --stuck "Fixture missing" --next "Ask in the channel"
git-into-gsoc log            # shows your recent notes
```

Each note has the structure mentors expect in progress updates: goal, what you tried or finished, links, what you learned, where you are stuck, and the next small step. Free-form notes still work with `progress note`. To read everything later, open `progress.json` in the data directory shown by `doctor`. See [local data](../configuration.md) for backup and editing instructions.

## Ask what to do next

```bash
git-into-gsoc next
```

`next` reads the program calendar, your profile, your phases, and your shortlist, re-checks every tracked issue live (and tells you if one was claimed since you last looked), and prints the next few actions with the exact commands. Run it at the start of each session.

There is no CLI command to remove an entry or undo a completed phase. The store is local, does not sync between computers, and does not send updates to mentors. Share a useful update yourself through the community's preferred channel.

If accepted, agree on a reporting schedule with your mentor. If you are not selected, you can still keep contributing, learn from feedback, or explore another project.

[Return to the guide](../README.md) · [Command reference](../cli.md)
