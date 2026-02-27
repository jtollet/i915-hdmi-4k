# Changelog

Chronological summary of the patch variants created during the HDMI 4K@60Hz
investigation.

## 2025-12-30
- Initial timing-delay proposal (add fixed sleeps before SCDC config and after
  DDI enable) to stabilize 4K@60Hz scrambling on sensitive sinks.

## 2026-01-07
- Polling variants for TMDS_Scrambler_Status (200 ms / 300 ms) to align with
  HDMI 2.0 spec behavior.

## 2026-01-08
- v2 polling patch posted (poll for up to 200 ms after enabling HDMI), with
  formal SCDC status polling helpers.

## 2026-01-13
- Test patch series moving the delay earlier in the HDMI enable sequence to
  isolate the minimal required timing window.

## 2026-01-14
- v2 SCDC processing delay patch: move the delay immediately after SCDC
  configuration for clarity and maintainability, plus improved rationale.

## 2026-01-21
- Ankit Nautiyal suggests considering panel-specific quirk instead of generic
  solution, given that all delay placements worked in testing.

## 2026-01-28
- Follow-up message asking maintainers for preferred approach: generic polling,
  panel-specific quirk, or fixed delay.

## 2026-02-09
- Gentle ping after 10 days with summary of three possible approaches awaiting
  maintainer feedback.

## 2026-02-14
- **Isolated testing on kernel 6.18.7 confirms: SCDC polling patch alone works
  perfectly. Delay patch is placement-dependent and less robust.**
- Conclusion: Generic SCDC polling (Ankit's patch) is the recommended solution
  as it aligns with HDMI 2.0 spec and Windows behavior.

## 2026-02-17
- Ankit Nautiyal responds, requests to continue discussion in original thread
  on lore.kernel.org instead of fragmenting across multiple threads.
- Jerome acknowledges and posts update with Feb 14 isolated testing results
  showing SCDC polling patch alone works perfectly.

## 2026-02-20
- Jerome confirms he will post final summary in the proper thread.

## 2026-02-21
- Final confirmation message sent to intel-gfx and dri-devel mailing lists with
  isolated testing results (kernel 6.19.2) confirming SCDC polling patch works
  perfectly (msg373961).

## 2026-02-27
- Gentle ping sent to intel-gfx, dri-devel, Ankit and Ville in reply to
  Ankit's Jan 21 message thread as requested.

## Additional variants (archive)
- Earlier combined fixes and experimental patches kept for reference in
  `patches/local-archives` and `patches/remote-archive`.

## Notes
- Full patch list: `docs/PATCHES_INDEX.md`
- Supporting docs: `docs/DOCS_INDEX.md`
- Mail and issue links: `docs/MAILS.md`
