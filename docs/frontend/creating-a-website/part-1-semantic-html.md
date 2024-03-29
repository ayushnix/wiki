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
which templating engines can do. However, if you want a preview of how simple a static site
generator can be, check out [Simple Blogging System][3].

Unfortunately, I haven't been able to find a definitive resource on the Internet for learning how to
write HTML. Fortunately, I was able to find websites like [Seirdy's Home Page][4], [Oriole Systems'
Blog][5], and the [Parity Bit Blog][6], all of which have a decent web page with quite informative
HTML.

If you want a recap of the basics of HTML, the [HTML Basics][7] page on MDN is a decent place to
start but for those who do have a basic idea of HTML works, this block of code should be a pretty
decent starting point.

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="color-scheme" content="light dark">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ayushnix home page</title>
  </head>
  <body>
  </body>
</html>
```

The `<meta>` tag containing `charset="utf-8"` should be the immediate child of the `<head>` tag. The
`<meta>` tag with the `viewport` makes our page responsive. The `color-scheme` `<meta>` tag is
interesting because it can [apply dark and light mode][8] colors on your web page without using any
custom CSS. The colors are derived from your operating system and your web browser. Since this is
part of the HTML itself, it also loads before any CSS file is downloaded and applied.

If you're still wondering why a static site generator might be needed, the `<title>` tag will need
to generated dynamically for each different page. Even if the tags don't change across web pages,
they'll still need to copied on all the pages that you write. It doesn't make sense to copy these
tags on multiple web pages or update tags based on context manually.

# Semantic HTML

As you may have guessed, the `<head>` tag is responsible for hosting the metadata for our website,
which typically involves using the `<meta>`, `<link>`, and the `<title>` tag. The `<body>` tag is
where the actual web page content begins. This is where it's imperative to realize that content
comes before presentation. If you're creating a personal website, it should work fine in console web
browsers like [w3m][9] and [lynx][10].

At the time this post is being written, I'm trying to create a personal blog website which features
a microblog as well. I was inspired to do it after I came across [zunzuncito][11], the personal
microblog of [Wolfgang Müller][12]. However, I didn't want to just blindly copy and paste his code.
I want to understand how to create basic websites using HTML and CSS.

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

At this point, we're in the messy territory of HTML5, its "semantic" tags, and its differences from
HTML4. In HTML4, the outline, or table of contents, of a web page was determined by using the `<h*>`
tags. To avoid messing things up, here are some simple rules about how to write HTML4 pages using
only heading tags:

- only one `<h1>` tag should be present in a HTML page
- `<h*>` tags should always be used in their ascending numerical order, i.e., `<h3>` can't appear
  before `<h2>` in an HTML page
- `<h*>` tags shouldn't skip order, i.e., `<h3>` can't appear if `<h2>` hasn't been used on a HTML
  page

This rigid structure is simple enough to understand and work with.

HTML5 introduced the `article`, `section`, `aside`, and `nav` tags which influence the structural
outline of a HTML page. If you're confused, [this answer by Robert Siemer][72] is a good place to
start. However, the document outline algorithm introduced by HTML5 and these semantic tags hasn't
actually been implemented in any web browser and there's been a disconnect between what's advised
and what's actually implemented in reality. This has been discussed on [a popular post][13] by Steve
Faulkner on the website HTML5 Doctor.

One of the promises of these semantic sectioning tags was that you could mess around with the order
of the appearance of the `<h*>` tags and it'd be fine. An `<h3>` tag could be followed by an `<h2>`
tag inside a page if they were inside distinct section tags. However, since this isn't actually the
reality and this would also mess up our heading level structure, it's better to stick to the HTML4
heading outline rules. We can still benefit from these HTML5 tags because they carry implicit ARIA
roles which are good for accessibility. It also beats writing `<header>` instead of `<div
class=header>`.

It should be kept in mind that the `<section>` tag should always have a `<h*>` tag as its child. If
you can't come up with heading tag, you shouldn't use the `<section>` tag. The `<article>` tag
should ideally have a child `<h*>` tag as well. The `<aside>` and `<nav>` tags should have a `<h*>`
child tag but it may not be feasible to come up with a heading for these tags, considering the
content they're likely to contain.

If you're interested, a [blog post on the cutecodedown][14] website makes a compelling case against
HTML5. The [Nu HTML Checker][15] can help you visualize the outline of your HTML document in terms
of headings and the new HTML5 section tags.

``` html
<body>
  <h1>Ayush Agarwal</h1>
  <header>
    <ul>
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
      <h2>the title of the post</h2>
      <header>
        <time datetime="2022-03-07T14:54:39.929">2022-03-27</time>
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

At this point, your web page should be ready to view on a text based web browser like w3m. If the
layout of your HTML page looks similar on w3m and a "modern" web browser, say Firefox, it's a good
thing. If you want to dive deeper into making your HTML friendly to search engines, readability
mode implementations in web browsers, and transform data on your website into well defined parser
friendly structured data, check out the [Microdata and Microformats][74] section.

# CSS

Unlike HTML, I was able to find a lot of resources to explain how CSS works and the different kinds
of layouts one can build for websites. I'd go so far as to say that there's too much information out
there and it can be overwhelming to make the "right" decisions.

There's no point in me going over CSS in detail because I'm not a web designer or developer, nor do
I intend to be. The [Learn CSS][16] module can be a good place to start, although it does seem to
make assumptions about its audience being familiar with CSS. The [MDN docs about CSS][17] are more
detailed and will probably serve as a better introduction.

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

elements and pseudo elements
:    `html {}` has a specificity score of `0,0,1`, `html body {}` has a score of `0,0,2`, `a::after
     {}` has a score of `0,0,2`

classes, pseudo classes, and attribute selectors
:    `.hyperlink::before {}` and `:target::after` have a score of `0,1,1`, `[role="list"] {}` has a
     score of `0,1,0`

id selectors
:    `#title {}` has a score of `1,0,0`

inline css

It's important to remember that `*`, `+`, `>` and `~` don't have any effect over specificity. And
no, don't use `!important` in your code, unless you're overriding CSS for another website for use
with the [Stylus][18] web browser add-on. If you want to check the specificity of your rules, the
[polypane specificity tool][75] is pretty good. There are some pseudo classes such as `:where()`
which have 0 specificity and `:is()` which derive specificity from their most specific selector
item.

Some CSS properties, like [`font-size`][19], might get inherited from the parent element and this
can be verified using the formal definition sections on MDN.

## CSS Selectors

The [CSS Diner][20] game should help you get familiar with CSS selectors.

## Box Model

The [MDN page on the box model][76], which links to a [post on CSS Tricks][77], recommends using the
following CSS reset to change the box model

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
the element will be inherited by its children. If we had used the following code, even if we change
the box model of a tag, its children won't inherit that box model.

``` css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

## Fonts

It's amazing how people just copy paste shit they find on the internet and not try to understand it.
[This article on CSS Tricks][60] talks about system font stack but never mentions what the hell does
the following piece of code even mean

``` css
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
}
```

and if it's even necessary. I mean, why not simply use

``` css
body {
  font-family: sans-serif;
}
```

and call it a day?

It's amusing how the information I was looking for was found on a [random comment on Reddit][61].
Apparently, `sans-serif` is a fallback font and if a user has set some weird font that shouldn't be
used but is used anyways, you may want to display your site in a better font. But then again, why
would I want to override a user's wishes to use a font of his choice?

The emoji fonts are further fallbacks in case your website uses emojis. Those emojis may not be
found in the font chosen by `sans-serif` (which isn't unlikely) so it makes sense to add them. This
is how `font-family` looks like for now

``` css
:root {
  --sans-font: sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Noto Color Emoji";
}
```

This should cover Apple, Windows, and Linux when it comes to emojis. However, some Linux users may
not have installed Noto Color Emoji on their desktops. I'm not sure if there's a good solution for
those people.

As expected, Apple have their own way of doing things (perhaps for good reasons in this case) and
that's what `-apple-system` is. I was able to find [this article on Smashing Magazine][62] which
does bother to go into detail about what each term inside `font-family` means. Basically,
`-apple-system` will pick the right font on the right Apple device for you and you don't have to
think about it. `BlinkMacSystemFont` is just for doing the same thing on Chrome on MacOS.

``` css
:root {
  --sans-font: -apple-system, BlinkMacSystemFont, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Noto Color Emoji";
}
```

This brings us to the question of specifying custom fonts before `sans-serif` and overriding user's
wishes, which is a difficult position to take, at least for me. There are two reasons override the
defaults,

- the user may be an idiot and may have chosen a default font that shouldn't be used
- the operating system provides shitty fonts out of the box and you don't want your website to look
  shitty as well

It doesn't help that Chromium browsers don't let you chose the fonts you want to use, unlike
Firefox[^2]. I'll go ahead and use two custom fonts — one for Windows and one for Linux.

I don't know if you care about typography and the fonts on your system but I do and it's no wonder
that some of them look like they belong in the garbage truck that arrives every morning. Arial,
Courier and its variants, Times New Roman, and Verdana belong in the garbage truck[^1]. Segoe UI is
bearable. Let's go with Segoe UI on Windows.

We don't need to worry about fonts on Android, just like we didn't have to worry about it when
considering Apple devices. For better or worse, both of them run a pretty tight ship and consumer
freedom belongs in `/dev/null` on both of these platforms. This certainly makes things easier for
developers but makes users dense and umimaginative. `sans-serif` will take care of fonts on Android.
The end result should be either `Roboto` or the newer Google Sans font.

The only thing that's left is Linux, which is sorta like the wild west. Unfortunately, this is
changing, thanks to GNOME. Yeah, I used the word unfortunately because they're converging towards
the same design decisions that Google and Apple are making on their platforms. The default font
choices on Linux, such as Noto Sans, DejaVu Sans, Liberation Sans, and Nimbus Sans look ... bland at
best and poor at worst. Opinion? Obviously. If there's one font that I would consider adding to this
stack, it's the [Inter font][67]. If it's installed, we definitely want to use that. I've also
covered this font on [this page][64] in my wiki.

I've omitted `system-ui` because of [this issue][65].

This brings us down to

``` css
:root {
  --sans-font: -apple-system, BlinkMacSystemFont, "Segoe UI", Inter, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Noto Color Emoji";
}
```

I should emphasize that if you want things to be simple, you could just use

``` css
:root {
  --sans-font: -apple-system, BlinkMacSystemFont, sans-serif;
}
```

and be done with it. This should work fine in almost all cases and on all platforms. If `system-ui`
didn't have the issues that it does, `--sans-font` could've been reduced to just two values.

That finally covers the sans serif fonts. What about the monospace fonts for the code sections?

``` css
:root {
  --mono-font: ui-monospace, "Cascadia Mono", Consolas, "SF Mono", Inconsolata, monospace;
}
```

or, if you want things to be simple

``` css
:root {
  --mono-font: ui-monospace, Consolas, monospace;
}
```

If you're wondering what these choices mean (as you should be),

- `ui-monospace` is something [supported only by Apple][66] devices for the moment and it should
  automatically select the best available monospace font on the platform, which is probably SF Mono
- `Consolas` is the most well known and relatively sensible monospace font on Windows, we absolutely
  do NOT want `monospace` to fallback to courier or courier new (man, what a terrible font)
- as expected, `monospace` is the equivalent of `sans-serif`, the default fallback monospace font

It's also likely that many Windows installations would have `Cascadia Code` installed so we can
choose to give that higher priority than `Consolas`. I've inserted `SF Mono` as my font of choice on
Linux. In addition, `Inconsolata` has been added because it looks decent (similar to Consolas) and
has a good chance of being installed on a typical Linux installation.

What about serif fonts? Unfortunately, they shouldn't be used on screen because most users use
shitty monitors which aren't HighDPI. However, you can use serif fonts or sans-serif fonts which
retain some serifs on devices with a display DPR of at least 2 and preferably 3. I'm not interested
in exploring this for the moment.

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
  margin-inline: auto;
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
- [simple.css][68]

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

# Hyperlinks

TL;DR?

- don't change the color of the link on hover to prevent [layout shift of text][70]
- don't underline your links by default

The first point should be self-explanatory after reading the GitHub issue specified in the ...
hyperlink. At the time of this writing, my own wiki has this issue. Fortunately, my microblog
doesn't because I built it myself. In case you didn't click on that link, changing the color of a
hyperlink on a hover can cause it to drift up or down and cause layout shifts on your monitor if the
color of the hovered text goes from blue to red and potentially other colors as well. Layout shift
is bad, m'kay?

The second point needs some explanation. Seirdy makes a compelling point about underlines serving a
purpose by highlighting that two hyperlinks are different. However, that comes at the [cost of
readability][71], which I believe is more of a problem these days than being able to distinguish two
different links. Sure, I still color my hyperlinks differently than the text color to distinguish
them and that does make things bad, underlining them makes it worse, especially if your page has a
lot of hyperlinks.

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

Don't forget to optimize your SVG image. I used SVGO to shave down the size of my SVG image from
~5KB to ~1KB. There are several articles about using the `style` tag inside SVG images to color them
differently in dark mode using media queries. However, I think we should avoid doing that because
users can edit their browser UI into colors that may not necessarily be dark while they are in dark
mode. In such cases, you don't want your SVG image to be illegible. The best choice in this case is
to add a background to your SVG text/icon which will look decent in most colors. Or, you can choose
an icon that looks decent on most color backgrounds.

[Here's a comment by Chris Morgan][69] on Hacker News about using media queries in SVG images

> Caution should also be exercised with the `prefers-color-scheme` media query on SVG icons. It does
> not reflect whether the icon is being displayed on a light or dark background.
> `(prefers-color-scheme: dark)` hints that your icon is probably being displayed against a dark
> background, but it could easily still be being displayed against a light background; and with
> `(prefers-color-scheme: light)` it will be quite common for the icon to be being displayed against
> a dark background. You need to carefully design your icon so it works well against almost any
> background colour.

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

# Microdata and Microformats

I didn't want to address this in the [Semantic HTML][73] section because it would probably overwhelm
anyone who wants to a setup a website. However, if you want to make your website friendly to

- search engines
- readability mode implementations in web browsers

you should consider adding metadata defined in schema.org microdata and microformats2 to your HTML markup.

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
[60]: https://css-tricks.com/snippets/css/system-font-stack/
[61]: https://old.reddit.com/r/web_design/comments/4sf18z/githubs_font_changed_without_any_announcement/d5938jw/
[62]: https://www.smashingmagazine.com/2015/11/using-system-ui-fonts-practical-guide/
[63]: ../others/monitors_and_laptops.md
[64]: ../others/fonts.md
[65]: https://infinnie.github.io/blog/2017/systemui.html
[66]: https://caniuse.com/extended-system-fonts
[67]: https://rsms.me/inter/
[68]: https://github.com/kevquirk/simple.css
[69]: https://news.ycombinator.com/item?id=25525246
[70]: https://github.com/fish-shell/fish-shell/issues/8462
[71]: https://memex.marginalia.nu/log/00-linkpocalypse.gmi
[72]: https://stackoverflow.com/a/26579514
[73]: #semantic-html
[74]: #microdata-and-microformats
[75]: https://polypane.app/css-specificity-calculator/
[76]: https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model
[77]: https://css-tricks.com/inheriting-box-sizing-probably-slightly-better-best-practice/
[99]: https://github.blog/2021-06-22-framework-building-open-graph-images/
[150]: https://seirdy.one/2020/11/23/website-best-practices.html

[^1]:
If we're being fair, most of the blame lies on HighDPI displays not being in widespread usage. It's
not readily apparent unless you've looked at a HighDPI display but everything looks like shit on 96
PPI displays. I've talked about this in the [monitors and laptops][63] page on this wiki. No wonder
San Francisco fonts are an absolute pleasure to look at and work with.

[^2]:
Let's hope Firefox doesn't follow suit and remove this feature because the designers at Mozilla have
made pretty stupid and annoying decisions.

--8<-- "include/abbreviations.md"
