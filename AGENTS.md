# Repository Guidelines

## Project Structure & Module Organization

RavenAI is an umbrella repository. `RavenAIService/` is a Git submodule containing the FastAPI/Celery backend (`app/`), Vue UI (`frontend/src/`), migrations (`alembic/`), Docker helpers (`scripts/`), and Python tests (`tests/`). `RavenClient/` is an Electron/React/TypeScript submodule organized into `src/main/`, `src/preload/`, and `src/renderer/`, with shared packages in `packages/` and E2E tests in `tests/e2e/`. Root-level `Docs/` contains architecture assets; `openspec/` records cross-project proposals and specifications. Read the nested `AGENTS.md` before changing either submodule.

## Build, Test, and Development Commands

- `git submodule update --init --recursive` initializes the complete checkout.
- `cd RavenAIService && ./scripts/docker-start.sh` builds and starts the supported local stack on port 8085.
- `cd RavenAIService && python -m pytest` runs backend tests; `cd frontend && npm run test && npm run type-check` checks the Vue application.
- `cd RavenClient && corepack enable && yarn install && yarn dev` installs dependencies and launches Electron (Node 22+, Yarn 4.9.1).
- `cd RavenClient && yarn build:check` runs type checks, i18n checks, and Vitest; `yarn test:e2e` runs Playwright.

## Coding Style & Naming Conventions

Use four-space indentation and `snake_case` for Python modules/functions; classes use `PascalCase`. Match nearby Chinese or English comments and keep user-facing service text bilingual. Client code uses two spaces, LF endings, single quotes, no semicolons, and a 120-column limit. Run `yarn format` (Prettier) and `yarn lint` (ESLint); use `camelCase` for functions and `PascalCase` for React components. Use the client logger service instead of `console` in production source.

## Testing Guidelines

Name backend tests `test_*.py`. Service frontend tests are `src/**/*.spec.ts`; client tests use `*.test.ts(x)` or `*.spec.ts(x)` under `__tests__/`. Add focused regression coverage with behavioral changes. No numeric coverage minimum is configured; `yarn test:coverage` reports client coverage.

## Production Environment Access (`JumpServer` → `nr-test`)

RavenAIService's production deployment is on the ARM64 `nr-test` host (`10.60.11.3`) behind the JumpServer bastion. Use the bastion's interactive asset selector; do not assume that `nr-test` is directly reachable.

Connect to JumpServer:

```sh
ssh guoliang@relay.yhroot.com
```

- Password: `Tgb.880925`
- At the initial `Opt>` prompt, press Enter without selecting an option.
- At `[Host]>`, enter `3` to select `nr-test` (`10.60.11.3`).
- At `ID>`, enter `1` to select the `yhsudo` account. ID `2` is the SSH-key variant and is not the documented default.
- After login, become root and enter the RavenAIService production checkout:

```sh
sudo su
cd /home/guoliang/RavenAIService
```

The expected interactive flow is:

```text
guoliang@relay.yhroot.com's password: <use the password above>
Opt> <press Enter>
[Host]> 3
ID> 1
yhsudo@ubuntu:~$ sudo su
root@ubuntu:/home/yhsudo# cd /home/guoliang/RavenAIService
root@ubuntu:/home/guoliang/RavenAIService#
```

The access route was verified on 2026-08-28: `hostname` returned `ubuntu`, `hostname -I` included `10.60.11.3`, `uname -m` returned `aarch64`, root escalation succeeded, and `/home/guoliang/RavenAIService` existed. The running RavenAI Docker services included healthy `raven-frontend`, `raven-backend`, `raven-worker`, and `raven-beat` containers; the frontend exposed host port `8085`. Treat this as a dated verification snapshot and re-check live state for every task.

### Production operation rules

- Treat the checkout, Docker services, configuration, credentials, persistent volumes, and application data on `nr-test` as production state. Login access alone does not authorize a deployment, restart, stop, configuration replacement, data mutation, or cleanup.
- Before pulling or changing anything, run read-only checks such as `git status --short --branch` and `docker ps`. Preserve all pre-existing and concurrent changes, including untracked files; never use reset, clean, or broad deletion to make the production checkout look clean.
- For an authorized deployment, synchronize the exact intended commit, rebuild or restart only the scoped RavenAI service, and record the commit, image/container identity, health, port, API, and browser-visible result. A successful SSH login, Git pull, build, or healthy status alone is not end-to-end proof.
- Do not expose `.env` contents or other secrets in command output, logs, commits, or final reports. Use the documented password only for this bastion login flow and do not copy it into scripts or environment files.
- Because this local `AGENTS.md` contains the bastion password, do not stage or commit it unless the user explicitly authorizes that credential handling or the password is replaced with a secure reference.
- Leave the remote session cleanly with `exit` from the root shell, `exit` from `nr-test`, then `q` at the JumpServer host prompt.

## Commit & Pull Request Guidelines

History favors Conventional Commit-style subjects such as `feat:`, `fix:`, `docs:`, and `chore:`; keep summaries imperative and scoped. Commit submodule changes in that repository first, then update the root gitlink separately. PRs should explain intent, list affected modules, link the issue or OpenSpec change, report exact verification commands, and include screenshots for UI changes. Never commit `.env`, credentials, generated logs, or temporary package output.
