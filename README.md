# semver-playground

A minimal Node.js project that demonstrates fully automated semantic versioning using Conventional Commits, Husky, Commitlint, Commitizen, and Semantic Release.

The focus is the release pipeline, not the application. The goal is to prove that versioning, Git tagging, CHANGELOG generation, and GitHub Release creation can be completely automated based on commit message conventions.

---

## How it works

```
Developer commits (main)
        │
        ▼
  Commitlint (Husky hook)
  validates message format
        │
        ├── ✗ bad format → commit rejected
        │
        ▼
  Push to main → nothing happens
        │
        ▼
  Merge main → release + push
        │
        ▼
  GitHub Actions triggers
        │
        ▼
  Semantic Release analyzes commits since last tag
        │
        ├── feat / fix / perf found → bump version, update CHANGELOG,
        │                              create Git tag, create GitHub Release
        │
        └── only chore / docs / ci / refactor / style → exit, no release
```

---

## Local setup

```bash
git clone <repo-url>
cd semver-playground
npm install
```

Husky hooks are installed automatically via the `prepare` script.

---

## Day-to-day workflow

```bash
# 1. Work on main
git checkout main

# 2. Stage your changes
git add .

# 3a. Commit using the interactive prompt (recommended if unsure of the format)
npm run commit

# 3b. Or commit manually if you know the Conventional Commits format
git commit -m "feat: add user authentication"

# 4. Push to main — no release is triggered
git push origin main

# 5. When ready to release, merge main into release and push
git checkout release
git merge main
git push origin release
# → GitHub Actions fires and Semantic Release takes over
```

---

## Commit message format

```
<type>(<optional scope>): <description>

[optional body]

[optional footer — use BREAKING CHANGE: <description> for major bumps]
```

### Commit type reference

| Type | Description | Creates release? | Version bump |
|---|---|---|---|
| `feat` | New feature | yes | minor |
| `fix` | Bug fix | yes | patch |
| `perf` | Performance improvement | yes | patch |
| `chore` | Config, tooling, maintenance | no | — |
| `docs` | Documentation only | no | — |
| `ci` | CI/CD changes | no | — |
| `refactor` | Code restructuring | no | — |
| `style` | Formatting, linting | no | — |
| `BREAKING CHANGE` (footer) | Any breaking change | yes | major |

**Examples:**

```bash
git commit -m "feat: add dark mode toggle"
git commit -m "fix(auth): handle expired token gracefully"
git commit -m "chore: update dependencies"
git commit -m "feat!: redesign public API"   # shorthand for BREAKING CHANGE
```

---

## Commit enforcement

Two layers prevent non-conforming commits from being saved:

1. **Commitizen** (`npm run commit`) — interactive prompt that guides you through building a valid message. Use this if you are unsure of the format.
2. **Commitlint + Husky** — validates the message on every `git commit`, regardless of how it was written. Rejects the commit if the format is wrong.

Both tools enforce the same [Conventional Commits](https://www.conventionalcommits.org) specification.

---

## What Semantic Release produces automatically

When a releasable commit is found on the `release` branch, Semantic Release:

1. Calculates the next version based on commit types since the last tag
2. Updates `package.json` with the new version
3. Generates / updates `CHANGELOG.md`
4. Commits those two files back with `chore(release): <version> [skip ci]`
5. Creates a Git tag (e.g. `v1.2.0`)
6. Creates a GitHub Release with auto-generated release notes

The `[skip ci]` suffix on the release commit prevents GitHub Actions from triggering a second workflow run.

---

## GitHub Actions

The release workflow is defined in [.github/workflows/release.yml](.github/workflows/release.yml) and only triggers on push to the `release` branch.

`GITHUB_TOKEN` is provided automatically by GitHub Actions — no manual secret configuration is needed.

---

## Project structure

```
.
├── .github/
│   └── workflows/
│       └── release.yml       # Release pipeline — triggers on push to `release`
├── .husky/
│   ├── commit-msg            # Runs commitlint on every commit
│   └── pre-commit            # Placeholder for lint/test commands
├── .gitignore
├── .releaserc.json           # Semantic Release configuration
├── commitlint.config.js      # Commitlint rules (Conventional Commits)
├── index.js
└── package.json
```
