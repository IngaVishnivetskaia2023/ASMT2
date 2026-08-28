### Git fast-forward vs non-fast-forward merge

#### Fast-Forward Merge
A fast-forward merge happens when there are no new commits on the target branch since you branched off. 
Git simply **moves the branch pointer forward**.

#### before merge
```
develop:  A --- B --- C
                       \
feature:                D --- E --- F
```

#### after git merge feature - fast-forward
```
develop:  A --- B --- C --- D --- E --- F  ← develop pointer moved here
```

#### What Happens
- No merge commit was created
- Git just moved the develop pointer from C to F
- History is linear — as if you committed directly on develop
- The feature branch commits become part of develop's straight line

#### command
```
git checkout develop
git merge feature        # Fast-forwards by default when possible
git merge --ff-only feature  # Fail if fast-forward isn't possible
```

#### Non-Fast-Forward Merge (True Merge)

A non-fast-forward merge happens when both branches have diverged — there are new commits on the target branch that don't exist on your branch.

#### Before Merge
```
develop:  A --- B --- C --- G --- H
                       \
feature:                D --- E --- F
```

#### After git merge feature (non-fast-forward)
```
develop:  A --- B --- C --- G --- H --- M  ← merge commit
                       \               /
feature:                D --- E --- F
```
#### What Happened
- A merge commit (M) was created with **two parents** (H and F)
- Both histories are preserved
- The graph is non-linear (has a visible branch/merge topology)

#### command
```
git checkout develop
git merge feature              # Creates merge commit (when FF not possible)
git merge --no-ff feature      # Force merge commit even if FF is possible
```

#### Why Forcing Non-Fast-Forward (Even When FF Is Possible)
```
develop:  A --- B --- C ----------- M  ← merge commit
                       \           /
feature:                D --- E --- F
```
- Preserves the semantic grouping of feature work
- `git log --first-parent` shows only merge commits on develop
- Easy to revert entire feature with git revert M
- Clear record that D, E, F were a logical unit

#### comparison
| Aspect | Fast-Forward | Non-Fast-Forward |
| :-- | :-- | :-- |
| Merge commit | ❌ None | ✅ Created (two parents) |
| History shape | Linear | Diamond/branching |
| When possible | Target hasn't diverged | Always possible |
| Information preserved | Less — branch boundary lost | More — branch boundary visible |
| Revert feature | Must revert each commit individually | git revert -m 1 <merge-sha> |
| git log | Flat list, no grouping | Visible branch topology |
| Bisect complexity | Simple linear search | May need to navigate branches |



### How Rebase Relates to Fast-Forward?

Rebase rewrites your branch to make fast-forward possible again:
```
Before (diverged — would require merge commit):
develop:  A --- B --- C --- G --- H
                       \
feature:                D --- E --- F

After `git rebase develop` on feature:
develop:  A --- B --- C --- G --- H
                                    \
feature:                             D' --- E' --- F'

Now fast-forward is possible:
develop:  A --- B --- C --- G --- H --- D' --- E' --- F'
```
This is why teams use rebase + fast-forward workflow — it keeps history linear.

#### Why Teams Prefer Linear History with Git Rebase

- **1. core reason:** A linear history makes your Git log read like a story — each commit is a logical step forward, with no tangled branching noise.

- 2. `git bisect` Is Dramatically Easier

- 3. `git blame` Points to Meaningful Commits

- 4.  Cherry-Picking Is Clean


### Trunk-Based Development (No Long-Lived Branches)

Commit directly to main/develop, or use very short-lived branches (< 1 day).

```
# Option A: Direct commit
git checkout main
git commit -m "Add payment validation"
git push

# Option B: Short branch, immediate merge
git checkout -b fix/typo
git commit -m "Fix typo in payment"
git checkout main
git merge --ff-only fix/typo
git push
git branch -d fix/typo
```




















