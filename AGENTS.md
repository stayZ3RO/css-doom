# Agent instructions

This is my public cssDOOM site, a Vite rebuild of Niels Leenheer's CSS renderer. The canonical URL is https://cssdoom.wtf. The repository is public. Anything committed here can be read by anyone.

## Public safety

- No secrets, tokens, private keys, or env files.
- No real LAN addresses. Documentation examples use RFC 5737 addresses only.
- No internal hostnames.
- No names of private repositories.
- No screenshots without redaction.

## Voice

Plain first-person engineer notes. No em dashes.

## Git

No commits or pushes without my explicit go. Open pull requests on Forgejo. No AI attribution trailers.

## CI

GitHub workflows run on the mirror on push, and deploy and gitleaks can also be run by hand (gitleaks also runs weekly). `.forgejo/workflows/validate.yml` runs on Forgejo. The Pages deploy is `.github/workflows/deploy.yml`.
