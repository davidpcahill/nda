# Setup (one-time, for the repo owner)

> **This page is for David.** If you're a signer, you want [SIGNING.md](SIGNING.md) instead — this file is only about the one-time plumbing to get the repo up and running.

You'll do this once. After that, onboarding anyone new is just "point them at this repo and merge their PR."

The last section (auto-invite) is **optional** — safe to skip if you're onboarding only a handful of people and are fine manually inviting them to your org after each merge.

---

## 1. Push this repo to GitHub

Make it a **public** repo so people who haven't signed yet can actually read the NDA and open a PR. Public means the repo is world-readable; the *signatures* that land in it are also public (that's fine — a signature is meant to be publicly provable).

Recommended location: under your personal account (e.g. `github.com/davidpcahill/nda`) so it's not tied to any one org. The NDA covers the Furbidden org, Cybrix.AI, and whatever future venture — the repo itself shouldn't live inside any one of them.

Using the GitHub CLI:

```bash
cd /path/to/this/nda/folder
git init -b main            # -b main forces the modern default branch name
git add .
git commit -m "Initial NDA repo"
gh repo create davidpcahill/nda --public --source=. --push
```

> **Note on branch names.** Older `git init` defaults to `master`. If you forget `-b main` and end up on `master`, rename it afterward via the GitHub website: **Settings → General → Default branch → pencil icon → rename to `main`**. GitHub handles redirects automatically. The workflow in this repo expects `main`.

Or on the website: create a new public repo at `github.com/davidpcahill/nda` (defaults to `main`), then follow the "push an existing repository" snippet GitHub gives you.

## 2. Protect the default branch

You want two guarantees on `main`:
- No one can push directly to `main` — everything must go through a PR.
- Every signer PR requires your review before it can merge (this is what makes your merge the countersignature).

GitHub now offers two equivalent ways to do this: the newer **Rulesets** UI (default on new repos) or **classic Branch protection rules**. Use whichever you see.

### Rulesets UI (newer — what most new repos show)

Repo → **Settings → Rules → Rulesets → New ruleset → New branch ruleset**.

- **Ruleset name:** `Protect main`
- **Enforcement status:** Active
- **Bypass list:** add yourself (`davidpcahill`) with bypass mode **Always**. Reason: GitHub doesn't let a PR author approve their own PR, so if you ever edit [NDA.md](NDA.md) yourself and you're the only code owner, you'd lock yourself out. Being in the bypass list lets you merge your own admin changes. Signer PRs still go through the full approval flow as normal — you'll just review and approve them before merging.
- **Target branches:** Include default branch.
- **Rules:**
  - ✅ **Restrict deletions**
  - ✅ **Block force pushes**
  - ✅ **Require a pull request before merging**
    - Required approvals: **1**
    - ✅ Dismiss stale pull request approvals when new commits are pushed
    - ✅ Require review from Code Owners
    - ✅ Require approval of the most recent reviewable push
  - Leave **Require status checks to pass** unchecked (there are no CI checks gating merges).

Click **Create**.

### Classic branch protection (legacy UI)

Repo → **Settings → Branches → Add classic branch protection rule** for `main`. Enable the equivalent options: require PR, 1 approval, require code owner review, dismiss stale approvals, block force push, block deletion. Leave "Do not allow bypassing the above settings" **unchecked** so you can merge your own future admin changes.

## 3. Configure your GitHub org(s) for gated access

Do this for each org whose private repos should be NDA-gated (Furbidden now, Cybrix.AI's org if separate, and any future org).

### 3a. Lock down org-wide visibility

**Org → Settings → Member privileges:**
- **Base permissions: No permission.** This is the critical one. It means a new org member sees zero private repos by default.
- **Repository visibility change:** Admins only.
- **Repository creation:** Members **cannot** create private repos (or set to admins only).

### 3b. Org description pointing to the NDA

**Org → profile page → Edit description** (pencil icon next to the org name, or Settings → Profile):

> *Access to private repositories requires signing the NDA at https://github.com/davidpcahill/nda before being added to the `signed-nda` team.*

This is visible to anyone who lands on the org profile, so new invitees have a clear pointer.

### 3c. Create the `signed-nda` team

**Org → Teams → New team:**
- **Name:** `signed-nda` (or whatever slug — remember it for step 5).
- **Description:**

  > *Members who have signed the NDA at https://github.com/davidpcahill/nda. Do not add members manually unless you have verified their signature PR is merged on the `main` branch.*

- **Team visibility:** Visible.
- **Parent team:** none.

### 3d. Grant the team access to gated repos

For each private repo that should be NDA-gated:
- Repo → Settings → Collaborators and teams → **Add teams** → `signed-nda` → permission level of your choosing (**Read** is the safest default; use Write / Maintain only for collaborators who need it).

Now: an unsigned person who somehow lands in the org sees nothing. Only `signed-nda` members see the gated repos.

## 4. Test it on yourself before you invite anyone real

Seriously do this. Five minutes. Catches any misconfiguration before a real signer hits it.

1. From a second GitHub account you control (or ask a trusted friend with a burner), fork the NDA repo.
2. Follow [SIGNING.md](SIGNING.md) Path A all the way through — open the PR.
3. On your main account, review and merge the PR.
4. Confirm the signature file is now in `main`.
5. If you enabled step 5 below: confirm the **Actions** tab shows the workflow succeeded and the test account received an org invite email.
6. From the test account, accept the invite and confirm you see the gated repo(s).

When done: remove the test account from the `signed-nda` team so it doesn't linger.

## 5. (Optional) Auto-invite on merge

Enables the workflow at [`.github/workflows/grant-access.yml`](.github/workflows/grant-access.yml), which automatically invites signers to your org + `signed-nda` team the moment you merge their PR.

Without this: after merging, you manually go to the org and add the person to `signed-nda`. For a handful of people that's fine.

With this: the instant you click merge, they get an invite email. No further action from you.

You have two options for the auth token the workflow uses. **Both work — pick one, not both.**

- **Option A (recommended for ongoing use): GitHub App.** One-time setup of ~20 min, then it runs forever without token rotation. Scales cleanly to multiple orgs.
- **Option B (quick): Fine-grained Personal Access Token.** 5 min to set up. Expires on a timer (max 1 year); you'll have to re-issue when it expires. Fine for a short trial run; annoying long-term.

### Option A — GitHub App (recommended)

#### 5A.1. Register the app

Register it under **your personal account** (`davidpcahill`), not under any org. Reason: one app can then be installed on multiple orgs you own (Furbidden, future Cybrix org, etc.), surviving any org restructuring.

Go to: **Your profile avatar → Settings → Developer settings → GitHub Apps → New GitHub App**.

Fill in:

| Field | Value |
|---|---|
| GitHub App name | `NDA Access Gate` (must be globally unique; if taken, try `nda-access-gate-davidpcahill`) |
| Description | `Invites NDA signers to the signed-nda team after their PR is merged.` |
| Homepage URL | `https://github.com/davidpcahill/nda` |
| Callback URL | *(leave blank)* |
| Setup URL | *(leave blank)* |
| Webhook → Active | **unchecked** (this app doesn't receive webhooks; it's called *from* your Action) |
| Webhook URL | *(leave blank — disabled)* |
| Webhook secret | *(leave blank)* |

**Permissions:**
- **Repository permissions:** leave all at "No access."
- **Organization permissions → Members:** **Read and write**. This is the only permission that matters.
- **Account permissions:** leave all at "No access."

**Where can this GitHub App be installed?** → **Only on this account** (since all your orgs are yours; no reason to share the app publicly).

Click **Create GitHub App**.

#### 5A.2. Generate and save the private key

On the app's settings page you just landed on:

1. Scroll down to **Private keys → Generate a private key**. A `.pem` file downloads. Open it in a text editor — you'll paste the *full contents* (including `-----BEGIN RSA PRIVATE KEY-----` and `-----END RSA PRIVATE KEY-----` lines) into a repo secret shortly.
2. Note the **App ID** shown near the top of the app settings page — a number like `1234567`.

Keep the `.pem` file somewhere safe; treat it like a password.

#### 5A.3. Install the app on your org(s)

Still on the app settings page → **Install App** in the left sidebar → click **Install** next to the org you want to gate (`furbidden`).

- **Repository access:** "All repositories" or "Only select repositories" — either works since the app doesn't need any repo permissions. "Only select repositories" + picking one random repo is marginally more conservative.
- Click **Install**.

Repeat for any additional org you want to gate.

#### 5A.4. Configure the NDA repo secrets/variables

In **the NDA repo** (`davidpcahill/nda`) → **Settings → Secrets and variables → Actions**:

**Secrets tab → New repository secret:**
| Name | Value |
|---|---|
| `ORG_ADMIN_APP_ID` | the App ID from 5A.2 (e.g. `1234567`) |
| `ORG_ADMIN_APP_PRIVATE_KEY` | the full contents of the `.pem` file from 5A.2, including BEGIN/END lines |

**Variables tab → New repository variable:**
| Name | Value |
|---|---|
| `NDA_ORG` | your org slug, e.g. `furbidden` |
| `NDA_TEAM` | `signed-nda` (or whatever you named it in step 3c) |

That's it. On next merge of a signature PR, the workflow will detect the App config and auto-invite.

### Option B — Fine-grained Personal Access Token

1. GitHub → **Your profile → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**.
2. **Token name:** `nda-access-gate`.
3. **Resource owner:** the organization (e.g. `furbidden`). Not your personal account. You'll need a separate token per org if gating multiple — the App path in Option A handles multi-org better.
4. **Expiration:** 90 days is a reasonable balance. GitHub will email you before expiry.
5. **Repository access:** "Public repositories (read-only)" — the token doesn't need repo access.
6. **Organization permissions → Members:** **Read and Write**.
7. Generate and copy the token (you'll only see it once).

Then configure the NDA repo: **Settings → Secrets and variables → Actions**:

**Secrets tab:**
| Name | Value |
|---|---|
| `ORG_ADMIN_TOKEN` | the token you just copied |

**Variables tab:**
| Name | Value |
|---|---|
| `NDA_ORG` | your org slug |
| `NDA_TEAM` | `signed-nda` |

When the token expires (90 days), repeat step 7 and update the `ORG_ADMIN_TOKEN` secret with the new value.

### Disabling or switching

- **Disable auto-invite entirely:** delete the `ORG_ADMIN_APP_ID` (or `ORG_ADMIN_TOKEN`) secret. The workflow detects it missing and no-ops silently.
- **Switch from PAT to App:** set the App secrets; the workflow prefers App auth when both are present. You can then delete `ORG_ADMIN_TOKEN`.

### Multiple orgs

The workflow targets one org + one team per run, driven by the `NDA_ORG` / `NDA_TEAM` variables. If you're gating both Furbidden and a future Cybrix.AI org:

- Install the same GitHub App (Option A) on both orgs.
- Duplicate the workflow file (`grant-access-furbidden.yml`, `grant-access-cybrix.yml`), each hardcoding its org/team — OR extend the single workflow to loop over a JSON list of org/team pairs. Tell me when you hit this and I'll extend it.

---

## Ongoing ops

- **Someone signs:** you get a PR review request (from CODEOWNERS). Skim their signature file, confirm the legal name looks real, merge. Done.
- **Someone leaves / is revoked:** remove them from the `signed-nda` team (org → team page → remove member). Their signature file stays in git history; the NDA stays in force per §8 (perpetual). You're just revoking repo *access*, not the agreement.
- **You want to update the NDA:** open a PR editing [NDA.md](NDA.md). Bump `nda_version` in [signatures/TEMPLATE.md](signatures/TEMPLATE.md) from `v1` to `v2`. New signers will sign the new version. Existing signers remain bound by whatever version was in `main` when they signed (the signature file references "at the version present in the default branch … at the time this file is merged"). If you want existing signers to re-sign the new version, that's a separate conversation with each of them.
