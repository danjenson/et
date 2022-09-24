---
layout: default
title: About
---

## Edward Tufte Jekyll Theme

This is an [Edward Tufte](https://en.wikipedia.org/wiki/Edward_Tufte) theme
based on [clayh53/tufte-jekyll](https://github.com/clayh53/tufte-jekyll). It is
updated to be
[jekyll-remote-theme](https://github.com/benbalter/jekyll-remote-theme)
compatible and tested with `ruby 3.1` and `jekyll 3.9`. You can learn how
Jekyll sites work and how to customize them [here](https://jekyllrb.com/docs/).

To use this theme, add the following to your `_config.yml`:

```yaml
remote_theme: danjenson/et
markdown: kramdown
plugins:
  - ...
  - jekyll-remote-theme
footer: # run `grep icon-` in project folder for icon names
  - link: https://twitter.com/edwardtufte
    icon: icon-twitter
  - link: https://github.com/danjenson/et
    icon: icon-github
```

Then, to run locally:

```bash
bundle add jekyll-remote-theme webrick
bundle install
bundle exec jekyll serve
```

You can also use latex with `$$`, e.g. `$$f^2$$` becomes $$f^2$$. To create a
block, create a newline before the opening `$$`. You can see [here](https://docs.mathjax.org/en/latest/input/tex/macros/index.html) the list of available mathjax commands.

Gotchas:

1. Occasionally, rendering will fail with single `$` delimiters, usually when
   there is a command followed by an underscore, i.e.
   `$\mathcal{H}_{edge}:\mathcal{F}_{Markov}(G,R)\to\mathcal{O}_{edge}$` will
   render as
   $\mathcal{H}_{edge}:\mathcal{F}_{Markov}(G,R)\to\mathcal{O}_{edge}$. The
   solution is to escape the `_` for the subscripts. So,
   `$\mathcal{H}\_{edge}:\mathcal{F}\_{Markov}(G,R)\to\mathcal{O}\_{edge}$` is
   correctly rendered as
   $\mathcal{H}\_{edge}:\mathcal{F}\_{Markov}(G,R)\to\mathcal{O}\_{edge}$.

Protips:

1. If you use (n)vim, add the
   [vim-markdown](https://github.com/preservim/vim-markdown) plugin and enable
   math markdown with `let g:vim_markdown_math = 1`; this will prevent all
   sorts of syntax highlighting issues when using `$` and `_`.
1. You can add `toc: true` to the front matter (yaml at the top of READMEs) to
   automatically include a table of contents. See `notes.md` for an example.
1. Copy the `_plugins` directory to your project to get access to the following Liquid tags:

```liquid
{% raw %}
# Capitalizes words
{% newthought "Example website: <a href='http://example.com'>example label</a>" %}
# Short quotation
{% epigraph ' "How did you go bankrupt?" Two ways. Gradually, then suddenly.' 'Ernest Hemingway' ' "The Sun Also Rises" ' %}
# Indexed note that appears in margin
{% sidenote "sidenote-id" "This is a random sidenote" %}
# Non-indexed note that appears in margin
{% marginnote "margin-note-id" "Random thought when drinking" %}
# Full-width image
{% fullwidth "assets/img/full-width.png" "A full-width image." %}
# Main-column image
{% maincolumn "http://example.com/image.jpg" "Main column image." %}
# Margin image
{% marginfigure "margin-figure-id" "assets/img/side.png" "This is the caption" %}
{% endraw %}
```
