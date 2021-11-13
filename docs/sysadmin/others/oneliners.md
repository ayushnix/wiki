---
title: "Useful One Liners"
summary: "Useful One Line Commands — sed, awk, regex, ffmpeg etc"
date: 2021-03-08
---

# sed, awk, vim, regex

print the content between line numbers `a` and `b`

```
sed -n 'a,bp' file.txt
```

replace all the MAC addresses in a file with a custom mac address using *quantifiers* (`{17}`)

```
sed -E -i 's|[a-f0-9A-F:]{17}|00:00:00:00:00:00|g' test.json
```

remove characters (`*` in this case) surrounding words in vim, where words contain alphabets,
numbers, and hyphen by grouping them (`\(` and `\)`) and using backreferences (`\1`)

```
:%s/\*\([[:alnum:]-]\+\)\*/\1/g
```

# ffmpeg

trim audio from a video without re-encoding the audio

```
ffmpeg -i mashiro_no_oto_episode_1.mkv \
  -ss 00:16:28.780 -t 167 -vn -acodec copy \
  setsu_first_performance.aac
```

concatenate/merge multiple audio files

```
ffmpeg -i "concat:setsu_solo_1.aac|setsu_solo_2.aac" \
  -c:a copy -map_metadata 0:s:0 setsu_solo.aac
```

# dd

use dd to write an iso file to a flash drive[^1]

```
dd if=/data/ubuntu.iso of=/dev/sda bs=4096 status=progress conv=fsync
```

[^1]: [See this](https://abbbi.github.io/dd/) for an explanation for why to use `conv=fsync`
