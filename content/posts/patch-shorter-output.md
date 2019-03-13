---
title: "Patch: Cleaner Output"
date: 2019-03-07T11:48:44-08:00
---

This patch streamlines some of the verbose Hugo console output.

The default Hugo output is overly verbose for certain kinds of messages and errors.
Fixing this is complicated by the fact that a single log print message can go
to both console and logfile and be filtered at different levels for console and logfile.

The approach that is taken is as follows:

- slim messages are output at WARN or INFO levels
- extra details are added at DEBUG or TRACE levels

This allows, for example, INFO level output to console and augmented debug info
written to a logfile.

It is possible for the paired WARN/INFO and DEBUG/TRACE messages to be intermingled
with output from other goroutines. The current logging system does not show any goroutine
identifier, nor does it keep log messages together in sections. That's a bigger defect
that will be addressed at some point. For example, maybe each operation shows an operation
ID in the log message. For this reason, the DEBUG/TRACE messages repeat information from
the WARN/INFO message. A future change to the logging system could remove the need for this.

See [hugo commit 3af4e13c05][] for the full diff.

[hugo commit 3af4e13c05]: https://github.com/neurocline/hugo/commit/3af4e13c0575c66ac1eeafa7a1b856ed2a834f70

## reduce verbosity for missing layouts

If a layout is not found, the current message is very verbose, and repetitive. Change the code
so that the current message is only seen in DEBUG or higher; if the log level is less than
DEBUG, show a shorter message at WARN level:

```
Building sites …
WARN 2019/02/19 15:36:46 No layout for type="page" lang="en" output format="HTML"
WARN 2019/02/19 15:36:46 No layout for type="home" lang="en" output format="HTML"
WARN 2019/02/19 15:36:46 No layout for type="taxonomyTerm" lang="en" output format="HTML"
WARN 2019/02/19 15:36:46 No layout for type="taxonomy" lang="en" output format="HTML"
WARN 2019/02/19 15:36:46 No layout for type="section" lang="en" output format="HTML"
```

The list of layout locations is dropped, and each missing layout is shown once per run, not once
per page render attempt.

If we have `--verbose=debug` or `--verbose=trace`, then the extra information is shown
as an extra log message. There will be DEBUG level messages without a paired WARN/INFO message
because we only show one WARN message for each distinct output.

```
WARN 2019/03/07 13:00:50 No layout for type="page" lang="en" output format="HTML"
DEBUG 2019/03/07 13:00:50 No layout for type="page" lang="en" output format="HTML": create a template below /layouts with one of these filenames: posts/single.en.html.html, posts/single.html.html, posts/single.en.html, posts/single.html, _default/single.en.html.html, _default/single.html.html, _default/single.en.html, _default/single.html
```

### reduce verbosity for page render

The DEBUG message for page render is useful at lower levels of logging. Split the message
into two parts - a slim message at INFO level, and the extra information in a second
message at DEBUG level.

At INFO level, it looks like this:

```
INFO 2019/03/07 13:09:37 Render type="page" lang="en" format="HTML" to "/posts/patch-console-no-time/index.html"
INFO 2019/03/07 13:09:37 Render type="page" lang="en" format="HTML" to "/posts/patch-shorter-output/index.html"
```

At DEBUG or higher, it looks like this:

```
INFO 2019/03/07 13:09:37 Render type="page" lang="en" format="HTML" to "/posts/patch-console-no-time/index.html"
DEBUG 2019/03/07 13:09:37 Render type="page" lang="en" format="HTML" to "/posts/patch-console-no-time/index.html" (with layouts ["posts/single.en.html.html" "posts/single.html.html" "posts/single.en.html" "posts/single.html" "_default/single.en.html.html" "_default/single.html.html" "_default/single.en.html" "_default/single.html"])
INFO 2019/03/07 13:09:37 Render type="page" lang="en" format="HTML" to "/posts/patch-shorter-output/index.html"
DEBUG 2019/03/07 13:09:37 Render type="page" lang="en" format="HTML" to "/posts/patch-shorter-output/index.html" (with layouts ["posts/single.en.html.html" "posts/single.html.html" "posts/single.en.html" "posts/single.html" "_default/single.en.html.html" "_default/single.html.html" "_default/single.en.html" "_default/single.html"])
```

The DEBUG message repeats the trimmed INFO message because the two log messages can be interleaved
with output from other goroutines. In fact, it's almost guaranteed to have this happen. Unfortunately,
with jwalterweatherman, there's no way to avoid the duplication of output, due to the teeing to file
and log with different thresholds applied.

And in this particular case, the layouts output is not as useful as it could be, since it is showing
the full set of theoretically possible layouts, and not calling out the layouts that actually exist.
I have a `hugo check` command in mind that would help to that end.

## tests

All tests were run with `go test ./...` from the root of the Hugo directory.

The `gofmt` tool was used on all modified files.
