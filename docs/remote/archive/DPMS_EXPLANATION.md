# DPMS ON/OFF Resume Issues - Explanation

## What is DPMS?

DPMS (Display Power Management Signaling) is a standard for managing display power states:
- **DPMS_ON**: Display fully active
- **DPMS_STANDBY**: Display in low-power standby
- **DPMS_SUSPEND**: Display suspended
- **DPMS_OFF**: Display completely powered off

kmscon uses DPMS_OFF after the configured timeout (default 5 minutes), then DPMS_ON when activity resumes.

## The Problem Observed

After kmscon DPMS timeout, when the display wakes up (DPMS_OFF → DPMS_ON), the monitor displays:
```
"Unable to decode incoming video signal"
```

This indicates the HDMI signal is present but the monitor cannot interpret it correctly.

## Root Cause Analysis

### HDMI 2.0 Scrambling State Machine

For 4K@60Hz (594 MHz pixel clock), HDMI 2.0 requires:
1. **Scrambling enabled** via SCDC (Status and Control Data Channel) - I2C communication with monitor
2. **High TMDS Character Rate** flag set in TRANS_DDI_FUNC_CTL register (bit 4)
3. **HDMI Scrambling** flag set in TRANS_DDI_FUNC_CTL register (bit 0)

### The DPMS Sequence in i915

#### Display Disable (DPMS_OFF):
1. `intel_ddi_post_disable_hdmi()` is called
2. Disables TRANS_DDI_FUNC_CTL register (bit 31 = 0)
3. **SCDC scrambling configuration may be lost**
4. Monitor enters power-saving mode

#### Display Enable (DPMS_ON):
1. `intel_ddi_pre_enable_hdmi()` is called
2. Re-enables TRANS_DDI_FUNC_CTL register with all flags
3. **But SCDC scrambling must be reconfigured via I2C**

### The Race Condition

The i915 driver sequence during DPMS resume:
```c
intel_ddi_pre_enable_hdmi()
  ├─ Configure TRANS_DDI_FUNC_CTL (registers)
  └─ intel_hdmi_configure_scdc() (I2C communication)
       ├─ Wait for monitor to be ready
       └─ Write SCDC scrambling enable
```

**Problem**: If the monitor isn't fully awake when SCDC configuration happens, the I2C transaction succeeds but the monitor doesn't properly apply the scrambling settings.

## Why the 100ms Delay Failed

The attempted fix was:
```c
/* Give monitor time to wake from DPMS before configuring scrambling */
msleep(100);
intel_hdmi_configure_scdc(encoder, crtc_state);
```

**Why it failed**: This delay was added to `intel_ddi_enable_hdmi()`, which is called:
1. At **boot time** (initial display configuration)
2. During **DPMS resume**
3. During **mode changes**

Adding a 100ms delay to ALL activations caused:
- Black screen at boot (too early in the sequence)
- Broken initial display configuration
- Mode change delays

## The VC Payload Bit Workaround Connection

The official Intel workaround we implemented:
```c
temp |= TRANS_DDI_DP_VC_PAYLOAD_ALLOC;  /* bit 8 */
```

**May actually help with DPMS issues** because:

1. **Maintains internal state**: Keeping bit 8 ON during HDMI/DVI maintains the VC (Virtual Channel) payload allocation state through power transitions

2. **Hardware state consistency**: The transcoder keeps its configuration more consistent between DPMS_OFF and DPMS_ON

3. **Reduces reconfiguration**: Less hardware state needs to be rebuilt during resume

However, this workaround **does not solve the SCDC I2C timing issue** - it only helps with the internal GPU state.

## Proper Fix Needed

A correct fix would:

1. **Detect DPMS resume** in `intel_ddi_pre_enable_hdmi()`:
```c
bool is_dpms_resume = /* detect if coming from DPMS_OFF */
if (is_dpms_resume) {
    /* Give monitor extra time to wake */
    msleep(100);
}
```

2. **Verify SCDC readiness** before configuring:
```c
/* Poll SCDC Device_Ready bit before writing */
int timeout = 200; /* ms */
while (timeout-- > 0) {
    if (scdc_read_device_ready(hdmi))
        break;
    msleep(1);
}
```

3. **Retry SCDC configuration** if it fails:
```c
for (int retry = 0; retry < 3; retry++) {
    if (intel_hdmi_configure_scdc(encoder, crtc_state) == 0)
        break;
    msleep(50);
}
```

## Current Workaround

Setting `dpms-timeout=0` in `/etc/kmscon/kmscon.conf` prevents the issue by disabling DPMS power management entirely. The display stays in DPMS_ON state continuously.

**Trade-offs**:
- ✅ No DPMS resume issues
- ❌ Higher power consumption when idle
- ❌ Reduced display lifespan (always on)

## Testing the VC Payload Bit Effect

To test if the VC payload bit workaround helps with DPMS:

1. Apply the current patch (VC payload bit always ON for HDMI)
2. Re-enable DPMS: `dpms-timeout=300` (5 minutes)
3. Wait for DPMS timeout
4. Move mouse to trigger DPMS_ON
5. Check if monitor correctly decodes signal

**Hypothesis**: The VC payload bit may improve DPMS resume reliability by maintaining more consistent hardware state, even if it doesn't completely solve the SCDC timing issue.

## References

- kmscon DPMS implementation: https://github.com/kmscon/kmscon/pull/177
- Intel Tiger Lake Vol 14 Workarounds: "[MF][20h1] Blank screen seen with 4 MST Displays"
- HDMI 2.0 SCDC specification: VESA SCDC 2.0
- i915 HDMI scrambling code: `drivers/gpu/drm/i915/display/intel_hdmi.c`
