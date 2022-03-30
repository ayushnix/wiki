---
title: "Fonts"
summary: "A list of fonts which I should use"
date: 2022-01-23
---

## Introduction

I've noticed that I often end up wasting a lot of time searching for the best fonts to use in my
terminal and in my web browser. This page will serve as a reminder about which fonts I should use
and, more importantly, which fonts I shouldn't use and shouldn't spend time looking at anymore.

The fonts that I reject on this page are deceptively good. I'll explain what that means later. This
means that I'll only list fonts which I like and fonts which I am tempted to like but end up
rejecting each and every time I start using them. I won't list fonts which I don't even consider
worth looking at. Obviously, there might be some good fonts out there which I haven't come across.

I imagine this issue would sound alien to people who like using smartphones or GNOME users but I
really love how different fonts can evoke different feelings and how some of them can make you feel
comfortable while others can turn you off. Suffice to say, there is no "one true font". Yes, I'm
talking about Cantarell.

## Selection Criteria

I guess it should obvious but font choices are subjective and won't necessarily have an objective
reason for not being used. Even so, I do think there are some criteria that can be used to judge a
font, especially monospace fonts.

In the case of monospace fonts, the font in question

- should'nt be ambiguous when it comes to characters like `0`, `O`, `o` and `1`, `l`, `I`
- characters like `->`, `<-`, `!=`, `=>` `${}`, `()`, `[]`, `~` should be legible and should have
  sensible alignment
- shouldn't enforce, promote, or encourage using ligatures
- look good on HighDPI monitors
- shouldn't be a bitmap font

## Monospace Fonts

### Yes :thumbsup_tone3:

- [SF Mono] [1]

    This has been my primary font of choice for a long time now. This is an exemplar of how a
    monospace font should be. It's also free from the ligature bullshit.

### No :thumbsdown_tone3:

- [Iosevka] [2]

    This is a one of a kind font with more than 13k stars as of Jan 2022. At face value, this looks
    like an ideal font because you can literally make your own font with your own preferences for
    character shapes. But, the end result has almost always been a disappointment, at least for me.

## Sans-Serif Fonts

### Yes :thumbsup_tone3:

- [Inter] [3]

    This font, combined with SF Mono, is my favorite sans-serif font. I really love how simple and
    legible the characters are.

- [Aktiv Grotesk Corp] [4]

    This is another great font with somewhat more curves and serifs than Inter but retains the
    professional and legible aesthetic of Inter. I often switch between Inter and Aktiv Grotesk.

### No :thumbsdown_tone3:

- [IBM Plex Sans] [5]

    This is a deceptively good font. I often come across IBM Plex Sans and I like it at face value
    but whenever I start using it, I usually go back to Inter or Aktiv Grotesk within a day or two.

- [Roboto] [6]

    This is kind of a workhorse font from Google, not unlike Noto, Droid Sans, DejaVu, and ChromeOS
    fonts which are metric compatible with their Microsoft counterparts. All of these, especially
    the Noto family of fonts, are awesome and should probably be the premier fallback font when you
    want to play safe. However, I find them too bland and devoid of any character.

- [Manrope] [7]

    This might look good at an initial glance but it's a bit too much *sans* serif. The `f` and `r`
    character of this font should tell you what I mean.

- [Red Hat Display and Text] [8]

    It looks like as if someone took a hammer, hit all the characters, and made them Gimli's
    siblings.

[1]: https://developer.apple.com/fonts/
[2]: https://github.com/be5invis/iosevka
[3]: https://rsms.me/inter/
[4]: https://fonts.adobe.com/fonts/aktiv-grotesk
[5]: https://www.ibm.com/plex/
[6]: https://fonts.google.com/specimen/Roboto
[7]: https://github.com/sharanda/manrope
[8]: https://www.redhat.com/en/about/brand/standards/typography
