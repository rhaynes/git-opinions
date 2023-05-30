# git-opinions
A collection of my opinions on git usage and some helpful common commands

# The most common commands
Don't leave home without these

### List all available branches (* next to current branch)
Useful, but also see [better-branch](https://github.com/rhaynes/better-branch)
```
git branch
```

### Checkout any branch (my convention is for branch names to have hyphens)
```
git checkout branch-name
```

### Get the latest code from the remote
```
git pull origin
```

### Merge the latest default into your current branch
```
git pull origin main
```

### Create a new local branch
```
git checkout -b branch-name
```

### Add a file to the "stage" (the pre-commit zone)
```
git add filename.c
```

# Add all files to the stage (excludes anything in .gitignore by default)
```
git add *
```

# Remove a file from the stage
```
git reset filename.c
```

# View the stage and currently selected branch
```
git status
```

### Commit what's on the stage to currently selected branch (good idea to run git branch or git status first)
```
git commit -m "Descriptive message"
```

Note that the tense of the descriptive message is up for debate. You should follow the convention of the repo you're contrinbuting to but here's some general advice:

* Present tense - Use present tense like "adds some feature" if you're offering commits that could in theory be accepted OR rejected. By doing this you're basically saying "this commit adds some feature". This is especially appropriate when adding to an open source repo.
* Past tense - This is the convetion I generally follow. If you use "added some feature", you're assuming that all commits are accepted so you're using the commit history as a log. By doing this you're saying "I added some feature".


### Push to the remote if branch exists and create it if it doesn't
```
git push origin branch-name
```

### Remove last local commit and return everything to the pre-staged state
Note: 
* does not work if you've already pushed to remote b/c you'd need to do a force push
* Change 1 to N if you need to go back N commits instead of just 1
```
git reset --soft HEAD~1
```

## DANGER ZONE:
### Delete local branch
Note: Do this only after merge... it's possible but difficult to recover branches b/c you need to know the commit hash of the branch. It's better to use [better-branch](https://github.com/rhaynes/better-branch) instead for an interactive command that checks the remote and makes sure all the commits have been merged.
```
git branch -d branch-name
```

### DELETE EVERYTHING SINCE LAST COMMIT 
(really dangerous, no possible way back, you lose all your uncommitted work if you aren't paying attention when you run this)
```
git reset --hard
```
Seriously, make sure you know this is what you want to do before doing it.

# Common workflows
With the commands above, you can create several common workflows.

### Start a new branch from latest main
This assumes you're using [better-branch](https://github.com/rhaynes/better-branch) and that it's in `/usr/local/bin` which is the easiest to make sure everything is always up to date. I've also listed the manual commands below if you're not using better-branch.
```
branch # interactive prompt (should switch to main and pull latest)
git checkout -b new-branch-name
```

The manual way:
```
git checkout main
git pull origin
git checkout -b new-branch-name
```
Note the `git pull origin` command, without it you can end up branching off an earlier version of the branch than the latest.

### Start a new branch from current branch
This is useful if you want to make additions to a branch someone else is working on or if you want to test something out on your current branch but not merge. Generally the better approach is to merge the development branch to main and then branch from main, but there are reasons to use this workflow if the development branch happens to be long lived.
```
git checkout existing-branch-to-branch-from
git pull origin
git checkout -b new-branch-name
```

### Commit and push updates
This is really important to get right so is worth taking time to do it. Start by seeing what's currently happening.
```
git status
```

With this command you'll see the branch name and items in red and green. You'll also see up to 3 sections: 
* the items in green - this is what will be committed if you run `git commit`. These items are called the "staged" items b/c they are on the "stage" for commit
* the items in red that are modified - these are the "unstaged" items. They can be added to the stage with `git add` and deleted completely with `git reset --hard`
* the items that are untracked - these are "untracked" items. They can be added to the stage with `git add` and importantly they are the only things not affected by `git reset --hard`

Keep in mind you may see only 1 section (for example untracked items or unstaged items). This all depends on what changes you've made. If you see no output when running `git status` other than the branch name, you have no changes to the repo and the workspace is clean.

After changes, initially nothing should be in green b/c nothing has been staged. I've found that the best workflow is to go item by item or at the most, folder by folder, to add changes to the stage. This makes sure you know exactly what you're committing.
```
git add full/path/to/file/to/add
```
Do this several times to get all the changes added. You might also run `git diff` first to see what you changed. Sometimes this helps you discover where you left debug code in:
```
git diff full/path/to/file/to/check
```
You can also just run `git diff` without any arguments and you can see all the changes made but sometimes this is cumbersome with a large number of changed files.

An alternative workflow is `git add *` or `git add folder-name/*` which will add everything to the stage. Only do this if you're absolutely sure you want to commit everything and don't have any cleaning up to do. Also run `git status` after this command just to make sure you didn't add a bunch of things you didn't mean to. The stage sometimes only lists a folder name so `git add *` can add 100s of files inside a folder that were not listed in the original `git status`.

Given all the above, here is a typical workflow that I use:
```
git status # read over list of files
git diff file1
git add file1
git diff file2
git add file2
git add folder/* # when I'm sure I want to add the entire folder
git status # a check to see that everything in green looks right, if not start from the top
```

At some point you might discover with `git status` that you added something that you didn't mean to. It's always safe to use `git reset` as long as you don't add `--hard` to it. `git reset` just pulls things off the stage so everything goes from green to red. Here's an example
```
git status
git add file1
git add folder/*
git status # oops... we discover here we added things we didn't want to from folder/*
git reset folder/*
```

Here's a workflow from `status` all the way to `commit`:
```
git status  # read over list
git diff  # check to see what changes you're committing (remove debug code if you see it)
git add file1
git add file2
git add file3
git status  # one final check, things in green will be committed
git commit -m 'Added feature a, b, and c'
git push origin branch-name
```
Always push after the commit just so you have a remote backup.

# Getting yourself out of a bind
It can happen that you accidentally commit the wrong files or commit to the wrong branch (like committing to main) even after taking all the steps above. Hopefully you realize this before pushing to the remote b/c it's easy to fix if you haven't run `git push` yet. Sometimes you discover it when you try `git push origin branch-name` and git rejects the push b/c you'd be pushing from `main`. If you haven't pushed, it's an easy fix:
```
git reset --soft HEAD~1
```
This will return the stage to its state right before the commit and you can go ahead and add things correctly.

If you have pushed, you might need to force push to the remote after updating the branch. Be careful because this can be dangerous since the `--force` command it telling git to ignore it's safety checks. Also, keep in mind that anyone who has pulled this branch since the update will no longer be able to push b/c their local branch will have diverged from the updated remote. This workflow will overwrite the remote branch:
```
git reset --soft HEAD~1
git status  # read over the list
git add file1
git add file2
git status #  check the stage
git commit -m 'Updated commit'
git push origin branch-name --force
```
