---
published: true
title: GIT SVN Crashcourse
---

Let's say your colleagues are working in SVN, but you prefer GIT. There is a solution for it - `git svn`. There are a lot of guides on how to use it, but most of the times they are relating to a standard SVN repository. What if yours is not?

## Cloning

Let's say the repo url is `https://svn.example.com/svn/projects/libraries/lib1` and there you have a standard folders like `trunk`, `branches` and `tags`.

Her's how to clone it:

`git svn clone --no-minimize-url --stdlayout https://svn.example.com/svn/projects/libraries/lib1`

or instead of `git svn clone ...` do `git svn init ...` to initialize it and later on, inside the folder do `git svn fetch` to fetch the content.

Here

- `--stdlayout` will tell that there are `trunk`, `branches` and `tags` folders under `lib1` project.
- `--no-minimize-url` will indicate that the whole url is the path to the project, otherwise it will try to clone the root url.

Next, run `git svn show-ignore > .git/info/exclude` to tell GIT to ignore the files that SVN is ignoring

## Pulling

`git svn rebase` will pull any changes from remote and rebase your local repo on top of it.

## Committing

Do your stuff, `git commit` to save it, `git svn dcommit` to "push" it

## Other usefull commands
- `git svn info` - show info from remote, with last update status
- `git reset --hard` - resets the current repo to the last commit

## Other resourses
- https://git-scm.com/book/en/v1/Git-and-Other-Systems-Git-and-Subversion
