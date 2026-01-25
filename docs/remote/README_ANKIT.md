# Files for Ankit Nautiyal - SCDC Polling Patch Testing

## 📋 Contents

This directory contains all materials for responding to Ankit's SCDC polling patch on GitLab.

### 1. Main Response
**File:** `RESPONSE_TO_ANKIT.md` (6.0KB)

Complete response for GitLab with:
- Test results confirming patch works perfectly
- Answer to "did the poll timeout?" with detailed analysis
- **Critical finding: Display works even with 200ms timeout**
- Statistical timing data from 3 independent boots
- Two timeout recommendations (200ms vs 300ms)
- Tested-by tag included

### 2. Patch Options

#### Option A: Spec Compliant (200ms)
**File:** `0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS-Scrambler-.patch` (2.6KB)

- Follows HDMI 2.0 specification exactly
- Display works correctly even when timeout occurs
- Enhanced timeout message clarifies it's not a functional failure
- Tested-by tag included

#### Option B: Real-World Compatible (300ms) ⭐ RECOMMENDED
**File:** `0001-drm-i915-hdmi-Poll-for-300-msec-for-TMDS-Scrambler-.patch` (2.9KB)

- 100% success rate on Cisco Desk Pro (all 3 boots)
- Clean logs with no spurious timeouts
- Reasonable compromise (50% over spec vs 150% for 500ms)
- Detailed commit message explaining rationale
- Tested-by tag included

### 3. Supporting Documentation

**File:** `CRITICAL_FINDING.md` (5.9KB)
- Detailed analysis showing display works despite 200ms timeout
- Explains two independent timing paths (hardware vs firmware)
- Root cause analysis of Cisco Desk Pro behavior

**File:** `SCDC_TIMING_ANALYSIS.md` (6.0KB)
- Statistical analysis of 3 boot tests
- Complete timing measurements
- Timeout recommendations with rationale

**File:** `dmesg_complete_investigation.txt` (214KB)
- Complete kernel logs with drm.debug=0x04
- Shows actual SCDC polling behavior
- Confirms perfect 4K@60Hz operation

## 🎯 Key Messages for Ankit

### 1. THE PATCH WORKS! ✅
The SCDC polling patch completely fixes the Cisco Desk Pro 4K@60Hz issue.

### 2. DISPLAY WORKS EVEN WITH 200ms TIMEOUT ✅
This is the most important finding:
- **Hardware scrambling:** ~5ms (what matters for display)
- **SCDC status bit:** 217-284ms (software verification only)
- Timeout only affects verification, not functionality

### 3. ROOT CAUSE IDENTIFIED ✅
Cisco Desk Pro has:
- ✅ Compliant scrambling hardware (<100ms, actual ~5ms)
- ❌ Non-compliant SCDC firmware (>200ms, actual 217-284ms)

### 4. NO OTHER PATCHES NEEDED ✅
- VC_PAYLOAD bit workaround is NOT required
- SCDC polling alone fixes the issue completely

## 📊 Test Results Summary

### Configuration
- Hardware: Intel Alder Lake-N N100 (device 0x46d1)
- Monitor: Cisco Desk Pro
- Target: 3840x2160@60Hz (594 MHz TMDS clock)
- Kernel: Linux 6.18.1
- Patches: Only SCDC polling patch (no other workarounds)

### Timing Measurements (3 boots)
```
Boot #1: 284.1 ms (42% over 200ms spec)
Boot #2: 267.4 ms (34% over 200ms spec)
Boot #3: 217.0 ms (9% over 200ms spec)
Average: 256.2 ms (28% over 200ms spec)
```

### Results
- ✅ Display works perfectly at 4K@60Hz
- ✅ Scrambling active (TMDS ratio 1/40)
- ✅ Zero "unable to decode" errors
- ⚠️ Poll times out with 200ms (but display still works!)
- ✅ Poll succeeds with 300ms (clean logs)

## 💡 My Recommendation

**Option B (300ms timeout)** because:
1. 100% proven success rate across all tests
2. Clean logs (no spurious timeout messages)
3. Reasonable real-world compromise
4. Accommodates monitors with slow SCDC firmware
5. Still maintains fast operation

However, **both options are valid**. The display works perfectly either way!

## 📤 How to Submit

### For GitLab Response:

1. Copy content of `RESPONSE_TO_ANKIT.md`

2. Attach chosen patch:
   - Option A: `0001-drm-i915-hdmi-Poll-for-200-msec-for-TMDS-Scrambler-.patch`
   - Option B: `0001-drm-i915-hdmi-Poll-for-300-msec-for-TMDS-Scrambler-.patch` ⭐

3. Optionally attach supporting docs:
   - `CRITICAL_FINDING.md`
   - `SCDC_TIMING_ANALYSIS.md`

4. Post on GitLab issue/merge request

### GitLab Location
https://gitlab.freedesktop.org/drm/i915/kernel

## 📁 Archive

All previous versions, test files, and drafts have been moved to `archive/` directory (36 files) for reference.

---

**Tested-by:** Jerome Tollet <jerome.tollet@gmail.com>
**Date:** January 7, 2025
**Status:** Ready for submission ✅
