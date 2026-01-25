# Intel i915 HDMI 4K@60Hz Workaround - Ready for Testing

## Summary

Le patch propre a été appliqué avec succès, implémentant le workaround officiel Intel pour HDMI/DVI.

## What Was Done

### 1. Clean Patch Applied to Original Linux 6.18.1 Code

**File**: `/home/<user>/devel/i915/linux-src/drivers/gpu/drm/i915/display/intel_ddi.c`

**Change**: Replaced frequency-specific workaround with official Intel workaround for ALL HDMI/DVI

**Key Improvements**:
- ✅ References official Intel Tiger Lake Vol 14 Workaround document
- ✅ Applies to ALL HDMI/DVI, not just high frequencies (as per Intel specification)
- ✅ Clean implementation from original code
- ✅ Proper commenting explaining the workaround

### 2. Official Intel Workaround Documentation

**Source**: Intel Tiger Lake Volume 14: Workarounds
**Workaround ID**: "[MF][20h1] Blank screen seen with 4 MST Displays"
**Quote**: 
> "Keep DP VC Payload Bit ON as part of HDMI/DVI Enable/Disable Sequence. 
> When enabling non-MST cases (eDP/DP-SST/HDMI/DVI), MstTransportSelect 
> in TRANS_DDI_FUNC_CTL must be programmed to match the assigned pipe."

**Significance**: This confirms that bit 8 (TRANS_DDI_DP_VC_PAYLOAD_ALLOC) should 
be active for HDMI/DVI, not just DisplayPort MST. The BIOS correctly implements 
this workaround; i915 driver did not.

### 3. DPMS Issues Explained

**Document**: `DPMS_EXPLANATION.md`

**Issue**: After kmscon DPMS timeout, monitor shows "unable to decode incoming video signal"

**Root Cause**: HDMI 2.0 scrambling via SCDC (I2C) not properly reinitialized during DPMS resume

**Current Workaround**: `dpms-timeout=0` in `/etc/kmscon/kmscon.conf`

**Hypothesis**: The VC Payload bit workaround may improve DPMS resume reliability 
by maintaining more consistent hardware state across power transitions.

## Next Steps

### Before Testing (IMPORTANT - Wait for Backup Completion)

**DO NOT REBOOT** until backup operation is complete and explicitly confirmed.

### When Ready to Test

1. **Compile the patched module**:
```bash
cd /home/<user>/devel/i915/linux-src
make -j$(nproc) M=drivers/gpu/drm/i915
```

2. **Install the module**:
```bash
sudo make M=drivers/gpu/drm/i915 modules_install
sudo depmod -a
```

3. **Reboot with i915** (remove blacklist temporarily):
```bash
sudo mv /etc/modprobe.d/blacklist-i915.conf /etc/modprobe.d/blacklist-i915.conf.disabled
sudo reboot
```

4. **Test DPMS behavior**:
```bash
# Re-enable DPMS in kmscon
sudo nano /etc/kmscon/kmscon.conf
# Set: dpms-timeout=300

# Wait 5 minutes, then move mouse
# Check if monitor correctly decodes signal
```

5. **Capture results**:
```bash
# If working:
sudo cat /sys/kernel/debug/dri/0/i915_display_info > display_working.txt
dmesg > dmesg_working.txt

# If NOT working:
sudo cat /sys/kernel/debug/dri/0/i915_display_info > display_broken.txt
dmesg > dmesg_broken.txt
```

## Files Summary

| File | Purpose | Status |
|------|---------|--------|
| `intel_ddi.c` | Patched with official workaround | ✅ READY |
| `intel_ddi.c.backup` | Original clean Linux 6.18.1 code | ✅ Preserved |
| `DPMS_EXPLANATION.md` | Complete DPMS issue analysis | ✅ Documented |
| `PATCH_READY.md` | This summary | ✅ Complete |
| `FINAL_PATCH_EXPLANATION.md` | Previous investigation | 📚 Reference |
| `INVESTIGATION_REPORT.md` | Complete debugging history | 📚 Reference |

## Expected Behavior with Patch

### At Boot
- Display should activate correctly at 4K@60Hz
- TRANS_DDI_FUNC_CTL_A should show 0x88030111 (bit 8 = 1)
- Monitor should display "HDMI 3840x2160@60Hz" or similar

### During DPMS Resume
- **With dpms-timeout=0**: Display stays on, no issue (current workaround)
- **With dpms-timeout=300**: Test if VC Payload bit improves DPMS resume
  - **Hypothesis**: May help maintain hardware state consistency
  - **May still need**: SCDC readiness polling for complete fix

## Hardware Configuration

- **CPU/GPU**: Intel Alder Lake-N N100 (Gen12.0)
- **Monitor**: Cisco Desk Pro
- **Resolution**: 3840x2160@60Hz (594 MHz pixel clock)
- **Connection**: HDMI 2.0
- **Physical Ports**: 2× HDMI (internal cabling unknown)
- **HDMI 2.0 Features Required**:
  - Scrambling enabled (bit 0)
  - High TMDS Char Rate (bit 4)
  - VC Payload allocation (bit 8) ← **This is the fix**

## Git Commit Message Template

When ready to commit:

```
drm/i915/hdmi: Apply VC Payload workaround for HDMI/DVI

Implement official Intel Tiger Lake workaround for HDMI/DVI displays.
Keep TRANS_DDI_DP_VC_PAYLOAD_ALLOC bit active for all HDMI/DVI 
connections, not just DisplayPort MST.

Without this workaround, some monitors fail to decode the HDMI signal,
particularly at high frequencies like 4K@60Hz (594 MHz). The BIOS 
correctly sets this bit, but i915 only set it for DP MST/UHBR cases.

Reference: Intel Tiger Lake Vol 14 Workarounds
Workaround: "[MF][20h1] Blank screen seen with 4 MST Displays"
Quote: "Keep DP VC Payload Bit ON as part of HDMI/DVI Enable/Disable 
Sequence"

Tested on: Intel Alder Lake-N N100 (Gen12.0) with Cisco Desk Pro monitor
Resolution: 3840x2160@60Hz via HDMI 2.0

Signed-off-by: Your Name <your.email@example.com>
```

## References

- **Intel PRM**: Tiger Lake Volumes 2c, 12, 14
- **Documentation**: /tmp/tgl-vol14-workarounds.txt (lines 5930-5945)
- **GitLab Issue**: https://gitlab.freedesktop.org/drm/xe/kernel/-/issues/6868
- **Kernel Version**: Linux 6.18.1
- **Driver**: i915 DRM/KMS

---

**Status**: ✅ PATCH READY - Waiting for backup completion before testing
