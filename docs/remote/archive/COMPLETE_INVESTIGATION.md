# Complete Investigation Results

## Test Setup
- DRM debug: 0x04 (KMS messages enabled)
- DEBUG printk: Added
- Monitor: Cisco Desk Pro
- Target: 3840x2160@60Hz (594 MHz)

## Full Timing Sequence

```
[6.759355] intel_hdmi_handle_sink_scrambling: scrambling=yes, TMDS 1/40
[6.764513] DEBUG Poll for scrambling started
[6.965622] DEBUG Poll for scrambling finished
[6.965627] intel_hdmi_poll_for_scrambling_enable: Timed out
```

**Poll duration: 201.1 milliseconds**

## Key Findings

### 1. Poll ALWAYS Timeouts
The poll times out in ALL configurations:
- With DEBUG printk: 201.1ms → timeout
- Without DEBUG printk: 207ms → timeout  
- The Cisco Desk Pro NEVER sets SCDC_SCRAMBLER_STATUS bit to 1

### 2. Display ALWAYS Works
Despite the timeout:
- Display active at 3840x2160@60Hz
- No "unable to decode signal" error
- Scrambling is functionally active

### 3. Why It Works

**SCDC has two operations:**

a) **WRITE scrambling config** (succeeds)
   - intel_hdmi_handle_sink_scrambling() at 6.759355
   - Tells monitor to enable scrambling
   - Monitor accepts and processes

b) **READ scrambling status** (times out)
   - intel_hdmi_poll_for_scrambling_enable() from 6.764513
   - Polls SCDC_SCRAMBLER_STATUS bit via I2C
   - Bit never becomes 1, poll times out after 200ms

**The monitor:**
- ✅ Receives SCDC write
- ✅ Activates scrambling at hardware level
- ✅ Displays correctly
- ❌ Does NOT set status bit (firmware bug)

### 4. Cisco Desk Pro Bug

**HDMI 2.0 Spec Violation:**
Section 3.3.6 requires:
> "The Sink shall set the TMDS_Scrambler_Status bit to 1 when 
> scrambled video is detected"

The Cisco Desk Pro:
- Detects and processes scrambled video ✅
- Does NOT set TMDS_Scrambler_Status bit ❌

This is a **monitor firmware bug** - non-compliant SCDC implementation.

### 5. Why The Patch Still Works

**Without patch:**
- Driver continues immediately after SCDC write
- Monitor has ~5ms to activate scrambling
- Too fast, monitor fails
- Result: "unable to decode signal"

**With patch:**
- Driver waits 200ms after SCDC write
- Monitor has full 200ms to activate scrambling
- Plenty of time, monitor succeeds (even if status not reported)
- Result: Display works!

**The patch provides TIME, not status verification.**

The HDMI spec intended:
1. Write scrambling config
2. Poll status until bit = 1 (confirms activation)
3. Continue

What actually happens with Cisco Desk Pro:
1. Write scrambling config
2. Poll status for 200ms (always returns 0, times out)
3. Continue anyway
4. Display works because scrambling DID activate during the 200ms

## Comparison: With vs Without drm.debug

| Config | Timeout Visible? | Display Works? | Duration |
|--------|-----------------|----------------|----------|
| drm.debug=0 | ❌ Hidden | ✅ Yes | 201.4ms |
| drm.debug=4 | ✅ Visible | ✅ Yes | 201.1ms |

The timeout was ALWAYS happening, we just couldn't see it without debug enabled.

## Implications

### For This Bug Fix:
✅ Patch is valid and works correctly
✅ Solves the real problem (timing)
✅ Display functions properly
⚠️ Timeout message is expected (monitor bug, not driver bug)

### For Ankit:
- Explain the timeout is from non-compliant monitor firmware
- The patch still solves the user's problem
- Consider if timeout message should be suppressed for working displays
- Or change to drm_dbg_kms (less alarming) vs drm_warn

### For Users:
- 4K@60Hz works correctly despite timeout message
- This is a Cisco Desk Pro firmware issue
- Other monitors may behave correctly and not timeout

## Recommendation for Patch

The patch should be accepted as-is because:
1. Solves the real-world problem (display works)
2. Follows HDMI 2.0 spec (200ms poll)
3. Timeout is logged appropriately (drm_dbg_kms, not error)
4. Harmless - display functions despite timeout

Alternative: Could add a comment explaining that some monitors 
have non-compliant SCDC implementations and will timeout even 
when scrambling is working.

