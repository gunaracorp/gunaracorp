# Hosting Strategy — 30+ Website Portfolio
Updated: 2026-10-29

## Objective

Keep recurring hosting cost below Rp100,000/month where practical, without breaking production sites, databases, authentication, SEO, or domains. The GitHub account inventory contained 65 repositories at the time of review (53 public and 12 private). A repository is not automatically a production website: notes, experiments, backend-only projects, workspaces, and empty repositories should not each receive a hosting project.

## Recommended architecture

### Track A — Static hosting first
Use Cloudflare Pages Free for a repo only after a successful audit confirms it can run as a static export. Suitable examples include brochure sites, public service pages, landing pages, and content sites that can be generated at build time.

Static-export checks:
- No required Next.js server runtime at request time.
- No required Next.js API route / Route Handler, SSR, server action, middleware/proxy, or server-only authentication in the exported application.
- Dynamic forms and data operations call a separately hosted API securely where needed.
- `npm run build` succeeds and outputs `out/` after static export is enabled.
- The generated website is tested on a preview URL before changing production DNS.

Official guide: https://developers.cloudflare.com/pages/framework-guides/nextjs/deploy-a-static-nextjs-site/

Cloudflare Pages Free limits to watch: 500 builds per month, one build at a time, up to 20,000 files per site, and 25 MiB per file. See https://developers.cloudflare.com/pages/platform/limits/

### Track B — Full-stack Next.js
Do not force a dynamic application into static export. For apps that require Node.js runtime, SSR, API routes, authentication, Prisma, database access, or persistent uploads:
1. Keep a Node.js-compatible runtime until a compatibility test proves a different platform works.
2. Put the reverse proxy (Caddy/Nginx) in front of applications.
3. Keep database access private and credentials in hosting-provider environment variables.
4. Put persistent uploads and database backups outside ephemeral application build directories.
5. Begin with one application, measure RAM/CPU/storage under realistic load, then decide how many more can share the host.

A single low-cost VPS is not assumed to handle 30+ full-stack applications safely. If most repositories need server-side runtime, the Rp100,000/month ceiling may not be realistic without reducing the number of active dynamic sites or moving appropriate functions to managed services.

Cloudflare Workers can run compatible full-stack Next.js applications, but the current Cloudflare guidance recommends vinext, which is in beta; Workers Free also has a 100,000-request/day quota and a 10 ms CPU limit per request. Treat it as a pilot, not a blind mass migration:
- Next.js guide: https://developers.cloudflare.com/workers/framework-guides/web-apps/nextjs/
- Workers limits: https://developers.cloudflare.com/workers/platform/limits/

### Track C — Do not configure yet
Keep a repository in the audit queue when its production domain, build command, runtime requirements, database, environment variables, and data persistence have not been confirmed. Never change DNS or the live hosting target just because a repository contains `next.config.ts`.

## Initial code findings

The following repositories are **runtime candidates based on repository configuration**, not certified deployment results:

- `gunaracorp/topkonsultan`: `next.config.ts` uses `output: "standalone"`; package scripts include Prisma and NextAuth-related dependencies. Its config also sets `typescript.ignoreBuildErrors: true`; remove that only after running and fixing the existing type-check, rather than hiding build failures.
- `gunaracorp/pusatperizinan`: `output: "standalone"`, PostgreSQL/Prisma-related setup, and a production build script with Prisma contract checks. Do not switch it to static export without a separate architecture change.
- `gunaracorp/gunaraconsultant` and `gunaracorp/gunara-consultant`: standalone Node output and Prisma/NextAuth-related dependencies. Keep on a compatible runtime pending application-route and database audit.

These are code-level observations; no full local build, load test, production smoke test, or DNS cutover has been performed by this document.

## Security baseline

- Do not commit `.env`, `.env.local`, `.env.production`, database dumps, private keys, or real API tokens.
- Commit a sanitized `.env.example` only when its variable names and documentation have been reviewed; never copy production values into it.
- Configure secrets in the deployment provider, then test the production build and runtime.
- Deleting a file from the latest commit does not erase earlier Git history, cloned copies, caches, or logs. Rotate any credential that may have been present in a public repository. If removal from history is needed, prepare and review a coordinated history rewrite; do not force-push blindly.
- Keep `.env*` ignored in `.gitignore`, with an explicit exception only for a sanitized `.env.example`.
- Use least-privilege credentials, protected main branches, dependency updates, HTTPS, and tested backups.

## Rollout checklist

For every repository, record:
- Repository and production domain
- Public/private status and whether it is really deployed
- Build command, Node.js version, package manager, and latest build result
- Static-export eligibility and reason
- SSR/API/auth/database/upload requirements
- Environment-variable names (never their values)
- Target provider, preview URL, DNS record, canonical host, sitemap, and rollback method
- Resource metrics and monthly cost

Migration sequence:
1. Audit all repositories; mark non-production, duplicate, empty, backend-only, and workspace repositories as “no website deployment”.
2. Pilot one simple static site on Cloudflare Pages.
3. Test page routes, canonical URL, sitemap, robots.txt, structured data, forms, mobile layout, 404, and HTTPS.
4. Migrate additional static-eligible sites in small batches.
5. Pilot one dynamic app on a compatible runtime and verify authentication, database, persistent files, logs, backup, and load.
6. Move production DNS only after the preview passes; keep the old host available for rollback.
7. Recalculate cost and capacity before adding each dynamic application.

## Current execution boundary

Repository-file cleanup can be made through GitHub, but creating Cloudflare Pages/Workers projects, setting provider secrets, configuring account-level billing, and changing live DNS require the hosting-provider account and explicit domain configuration. Do not claim a site is deployed until a real deployment and production-domain test succeed.
