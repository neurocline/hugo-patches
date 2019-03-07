---
title: "Patch: Quiet Feedback"
date: 2019-03-06T22:34:01-08:00
---

## Change --quiet to also suppress FEEDBACK loggers

Hugo doesn't really have a quiet mode - even without verbose logging, there's
a fair amount of chatter written to the screen. This patch makes `--quiet`
disable the FEEDBACK logger so that we really have a quiet mode. Messages of
level `ERROR` and above will still be displayed.

This requires a patched jwalterweatherman library; until that patch is
accepted to [spf13/jwalterweatherman][], this patch uses the version at
[neurocline/jwalterweatherman][]; see [jww commit 325f63e917][].

[spf13/jwalterweatherman]: https://github.com/spf13/jwalterweatherman
[neurocline/jwalterweatherman]: https://github.com/neurocline/jwalterweatherman
[jww commit 325f63e917]: https://github.com/neurocline/jwalterweatherman/commit/325f63e91740255bf9a974481b707875ac41b414

The reason for the dependency on a patch to jwalterweatherman is that
the alternative is to wrap every usage of a FEEDBACK logger in Hugo with
a check against `--quiet`; this is error-prone and invasive. Instead, we
add a `SetQuiet` method to the `common/loggers` package, which enables or
disables the FEEDBACK loggers in jwalterweatherman.

See [hugo commit 1e53050aa7][] for the full details.

[hugo commit 1e53050aa7]: https://github.com/neurocline/hugo/commit/1e53050aa750cf9f4fb0136aa1b2a4c412772d8f

## jwalterweatherman: Add control of FEEDBACK loggers

The jwalterweatherman library has a set of error/status loggers, and a
separate FEEDBACK logger. Unfortunately, there is no mechanism to enable or
disable output for FEEDBACK, unlike for the other loggers.

This patch adds a new global function `EnableDisableFeedback()`, and a per-log
function of the same name. By default, the FEEDBACK logger is enabled; if
it is disabled at runtime, then FEEDBACK output is suppressed.

The point of this patch is to allow applications using jwalterweatherman to
implement a feature like `--quiet`.

This patch should have no affect on programs using jwalterweatherman, as this
adds a new API call and the default behavior is as before.

Bikeshedding on the API used to enable/disable the FEEDBACK logger is welcome.

See [jww commit 325f63e917][] for the full details.
