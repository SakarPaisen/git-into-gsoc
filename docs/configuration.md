# Configuration and local data

You can use the tool without changing any settings. Run `git-into-gsoc doctor` to see your data locations and where your GitHub token came from. The token itself is never printed.

## GitHub token

GitHub allows 60 unauthenticated API requests per hour and 5,000 with a token. Repository and issue checks make several requests each, so you will hit the limit quickly without one. You can also skip a token entirely and keep working: the tool then makes fewer requests and tells you when the limit is reached.

The tool looks for a token in this order. The source in brackets is what `git-into-gsoc doctor` prints under `GitHub token:`.

1. `GIG_GITHUB_TOKEN`, `GITHUB_TOKEN`, or `GH_TOKEN` in the process environment, checked in that order (prints as `found via environment` or `found via $GITHUB_TOKEN`)
2. The [GitHub CLI](https://cli.github.com/): if `gh` is installed and you have run `gh auth login`, the tool runs `gh auth token` for you (prints as `found via gh auth token`). Set `GIG_USE_GH_CLI=false` to turn this off.
3. The same three names in a `.env` file in the current folder (prints as `found via environment`)

Besides the higher limit, a token lets `repo` use one GraphQL query, which reads each pull request's own timeline for the first human reply instead of estimating it from recent comments, and uses fewer requests.

### Option 1: `gh auth login` (easiest)

The GitHub CLI works without a repository clone and does not depend on which terminal exported a variable, so the MCP server finds the token too. Install `gh` from [cli.github.com](https://cli.github.com/), then run:

```bash
gh auth login
```

Choose **GitHub.com** and follow the prompts. Check that the tool found it:

```bash
git-into-gsoc doctor
```

You should see `GitHub token: found via gh auth token`.

### Option 2: an environment variable in your terminal

Use this when you prefer a personal access token, or when `gh` is not available. First create a token using [GitHub's instructions](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens). The tool reads public data only, so choose no scopes (or `public_repo` on classic tokens) and an expiration date.

In the example below, `YOUR_TOKEN` is a placeholder. Replace it with the token you copied, and never paste a real token into an issue, a screenshot, or a chat.

macOS and Linux, in the current terminal:

```bash
export GITHUB_TOKEN=YOUR_TOKEN
git-into-gsoc doctor
```

PowerShell, in the current terminal:

```powershell
$env:GITHUB_TOKEN = "YOUR_TOKEN"
git-into-gsoc doctor
```

`doctor` should print `GitHub token: found via environment`.

A variable set this way lasts only for the terminal you typed it in. To keep it for every new macOS or Linux shell, add the `export` line to your shell's startup file, such as `~/.zshrc` on macOS or `~/.bashrc` on Linux, then open a new terminal. On Windows, `setx` saves it for future terminals:

```powershell
setx GITHUB_TOKEN "YOUR_TOKEN"
```

The current PowerShell window does not see a variable that `setx` just saved. Open a new one, then run `git-into-gsoc doctor` again. Keeping a token in a startup file means it is stored in plain text on your computer; the `.env` option below keeps it in one project instead.

### Option 3: a `.env` file (when you work from a clone)

This option needs a repository clone. From the repository folder:

```bash
cp .env.example .env
```

Open `.env` in a text editor, replace the `YOUR_TOKEN` placeholder with your token, and save the file:

```text
GITHUB_TOKEN=YOUR_TOKEN
```

`.env` is ignored by Git in this repository. The file is read from the **current working directory**, so it is not found automatically if you run the tool from another folder. If you installed with the one-line installer and have no clone, use Option 1 or Option 2 instead.

For safety, a `.env` in the current folder cannot change where requests go. Settings such as `GIG_GITHUB_API_URL`, `GIG_GITLAB_HOST`, `GIG_CACHE_DIR`, and `GIG_DATA_DIR` are ignored there, with a warning, because that file may belong to a repository you just cloned. Set those in the environment if you really need them. The GitLab token, which is sent to the configured GitLab server, is read from `.env` but its host is not.

After changing configuration, run a new command or restart the assistant's MCP connection; settings are cached inside a running process.

## Settings reference

Put these in `.env` or export them in the process environment. Environment variables take precedence over `.env`. The service addresses (the `GIG_..._API_URL` settings), `GIG_CACHE_DIR`, `GIG_DATA_DIR`, and `GITLAB_HOST` are read from the environment only, for the safety reason given in Option 3 above.

| Name | Default | Use |
|---|---|---|
| `GITHUB_TOKEN`, `GH_TOKEN`, `GIG_GITHUB_TOKEN` | unset | GitHub credential (see above) |
| `GIG_USE_GH_CLI` | `true` | Ask the GitHub CLI for a token when none is set |
| `GITLAB_TOKEN`, `GIG_GITLAB_TOKEN` | unset | A GitLab personal access token with `read_api`. GitLab hides issue comments without a sign-in, so with a token `issue` and `next` also read claims in the comments of GitLab issues |
| `GITLAB_HOST`, `GIG_GITLAB_HOST` | `gitlab.com` | The GitLab server the token belongs to, such as `gitlab.gnome.org`. The token is sent to this server only. Read from the environment only, so a cloned repository's `.env` cannot redirect it |
| `GIG_HOME` | your home folder | Where `setup` and `doctor` look for assistant config folders, mainly for tests |
| `GIG_TARGET_YEAR` | `2027` | Program year used when a command has no `--year` |
| `GIG_CACHE_TTL_SECONDS` | `3600` | How long GitHub responses stay fresh before revalidation |
| `GIG_ARCHIVE_TTL_SECONDS` | `86400` | Same for organization lists, archives, and ideas pages |
| `GIG_CACHE_MAX_AGE_DAYS` | `30` | Cached responses not refreshed for this many days are deleted; `0` keeps them |
| `GIG_CACHE_MAX_MB` | `500` | Above this size the oldest cached responses are deleted first; `0` turns the cap off |
| `GIG_HTTP_TIMEOUT_SECONDS` | `30` | Maximum wait for one HTTP request |
| `GIG_HTTP_MAX_RETRIES` | `2` | Retries for network errors and 5xx responses |
| `GIG_HTTP_MAX_RETRY_WAIT_SECONDS` | `20` | Longest pause accepted when GitHub asks the tool to retry later |
| `GIG_MAX_CONCURRENCY` | `6` | Parallel GitHub requests when verifying issues or grading a repository |
| `GIG_SEARCH_MIN_INTERVAL_SECONDS` | `2` | Minimum spacing between GitHub search calls, which have a separate, lower limit |
| `GIG_CACHE_DIR`, `GIG_DATA_DIR` | platform defaults | Override where the cache and `progress.json` live |
| `GIG_GITHUB_API_URL`, `GIG_GSOC_ORGS_API_URL`, `GIG_GSOC_OFFICIAL_API_URL`, `GIG_ECOSYSTEMS_COMMITS_API_URL` | public endpoints | Alternate endpoints, mainly for development and tests |
| `GIG_USER_AGENT` | package name and version | How the tool identifies itself to the APIs; usually left unchanged |

## What is saved locally?

`doctor` prints the exact paths for your operating system:

- **Cache:** API responses in an `http` subfolder. Fresh entries are served without a request. Stale entries are revalidated with the server; a "not modified" reply refreshes them without counting against your GitHub quota. If the network fails or you are rate limited, a stale entry up to a week old is served instead of an error. Once a day, entries not refreshed in 30 days are deleted, and the oldest go first when the cache passes 500 MiB. `git-into-gsoc cache` shows the size and the limits, `cache --prune` applies them now, and `cache --clear` deletes everything.
- **Data:** `progress.json`, created when you save a phase, shortlist, tracked link, note, or inferred GitHub profile.

The tool does not sync progress to GitHub or another computer. Queries go to GitHub, the official GSoC site, the organization archive, and organizations' ideas pages as needed. MCP returns tool results to the assistant that called it, so its data-handling rules also apply. Avoid storing secrets in progress notes.

To back up progress, copy `progress.json` somewhere you control. To correct entries, stop any assistant connection using the tool, back up the file, and edit the JSON carefully. To start over, rename the file; the next save creates a new one.

Most GitHub responses stay fresh for an hour, but issue checks refresh after 5 minutes and official program dates after 6 hours. `GIG_CACHE_TTL_SECONDS` changes only the one-hour default, so for a guaranteed fresh look run `git-into-gsoc cache --clear` first.

[Guide contents](README.md) · [Troubleshooting](troubleshooting.md)
