# AGENTS.md

## Run the app (Base44 sandbox)

```
docker compose -f docker-compose.base44.yml up -d --build
```

Web entry point: host port **3000** (`next dev` inside the `web` service, source bind-mounted from
the repo root, dependencies installed into the `node_modules` volume at startup from
`package-lock.json`).

## Non-obvious findings

- **This repository currently contains no application source.** The only tracked files are
  `package.json`, `package-lock.json` and `.gitignore` — there is no `app/` or `pages/` directory,
  no `next.config.js`, and no components. `next dev` therefore starts and then exits with:

  ```
  Error: > Couldn't find any `pages` or `app` directory. Please create one under the project root
  ```

  The compose stack cannot serve anything until the source tree is added. The container is
  crash-looping by design (`restart: on-failure`) in this state, so bring the stack down rather than
  leaving it looping.
- Declared dependencies are only `next@14.2.15`, `react@18.3.1`, `react-dom@18.3.1`. There is **no
  database, cache, queue or third-party API client** in the manifest, so no external service and no
  credentials are required to boot — only the missing source.
- `next@14.2.15` is flagged by npm as carrying a security vulnerability; flag a version bump to the
  owner rather than changing it unprompted.

## How to verify it works

Once source is present, the check is that the served page is live-rendered source, not a bundle:
`curl -sI http://localhost:3000/` should return `200` from the dev server, and the service log
should show Next.js compiling routes on request.
