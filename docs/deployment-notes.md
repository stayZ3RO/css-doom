# Deployment Notes

## Current deployment

https://cssdoom.wtf is the canonical URL. Cloudflare serves it in front of the GitHub Pages build from `.github/workflows/deploy.yml`. https://stayz3ro.github.io/css-doom/ is the fallback. The workflow runs on main, installs locked dependencies, builds with `--base=/css-doom/`, uploads `dist`, and deploys the Pages artifact.

## Local check

```sh
npm ci
npm run build -- --base=/css-doom/
```

The Pages build has no backend, database, VM, or Docker container. Cloudflare is the front for the canonical URL.
