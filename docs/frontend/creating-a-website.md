---
title: "Creating A Website"
summary: "My thoughts about creating my own website"
date: 2022-03-26
---

As if "modern" web development wasn't already hard, we'll need to learn and use a templating engine
coupled with a static site generator to create a functional blog that we can truly call our own.
This isn't strictly necessary but it isn't really feasible or desirable for me to write HTML to
produce ... HTML. Although there are awesome tools like [pandoc][1] and [lowdown][2] which can
convert markdown to HTML, we need a method to replicate information across web pages, something
which templating engines do. However, if you want a preview of how simple a static site generator
can be, check out [Simple Blogging System][3].

Unfortunately, I haven't been able to find a definitive resource on the Internet for learning how to
write HTML. Fortunately, I was able to find websites like [Seirdy's Home Page][4], [Oriole Systems'
Blog][5], and the [Parity Bit Blog][6], all of which have a decent web page with quite informative
HTML.

If you want a recap of the basics of HTML, the [HTML Basics][7] page on MDN is a decent place to
start but for those who do have a basic idea of HTML works, this block of code should be a pretty
decent starting point for writing a web page.

``` html
<!DOCTYPE html>
<html lang="en">

  <head>
    <meta charset="utf-8">
    <meta name="color-scheme" content="light dark">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="The personal microblog of Ayush Agarwal">
    <title>@ayushnix</title>
  </head>

  <body>
  </body>

</html>
```

The `meta` tag containing `charset="utf-8"` should be the immediate child of the `head` tag. The
`meta` tag with the `viewport` makes our page responsive for mobile phones. The `color-scheme`
`meta` tag is interesting because it can [apply dark and light mode][8] colors on your web page
without using any CSS provided by your website. The colors are derived from your operating system
and your web browser. Since this is part of the HTML itself, it also loads before any CSS file is
downloaded and applied. The `description` `meta` tag is meant for the description of a website shown
by search engines.

If you're still wondering why a static site generator might be needed, the `title` tag will need to
generated dynamically for each different page. The `time` tag can be updated dynamically with each
passing year. Even if the tags don't change across web pages, like the `viewport` `meta` tag,
they'll still need to copied on all the pages that you write. It doesn't make sense to copy these
tags on multiple web pages or update tags based on context manually.

# Semantic HTML

As you may have guessed, the `head` tag is responsible for hosting the metadata for our website,
which typically involves using the `meta`, `link`, and the `title` tag. The `body` tag is where the
actual web page content begins. This is where it's imperative to realize that content comes before
presentation. If you're creating a personal website, it should work fine in console web browsers
like [w3m][9] and [lynx][10].

As I'm writing this post, I'm trying to create a personal microblog website. I was inspired to do it
after I came across [zunzuncito][11], the personal microblog of [Wolfgang Müller][12]. However, I
didn't want to just blindly copy and paste his code and I want to understand how to create basic
websites using HTML and CSS.

Let's start by writing a text file which would represent the basic structure of a microblog website.
Again, we're writing content without any sort of representation which excludes hyperlinks, images,
colors, and the placement of content itself.

```
Ayush Agarwal
@ayushnix
The personal microblog of Ayush Agarwal
about
atom feed
json feed
tags
github

2022-03-28
the title of the post
tag 1
tag 2
Lorem ipsum dolor sit amet, consectetur adipiscing elit,
sed do eiusmod tempor incididunt ut labore et dolore
magna aliqua. Lacus sed viverra tellus in hac
habitasse platea dictumst. Ultricies leo
integer malesuada nunc vel risus commodo.
share
discuss on github

copyright &copy; 2022 Ayush Agarwal
content CC-BY-SA 4.0
source code ISC
```

At this point, we're in the messy territory of HTML 5, its "semantic" tags, and its differences from
HTML 4. In HTML 4, the outline, or table of contents, of a web page was usually determined by using
the `h*` tags. To avoid messing things up, it was advised that only one `h1` tag may be present
inside a web page and the order of appearance and nesting of the remaining `h*` tags matter.
Basically, `h3` shouldn't come before `h2`. In addition, it was also advised against skipping
heading levels. `h1` shouldn't be followed by a `h3`. This was a simple, albeit rigid, structure
which makes sense.

HTML 5 introduced the tags `section`, `aside`, `nav`, and `article` which can used to create a
document outline instead of the `h*` tags. Well, these HTML 5 tags still need the `h*` tags as their
descendants for their titles but now, you could mess around with the order and choice of the `h*`
tags if you wanted to without messing up the document outline. Oh, and these tags are supposed to be
good for a11y as well because they have implicitly defined ARIA roles. However, the reality seems to
be a bit different and it looks like most browser user-agents and screen readers haven't implemented
the HTML 5 outlining algorithm as mentioned on [this][13] blog post. It's your choice about how you
want to proceed here but I'll go ahead and use these semantic HTML 5 tags without trying to violate
the `h*` outline structure algorithm used in HTML 4. If nothing else, writing `header` beats writing
`div class=header`. The [cutecodedown][14] website makes a compelling case against HTML 5 and
advocates for sticking to HTML 4.

``` html
<body>
  <header>
    <ul>
      <li><h1>Ayush Agarwal</h1></li>
      <li><a href="#">@ayushnix</a></li>
      <li>The personal microblog of Ayush Agarwal</li>
    </ul>
  </header>

  <nav>
    <ul>
      <li><a href="#">about</a></li>
      <li><a href="#">atom feed</a></li>
      <li><a href="#">tags</a></li>
    </ul>
  </nav>

  <main>
    <article>
      <header>
        <time datetime="2022-03-07T14:54:39.929">2022-03-27</time>
        <h2>the title of the post</h2>
        <ul>
          <li><a href="#">tag 1</a></li>
        </ul>
      </header>
      Laoreet sit amet cursus sit. Sociis natoque penatibus et
      magnis dis parturient montes nascetur. Neque volutpat ac
      habitasse platea dictumst vestibulum rhoncus est.
      <footer>
        <ul>
          <li><a href="#">share</a></li>
          <li><a href="#">discuss on github</a></li>
        </ul>
      </footer>
    </article>
  </main>

  <footer>
    <ul>
      <li>copyright &copy; 2022 Ayush Agarwal</li>
      <li>license <a href="#">ISC</a></li>
    </ul>
  </footer>
</body>
```

If you're restricting yourself to HTML 4, it's easy to determine the outline of your web page
because only the `h*` tags can create a new section. However, when using HTML 5, in addition to the
`h*` tags,

- `article`
- `section`
- `aside`
- `nav`

can create new sections in a web page. The [Nu HTML Checker][15] can help you visualize the outline
of your HTML document from the perspective of both HTML 4 and HTML 5. If you can't figure out which
tag to use to wrap some content on your web page, you can always fall back to using `div` and
`span`.

At this point, your web page is ready to be consumed on text based browsers like w3m. In fact, the
output in w3m and a web browser should be mostly identical, which is a good thing.

# CSS

Unlike HTML, I was able to find a lot of resources to explain how CSS works and the different kinds
of layouts one can build for websites. There's no point in me going over CSS in detail because I'm
not a web designer or developer, nor do I intend to be. The [Learn CSS][16] module can be a good
place to start, although it does seem to make assumptions about the reader being familiar with CSS.
The [MDN docs about CSS][17] are more detailed and will probably serve as a better introduction.

## Cascade, Specificity, Inheritance

The cascading property simply means that the order of the rules matter and if two rules with equal
specificity are mentioned, the one mentioned later is applied. Specificity?

``` css
h1 {
  color: blue;
}
.first-heading {
  color: red;
}
```

If an `h1` element with a class of `first-heading` uses the CSS code mentioned above, the color of
the heading will be red rather than blue. The order of specificity, from the least to the most, is

- element and pseudo-element selector
- class, pseudo-class, or attribute selector
- ID selector
- inline style

It's important to remembet that `*`, `+`, `>` and `~` don't have any effect over specificity. And
no, don't use `!important` in your code, unless you're overriding CSS for another website for use
with the [Stylus][18] web browser add-on.

Some CSS properties, like [`font-size`][19], might get inherited from the parent element and this can be
verified using the formal definition sections on MDN.

## CSS Selectors

The [CSS Diner][20] game should help you get familiar with CSS selectors.

## Box Model

The MDN page on the box model, which in turn links to CSS Tricks, recommends using the following CSS
reset to change the box model

``` css
html {
  box-sizing: border-box;
}
*,
*::before,
*::after {
  box-sizing: inherit;
}
```

Since the `html` element has a higher specificity than the universal selector `*` (which doesn't
have any specificity), `html` will have its `box-sizing` set to `border-box`. All the other elements
on the page will then inherit `box-sizing` from their parents, which is ultimately `html`. This
makes it easier to change the box model for an element and its children because the box model of
the element will be inherited by its children. If we had used

``` css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

instead, even if we change the box model of an element, its children won't inherit it.

To avoid confusion about the size of the boxes and not get confused with the default margins that
web browsers use, it would be a good idea to add

``` css
* {
  margin: 0;
}
```

at the very least. `padding: 0;` can also be added at the cost of losing list style.

Besides the obviously important stuff about padding, borders, and margin, and what `border-box` and
`content-box` means, we'll need to understand the *inner* and *outer* display types of boxes.

## Layout

This, in my opinion, is the most important part of learning how to write CSS and not getting
frustrated and overwhelmed in the process. I don't know about others but not seeing my website laid
out properly made me mad. This was the first thing I wanted to fix.

By default, elements like `h1` and `p` have the outer display type of `block` and `em` and `strong`
have the outer display type of `inline`. `block` elements will occupy the entire horizontal space
they're in and will create a new line. They'll obey `width` and `height` and faithfully reproduce
all the features of `padding`, `border`, and `margin`, both in the horizontal and vertical
direction. In contrast, `inline` elements

- don't create a newline
- ignore `width` and `height`
- vertical `padding`, `border`, and `margin` will apply but get overlapped and messed up

This behavior — layout of text (and optionally, elements) inside a `block` or an `inline` box — is
also called *normal flow* and this is the *inner* display type that we're talking about. In fact, if
we consider the [multi-keyword version of `display`][21], it's mentioned explicitly.

| single keyword | multi keyword      |
|----------------|--------------------|
| `block`        | `block flow`       |
| `inline`       | `inline flow`      |
| `flex`         | `block flex`       |
| `grid`         | `block grid`       |
| `inline-flex`  | `inline flex`      |
| `inline-grid`  | `inline grid`      |
| `inline-block` | `inline flow-root` |
| `flow-root`    | `block flow-root`  |

In the 2nd colum of this table, the first value is the outer display type and the second is the
inner display type. Unfortunately, the multi-keyword version of `display` [doesn't work on
Chromium][22] based web browsers as on April 2022.

At this point, I don't claim to understand what formatting contexts in CSS are and what exactly does
`flow` mean when specified in `display` as an inner display type value. For now, I'll assume that
`flow` means whatever the browser default is and the child elements of a `display: block flow`
element can be either `block` or `inline` and `flow` is just an explicit expression of the default
web browser behavior.

I'm not familiar with how messy things were before `flex` and `grid` came into the picture but I can
imagine it was a mess considering [memes to center a div][23] exist. Fortunately, making out HTML
code look good isn't difficult thanks to modern, and more importantly native, CSS features like
flexbox and grid.

### Flexbox

A flexbox is useful when the layout configuration is needed in a single dimension, either on a row
or on a column. You won't be able to meaningfully manipulate flex items on both dimensions at the
same time and if you want to do that, you should use grid.

The [MDN page on flexbox][24] is a pretty decent introduction on how flexbox works. The [web.dev
page on flexbox][25] is a pretty good as well. The [flexboxfroggy game][26] should help as well but
keep in mind the caveats mentioned in the web.dev page on flexbox — avoid using flexbox features
such as `order`, `row-reverse`, and `column-reverse`. The [CSS Tricks page on flexbox][27] has a
pretty good presentation on flexbox features as well.

We can consider the entire `body` as a one-dimensional flexbox with the `header`, `nav`, `main`, and
`footer` child elements as flex items rather than normal flow items.

``` css
body {
  display: flex;
  flex-direction: column;
}
```

Since the height is automatically figured out by the browser as the content size increases,
`justify-content` along the main axis, which in this case is vertical, doesn't really make much
sense.

For now, this is what we should focus on, the layout for each HTML element on the page and how it
interacts with other elements around it in the context of its layout. This brings us to the `nav`
element which is the direct descandant of the `body` element. Even though `nav` itself is a flex
item, the `ul` and the `li` items within it aren't. They are block level elements. We'll need to
change `ul` to become another flex container which will make its `li` elements as flex items.

Similarly, at least in this HTML document, the `header` inside `article` can also be expressed a
flex container. The outer display type of the `h2` and `ul` elements inside the `header` need to be
`inline` rather than block. In addition, `ul` inside the `footer` elements, which are children of
`article`, can also be flex containers. The same goes for the `ul` inside the primary `footer`
element, the direct child of `body`.

This is what we end up with

``` css
:is(nav, footer) > ul,
article > header {
  display: flex;
  flex-flow: row wrap;
}

article > header > h2 {
  display: inline-block;
}

article > header > ul {
  display: inline-flex;
}
```

A lot of sites recommend adding `min-height: 100vh` to `body` but that just makes things weird when
the content doesn't need scrolling and elements get spaced out unnaturally when flexbox in the
column direction is in use. This recommendation might make sense in cases where we need a sticky
element somewhere on the page.

There's an interesting choice we have when it comes to aligning the flex child items horizontally.
We can use `align-items: center` in `body`. This will do what we want but the child items, like
`nav`, won't have any margins. Alternatively, we can add

``` css
body > * {
  margin-left: auto;
  margin-right: auto;
}
```

This effectively does the same thing as `align-items: center` but it also adds margins in the
horizontal direction for the flex items. This is explained in [this article on CSS Tricks][28]. We
can couple this in addition to

``` css
margin: 1.618rem auto;
width: min(90%, 60ch);
```

inside the `body` and get a layout that's responsive for both mobile and the desktop.

We still need margins (or maybe padding?) though. For example, the `article` elements aren't
separated by any margins right now, assuming the margin for all elements was reset to `0` in the CSS
reset code mentioned in the [box model section][29] written above. Considering what we already have,
I came up with this

``` css
:is(body, main) > * + * {
  margin-top: 1.618rem;
}
```

This starts from the `nav` (yes, it skips `header`), goes on to `main` and `footer` and adds a
margin on the top. It also ventures into `main` and starts from the 2nd `article` until the last one
and adds margins on the top. Neat huh?!

I don't need to use **grid** for this microblog website so I won't talk about it in this post.
However, I do plan on creating my own blog and that'll probably require using `display: grid`
because it'll have an `aside` HTML element with markdown footnotes. I was inspired to do this after
looking at [Chris Morgan's website][30].

At this point, I realize that some of the typography numbers that I've used (the golden ratio) may
not have been the right choice. Nevertheless, the principles highlighted here do apply. Besides all
the resources that I've already mentioned, here are some other resources which might help

- [Andy Bell's Website][31]
- [Amelia Wattenberger's Blog Post on % in CSS][32]
- [Josh Comeau's Blog Posts][33]
- [Smashing Magazine's articles on CSS][34]
- [Things I Wish I'd Known About CSS][35]
- [Minimal CSS Drop In Files][36] and [CSS "Frameworks"][37]
- the [CSS for GOV.UK website][38]
- [Ahmed Shadeed's Blog Posts][39]
- [SmolCSS][40] and [ModernCSS][41] by [Stephanie Eckles][42]
- [A List Apart Blog][43]
- the [Modular Scale][44] for responsive typography templates
- [1 line layouts][45] and [explanations on web.dev][46]
- [HTML5 Boilerplate][47] and [another one][48]
- [Responsive Breakpoints][49]

# Static Site Generator

Although [Hugo][50] is extremely popular, I [despise its documentation][51]. I was never able to
understand how to modify things or create my own theme with Hugo because of this. I'll go ahead and
try to use [Zola][52] and turn this little microblog project into a proper theme for wider
consumption.

As we said earlier, we need a SSG if we want to build a website which can be dynamically generated.
This is done using template engines. Zola uses [Tera][53], a template engine inspired by
[Jinja][54]. Jinja seems to be pretty popular and it's also used by Ansible, which makes learning a
templating engine worthwhile. Yes, this essentially makes using templating engines to write HTML the
same thing as creating "declarative" HTML pages.

Let's go back to our HTML code. How do we ensure that it can be used by anyone and the page can be
dynamically generated based on user customization? When deciding how to write the `index.html` or
`base.html`, think of the HTML elements which don't need user configuration and customization and
which will be present on all possible pages of the website. A lot of `meta` tags would qualify this
criteria. Such data can be wrapped inside re-usable functions, called **macros** in Jinja and Tera.

``` jinja
{% macro opengraph_basic_meta() %}
    <meta property="og:title" content="{{ self::title() }}">
    <meta property="og:type" content="website">
    <meta property="og:url" content="{{ current_url }}">
    {%- if config.extra.ogimage is defined %}
    <meta property="og:image" content="{{ get_url(path=config.extra.ogimage) }}">
    {% endif -%}
{% endmacro %}
```

This macro can then be used inside `index.html` by first importing the macro, `{% import
"macros/head.html" as head -%}`, and then using `{{ head::opengraph_basic_meta() }}` to embed the
same lines written before at the time when the HTML page will be processed by Zola.

In addition, there should also be a general idea of the different number of pages the website in
question will have. In this case,

- the home page
- individual pages for each post
- list of tags and their counts and a total count of the number of posts and tags as well
- individual pages for each tag containing related posts
- a 404 page

This knowledge is needed to write the templates that we need to generate the website.

Finally, here's the skeleton HTML page that should be common for all the pages mentioned before.

``` html
<!DOCTYPE html>
<html lang="{{ lang }}" prefix="og: https://ogp.me/ns/website#">
  <head>
  </head>
  <body>
    <header>
      <ul role="list">
      </ul>
    </header>
    <nav>
      <ul role="list">
      </ul>
    </nav>
    <main>
    </main>
    <footer>
      <ul role="list">
      </ul>
    </footer>
  </body>
</html>
```

I guess the `header`, `nav`, and `footer` could be optional but I'm not concerned about
incorporating such complexity into this project right now. Even in the 404 page, the `main` tag will
probably have something like `¯\_(ツ)_/¯` but the `header`, `nav`, and `footer` would still be there
for the website that I have in mind.

## Taxonomies

This may not be readily apparent from the document for taxonomies but the fields of `TaxonomyTerm`
should be used by prefixing `term.`. For example, `term.pages` and `term.name`. This is unrelated to
the list of taxonomies you've defined in your `config.toml` file. It becomes active (gets active in
the context you're working on) when you actually click on a tag and view the list of posts with that
tag, the format for which is defined by `tags/single.html`.

On the other hand, the variables in `TaxonomyConfig` have to be referenced by using the prefix
`config.taxonomies`. Assuming that this is the taxonomies configuration defined in your
`config.toml`,

``` toml
taxonomies = [
  { name = "tags", feed = true }
]
```

you'll reference `name` and `feed` by using

``` jinja
{% if config.taxonomies %}
  {% for tax in config.taxonomies %}
    <p>taxonomy name: {{ tax.name }}</p>
    <p>generate feed: {{ tax.feed }}</p>
  {% endfor %}
{% endif %}
```

In addition, if you want to traverse the entire taxonomy structure wherever (in any context and in
any template) you want, you'll find this snippet interesting

``` jinja
{% if config.taxonomies %}
  {% for tax in config.taxonomies %}
    {% set elements = get_taxonomy(kind=tax.name) %}
    <p>taxonomy name: {{ elements.kind.name }}</p>
    <p>total number of {{ elements.kind.name }}: {{ elements.items | length }}</p>
    {% for tag in elements.items %}
    {% set url = get_taxonomy_url(kind=tax.name, name=tag.name) %}
    <p>the url for {{ tag.name }} is {{ url }}</p>
    <p>{{ tag.name }} - has {{ tag.pages | length }} pages</p>
    {% endfor %}
  {% endfor %}
{% endif %}
```

The `elements` var will get access to two elements — `kind` and `items`. As mentioned in the zola
documentation, `kind` gives you access to `TaxonomyConfig` which means the variable `taxonomies` in
the `config.toml` file and its values. This is functionally equivalent to the `tax` variable. The
`items` variable gets access to the entire list of tags you've defined inside `tax.name`. You can
iterate over these tags as done in `for tag in elements.items` and get access to `TaxonomyTerm`
values, which can also be called using `term.pages`, `term.name`, and `term.path` in a different and
restricted context when `term` is already defined by zola, which is when browsing the list of pages
tagged using a specific tag.

# Favicons and Web Manifest

This section maybe unexpected in this post but it turns out that, just like the rest of the modern
web, the favicon situation is messed up. The blog post titled [The Definitive Edition of "How To
Favicon in 2021"][55] offers a lot of insight and justification about the choice that is
recommended.

``` html
<link rel="icon" href="/favicon.ico" sizes="any">
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
<link rel="apple-touch-icon" href="/apple-touch-icon.png"/>
<link rel="manifest" href="/manifest.webmanifest" />
```

This advice is mostly fine but I decided to replace the ICO favicon with a PNG favicon because it is
supported by both Safari and IE. The ICO favicon is needed only if you intend to provide support for
web browsers and operating systems from the dark ages which shouldn't exist on this planet anymore.

I decided to use

``` html
<link rel="icon" href="/favicon.png" type="image/png">
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
<link rel="apple-touch-icon" href="/apple-touch-icon.png"/>
<link rel="manifest" href="/manifest.webmanifest" />
```

What? Did you think that was it? If you want people using Android to have a good experience, you
have to install a [web app manifest][56] as well with yet another set of icons. Nice huh? At the
very least, you'll have to prepare two PNG images with the size of 192px and 512px respectively and
an SVG icon. This shouldn't be a problem if you have a SVG icon to begin with. Oh, and if you don't
want your favicon to look like shit on Android in certain cases, you'll need a ["maskable" icon][57]
which should either be a 196px PNG image or a SVG image. If you wanna go all the way, you can add a
[monochrome icon][58] for notifications on Android as well.

Confused? Here are the list of icons you want

- a "master" SVG icon to begin with
- a 32px PNG favicon.png icon
- a 192px PNG or 180px PNG image for Apple devices
- a 192px PNG image for the Android web manifest
- a 512px PNG image for the Android web manifest
- the same "master" SVG icon for the Android web manifest
- a "maskable" version of the "master" SVG image
- a "monochrome" version of the "master" SVG image

# Comments

I've thought about including comments on this microblog website but, ultimately, I decided against
it because it adds additional complexity and imposes the burden of combating spam and performing
moderation.

The solution? I initially thought about adding a link in the footer of my post which says "discuss
on github" and it would use the GitHub URL where the microblog is hosted and start a GitHub
discussion on it using a URL of this format

```
https://github.com/ayushnix/microblog/discussions/new?category=general&title=The+Post+Title
```

I recently noticed that [Solène Rapenne][59] had embedded links on her blog posts that read
"comments on fediverse/mastodon". These links pointed to a Mastodon bot account which posted links
to her blog post and allowed people to comment on Mastodon instead.

The Mastodon comment system sounds better because it isn't tied to your GitHub account which might
be professional in nature and you may not want comments from random internet strangers on it. I'll
try to build a small program which will use the Mastodon API and post the complete text of my
microblog posts, the tags that I use in the post, and the direct link to the post itself.

[1]: https://pandoc.org/
[2]: https://kristaps.bsd.lv/lowdown/
[3]: https://git.sr.ht/~jbauer/sbs/
[4]: https://seirdy.one/
[5]: https://oriole.systems
[6]: https://www.paritybit.ca/
[7]: https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/HTML_basics/
[8]: https://web.dev/color-scheme/
[9]: https://salsa.debian.org/debian/w3m
[10]: https://lynx.invisible-island.net/
[11]: https://zunzuncito.oriole.systems/
[12]: https://oriole.systems/
[13]: https://html5doctor.com/computer-says-no-to-html5-document-outline/
[14]: https://cutcodedown.com/article/progressive_enhancement
[15]: https://validator.w3.org/nu/
[16]: https://web.dev/learn/css/
[17]: https://developer.mozilla.org/en-US/docs/Learn/CSS
[18]: https://github.com/openstyles/stylus/
[19]: https://developer.mozilla.org/en-US/docs/Web/CSS/font-size#formal_definition
[20]: http://flukeout.github.io/
[21]: https://developer.mozilla.org/en-US/docs/Web/CSS/display
[22]: https://caniuse.com/mdn-css_properties_display_multi-keyword_values
[23]: https://old.reddit.com/r/webdev/comments/pqbxst/why_is_there_so_many_memes_about_how_hard/
[24]: https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox
[25]: https://web.dev/learn/css/flexbox/
[26]: https://flexboxfroggy.com/
[27]: https://css-tricks.com/snippets/css/a-guide-to-flexbox/
[28]: https://css-tricks.com/the-peculiar-magic-of-flexbox-and-auto-margins/
[29]: #box-model
[30]: https://chrismorgan.info/
[31]: https://piccalil.li/
[32]: https://wattenberger.com/blog/css-percents
[33]: https://www.joshwcomeau.com/tutorials/css/
[34]: https://www.smashingmagazine.com/category/css/
[35]: https://cssfordesigners.com/articles/things-i-wish-id-known-about-css
[36]: https://github.com/dohliam/dropin-minimal-css
[37]: https://github.com/troxler/awesome-css-frameworks
[38]: https://gdcss.netlify.app/gd.css
[39]: https://ishadeed.com/
[40]: https://smolcss.dev/
[41]: https://moderncss.dev/
[42]: https://thinkdobecreate.com/
[43]: https://alistapart.com/articles/
[44]: https://www.modularscale.com
[45]: https://1linelayouts.glitch.me/
[46]: https://web.dev/one-line-layouts/
[47]: https://github.com/iandevlin/html5bones/
[48]: https://github.com/h5bp/html5-boilerplate
[49]: https://www.freecodecamp.org/news/the-100-correct-way-to-do-css-breakpoints-88d6a5ba1862/
[50]: https://gohugo.io/
[51]: https://sagar.se/blog/hugo-documentation/
[52]: https://www.getzola.org
[53]: https://github.com/Keats/tera
[54]: https://jinja.palletsprojects.com/
[55]: https://dev.to/masakudamatsu/favicon-nightmare-how-to-maintain-sanity-3al7
[56]: https://web.dev/add-manifest/
[57]: https://web.dev/maskable-icon/
[58]: https://monochrome.fyi
[59]: https://dataswamp.org/~solene/

[99]: https://github.blog/2021-06-22-framework-building-open-graph-images/
[150]: https://seirdy.one/2020/11/23/website-best-practices.html

--8<-- "include/abbreviations.md"
