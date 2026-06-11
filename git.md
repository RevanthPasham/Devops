# Git Complete Handbook

A practical Git reference with:

- Command
- Undo Command
- Meaning
- When to Use

---

# 1. Repository Setup

## git init

| Action | Command | Undo / Reverse | Meaning | When To Use |
|----------|----------|----------|----------|----------|
| Create Repository | `git init` | Delete `.git` folder | Initializes Git repository | New project |
| Reinitialize Repo | `git init` | N/A | Recreates Git metadata | Existing project |

---

## git clone

| Action | Command | Undo / Reverse | Meaning | When To Use |
|----------|----------|----------|----------|----------|
| Clone Repository | `git clone URL` | Delete folder | Download repo locally | Existing project |
| Clone Specific Branch | `git clone -b dev URL` | Delete folder | Clone one branch | Branch specific work |
| Shallow Clone | `git clone --depth 1 URL` | Delete folder | Latest commits only | Large repositories |

---

# 2. Staging Files

## git add

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Add All Files | `git add .` | `git reset` | Stage all files |
| Add Single File | `git add app.js` | `git reset app.js` | Stage one file |
| Add Tracked Files | `git add -u` | `git reset` | Stage modified files |
| Interactive Add | `git add -p` | `git reset` | Stage selected changes |

---

# 3. Commits

## git commit

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Create Commit | `git commit -m "msg"` | `git reset --soft HEAD~1` | Remove commit keep staged |
| Create Commit | `git commit -m "msg"` | `git reset HEAD~1` | Remove commit keep changes |
| Create Commit | `git commit -m "msg"` | `git reset --hard HEAD~1` | Remove commit completely |
| Amend Commit | `git commit --amend` | Re-amend | Edit latest commit |

---

# 4. Reset

## git reset

| Action | Command | Result |
|----------|----------|----------|
| Soft Reset | `git reset --soft HEAD~1` | Remove commit keep staged |
| Mixed Reset | `git reset HEAD~1` | Remove commit keep changes |
| Hard Reset | `git reset --hard HEAD~1` | Delete commit and changes |
| Unstage Files | `git reset` | Remove staged files |

---

# 5. Restore

## git restore

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Restore File | `git restore file.txt` | Edit again | Discard local changes |
| Restore Staged File | `git restore --staged file.txt` | `git add file.txt` | Unstage file |
| Restore All | `git restore .` | N/A | Remove all local changes |

---

# 6. Branches

## git branch

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Create Branch | `git branch feature` | `git branch -D feature` | Create branch |
| List Branches | `git branch` | N/A | Show branches |
| List All Branches | `git branch -a` | N/A | Local + remote |
| Rename Branch | `git branch -m old new` | Rename back | Rename branch |
| Delete Branch | `git branch -d feature` | Recover using reflog | Delete merged branch |
| Force Delete | `git branch -D feature` | Recover using reflog | Delete unmerged branch |

---

## git switch

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Switch Branch | `git switch main` | Switch back | Change branch |
| Create + Switch | `git switch -c feature` | Delete branch | Create and move |

---

## git checkout

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Checkout Branch | `git checkout main` | Checkout previous | Switch branch |
| Create Branch | `git checkout -b feature` | Delete branch | Create + switch |
| Restore File | `git checkout -- file.txt` | Edit file | Restore file |

---

# 7. Merging

## git merge

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Standard Merge | `git merge feature` | `git reset --hard ORIG_HEAD` | Merge branches |
| Fast Forward | `git merge --ff-only feature` | `git reset --hard ORIG_HEAD` | Linear history |
| No Fast Forward | `git merge --no-ff feature` | `git reset --hard ORIG_HEAD` | Create merge commit |
| Squash Merge | `git merge --squash feature` | `git reset --hard HEAD` | Single commit merge |
| Abort Merge | `git merge --abort` | N/A | Cancel merge |

---

# 8. Cherry Pick

## git cherry-pick

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Single Commit | `git cherry-pick HASH` | `git reset --hard HEAD~1` | Copy commit |
| Multiple Commits | `git cherry-pick HASH1 HASH2` | Reset | Copy commits |
| Commit Range | `git cherry-pick A^..B` | Reset | Copy range |
| Continue | `git cherry-pick --continue` | Abort | Continue |
| Abort | `git cherry-pick --abort` | N/A | Cancel |

---

# 9. Rebase

## git rebase

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Rebase Branch | `git rebase main` | `git rebase --abort` | Move commits |
| Interactive Rebase | `git rebase -i HEAD~5` | Abort | Rewrite history |
| Continue Rebase | `git rebase --continue` | Abort | Continue |
| Abort Rebase | `git rebase --abort` | N/A | Cancel |

---

# 10. Stash

## git stash

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Save Work | `git stash` | `git stash pop` | Temporary save |
| Named Stash | `git stash save "msg"` | Pop | Save with name |
| List Stash | `git stash list` | N/A | View stash |
| Apply Stash | `git stash apply` | Reset | Apply keep stash |
| Pop Stash | `git stash pop` | N/A | Apply remove stash |
| Delete Stash | `git stash drop` | Reflog | Delete stash |

---

# 11. Tags

## git tag

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Create Tag | `git tag v1.0` | `git tag -d v1.0` | Release marker |
| Annotated Tag | `git tag -a v1.0 -m "release"` | Delete tag | Detailed tag |
| Push Tag | `git push origin v1.0` | Delete remote tag | Upload tag |
| List Tags | `git tag` | N/A | Show tags |

---

# 12. Reflog

## git reflog

| Action | Command | Purpose |
|----------|----------|----------|
| View History | `git reflog` | Show HEAD history |
| Recover Commit | `git reset --hard HASH` | Restore commit |
| Recover Branch | `git checkout -b branch HASH` | Restore branch |

---

# 13. Remote Repositories

## git remote

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Add Remote | `git remote add origin URL` | `git remote remove origin` | Connect remote |
| Show Remotes | `git remote -v` | N/A | List remotes |
| Rename Remote | `git remote rename old new` | Rename back | Rename |
| Remove Remote | `git remote remove origin` | Add again | Remove connection |

---

# 14. Push

## git push

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Push Branch | `git push origin main` | Revert commit | Upload changes |
| Push New Branch | `git push -u origin feature` | Delete branch | Track branch |
| Force Push | `git push --force` | Recover via reflog | Replace history |
| Safe Force Push | `git push --force-with-lease` | Recover via reflog | Safer force push |

---

# 15. Pull

## git pull

| Action | Command | Undo | Meaning |
|----------|----------|----------|----------|
| Pull Updates | `git pull origin main` | `git reset --hard ORIG_HEAD` | Fetch + Merge |
| Pull Rebase | `git pull --rebase` | `git rebase --abort` | Cleaner history |
| Fetch Only | `git fetch` | N/A | Download only |

---

# 16. Emergency Recovery

| Problem | Fix |
|----------|----------|
| Added wrong files | `git reset` |
| Wrong commit message | `git commit --amend` |
| Wrong commit | `git reset --soft HEAD~1` |
| Delete commit | `git reset --hard HEAD~1` |
| Wrong merge | `git merge --abort` |
| Wrong rebase | `git rebase --abort` |
| Wrong cherry-pick | `git cherry-pick --abort` |
| Deleted branch | `git reflog` |
| Deleted commit | `git reflog` |
| Lost work | `git fsck --lost-found` |
| Wrong push | `git revert` |
| Force push mistake | `git reflog` |

---

# Golden Rule

Before using:

```bash
git reset --hard
git push --force
git rebase -i
```

Always run:

```bash
git reflog
```
