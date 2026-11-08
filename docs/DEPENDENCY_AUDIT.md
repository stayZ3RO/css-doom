# Dependency audit

Date: 2026-10-04

`npm audit` reported 0 vulnerabilities before and after `npm audit fix`. `npm outdated` reported no in-range package updates, so the lockfile was left unchanged. `npm run build` passed. This project defines no test script.

## Update 2026-10-23

A later `npm audit` found vulnerable build dependencies. Commit `27d6bbe` (#4) updated the lockfile to fix them, and it now resolves vite 8.3.5. The 2026-10-04 result above no longer describes the lockfile.
