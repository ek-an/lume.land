---
title: Multilanguage
description: Create multiple language versions of the same page
mod: plugins/multilanguage.ts
tags:
  - nav
---

## Description

This plugin provides some features to make easier the creation of multilanguage
sites.

## Installation

Import this plugin in your `_config.ts` file to use it:

```js
import lume from "lume/mod.ts";
import multilanguage from "lume/plugins/multilanguage.ts";

const site = lume();

site.use(multilanguage({
  languages: ["en", "gl", "es"], // Available languages
}));

export default site;
```

Due to the
[automatic alternate links generating](#automatic-rel%3Dalternate-links), the
plugin must be registered after all URL modifiers to work probably.

```js
import lume from "lume/mod.ts";
import basePath from "lume/plugins/base_path.ts";
import multilanguage from "lume/plugins/multilanguage.ts";

site.use(basePath()); // modify url
site.use(multilanguage({
  languages: ["en", "gl", "es"],
}));
// site.use(basePath()); // this modification will not be applied

export default site;
```

## Create pages in multiple languages

This plugin uses the variable `id` (combined with `type` if it's defined) to
detect the different translations of the same page. For example, let's say we
have the following two pages:

```yml
---
lang: en
id: about
url: /about-me/
---

# About me
```

```yml
---
lang: gl
id: about
url: /acerca-de-min/
---

# Acerca de min
```

The plugin interprets these two pages as the same content but in different
languages, because they have the same id (`about`) and the same `type`
(undefined).

The pages will be exported as `/en/about-me/`, `/gl/acerca-de-min/`. Note that
the `/en/` and `/gl/` prefixes are automatically added.

It's possible to configure one language as default, so any page in this language
won't have the prefix in the URL. For example, consider the following
configuration:

```js
site.use(multilanguage({
  languages: ["en", "gl", "es"], // Available languages
  defaultLanguage: "en", // The default language
}));
```

Now, the pages will be exported as `/about-me/` and `/gl/acerca-de-min/`. The
english version doesn't have the `/en/` prefix because it's the default
language.

## Create links to translations

### Automatic rel=alternate links

Because this plugin detects the different language version for every page, it
inserts automatically the
`<link rel="alternate" hreflang="{lang}" href="{url}" />` elements in each
pages. For example:

```html
<!doctype html>
<html lang="en">
  <head>
    <title>About me</title>

    <link rel="alternate" hreflang="gl" href="/gl/acerca-de-min/" />
  </head>
  <body>
    ...
  </body>
</html>
```

The attribute `lang` is inserted automatically in the `html` element if it's
missing.{.tip}

### Create a language switcher menu

The plugin also exposes the `alternates` variable, that you can use to build a
menu to change the language of the current page. This is an example in nunjucks:

<lume-code>

```vento {title=_includes/layout.vto}
<ul class="languages">
{{ for alt of alternates }}
  <li>
    <a href="{{ alt.url }}" {{ if alt.lang == lang }}aria-current="page"{{ /if }}>
      {{ alt.title }} ({{ alt.lang }})
    </a>
  </li>
{{ /for }}
</ul>
```

</lume-code>

This code outputs something like:

```html
<ul class="languages">
  <li>
    <a href="/about-me/" aria-current="page">
      About me (en)
    </a>
  </li>
  <li>
    <a href="/sobre-min/">
      Sobre min (gl)
    </a>
  </li>
  <li>
    <a href="/acerca-de-mi/">
      Acerca de mí (es)
    </a>
  </li>
</ul>
```

### Sitemap

The [Sitemap plugin](./sitemap.md) is compatible with this plugin, so the
generated sitemap will contain the translated versions of the pages.

## Multilanguage data

With this plugin, you can have your data in different languages that will be
correctly resolved according to the language of each page. For example, let's
say we have the following `_data.yml` file for our site with the languages `en`,
`gl` and `es`:

```yml
site_name: The Óscar's blog
gl:
  site_name: O blog de Óscar
es:
  site_name: El blog de Óscar
```

The variable `site_name` has a different value for the languages `gl` and `es`.
Any variable stored inside a variable named like one of the available languages,
is considered a translation. We can use the `site_name` variable in our pages
and the value will be different depending on the page's language:

<lume-code>

```vento{title="english"}
---
lang: en
---

<h1>{{ site_name }}</h1> <!-- Outputs: The Óscar blog -->
```

```vento{title="galician"}
---
lang: gl
---

<h1>{{ site_name }}</h1> <!-- Outputs: O blog de Óscar -->
```

```vento{title="spanish"}
---
lang: es
---

<h1>{{ site_name }}</h1> <!-- Outputs: El blog de Óscar -->
```

</lume-code>

It's also possible to use objects to replace specific inner values. For example:

```yml
site:
  name: The Óscar's blog
  logo: /logo.png

# Translate only site.name, but leave site.logo as is
gl:
  site:
    name: O blog de Óscar

es:
  site:
    name: El blog de Óscar
```

## Multilanguage pages from a single file

With this plugin it's possible to export the same page multiple times, once per
language. To configure a page as multilanguage, just set in the `lang` variable
to an array with the available languages. For example:

<lume-code>

```yml {title=about-me.yml}
# This page is in 3 different languages: English, Galician, and Spanish.
lang: [en, gl, es]
title: About me
layout: base-layout.vto
```

</lume-code>

Lume will generate three pages, one per language, prefixing the output path of
each page with the language code:

```txt
/en/about-me/index.html
/gl/about-me/index.html
/es/about-me/index.html
```

We can assign different data per language:

<lume-code>

```yml {title=about-me.yml}
lang: [en, gl, es]

# Common values for all languages
layout: base-layout.vto
title: About me
url: /about-me/

# Galician translations
gl:
  title: Acerca de min
  url: /sobre-min/

# Spanish translations
es:
  title: Acerca de mí
  url: /acerca-de-mi/
```

</lume-code>

## Multilanguage + pagination

If your site is using the [paginate plugin](./paginate.md), this is an example
showing how to apply multilanguage:

<lume-code>

```js {title=paginate.page.js}
export const layout = "layouts/my-layout.vto";

export default function* ({ search, paginate }) {
  const langs = ["gl", "en"];

  for (const lang of langs) {
    const pages = search.pages(`type=article lang=${lang}`);

    yield* paginate(pages, {
      url: (n) => `/${lang}/articles/${n}/`,
      each(page, number) {
        page.lang = lang;
        page.id = `articles-page-${number}`;
      },
    });
  }
}
```

</lume-code>

- In this example, the generator is generating all pages in all languages.
- In the `each` option, we are adding the `lang` and `id` values to the new
  pages.
- The `id` ensures that the page `/gl/articles/1/` and `/en/articles/1/` are
  treated as translations of the same content, because both have the same `id`.
