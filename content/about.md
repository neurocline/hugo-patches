---
title: "About"
date: 2019-03-07T00:37:24-08:00
---

This site has extended patch notes for the patches I'm making for Hugo.
Some of the patches are big in scope, others change behavior, so they
need more than just a commit message.

I am maintaining an [enhanced version of Hugo][] until these patches are
accepted. It contains the following patches:

- [Quiet Feedback]({{< relref "patch-quiet-feedback.md" >}}) (branch: [patch-quiet-feedback][])
- [Verbose Levels]({{< relref "patch-verbose-levels.md" >}}) (branch: [patch-verbose-levels-enhanced][])
- [Cleaner Output]({{< relref "patch-shorter-output.md" >}}) (branch: [patch-cleaner-output][])
- static files performance (branch: [patch-static-files-perf][])
- limit num workers (branch: [patch-num-workers][])
- fix path generation (branch: [patch-paths-fix][])

[enhanced version of Hugo]: https://github.com/neurocline/hugo/commit/baa858e8f77fdfe6080d6f11820602e5f9ae2086

[patch-quiet-feedback]: https://github.com/neurocline/hugo/commit/1e53050aa750cf9f4fb0136aa1b2a4c412772d8f
[patch-verbose-levels-enhanced]: https://github.com/neurocline/hugo/commit/e0f974139e4d01b00e60ff76547fbf24016b4d6e
[patch-cleaner-output]: https://github.com/neurocline/hugo/commit/56cf4436d6749a199000c478c9f724f1873765bd
[patch-static-files-perf]: https://github.com/neurocline/hugo/commit/e42b84528093d1110094ce45c3f3b0b03ca10bba
[patch-num-workers]: https://github.com/neurocline/hugo/commit/7fa800cf63c2d586fe4c5a6f5c7aa3d7696f2405
[patch-paths-fix]: https://github.com/neurocline/hugo/commit/206199a5db88f3936f05e66aa4d7c12ccd383562

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
$ git merge --no-ff patch-quiet-feedback
$ git merge --no-ff patch-verbose-levels-enhanced
$ git merge --no-ff patch-cleaner-output
$ git merge --no-ff patch-static-files-perf
$ git merge --no-ff patch-num-workers
$ git merge --no-ff patch-paths-fix
```

### Windows

Here is a batch file that will create the `hugo-enhanced` branch from `master`, with a minimum
of error handling.

```
@echo off
git checkout -b hugo-enhanced master
if %errorlevel% neq 0 goto :FATAL

for %%G in (patch-quiet-feedback,patch-verbose-levels-enhanced,patch-cleaner-output,patch-static-files-perf,patch-num-workers,patch-paths-fix) do (
  echo:
  echo merging %%G
  git merge --no-ff %%G
  if %errorlevel% neq 0 goto :FATAL
)
exit /B 0

:FATAL
echo Error %errorlevel%
```

### Linux/Mac

Write bash script to do the same.
