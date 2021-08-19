---
title: "Useful One Liners"
summary: "Useful One Line Commands — sed, awk, regex, ffmpeg etc"
date: 2021-03-08
---

# sed, awk, regex

print the content between line numbers `a` and `b`

```
sed -n 'a,bp' file.txt
```

replace all the MAC addresses in a file with a custom mac address

```
sed -E -i 's|[a-f0-9A-F:]{17}|00:00:00:00:00:00|g' test.json
```

# ffmpeg

trim audio from a video without re-encoding the audio

```
ffmpeg -i mashiro_no_oto_episode_1.mkv -ss 00:16:28.780 -t 167 -vn -acodec copy setsu_first_performance.aac
```

# dd

use dd to write an iso file to a flash drive[^1]

```
dd if=/data/ubuntu.iso of=/dev/sda bs=4096 status=progress conv=fsync
```

[^1]: [See this](https://abbbi.github.io/dd/) for an explanation for why to use `conv=fsync`
