# Branch Protection Rules (Recommended)

This repository uses GitHub branch protection to enforce baseline OSS quality and security checks.

## Where to Configure

In GitHub:
1. Go to `Settings` → `Branches`
2. Under **Branch protection rules**, select **Add rule**
3. Apply to branch pattern: `main`

## Recommended Rule Settings

Enable the following:

- ✅ **Require a pull request before merging**
  - Require approvals: `1` (or more)
  - Dismiss stale pull request approvals when new commits are pushed

- ✅ **Require status checks to pass before merging**
  - Require branches to be up to date before merging
  - Add required checks:
    - `Bicep Lint / lint-bicep`
    - `Secret Scan / secret-scan`
    - `PR Validate / validate-repo`

- ✅ **Require conversation resolution before merging**

- ✅ **Block force pushes**

- ✅ **Block branch deletion**

Optional (recommended for tighter governance):
- Require review from Code Owners
- Restrict who can push to matching branches
- Require signed commits

## Notes

- These checks can be configured now, even if `bicep/` is currently empty.
- The Bicep lint workflow skips gracefully when no `.bicep` files are present.
- Secret scanning and PR validation remain active immediately.
