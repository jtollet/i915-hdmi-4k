# 4K@60Hz HDMI Solution on Intel Alder Lake-N

## Problem
The Cisco Desk Pro monitor could not decode the 4K@60Hz HDMI signal with the i915 driver, while the BIOS worked correctly.

## Root Cause
The `PIPE_DDI_FUNC_CTL_A` register differed between BIOS and i915:
- **BIOS**: `0x88030111` (bit 8 enabled)
- **i915**: `0x88030011` (bit 8 disabled)

Bit 8 (`TRANS_DDI_DP_VC_PAYLOAD_ALLOC`) is normally used for DisplayPort, but some HDMI 2.0 monitors require it for high bandwidth modes (≥ 594 MHz).

## Solution
Patch in `drivers/gpu/drm/i915/display/intel_ddi.c` (line ~582):

```c
/* For 4K@60Hz, set bit 8 like BIOS does */
if (crtc_state->port_clock >= 594000)
    temp |= TRANS_DDI_DP_VC_PAYLOAD_ALLOC;
```

This condition enables bit 8 only for HDMI modes with clock ≥ 594 MHz (4K@60Hz and higher resolutions).

## Verifications Performed

✅ **VBT Configuration**: 594 MHz max (correct)  
✅ **HDMI 2.0 Scrambling**: Enabled correctly (1/40 ratio)  
✅ **BPC**: 8 bits per color (identical to BIOS)  
✅ **Video Timings**: CVT standard 3840x2160@60Hz  
✅ **PIPE_DDI_FUNC_CTL_A Register**: Now identical to BIOS

## Installation

```bash
cd /home/<user>/devel/i915/linux-src
git apply /home/<user>/devel/i915/fix-4k60hz-hdmi.patch
make -j$(nproc) M=drivers/gpu/drm/i915
cd /home/<user>/devel/i915
sudo ./install_i915.sh
sudo reboot
```

## Tested Hardware
- **GPU**: Intel Alder Lake-N (N100)
- **Monitor**: Cisco Desk Pro
- **Resolution**: 3840x2160@60Hz via HDMI
- **Kernel**: 6.18.1

## Investigation Details

### Register Comparison (BIOS vs i915)
Out of 173 registers captured, only 17 differed. The critical difference was:

```
PIPE_DDI_FUNC_CTL_A:
  BIOS: 0x88030111
  i915: 0x88030011
  XOR:  0x00000100 (bit 8 = TRANS_DDI_DP_VC_PAYLOAD_ALLOC)
```

### What Works Correctly in i915
- VBT limits: 594 MHz (sufficient for 4K@60Hz)
- HDMI 2.0 scrambling: Properly activated with 1/40 TMDS ratio
- SCDC communication: Working
- Video timings: Identical to BIOS (CVT 3840x2160@60Hz)
- BPC: 8 bits per color (matching BIOS)

### What Was Missing
The `TRANS_DDI_DP_VC_PAYLOAD_ALLOC` bit (bit 8) was not set for HDMI high bandwidth modes. This bit is typically used for DisplayPort VC (Virtual Channel) payload allocation, but some HDMI 2.0 monitors appear to require it for proper signal decoding at high pixel clocks.

## Before/After

**Before patch:**
- Monitor displays: "unable to decode incoming video signal"
- PIPE_DDI_FUNC_CTL_A = 0x88030011
- Screen remains black

**After patch:**
- 4K@60Hz works perfectly
- PIPE_DDI_FUNC_CTL_A = 0x88030111 (matches BIOS)
- Full resolution and refresh rate achieved

## Resolution Date
December 29, 2025
