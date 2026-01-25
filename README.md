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
| 2025-01-13 17:40 | [patches/remote/test1_delay_before_buf_enable.patch](./patches/remote/test1_delay_before_buf_enable.patch) | Test: delay inserted before buf enable (timing window probe) |
| 2025-01-13 17:45 | [patches/remote/test2_delay_before_power_up_lanes.patch](./patches/remote/test2_delay_before_power_up_lanes.patch) | Test: delay inserted before power up lanes (timing window probe) |
| 2025-01-13 17:50 | [patches/remote/test3_delay_after_signal_levels.patch](./patches/remote/test3_delay_after_signal_levels.patch) | Test: delay inserted after signal levels (timing window probe) |
| 2025-01-13 17:55 | [patches/remote/test4_delay_after_buffer_prep.patch](./patches/remote/test4_delay_after_buffer_prep.patch) | Test: delay inserted after buffer prep (timing window probe) |
| 2025-12-30 09:00 | [patches/home-i915/0001-drm-i915-hdmi-Fix-4K-60Hz-HDMI-display-with-SCDC-timing.patch](./patches/home-i915/0001-drm-i915-hdmi-Fix-4K-60Hz-HDMI-display-with-SCDC-timing.patch) | Add 100ms pre-SCDC + 150ms post-DDI delays; stable 4K@60Hz init |
| 2026-01-06 14:37 | [patches/remote-linux/0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS_Scrambler_S.patch](./patches/remote-linux/0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS_Scrambler_S.patch) | Poll SCDC status for up to 200ms; linux-src variant with full mail header |
| 2026-01-06 18:47 | [patches/remote-linux/0002-drm-i915-hdmi-Debug-for-scrambling-polling.patch](./patches/remote-linux/0002-drm-i915-hdmi-Debug-for-scrambling-polling.patch) | Extra debug around scrambling polling (instrumentation) |
| 2026-01-14 08:00 | [patches/remote/v2-0001-drm-i915-hdmi-Add-SCDC-processing-delay-for-scramb.patch](./patches/remote/v2-0001-drm-i915-hdmi-Add-SCDC-processing-delay-for-scramb.patch) | 150ms delay right after SCDC config; clarified rationale in v2 |

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
