# How to sign the NDA

You'll add one file to this repo: `signatures/<your-github-username>.md`. You open a pull request (PR) with that file. David reviews and merges. That's the signature.

**Most people should use Path A.** It's the recommended path and is legally equivalent to Path B.

- **Path A — GitHub website only (recommended).** No installs. Everything in your browser. This is a valid electronic signature under the E-SIGN Act and UETA, and is what we expect most signers to use.
- **Path B — Git on your computer (optional).** Only bother with this if you *already* have Git installed and commit signing configured, and you'd prefer the green "Verified" badge on your signing commit. It is *not* required and offers no meaningful additional legal weight — skip it if setting up Git or signing keys would be new work for you.

Before either path: **read [NDA.md](NDA.md) in full.** By signing, you are agreeing to it.

---

## Path A — Sign on the GitHub website (no installs)

### 1. Fork this repository

1. Go to the repository's main page on GitHub.
2. Click the **Fork** button in the top-right.
3. Leave the defaults and click **Create fork**. You now have your own copy of this repo under your GitHub account.

### 2. Open the signature template

1. In *your fork*, click into the `signatures/` folder.
2. Click the file `TEMPLATE.md`.
3. Click the **Copy raw file** button (or just select all the text and copy it). You'll paste this in a moment.

### 3. Create your signature file

1. Still in your fork, still inside the `signatures/` folder, click **Add file → Create new file** (top-right of the file list).
2. For the filename, type **exactly** your GitHub username followed by `.md`. For example, if your username is `janedoe`, the filename is `janedoe.md`.
3. Paste the template contents into the file.
4. Fill in every field marked `<FILL IN>`:
   - Your **full legal name** (not a nickname or handle).
   - Your **GitHub username**.
   - The **email address** associated with your GitHub account.
   - Today's **date** in `YYYY-MM-DD` format.
5. Do not delete or edit the agreement statement at the bottom. That's your signature text.

### 4. Commit the file

1. Scroll to the bottom of the page.
2. In the commit message box, type: `Sign NDA — <your full legal name>`.
3. Leave **Commit directly to the `main` branch** selected.
4. Click **Commit new file**.

### 5. Open the pull request

1. GitHub will show a banner near the top of your fork that says "This branch is 1 commit ahead of …". Click **Contribute → Open pull request**. (If you don't see the banner, click the **Pull requests** tab, then **New pull request**, then **Create pull request**.)
2. Make sure the PR is from *your fork's* `main` into *this repo's* `main`.
3. The PR title should be: `Sign NDA — <your full legal name>`.
4. In the PR description, write: `I have read the NDA and agree to be bound by it.`
5. Click **Create pull request**.

### 6. Wait for merge

David will review and merge your PR. Once merged, your signature is on record and you'll be invited to the relevant repositories or organization.

---

## Path B — Sign with Git on your computer (optional)

> **Skip this section unless you already use Git and have commit signing configured.** Path A is the recommended route and is legally equivalent. Path B exists only because some contributors already sign every commit and may prefer consistency.

This path produces a cryptographically signed commit. GitHub will display a green **Verified** badge next to your signature commit. This is aesthetic and offers no meaningful additional legal weight over Path A.

### 1. One-time setup: configure a signing key

If you already have commit signing set up (you see green "Verified" badges on your past commits), skip to step 2.

If not, follow GitHub's official guide once:

- **SSH signing (easiest if you already use SSH for Git):** https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key
- **GPG signing (more traditional):** https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key

The short version for SSH signing (run in your terminal):

```bash
# Tell Git to sign commits with SSH and point it at your existing SSH key
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

Then add that SSH key to your GitHub account as a **Signing Key** (not just an Authentication Key): GitHub → Settings → SSH and GPG keys → **New SSH key** → Key type: **Signing Key**.

### 2. Fork and clone

1. Fork this repo on GitHub (same as Path A, step 1).
2. In your terminal:

   ```bash
   git clone https://github.com/<your-username>/<this-repo-name>.git
   cd <this-repo-name>
   ```

### 3. Create your signature file

```bash
cp signatures/TEMPLATE.md signatures/<your-github-username>.md
```

Open `signatures/<your-github-username>.md` in any text editor and fill in every `<FILL IN>` field:

- Your full legal name
- Your GitHub username
- The email address associated with your GitHub account (must match `git config user.email`)
- Today's date in `YYYY-MM-DD` format

Do not edit the agreement statement at the bottom.

### 4. Commit (signed) and push

```bash
git add signatures/<your-github-username>.md
git commit -S -m "Sign NDA — <your full legal name>"
git push origin main
```

The `-S` flag signs the commit. If you set `commit.gpgsign true` in step 1, you can omit `-S`.

Verify the signature worked:

```bash
git log --show-signature -1
```

You should see a line like `Good "git" signature for …`.

### 5. Open the pull request

On GitHub, go to your fork. Click **Contribute → Open pull request** (or **Pull requests → New pull request**).

- Title: `Sign NDA — <your full legal name>`
- Description: `I have read the NDA and agree to be bound by it.`
- Confirm the commit shows a green **Verified** badge on the PR's **Commits** tab.

Click **Create pull request**.

### 6. Wait for merge

Same as Path A, step 6.

---

## Rules for a valid signature

- Filename is `<your-github-username>.md`, inside `signatures/`. Match the exact spelling of your GitHub username (case as it appears on your profile).
- All `<FILL IN>` fields are filled in with real values.
- The full legal name matches the name you'd sign on a paper contract — not a nickname, pseudonym, or GitHub handle.
- You opened the PR from an account whose email is verified on GitHub.
- You do not modify any file other than your own signature file in the signing PR. (If you spot a typo in the NDA or instructions, open a separate PR or issue.)

## Revocation / disputes

You cannot unilaterally revoke a signed NDA. If you believe your signature was submitted in error or under duress, contact David immediately.

## Questions

Open an issue on this repo, or reach out to David directly.
