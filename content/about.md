---
title: "About"
date: 2019-03-07T00:37:24-08:00
---

This site has extended patch notes for the patches I'm making for Hugo.
Some of the patches are big in scope, others change behavior, so they
need more than just a commit message.

I am maintaining an [enhanced version of Hugo][] until these patches are
accepted. It contains the following patches:

- [LF line endings]({{< relref "patch-lf-go-mod.md" >}}) (branch: [patch-lf-go-mod][])
- [Quiet Feedback]({{< relref "patch-quiet-feedback.md" >}}) (branch: [patch-quiet-feedback][])
- [Verbose Levels]({{< relref "patch-verbose-levels.md" >}}) (branch: [patch-verbose-levels][])
- [Cleaner Output]({{< relref "patch-shorter-output.md" >}}) (branch: [patch-cleaner-output][])
- [Static Files Perf]({{< relref "patch-static-file-perf.md" >}}) (branch: [patch-static-files-perf][])
- [Limit num workers]({{< relref "patch-num-workers.md" >}}) (branch: [patch-num-workers][])
- [Fix path-absolute-URLs]({{< relref "patch-path-abs-url.md" >}}) (branch: [patch-path-abs-url][])
- fix path-relative-URLs

[enhanced version of Hugo]: https://github.com/neurocline/hugo/tree/hugo-enhanced

[patch-lf-go-mod]: https://github.com/neurocline/hugo/commit/4a6a6b78f2e9306127fd6685288cb6ad08777bcd
[patch-quiet-feedback]: https://github.com/neurocline/hugo/commit/eb0132b48787e25e1081c0189beda5ea5f00c318
[patch-verbose-levels]: https://github.com/neurocline/hugo/commit/507d08c60c36543ec5a65a28b7db9a04ec3da2bd
[patch-cleaner-output]: https://github.com/neurocline/hugo/commit/3af4e13c0575c66ac1eeafa7a1b856ed2a834f70
[patch-static-files-perf]: https://github.com/neurocline/hugo/commit/d2be6f785f2531b49ee6a1474e2ad9d7489e4514
[patch-num-workers]: https://github.com/neurocline/hugo/commit/dedeab73f8d52425702260eba6e83fd303991d36
[patch-path-abs-url]: https://github.com/neurocline/hugo/commit/98f58ec5a147679e8b36de7d7d15adac33f022a2

Not all of these patches have been submitted to the Hugo team yet. When patches are accepted,
I'll note that.

This site is built with my enhanced version of Hugo. The most important patch was the
`patch-paths-fix` branch, which makes `canonifyURLS`, `relativeURLs`, and the lack of either
actually work properly even with a `baseURL` that has a path.

## Rebuilding the `hugo-enhanced` branch.

We force merges in all cases so that it's clear that the `hugo-enhanced`
branch is merging in a set of patches.

```
$ git checkout -b hugo-enhanced master
$ git merge --no-ff patch-lf-go-mod
$ git merge --no-ff patch-quiet-feedback
$ git merge --no-ff patch-verbose-levels
$ git merge --no-ff patch-cleaner-output
$ git merge --no-ff patch-static-files-perf
$ git merge --no-ff patch-num-workers
$ git merge --no-ff patch-path-abs-url
```

### Windows

Here is a batch file that will create the `hugo-enhanced` branch from `master`, with a minimum
of error handling.

```
@echo off
git checkout -b hugo-enhanced master
if %errorlevel% neq 0 goto :FATAL

for %%G in (patch-lf-go-mod,patch-quiet-feedback,patch-verbose-levels,patch-cleaner-output,patch-static-files-perf,patch-num-workers,patch-path-abs-url) do (
  echo:
  echo merging %%G
  git merge --no-ff %%G
  if %errorlevel% neq 0 goto :FATAL
)
exit /B 0

:FATAL
echo Error %errorlevel%
```

How annoying. I've made a merge conflict. It shouldn't be a merge conflict, two patches
introduce distinct changes, but it looks like a merge conflict to Git, because I'm changing
what appears to be the same line.

### Linux/Mac

Write bash script to do the same.

## Rebasing the patches

Before submitting patches, they need to be rebased on the most current `master`.

```
$ git fetch origin
$ git checkout master
$ git merge --ff-only origin/master
$ git rebase patch-lf-go-mod master
$ git rebase patch-quiet-feedback master
$ git rebase patch-verbose-levels master
$ git rebase patch-cleaner-output master
$ git rebase patch-static-files-perf master
$ git rebase patch-num-workers master
$ git rebase patch-paths-fix master
```
