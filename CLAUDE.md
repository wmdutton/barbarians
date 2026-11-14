# Barbarians

A text-based, single-page HTML survival/settlement game. The player picks a
vanguard of men from a rolled candidate pool, then assigns them daily actions
(hunting, cooking, forestry, earthworks, building, keeping ward) to keep them fed
and fortify a camp on contested ground.

## Personal project — keep separate from work

- This is a **personal** project. It must never mix with work code or accounts.
- GitHub account: **wmdutton** (`wmdutton@gmail.com`). Never use a work GitHub
  account or work email here.
- Git identity and SSH key come from `~/.gitconfig-personal` (applied via
  `includeIf "gitdir:~/code/personal/"`), using `~/.ssh/id_ed25519_personal`.
  Remote: `git@github.com:wmdutton/barbarians.git`.
- Before committing, `git config user.email` should print `wmdutton@gmail.com`.
- If using `gh` for PRs: run `gh auth switch -u wmdutton` first, and switch back
  to the work account afterwards.
- Do not read or touch anything under `~/code/work`.

## Running locally

No build step and no dependencies. From the repo root:

```sh
python3 -m http.server 8000
```

Open http://localhost:8000. To test on a phone on the same Wi-Fi, use
`http://$(ipconfig getifaddr en0):8000`.

## Workflow

- Work on the `development` branch.
- Make small, focused commits.
- Push `development`, then open a PR from `development` into `main`.
- Don't push unless asked.

## Branch sync

- After merging a PR into `main`, immediately sync `development`:
  ```sh
  git checkout development
  git fetch origin
  git merge --ff-only origin/main
  git push origin development
  ```
- At the start of every session, run `git fetch origin`. If `origin/main` is
  ahead of `development` and `development` has no unpushed commits, sync it the
  same way. If the fast-forward fails or `development` has unpushed work, stop
  and tell Mason instead of merging anything.
- Never push directly to `main` and never force-push.
- Only merge a PR when Mason explicitly asks.

## Layout

- `index.html` — the entire game (HTML, CSS and JS in one file). The version
  tag is in the top comment and in `.sb-version`; bump both together.
- `design-rationale.md` — the "why" behind prototype code decisions.
- `docs/components.md` — UI component specs.
- `docs/tokens.md` — CSS design tokens (parchment theme).

Note: the docs lag the code. `index.html` is v0.2.4.0 and uses a die-face skill
system (9 skills, actions grouped into industry pipelines).
`design-rationale.md` is tagged v0.5.0.0 and describes an older tier-based
build. `docs/` is verified against v0.2.3.0. Treat `index.html` as the source
of truth and update docs when touching related systems. The locked design
docs it references (`barbarians-design-doc-*.md`) are not in this repo.
