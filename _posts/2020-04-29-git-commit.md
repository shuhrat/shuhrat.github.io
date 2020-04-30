---
title: Git commit messages
categories: programming
---

From time to time I come across with poor commit messages in GitHub. Those commit messages look like,
```
* final
* better
* fix3
* another fix
* fix
```

I was thinking on how to discipline myself to write good commit messages that help me, and my team some time later. 
So I collected some good practices and want to share, maybe it will be helpful for others. 

## Avoid laziness, let Git to help you, squeeze juices out of it.

### Compose a template for commit messages 

```
# <type>: <subject>
# |<----  Using a Maximum Of 50 Characters  ---->|

# Explain why this change is being made
# |<----   Try To Limit Each Line to a Maximum Of 72 Characters   ---->|

# Provide links or keys to any relevant tickets, articles or other resources
# ISSUE: ISSUE-LINK

# --- COMMIT END ---
#
# Type can be:
#   fix      bug fix
#   feat     new feature
#   docs     changes to documentation
#   style    formatting, missing semi colons, etc; no code change
#   refactor refactoring production code
#   test     adding missing tests, refactoring tests; no production code change
#
```

Save it to your local machine, let's say in `$HOME/.git-commit.template`. To apply the template,

```console
$ git config --global commit.template $HOME/.git-commit.template
```

### Text editor

Configure your favorite editor to use it with Git.

I prefer Sublime Text to draw up commit messages,

```console
$ git config --global core.editor "subl --new-window --wait"
```

Configurations for other popular editors can be found in GitHub [post][associating-text-editors-with-git].

[associating-text-editors-with-git]: https://help.github.com/en/github/using-git/associating-text-editors-with-git

#### Spell checking

Power up your editor with spell checking. For Sublime Text check for a [documentation][spell_checking].

[spell_checking]: https://www.sublimetext.com/docs/3/spell_checking.html

### Verbose git commit

Make your git commit verbose, 

```console
$ git config −−global commit.verbose
```

All changes will appear under your hand, in text editor. It gives you context to come up with nice and meaningful commit messages.

Keep your commits atomic, small and commit often. And remember, your best friend is interactive rebase.
