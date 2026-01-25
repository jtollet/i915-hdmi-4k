# Response to Ankit Nautiyal - SCDC Polling Test Results

Hi @aknautiyal,

Thank you for the patches! I've tested them thoroughly and I'm happy to report **the SCDC polling patch works perfectly**.

## Test Summary

### Configuration
- **Hardware**: Intel Alder Lake-N N100 (device ID 0x46d1)
- **Monitor**: Cisco Desk Pro
- **Target**: 3840x2160@60Hz (594 MHz pixel clock, HDMI 2.0 with scrambling)
- **Kernel**: Linux 6.18.1
- **Patches**: Only your SCDC polling patch + debug printk (no other workarounds)

### Result: ✅ SUCCESS

**Display works perfectly at 4K@60Hz!**
- No "unable to decode incoming video signal" error
- Display activates correctly on boot
- Stable operation

## Polling Timeout - Answer to Your Question

**The poll did NOT timeout.** Here are the detailed timings from your debug patch:

```
[    6.452101] DEBUG Poll for scrambling started
[    6.653498] DEBUG Poll for scrambling finished
```

**Polling duration: ~201.4 milliseconds**

Key observations:
- The scrambling bit took approximately 201ms to activate
- This is just at the edge of the 200ms spec timeout
- **No "Timed out waiting for scrambling enable" message appeared**
- The poll completed successfully just before timeout

This suggests that:
1. The Cisco Desk Pro monitor needs almost the full 200ms timeout period
2. The polling mechanism works correctly
3. Without the polling, the driver would continue too early, before the monitor is ready

## Important Finding

I previously had a workaround patch that set the TRANS_DDI_DP_VC_PAYLOAD_ALLOC bit (bit 8) for high bandwidth HDMI. 

**I removed that patch entirely for this test**, and the SCDC polling patch alone fixes the issue completely. This confirms that:
- The root cause was the missing SCDC polling delay
- The VC_PAYLOAD bit workaround is NOT needed
- Your patch is the correct fix

## Regarding Xe Driver

Good point about intel-xe@lists.freedesktop.org. Since the display code is shared, this fix will benefit both drivers. I'll send the patch there as well.

## Attached Files

1. **dmesg_scdc_polling_only.txt** (82KB) - Complete kernel log with debug messages
2. **display_info_scdc_polling_only.txt** (6.3KB) - Full i915_display_info output
3. **scdc_debug_extract.txt** - Relevant SCDC log excerpt
4. **TEST_RESULTS_SCDC_POLLING.md** - Detailed test report

Let me know if you need any additional information or testing!

Tested-by: Jerome Tollet <jerome.tollet@gmail.com>
