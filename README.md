# Auditor Access Portal · 審計員入口

Standalone, isolated web entry for the independent auditor to open the accounting
systems of **Gold Plume Limited (GPL)**, **Essential Investment Consultants
Limited (EIC)** and **Meyer Properties Limited (MPL)** — without any access to
MyVault OS or its vault.

Live: <https://anthonymankaho.github.io/auditor-portal/>

## How it works

- Single self-contained `index.html` — no build step, no external assets.
- Each company is unlocked with its own access code, set by the owner in the
  MyVault OS Accounting view. The code is sent to the MyVault Worker
  (`accounting-system` action), verified server-side, and only then does the
  Worker stream the private system HTML from the company's private GitHub repo
  using a server-side read token.
- Codes live in `sessionStorage` only (per browser tab). Closing the tab locks
  everything. No GitHub sign-in, no vault access, no other content is reachable.
- Wrong codes: 5 attempts, then a 30-second client pause (server remains the
  authority). The system opens in a full-screen viewer with reload / lock.
- **No second login:** the accounting systems carry their own username/password
  workflow gate. The portal provisions a dedicated `auditor` account through the
  system's own account machinery and opens a session automatically — MPL in
  read-only *review* mode, with the external FINSYS tile hidden. An existing
  session (e.g. the owner's master account) is never replaced. Auditors only
  ever need the access code; no login ID is required anywhere.
- Opened anywhere other than the live origin (e.g. a local file preview), the
  page shows a "design preview" banner and does not attempt to unlock.

## Administration (owner only)

**Access codes are set in the app, not on this machine.** Sign in with GitHub
at <https://anthonymankaho.github.io/myvault-os/#accounting> and use the
**Owner Console / 管理員** panel to type one code per company (GPL / EIC / MPL)
and press **Save code**. Codes are stored server-side in the Worker's KV
(`auditor-code:<system>`), take effect immediately, and are never displayed
again — type a new one any time to replace it. No codes are generated or stored
locally.

The Worker still needs one secret, set interactively (never commit the value):

```
cd <myvault-os>/worker
npx wrangler secret put GH_REPO_READ_TOKEN   # fine-grained PAT, Contents:read on GPL/EIC/MPL only
```

Create the fine-grained PAT at
<https://github.com/settings/personal-access-tokens/new> — repository access:
only `GPL`, `EIC`, `MPL`; permissions: Contents → Read-only.

Until a company's code is set the portal shows a clear "service not ready"
message for that company and releases no data (fail-closed).
