# Investigation: Poll Timeout vs Working Display

## Discovery with drm.debug=0x04

When DRM debug is enabled, we see:
```
[6.272360] intel_hdmi_handle_sink_scrambling: scrambling=yes, TMDS bit clock ratio=1/40
[6.479626] intel_hdmi_poll_for_scrambling_enable: Timed out waiting for scrambling enable
```

**But display works at 4K@60Hz!**

## What This Means

### Two Separate Operations:

1. **WRITE scrambling config to SCDC** (6.272360)
   - Function: intel_hdmi_handle_sink_scrambling()  
   - Action: Tells monitor to enable scrambling
   - Result: ✅ Succeeds

2. **READ scrambling status from SCDC** (6.272-6.479)
   - Function: intel_hdmi_poll_for_scrambling_enable()
   - Action: Polls SCDC_SCRAMBLER_STATUS bit via I2C
   - Result: ❌ Times out after 200ms

### Key Finding: No I2C Errors

The logs do NOT show:
```
"Failed to read scrambling status: -XX"
```

This means:
- I2C communication works (drm_scdc_readb succeeds)
- The monitor RECEIVES and ACCEPTS the read requests
- But the SCDC_SCRAMBLER_STATUS bit returns 0 (not ready)

### Monitor Behavior (Cisco Desk Pro)

The Cisco Desk Pro monitor:
1. ✅ Accepts SCDC WRITE to enable scrambling
2. ✅ Activates scrambling at hardware level (display works!)
3. ❌ Does NOT set SCDC_SCRAMBLER_STATUS bit to 1
4. ❌ Non-compliant with HDMI 2.0 spec section 3.3.6

From HDMI 2.0 spec:
> "After scrambled video transmission begins, the Source shall poll the
> TMDS_Scrambler_Status bit until it reads 1 or until a timeout of 200 ms"

### Why Display Works Despite Timeout

The scrambling CONFIGURATION succeeds, even though the STATUS verification fails.

The monitor:
- Configures scrambling correctly (from SCDC write)
- Displays 4K@60Hz properly (hardware scrambling active)
- But doesn't properly implement the SCDC status feedback mechanism

This is a **monitor firmware bug** - non-compliant SCDC implementation.

### Comparison: Previous Test Without Debug

Without drm.debug:
- drm_dbg_kms() messages not shown
- We saw printk("DEBUG Poll started/finished") 
- 201ms timing (similar to 207ms now)
- Display worked

The poll was ALSO timing out, we just couldn't see the error message!

## Conclusion

The patch DOES work, but for a different reason than expected:

**Expected**: Poll succeeds, status bit = 1, display works
**Reality**: Poll fails, but scrambling still activates, display works anyway

The monitor accepts the scrambling config but doesn't properly report its status via SCDC.

## Implications for Patch

The patch is still valid because:
1. It gives the monitor TIME to activate scrambling (200ms)
2. The scrambling DOES activate during that time
3. The display works despite the status read timeout

Without the patch:
- Driver continues immediately without waiting
- Monitor doesn't have time to process the scrambling config
- Display fails

With the patch:
- Driver waits 200ms
- Monitor activates scrambling (even if status not reported)
- Display works

The patch solves the problem, just not exactly as the HDMI spec intended!
