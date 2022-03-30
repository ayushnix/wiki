---
title: "Monitors and Laptops"
summary: "A list of digital displays that should never be bought"
date: 2021-09-26
---

# Say No To Fractional Scaling

If you have an option, **never** buy a monitor or a laptop which needs fractional scaling. There are
a lot of morons on the Internet who'll tell you that they don't need fractional scaling because

- they can just compromise and scale the fonts and make all GUI elements look out of place
- they don't need to scale because their eyes can either see text meant to be read with a magnifying
  glass or they are comfortable reading text with a size that belongs on billboards
- they don't think fractional scaling makes any difference in the quality of text rendering
- they can just use things like `GDK_SCALE` to selectively scale certain software on Linux
- they don't use GUI
- they've never noticed anything bad and everything "works" for them
- scaling, like religion, is a personal preference

and lots of other crap. Don't listen to them.

Yes, I know it feels bad to spend money on a nice and shiny 4K monitor or an expensive laptop with a
high resolution and finding out how fractional scaling is messing things up but it is what it is.
It's better not to deal with the mental stress that fractional scaling creates and just buy a
display that can do integer scaling, if it's possible.

Here are a list of blog posts and comments that might help you make a decision.

- [What is HiDPI] [1]
- [Notes on HiDPI on the SwayWM wiki] [2]

## PPI

You can either use [this] [3] website or calculate the PPI manually using the following formula.

```
sqrt(width^2 + height^2)
------------------------
display size
```

In my case, as on 2021-09-26, I've compromised and scaled my fonts instead of fractionally scaling
my 27 inch 4K monitor which has a PPI of `sqrt((3840^2) + (2160^2)) / 27 ≈ 163`.

## Desktop Monitors

You want a monitor with a PPI exactly, or roughly divisible, by 96. This means that the following
monitors can be bought without needing to deal with fractional scaling

| Name                  | Size      | Resolution     | PPI/96            |
| --------------------- | --------- | -------------- | ----------------- |
| Dell UP3218K          | 31.5 inch | 7680x4320 (8K) | 279/96 ≈ 2.9 ➙ 3x |
| Apple Pro Display XDR | 32 inch   | 6016x3384 (6K) | 215/96 ≈ 2.2 ➙ 2x |
| LG 27MD5KL-B          | 27 inch   | 5120x2880 (5K) | 217/96 ≈ 2.2 ➙ 2x |
| Apple Studio Display  | 27 inch   | 5120x2880 (5K) | 217/96 ≈ 2.2 ➙ 2x |

Yes, there aren't a lot of options out there right now. However, if I do buy a monitor next time,
I'll try to get one of them if possible, even if I have to spend more money than I'd like. No, I
won't buy a shitty 1080p or 1440p monitor to get integer scaling but font rendering from the middle
ages.

## Laptops

Laptops are an interesting case because the rule of divide by 96 doesn't apply there because of the
reduction in physical distance between you and your laptop display. [Chris Morgan gave] [4] some
useful insights about which laptops won't need fractional scaling. He uses 110 - 140 PPI instead of
96. I'm gonna use 125.

| Size      | Resolution        | PPI/96             | Buy                |
| --------- | ----------------- | ------------------ | ------------------ |
| 15.6 inch | 2560x1440 (1440p) | 188/125 ≈ 1.5      | :x:                |
| 15.6 inch | 3840x2160 (4K)    | 282/125 ≈ 2.2      | :white_check_mark: |
| 14 inch   | 2560x1440 (1440p) | 209/125 ≈ 1.6      | :x:                |
| 14 inch   | 3840x2160 (4K)    | 314/125 ≈ 2.5      | :x:                |
| 13.3 inch | 2560x1440 (1440p) | 220/125 ≈ 1.7      | :x:                |
| 13.3 inch | 3840x2160 (4K)    | 331/125 ≈ 2.6      | :x:                |

Some laptops like the Microsoft Surface Book 2 have a screen size of 13.5 inch and a resolution of
3000x2000 which is 267 PPI which means it'll work with 2x scaling just fine. At least someone has
the sense to go out of their way and provide a sensible resolution. Their surface studio all-in-one
PC has a resolution of 4500x3000 on a 28 inch screen which is 192 PPI and works at exactly 2x
scaling. I have no idea about the quality of their laptops or PCs but they got their resolution
right[^1], unlike most manufacturers out there.

[The Framework laptop] [5], probably the most hyped laptop in recent memory (and mostly for good
reasons), also suffers from this issue. It has a 13.5 inch 2256x1504 screen with a PPI of 200 which
leaves us with 1.6x scaling.

# Conclusion

If you care about the quality of your fonts and text rendering and want to avoid issues like
applications going haywire and appearing blurry, avoid monitors and laptops which need fractional
scaling. If you happen to agree with one of the reasons I mentioned in the 1st section above, good
for you.

[^1]: Some of their laptops, the Surface Laptop 2 and 3 apparently need 1.5x scaling. `¯\_(ツ)_/¯`

[1]: https://blog.elementary.io/what-is-hidpi/
[2]: https://github.com/swaywm/sway/wiki#hidpi
[3]: https://www.sven.de/dpi/
[4]: https://news.ycombinator.com/item?id=28375550
[5]: https://frame.work/laptop
