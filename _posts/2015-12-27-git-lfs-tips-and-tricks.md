---
title: "Git LFS tips and tricks"
categories: programming
---

A couple of weeks ago we started to use Github's [Large File Storage](git-lfs) extension for git. That was a really challange, now 
I want to share some tips and tricks with you.


#### 1. Get rid of `git lfs install`

`git lfs install` does actually two things:

1. Adds pre-push hook to the repository, which prevents you to push if LFS client is not installed
2. Adds some configurations to rpository/.git/.gitconfig

Add LFS client's configurations to your global `$HOME/.gitconfig`.

```bash
git config --global filter.lfs.required true
git config --global filter.lfs.clean "git-lfs clean %f"
git config --global filter.lfs.smudge "git-lfs smudge %f"
```

It will save you from having to initialize LFS every time after you clone repository.


#### 2. Credentials

Change your remotes' protocols to `ssh`, so you will not need to use git's [credential-cache](credential-cache).
Otherwise, you have to type login and password for every file you download from GH media server.


#### 3. It takes long time to fetch

LFS has batch mode to download files from server. But it does not work on clone and checkout due to limitation of git's smudge filters.
You can skip LFS's smudge filter and fetch LFS objects on demand. For that:

##### 1. Change smudge filter configuration.

```bash
git config --global filter.lfs.smudge "git-lfs smudge --skip %f"
```
 Run `git lfs env` and be sure that smudge filter is skipped.

##### 2. Fetch LFS objects after checkout or clone 
 
```bash
git lfs fetch # downloads objects with batch mode
git lfs checkout # changes objects to binary files
```


#### 4. Commited binary file, can not rebase, checkout or hard reset

You have a trouble if anyone commits binary file and pushes to remote. Anyone who rebased 
from remote now have that file on stage, file cannot be removed, reseted.

Pre-push hook provided by Github will not save you. It checks if LFS client is installed or not, nothing less nothing more.
You could commit binary file and install LFS client just before pushing.

As a workaround, you can:

1. Revert commit with binary file
2. Remove LFS client configurations from `.gitconfig`, can be checked easily by `git lfs env`
3. Rebase
4. Turn back configurations to `.gitconfig`

Such solution won't prevent problem from repeating in future. So, we wrote a pre-commit hook.

```bash
#!/usr/bin/env bash

set -e

BINARY_FILES=""
CHANGED_FILES=$(git diff --cached --name-only --diff-filter=ACM)
LFS_FILES=$(echo $CHANGED_FILES | xargs git check-attr filter | grep 'filter: lfs$' | sed -e 's/: filter: lfs//')

for FILE in $LFS_FILES; do
    SOFT_SHA=$(git hash-object -w $FILE)
    RAW_SHA=$(git hash-object -w --no-filters $FILE)

    if [ $SOFT_SHA == $RAW_SHA ]; then
        BINARY_FILES="$FILE\n$BINARY_FILES"
    fi
done

if [[ -n "$BINARY_FILES" ]]; then
    echo "Attention!"
    echo "----------"
    echo "You tried to commit binary files:"
    echo -e "\x1B[31m$BINARY_FILES\x1B[0m"
    echo "Revert your changes and commit those files with git-lfs!"
    echo "----------"
    exit 1
fi
```

But you can't commit changes in .git folder, for that purposes we use small, but powerfull 
manager [git-hooks-js](git-hooks-js). Simply, you can save hook to `.githooks/pre-commit/git-lfs.sh`, and 
provide it to team members, for more information see [documentation](git-hooks-js-doc)

And remember, your friends are

```bash
git lfs env
GIT_TRACE=1 git lfs <cmd>
git lfs logs last
```

[git-lfs]: https://git-lfs.github.com/
[credential-cache]: http://git-scm.com/docs/git-credential-cache
[git-hooks-js]: https://github.com/tarmolov/git-hooks-js
[git-hooks-js-doc]: https://github.com/tarmolov/git-hooks-js/blob/master/README.md
