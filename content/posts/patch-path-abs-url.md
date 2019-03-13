---
title: "Patch: Make path-absolute-URL links"
date: 2019-03-08T14:42:17-08:00
---

This patch harmonizes markup generation (which makes site-absolute-URL links)
with Hugo template processing (which makes path-absolute-URL links), removing
a host of bugs and simplifying the code a little. This patch, or something like
it, is absolutely needed to make Hugo link generation work in all cases.

Note that this is only for HTML output, because that's the only path that
markup-generated text can show up in. If that isn't actually the case, this
needs to be updated.

See [hugo commit 98f58ec5a1][] for details.

[hugo commit 98f58ec5a1]: https://github.com/neurocline/hugo/commit/98f58ec5a147679e8b36de7d7d15adac33f022a2

## Postprocess markup-generated HTML to have path-absolute-URL links

WhatWG uses this terminology in its [URL: Living Standard](https://url.spec.whatwg.org/#url-writing):

- **absolute-URL**: scheme + host + path-absolute-URL
- **path-absolute-URL**: starts with / and is followed by a path-relative-URL string.
- **path-relative-URL**: zero or more URL-path-segment strings joined by /, and does not start with /

Some examples will make it a little clearer:

- `http://example.com` is an absolute-URL that's just scheme+host
- `http://example.com/path/to/a.html` is an absolute-URL that points to a file
- `/path/to/a.html` is a path-absolute-URL that assumes a host will be prefixed
- `to/a.html` is a path-relative-URL; in this case, it assumes some file `http://example.com/path/f.html` contains the link

That's all well and good. However, Hugo introduced a new concept. A generated site
is rooted in `baseURL`, which can have a `BasePath` component. E.g.

- `baseURL = http://example.com/subsite`

means that all the links in the site get prefixed with this, either explicitly or
implicitly. Let's introduce a new name for this:

- **site-absolute-URL**: starts with / and is followed by a path-relative-URL string, and sits inside `baseURL.BasePath`

The reason for this distinction is markup generation. Markup renderers like Blackfriday don't
know about Hugo's `baseURL.BasePath`. So, if the user has an image link like:

```
![Pretty picture](/img/cool.png)
```

then the generated HTML will look like this

```
<img src="/img/cool.png" alt="Pretty picture" />
```

Of course, this link won't work with `baseURL=http://example.com/subsite`. There was a hack
to try to make this work in Hugo, but it was incomplete, and caused as many problems as it fixed.

The user could try to use relative links everywhere, but this is both hard to
do (all those `../../../path/to` are hard to reason about), and still breaks
in the presence of pretty URLs (which shift the page relative to where the
user sees it in `content`). That's a separate issue to fix. When writing
content, it's just easier to use site-absolute-URL links in the content; it's
easier to ensure that it's correct just by eyeballing it. More importantly, we should
avoid making the content writer's life harder; we should not require shortcodes simply
to make links to files that the user knows the path to.

So, consider a simplified version of the Hugo render pipeline:

![Render process](/img/2019-03-08-3-render-before.svg)

We have two files being merged together (well, specifically, a layout uses `{{ .Content }}` to insert
the output of markup rendering into the template). And when you look at it like this,
you can see the conflict - if we mix path-absolute-URL and site-absolute-URL links together,
we can't tell them apart with confidence. Trying to guess by "does this begin with the user BasePath?"
is begging for trouble.

It would be better to fix the markup-generated HTML right away so that it has
path-absolute-URL links instead of site-absolute-URL links.

![Render process](/img/2019-03-08-4-render-after.svg)

Now, when a layout uses `{{ .Content }}` to pull in some markup-rendered HTML, we can
be confident that URLs in that are consistent with our `baseURL.BasePath` setting, whatever
that may be. This means that all of the Hugo template functions like `.Permalink`, `absURL`,
and so on are compatible with the markup-rendered HTML.

Specifically, after we call `renderBytes`, we call a new function `urlreplacers.SiteAbsToPathAbs`,
which uses the existing `transformers` code to prepend `baseURL.BasePath` to any site-absolute-URL
links in the text. We can't call this from inside `renderBytes` due to dependencies; `helpers`
is further down on the dependency chain and can't use `transformers`. This is a little
awkward, as we have to duplicate fixup in several places. Until a fairly big reorg of code
occurs, this is necessary.

## Remove canonifyURL hacks

The previous attempt was to rely on use of `canonifyURLs=true` to try to fix things up.
Of course, if you didn't use this var, you had broken output, and if you used this var,
you still had broken output in some cases.

Once markup-generated HTML is consistent with `baseURL.BasePath`, we can remove all the
hacks in the code related to this. Some of the hacks were in tests, and some of the tests
were written simply to make tests pass - there were bugs that were caught by the tests
(pointing out bad URLs) but these were papered over by making the tests work.

### PrependBasePath

This no longer takes or needs an `isAbs bool` parameter, because `GetBasePath` doesn't
need a parameter.

### GetBasePath

This no longer takes or needs an `isRelativeURL bool` parameter, and it simply
returns `BasePath`.

### RelURL

This is rewritten so that it no longer needs to call `AddContextRoot`.

### AddContextRoot

Removed as no longer necessary. Calling sites use `PrependBasePath` instead.

## Open questions

The Hugo source uses "context root" in several places to refer to `baseURL.BasePath`. I
agree that `BasePath` isn't really a term that's reasonable to use, because it's mixed
with `baseURL` itself. I'm just not sure that "context root" is reasonable either. Maybe
`siteHome` or `siteRoot`?

It also uses "BaseURL root" to refer to the scheme+host portion. Again, unified
terminology would be good, and preferrably sticking close to established terms.
There's already a function `HostURL`, so maybe we should just use `baseURL.HostURL` and
`baseURL.BasePath` when talking about these?

The Hugo documentation unfortunately uses the term `Relative` for path-absolute-URLs.
The `RelURL` function creates a path-absolute-URL, not a path-relative-URL. It's good
that it does this, but it's definitely confusing to users. I have another patch with
suggestions along this line.

## Tests

Tests no longer need `canonifyURL` hacks.

A new and fairly simple test of `SiteAbsToPathAbs` was added.

Several existing tests that were broken by the change were updated. In two
cases, the test itself seems to be defective; one test (`TestAbsURLify`) was
marked as skip, and another test (`TestSkipRender`) had one of its test case
commented out. The first test needs to be rewritten completely, as
it was based on a premise that contradicts how Hugo is actually used.

The `TestPageBundlerSiteRegular` and `TestPaginationURLFactory` tests
were fine, they just had their `canonifyURL` hacks removed.

One test revealed flaws in the test thinking. The `TestPermalink`
test appears to have been written to pass, ignoring that the links being
asserted would never work in generated output; those are now fixed to expected
values.

Once these were fixed, `go test ./...` passed all tests.

`go fmt` was run on all changed files.
