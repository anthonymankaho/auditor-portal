# Auditor Access Portal · 審計員入口

Standalone, isolated web entry for the independent auditor to open the accounting
systems of **Gold Plume Limited (GPL)**, **Essential Investment Consultants
Limited (EIC)** and **Meyer Properties Limited (MPL)** — without any access to
MyVault OS or its vault.

Live: <https://anthonymankaho.github.io/auditor-portal/>

## How it works

- Single self-contained `index.html` — no build step, no external assets.
- Each company is unlocked with its own access code. The code is sent to the
  MyVault Worker (`accounting-system` action), verified server-side against a
  Worker secret, and only then does the Worker stream the private system HTML
  from the company's private GitHub repo using a server-side read token.
- Codes live in `sessionStorage` only (per browser tab). Closing the tab locks
  everything. No GitHub sign-in, no vault access, no other content is reachable.
- Wrong codes: 5 attempts, then a 30-second client pause (server remains the
  authority). The system opens in a full-screen viewer with reload / lock.

## Administration (owner only)

The Worker needs four secrets (set interactively — never commit values):

```
cd <myvault-os>/worker
npx wrangler secret put GH_REPO_READ_TOKEN     # fine-grained PAT, Contents:read on GPL/EIC/MPL only
npx wrangler secret put GPL_STANDALONE_CODE    # auditor code for GPL
npx wrangler secret put EIC_STANDALONE_CODE    # auditor code for EIC
npx wrangler secret put MPL_STANDALONE_CODE    # auditor code for MPL
```

Create the fine-grained PAT at
<https://github.com/settings/personal-access-tokens/new> — repository access:
only `GPL`, `EIC`, `MPL`; permissions: Contents → Read-only.

Until the secrets are set the portal shows a clear "service not ready" message
and releases no data.
