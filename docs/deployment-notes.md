# Deployment Notes

## Current deployment

GitHub Pages serves the static Vite build at https://stayz3ro.github.io/css-doom/. The workflow in .github/workflows/deploy.yml runs on main, installs locked dependencies, builds with `--base=/css-doom/`, uploads `dist`, and deploys the Pages artifact.

## Local check

```sh
npm ci
npm run build -- --base=/css-doom/
```

No backend, database, VM, Docker container, or custom domain is required for this deployment.
