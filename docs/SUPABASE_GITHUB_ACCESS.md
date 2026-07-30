# Supabase LAWSA — GitHub Access

## Target

- Supabase project: `LAWSA`
- Project reference: `bzkjxxptdfpqmuwoxtjp`
- Project URL: `https://bzkjxxptdfpqmuwoxtjp.supabase.co`
- GitHub repository: `wasalstor-web/mubassat-law`

## Credential model

### Browser-safe values

These values are declared in `.env.example` and may be used by the Next.js browser client:

- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`

The publishable key does not bypass Row Level Security. Authorization must remain enforced through Supabase Auth and RLS policies.

### Server-only secrets

Never commit real values for these variables:

- `SUPABASE_SECRET_KEY`: privileged server-side API operations; bypasses RLS and must never reach client code.
- `SUPABASE_ACCESS_TOKEN`: Supabase account token used by the CLI and GitHub Actions.
- `SUPABASE_DB_PASSWORD`: LAWSA database password used by the CLI.

## Add encrypted secrets to GitHub

1. Open the repository `wasalstor-web/mubassat-law`.
2. Open **Settings**.
3. Open **Environments** and create or open `production`.
4. Under **Environment secrets**, add:
   - `SUPABASE_ACCESS_TOKEN`
   - `SUPABASE_DB_PASSWORD`
   - `SUPABASE_SECRET_KEY`
5. Optionally require reviewers for the `production` environment so deployment cannot run without approval.

The workflow uses environment `production`, so environment-level secrets take precedence and production access stays gated.

## Where each value comes from

- `SUPABASE_ACCESS_TOKEN`: Supabase Dashboard → Account → Access Tokens.
- `SUPABASE_DB_PASSWORD`: the database password set for project LAWSA. Reset it from database settings if it is unknown.
- `SUPABASE_SECRET_KEY`: Supabase Dashboard → Project LAWSA → Settings → API Keys → Secret key. Prefer a modern `sb_secret_...` key rather than a legacy service-role JWT.

## Run access verification

1. Open GitHub → **Actions**.
2. Select **Supabase LAWSA Access**.
3. Select **Run workflow**.
4. Choose a mode:
   - `verify`: link the repository and read migration history only.
   - `dry-run`: preview pending committed migrations without applying them.
   - `deploy`: apply pending committed migrations to LAWSA.

`deploy` is deliberately manual and runs through the protected `production` environment.

## Access from Next.js

### Browser and authenticated SSR client

Use only the URL and publishable key:

```ts
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY!
  )
}
```

### Privileged server client

Use only from server code, Route Handlers, Server Actions, or trusted jobs:

```ts
import 'server-only'
import { createClient } from '@supabase/supabase-js'

export function createAdminClient() {
  const url = process.env.NEXT_PUBLIC_SUPABASE_URL
  const secretKey = process.env.SUPABASE_SECRET_KEY

  if (!url || !secretKey) {
    throw new Error('Missing server-side Supabase credentials')
  }

  return createClient(url, secretKey, {
    auth: {
      autoRefreshToken: false,
      persistSession: false,
    },
  })
}
```

Do not import the privileged client into Client Components. Do not prefix the secret key with `NEXT_PUBLIC_`.

## Local setup

```bash
cp .env.example .env.local
```

Fill server-only values locally, then keep `.env.local` untracked. The repository `.gitignore` already excludes local environment files.

## Database change policy

- Store schema changes as SQL files under `supabase/migrations/`.
- Run `verify` first.
- Run `dry-run` and review the output.
- Run `deploy` only after approval.
- Never use `supabase db reset --linked` against LAWSA production.
