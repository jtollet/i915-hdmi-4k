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
| 2025-01-13 17:40 | [patches/test1_delay_before_buf_enable.patch](./patches/test1_delay_before_buf_enable.patch) | Test placement: delay before buf enable; validates timing window |
| 2025-01-13 17:45 | [patches/test2_delay_before_power_up_lanes.patch](./patches/test2_delay_before_power_up_lanes.patch) | Test placement: delay before lane power-up; validates timing window |
| 2025-01-13 17:50 | [patches/test3_delay_after_signal_levels.patch](./patches/test3_delay_after_signal_levels.patch) | Test placement: delay after signal levels; validates timing window |
| 2025-01-13 17:55 | [patches/test4_delay_after_buffer_prep.patch](./patches/test4_delay_after_buffer_prep.patch) | Test placement: delay after buffer prep; validates timing window |
| 2025-12-30 09:00 | [patches/0001-drm-i915-hdmi-Fix-4K-60Hz-HDMI-display-with-SCDC-timing.patch](./patches/0001-drm-i915-hdmi-Fix-4K-60Hz-HDMI-display-with-SCDC-timing.patch) | Fixed delays: 100ms before SCDC + 150ms after DDI; no code restructuring |
| 2026-01-06 14:37 | [patches/0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS_Scrambler_S.patch](./patches/0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS_Scrambler_S.patch) | Poll SCDC status (<=200ms) after HDMI enable; adds helper + prototype |
| 2026-01-06 18:47 | [patches/0002-drm-i915-hdmi-Debug-for-scrambling-polling.patch](./patches/0002-drm-i915-hdmi-Debug-for-scrambling-polling.patch) | Adds debug around SCDC polling; instrumentation only |
| 2026-01-14 08:00 | [patches/v2-0001-drm-i915-hdmi-Add-SCDC-processing-delay-for-scramb.patch](./patches/v2-0001-drm-i915-hdmi-Add-SCDC-processing-delay-for-scramb.patch) | 150ms delay immediately after SCDC config; v2 rationale + tests |

## Repo layout
- patches/ : all patch versions (consolidated)
- docs/    : docs and reference links

## Indexes
- docs/MAILS.md

## References
- https://gitlab.freedesktop.org/drm/xe/kernel/-/issues/6868
- https://patchwork.freedesktop.org/series/160028/
- https://lore.kernel.org/dri-devel/20251230091037.5603-1-jerome.tollet@gmail.com/
- https://mail-archive.com/dri-devel@lists.freedesktop.org/msg584154.html
