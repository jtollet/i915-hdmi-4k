## Solution Found ✅

After extensive investigation, I have identified the root cause and developed a patch that fixes 4K@60Hz on my Cisco Desk Pro monitor.

### Root Cause

The `PIPE_DDI_FUNC_CTL_A` register differs between BIOS (working) and i915:
- **BIOS**: `0x88030111` (bit 8 = 1)
- **i915**: `0x88030011` (bit 8 = 0)

Bit 8 (`TRANS_DDI_DP_VC_PAYLOAD_ALLOC`) is normally used for DisplayPort, but some HDMI 2.0 monitors require it for high bandwidth modes (≥ 594 MHz).

### Solution

The following patch conditionally enables bit 8 for HDMI modes ≥ 594 MHz:

```diff
diff --git a/drivers/gpu/drm/i915/display/intel_ddi.c b/drivers/gpu/drm/i915/display/intel_ddi.c
index c09aa759f..a5a8df16c 100644
--- a/drivers/gpu/drm/i915/display/intel_ddi.c
+++ b/drivers/gpu/drm/i915/display/intel_ddi.c
@@ -580,6 +580,9 @@ intel_ddi_transcoder_func_reg_val_get(struct intel_encoder *encoder,
 			temp |= TRANS_DDI_HDMI_SCRAMBLING;
 		if (crtc_state->hdmi_high_tmds_clock_ratio)
 			temp |= TRANS_DDI_HIGH_TMDS_CHAR_RATE;
+		/* For 4K@60Hz, set bit 8 like BIOS does */
+		if (crtc_state->port_clock >= 594000)
+			temp |= TRANS_DDI_DP_VC_PAYLOAD_ALLOC;
 		if (DISPLAY_VER(display) >= 14)
 			temp |= TRANS_DDI_PORT_WIDTH(crtc_state->lane_count);
 	} else if (intel_crtc_has_type(crtc_state, INTEL_OUTPUT_ANALOG)) {
```

### Verifications Performed

✅ **VBT Configuration**: 594 MHz max (correct)  
✅ **HDMI 2.0 Scrambling**: Enabled correctly (1/40 ratio)  
✅ **BPC**: 8 bits per color (identical to BIOS)  
✅ **Video Timings**: CVT standard 3840x2160@60Hz  
✅ **PIPE_DDI_FUNC_CTL_A Register**: Now identical to BIOS  
✅ **Result**: 4K@60Hz works perfectly ✨

### Tested Hardware
- **GPU**: Intel Alder Lake-N (N100)
- **Monitor**: Cisco Desk Pro
- **Resolution**: 3840x2160@60Hz via HDMI
- **Kernel**: 6.18.1

### Investigation Summary

Out of 173 GPU registers compared between BIOS and i915, only 17 differed. The critical difference was bit 8 in `PIPE_DDI_FUNC_CTL_A`. 

Everything else was correctly configured by i915:
- VBT TMDS clock limits
- HDMI 2.0 scrambling and high TMDS ratio
- Video timings (CVT standard)
- Color depth (8 bpc)

The `TRANS_DDI_DP_VC_PAYLOAD_ALLOC` bit is typically used for DisplayPort Virtual Channel payload allocation, but it appears some HDMI 2.0 monitors require it for proper signal decoding at high pixel clocks.

**Before patch**: Monitor shows "unable to decode incoming video signal"  
**After patch**: 4K@60Hz works perfectly

I have a properly formatted patch ready with `Signed-off-by` that passes `checkpatch.pl` with 0 errors, 0 warnings. Should I create a Merge Request?
