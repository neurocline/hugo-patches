---
title: "Patch: Date/time only shown in file logs"
date: 2019-03-07T12:09:15-08:00
---

This patch is separated from the other "reduce output length" patch because this
changes visible behavior. I feel like this is a good change, because console output
is evanescent and people who want to save log output should be using `--logFile` to
write it to a file. However, users are an interesting lot, and don't always behave
like I do.

## Don't show date/time in console log messages

The logger shows date/time messages largely for the use case of file-based
logging; such logs might hang around for days, weeks, or even years. This is
overly verbose when showing messages to the console.

Remove date/time output when sending log messages to the console. Log messages written
to the console now look like this.

```
Building sites …
WARN No layout for type="page" lang="en" output format="HTML"
WARN No layout for type="home" lang="en" output format="HTML"
WARN No layout for type="taxonomyTerm" lang="en" output format="HTML"
WARN No layout for type="taxonomy" lang="en" output format="HTML"
WARN No layout for type="section" lang="en" output format="HTML"
```

There is a new commandline/config var that controls this behavior; `--consoleLogTime` and `consoleLogTime`.
This defaults to `false`; setting it to `true` restores the old behavior.
