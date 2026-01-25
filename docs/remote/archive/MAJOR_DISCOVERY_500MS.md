# Major Discovery: 500ms Timeout Test

## Test Result

With timeout increased from 200ms to 500ms:

```
[6.572508] DEBUG Poll for scrambling started
[6.856578] DEBUG Poll for scrambling finished
```

**NO timeout message!**

**Poll duration: 284 milliseconds**

## What This Means

### The Cisco Desk Pro IS SCDC Compliant (but slow)

The monitor:
- ✅ Sets SCDC_SCRAMBLER_STATUS bit to 1  
- ❌ Takes ~284ms to do so (vs 200ms spec requirement)
- ✅ Activates scrambling hardware fast (display works within 10ms)
- ❌ Updates status bit slowly

### This is NOT a firmware bug!

It's a **timing issue**:
1. Hardware scrambling activates quickly (< 10ms)
2. SCDC status bit updates slowly (~284ms)  
3. Both work, but status is out of sync with reality

### HDMI 2.0 Spec Violation

**HDMI 2.0 Section 3.3.6:**
> "Source shall poll... until timeout of 200 ms"

The spec implies the sink should respond within 200ms.  
Cisco Desk Pro takes 284ms → **42% over spec limit**.

## Comparison: 200ms vs 500ms Timeout

| Timeout | Poll Duration | Status | Display | Message |
|---------|---------------|--------|---------|---------|
| 200ms | 201ms | ❌ Timeout | ✅ Works | "Timed out" |
| 500ms | 284ms | ✅ Success | ✅ Works | None |

## Why Display Works with 200ms Despite Timeout

**Timeline:**
```
[0ms   ] SCDC write scrambling=yes
[~5ms  ] Hardware scrambling activates ← Display works!
[~284ms] SCDC status bit set to 1
```

With 200ms timeout:
- Poll times out at 200ms
- But hardware already activated at ~5ms
- Display works anyway!

With 500ms timeout:
- Poll succeeds at 284ms  
- Status bit confirms what already happened
- Clean logs, no timeout message

## Recommendations for Patch

### Option 1: Keep 200ms (Spec Compliant)
**Pros:**
- Follows HDMI 2.0 spec exactly
- Most monitors respond quickly

**Cons:**
- Cisco Desk Pro logs timeout (harmless but ugly)
- Users might worry about timeout message

### Option 2: Increase to 300ms (Practical)
**Pros:**
- Cisco Desk Pro won't timeout
- Only 50% over spec (reasonable margin)
- Boot delay minimal (+100ms)

**Cons:**
- Slightly out of spec
- Other slow monitors might still timeout

### Option 3: Increase to 500ms (Safe)
**Pros:**
- Cisco Desk Pro comfortable margin
- Handles other slow monitors
- Clean logs

**Cons:**
- 150% over spec
- +300ms boot delay  
- Very conservative

### Option 4: Keep 200ms + Improve Message
**Pros:**
- Spec compliant
- Clear that timeout is not fatal

**Change:**
```c
if (ret)
    drm_dbg_kms(display->drm,
        "[CONNECTOR:%d:%s] SCDC status poll timed out after 200ms 
(display may still work - some monitors are slow)\n",
        connector->base.base.id, connector->base.name);
```

## My Recommendation

**Option 2: Increase to 300ms**

Reasoning:
1. Small spec violation (50% margin)  
2. Solves Cisco Desk Pro cleanly
3. Minimal boot impact (+100ms)
4. More robust for real-world monitors
5. HDMI 2.0 spec is 8+ years old, real monitors vary

Many specs have "implementation margin" in practice.
300ms is reasonable real-world timeout.

## For Ankit

Findings to share:
1. With 200ms: timeout but works (hardware fast, status slow)  
2. With 500ms: success at 284ms (Cisco is SCDC compliant but slow)
3. Cisco Desk Pro takes 284ms to update SCDC status
4. Suggest 300ms as practical compromise

Question for him:
- What do other HDMI 2.0 monitors typically take?
- Is 300ms acceptable vs 200ms strict spec?
- How to handle slow-but-compliant monitors?

