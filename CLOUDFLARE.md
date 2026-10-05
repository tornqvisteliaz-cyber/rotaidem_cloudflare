# Cloudflare Workers migration

The existing Flask site cannot run as a process on the Cloudflare Workers free tier. This project keeps every template, page, and route, and serves them from a Worker.

Do not use Gunicorn, the Dockerfile, or the Procfile on Cloudflare. Those files remain only so the old Python app is still in the repository.

## One-time setup

```sh
npm install
npx wrangler login
npx wrangler d1 create rotaidem
npx wrangler r2 bucket create rotaidem-uploads
```

Copy the database id from `wrangler d1 create` into `wrangler.toml` (`database_id`).

Add the domain `rotaidem.com` to this Cloudflare account, then:

```sh
npx wrangler secret put SECRET_KEY
npx wrangler secret put ADMIN_USERNAME
npx wrangler secret put ADMIN_PASSWORD
npx wrangler secret put ADMIN_EMAIL
npx wrangler secret put RESEND_API_KEY
npx wrangler secret put RESEND_FROM
npx wrangler secret put SEABEE_DOWNLOAD_URL
npx wrangler secret put ADMIN_RESET_PASSWORD
npx wrangler secret put RSG_DEV_UNLOCK
```

`ADMIN_RESET_PASSWORD` must stay `false` except for one deploy used to reset the owner password.

## Local

```sh
cp .dev.vars.example .dev.vars
npm run dev
```

## Deploy

```sh
npm run migrate:remote
npm run deploy
```

Routes are attached to `rotaidem.com` and `www.rotaidem.com`.

## Database

D1 replaces PostgreSQL. Free Workers cannot open a TCP connection to Postgres. Schema is in `migrations/0001_schema.sql` and is also created on first request if missing.

Existing Postgres rows are not copied automatically. Export them and import with `wrangler d1 execute rotaidem --remote --file=...`. Password hashes created by Flask-Bcrypt can be verified, but bcrypt cost 12 can exceed the free-tier CPU limit. New passwords use PBKDF2-SHA256. Reset imported accounts once if login fails on the free plan.

Newsletter images and invoices go to the R2 bucket `rotaidem-uploads`, not the local disk. Public image URLs stay `/static/uploads/<file>`.
