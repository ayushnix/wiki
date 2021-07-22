---
title: "Linux Desktop Is a Shit Show"
summary: >-
  "Unless you enjoy fixing broken shit more than getting actual work done, you're not gonna like
  Linux on your desktop or laptop."
date: 2021-03-01
---

This article is meant to serve as the collection of my gripes and frustrations I'm either facing
right now, or have faced in the past, when using the Linux on the desktop. I've been using Linux on
my desktop exclusively since 2015 instead of Windows or MacOS.

For people out there who're pedantic and radicalized enough to miss the point of the article, this
isn't just about the Linux kernel itself but rather about using the Linux ecosystem on a desktop. I
doubt you can use the Linux kernel itself on your desktop to do anything meaningful.

To be clear, me writing this article doesn't mean that I'm anti-Linux (which is a pretty naive way
of looking at things) or against using Linux on the desktop. As I said, I've been using Linux as my
desktop since 2015 and probably will continue to do so although I haven't used MacOS yet so I guess
I might change my mind once I use it. The primary reasons I'm still using Linux is because I've
gotten used to its customizability and user freedom. I tried using KDE for a short amount of time
and although it was a breath of fresh air considering how insane GNOME is, I quickly went back to
[i3wm](https://i3wm.org/). However, this freedom and customization comes at the cost of security,
reliability, polish, and your time.

# Doing The Unusual — Criticising Linux

People on Reddit and other online communities see Linux as this perfect panacea of user freedom,
customization, and power. The shilling never stops and unsuspecting users are led to believe that
Microsoft and Apple are sitting on their asses and doing nothing on their desktops while a community
of open source developers are doing something better. They fail to mention things like Linux is
still stuck with an ancient display server called Xorg which [has been called a security
circus](https://blog.invisiblethings.org/2011/04/23/linux-security-circus-on-gui-isolation.html).
Wayland is supposed to be a replacement for Xorg but it is nowhere close to being a daily driver, at
least not for me. No, I'm not a gamer or a streamer or someone with an exotic use-case. I simply
browse the web, use neovim and vscode, and watch videos. I tried using [sway](https://swaywm.org/)
from mid 2020 to early 2021 and at the end, I thought about throwing my laptop out the window.
Fortunately, I had the good sense to go back to [i3wm](https://i3wm.org/). My laptop is still on my
table since then.

# Undisclosed Disclaimers Before You Start Using Linux

You've started using Linux on the desktop but there are some issues which are preventing you from
starting your work. You start diagnosing and searching for fixes. You end up wasting several hours
or maybe days. Eventually, you feel frustrated and open a support thread hoping to find solutions.
When you don't, you express frustration. To your amusement, people on the support thread start
blaming you.

> You are just blabbering and complaining here. This is open-source, volunteer-driven software. If
> you don't like the service, either find something else, or roll up your sleeves and help fix the
> problem(s).

Here's another such sentiment.

> In the case of Wayland, the “vague authority” are a bunch of volunteers who have devoted tens of
> thousands of hours of their free time towards making free shit for you.

In essence, what they mean is

> Either be grateful for the free shit you're using or fuck off. You don't get to complain.

Why don't Linux distributions write something like this on their websites? Instead of making blog
posts like [this](https://drewdevault.com/2021/06/14/Provided-as-is-without-warranty.html), why not
write this at the start of the `README.md` of your projects? It would save the users a lot of time
wouldn't it? A polite, and appropriate disclaimer, can be mentioned at the start of your project.

> This project isn't meant for serious use. Please don't expect that you'll be able to carry on your
> personal or professional work reliably using this software.

If something like this would've been mentioned on sway's GitHub page, I probably wouldn't have
wasted 6 months on it.

> We've sacrificed our spare time to build this for you for free. If you turn around and harass us
> based on some utterly nonsensical conspiracy theories, then you’re a fucking asshole.

If you go around proselytizing Wayland and sway or some other OSS software and call it "stable" and
"ready for daily use" and then basic shit like VSCode[^1] and Firefox doesn't work as expected, perhaps
you're the asshole for wasting people's time by baiting them to switch from their established
workflows with promises of a better software.

Since the praise for Linux, and criticism for anyone who speaks otherwise, never stops and no one
bothers to highlight serious issues with the Linux desktop, this is my attempt to do so.

# List of Issues

## AMDGPU

### System Crashes

For almost the entire two months of May and June 2021, my Thinkpad E495 with AMD Ryzen 3500U CPU
kept [crashing](https://bugzilla.kernel.org/show_bug.cgi?id=201957) almost daily in the middle of my
work and I had to do hard reboots. This was most probably an [AMDGPU
issue](https://bbs.archlinux.org/viewtopic.php?id=266358), going back all the way to 2018 or maybe
earlier.

I couldn't reproduce it reliably but it happened enough that I sometimes wanted to flip my table. I
tried using the LTS kernel 5.10.23 to 5.10.41 but that was basically broken. I got stack traces,
page faults, and system freezes almost everyday. This also happened in 5.11.x but lesser than
5.10.x. Switching to 5.12.x made this problem infrequent but it was still there.

??? failure "AMD GPU crash logs with kernel 5.12.6"
    ```
    [205665.449963] myhost kernel: amdgpu 0000:04:00.0: AMD-Vi: Event logged [IO_PAGE_FAULT domain=0x0000 address=0x10b500040 flags=0x0070]
    [205665.449976] myhost kernel: amdgpu 0000:04:00.0: AMD-Vi: Event logged [IO_PAGE_FAULT domain=0x0000 address=0x10b540000 flags=0x0070]
    [205666.437814] myhost kernel: hub 1-1:1.0: hub_ext_port_status failed (err = -71)
    [205666.437939] myhost kernel: usb 1-1-port4: cannot disable (err = -71)
    [205666.437944] myhost kernel: PM: dpm_run_callback(): usb_dev_resume+0x0/0x10 returns -71
    [205666.437956] myhost kernel: usb 1-1.4: PM: failed to resume async: error -71
    [205666.445050] myhost kernel: hub 1-1:1.0: hub_ext_port_status failed (err = -71)
    [205666.445502] myhost kernel: done.
    [205667.208609] myhost upowerd[942]: treating change event as add on /sys/devices/pci0000:00/0000:00:08.1/0000:04:00.3/usb1/1-2
    [205667.227924] myhost upowerd[942]: treating change event as add on /sys/devices/pci0000:00/0000:00:08.1/0000:04:00.3/usb1/1-4
    [205667.236266] myhost upowerd[942]: treating change event as add on /sys/devices/pci0000:00/0000:00:08.1/0000:04:00.3/usb1/1-3
    [205667.245469] myhost upowerd[942]: treating change event as add on /sys/devices/pci0000:00/0000:00:08.1/0000:04:00.3/usb1/1-4
    [205667.254725] myhost upowerd[942]: treating change event as add on /sys/devices/pci0000:00/0000:00:08.1/0000:04:00.3/usb1/1-1
    [205667.263348] myhost upowerd[942]: treating change event as add on /sys/devices/pci0000:00/0000:00:08.1/0000:04:00.3/usb1/1-2
    [205667.271963] myhost upowerd[942]: treating change event as add on /sys/devices/pci0000:00/0000:00:08.1/0000:04:00.3/usb1/1-3
    [205676.724843] myhost kernel: [drm:drm_atomic_helper_wait_for_flip_done [drm_kms_helper]] *ERROR* [CRTC:67:crtc-0] flip_done timed out
    [205676.935702] myhost kernel: [drm:amdgpu_dm_atomic_check [amdgpu]] *ERROR* [CRTC:67:crtc-0] hw_done or flip_done timed out
    [205686.961512] myhost kernel: [drm:drm_atomic_helper_wait_for_dependencies [drm_kms_helper]] *ERROR* [CRTC:67:crtc-0] flip_done timed out
    [205696.987694] myhost kernel: [drm:drm_atomic_helper_wait_for_dependencies [drm_kms_helper]] *ERROR* [PLANE:55:plane-3] flip_done timed out
    [205697.051522] myhost kernel: ------------[ cut here ]------------
    [205697.051525] myhost kernel: WARNING: CPU: 0 PID: 241 at drivers/gpu/drm/amd/amdgpu/../display/amdgpu_dm/amdgpu_dm.c:7960 amdgpu_dm_atomic_commit_tail+0x25c1/0x2630 [amdgpu]
    [205697.051830] myhost kernel: Modules linked in: snd_seq_dummy snd_hrtimer snd_seq snd_seq_device wireguard curve25519_x86_64 libchacha20poly1305 chacha_x86_64 poly1305_x86_64 libblake2s blake2s_x86_64 libcurve25519_generic libchacha libblake2s_generic ip6_udp_tunnel udp_tunnel lm92 iwlmvm xxhash_generic mac80211 snd_hda_codec_conexant btrfs libarc4 snd_hda_codec_generic snd_hda_codec_hdmi intel_rapl_msr intel_rapl_common iwlwifi blake2b_generic snd_hda_intel xor raid6_pq edac_mce_amd snd_intel_dspcfg wmi_bmof libcrc32c snd_intel_sdw_acpi kvm_amd cfg80211 snd_hda_codec nls_iso8859_1 vfat kvm snd_hda_core fat snd_hwdep irqbypass ucsi_acpi snd_pcm r8169 rapl snd_rn_pci_acp3x typec_ucsi thinkpad_acpi sp5100_tco pcspkr psmouse platform_profile realtek snd_timer k10temp mdio_devres ledtrig_audio cdc_acm snd_pci_acp3x i2c_piix4 typec joydev mousedev libphy rfkill roles snd wmi tpm_crb video soundcore i2c_scmi tpm_tis tpm_tis_core pinctrl_amd mac_hid acpi_cpufreq pkcs8_key_parser fuse crypto_user bpf_preload
    [205697.051903] myhost kernel:  ip_tables x_tables ext4 crc32c_generic crc16 mbcache jbd2 usbhid dm_crypt cbc encrypted_keys dm_mod trusted tpm serio_raw atkbd libps2 crct10dif_pclmul crc32_pclmul crc32c_intel ghash_clmulni_intel aesni_intel crypto_simd cryptd ccp xhci_pci xhci_pci_renesas rng_core i8042 serio amdgpu drm_ttm_helper ttm gpu_sched i2c_algo_bit drm_kms_helper syscopyarea sysfillrect sysimgblt fb_sys_fops cec drm agpgart
    [205697.051940] myhost kernel: CPU: 0 PID: 241 Comm: kworker/0:1H Not tainted 5.12.5-arch1-1 #1
    [205697.051944] myhost kernel: Hardware name: LENOVO 20NECTO1WW/20NECTO1WW, BIOS R11ET36W (1.16 ) 03/30/2020
    [205697.051947] myhost kernel: Workqueue: events_highpri dm_irq_work_func [amdgpu]
    [205697.052246] myhost kernel: RIP: 0010:amdgpu_dm_atomic_commit_tail+0x25c1/0x2630 [amdgpu]
    [205697.052543] myhost kernel: Code: ff ff 01 c7 85 1c fd ff ff 37 00 00 00 c7 85 24 fd ff ff 20 00 00 00 e8 9d fd 12 00 e9 05 fb ff ff 0f 0b e9 30 f9 ff ff 0f 0b <0f> 0b e9 a0 f9 ff ff 0f 0b e9 b9 f9 ff ff 49 8b 06 41 0f b6 8e 2d
    [205697.052546] myhost kernel: RSP: 0018:ffff9676c0fe7a28 EFLAGS: 00010002
    [205697.052550] myhost kernel: RAX: 0000000000000002 RBX: 0000000000000004 RCX: ffff8a1ccb58e118
    [205697.052552] myhost kernel: RDX: 0000000000000001 RSI: 0000000000000297 RDI: ffff8a1ccb480178
    [205697.052554] myhost kernel: RBP: ffff9676c0fe7dc0 R08: 0000000000000005 R09: 0000000000000000
    [205697.052556] myhost kernel: R10: ffff9676c0fe7988 R11: ffff9676c0fe798c R12: 0000000000000286
    [205697.052558] myhost kernel: R13: ffff8a1ccb58e000 R14: ffff8a1e9fb9c600 R15: ffff8a1d4b44a200
    [205697.052560] myhost kernel: FS:  0000000000000000(0000) GS:ffff8a1f70a00000(0000) knlGS:0000000000000000
    [205697.052563] myhost kernel: CS:  0010 DS: 0000 ES: 0000 CR0: 0000000080050033
    [205697.052565] myhost kernel: CR2: 00007f4c22177000 CR3: 000000017eb28000 CR4: 00000000003506f0
    [205697.052568] myhost kernel: Call Trace:
    [205697.052583] myhost kernel:  commit_tail+0x94/0x120 [drm_kms_helper]
    [205697.052611] myhost kernel:  drm_atomic_helper_commit+0x113/0x140 [drm_kms_helper]
    [205697.052636] myhost kernel:  dm_restore_drm_connector_state+0xef/0x170 [amdgpu]
    [205697.052933] myhost kernel:  handle_hpd_irq+0x118/0x150 [amdgpu]
    [205697.053228] myhost kernel:  process_one_work+0x214/0x3e0
    [205697.053235] myhost kernel:  worker_thread+0x4d/0x3d0
    [205697.053238] myhost kernel:  ? process_one_work+0x3e0/0x3e0
    [205697.053242] myhost kernel:  kthread+0x133/0x150
    [205697.053246] myhost kernel:  ? kthread_associate_blkcg+0xc0/0xc0
    [205697.053250] myhost kernel:  ret_from_fork+0x22/0x30
    [205697.053258] myhost kernel: ---[ end trace 752ed8c5d8483260 ]---
    [205697.053283] myhost kernel: ------------[ cut here ]------------
    [205697.053284] myhost kernel: WARNING: CPU: 0 PID: 241 at drivers/gpu/drm/amd/amdgpu/../display/amdgpu_dm/amdgpu_dm.c:7560 amdgpu_dm_atomic_commit_tail+0x25c8/0x2630 [amdgpu]
    [205697.053581] myhost kernel: Modules linked in: snd_seq_dummy snd_hrtimer snd_seq snd_seq_device wireguard curve25519_x86_64 libchacha20poly1305 chacha_x86_64 poly1305_x86_64 libblake2s blake2s_x86_64 libcurve25519_generic libchacha libblake2s_generic ip6_udp_tunnel udp_tunnel lm92 iwlmvm xxhash_generic mac80211 snd_hda_codec_conexant btrfs libarc4 snd_hda_codec_generic snd_hda_codec_hdmi intel_rapl_msr intel_rapl_common iwlwifi blake2b_generic snd_hda_intel xor raid6_pq edac_mce_amd snd_intel_dspcfg wmi_bmof libcrc32c snd_intel_sdw_acpi kvm_amd cfg80211 snd_hda_codec nls_iso8859_1 vfat kvm snd_hda_core fat snd_hwdep irqbypass ucsi_acpi snd_pcm r8169 rapl snd_rn_pci_acp3x typec_ucsi thinkpad_acpi sp5100_tco pcspkr psmouse platform_profile realtek snd_timer k10temp mdio_devres ledtrig_audio cdc_acm snd_pci_acp3x i2c_piix4 typec joydev mousedev libphy rfkill roles snd wmi tpm_crb video soundcore i2c_scmi tpm_tis tpm_tis_core pinctrl_amd mac_hid acpi_cpufreq pkcs8_key_parser fuse crypto_user bpf_preload
    [205697.053646] myhost kernel:  ip_tables x_tables ext4 crc32c_generic crc16 mbcache jbd2 usbhid dm_crypt cbc encrypted_keys dm_mod trusted tpm serio_raw atkbd libps2 crct10dif_pclmul crc32_pclmul crc32c_intel ghash_clmulni_intel aesni_intel crypto_simd cryptd ccp xhci_pci xhci_pci_renesas rng_core i8042 serio amdgpu drm_ttm_helper ttm gpu_sched i2c_algo_bit drm_kms_helper syscopyarea sysfillrect sysimgblt fb_sys_fops cec drm agpgart
    [205697.053678] myhost kernel: CPU: 0 PID: 241 Comm: kworker/0:1H Tainted: G        W         5.12.5-arch1-1 #1
    [205697.053681] myhost kernel: Hardware name: LENOVO 20NECTO1WW/20NECTO1WW, BIOS R11ET36W (1.16 ) 03/30/2020
    [205697.053683] myhost kernel: Workqueue: events_highpri dm_irq_work_func [amdgpu]
    [205697.053979] myhost kernel: RIP: 0010:amdgpu_dm_atomic_commit_tail+0x25c8/0x2630 [amdgpu]
    [205697.054273] myhost kernel: Code: ff ff 37 00 00 00 c7 85 24 fd ff ff 20 00 00 00 e8 9d fd 12 00 e9 05 fb ff ff 0f 0b e9 30 f9 ff ff 0f 0b 0f 0b e9 a0 f9 ff ff <0f> 0b e9 b9 f9 ff ff 49 8b 06 41 0f b6 8e 2d 01 00 00 48 c7 c6 48
    [205697.054276] myhost kernel: RSP: 0018:ffff9676c0fe7a28 EFLAGS: 00010086
    [205697.054278] myhost kernel: RAX: 0000000000000001 RBX: 0000000000000004 RCX: ffff8a1ccb58e118
    [205697.054280] myhost kernel: RDX: 0000000000000001 RSI: 0000000000000297 RDI: ffff8a1ccb480178
    [205697.054282] myhost kernel: RBP: ffff9676c0fe7dc0 R08: 0000000000000005 R09: 0000000000000000
    [205697.054284] myhost kernel: R10: ffff9676c0fe7988 R11: ffff9676c0fe798c R12: 0000000000000286
    [205697.054286] myhost kernel: R13: ffff8a1ccb58e000 R14: ffff8a1e9fb9c600 R15: ffff8a1d4b44a200
    [205697.054287] myhost kernel: FS:  0000000000000000(0000) GS:ffff8a1f70a00000(0000) knlGS:0000000000000000
    [205697.054290] myhost kernel: CS:  0010 DS: 0000 ES: 0000 CR0: 0000000080050033
    [205697.054292] myhost kernel: CR2: 00007f4c22177000 CR3: 000000017eb28000 CR4: 00000000003506f0
    [205697.054294] myhost kernel: Call Trace:
    [205697.054299] myhost kernel:  commit_tail+0x94/0x120 [drm_kms_helper]
    [205697.054299] myhost kernel:  drm_atomic_helper_commit+0x113/0x140 [drm_kms_helper]
    [205697.054299] myhost kernel:  dm_restore_drm_connector_state+0xef/0x170 [amdgpu]
    [205697.054299] myhost kernel:  handle_hpd_irq+0x118/0x150 [amdgpu]
    [205697.054299] myhost kernel:  process_one_work+0x214/0x3e0
    [205697.054299] myhost kernel:  worker_thread+0x4d/0x3d0
    [205697.054299] myhost kernel:  ? process_one_work+0x3e0/0x3e0
    [205697.054299] myhost kernel:  kthread+0x133/0x150
    [205697.054299] myhost kernel:  ? kthread_associate_blkcg+0xc0/0xc0
    [205697.054299] myhost kernel:  ret_from_fork+0x22/0x30
    [205697.054299] myhost kernel: ---[ end trace 752ed8c5d8483261 ]---
    ```

??? failure "AMDGPU crash logs with kernel 5.12.8"
    ```
    [14758.379934] myhost kernel: amdgpu 0000:04:00.0: amdgpu: [gfxhub0] retry page fault (src_id:0 ring:0 vmid:3 pasid:32769, for process Xorg pid 778 thread Xorg:cs0 pid 783)
    [14758.379942] myhost kernel: amdgpu 0000:04:00.0: amdgpu:   in page starting at address 0x8001023c0000 from client 27
    [14758.379948] myhost kernel: amdgpu 0000:04:00.0: amdgpu: VM_L2_PROTECTION_FAULT_STATUS:0x00341051
    [14758.379950] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          Faulty UTCL2 client ID: TCP (0x8)
    [14758.379952] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MORE_FAULTS: 0x1
    [14758.379953] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          WALKER_ERROR: 0x0
    [14758.379954] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          PERMISSION_FAULTS: 0x5
    [14758.379955] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MAPPING_ERROR: 0x0
    [14758.379956] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          RW: 0x1
    [14758.379960] myhost kernel: amdgpu 0000:04:00.0: amdgpu: [gfxhub0] retry page fault (src_id:0 ring:0 vmid:3 pasid:32769, for process Xorg pid 778 thread Xorg:cs0 pid 783)
    [14758.379963] myhost kernel: amdgpu 0000:04:00.0: amdgpu:   in page starting at address 0x8001023c1000 from client 27
    [14758.379968] myhost kernel: amdgpu 0000:04:00.0: amdgpu: VM_L2_PROTECTION_FAULT_STATUS:0x00341051
    [14758.379969] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          Faulty UTCL2 client ID: TCP (0x8)
    [14758.379970] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MORE_FAULTS: 0x1
    [14758.379971] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          WALKER_ERROR: 0x0
    [14758.379972] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          PERMISSION_FAULTS: 0x5
    [14758.379973] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MAPPING_ERROR: 0x0
    [14758.379974] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          RW: 0x1
    [14758.379976] myhost kernel: amdgpu 0000:04:00.0: amdgpu: [gfxhub0] retry page fault (src_id:0 ring:0 vmid:3 pasid:32769, for process Xorg pid 778 thread Xorg:cs0 pid 783)
    [14758.379979] myhost kernel: amdgpu 0000:04:00.0: amdgpu:   in page starting at address 0x8001023c4000 from client 27
    [14758.379984] myhost kernel: amdgpu 0000:04:00.0: amdgpu: VM_L2_PROTECTION_FAULT_STATUS:0x00341051
    [14758.379985] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          Faulty UTCL2 client ID: TCP (0x8)
    [14758.379987] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MORE_FAULTS: 0x1
    [14758.379988] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          WALKER_ERROR: 0x0
    [14758.379988] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          PERMISSION_FAULTS: 0x5
    [14758.379989] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MAPPING_ERROR: 0x0
    [14758.379990] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          RW: 0x1
    [14758.379992] myhost kernel: amdgpu 0000:04:00.0: amdgpu: [gfxhub0] retry page fault (src_id:0 ring:0 vmid:3 pasid:32769, for process Xorg pid 778 thread Xorg:cs0 pid 783)
    [14758.379995] myhost kernel: amdgpu 0000:04:00.0: amdgpu:   in page starting at address 0x8001023c6000 from client 27
    [14758.380000] myhost kernel: amdgpu 0000:04:00.0: amdgpu: VM_L2_PROTECTION_FAULT_STATUS:0x00341051
    [14758.380001] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          Faulty UTCL2 client ID: TCP (0x8)
    [14758.380002] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MORE_FAULTS: 0x1
    [14758.380003] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          WALKER_ERROR: 0x0
    [14758.380004] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          PERMISSION_FAULTS: 0x5
    [14758.380005] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MAPPING_ERROR: 0x0
    [14758.380005] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          RW: 0x1
    [14758.380008] myhost kernel: amdgpu 0000:04:00.0: amdgpu: [gfxhub0] retry page fault (src_id:0 ring:0 vmid:3 pasid:32769, for process Xorg pid 778 thread Xorg:cs0 pid 783)
    [14758.380010] myhost kernel: amdgpu 0000:04:00.0: amdgpu:   in page starting at address 0x8001023c5000 from client 27
    [14758.380015] myhost kernel: amdgpu 0000:04:00.0: amdgpu: VM_L2_PROTECTION_FAULT_STATUS:0x00341051
    [14758.380016] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          Faulty UTCL2 client ID: TCP (0x8)
    [14758.380017] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MORE_FAULTS: 0x1
    [14758.380018] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          WALKER_ERROR: 0x0
    [14758.380019] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          PERMISSION_FAULTS: 0x5
    [14758.380020] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MAPPING_ERROR: 0x0
    [14758.380021] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          RW: 0x1
    [14758.380023] myhost kernel: amdgpu 0000:04:00.0: amdgpu: [gfxhub0] retry page fault (src_id:0 ring:0 vmid:3 pasid:32769, for process Xorg pid 778 thread Xorg:cs0 pid 783)
    [14758.380026] myhost kernel: amdgpu 0000:04:00.0: amdgpu:   in page starting at address 0x8001023c7000 from client 27
    [14758.380032] myhost kernel: amdgpu 0000:04:00.0: amdgpu: VM_L2_PROTECTION_FAULT_STATUS:0x00341051
    [14758.380033] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          Faulty UTCL2 client ID: TCP (0x8)
    [14758.380034] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MORE_FAULTS: 0x1
    [14758.380035] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          WALKER_ERROR: 0x0
    [14758.380036] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          PERMISSION_FAULTS: 0x5
    [14758.380037] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MAPPING_ERROR: 0x0
    [14758.380038] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          RW: 0x1
    [14758.380040] myhost kernel: amdgpu 0000:04:00.0: amdgpu: [gfxhub0] retry page fault (src_id:0 ring:0 vmid:3 pasid:32769, for process Xorg pid 778 thread Xorg:cs0 pid 783)
    [14758.380042] myhost kernel: amdgpu 0000:04:00.0: amdgpu:   in page starting at address 0x8001023c3000 from client 27
    [14758.380047] myhost kernel: amdgpu 0000:04:00.0: amdgpu: VM_L2_PROTECTION_FAULT_STATUS:0x00341051
    [14758.380048] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          Faulty UTCL2 client ID: TCP (0x8)
    [14758.380049] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MORE_FAULTS: 0x1
    [14758.380050] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          WALKER_ERROR: 0x0
    [14758.380051] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          PERMISSION_FAULTS: 0x5
    [14758.380052] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MAPPING_ERROR: 0x0
    [14758.380052] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          RW: 0x1
    [14758.380055] myhost kernel: amdgpu 0000:04:00.0: amdgpu: [gfxhub0] retry page fault (src_id:0 ring:0 vmid:3 pasid:32769, for process Xorg pid 778 thread Xorg:cs0 pid 783)
    [14758.380057] myhost kernel: amdgpu 0000:04:00.0: amdgpu:   in page starting at address 0x8001023c2000 from client 27
    [14758.380062] myhost kernel: amdgpu 0000:04:00.0: amdgpu: VM_L2_PROTECTION_FAULT_STATUS:0x00341051
    [14758.380063] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          Faulty UTCL2 client ID: TCP (0x8)
    [14758.380064] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MORE_FAULTS: 0x1
    [14758.380065] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          WALKER_ERROR: 0x0
    [14758.380066] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          PERMISSION_FAULTS: 0x5
    [14758.380066] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MAPPING_ERROR: 0x0
    [14758.380067] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          RW: 0x1
    [14758.380070] myhost kernel: amdgpu 0000:04:00.0: amdgpu: [gfxhub0] retry page fault (src_id:0 ring:0 vmid:3 pasid:32769, for process Xorg pid 778 thread Xorg:cs0 pid 783)
    [14758.380072] myhost kernel: amdgpu 0000:04:00.0: amdgpu:   in page starting at address 0x8001023c8000 from client 27
    [14758.380077] myhost kernel: amdgpu 0000:04:00.0: amdgpu: VM_L2_PROTECTION_FAULT_STATUS:0x00341051
    [14758.380078] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          Faulty UTCL2 client ID: TCP (0x8)
    [14758.380079] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MORE_FAULTS: 0x1
    [14758.380080] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          WALKER_ERROR: 0x0
    [14758.380080] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          PERMISSION_FAULTS: 0x5
    [14758.380081] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MAPPING_ERROR: 0x0
    [14758.380082] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          RW: 0x1
    [14758.380084] myhost kernel: amdgpu 0000:04:00.0: amdgpu: [gfxhub0] retry page fault (src_id:0 ring:0 vmid:3 pasid:32769, for process Xorg pid 778 thread Xorg:cs0 pid 783)
    [14758.380086] myhost kernel: amdgpu 0000:04:00.0: amdgpu:   in page starting at address 0x8001023c9000 from client 27
    [14758.380091] myhost kernel: amdgpu 0000:04:00.0: amdgpu: VM_L2_PROTECTION_FAULT_STATUS:0x00341051
    [14758.380092] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          Faulty UTCL2 client ID: TCP (0x8)
    [14758.380093] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MORE_FAULTS: 0x1
    [14758.380094] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          WALKER_ERROR: 0x0
    [14758.380095] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          PERMISSION_FAULTS: 0x5
    [14758.380096] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          MAPPING_ERROR: 0x0
    [14758.380097] myhost kernel: amdgpu 0000:04:00.0: amdgpu:          RW: 0x1
    [14768.590863] myhost kernel: [drm:amdgpu_job_timedout [amdgpu]] *ERROR* ring gfx timeout, signaled seq=1415343, emitted seq=1415345
    [14768.591253] myhost kernel: [drm:amdgpu_job_timedout [amdgpu]] *ERROR* Process information: process Xorg pid 778 thread Xorg:cs0 pid 783
    [14768.593293] myhost kernel: amdgpu 0000:04:00.0: AMD-Vi: Event logged [IO_PAGE_FAULT domain=0x0000 address=0x10b4c0000 flags=0x0070]
    [14768.593322] myhost kernel: amdgpu 0000:04:00.0: AMD-Vi: Event logged [IO_PAGE_FAULT domain=0x0000 address=0x10b4a0100 flags=0x0070]
    [14768.593395] myhost kernel: amdgpu 0000:04:00.0: AMD-Vi: Event logged [IO_PAGE_FAULT domain=0x0000 address=0x10b4a01c0 flags=0x0070]
    [14768.593405] myhost kernel: AMD-Vi: Event logged [IO_PAGE_FAULT device=04:00.0 domain=0x0000 address=0x10b4a01e0 flags=0x0070]
    [14768.593448] myhost kernel: AMD-Vi: Event logged [IO_PAGE_FAULT device=04:00.0 domain=0x0000 address=0x10b4a0200 flags=0x0070]
    [14768.593532] myhost kernel: AMD-Vi: Event logged [IO_PAGE_FAULT device=04:00.0 domain=0x0000 address=0x10b4a02c0 flags=0x0070]
    [14768.790937] myhost kernel: [Hardware Error]: Deferred error, no action required.
    [14768.790939] myhost kernel: [Hardware Error]: CPU:0 (17:18:1) MC20_STATUS[-|-|-|AddrV|-|-|SyndV|UECC|Deferred|-|-]: 0x942030000001085b
    [14768.790950] myhost kernel: [Hardware Error]: Error Addr: 0x0000fffcffffff00
    [14768.790952] myhost kernel: [Hardware Error]: IPID: 0x0000002e00000000, Syndrome: 0x000000005b240203
    [14768.790957] myhost kernel: [Hardware Error]: Coherent Slave Ext. Error Code: 1, Address Violation.
    [14768.790960] myhost kernel: [Hardware Error]: cache level: L3/GEN, mem/io: IO, mem-tx: IRD, part-proc: SRC (no timeout)
    [14779.257530] myhost kernel: [drm:amdgpu_job_timedout [amdgpu]] *ERROR* ring gfx timeout, signaled seq=1415360, emitted seq=1415362
    [14779.257927] myhost kernel: [drm:amdgpu_job_timedout [amdgpu]] *ERROR* Process information: process Xorg pid 778 thread Xorg:cs0 pid 783
    [14779.908193] myhost kernel: amdgpu 0000:04:00.0: AMD-Vi: Event logged [IO_PAGE_FAULT domain=0x0000 address=0x10b480380 flags=0x0070]
    [14779.908311] myhost kernel: amdgpu 0000:04:00.0: AMD-Vi: Event logged [IO_PAGE_FAULT domain=0x0000 address=0x10b480460 flags=0x0070]
    [14779.908316] myhost kernel: amdgpu 0000:04:00.0: AMD-Vi: Event logged [IO_PAGE_FAULT domain=0x0000 address=0x10b4804c0 flags=0x0070]
    [14779.908328] myhost kernel: AMD-Vi: Event logged [IO_PAGE_FAULT device=04:00.0 domain=0x0000 address=0x10b4804e0 flags=0x0070]
    [14779.908359] myhost kernel: AMD-Vi: Event logged [IO_PAGE_FAULT device=04:00.0 domain=0x0000 address=0x10b4805c0 flags=0x0070]
    [14779.908364] myhost kernel: AMD-Vi: Event logged [IO_PAGE_FAULT device=04:00.0 domain=0x0000 address=0x10b4805a0 flags=0x0070]
    [14779.908284] myhost kernel: [drm:amdgpu_cs_ioctl [amdgpu]] *ERROR* Failed to initialize parser -125!
    [14786.658710] myhost kernel: [drm:amdgpu_cs_ioctl [amdgpu]] *ERROR* Failed to initialize parser -125!
    [14789.659420] myhost kernel: [drm:amdgpu_cs_ioctl [amdgpu]] *ERROR* Failed to initialize parser -125!
    [14789.914311] myhost kernel: [drm:amdgpu_job_timedout [amdgpu]] *ERROR* ring gfx timeout, but soft recovered
    [14789.932413] myhost kernel: [drm:amdgpu_cs_ioctl [amdgpu]] *ERROR* Failed to initialize parser -125!
    [14789.933420] myhost kernel: [drm:amdgpu_cs_ioctl [amdgpu]] *ERROR* Failed to initialize parser -125!
    [14789.934193] myhost kernel: [drm:amdgpu_cs_ioctl [amdgpu]] *ERROR* Failed to initialize parser -125!
    ```

In the end, it turned out to be an issue in the `linux-firmware` package. Downgrading from
`20210511.7685cf4-1` to `20210315.3568f96-2` fixed the issue but I don't think this counts as a
solution which is obvious considering that this issue hasn't been solved for other people on the
Arch Linux thread I linked above. I've basically ignored the `linux-firmware` package in
`/etc/pacman.conf` but how long do I keep ignoring it? Months? Years?

### Suspend and Resume

I've been using Linux on the desktop since 2015 but I can't recall the last time I suspended my
Intel Haswell desktop or my AMD Ryzen laptop with confidence that they'll wake up again. For
reference, I got my desktop in 2015 and my laptop in 2020.

Technically speaking, my system does come online but, sometimes, the screen remains turned off and
the CPU usage often spikes to 100%. Sometimes the screen does turn on but the system is frozen. I
have no option at that point but to reboot forcibly and lose any potential data I was working on.
When I asked for help on the Arch forums, [I was told to stop whining and compile my own
kernel](https://bbs.archlinux.org/viewtopic.php?pid=1962561#p1962561) with patches found on gitlab
which the developer himself called hacks.

Let's see how many years it takes for this bug to be fixed.

??? success "Successful Resume on kernel 5.10.23"
    ```
    host kernel: Freezing user space processes ... (elapsed 0.098 seconds) done.
    host kernel: OOM killer disabled.
    host kernel: Freezing remaining freezable tasks ... (elapsed 0.067 seconds) done.
    host kernel: printk: Suspending console(s) (use no_console_suspend to debug)
    host kernel: r8169 0000:02:00.0 enp2s0: Link is Down
    host kernel: [drm] free PSP TMR buffer
    host kernel: ACPI: EC: interrupt blocked
    host kernel: ACPI: Preparing to enter system sleep state S3
    host kernel: ACPI: EC: event blocked
    host kernel: ACPI: EC: EC stopped
    host kernel: PM: Saving platform NVS memory
    host kernel: Disabling non-boot CPUs ...
    host kernel: smpboot: CPU 1 is now offline
    host kernel: smpboot: CPU 2 is now offline
    host kernel: smpboot: CPU 3 is now offline
    host kernel: smpboot: CPU 4 is now offline
    host kernel: smpboot: CPU 5 is now offline
    host kernel: smpboot: CPU 6 is now offline
    host kernel: smpboot: CPU 7 is now offline
    host kernel: ACPI: Low-level resume complete
    host kernel: ACPI: EC: EC started
    host kernel: PM: Restoring platform NVS memory
    host kernel: Enabling non-boot CPUs ...
    host kernel: x86: Booting SMP configuration:
    host kernel: smpboot: Booting Node 0 Processor 1 APIC 0x1
    host kernel: microcode: CPU1: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C001: Found 2 idle states
    host kernel: CPU1 is up
    host kernel: smpboot: Booting Node 0 Processor 2 APIC 0x2
    host kernel: microcode: CPU2: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C002: Found 2 idle states
    host kernel: CPU2 is up
    host kernel: smpboot: Booting Node 0 Processor 3 APIC 0x3
    host kernel: microcode: CPU3: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C003: Found 2 idle states
    host kernel: CPU3 is up
    host kernel: smpboot: Booting Node 0 Processor 4 APIC 0x4
    host kernel: microcode: CPU4: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C004: Found 2 idle states
    host kernel: CPU4 is up
    host kernel: smpboot: Booting Node 0 Processor 5 APIC 0x5
    host kernel: microcode: CPU5: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C005: Found 2 idle states
    host kernel: CPU5 is up
    host kernel: smpboot: Booting Node 0 Processor 6 APIC 0x6
    host kernel: microcode: CPU6: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C006: Found 2 idle states
    host kernel: CPU6 is up
    host kernel: smpboot: Booting Node 0 Processor 7 APIC 0x7
    host kernel: microcode: CPU7: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C007: Found 2 idle states
    host kernel: CPU7 is up
    host kernel: ACPI: Waking up from system sleep state S3
    host kernel: ACPI: EC: interrupt unblocked
    host kernel: ACPI: EC: event unblocked
    host kernel: [drm] PCIE GART of 1024M enabled (table at 0x000000F400900000).
    host kernel: [drm] PSP is resuming...
    host kernel: [drm] reserve 0x400000 from 0xf47fc00000 for PSP TMR
    host kernel: nvme nvme0: 16/0/0 default/read/poll queues
    host kernel: amdgpu 0000:04:00.0: amdgpu: RAS: optional ras ta ucode is not available
    host kernel: amdgpu 0000:04:00.0: amdgpu: RAP: optional rap ta ucode is not available
    host kernel: [drm] kiq ring mec 2 pipe 1 q 0
    host kernel: amdgpu: dpm has been enabled
    host kernel: usb 1-3: reset full-speed USB device number 5 using xhci_hcd
    host kernel: [drm] VCN decode and encode initialized successfully(under SPG Mode).
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring gfx uses VM inv eng 0 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.0.0 uses VM inv eng 1 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.1.0 uses VM inv eng 4 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.2.0 uses VM inv eng 5 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.3.0 uses VM inv eng 6 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.0.1 uses VM inv eng 7 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.1.1 uses VM inv eng 8 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.2.1 uses VM inv eng 9 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.3.1 uses VM inv eng 10 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring kiq_2.1.0 uses VM inv eng 11 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring sdma0 uses VM inv eng 0 on hub 1
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring vcn_dec uses VM inv eng 1 on hub 1
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring vcn_enc0 uses VM inv eng 4 on hub 1
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring vcn_enc1 uses VM inv eng 5 on hub 1
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring jpeg_dec uses VM inv eng 6 on hub 1
    host kernel: psmouse serio1: synaptics: queried max coordinates: x [..5678], y [..4694]
    host kernel: OOM killer enabled.
    host kernel: Restarting tasks ...
    host kernel: pci_bus 0000:01: Allocating resources
    host kernel: pci_bus 0000:02: Allocating resources
    host kernel: pci_bus 0000:03: Allocating resources
    host kernel: pci_bus 0000:04: Allocating resources
    host kernel: done.
    host kernel: thermal thermal_zone0: failed to read out thermal zone (-61)
    host kernel: audit: type=1334 audit(1616172654.750:161): prog-id=34 op=LOAD
    host kernel: audit: type=1334 audit(1616172654.750:162): prog-id=35 op=LOAD
    host audit: BPF prog-id=34 op=LOAD
    host audit: BPF prog-id=35 op=LOAD
    host systemd-sleep[36158]: System resumed.
    host kernel: PM: suspend exit
    host systemd[1]: systemd-suspend.service: Succeeded.
    host systemd[1]: Finished Suspend.
    host audit[1]: SERVICE_START pid=1 uid=0 auid=4294967295 ses=4294967295 msg='unit=systemd-suspend comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
    host audit[1]: SERVICE_STOP pid=1 uid=0 auid=4294967295 ses=4294967295 msg='unit=systemd-suspend comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
    host systemd[1]: Stopped target Sleep.
    host systemd[1]: Reached target Suspend.
    host systemd[1]: Stopped target Suspend.
    host systemd-logind[599]: Operation 'sleep' finished.
    host kernel: psmouse serio1: synaptics: queried min coordinates: x [1266..], y [1162..]
    host kernel: usb 1-1: new high-speed USB device number 10 using xhci_hcd
    host kernel: usb 1-1: New USB device found, idVendor=0451, idProduct=xxxx, bcdDevice= 1.00
    host kernel: usb 1-1: New USB device strings: Mfr=0, Product=0, SerialNumber=1
    host kernel: usb 1-1: SerialNumber: BD0B00xxxxxx
    host kernel: hub 1-1:1.0: USB hub found
    host kernel: hub 1-1:1.0: 4 ports detected
    host kernel: usb 1-1.4: new full-speed USB device number 11 using xhci_hcd
    host kernel: usb 1-1.4: New USB device found, idVendor=043e, idProduct=xxxx, bcdDevice= 2.03
    host kernel: usb 1-1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    host kernel: usb 1-1.4: Product: USB Controls
    host kernel: usb 1-1.4: Manufacturer: LG Electronics Inc.
    host kernel: hid-generic 0003:043E:xxxx.000A: hiddev97,hidraw4: USB HID v1.11 Device [LG Electronics Inc. USB Controls] on usb-0000:04:00.3-1.4/input0
    host kernel: cdc_acm 1-1.4:1.1: ttyACM0: USB ACM device
    ```

??? failure "Unsuccessful Resume on kernel 5.10.23"
    ```
    host kernel: Freezing user space processes ... (elapsed 0.001 seconds) done.
    host kernel: OOM killer disabled.
    host kernel: Freezing remaining freezable tasks ... (elapsed 0.001 seconds) done.
    host kernel: printk: Suspending console(s) (use no_console_suspend to debug)
    host kernel: r8169 0000:02:00.0 enp2s0: Link is Down
    host kernel: ------------[ cut here ]------------
    host kernel: WARNING: CPU: 0 PID: 1000 at drivers/gpu/drm/amd/amdgpu/../display/dc/core/dc_link.c:2558 dc_link_set_backlight_level+0x8a/0xf0 [amdgpu]
    host kernel: Modules linked in: tpm_crb wireguard curve25519_x86_64 libchacha20poly1305 chacha_x86_64 poly1305_x86_64 libblake2s blake2s_x86_64 ip6_udp_tunnel udp_tunnel libcurve25519_generic libchacha libblake2s_generic lm92 xxhash_generic iwlmvm btrfs blake2b_generic snd_hda_codec_conexant xor edac_mce_amd snd_hda_codec_hdmi mac80211 snd_hda_codec_generic raid6_pq wmi_bmof kvm_amd libcrc32c snd_hda_intel libarc4 kvm snd_intel_dspcfg soundwire_intel soundwire_generic_allocation soundwire_cadence snd_hda_codec snd_hda_core nls_iso8859_1 snd_hwdep soundwire_bus vfat irqbypass fat iwlwifi rapl psmouse snd_soc_core pcspkr snd_compress k10temp sp5100_tco thinkpad_acpi snd_rn_pci_acp3x ac97_bus i2c_piix4 snd_pcm_dmaengine snd_pci_acp3x cfg80211 ledtrig_audio snd_pcm cdc_acm joydev mousedev r8169 snd_timer realtek rfkill mdio_devres libphy snd ucsi_acpi typec_ucsi soundcore typec tpm_tis wmi tpm_tis_core mac_hid pinctrl_amd video i2c_scmi acpi_cpufreq pkcs8_key_parser fuse bpf_preload ip_tables
    [  277.339389] host kernel:  x_tables ext4 crc32c_generic crc16 mbcache jbd2 usbhid dm_crypt cbc encrypted_keys dm_mod trusted tpm crct10dif_pclmul crc32_pclmul crc32c_intel ghash_clmulni_intel aesni_intel crypto_simd cryptd glue_helper serio_raw ccp xhci_pci xhci_pci_renesas rng_core amdgpu gpu_sched ttm i2c_algo_bit drm_kms_helper syscopyarea sysfillrect sysimgblt fb_sys_fops cec drm agpgart
    host kernel: CPU: 0 PID: 1000 Comm: systemd-sleep Tainted: G        W         5.10.23-1-lts #1
    host kernel: Hardware name: LENOVO 20NECTO1WW/20NECTO1WW, BIOS R11ET36W (1.16 ) 03/30/2020
    host kernel: RIP: 0010:dc_link_set_backlight_level+0x8a/0xf0 [amdgpu]
    host kernel: Code: 70 03 00 00 31 c0 48 8d 96 c0 01 00 00 48 8b 0a 48 85 c9 74 06 48 3b 59 08 74 20 83 c0 01 48 81 c2 d8 04 00 00 83 f8 06 75 e3 <0f> 0b 45 31 e4 5b 44 89 e0 5d 41 5c 41 5d 41 5e c3 48 98 48 69 c0
    host kernel: RSP: 0018:ffffaee1c3c23c38 EFLAGS: 00010246
    host kernel: RAX: 0000000000000006 RBX: ffff8d8d8ac9c400 RCX: 0000000000000000
    host kernel: RDX: ffff8d8dd6741ed0 RSI: ffff8d8dd6740000 RDI: 0000000000000000
    host kernel: RBP: ffff8d8d8aee0000 R08: 000000000000004e R09: ffff8d8da2485d80
    host kernel: R10: ffff8d8d80043a00 R11: 0000000000000000 R12: 0000000000005001
    host kernel: R13: 0000000000000000 R14: 0000000000005065 R15: 0000000000000003
    host kernel: FS:  00007f214af4da40(0000) GS:ffff8d9030a00000(0000) knlGS:0000000000000000
    host kernel: CS:  0010 DS: 0000 ES: 0000 CR0: 0000000080050033
    host kernel: CR2: 000055d9351cbda0 CR3: 0000000153afa000 CR4: 00000000003506f0
    host kernel: Call Trace:
    host kernel:  amdgpu_dm_backlight_update_status+0xb4/0xc0 [amdgpu]
    host kernel:  backlight_suspend+0x6a/0x80
    host kernel:  ? brightness_store+0x80/0x80
    host kernel:  dpm_run_callback+0x4c/0x120
    host kernel:  __device_suspend+0x11c/0x480
    host kernel:  dpm_suspend+0xef/0x210
    host kernel:  dpm_suspend_start+0x77/0x80
    host kernel:  suspend_devices_and_enter+0x109/0x780
    host kernel:  pm_suspend.cold+0x329/0x374
    host kernel:  state_store+0x71/0xd0
    host kernel:  kernfs_fop_write_iter+0x124/0x1b0
    host kernel:  new_sync_write+0x159/0x1f0
    host kernel:  vfs_write+0x1b5/0x280
    host kernel:  ksys_write+0x67/0xe0
    host kernel:  do_syscall_64+0x33/0x40
    host kernel:  entry_SYSCALL_64_after_hwframe+0x44/0xa9
    host kernel: RIP: 0033:0x7f214b8be0f7
    host kernel: Code: 0d 00 f7 d8 64 89 02 48 c7 c0 ff ff ff ff eb b7 0f 1f 00 f3 0f 1e fa 64 8b 04 25 18 00 00 00 85 c0 75 10 b8 01 00 00 00 0f 05 <48> 3d 00 f0 ff ff 77 51 c3 48 83 ec 28 48 89 54 24 18 48 89 74 24
    host kernel: RSP: 002b:00007ffea6f8d168 EFLAGS: 00000246 ORIG_RAX: 0000000000000001
    host kernel: RAX: ffffffffffffffda RBX: 0000000000000004 RCX: 00007f214b8be0f7
    host kernel: RDX: 0000000000000004 RSI: 00007ffea6f8d250 RDI: 0000000000000004
    host kernel: RBP: 00007ffea6f8d250 R08: 000055e7bb8efb70 R09: 00007f214b9540c0
    host kernel: R10: 00007f214b953fc0 R11: 0000000000000246 R12: 0000000000000004
    host kernel: R13: 000055e7bb8eb3c0 R14: 0000000000000004 R15: 00007f214b990700
    host kernel: ---[ end trace 5402e2e3d5c42003 ]---
    host kernel: [drm] free PSP TMR buffer
    host kernel: ACPI: EC: interrupt blocked
    host kernel: ACPI: Preparing to enter system sleep state S3
    host kernel: ACPI: EC: event blocked
    host kernel: ACPI: EC: EC stopped
    host kernel: PM: Saving platform NVS memory
    host kernel: Disabling non-boot CPUs ...
    host kernel: IRQ 76: no longer affine to CPU1
    host kernel: smpboot: CPU 1 is now offline
    host kernel: IRQ 77: no longer affine to CPU2
    host kernel: smpboot: CPU 2 is now offline
    host kernel: IRQ 78: no longer affine to CPU3
    host kernel: smpboot: CPU 3 is now offline
    host kernel: smpboot: CPU 4 is now offline
    host kernel: smpboot: CPU 5 is now offline
    host kernel: smpboot: CPU 6 is now offline
    host kernel: smpboot: CPU 7 is now offline
    host kernel: ACPI: Low-level resume complete
    host kernel: ACPI: EC: EC started
    host kernel: PM: Restoring platform NVS memory
    host kernel: Enabling non-boot CPUs ...
    host kernel: x86: Booting SMP configuration:
    host kernel: smpboot: Booting Node 0 Processor 1 APIC 0x1
    host kernel: microcode: CPU1: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C001: Found 2 idle states
    host kernel: CPU1 is up
    host kernel: smpboot: Booting Node 0 Processor 2 APIC 0x2
    host kernel: microcode: CPU2: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C002: Found 2 idle states
    host kernel: CPU2 is up
    host kernel: smpboot: Booting Node 0 Processor 3 APIC 0x3
    host kernel: microcode: CPU3: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C003: Found 2 idle states
    host kernel: CPU3 is up
    host kernel: smpboot: Booting Node 0 Processor 4 APIC 0x4
    host kernel: microcode: CPU4: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C004: Found 2 idle states
    host kernel: CPU4 is up
    host kernel: smpboot: Booting Node 0 Processor 5 APIC 0x5
    host kernel: microcode: CPU5: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C005: Found 2 idle states
    host kernel: CPU5 is up
    host kernel: smpboot: Booting Node 0 Processor 6 APIC 0x6
    host kernel: microcode: CPU6: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C006: Found 2 idle states
    host kernel: CPU6 is up
    host kernel: smpboot: Booting Node 0 Processor 7 APIC 0x7
    host kernel: microcode: CPU7: patch_level=0x08108102
    host kernel: ACPI: \_PR_.C007: Found 2 idle states
    host kernel: CPU7 is up
    host kernel: ACPI: Waking up from system sleep state S3
    host kernel: ACPI: EC: interrupt unblocked
    host kernel: ACPI: EC: event unblocked
    host kernel: [drm] PCIE GART of 1024M enabled (table at 0x000000F400900000).
    host kernel: [drm] PSP is resuming...
    host kernel: [drm] reserve 0x400000 from 0xf47fc00000 for PSP TMR
    host kernel: nvme nvme0: 16/0/0 default/read/poll queues
    host kernel: r8169 0000:02:00.0 enp2s0: Link is Down
    host kernel: amdgpu 0000:04:00.0: amdgpu: RAS: optional ras ta ucode is not available
    host kernel: amdgpu 0000:04:00.0: amdgpu: RAP: optional rap ta ucode is not available
    host kernel: [drm] kiq ring mec 2 pipe 1 q 0
    host kernel: amdgpu: dpm has been enabled
    host kernel: usb 1-3: reset full-speed USB device number 5 using xhci_hcd
    host kernel: [drm] VCN decode and encode initialized successfully(under SPG Mode).
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring gfx uses VM inv eng 0 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.0.0 uses VM inv eng 1 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.1.0 uses VM inv eng 4 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.2.0 uses VM inv eng 5 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.3.0 uses VM inv eng 6 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.0.1 uses VM inv eng 7 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.1.1 uses VM inv eng 8 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.2.1 uses VM inv eng 9 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring comp_1.3.1 uses VM inv eng 10 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring kiq_2.1.0 uses VM inv eng 11 on hub 0
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring sdma0 uses VM inv eng 0 on hub 1
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring vcn_dec uses VM inv eng 1 on hub 1
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring vcn_enc0 uses VM inv eng 4 on hub 1
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring vcn_enc1 uses VM inv eng 5 on hub 1
    host kernel: amdgpu 0000:04:00.0: amdgpu: ring jpeg_dec uses VM inv eng 6 on hub 1
    host kernel: [drm] Fence fallback timer expired on ring sdma0
    host kernel: amdgpu 0000:04:00.0: AMD-Vi: Event logged [IO_PAGE_FAULT domain=0x0000 address=0x10b000040 flags=0x0070]
    host kernel: amdgpu 0000:04:00.0: AMD-Vi: Event logged [IO_PAGE_FAULT domain=0x0000 address=0x10b040000 flags=0x0070]
    host kernel: psmouse serio1: synaptics: queried max coordinates: x [..5678], y [..4694]
    host kernel: psmouse serio1: synaptics: queried min coordinates: x [1266..], y [1162..]
    host kernel: hub 1-1:1.0: hub_ext_port_status failed (err = -71)
    host kernel: usb 1-1-port4: cannot disable (err = -71)
    host kernel: PM: dpm_run_callback(): usb_dev_resume+0x0/0x10 returns -71
    host kernel: PM: Device 1-1.4 failed to resume async: error -71
    host kernel: OOM killer enabled.
    host kernel: Restarting tasks ...
    host kernel: pci_bus 0000:01: Allocating resources
    host kernel: pcieport 0000:00:01.1: bridge window [io  0x1000-0x0fff] to [bus 01] add_size 1000
    host kernel: pcieport 0000:00:01.1: bridge window [mem 0x00100000-0x000fffff 64bit pref] to [bus 01] add_size 200000 add_align 100000
    host kernel: pci_bus 0000:02: Allocating resources
    host kernel: pcieport 0000:00:01.2: bridge window [mem 0x00100000-0x000fffff 64bit pref] to [bus 02] add_size 200000 add_align 100000
    host kernel: pci_bus 0000:03: Allocating resources
    host kernel: usb 1-1: USB disconnect, device number 2
    host kernel: usb 1-1.4: USB disconnect, device number 4
    host kernel: pcieport 0000:00:01.1: BAR 15: assigned [mem 0xd0b00000-0xd0cfffff 64bit pref]
    host kernel: pcieport 0000:00:01.2: BAR 15: assigned [mem 0xd0d00000-0xd0efffff 64bit pref]
    host kernel: pcieport 0000:00:01.1: BAR 13: assigned [io  0x4000-0x4fff]
    host kernel: done.
    host kernel: thermal thermal_zone0: failed to read out thermal zone (-61)
    host kernel: pci_bus 0000:04: Allocating resources
    host kernel: audit: type=1334 audit(1615990449.569:60): prog-id=20 op=LOAD
    host kernel: audit: type=1334 audit(1615990449.569:61): prog-id=21 op=LOAD
    host audit: BPF prog-id=20 op=LOAD
    host audit: BPF prog-id=21 op=LOAD
    host kernel: [drm] amdgpu_dm_irq_schedule_work FAILED src 4
    host systemd-timesyncd[581]: Initial synchronization to time server 139.59.55.93:123 (3.arch.pool.ntp.org).
    host kernel: [drm:drm_atomic_helper_wait_for_flip_done [drm_kms_helper]] *ERROR* [CRTC:65:crtc-1] flip_done timed out
    host kernel: [drm:amdgpu_dm_atomic_check [amdgpu]] *ERROR* [CRTC:65:crtc-1] hw_done or flip_done timed out
    host kernel: [drm:drm_atomic_helper_wait_for_dependencies [drm_kms_helper]] *ERROR* [CRTC:65:crtc-1] flip_done timed out
    host systemd-logind[588]: Suspending...
    host kernel: Lockdown: systemd-logind: hibernation is restricted; see man kernel_lockdown.7
    host systemd[1]: Reached target Sleep.
    host systemd[1]: Starting Suspend...
    host systemd-sleep[1094]: Suspending system...
    host kernel: PM: suspend entry (deep)
    host kernel: Filesystems sync: 0.010 seconds
    host kernel: Freezing user space processes ...
    host kernel: [drm:drm_atomic_helper_wait_for_dependencies [drm_kms_helper]] *ERROR* [CONNECTOR:89:DP-2] flip_done timed out
    host kernel: [drm:drm_atomic_helper_wait_for_dependencies [drm_kms_helper]] *ERROR* [PLANE:48:plane-2] flip_done timed out
    host kernel:
    host kernel: Freezing of tasks failed after 20.009 seconds (1 tasks refusing to freeze, wq_busy=0):
    host kernel: task:sway            state:D stack:    0 pid:  672 ppid:   594 flags:0x00004084
    host kernel: Call Trace:
    host kernel:  __schedule+0x292/0x7f0
    host kernel:  ? usleep_range+0x90/0x90
    host kernel:  schedule+0x46/0xb0
    host kernel:  schedule_timeout+0x98/0x140
    host kernel:  ? __next_timer_interrupt+0x100/0x100
    host kernel:  __wait_for_common+0xc4/0x170
    host kernel:  drm_atomic_helper_wait_for_dependencies+0x16f/0x1f0 [drm_kms_helper]
    host kernel:  commit_tail+0x37/0x130 [drm_kms_helper]
    host kernel:  drm_atomic_helper_commit+0x113/0x140 [drm_kms_helper]
    host kernel:  drm_mode_atomic_ioctl+0x8f8/0x9e0 [drm]
    host kernel:  ? drm_atomic_set_property+0xae0/0xae0 [drm]
    host kernel:  drm_ioctl_kernel+0xb2/0x100 [drm]
    host kernel:  drm_ioctl+0x215/0x390 [drm]
    host kernel:  ? drm_atomic_set_property+0xae0/0xae0 [drm]
    host kernel:  amdgpu_drm_ioctl+0x49/0x80 [amdgpu]
    host kernel:  __x64_sys_ioctl+0x83/0xb0
    host kernel:  do_syscall_64+0x33/0x40
    host kernel:  entry_SYSCALL_64_after_hwframe+0x44/0xa9
    host kernel: RIP: 0033:0x7fbd05921e6b
    host kernel: RSP: 002b:00007fff61b147d8 EFLAGS: 00000246 ORIG_RAX: 0000000000000010
    host kernel: RAX: ffffffffffffffda RBX: 00007fff61b14820 RCX: 00007fbd05921e6b
    host kernel: RDX: 00007fff61b14820 RSI: 00000000c03864bc RDI: 0000000000000009
    host kernel: RBP: 00000000c03864bc R08: 0000000000000003 R09: 0000000000000003
    host kernel: R10: 000055cc559146c0 R11: 0000000000000246 R12: 000055cc559178b0
    host kernel: R13: 0000000000000009 R14: 000055cc558c47a0 R15: 000055cc559146a0
    host kernel:
    host kernel: OOM killer enabled.
    host kernel: Restarting tasks ... done.
    host kernel: thermal thermal_zone0: failed to read out thermal zone (-61)
    ```

I don't know but this looks like a AMDGPU issue to me. People rave about how Nvidia is bad for
Linux and how AMD is good because they have open source drivers and how everything works
"perfectly" for them but apparently, basic functionalities like suspend and resume is broken and
your system will keep crashing randomly in the middle of work.

Should I restrict my purchases to Intel only in the future? I asked around and people said that
the same problem exists for them on their Intel laptops for as long as they can remember.

## (Neo)Vim

### Spell Check

(Neo)Vim uses its own ancient spelling dictionaries and, apparently, you can't use hunspell or
nuspell with it.

Someone raised a [PR](https://github.com/vim/vim/pull/2500) about replacing apparently the second
biggest source code file in Vim, known as `spell.c`, but the PR was WIP and wasn't merged due to
[lack of interest](https://github.com/neovim/neovim/issues/12064#issuecomment-625729395).

### Syntax Highlighting and Code Folding

(Neo)Vim can't do syntax highlighting correctly, even with plugins, for a lot of things including
gitconfig files and markdown documents.

The markdown syntax highlighting library from
[plasticboy](https://github.com/plasticboy/vim-markdown) is abandoned and full of bugs. I tried
using [vim-pandoc-syntax](https://github.com/vim-pandoc/vim-pandoc-syntax) and that turned out to be
similarly buggy and extremely slow.

Code folding is basically broken in markdown on (Neo)Vim.

Here's a preview of broken code folding when using `vim-pandoc-syntax`.

[![asciicast](https://asciinema.org/a/uMlSQOG67IxQFyTHx1zHSTzrK.svg)](https://asciinema.org/a/uMlSQOG67IxQFyTHx1zHSTzrK)

Here's a preview of broken code folding when using `vim-markdown`.

[![asciicast](https://asciinema.org/a/51LqtMV3ZNl64EGxI2Q5eJFSL.svg)](https://asciinema.org/a/51LqtMV3ZNl64EGxI2Q5eJFSL)

When people rave about (Neo)Vim, they should mention that if you want to do common stuff like
writing documentation using markdown, you might be disappointed.

It looks like experimental support for [tree-sitter](https://tree-sitter.github.io/tree-sitter/) has
landed in NeoVim version 0.5.0. Since the support is experimental, I haven't used it yet and I'm not
going to unless it becomes stable, no matter how many Reddit comments hold collective circlejerk
rituals and chant *it works like a charm*.

## Firefox

### Invisible Pop Up Menus in Wayland

This issue was my breaking point and why I finally dropped the idea of using Wayland.

Pop up menus in Firefox [weren’t](https://github.com/swaywm/sway/issues/6147) visible after I right
clicked on something. This was incredibly annoying. Sometimes I end up clicking something which I
didn’t intend to.

# List of Issues Fixed After Distress

## Firefox

### Flickering Pop Up Menus in Wayland

When I upgraded to Firefox 87, I experienced [intense
flickering](https://bugzilla.mozilla.org/show_bug.cgi?id=1694967) on pop menus. Although this issue
was fixed in sway version 1.6, I had to downgrade and keep Firefox at version 86 for almost a month
after version 87 was released.

### Blurry Firefox in Wayland

With fractional scaling enabled and when waking up my laptop from sleep, Firefox almost always
became blurry. Someone on Reddit told me that this was because of
[this](https://github.com/swaywm/wlroots/issues/2466) bug.

What would you prefer after coming back home from the office?

- opening Firefox and browsing the web like a normal person
- saving your Firefox session and restarting Firefox and logging into each and every website again
  because your Firefox is blurry

This issue was fixed in sway 1.6 but I had to deal with this issue for months.

# Stupidity on Linux

## GNOME

Ah, GNOME, the most popular desktop environment on the Linux desktop. Surely it must be good right?

Before I begin, here's a quick short story. I used KDE for the first time in April 2021. Before I
used its screenshot tool, [spectacle](https://apps.kde.org/spectacle/), I thought that there were no
decent screenshot tools on Linux, except maybe
[flameshot](https://github.com/flameshot-org/flameshot), which isn't a GNOME software. At the time,
I was using Wayland and had resorted to using
[grimshot](https://github.com/swaywm/sway/blob/master/contrib/grimshot) and
[swappy](https://github.com/jtheoof/swappy). This wasn't a pleasant experience, to say the least.
However, all of this was much better than using [GNOME
Screenshot](https://gitlab.gnome.org/GNOME/gnome-screenshot.git). The next time you're confused
about whether to use KDE or GNOME, try installing Spectacle and GNOME Screenshot and see which of
them you like.

Speaking politely, GNOME is an extremely opinionated project. If your views on user interface and
user experience align with GNOME developers, you might like GNOME. If not, it would be better for
you to stay away from any GTK desktop environment or project or library out there. Unfortunately, if
you're using Linux on desktop, that isn't really possible.

Speaking frankly, for me, GNOME is a project for retards by retards.

[Here's](https://blogs.gnome.org/tbernard/2021/07/13/community-power-4/) a recent blog post by a
GNOME designer to get you started. Let's keep in mind that the title of the blog post contains the
words *community power*.

- Apparently, one of the hallmarks and major benefits of using Linux, package management, should be
  killed in favor of developers distributing their own binaries.

- System wide theming should be killed in favor of app developers making their own themes.

- GNOME Shell Extensions are a niche thing which GNOME doesn't give a shit about even if its users
  do.

- Flatpak is the future of app distribution, even if fonts and themes in Flatpak apps are broken out
  of the box. But hey, who knows, maybe custom fonts should be dead too.

Nice huh? Let's move on.

[The](https://bugzilla.gnome.org/show_bug.cgi?id=728585) comments by Allan Day, a GNOME UX designer
and member of its board of directors, on this bugzilla thread are probably one of the most stupid
things I've read in my life.

> I like the absence of a volume control in Music.

> Maybe the other apps should maybe be consistent with Music.

> I haven't heard a compelling reason why a volume control is necessary.

How do you react to something like this? This is how the guy who opened the bug report reacted.

> Ok, I give up :)

Moving on.

[This](https://bugzilla.gnome.org/show_bug.cgi?id=141154) issue should be the highlight of GNOME
desktop. It's the epitome of how stupid, arrogant, and insane a group of people can be.
[Here's](https://jayfax.neocities.org/mediocrity/gnome-has-no-thumbnails-in-the-file-picker.html) a
nice blog post about it and a corresponding [HN
thread](https://news.ycombinator.com/item?id=25719796). There's even
[memes](https://wiki.installgentoo.com/wiki/File_Picker_meme) about it.

Basically, if you have folder with 100 images and you want to pick one for uploading it somewhere on
the web, you're gonna have to remember the file name of the image you wanna pick or find the file
and drag and drop it in the browser.

This issue hasn't been fixed since 2004 and I doubt it ever will be.

If you were thinking about using Linux on the desktop, do yourself a favor and avoid distributions
which ship GNOME or any other GTK based desktop environment as their default. This includes MATE,
Cinnamon, XFCE, LXDE, Budgie, Pantheon. All of them suffer from the fatal flaw of using GTK and
being dependent on the whims of GNOME designers and developers. Better yet, stick to Windows or
MacOS.

[^1]: VSCode, and all apps based on Electron, didn't work on Wayland as expected until Electron 12
  was released on March 2nd, 2021. There were (are?) still lingering issues even after the release.
