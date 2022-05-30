---
title: "Fonts And Colors"
summary: "A list of fonts and colorschemes which I should use"
date: 2022-01-23
---

# Introduction

I've noticed that I often end up wasting a lot of time searching for fonts and colorschemes to use
in my terminal and in my web browser. This page is meant to serve as a reminder about my aesthetic
tastes and to prevent me from going down into rabbit holes I have already gone into before and do
not want to go into again.

The fonts that I'll reject on this page are usually deceptively good, which basically means that
they look awesome at face value and at an initial cursory glance. However, they turn out to be a
disappointment when put to use. I won't list fonts which I don't even consider worth looking at.
Obviously, there might be some good fonts out there which I haven't come across.

Fortunately, I've already discovered a workhorse dark colorscheme that is comfortable to use, has a
widespread ecosystem, and doesn't give me halation. It fails WCAG 2.0 AAA and WCAG 3.0 APCA tests
but it is the most comfortable dark colorscheme I've found. It's also shipped out of the box in the
Atom text editor. Yes, it's [OneDark][12].

I imagine this issue would sound alien to people who like using smartphones or GNOME users who
assume that everyone on the planet likes the same colors and fonts. It's probably useless but I hope
they realize that people aren't zombies (at least some of them aren't).

# Font Selection Criteria

It should be obvious but font choices are subjective and won't necessarily have an objective reason
for not being used. Even so, I do think there are some criteria that can be used to judge a font,
especially monospace fonts.

In the case of monospace fonts, the font in question

- shouldn't be ambiguous when writing characters like `0`, `O`, `o` and `1`, `l`, `I`
- should have centered alignment for character combinations like `->`, `<-`, `!=`, `=>`  `~`
- should have decent serifs for character combinations like `${}`, `()`, `[]`,
- shouldn't enforce, promote, or encourage using ligatures
- should look good on HighDPI monitors
- shouldn't be a bitmap font

## Monospace Fonts

### Yes :thumbsup_tone3:

- [SF Mono][1]

    This has been my primary font of choice for a long time now. This is an exemplar of how a
    monospace font should be. It's also free from the ligature bullshit.

- [Fira Code][2]

    This is pretty decent font with more serifs than I prefer. Or maybe it just seems that way on my
    monitor which doesn't do integer scaling. For now, this is my second choice when I get bored of
    using SF Mono.

### No :thumbsdown_tone3:

- [Iosevka][2]

    This is a one of a kind font with more than 13k stars as of Jan 2022. At face value, this looks
    like an ideal font because you can literally make your own font with your own preferences for
    character shapes. But, the end result has almost always been a disappointment, at least for me.

## Sans-Serif Fonts

### Yes :thumbsup_tone3:

- [Inter][3]

    This font, combined with SF Mono, is my favorite sans-serif font. I really love how simple and
    legible the characters are. The amount of serifs are perfect as well.

- [Aktiv Grotesk Corp][4]

    This is another great font with somewhat more curves and serifs than Inter but retains the
    professional and legible aesthetic of Inter. I often switch between Inter and Aktiv Grotesk.

- [SF Pro Rounded][11]

    This an extremely pleasant and playful font to look at. I don't think sans-serif fonts get any
    more beautiful than this. It's not as formal as SF Pro or Inter but not really informal or
    unprofessional either. Just a subtle addition of rounded corners in alphabets.

### No :thumbsdown_tone3:

- [IBM Plex Sans][5]

    This is a deceptively good font. I often come across IBM Plex Sans and I like it at face value
    but whenever I start using it, I usually go back to Inter or Aktiv Grotesk within a day or two.

- [Roboto][6]

    This is kind of a workhorse font from Google, not unlike Noto, Droid Sans, DejaVu, and ChromeOS
    fonts which are metric compatible with their Microsoft counterparts. All of these, especially
    the Noto family of fonts, are awesome and should probably be the premier fallback font when you
    want to play safe. However, I find them too bland and devoid of any character.

- [Manrope][7]

    This might look good at an initial glance but it's a bit too much *sans* serif. The `f` and `r`
    character of this font should tell you what I mean.

- [Red Hat Display and Text][8]

    It looks like as if someone took a hammer, hit all the characters, and made them Gimli's
    siblings.

- [Krub][9]

    Krub might've been a good choice as one of the primary sans-serif fonts in use but the font
    weight of its regular variant is lesser than I prefer.

- [Karla][10]

    A bit too short, wide, and serif-y for my taste.

# Colorscheme Selection Criteria

It should be obvious but color choices are subjective and won't necessarily have an objective reason
for not being used. Even so, I do think there are some criteria that can be used to judge a set of
colors.

If it's a light colorscheme, it should

- not use red or colors near red unless absolutely necessary

    Besides issues that can cause layout shift of text on a terminal, red colors should be reserved
    for cases when there's an error or for things that need immediate attention.

If it's a dark colorscheme, besides what's mentioned for light colorschemes, it should

- not cause halation in my eyes when I read text

    If the colors for the text and the background in a dark colorscheme have a contrast value
    highter than 90 in WCAG 3.0 APCA, those colors almost certainly cause halation in my eyes. I
    don't know if this is something that I only experience but it's enough for me to reject APCA as
    something to always adhere to when considering dark color schemes.

## Dark Colorschemes

### Yes :thumbsup_tone3:

- [OneDark][12]

    As I've mentioned before, I really like the medium contrast OneDark colorscheme. It's one of the
    very few colorschemes I've found which doesn't cause halation and can be used everywhere thanks
    to its widespread ecosystem.

### No :thumbsdown_tone3:

## Light Colorschemes

### Yes :thumbsup_tone3:

### No :thumbsdown_tone3:

- [TokyoNight Day][13]

    The text color is blue. No.

[1]: https://developer.apple.com/fonts/
[2]: https://github.com/be5invis/iosevka
[3]: https://rsms.me/inter/
[4]: https://fonts.adobe.com/fonts/aktiv-grotesk
[5]: https://www.ibm.com/plex/
[6]: https://fonts.google.com/specimen/Roboto
[7]: https://github.com/sharanda/manrope
[8]: https://www.redhat.com/en/about/brand/standards/typography
[9]: https://github.com/cadsondemak/Krub
[10]: https://github.com/googlefonts/karla
[11]: https://developer.apple.com/fonts/
[12]: https://github.com/atom/atom/tree/master/packages/one-dark-syntax
[13]: https://github.com/folke/tokyonight.nvim
