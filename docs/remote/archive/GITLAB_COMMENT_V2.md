## Update: Partial Analysis and Unstable Behavior

Hello,

I've made progress analyzing the 4K@60Hz HDMI issue with my Intel Alder Lake-N (N100).

### Hardware Configuration
- GPU: Intel Alder Lake-N (device 0x46d1)
- Monitor: Cisco Desk Pro (CS-DESKPRO-2)
- Resolution: 3840x2160@60Hz (594 MHz pixel clock)
- Works perfectly in BIOS
- Fails when i915 driver loads

### Register Analysis

I compared registers between BIOS state (working) and i915 (failing) and identified a critical difference in the `PIPE_DDI_FUNC_CTL_A` register:

**BIOS state (working):** `0x88030111`
**i915 state (failing):** `0x88030011`

The difference is **bit 8** (`TRANS_DDI_DP_VC_PAYLOAD_ALLOC`).

### Tested Patch

I created a patch that forces bit 8 for clocks ≥ 594 MHz in `intel_ddi_transcoder_func_reg_val_get()`:

```c
/* For 4K@60Hz HDMI, set bit 8 like BIOS does */
if (crtc_state->port_clock >= 594000)
    temp |= TRANS_DDI_DP_VC_PAYLOAD_ALLOC;
```

### Observed Behavior

The patch shows **unstable behavior**:
- ✅ Sometimes works on initial boot
- ❌ Becomes unreliable after system use or reboots
- ✅ Register correctly shows `0x88030111`
- ❌ But monitor displays "unable to decode incoming video signal"
- ✅ In comparison, `simpledrm` (using BIOS configuration) works consistently

### Questions for Maintainers

1. **Is it legitimate to use `TRANS_DDI_DP_VC_PAYLOAD_ALLOC` for HDMI?** This bit appears DisplayPort-specific according to its documentation, but BIOS clearly enables it for high-frequency HDMI 2.0.

2. **Are other registers needed?** The fact that bit 8 alone doesn't achieve stable behavior suggests there might be additional configuration differences.

3. **Are there known timing issues?** HDMI 2.0 scrambling and SCDC configuration might require specific delays or initialization sequences.

### Ongoing Investigation

I'm continuing the analysis by comparing **all registers** between simpledrm (stable) and i915 (unstable) to identify any additional differences beyond bit 8.

Results from this comprehensive analysis will be shared soon.

Thank you for your help on this issue.
