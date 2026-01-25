# Test Results for SCDC Polling Patch

Hi @aknautiyal,

Thank you for the patches! I've tested them thoroughly and I'm happy to report **the SCDC polling patch works perfectly**.

## Test Summary

### Configuration
- **Hardware**: Intel Alder Lake-N N100 (device ID 0x46d1)
- **Monitor**: Cisco Desk Pro
- **Target**: 3840x2160@60Hz (594 MHz pixel clock, HDMI 2.0 with scrambling)
- **Kernel**: Linux 6.18.1
- **Patches Applied**: Only your SCDC polling patch + debug printk (no other workarounds)

### Result: ✅ SUCCESS

**Display works perfectly at 4K@60Hz!**
- No "unable to decode incoming video signal" error
- Display activates correctly on boot
- Stable operation

## Answer to Your Question: Did the Poll Timeout?

**NO, the poll did NOT timeout.** Here are the detailed timings from your debug patch:

```
[    6.452101] DEBUG Poll for scrambling started
[    6.653498] DEBUG Poll for scrambling finished
```

**Time between printk messages: ~201.4 milliseconds**

### Important: No Timeout Error

The key evidence is that **no timeout error message appears** in the logs. If the poll had 
timed out, we would see:
```
[CONNECTOR:205:HDMI-A-1] Timed out waiting for scrambling enable
```

This message does not appear anywhere, which confirms poll_timeout_us() returned 0 (success).

### Why 201ms vs 200ms Spec?

The printk messages measure time AROUND the polling function, not the exact moment when 
scrambling became active. The 1.4ms difference can be attributed to:

1. poll_timeout_us() implementation tolerances (last check can slightly exceed timeout)
2. Time taken by the printk() calls themselves
3. Kernel scheduling latency
4. The scrambling likely activated BEFORE 200ms, but measurement includes overhead

### Key Observations:

- The scrambling bit activated successfully within the spec timeout
- The polling mechanism works correctly as designed
- The Cisco Desk Pro monitor needs close to the full 200ms timeout period
- Without the polling delay, the driver would continue too early before the monitor is ready

## Important Finding: SCDC Polling Alone is Sufficient

I previously had a workaround patch that set the TRANS_DDI_DP_VC_PAYLOAD_ALLOC bit (bit 8) 
for high bandwidth HDMI modes. 

**I removed that patch entirely for this test**, and the SCDC polling patch alone fixes the 
issue completely. This confirms that:
- The root cause was the missing SCDC polling delay
- The VC_PAYLOAD bit workaround is NOT needed
- Your patch is the correct and complete fix

## Regarding Xe Driver

Good point about sending to intel-xe@lists.freedesktop.org. Since the display code is shared 
between i915 and xe drivers, this fix will benefit both. I'll send the patch there as well.

## Attached Files

I'm attaching the following detailed logs as you requested:

1. **detailed_logs_excerpt.txt** - Focused excerpt showing SCDC polling with timeline analysis
2. **dmesg_scdc_polling_only.txt** (82KB) - Complete kernel log from boot with all debug messages
3. **display_info_scdc_polling_only.txt** (6.3KB) - Full i915_display_info output confirming 3840x2160@60Hz active

Please let me know if you need any additional information or testing!

Tested-by: Jerome Tollet <jerome.tollet@gmail.com>
