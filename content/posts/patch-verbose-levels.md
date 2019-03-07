---
title: "Patch: Verbose Levels"
date: 2019-03-06T23:04:13-08:00
---

## Add log levels to --verbose and --log

Hugo uses jwalterweatherman and has a fair amount of logging controlled by different
levels. However, it's a little awkward to get at it. To that reason, the `--verbose`
and `--log` options are extended to take an optional logging level.

For `--verbose`:

- `--verbose` defaults to INFO
- `--verbose=warn` for WARN (least verbose)
- `--verbose=info` for INFO
- `--verbose=debug` for DEBUG
- `--verbose=trace` for TRACE (most verbose)

For `--log`:

- `--log` defaults to WARN
- `--log=warn` for WARN (least verbose)
- `--log=info` for INFO
- `--log=debug` for DEBUG
- `--log=trace` for TRACE (most verbose)

This also updates quiet: if `--quiet` is used, console logging is set to ERROR level, independent
of any log settings.

The existing boolean `verbose` config value is preserved. The only code that
looks at the extended `--verbose` values is the logging configuration. This
was done to minimize the number of changes in the system, specifically in the
config unification of command-line values with config vars.

The existing vars `--debug` and `--verboseLog` are still allowed, but are
considered deprecated, and overriden by use of `--verbose=LEVEL` or
`--log=LEVEL`.

This is the new behavior in full.

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

See [hugo commit e0f974139e][] for details.

[hugo commit e0f974139e]: https://github.com/neurocline/hugo/commit/e0f974139e4d01b00e60ff76547fbf24016b4d6e
