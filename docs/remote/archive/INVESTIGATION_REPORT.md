# 4K@60Hz HDMI Investigation Report - Intel Alder Lake-N

## Hardware Configuration
- **GPU**: Intel Alder Lake-N (N100), device ID 0x46d1
- **Monitor**: Cisco Desk Pro (CS-DESKPRO-2)
- **Resolution**: 3840x2160@60Hz
- **Pixel Clock**: 594 MHz (HDMI 2.0 with scrambling)
- **Kernel**: 6.18.1

## Problem Statement
4K@60Hz HDMI works perfectly in BIOS and with simpledrm, but fails when i915 driver loads. Monitor displays "unable to decode incoming video signal".

## Investigation Results

### 1. Register Comparison: simpledrm (working) vs i915 (broken)

Complete register dump comparison revealed **17 differences**. Key findings:

#### Critical Register: PIPE_DDI_FUNC_CTL_A (0x60400)
- **simpledrm**: 0x88030111 (bit 8 SET)
- **i915**: 0x88030011 (bit 8 CLEAR)
- **Difference**: Bit 8 (TRANS_DDI_DP_VC_PAYLOAD_ALLOC)

**Patch attempted**: Force bit 8 for port_clock >= 594000
**Result**: Bit 8 gets set correctly, but display still fails

#### Critical Finding: Display Plane Configuration

**DSPACNTR** (Display Plane A Control, 0x70180):
- **simpledrm**: 0x84000008
  - Format bits [29:26] = 1 (UNDOCUMENTED FORMAT)
  - Bit 10 (TILED) = 0 (linear)
- **i915**: 0x94000400
  - Format bits [29:26] = 5 (BGRX565, 16bpp)
  - Bit 10 (TILED) = 1 (X-tiled)

**DSPASTRIDE** (Display Plane A Stride, 0x70188):
- **simpledrm**: 0x000000f0 (240 x 64 = 15360 bytes for 3840x4 = 32bpp)
- **i915**: 0x0000001e (30 x 64 = 1920 bytes - INCORRECT)

**DSPASURF** (Display Plane A Surface, 0x7019c):
- **simpledrm**: 0x00000000
- **i915**: 0x0002c000 (different memory location)

### 2. Root Cause Analysis

**BIOS uses UNDOCUMENTED display format value 1**. This format is not listed in Intel documentation:
- Documented formats: 2-15 (8BPP, BGRA555, BGRX565, etc.)
- BIOS format: 1 (unknown, appears to be 32bpp)

When i915 calls i9xx_get_initial_plane_config():
1. Reads format value 1 from DSPACNTR
2. i9xx_format_to_fourcc() doesn't recognize format 1
3. Falls through to default: XRGB8888
4. **BUT** i915 then reconfigures the entire display instead of reusing BIOS framebuffer
5. New configuration has wrong stride and format

### 3. Power Well Differences

**HSW_PWR_WELL_CTL1** (0x45400):
- simpledrm: 0x00005405
- i915: 0x00000401

**HSW_PWR_WELL_CTL2** (0x45404):
- simpledrm: 0x0000fc0f
- i915: 0x00000c03

**HSW_PWR_WELL_CTL4** (0x4540c):
- simpledrm: 0x00005405
- i915: 0x00000401

### 4. Attempted Solutions

#### Attempt 1: Set bit 8 only
- **Code**: Force TRANS_DDI_DP_VC_PAYLOAD_ALLOC for port_clock >= 594000
- **Result**: Bit 8 set correctly (0x88030111) but display still fails
- **Conclusion**: Bit 8 alone is insufficient

#### Attempt 2: Handle format 1 in i9xx_get_initial_plane_config
- **Code**: Detect format 1, force BGRX888 and linear tiling
- **Result**: Code not executed (i915 doesn't reuse BIOS framebuffer)
- **Conclusion**: i915 creates new framebuffer instead of reusing

#### Attempt 3: Use i915.fastboot=1
- **Code**: Kernel parameter to preserve BIOS configuration
- **Result**: Still fails, i915 cannot interpret format 1
- **Conclusion**: fastboot can't work without recognizing format 1

#### Attempt 4: Manually force all registers
- **Code**: Write simpledrm values to all display registers
- **Result**: i915 immediately overwrites them
- **Conclusion**: Register forcing not viable

### 5. Complete Register Difference List

All 17 register differences between working (simpledrm) and broken (i915):

```
GEN6_RPNSWREQ (0xa008):           0x06000000 -> 0x16800000
HSW_PWR_WELL_CTL1 (0x45400):      0x00005405 -> 0x00000401
HSW_PWR_WELL_CTL2 (0x45404):      0x0000fc0f -> 0x00000c03
HSW_PWR_WELL_CTL4 (0x4540c):      0x00005405 -> 0x00000401
PIPE_DDI_FUNC_CTL_B (0x61400):    0x00030000 -> 0x00000000
PIPE_DDI_FUNC_CTL_C (0x62400):    0x00030000 -> 0x00000000
DDI_BUF_CTL_C (0x64200):          0x00000080 -> 0x00000000
DDI_BUF_CTL_D (0x64300):          0x00000080 -> 0x00000000
DDI_BUF_CTL_E (0x64400):          0x00000080 -> 0x00000000
DSPACNTR (0x70180):               0x84000008 -> 0x94000400
DSPASTRIDE (0x70188):             0x000000f0 -> 0x0000001e
DSPASURF (0x7019c):               0x00000000 -> 0x0002c000
DSPBCNTR (0x71180):               0x00000008 -> 0x00000000
DSPCCNTR (0x72180):               0x00000008 -> 0x00000000
VBLANK_A (0x60010):               0x08c9086f -> 0x08c90000
SDEISR (0xc4000):                 0x00010000 -> 0x00013000
RC6_RESIDENCY_TIME (0x138108):    0x0355e513 -> 0x19fa4255
```

## Questions for Intel Maintainers

1. **What is display format value 1?**
   - Is it a valid format for Alder Lake-N?
   - What pixel format does it represent?
   - Why isn't it documented in i9xx_plane_regs.h?

2. **Why does BIOS use format 1 specifically for 4K@60Hz?**
   - Lower resolutions don't use this format
   - Is it related to bandwidth/performance?

3. **Why is bit 8 (TRANS_DDI_DP_VC_PAYLOAD_ALLOC) set for HDMI?**
   - This bit is documented for DisplayPort
   - BIOS sets it for HDMI 2.0 at high pixel clocks
   - Is this intentional or BIOS bug?

4. **How to make i915 properly reuse BIOS framebuffer?**
   - i9xx_get_initial_plane_config() seems not effective
   - fastboot=1 doesn't help when format is unknown
   - Should format 1 be added to i9xx_format_to_fourcc()?

## Workaround

**Current solution**: Blacklist i915, use simpledrm

```bash
echo "blacklist i915" > /etc/modprobe.d/blacklist-i915.conf
update-initramfs -u
```

**Result**: Stable 4K@60Hz using BIOS framebuffer configuration.

## Files for Reference

- registers_simpledrm_working.txt - Full register dump with simpledrm
- registers_i915_broken.txt - Full register dump with i915
- register_differences.txt - Detailed comparison
- combined_4k_fix.patch - Attempted fix (bit 8 + format 1 handling)

## System Information

```
GPU: Intel Alder Lake-N (N100)
PCI ID: 8086:46d1
Kernel: 6.18.1-061801-generic
i915 version: Compiled from mainline 6.18.1
HDMI: HDMI 2.0 with scrambling enabled
EDID: 3840x2160@60Hz, 594 MHz pixel clock
```
