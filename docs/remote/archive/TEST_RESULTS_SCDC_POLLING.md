# Test Results: SCDC Polling Patch (Without VC_PAYLOAD bit 8 patch)

## Test Configuration
- **Date**: 2026-01-07
- **Kernel**: Linux 6.18.1
- **GPU**: Intel Alder Lake-N N100 (device ID 0x46d1)
- **Monitor**: Cisco Desk Pro
- **Target Resolution**: 3840x2160@60Hz (594 MHz pixel clock)

## Patches Applied
1. ✅ **SCDC Polling Patch**: Poll for 200ms for TMDS_Scrambler_Status (commit 28d53174c)
2. ✅ **Debug Patch**: Ankit's debug printk to measure polling duration
3. ❌ **VC_PAYLOAD Patch**: NOT applied (intentionally removed to test SCDC polling alone)

## Test Results

### ✅ SUCCESS - Display Works Perfectly!

**Display Status:**
- Resolution: 3840x2160@60Hz
- Pixel Clock: 594 MHz
- Status: enable=yes, active=yes
- HDMI Port: HDMI-A-1
- No "unable to decode signal" error

**SCDC Polling Timing:**
```
[    6.452101] DEBUG Poll for scrambling started
[    6.653498] DEBUG Poll for scrambling finished
```

**Polling Duration: ~201.4 milliseconds**
- Very close to the 200ms timeout
- **NO timeout error message** - scrambling activated successfully
- Poll completed just before timeout

### Key Findings

1. **SCDC Polling Patch ALONE is sufficient**
   - The VC_PAYLOAD_ALLOC bit 8 patch is NOT necessary
   - Display works perfectly with only SCDC polling

2. **Scrambling Activation Time**
   - Takes approximately 201ms to activate
   - Just under the 200ms spec timeout
   - Suggests the sink needs almost the full timeout period

3. **No Errors or Warnings**
   - No "Timed out waiting for scrambling enable" message
   - i915 loads cleanly
   - Display activates on first boot

## Conclusion

The SCDC polling patch successfully resolves the 4K@60Hz HDMI 2.0 issue without requiring any additional workarounds. The previous VC_PAYLOAD bit 8 patch was not the root cause fix.

## Files Attached
- dmesg_scdc_polling_only.txt - Full kernel log
- display_info_scdc_polling_only.txt - Complete display configuration
- scdc_debug_extract.txt - SCDC-specific log excerpt

## Hardware Details
```
GPU: Intel Alder Lake-N (N100), Gen 13.0
PCI ID: 8086:46d1
HDMI: HDMI 2.0 with scrambling
Monitor: Cisco Desk Pro (3840x2160@60Hz native)
```

Tested-by: Jerome Tollet <jerome.tollet@gmail.com>
