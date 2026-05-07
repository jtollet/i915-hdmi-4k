# drm/i915/hdmi: Poll for 200 msec for TMDS_Scrambler_Status

Patch fixing 4K@60Hz HDMI display issues on Intel i915 (Alder Lake and similar).

## Problem

Some HDMI 2.0 sinks fail to come up at 4K@60Hz (594 MHz) because the sink
has not finished its scrambling setup before the source proceeds. This results
in a blank screen or failed modeset.

## Fix

HDMI 2.0 section 6.1.3.1 specifies that after enabling scrambling, the source
should poll `TMDS_Scrambler_Status` until it reads 1, or until a 200 ms
timeout expires. This patch adds that polling step in `intel_ddi_enable_hdmi()`.

## Applying the patch

```bash
git apply 0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS_Scrambler_S.patch
```

Or with `git am` from a kernel tree:

```bash
git am 0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS_Scrambler_S.patch
```

## Status

- Reviewed-by: Arun R Murthy <arun.r.murthy@intel.com>
- Submitted to intel-gfx mailing list (v3)
- Patchwork: https://patchwork.freedesktop.org/series/160028/
- Issue: https://gitlab.freedesktop.org/drm/xe/kernel/-/issues/6868
