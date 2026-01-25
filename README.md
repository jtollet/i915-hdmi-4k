# i915 HDMI 4K

This repo tracks all known patch versions produced during the HDMI 4K@60Hz
investigation and fix work, plus the supporting documentation and links.

## Problem summary
Some HDMI 2.0 monitors fail to decode 4K@60Hz when SCDC scrambling is configured
too quickly. The sink may drop sync during the SCDC transition even when I2C
transactions succeed, leading to a persistent "format detection" error.

## Solution summary
Multiple patch variants were produced while validating the root cause and
converging on a minimal fix:
- Fixed delays before/after SCDC configuration and DDI enable
- Polling TMDS_Scrambler_Status for up to 200 ms (HDMI 2.0 spec)
- A focused SCDC processing delay immediately after sink scrambling config

See the patch index and docs for full details and test results.

## Patch list (by date)
| Date | Patch | Comment |
|---|---|---|
| 2025-12-30 10:10 | [patches/home-i915/0001-drm-i915-hdmi-Fix-4K-60Hz-HDMI-display-with-SCDC-timing.patch](./patches/home-i915/0001-drm-i915-hdmi-Fix-4K-60Hz-HDMI-display-with-SCDC-timing.patch) | Fixed SCDC timing delays (100ms/150ms) |
| 2026-01-07 10:18 | [patches/local-root/0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS-Scrambler-.patch](./patches/local-root/0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS-Scrambler-.patch) | Poll SCDC status (200 ms) |
| 2026-01-08 09:54 | [patches/remote-linux/0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS_Scrambler_S.patch](./patches/remote-linux/0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS_Scrambler_S.patch) | Poll SCDC status (200 ms, linux-src) |
| 2026-01-25 12:57 | [patches/remote-linux/0002-drm-i915-hdmi-Debug-for-scrambling-polling.patch](./patches/remote-linux/0002-drm-i915-hdmi-Debug-for-scrambling-polling.patch) | Debug for scrambling polling |
| 2026-01-25 12:57 | [patches/remote/test1_delay_before_buf_enable.patch](./patches/remote/test1_delay_before_buf_enable.patch) | Test: delay before buf enable |
| 2026-01-25 12:57 | [patches/remote/test2_delay_before_power_up_lanes.patch](./patches/remote/test2_delay_before_power_up_lanes.patch) | Test: delay before power up lanes |
| 2026-01-25 12:57 | [patches/remote/test3_delay_after_signal_levels.patch](./patches/remote/test3_delay_after_signal_levels.patch) | Test: delay after signal levels |
| 2026-01-25 12:57 | [patches/remote/test4_delay_after_buffer_prep.patch](./patches/remote/test4_delay_after_buffer_prep.patch) | Test: delay after buffer prep |
| 2026-01-25 12:57 | [patches/remote/v2-0001-drm-i915-hdmi-Add-SCDC-processing-delay-for-scramb.patch](./patches/remote/v2-0001-drm-i915-hdmi-Add-SCDC-processing-delay-for-scramb.patch) | Delay after SCDC config (v2) |

## Repo layout
- patches/ : all patch versions grouped by source
- docs/    : local and remote docs, plus references and mail links

## Indexes
- docs/PATCHES_INDEX.md
- docs/DOCS_INDEX.md
- docs/PATCH_SIZES.md
- docs/MAILS.md

## References
- https://gitlab.freedesktop.org/drm/xe/kernel/-/issues/6868
- https://patchwork.freedesktop.org/series/160028/
- https://lore.kernel.org/dri-devel/20251230091037.5603-1-jerome.tollet@gmail.com/
- https://mail-archive.com/dri-devel@lists.freedesktop.org/msg584154.html
