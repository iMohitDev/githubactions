
# Git for DevOps — OneNote cheat sheet (Beginner → Advanced)
🟢 = normally safe/read-only; 
🟡 = changes your local repository; 
🔴 = can change shared history or delete work—slow down and verify first.


## 0. Git in one minute

Your files  →  staging area  →  local commits  →  shared remote  →  CI/CD  →  release/deploy
  edit          git add          git commit          git push     build/test    tag/deploy

**The four ideas that prevent most Git mistakes**

1. `git fetch` updates your local view of the remote; it does not change your files.
2. `git status` tells you what will be affected before the next action.
3. A commit is a permanent snapshot; a branch is only a movable label pointing to one.
4. For shared history, use `git revert`; for private/local history, `reset`, amend, and rebase can be appropriate.

### Key words
- **Working tree:** the files you currently see and edit.
- **Staging area (index):** the exact snapshot queued for the next commit.
- **Commit SHA:** the immutable ID for a snapshot, for example `a1b2c3d`.
- **Branch:** a moving label such as `main` or `feature/alerts`.
- **Remote:** another repository, commonly named `origin`.
- **`origin/main`:** your locally cached view of the remote `main` branch—not a live server query.
- **Tag:** a named marker, usually a version/release such as `v2.4.0`.
- **Detached HEAD:** checked out at a specific commit/tag rather than a branch; ideal for builds and investigation.

## Choose the integration method
- **Merge:** combines histories; often best for protected/shared branches.
- **Rebase:** rewrites your private commits onto a newer base; gives a tidy linear feature branch.
- **Cherry-pick:** copies one specific commit onto another branch; ideal for a hotfix backport.
- **Squash merge:** creates one combined commit from a branch; useful when the team prefers concise default-branch history.


# Level 1 — Start safely
## 1.1 Set up Git once on your computer


git --version                                              # Confirm Git is installed and see which version you are using.
git config --global user.name "Your Name"                  # Set the author name shown on new commits.
git config --global user.email "you@company.example"       # Set the author email shown on new commits.
git config --global init.defaultBranch main                # Use main when creating a new repository.



git config --global pull.ff only                           # Refuse surprise merge commits when pulling changes.
git config --global fetch.prune true                       # Remove stale remote-branch references when fetching.
git config --global rerere.enabled true                    # Remember conflict resolutions that you have already solved.
git config --global merge.conflictstyle zdiff3             # Show clearer conflict context on modern Git versions.
git config --show-origin --list                            # Show effective settings and where each setting came from.


> Tip: use `--local` instead of `--global` if one repository needs a different work email: `git config --local user.email "work@company.example"`.
## 1.2 Get a repository
```bash
git clone <repository-url>                                 # Download an existing repository and its full working copy.
git clone <repository-url> <folder-name>                   # Clone it into a folder with your chosen name.
git clone --recurse-submodules <repository-url>            # Clone and initialize required submodules too.
git init -b main <folder-name>                              # Create a brand-new repository with main as its first branch.
git remote add origin <repository-url>                     # Connect a new local repository to its shared remote.
```
> Modern Git uses `git switch` for branches and `git restore` for files. Older documentation often uses `git checkout` for both.
## 1.3 Your first three commands in any repository
```bash
git status -sb                                              # Show branch, ahead/behind state, and concise file changes.
git log --oneline --graph --decorate --all                  # Show a compact picture of commits, branches, and tags.
git remote -v                                               # Show the fetch and push URLs for each remote.
```
Use `git status -sb` before switching branches, merging, rebasing, pushing, cleaning a folder, or responding to an incident.
## 1.4 The 90% workflow — follow this for most changes
```bash
git status -sb                                              # Check that you understand the current branch and local changes.
git fetch --prune origin                                   # Update your local view of the server without changing working files.
git switch main                                            # Move to the default branch before starting a new change.
git merge --ff-only origin/main                        # Update main only when Git can do so without an unexpected merge commit.
git switch -c feature/meaningful-change                    # Create a private branch for one focused change.
git diff                                                    # Review your edits before staging them.
git add -p                                                  # Stage only the reviewed parts you intend to commit.
git diff --staged                                           # Confirm the exact snapshot that will be committed.
git commit -m "feat: explain the change"                   # Save the reviewed snapshot locally.
git push -u origin feature/meaningful-change                # Publish your branch for review and connect its upstream. 🔴
```
🟡 **Checkpoint:** open a pull request after the push. Git itself does not create pull requests; use your Git hosting platform. Do not merge directly into `main` unless your team explicitly permits it.
---
# Level 2 — Make and save a change
## 2.1 Everyday “edit → review → commit” flow
```bash
git status -sb                                              # Check what changed before doing anything else.
git diff                                                    # Review unstaged changes in your working files.
git diff --check                                            # Find whitespace errors before you stage a change.
git add <file-or-folder>                                   # Stage a chosen file or folder for the next commit.
git add -p                                                  # Stage only selected hunks of a file interactively.
git diff --staged                                           # Review exactly what the next commit will contain.
git commit -m "feat: explain the change"                   # Save the staged snapshot as a local commit.
git status -sb                                              # Confirm the working tree is clean after committing.
```
### Useful staging variations
```bash
git add -A                                                  # Stage all additions, edits, and deletions in this repository.
git rm <file>                                               # Remove a tracked file and stage its deletion.
git mv <old-path> <new-path>                                # Rename a tracked file and stage the rename.
git diff HEAD                                               # Show all staged and unstaged tracked changes since the last commit.
git diff --name-only HEAD                                   # List only the names of changed files since the last commit.
```
### Fix a staging mistake before committing
```bash
git restore --staged <file>                                 # Unstage a file but keep its working-file edits.
git restore <file>                                   # Discard unstaged tracked-file edits by restoring the staged version. 🔴
git restore --source=HEAD --staged --worktree <file>      # Discard both staged and unstaged edits to one tracked file. 🔴
```
> Pause before either destructive `restore` command. If the work may matter, first create a temporary commit or use `git stash push -u -m "WIP: before restore"`.
## 2.2 Write commits that help operations
Use a short, imperative subject that explains **why** the change exists. For example:
```text
feat: add a readiness probe for the API
fix: prevent the deploy job from using a mutable tag
chore: update the base image digest
docs: explain rollback steps
```
Small commits are easier to review, revert, cherry-pick, bisect, and audit during an outage.
## 2.3 Correct your most recent local commit
```bash
git commit --amend --no-edit                       # Add staged fixes to the latest commit without changing its message. 🟡
git commit --amend -m "fix: clearer message"                # Replace the latest commit and its message. 🟡
git commit --fixup <commit-sha>                              # Create a labeled fixup commit for later autosquashing. 🟡
git rebase -i --autosquash <base-commit-or-branch>           # Fold fixup commits into their targets interactively. 🟡
```
> Amending or rebasing changes commit IDs. Do this freely only before others have based work on your branch.
---
# Level 3 — Work with branches and teammates
## 3.1 Create and switch branches
```bash
git branch --show-current                                    # Print the name of the current local branch.
git branch -vv                                               # List local branches, their upstreams, and last commits.
git branch -r                                                # List remote-tracking branches.
git switch main                                              # Switch to the local main branch.
git switch -c feature/meaningful-change                      # Create and switch to a new feature branch.
git switch --track -c feature/alerts origin/feature/alerts   # Create a local branch that tracks an existing remote branch.
git switch --detach <commit-or-tag>                          # Inspect/build a fixed commit or tag without moving a branch.
```
### Recommended feature-branch workflow
```bash
git fetch --prune origin                                     # Refresh remote information without changing your working files.
git switch main                                              # Move to the local default branch.
git merge --ff-only origin/main                              # Bring main up to date without creating a merge commit.
git switch -c feature/meaningful-change                      # Start an isolated branch for your work.
git push -u origin feature/meaningful-change                 # Publish it and remember its remote upstream. 🔴
```
After the first push, plain `git push` and `git pull --ff-only` know which remote branch to use.
## 3.2 Understand fetch, pull, and push
```bash
git fetch --prune origin                                     # Download remote commits/refs and prune stale remote-branch names; change no files.
git pull --ff-only origin main                               # Fetch main and fast-forward your current branch only if safe. 🟡
git push                                              # Publish commits on the current branch to its configured upstream. 🔴
git push -u origin <branch>                                  # Publish a branch and set its upstream in one step. 🔴
git push origin HEAD:refs/heads/<branch>                     # Push the current commit to an explicit remote branch. 🔴
```
**Safe habit:** for important work, prefer `fetch` + inspect + `merge --ff-only` over blindly pulling.
## 3.3 Inspect branch differences before a pull request
```bash
git fetch --prune origin                                     # Refresh the local view of the remote first.
git log --oneline origin/main..HEAD                          # Show commits on your branch that main does not have.
git diff --stat origin/main...HEAD                           # Summarize your branch changes since its common base with main.
git diff --name-only origin/main...HEAD                      # List files your branch changes relative to main.
git diff origin/main...HEAD                                  # Review the complete pull-request-style diff.
```
## 3.4 Manage remote branches carefully
```bash
git remote get-url --all origin                              # Show every fetch/push URL for origin.
git remote add upstream <canonical-repository-url>           # Add the canonical repository when you work from a fork.
git remote set-url origin <new-repository-url>               # Replace the remote URL for origin. 🟡
git remote show -n origin                                    # Display cached remote details without contacting the server.
git ls-remote --heads --tags origin                          # List remote branch/tag refs without cloning or fetching.
git push origin --delete <branch>                      # Delete a remote branch after checking that nothing relies on it. 🔴
```
Never use `git push --mirror`, `git push --all`, or routine `git push --tags` unless you have intentionally reviewed every ref that will be published or deleted.
---
# Level 4 — Integrate changes (merge, rebase, cherry-pick)
## 4.1 Choose the integration method
- **Merge:** combines histories; often best for protected/shared branches.
- **Rebase:** rewrites your private commits onto a newer base; gives a tidy linear feature branch.
- **Cherry-pick:** copies one specific commit onto another branch; ideal for a hotfix backport.
- **Squash merge:** creates one combined commit from a branch; useful when the team prefers concise default-branch history.
## 4.2 Merge a branch
```bash
git switch main                                              # Check out the branch that will receive the change.
git fetch --prune origin                                     # Refresh remote refs before integrating work.
git merge --ff-only origin/main                              # Update main safely if it can fast-forward.
git merge --no-ff feature/meaningful-change                  # Create an explicit merge commit for the feature. 🟡
git merge --squash feature/meaningful-change                 # Stage one combined change; commit it after review. 🟡
git merge --abort                                             # Cancel an in-progress merge and return to its prior state. 🟡
```
## 4.3 Rebase a private feature branch
```bash
git switch feature/meaningful-change                         # Select the private branch you want to update.
git fetch --prune origin                                     # Refresh origin/main before rebasing.
git rebase origin/main                                       # Replay your branch commits on the latest main. 🟡
git rebase -i origin/main                                    # Interactively reorder, squash, reword, or drop private commits. 🟡
git rebase --continue                                        # Continue after resolving and staging rebase conflicts. 🟡
git rebase --abort                                           # Cancel the rebase and restore the pre-rebase branch. 🟡
git push --force-with-lease origin feature/meaningful-change # Safely update the rewritten private remote branch. 🔴
```
> Never rebase a shared/protected branch without explicit coordination. `--force-with-lease` is safer than `--force`, but it is still a remote history change.
## 4.4 Cherry-pick a precise fix
```bash
git switch release/2.x                                      # Move to the target release branch.
git pull --ff-only origin release/2.x                        # Update that branch safely before the backport.
git switch -c backport/fix-123                               # Create a reviewable branch for the backport.
git cherry-pick -x <commit-sha>                              # Copy one commit and record its original SHA in the message. 🟡
git cherry-pick --continue                                   # Finish after resolving and staging conflicts. 🟡
git cherry-pick --abort                                      # Cancel the current cherry-pick safely. 🟡
git push -u origin backport/fix-123                          # Publish the reviewable backport branch. 🔴
```
## 4.5 Resolve conflicts without guessing
```bash
git status                                                   # Identify the conflicted files and the operation in progress.
git diff --name-only --diff-filter=U                         # List only unresolved files.
git diff --cc -- <file>                                      # Show the combined conflict diff for one file.
git add <resolved-file>                                      # Mark a carefully resolved file as complete. 🟡
git diff --cached --check                                    # Check staged resolution for whitespace problems.
git merge --continue                                         # Finish a merge after all conflicts are staged. 🟡
git rebase --continue                                        # Finish the current rebase step after all conflicts are staged. 🟡
git cherry-pick --continue                                   # Finish the current cherry-pick after all conflicts are staged. 🟡
```
During a normal merge, `--ours` means the branch you had checked out and `--theirs` means the incoming branch. During a rebase those labels feel reversed. Review the diff before accepting either entire side.
---
# Level 5 — Releases, tags, and deployment evidence
## 5.1 Create a safe release tag
```bash
git fetch --prune --tags origin                              # Refresh branches and tags before release work.
git switch main                                              # Move to the approved release branch.
git pull --ff-only origin main                               # Ensure local main exactly matches the approved remote history.
git status -sb                                               # Confirm no uncommitted local changes exist.
git tag -a v2.4.0 -m "Release v2.4.0" <commit-sha>           # Create an annotated release tag at an explicit commit. 🟡
git tag -s v2.4.0 -m "Release v2.4.0" <commit-sha>           # Create a signed tag when signing is configured. 🟡
git show v2.4.0                                              # Inspect the tag target and its message/signature details.
git push origin refs/tags/v2.4.0                             # Publish only the intended release tag. 🔴
```
Choose **one** tag command—`-a` for annotated or `-s` for signed. Prefer signed, annotated, protected tags for production releases.
## 5.2 Verify what was built or deployed
```bash
git switch --detach v2.4.0                                   # Check out the exact release tag without moving a branch.
git rev-parse HEAD                                           # Print the immutable commit SHA currently checked out.
git rev-parse --verify 'v2.4.0^{commit}'                     # Resolve the tag to its exact commit SHA.
git verify-tag v2.4.0                                        # Verify a signed annotated tag using trusted keys.
git verify-commit <commit-sha>                               # Verify a signed commit using trusted keys.
git describe --tags --always --dirty                         # Print a human-readable source version and dirty-state hint.
git log --oneline v2.3.0..v2.4.0                             # List commits introduced between two release tags.
git diff --stat v2.3.0..v2.4.0                               # Summarize file changes between two release tags.
```
> A production deployment record should include the repository, exact commit SHA, tag, CI build ID, artifact digest, environment, deployer identity, and timestamp. A branch name alone is not provenance.
## 5.3 Tag rules that protect production
- Build and deploy an exact tag/commit SHA, never a moving branch head.
- Protect release tag namespaces such as `v*`; allow only a release identity to create them.
- Do not retag a public release. Publish a new version instead.
- Push one tag by its full ref: `git push origin refs/tags/<tag>`.
- Treat a tag deletion or force-update as a change-management event.
---
# Level 6 — CI/CD Git patterns
## 6.1 Make CI noninteractive
```bash
export GIT_TERMINAL_PROMPT=0                                 # Bash/zsh: make Git fail rather than wait for an interactive credential prompt.
$env:GIT_TERMINAL_PROMPT = "0"                               # PowerShell: make Git fail rather than wait for an interactive credential prompt.
git status --porcelain=v1                                    # Produce script-friendly status output with no extra decoration.
git diff --check                                              # Fail early on whitespace/conflict-marker errors.
```
Use your CI platform’s trusted checkout feature and short-lived job token where possible. Keep normal checkout tokens read-only.
## 6.2 Checkout an immutable commit for a build/deploy
```bash
git init source                                               # Create a clean local repository in a disposable CI folder.
git -C source remote add origin <trusted-repository-url>     # Add an allowlisted repository URL supplied by trusted pipeline config.
git -C source -c protocol.file.allow=never -c protocol.ext.allow=never fetch --no-tags --depth=1 origin +<trusted-fully-qualified-ref>:refs/remotes/ci/source  # Fetch only a trusted ref while disallowing risky local/ext transports.
git -C source rev-parse refs/remotes/ci/source                # Print the fetched SHA; compare it with the SHA promised by the CI provider.
git -C source switch --detach <verified-commit-sha>          # Build from the verified immutable commit, not from a moving branch.
git -C source rev-parse HEAD                                  # Record the SHA actually used by the build.
```
**Important:** only use `trusted-fully-qualified-ref`, `trusted-repository-url`, and `verified-commit-sha` values from the CI provider or protected pipeline configuration—not values supplied by an untrusted pull request.
## 6.3 Shallow clones: fast but intentionally incomplete
```bash
git clone --depth 1 --branch <branch-or-tag> --single-branch <repository-url> <folder>  # Clone only the latest history needed for a quick job.
git -C <folder> rev-parse --is-shallow-repository            # Confirm whether the local clone has truncated history.
git -C <folder> fetch --unshallow                            # Download full history when versioning or ancestry checks need it.
git -C <folder> fetch --tags                                 # Download tags when the runner needs release/version data.
```
Shallow history can break `git describe`, changelog generation, `merge-base`, and comparisons with older commits. Make checkout depth/tags an explicit CI decision.
## 6.4 Clean a disposable CI workspace only
```bash
git clean -nd                                                 # Preview untracked files/directories that cleanup would remove.
git clean -fd                                                 # Delete untracked files/directories in a disposable workspace. 🔴
git clean -ndx                                                # Preview deletion including ignored files such as caches and local configuration.
git clean -ffdx                                   # Delete untracked and ignored files in a known disposable CI workspace. 🔴
```
Never use `git clean -fdx` in a developer checkout, shared deployment folder, Terraform directory with local state, or any directory with files you cannot recreate.
## 6.5 Untrusted pull-request rules
- Do not provide forks/untrusted pull requests with production secrets, write tokens, deployment credentials, or privileged runners.
- Do not let a pull request choose repository URLs, refspecs, runner labels, deployment targets, or secrets.
- Pin third-party CI actions/plugins to immutable commit SHAs, not mutable tags.
- Review workflow files, `.gitmodules`, `.lfsconfig`, and hook code like executable code.
---
# Level 7 — DevOps repository features
## 7.1 Ignore local files and prevent EOL churn
**`.gitignore` example**
```gitignore
# Ignore local environment values; commit .env.example instead.
.env
# Ignore private key files.
*.pem
# Ignore Terraform’s local working directory and state files.
/.terraform/
*.tfstate
# Ignore generated build output.
/dist/
```
```bash
git check-ignore -v <path>                                   # Show the exact ignore rule that matched a path.
git rm --cached <path>                       # Stop tracking an already committed file but leave the local file in place. 🟡
git status --ignored                                         # Show ignored files when diagnosing unexpected omissions.
```
Ignoring a secret does **not** make it safe if it was ever committed. Rotate/revoke it immediately, then follow your organization’s history-remediation process.
**`.gitattributes` example**
```gitattributes
# Let Git identify normal text files.
* text=auto
# Keep shell and YAML configuration on Unix line endings.
*.sh text eol=lf
*.yaml text eol=lf
*.yml text eol=lf
# Keep PowerShell scripts on Windows line endings when required.
*.ps1 text eol=crlf
# Treat these assets as binary.
*.png -text
*.zip -text
```
## 7.2 Submodules
```bash
git clone --recurse-submodules <repository-url>              # Clone the main repository and initialize its pinned submodules.
git submodule sync --recursive                                # Update local submodule URLs from the reviewed .gitmodules file.
git submodule update --init --recursive                       # Check out the exact submodule SHAs recorded by the parent repository.
git submodule status --recursive                              # Show each submodule’s pinned commit and initialization state.
git diff --submodule=log                                     # Show the submodule commits introduced by a parent-repository change.
```
Submodules are pinned commit references, not floating dependencies. Do not use `git submodule update --remote` in a reproducible build: it follows a moving upstream branch.
## 7.3 Git LFS for large versioned source assets
```bash
git lfs install --local                                       # Enable Git LFS behavior for this repository (Git LFS must be installed).
git lfs track "*.iso"                                        # Track future ISO files using LFS and update .gitattributes.
git add .gitattributes                                        # Stage the LFS tracking rules for the team.
git add <large-file>                                         # Stage the LFS pointer and file content.
git lfs ls-files                                              # List files currently stored through LFS.
git lfs pull                                                  # Download LFS content needed by this checkout.
```
LFS is not a secret store and is not a general artifact repository. CI must have LFS installed and authorized for its LFS endpoint when the build needs LFS content.
## 7.4 Monorepos: sparse checkout and worktrees
```bash
git clone --filter=blob:none --no-checkout <repository-url> <folder>  # Clone metadata first and defer file-content downloads where the server supports it.
git -C <folder> sparse-checkout init --cone                  # Enable fast directory-based sparse checkout.
git -C <folder> sparse-checkout set infrastructure services/api  # Populate only the directories this job needs.
git -C <folder> switch main                                   # Check out main with only the selected paths present.
git -C <folder> sparse-checkout list                          # Show the paths currently included in the sparse working tree.
git worktree add -b hotfix/urgent <separate-folder> origin/main # Create a separate working tree for a parallel hotfix branch.🟡
git worktree list                                             # List all linked working trees.
git worktree remove <separate-folder>                         # Remove a no-longer-needed linked worktree. 🔴
```
Sparse checkout is a performance optimization, not an access-control boundary. A worktree shares Git objects with its main checkout, so treat destructive Git operations as shared operations.
---
# Level 8 — Undo, rollback, and recovery
## 8.1 Choose the least-destructive fix
```bash
git restore --staged <file>                                  # Unstage a file and keep its edits. 🟡
git restore <file>                                           # Discard one unstaged tracked-file edit. 🔴
git commit --amend --no-edit                                 # Replace the newest local commit with the staged version. 🟡
git revert <shared-bad-commit-sha>                           # Create a new commit that reverses a shared/deployed commit. 🟡
git revert -m 1 <merge-commit-sha>                           # Reverse a merge while keeping its first parent as mainline. 🟡
git reset --soft HEAD~1                                      # Uncommit the latest local commit but keep changes staged. 🟡
git reset HEAD~1                                             # Uncommit the latest local commit and keep changes unstaged. 🟡
git reset --hard HEAD~1                                      # Discard the latest local commit and tracked changes. 🔴
```
**Rule:** use `revert` for a commit that is already merged, shared, or deployed. Use `reset` only for private/local history that you are certain you can discard or recover.
## 8.2 Recover “lost” local work
```bash
git reflog --date=iso                                        # Show recent local positions of HEAD and branches.
git show <sha-from-reflog>                                   # Inspect a candidate lost commit before restoring it.
git switch -c rescue/recovered-work <sha-from-reflog>        # Create a safe rescue branch pointing at that commit. 🟡
git reflog show stash                                        # Inspect stash history when a stash was dropped or misapplied.
git fsck --no-reflogs --lost-found                           # Find dangling objects as a last resort when reflog recovery fails.
```
During recovery, stop running cleanup. Do **not** run `git gc`, `git prune`, or an aggressive reset before creating a `rescue/...` branch for promising SHAs.
## 8.3 Save work temporarily with stash
```bash
git stash push -u -m "WIP: before incident investigation"   # Save tracked and untracked work locally with a helpful label. 🟡
git stash list                                               # List saved stashes.
git stash show -p stash@{0}                                  # Review the patch stored in the newest stash.
git stash apply stash@{0}                                    # Apply a stash but retain it as a fallback. 🟡
git stash pop stash@{0}                                      # Apply a stash and remove it only after a clean application. 🟡
git stash branch rescue/stash-work stash@{0}              # Create a branch at the stash’s original base and apply it there. 🟡
```
A stash is local and temporary—not a backup. A named WIP branch pushed to the remote is more durable for important work.
## 8.4 Recover a badly overwritten remote branch
1. Freeze further writers and avoid another blind force-push.
2. Ask anyone who had the old branch tip to create and push a `rescue/...` branch.
3. Inspect local remote-tracking reflogs and the Git host’s audit/restore tools.
4. Restore through a reviewed pull request or an explicitly approved `--force-with-lease` update.
---
# Level 9 — Investigate a defect or Git incident
## 9.1 Find the change that introduced behavior
```bash
git log --follow -- <file>                                   # Show the history of one file across renames where possible.
git log -S '<literal-text>' --all                            # Find commits that changed the number of occurrences of text.
git log -G '<regular-expression>' -p --all                   # Find patches whose added/removed lines match a regex.
git blame -L <start>,<end> -- <file>                         # Show the most recent commit that changed each line in a range.
git show <commit-sha> -- <file>                              # Inspect one historical commit’s change to a file.
git tag --contains <commit-sha>                              # List release tags that include a fix.
git branch --contains <commit-sha>                           # List branches that include a fix.
```
`git blame` shows code lineage, not responsibility. Use it as a path to history and review context, not as a blame assignment.
## 9.2 Find the first bad commit with bisect
```bash
git bisect start                                              # Start a binary search through history. 🟡
git bisect bad HEAD                                           # Mark the current version as bad. 🟡
git bisect good <known-good-tag-or-sha>                       # Mark a known working version as good. 🟡
git bisect run <deterministic-test-command>                   # Let Git run a repeatable test at each candidate commit. 🟡
git bisect reset                                              # Return to the original branch when the investigation ends. 🟡
```
For `git bisect run`, exit `0` means good, `1–124` means bad, and `125` means skip because the revision cannot be tested. Keep the test deterministic: pin dependencies/images and avoid flaky external systems.
## 9.3 Check repository health
```bash
git count-objects -vH                                        # Show local object counts and storage size.
git fsck --full --strict                                      # Check Git object/connectivity integrity.
git maintenance run --auto                                   # Run normal automatic maintenance when not recovering work. 🟡
git check-attr -a -- <file>                                  # Diagnose line-ending, LFS, diff, and merge attributes for a file.
```
If object corruption is suspected, stop writes, preserve the checkout, run `git fsck --full --strict`, and compare with a clean clone or known-good mirror. Do not manually delete pack/object files to silence an error.
---
# Level 10 — Security, governance, and self-hosted operations
## 10.1 Authentication rules
```bash
ssh-keygen -t ed25519 -C "ci-deploy-key"                    # Create a modern SSH key for a separate CI/deploy identity.
git ls-remote --heads origin                                 # Test that the configured Git remote can be contacted and read.
git remote set-url origin git@<git-host>:<org>/<repo>.git    # Change origin to an SSH remote URL. 🟡
git config --global credential.helper manager        # Use Git Credential Manager if that is your approved credential helper. 🟡
```
- Use separate least-privilege identities for people, CI, deployments, and different trust boundaries.
- Use short-lived/repo-scoped tokens or deploy keys where possible.
- Keep private keys/tokens out of remote URLs, shell history, logs, `.git/config`, committed files, and build artifacts.
- Verify SSH host fingerprints through a trusted channel. Never disable host-key checking just to make CI pass.
## 10.2 Server and repository controls
For a hosted Git provider, configure protected branches/tags, required review/checks, restricted bypass permissions, secret scanning, push protection, audit logs, and read-only default CI tokens.
For a self-hosted bare repository:
```bash
git init --bare --shared=group /srv/git/project.git          # Create a shared server-side bare repository.
git -C /srv/git/project.git config receive.denyNonFastForwards true  # Reject force updates on the server. 🟡
git -C /srv/git/project.git config receive.denyDeletes true  # Reject ref deletions on the server. 🟡
git -C /srv/git/project.git config receive.fsckObjects true  # Validate received Git objects before accepting pushes. 🟡
```
Do not push directly into a live deployment folder. Build immutable artifacts in CI, then deploy them through an explicit, auditable deployment process.
## 10.3 Back up a repository correctly
```bash
git clone --mirror <repository-url> <backup-folder>.git      # Clone all refs for a full Git mirror backup.
git -C <backup-folder>.git remote update --prune             # Refresh every remote ref in the mirror and prune deleted refs.
git -C <backup-folder>.git fsck --full                        # Validate the backup’s Git objects.
```
Keep backups separately protected and test restoration. A backup that has never been restored is only an assumption.
---
# Under-pressure mini playbooks
## A. A bad commit reached a shared branch
```bash
git fetch --prune origin                                     # Refresh the remote state before choosing a rollback.
git show <bad-commit-sha>                                    # Confirm the exact change and deployment impact.
git revert <bad-commit-sha>                                  # Create a reviewable shared-history rollback. 🟡
git push origin <protected-branch>                           # Publish the approved revert. 🔴
```
Also roll back the **deployment/infrastructure/database** separately if needed; a Git revert does not undo a running artifact or a cloud resource.
## B. A push was rejected as non-fast-forward
```bash
git fetch --prune origin                                     # Refresh the remote branch before resolving the divergence.
git log --left-right --graph --oneline HEAD...@{upstream}    # See commits unique to your branch and its upstream.
git rebase @{upstream}                                       # Replay private work on the upstream when team policy allows. 🟡
git push --force-with-lease                                  # Update a rewritten private branch only after review. 🔴
```
For a shared branch, stop and use the team’s pull-request/merge process instead of overwriting it.
## C. Find whether a release includes a fix
```bash
git merge-base --is-ancestor <fix-commit-sha> <release-tag>  # Exit 0 means the release contains the fix.
git tag --contains <fix-commit-sha>                          # List all tags/releases that contain the fix.
git log --oneline <previous-tag>..<release-tag>              # List commits introduced by the release.
```
## D. A secret was committed
1. Revoke/rotate it immediately; assume it is exposed in clones, caches, logs, forks, and backups.
2. Remove it from current files and replace it with a secret reference/template.
3. Coordinate any history rewrite using the organization’s approved process (commonly `git filter-repo`, an external tool).
4. Invalidate caches/old clones where required, audit use, and add secret scanning/push protection.
`git rm`, `.gitignore`, a revert, or a force-push by itself does not make an exposed credential safe.
---
# Final safety card — memorize these
```bash
git status -sb                                                # First command before a risky action.
git fetch --prune --tags origin                               # Refresh remote branches and tags without changing your files.
git log --oneline --graph --decorate --all                    # See the history shape before changing it.
git diff <good-ref>..<bad-ref>                                # Understand a suspected regression.
git revert <shared-bad-commit>                                # Safely undo a shared/deployed commit.
git reflog                                                    # Find recently lost local commits/branch tips.
git switch -c rescue/<name> <sha>                             # Anchor a recovery candidate before experimenting.
git push --force-with-lease origin <private-branch>           # Safest normal way to update rewritten private remote history.
git rev-parse --verify '<tag>^{commit}'                       # Resolve a release tag to its immutable commit SHA.
```
**When uncertain:** stop, run `git status -sb`, fetch, inspect the log/diff, and create a rescue branch before using any destructive command.
