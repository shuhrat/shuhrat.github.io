---
title: "Git LFS tips and tricks"
categories: programming
---

A couple of weeks ago we started to use Github's [Large File Storage][git-lfs] extension for git. That was a real challenge and now I want to share some tips and tricks with you.


#### 1. Get rid of `git lfs install`

`git lfs install` does actually two things:

1. Adds pre-push hook to the repository, preventing you from pushing when LFS client is not installed
2. Adds some configurations to `<repository>/.git/.gitconfig`

Add LFS client's configurations to your global `$HOME/.gitconfig`:

```bash
git config --global filter.lfs.required true
git config --global filter.lfs.clean "git-lfs clean %f"
git config --global filter.lfs.smudge "git-lfs smudge %f"
```

You won't have to initialize LFS every time you clone a repository.


#### 2. Credentials

Change your remotes' protocols to `ssh`, so you will not need to use git's [credential-cache][credential-cache].
Otherwise, you have to type login and password for every file you download from the GH media server.


#### 3. It takes long time to fetch

LFS has the batch mode for downloading files from a server. But it does not work on clone and checkout due to limitation of git's smudge filters.
You can skip LFS's smudge filter and fetch LFS objects on demand. For that:

1\. Change the smudge filter configuration:

```bash
git config --global filter.lfs.smudge "git-lfs smudge --skip %f"
```
 Run `git lfs env` and be sure that the smudge filter is skipped.

2\. Fetch LFS objects after checkout or clone:
 
```bash
git lfs fetch # downloads objects with batch mode
git lfs checkout # changes objects to binary files
```


#### 4. Commited binary file, can not rebase, checkout or hard reset

You've got a trouble if someone commits binary file and pushes it to remote. After rebase from remote everybody 
will have that file on stage, it cannot be removed or resetted.

Pre-push hook provided by Github will not save you. It will check whether LFS client is installed or not, exactly that.
You could commit a binary file and install LFS client just before pushing.

As a workaround, you can:

1. Revert commit with binary file
2. Remove LFS client configurations from `.gitconfig`. It can be checked easily by `git lfs env`
3. Rebase
4. Get configurations back to `.gitconfig`

Since that solution won't prevent this problem in the future we wrote a pre-commit hook:

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

But you can't commit changes in the .git folder, for that purpose we use small, but powerful 
manager [git-hooks-js][git-hooks-js]. Simply, you can save a hook to `.githooks/pre-commit/git-lfs.sh`, and 
provide it to team members. For more information see [documentation][git-hooks-js-doc].

And remember, your friends are:

```bash
git lfs env
GIT_TRACE=1 git lfs <cmd>
git lfs logs last
```

[git-lfs]: https://git-lfs.github.com/
[credential-cache]: http://git-scm.com/docs/git-credential-cache
[git-hooks-js]: https://github.com/tarmolov/git-hooks-js
[git-hooks-js-doc]: https://github.com/tarmolov/git-hooks-js/blob/master/README.md
