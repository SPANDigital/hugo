---
title: Link render hooks
linkTitle: Links
description: Create a link render hook to override the rendering of Markdown links to HTML.
categories: []
keywords: []
---

## Markdown

A Markdown link has three components: the link text, the link destination, and optionally the link title.

```text
[Post 1](/posts/post-1 "My first post")
 ------  -------------  -------------
  text    destination       title
```

These components are passed into the render hook [context](g) as shown below.

## Context

Link render hook templates receive the following context:

Destination
: (`string`) The link destination.

Page
: (`page`) A reference to the current page.

PageInner
: {{< new-in 0.125.0 />}}
: (`page`) A reference to a page nested via the [`RenderShortcodes`] method. [See details](#pageinner-details).

PlainText
: (`string`) The link description as plain text.

Text
: (`template.HTML`) The link description.

Title
: (`string`) The link title.

## Examples

> [!note]
> With inline elements such as images and links, remove leading and trailing whitespace using the `{{‑ ‑}}` delimiter notation to prevent whitespace between adjacent inline elements and text.

In its default configuration, Hugo renders Markdown links according to the [CommonMark specification]. To create a render hook that does the same thing:

```go-html-template {file="layouts/_markup/render-link.html" copy=true}
<a href="{{ .Destination | safeURL }}"
  {{- with .Title }} title="{{ . }}"{{ end -}}
>
  {{- with .Text }}{{ . }}{{ end -}}
</a>
{{- /* chomp trailing newline */ -}}
```

To include a `rel` attribute set to `external` for external links:

```go-html-template {file="layouts/_markup/render-link.html" copy=true}
{{- $u := urls.Parse .Destination -}}
<a href="{{ .Destination | safeURL }}"
  {{- with .Title }} title="{{ . }}"{{ end -}}
  {{- if $u.IsAbs }} rel="external"{{ end -}}
>
  {{- with .Text }}{{ . }}{{ end -}}
</a>
{{- /* chomp trailing newline */ -}}
```

## Default

{{< new-in 0.123.0 />}}

Hugo includes an [embedded link render hook] to resolve Markdown link destinations. Disabled by default, you can enable it in your site configuration:

{{< code-toggle file=hugo >}}
[markup.goldmark.renderHooks.link]
enableDefault = true
{{< /code-toggle >}}

A custom render hook, even when provided by a theme or module, will override the embedded render hook regardless of the configuration setting above.

> [!note]
> The embedded link render hook is automatically enabled for multilingual single-host sites if [duplication of shared page resources] is disabled. This is the default configuration for multilingual single-host sites.

The embedded link render hook resolves internal Markdown destinations by looking for a matching page, falling back to a matching [page resource](g), then falling back to a matching [global resource](g). Remote destinations are passed through, and the render hook will not throw an error or warning if unable to resolve a destination.

You must place global resources in the `assets` directory. If you have placed your resources in the `static` directory, and you are unable or unwilling to move them, you must mount the `static` directory to the `assets` directory by including both of these entries in your site configuration:

{{< code-toggle file=hugo >}}
[[module.mounts]]
source = 'assets'
target = 'assets'

[[module.mounts]]
source = 'static'
target = 'assets'
{{< /code-toggle >}}

{{% include "/_common/render-hooks/pageinner.md" %}}

[`RenderShortcodes`]: /methods/page/rendershortcodes
[CommonMark specification]: https://spec.commonmark.org/current/
[duplication of shared page resources]: /configuration/markup/#duplicateresourcefiles
[embedded link render hook]: {{% eturl render-link %}}
