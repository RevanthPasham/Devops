# Ultimate Git README

This README contains Git setup, commits, branches, merging, cherry-pick, rebase, stash, tags, recovery and undo commands.

## Highlights

- git init
- git clone
- git add
- git commit
- git reset
- git restore
- git branch
- git switch
- git checkout
- git merge
- git cherry-pick
- git rebase
- git stash
- git tag
- git reflog
- git remote
- git push
- git pull

## Branch Commands

| Action | Command | Undo |
|----------|----------|----------|
| Create branch | git branch feature | git branch -D feature |
| Create and switch | git checkout -b feature | switch back then delete |
| Create and switch (modern) | git switch -c feature | switch back then delete |
| Delete branch | git branch -d feature | recover using reflog |
| Force delete | git branch -D feature | recover using reflog |

## Merge Commands

| Method | Command | Undo |
|----------|----------|----------|
| Standard merge | git merge feature | git reset --hard ORIG_HEAD |
| No fast forward | git merge --no-ff feature | git reset --hard ORIG_HEAD |
| Fast forward | git merge --ff-only feature | git reset --hard ORIG_HEAD |
| Squash merge | git merge --squash feature | git reset --hard HEAD |
| Abort merge | git merge --abort | N/A |

## Cherry Pick

| Action | Command | Undo |
|----------|----------|----------|
| Single commit | git cherry-pick HASH | git reset --hard HEAD~1 |
| Multiple commits | git cherry-pick HASH1 HASH2 | git reset --hard HEAD~n |
| Abort | git cherry-pick --abort | N/A |

## Recovery

| Problem | Command |
|----------|----------|
| Recover deleted commit | git reflog |
| Recover deleted branch | git checkout -b branch HASH |
| Restore commit | git reset --hard HASH |
