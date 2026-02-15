# i915 HDMI 4K

This repo tracks all known patch versions produced during the HDMI 4K@60Hz
investigation and fix work, plus the supporting documentation and links.

## Problem summary
Some HDMI 2.0 monitors fail to decode 4K@60Hz when SCDC scrambling is configured
too quickly. The sink may drop sync during the SCDC transition even when I2C
transactions succeed, leading to a persistent "format detection" error.

## Solution summary (UPDATED Feb 14, 2026)

**RECOMMENDED SOLUTION: SCDC Polling Patch (Ankit Nautiyal)**

Isolated testing on kernel 6.18.7 confirms:
- ✅ **SCDC polling patch ALONE**: Works perfectly
- ⚠️ **Delay patch (150ms)**: Works when properly placed, but placement-dependent
- ⚠️ **Both patches together**: Works (but delay becomes redundant with polling)

**Conclusion**: The generic SCDC polling approach (polling TMDS_Scrambler_Status
for up to 200ms after port enable) is the preferred solution because it:
- Aligns with HDMI 2.0 specification requirements
- Matches Windows driver behavior
- Provides a generic solution for all affected monitors
- Is not placement-dependent like the fixed delay approach

The fixed delay approach (v2 patch) can work but is less robust than polling.

See the patch index and docs for full details and test results.

## Patch list (by date)
| # | Date | Patch | Comment |
|---:|---|---|---|
| 1 | 2025-01-13 17:40 | [patches/test1_delay_before_buf_enable.patch](./patches/test1_delay_before_buf_enable.patch) | Test placement: delay before buf enable; validates timing window |
| 2 | 2025-01-13 17:45 | [patches/test2_delay_before_power_up_lanes.patch](./patches/test2_delay_before_power_up_lanes.patch) | Test placement: delay before lane power-up; validates timing window |
| 3 | 2025-01-13 17:50 | [patches/test3_delay_after_signal_levels.patch](./patches/test3_delay_after_signal_levels.patch) | Test placement: delay after signal levels; validates timing window |
| 4 | 2025-01-13 17:55 | [patches/test4_delay_after_buffer_prep.patch](./patches/test4_delay_after_buffer_prep.patch) | Test placement: delay after buffer prep; validates timing window |
| 5 | 2025-12-30 09:00 | [patches/0001-drm-i915-hdmi-Fix-4K-60Hz-HDMI-display-with-SCDC-timing.patch](./patches/0001-drm-i915-hdmi-Fix-4K-60Hz-HDMI-display-with-SCDC-timing.patch) | Fixed delays: 100ms before SCDC + 150ms after DDI; no code restructuring |
| 6 | 2026-01-06 14:37 | [patches/0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS_Scrambler_S.patch](./patches/0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS_Scrambler_S.patch) | Poll SCDC status (<=200ms) after HDMI enable; adds helper + prototype |
| 7 | 2026-01-06 18:47 | [patches/0002-drm-i915-hdmi-Debug-for-scrambling-polling.patch](./patches/0002-drm-i915-hdmi-Debug-for-scrambling-polling.patch) | Adds debug around SCDC polling; instrumentation only |
| 8 | 2026-01-14 08:00 | [patches/v2-0001-drm-i915-hdmi-Add-SCDC-processing-delay-for-scramb.patch](./patches/v2-0001-drm-i915-hdmi-Add-SCDC-processing-delay-for-scramb.patch) | 150ms delay immediately after SCDC config; v2 rationale + tests |

## Community feedback
| Date | Person | Topic | Link |
|---|---|---|---|
| 2025-12-30 | Jani Nikula | Request for GitLab issue + debug logs | [msg582211](https://mail-archive.com/dri-devel@lists.freedesktop.org/msg582211.html) |
| 2026-01-06 | Ankit Nautiyal | SCDC polling patch proposal (200ms) | [GitLab #6868](https://gitlab.freedesktop.org/drm/xe/kernel/-/issues/6868) |
| 2026-01-13 | Ankit Nautiyal | RESEND polling patch to intel-gfx | [msg371959](https://mail-archive.com/intel-gfx@lists.freedesktop.org/msg371959.html) |
| 2026-01-14 | Ville Syrjälä | Questions placement: move delay forward to find exact requirement | [msg371965](https://mail-archive.com/intel-gfx@lists.freedesktop.org/msg371965.html) |
| 2026-01-21 | Ankit Nautiyal | Response to Ville: all placements work; suggests panel-specific quirk? | [msg372336](https://mail-archive.com/intel-gfx@lists.freedesktop.org/msg372336.html) |
| 2026-01-28 | Jerome Tollet | Follow-up asking for preferred approach (polling vs quirk vs delay) | [msg372765](https://mail-archive.com/intel-gfx@lists.freedesktop.org/msg372765.html) |
| 2026-02-09 | Jerome Tollet | Gentle ping summarizing three possible approaches | [msg373401](https://mail-archive.com/intel-gfx@lists.freedesktop.org/msg373401.html) |
| 2026-02-14 | Jerome Tollet | **Isolated testing confirms: SCDC polling alone is the solution** | _(pending archive)_ |

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
