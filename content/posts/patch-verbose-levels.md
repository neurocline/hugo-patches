---
title: "Patch - Verbose Levels"
date: 2019-03-06T23:04:13-08:00
---

## Add levels to --verbose

Hugo uses jwalterweatherman and has a fair amount of logging controlled by different
levels. However, it's a little awkward to get at it.

The `--verbose` switch is now a string var with default value `verbose`, and can
take on extra values:

- `--verbose` defaults to INFO
- `--verbose=warn` for WARN (least verbose)
- `--verbose=info` for INFO
- `--verbose=debug` for DEBUG
- `--verbose=trace` for TRACE (most verbose)

This only alters console logging - file-based logging is as before, controlled with
`--log`, `--logFile`, and `--verboseLog`. This should probably all be unified.

The older vars and options are still supported, but `--verbose=<level>` takes precedence. Here
is the cascade

- `--quiet`: ERROR level
- nothing: WARN level
- `--verbose` only: INFO level
- `--verbose` + `--debug`|`debug`: DEBUG level
- `--verbose=<level>`: `<level>` level (any of WARN, INFO, DEBUG, TRACE)

This also updates quiet: if `--quiet` is used, console logging is set to ERROR level, independent
of any log settings.

It's a little goofy in that the existing boolean `verbose` config value is preserved. The only
code that looks at the extended `--verbose` values is the logging configuration. This was done
to minimize the number of changes in the system, specifically in the config unification of
command-line values with config vars.

See [hugo commit 7bbbc64730][] for details. However, the next section details a more
extensive but I think better patch.

[hugo commit 7bbbc64730]: https://github.com/neurocline/hugo/commit/7bbbc647300b60cd24abed71f6383c163eb09e54

## Suggested but more invasive change

A better change might be as follows:

- `--log` is as `--verbose`: it becomes a string parameter with the default value of `WARN`
- `--verboseLog` is deprecated in much the same way as `--debug`

Here's the behavior I'm talking about in full

Console:

- with `--quiet`, log at ERROR level (this overrides everything else)
- with no flags, log at WARN level
- with `--verbose`, log at INFO level
- with `--debug`, log at DEBUG level
- with `--verbose=LEVEL`, log at specified level (this overrides everything except `--quiet`)

File:

- with no flags, there is no file logging
- with `--log` or `--logFile=FILE`, log at WARN level
- with `--verboseLog`, log at INFO level
- with `--verboseLog` and `--debug`, log at DEBUG level
- with `--log=LEVEL`, log at specified level (this overrides everything else)

The reason for using `--log=LEVEL` and not `--verboseLog=LEVEL` is arbitrary. However,
there's a slight edge towards using `--log`, because we wouldn't need `--verboseLog`
at that point. On the other hand, `verboseLog` is a config var and `log`/`logging` is not.

The non-deprecated set of options is this:

- `--quiet`
- `--verbose`
- `--log`
- `--logFile`

Everything can be accomplished with just these options. The others will be maintained for backwards
compatibility but deprecated.

This is implemented in a commit that replaces the one above. See
[hugo commit e0f974139e][] for details.

[hugo commit e0f974139e]: https://github.com/neurocline/hugo/commit/e0f974139e4d01b00e60ff76547fbf24016b4d6e
