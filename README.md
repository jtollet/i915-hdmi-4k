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
