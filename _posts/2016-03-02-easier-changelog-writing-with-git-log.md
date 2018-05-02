---
title: Easier Changelog writing with git log
---

Tired of writing Changelog? Here's an idea: prepend your commit messages with keywords:

- `NEW` for some new functionality
- `FIX` for fixing bugs
- `MOD` for major modifications

now run `git log --pretty=oneline --pretty=format:'%s' --grep=NEW --grep=MOD --grep=FIX` to show thise commitse.

Interested to see the changelog from previous tag? Easy: first, find out the last tag `git describe --abbrev=0 --tags` and run the previous `git log` with adding `TAG..HEAD`

Or create a short script in root of the project to do all:

```sh
#!/bin/bash


FROM=`git describe --abbrev=0 --tags`
TO=HEAD

if [ -n "$1" ]; then
  FROM=$1
fi

git log --pretty=oneline --pretty=format:'%s' ${FROM}..${TO} --grep=NEW --grep=MOD --grep=FIX 
```
